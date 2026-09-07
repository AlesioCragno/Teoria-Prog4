# 1. TEORÍA
# 1.1 Promesas y `async/await`
- Una función `async` **siempre** retorna `Promise<T>` (donde `T` es el tipo del `return` ).
- `await` suspende hasta que la promesa se resuelva o rechace.
- Buenas prácticas: usar `try/catch/finally` para **limpieza de recursos** y mapeo de errores.
``` typescript
async function leer(): Promise<string> {
  // ...
  return "ok" // Promise<string>
}
```

# 1.2 Tipado de errores
- El canal de error de una `Promise` es **no tipado** . En TS, `catch (e)` es `unknown` .
- Preferí **modelar errores** en el tipo de retorno ( `Result` / `Either` ) o normalizar `unknown` a un error propio.
``` typescript
function toError(e: unknown): Error {
  return e instanceof Error ? e : new Error(String(e))
}
```

# 1.3 Concurrencia
- `Promise.all` : falla rápido si una falla (rechazo corto).
- `Promise.allSettled` : nunca lanza; reporta estados.
- `Promise.race` : primero en resolver/rechazar.
- **No confundir** concurrencia (varias promesas en curso) con paralelismo (varios núcleos).

# 1.4 Timeouts y cancelación
- `AbortController` + `fetch` (Node ≥18) para cancelación.
- `Promise.race` para implementar timeouts de alto nivel.

# 1.5 Validación como contrato
Esquemas (Zod) para **entrada** ( `req.body , params , query` ) y **salida** ( `response` ).
Tipos inferidos de los esquemas → **tipado end-to-end** .

# 1.6 Endpoints tipados en Fastify
- `fastify-type-provider-zod` conecta Zod con Fastify: valida y **tipa** `req` y `reply` .
- Manejo centralizado de errores; respuestas con **uniones discriminadas** (p. ej., `{ ok: true, data } | { ok: false, error }` ).

# 2. PRÁCTICA
# 2.1 Instalación
npm i fastify zod
npm i -D tsx fastify-type-provider-zod
**Scripts** ( `package.json` ):
``` typescript
{
  "scripts": {
    "serve": "tsx watch src/server.ts",
    "typecheck": "tsc --noEmit",
    "build": "tsc -p tsconfig.json",
    "start": "node dist/server.js"
  }
}
```

# 2.2 Utilidades de asincronía ( `src/lib/async.ts` )
``` typescript
export class TimeoutError extends Error { constructor(msg = "Timeout") { super(msg) } }

export async function withTimeout<T>(p: Promise<T>, ms: number): Promise<T> {
  let t: NodeJS.Timeout
  const timeout = new Promise<never>((_, rej) => (t = setTimeout(() => rej(new TimeoutError()), ms)))
  try { return await Promise.race([p, timeout]) } finally { clearTimeout(t!) }
}

export function toError(e: unknown): Error { return e instanceof Error ? e : new Error(String(e)) }
```

# 2.3 Esquemas de dominio ( `src/schemas.ts` )
``` typescript
import { z } from "zod"

export const Item = z.object({
  id: z.string().uuid(),
  name: z.string().min(1),
  price: z.number().nonnegative()
})
export const NewItem = Item.omit({ id: true })
export const IdParam = z.object({ id: z.string().uuid() })

export type TItem = z.infer<typeof Item>
export type TNewItem = z.infer<typeof NewItem>

```

# 2.4 Repositorio en memoria ( `src/lib/repo.ts` )
``` typescript
import type { TItem, TNewItem } from "../schemas"

export class ItemRepo {
  #m = new Map<string, TItem>()
  list(): TItem[] { return [...this.#m.values()] }
  get(id: string) { return this.#m.get(id) }
  create(data: TNewItem): TItem {
    const id = crypto.randomUUID()
    const item = { id, ...data }
    this.#m.set(id, item)
    return item
  }
  update(id: string, data: TNewItem): TItem | undefined {
    if (!this.#m.has(id)) return undefined
    const item = { id, ...data }
    this.#m.set(id, item)
    return item
  }
  remove(id: string) { return this.#m.delete(id) }
}
```

# 2.5 Rutas ( src/routes/items.ts )
``` typescript
import { z } from "zod"
import { Item, NewItem, IdParam } from "../schemas"
import type { FastifyPluginAsync } from "fastify"

const ItemsRoutes: FastifyPluginAsync = async (app) => {
  app.get("/items", { schema: { response: { 200: z.array(Item) } } }, async () => {
    return app.repo.list()
  })

  app.get("/items/:id", { schema: { params: IdParam, response: { 200: Item } } }, async (req, reply) => {
    const item = app.repo.get(req.params.id)
    if (!item) return reply.code(404).send()
    return item
  })

  app.post("/items", { schema: { body: NewItem, response: { 201: Item } } }, async (req, reply) => {
    const created = app.repo.create(req.body)
    return reply.code(201).send(created)
  })

  app.put("/items/:id", { schema: { params: IdParam, body: NewItem, response: { 200: Item } } }, async (req, reply) => {
    const updated = app.repo.update(req.params.id, req.body)
    if (!updated) return reply.code(404).send()
    return updated
  })

  app.delete("/items/:id", { schema: { params: IdParam } }, async (req, reply) => {
    const ok = app.repo.remove(req.params.id)
    return reply.code(ok ? 204 : 404).send()
  })
}

export default ItemsRoutes
```

