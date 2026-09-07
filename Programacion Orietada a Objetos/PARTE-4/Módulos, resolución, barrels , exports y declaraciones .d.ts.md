# 1. TEORÍA
# 1.1 ES Modules (ESM) vs. CommonJS (CJS): Los Dos Sistemas de Módulos
En el ecosistema de JavaScript, coexisten dos sistemas principales para organizar y compartir código entre archivos. Entenderlos es clave para configurar correctamente tu paquete.

- **CommonJS (CJS): El Sistema Histórico de Node.js**
    * **Sintaxis**: Usa `require()` para importar y `module.exports` para exportar.
    * **Características**: Es un sistema dinámico y síncrono. Cuando Node.js encuentra un `require()` , detiene la ejecución, carga y ejecuta el archivo, y luego continúa. Es el sistema tradicional del ecosistema Node.js y sigue siendo muy común.

- **ES Modules (ESM): El Estándar Moderno de JavaScript**

    * **Sintaxis**: Usa `import` y `export` .
    * **Características**: Es el estándar oficial de JavaScript, soportado tanto en navegadores como en versiones modernas de Node.js. Su naturaleza estática (las importaciones y exportaciones se resuelven en tiempo de compilación/análisis, no en ejecución) permite optimizaciones como el **tree-shaking**, donde los bundlers (como Webpack o Vite) pueden eliminar de forma segura el código que no se utiliza, resultando en paquetes más pequeños.

# 1.2 Campos Clave en `package.json` para Módulos y Tipos
El archivo `package.json` es el cerebro de tu paquete. Los siguientes campos le indican a herramientas como Node.js, bundlers y TypeScript cómo encontrar y utilizar tu código.

- `type` : El interruptor principal.
    * `"module"` : Le dice a Node.js que los archivos .js de tu proyecto deben ser tratados como ES Modules por defecto.
    * `"commonjs"` : (Valor por defecto si no se especifica) Los archivos .js se tratan como CommonJS.

- `main` , `module` : Los Puntos de Entrada "Legados".
    * `"main"` : El punto de entrada principal para el entorno CommonJS. Es el que Node.js usaba tradicionalmente al hacer `require('mi-paquete')` .
    * `"module"` : Un campo no oficial pero ampliamente adoptado por bundlers para encontrar la versión ESM de tu paquete. Ha sido mayormente reemplazado por `exports` .

- `exports` : La Solución Moderna y Recomendada. Este campo es un "mapa de entradas" que le permite a tu paquete exponer diferentes archivos para diferentes entornos de forma explícita y controlada. Resuelve la ambigüedad de los campos anteriores y ofrece un control mucho más granular.
``` typescript
    {
      "exports": {
        // Punto de entrada principal (ej: import &#39;mi-paquete&#39;)
        ".": {
          "types": "./dist/index.d.ts",   // Para TypeScript
          "import": "./dist/index.js",    // Para import (ESM)
          "require": "./dist/index.cjs"   // Para require (CJS)
        },
        // Punto de entrada secundario (ej: import &#39;mi-paquete/utils&#39;)
        "./utils": {
          "types": "./dist/utils.d.ts",
          "import": "./dist/utils.js",
          "require": "./dist/utils.cjs"
        }
      }
    }
```
**Ventaja**: `exports` es la única fuente de verdad. Si un archivo no está listado aquí, no se puede importar desde fuera del paquete, lo que mejora la encapsulación.

- `types` (o typings ): El Punto de Entrada de Tipos.
    * Indica la ubicación de tu archivo principal de declaración de tipos ( `.d.ts` ). Es crucial para que los usuarios de _TypeScript_ obtengan autocompletado y seguridad de tipos.
    * **Nota**: Si usas el campo `exports` , la forma moderna es definir los tipos dentro de cada entrada con `"types"` , como se ve en el ejemplo anterior.

- `files` : El "Whitelist" de Publicación.
    * Es un array de archivos y directorios que se incluirán cuando publiques tu paquete en npm. Esto evita que publiques archivos innecesarios (como tests, configuraciones, etc.).
    * Ejemplo: [`"dist/", "README.md", "LICENSE"`] .

- `sideEffects` : Una Pista para la Optimización.
    * Este campo le dice a los bundlers si tu código tiene "efectos secundarios" (por ejemplo, si un archivo modifica variables globales o importa un CSS solo por existir).
    * Al configurarlo como `"sideEffects": false` , garantizas que tus módulos son puros. Esto le da permiso al bundler para aplicar el tree-shaking de forma mucho más agresiva, ya que puede eliminar un import si no se usa directamente, sabiendo que no romperá nada.

# 1.3 Resolución de módulos en TS
- `moduleResolution : Bundler` (recomendado con Vite/tsup), `NodeNext` (Node ESM/CJS), o `Node` (legacy).
- `baseUrl` / `paths` : alias de importación (no confundir con `exports` ).
```typescript
{
  "compilerOptions": {
    "baseUrl": ".",
    "paths": { "@lib/*": ["src/*"] }
  }
}
```
_Los alias **no** cambian el runtime; bundlers o tsconfig-paths deben resolverlos._

# 1.4 Barrels (archivos `index.ts` )
- Re-exportan símbolos desde un "paquete" interno para simplificar imports.
- Preferí `export type` para re-exportar **solo tipos** y evitar importaciones en runtime.
- Cuidado con **ciclos** de imports al encadenar muchos barrels .

# 1.5 Declaraciones .d.ts
- Se generan automáticamente con `tsc` ( `"declaration": true` ) o con bundlers (p. ej., `tsup --dts` ).
- **Ambient declarations** : agregan tipos sin importar un módulo concreto.
``` typescript
// global.d.ts
declare global { interface Window { appVersion?: string } }
export {} // convierte el archivo en módulo y aplica las globals
```
- Module declarations : tipar módulos JS sin tipos:
``` typescript
// types/untyped-lib.d.ts
declare module "untyped-lib" { export function greet(n: string): string }
```
- Module augmentation : extender tipos existentes de una lib:
``` typescript
// types/fastify.d.ts
declare module "fastify" { interface FastifyRequest { userId?: string } }
```

# 1.6 `import type` / `export type`
- Eliminan importaciones **solo de tipos** en JS emitido, reduciendo sobrecarga y ciclos.

# 1.7 Estrategias de publicación
- Monopackage simple: compilar a `dist/` con ESM/CJS + d.ts.
- Verificá **consumo dual** (ESM y CJS) y los tipos en un proyecto consumer .
- Publicación real: `npm publish` (usar `npm pack` y `files` para previsualizar el contenido).

# 2. Práctica con npm
Carpeta `lib/` (librería) y carpeta `consumer/` (proyecto de prueba). Asumimos `tsup` instalado.

# 2.1 Estructura
lib/
  src/
    index.ts
    utils.ts
  tsconfig.json
  package.json
consumer/
  src/demo.ts
  tsconfig.json
  package.json

# 2.2 Configuración de la librería ( `lib/` )
`lib/tsconfig.json`
``` typescript
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "ESNext",
    "moduleResolution": "Bundler",
    "declaration": true,
    "emitDeclarationOnly": false,
    "outDir": "dist",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "baseUrl": ".",
    "paths": { "@lib/*": ["src/*"] }
  },
  "include": ["src", "types"]
}
```
----------------------------------------------------
`lib/package.json` (fragmento)
``` typescript
{
  "name": "@demo/ts-lib",
  "version": "0.1.0",
  "type": "module",
  "main": "./dist/index.cjs",
  "module": "./dist/index.js",
  "types": "./dist/index.d.ts",
  "exports": {
    ".": {
      "types": "./dist/index.d.ts",
      "import": "./dist/index.js",
      "require": "./dist/index.cjs"
    },
    "./utils": {
      "types": "./dist/utils.d.ts",
      "import": "./dist/utils.js",
      "require": "./dist/utils.cjs"
    }
  },
  "files": ["dist", "README.md", "LICENSE"],
  "sideEffects": false,
  "scripts": {
    "typecheck": "tsc --noEmit",
    "build": "tsup src/index.ts src/utils.ts --format esm,cjs --dts --sourcemap",
    "clean": "rimraf dist"
  },
  "devDependencies": {
    "tsup": "^8.0.0",
    "typescript": "^5.5.0",
    "rimraf": "^5.0.0"
  }
}
```
--------------------------------------------------
`lib/src/utils.ts`
``` typescript
export type NonEmpty<T extends string> = T extends "" ? never : T
export function greet<T extends string>(name: NonEmpty<T>) {
  return `Hello, ${name}!`
}
export const sum = (a: number, b: number) => a + b
```
--------------------------------------------------
`lib/src/index.ts`
``` typescript
export { greet, sum } from "./utils"
export type { NonEmpty } from "./utils" // export type evita emitir import en JS
```
---------------------------------------------------
**Augmentación opcional**( `lib/types/fastify.d.ts` )
``` typescript
declare module "fastify" { interface FastifyRequest { userId?: string } }
```
---------------------------------------------------
**Build**
cd lib
npm run typecheck
npm run build

# 2.3 Publicación local y consumo
**Crear paquete y consumirlo**
cd lib
npm pack                   # genera @demo-ts-lib-0.1.0.tgz
cd ../consumer
npm init -y
npm i -D typescript ts-node
npm i ../lib/@demo-ts-lib-0.1.0.tgz

`consumer/tsconfig.json`
``` typescript
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "ESNext",
    "moduleResolution": "Bundler",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true
  },
  "include": ["src"]
}
```
---------------------------------------------------
`consumer/src/demo.ts`
``` typescript
import { greet, sum, type NonEmpty } from "@demo/ts-lib"

const n: NonEmpty<"Ada"> = "Ada"
console.log(greet(n))
console.log(sum(2, 3))
```
---------------------------------------------------
**Ejecución**
npx ts-node src/demo.ts
_Probá también importar subruta: import { greet } from "@demo/ts-lib/utils" ._

# 2. 4 Aliases vs `exports`
- Si usás `paths` ( `@lib/*` ) dentro de **lib** , configurá el bundler o usa `tsconfig-paths` para runtime.
- El consumer no ve tus `paths` ; expone rutas estables vía `exports` .

# 2.5 Buenas prácticas
- `export type` / `import type` para evitar emisiones innecesarias.
- `files` + `sideEffects: false` para paquetes limpios y tree-shakeables .
- Verificá el **contenido del tarball** : `npm pack --dry-run` .
- Evitá ciclos en barrels ; si aparecen, rompe el ciclo o convierte re-exports a `import type` .

# Comandos útiles
# en lib
npm run clean && npm run build
npm pack --dry-run
# en consumer
npx ts-node src/demo.ts