# 2.6 Servidor ( `src/server.ts` )
``` typescript
import Fastify from "fastify"
import { ZodTypeProvider, serializerCompiler, validatorCompiler } from "fastify-type-provider-zod"
import ItemsRoutes from "./routes/items"
import { ItemRepo } from "./lib/repo"
import { toError } from "./lib/async"

declare module "fastify" {
  interface FastifyInstance { repo: ItemRepo }
}

const app = Fastify({ logger: true }).withTypeProvider<ZodTypeProvider>()
app.setValidatorCompiler(validatorCompiler)
app.setSerializerCompiler(serializerCompiler)
app.decorate("repo", new ItemRepo())

app.get("/health", async () => ({ ok: true }))
app.register(ItemsRoutes)

app.setErrorHandler((err, _req, reply) => {
  const e = toError(err)
  app.log.error(e)
  reply.code(500).send({ ok: false, error: e.message })
})

const port = Number(process.env.PORT ?? 3000)
app.listen({ port, host: "0.0.0.0" }).catch((e) => {
  app.log.error(e); process.exit(1)
})
```

# 2.7 Cliente con cancelación ( `src/client.ts` )
``` typescript
import { withTimeout, toError } from "./lib/async"

export async function getJson<T>(url: string, ms = 3000): Promise<T> {
  const ctrl = new AbortController()
  const p = (async () => {
    const res = await fetch(url, { signal: ctrl.signal })
    if (!res.ok) throw new Error(`HTTP ${res.status}`)
    return (await res.json()) as T
  })()
  try { return await withTimeout(p, ms) } catch (e) { ctrl.abort(); throw toError(e) }
}

// Demo: getJson<{ ok: boolean }>("http://localhost:3000/health").then(console.log)
```

# 3. Mini–katas (entregables)
- Agregá `GET /search?minPrice&maxPrice` con validación de `querystring` (Zod) y retorno `200: Item[]` .
- Añadí un **middleware** que mida duración de cada request y la registre en el logger.
- Implementá `getJson` con `retry` exponencial (máx. 3 intentos) manteniendo tipos del genérico `T` .
- Tipá un resultado discriminado `{ ok: true; data: T } | { ok: false; error: string }` y usalo en `client.ts` .
- Extraé el contrato de `/items` a un archivo de tipos y usalo tanto en el servidor como en el cliente ( compile-time contract ).

# 4. Comandos útiles
npm run serve         # desarrollo con recarga
npm run typecheck     # verificación de tipos
npm run build && npm start
curl -s http://localhost:3000/health | jq

# Apéndices
- A. `tsconfig.json` mínimo estricto
```typescript
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "ESNext",
    "moduleResolution": "Bundler",
    "strict": true,
    "noUncheckedIndexedAccess": true,
    "forceConsistentCasingInFileNames": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "outDir": "dist"
  },
  "include": ["src"]
}
```
- B. Plantilla de `package.json` (scripts clave)
```typescript
{
  "type": "module",
  "scripts": {
    "dev": "ts-node src/index.ts",
    "watch": "tsc -w",
    "typecheck": "tsc --noEmit",
    "build": "tsup src/index.ts --format cjs,esm --dts --sourcemap",
    "start": "node dist/index.js",
    "clean": "rimraf dist",
    "lint": "eslint . --ext .ts",
    "lint:fix": "eslint . --ext .ts --fix",
    "format": "prettier -w .",
    "format:check": "prettier -c .",
    "test": "vitest run",
    "test:watch": "vitest",
    "test:cov": "vitest run --coverage",
    "serve": "tsx watch src/server.ts",
    "ci": "npm run typecheck && npm run lint && npm run test && npm run build",
    "prepublishOnly": "npm run typecheck && npm run build"
  },
  "lint-staged": {
    "**/*.{ts,tsx}": ["prettier -w", "eslint --fix"]
  }
}
```
- A. `tsconfig.json` mínimo estricto
├─ src/
│  ├─ lib/
│  ├─ server.ts
│  ├─ index.ts
│  ├─ 01-types.ts
│  ├─ 02-modelado.ts
│  └─ 04-generics.ts
├─ tests/
│  └─ server.test.ts
├─ dist/
├─ tsconfig.json
├─ package.json
└─ README.md