---
title: Crear y probar en Node.js
intro: Puedes crear un flujo de trabajo de integración continua (CI) para construir y probar tu proyecto Node.js.
redirect_from:
  - /actions/automating-your-workflow-with-github-actions/using-nodejs-with-github-actions
  - /actions/language-and-framework-guides/using-nodejs-with-github-actions
  - /actions/guides/building-and-testing-nodejs
versions:
  fpt: '*'
  ghes: '*'
  ghae: '*'
  ghec: '*'
type: tutorial
hidden: true
topics:
  - CI
  - Node
  - JavaScript
shortTitle: Crear & probar con Node.js
hasExperimentalAlternative: true
---

{% data reusables.actions.enterprise-beta %}
{% data reusables.actions.enterprise-github-hosted-runners %}

## Introducción

Esta guía te muestra cómo crear un flujo de trabajo de integración continua (CI) que construye y prueba código Node.js. Si tus pruebas de CI se superan, es posible que desees implementar tu código o publicar un paquete.

## Prerrequisitos

Te recomendamos que tengas una comprensión básica de Node.js, YAML, las opciones de configuración de flujo de trabajo y cómo crear un archivo de flujo de trabajo. Para obtener más información, consulta:

- "[Aprende sobre las {% data variables.product.prodname_actions %}](/actions/learn-github-actions)"
- "[Iniciar con Node.js](https://nodejs.org/en/docs/guides/getting-started-guide/)"

{% data reusables.actions.enterprise-setup-prereq %}

## Utilizar el flujo de trabajo inicial de Node.js

{% data variables.product.prodname_dotcom %} proporciona un flujo de trabajo inicial de Node.js que funcionará para la mayoría de los proyectos de Node.js. Esta guía incluye ejemplos de Yarn y de npm que puedes utilizar para personalizar el flujo de trabajo inicial. Para obtener más información, consulta el [flujo de trabajo inicial de Node.js](https://github.com/actions/starter-workflows/blob/main/ci/node.js.yml).

Para comenzar rápidamente, agrega el flujo de trabajo inicial al directorio de `.github/workflows` de tu repositorio. El flujo de trabajo que se muestra a continuación asume que la rama predeterminada de tu repositorio es `main`.

```yaml{:copy}
name: Node.js CI

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  build:

    runs-on: ubuntu-latest

    strategy:
      matrix:
        node-version: [10.x, 12.x, 14.x, 15.x]

    steps:
      - uses: {% data reusables.actions.action-checkout %}
      - name: Use Node.js {% raw %}${{ matrix.node-version }}{% endraw %}
        uses: {% data reusables.actions.action-setup-node %}
        with:
          node-version: {% raw %}${{ matrix.node-version }}{% endraw %}
      - run: npm ci
      - run: npm run build --if-present
      - run: npm test
```

{% data reusables.actions.example-github-runner %}

## Especificar la versión de Node.js

La forma más fácil de especificar una versión de Node.js es por medio de la acción `setup-node` proporcionada por {% data variables.product.prodname_dotcom %}. Para obtener más información, consulta [`setup-node`](https://github.com/actions/setup-node/).

La acción `setup-node` toma una versión de Node.js como una entrada y configura esa versión en el ejecutor. La acción `setup-node` encuentra una versión específica de Node.js de la caché de herramientas en cada ejecutor y añade los binarios necesarios a `PATH`, que continúan para el resto del trabajo. Usar la acción `setup-node` es la forma recomendada de usar Node.js con {% data variables.product.prodname_actions %} porque asegura un comportamiento consistente a través de diferentes ejecutores y diferentes versiones de Node.js. Si estás usando un ejecutor autoalojado, debes instalar Node.js y añadirlo a `PATH`.

El flujo de trabajo inicial incluye una estrategia de matriz que compila y prueba tu código con cuatro versiones de Node.js: 10.x, 12.x, 14.x y 15.x. La 'x' es un carácter comodín que coincide con el último lanzamiento menor y de parche disponible para una versión. Cada versión de Node.js especificada en la matriz `node-version` crea un trabajo que ejecuta los mismos pasos.

Cada trabajo puede acceder al valor definido en la matriz `node-version` por medio del contexto `matrix`. La acción `setup-node` utiliza el contexto como la entrada `node-version`. La acción `setup-node` configura cada trabajo con una versión diferente de Node.js antes de construir y probar código. Para obtener más información acerca de las estrategias y los contextos de la matriz, consulta las secciones "[Sintaxis de flujo de trabajo para las {% data variables.product.prodname_actions %}](/actions/automating-your-workflow-with-github-actions/workflow-syntax-for-github-actions#jobsjob_idstrategymatrix)" y "[Contextos](/actions/learn-github-actions/contexts)".

```yaml{:copy}
strategy:
  matrix:
    node-version: [10.x, 12.x, 14.x, 15.x]

steps:
- uses: {% data reusables.actions.action-checkout %}
- name: Use Node.js {% raw %}${{ matrix.node-version }}{% endraw %}
  uses: {% data reusables.actions.action-setup-node %}
  with:
    node-version: {% raw %}${{ matrix.node-version }}{% endraw %}
```

Como alternativa, puedes construir y probar con las versiones exactas de Node.js.

```yaml{:copy}
strategy:
  matrix:
    node-version: [8.16.2, 10.17.0]
```

O bien, puedes construir y probar mediante una versión única de Node.js.

```yaml{:copy}
name: Node.js CI

on: [push]

jobs:
  build:

    runs-on: ubuntu-latest

    steps:
      - uses: {% data reusables.actions.action-checkout %}
      - name: Use Node.js
        uses: {% data reusables.actions.action-setup-node %}
        with:
          node-version: '12.x'
      - run: npm ci
      - run: npm run build --if-present
      - run: npm test
```

Si no especificas una versión de Node.js, {% data variables.product.prodname_dotcom %} utiliza la versión de Node.js por defecto del entorno.
{% ifversion ghae %} {% data reusables.actions.self-hosted-runners-software %}
{% else %} Para obtener más información, consulta la sección "[Especificaciones para los ejecutores hospedados en {% data variables.product.prodname_dotcom %}](/actions/reference/specifications-for-github-hosted-runners/#supported-software)".
{% endif %}

## Instalar dependencias

Los ejecutores alojados en {% data variables.product.prodname_dotcom %} tienen instalados administradores de dependencias de npm y Yarn. Puedes usar npm y Yarn para instalar dependencias en tu flujo de trabajo antes de construir y probar tu código. Los ejecutores Windows y Linux alojados en {% data variables.product.prodname_dotcom %} también tienen instalado Grunt, Gulp y Bower.

{% if actions-caching %}You can also cache dependencies to speed up your workflow. For more information, see "[Caching dependencies to speed up workflows](/actions/using-workflows/caching-dependencies-to-speed-up-workflows)."{% endif %}

### Ejemplo con npm

Este ejemplo instala las dependencias definidas en el archivo *package.json*. Para obtener más información, consulta [`Instalar npm`](https://docs.npmjs.com/cli/install).

```yaml{:copy}
steps:
- uses: {% data reusables.actions.action-checkout %}
- name: Use Node.js
  uses: {% data reusables.actions.action-setup-node %}
  with:
    node-version: '12.x'
- name: Install dependencies
  run: npm install
```

Mediante `npm ci` se instalan las versiones en el archivo *package-lock.json* o *npm-shrinkwrap.json* y se evitan las actualizaciones al archivo de bloqueo. Usar `npm ci` generalmente es más rápido que ejecutar `npm install`. Para obtener más información, consulta [`npm ci`](https://docs.npmjs.com/cli/ci.html) e [Introducir `npm ci` para construcciones más rápidas y confiables](https://blog.npmjs.org/post/171556855892/introducing-npm-ci-for-faster-more-reliable)."

```yaml{:copy}
steps:
- uses: {% data reusables.actions.action-checkout %}
- name: Use Node.js
  uses: {% data reusables.actions.action-setup-node %}
  with:
    node-version: '12.x'
- name: Install dependencies
  run: npm ci
```

### Ejemplo con Yarn

Este ejemplo instala las dependencias definidas en el archivo *package.json*. Para obtener más información, consulta [`Instalar yarn`](https://yarnpkg.com/en/docs/cli/install).

```yaml{:copy}
steps:
- uses: {% data reusables.actions.action-checkout %}
- name: Use Node.js
  uses: {% data reusables.actions.action-setup-node %}
  with:
    node-version: '12.x'
- name: Install dependencies
  run: yarn
```

Alternatively, you can pass `--frozen-lockfile` to install the versions in the `yarn.lock` file and prevent updates to the `yarn.lock` file.

```yaml{:copy}
steps:
- uses: {% data reusables.actions.action-checkout %}
- name: Use Node.js
  uses: {% data reusables.actions.action-setup-node %}
  with:
    node-version: '12.x'
- name: Install dependencies
  run: yarn --frozen-lockfile
```

### Ejemplo de uso de un registro privado y la creación del archivo .npmrc

{% data reusables.actions.setup-node-intro %}

Para autenticar tu registro privado, necesitarás almacenar tu token de autenticación de npm como un secreto. Por ejemplo, crea un repositorio secreto que se llame `NPM_TOKEN`. Para más información, consulta "[Crear y usar secretos cifrados](/actions/automating-your-workflow-with-github-actions/creating-and-using-encrypted-secrets)."

En el siguiente ejemplo, el secreto `NPM_TOKEN` almacena el token de autenticación npm. La acción `setup-node` configura el archivo *.npmrc* para leer el token de autenticación npm desde la variable de entorno `NODE_AUTH_TOKEN`. Cuando utilices la acción `setup-node` para crear un archivo *.npmrc*, debes configurar la variable de ambiente `NODE_AUTH_TOKEN` con el secreto que contiene tu token de autenticación de npm.

Antes de instalar dependencias, utiliza la acción `setup-node` para crear el archivo *.npmrc*. La acción tiene dos parámetros de entrada. El parámetro `node-version` establece la versión de Node.js y el parámetro `registry-url` establece el registro predeterminado. Si tu registro de paquetes usa ámbitos, debes usar el parámetro `scope`. Para obtener más información, consulta [`npm-scope`](https://docs.npmjs.com/misc/scope).

```yaml{:copy}
steps:
- uses: {% data reusables.actions.action-checkout %}
- name: Use Node.js
  uses: {% data reusables.actions.action-setup-node %}
  with:
    always-auth: true
    node-version: '12.x'
    registry-url: https://registry.npmjs.org
    scope: '@octocat'
- name: Install dependencies
  run: npm ci
  env:
    NODE_AUTH_TOKEN: {% raw %}${{ secrets.NPM_TOKEN }}{% endraw %}
```

El ejemplo anterior crea un archivo *.npmrc* con el siguiente contenido:

```ini
//registry.npmjs.org/:_authToken=${NODE_AUTH_TOKEN}
@octocat:registry=https://registry.npmjs.org/
always-auth=true
```

{% if actions-caching %}

### Ejemplo de dependencias en caché

You can cache and restore the dependencies using the [`setup-node` action](https://github.com/actions/setup-node).

El siguiente ejemplo guarda las dependencias en caché para npm.

```yaml{:copy}
steps:
- uses: {% data reusables.actions.action-checkout %}
- uses: {% data reusables.actions.action-setup-node %}
  with:
    node-version: '14'
    cache: 'npm'
- run: npm install
- run: npm test
```

El siguiente ejemplo guarda las dependencias en caché para Yarn.

```yaml{:copy}
steps:
- uses: {% data reusables.actions.action-checkout %}
- uses: {% data reusables.actions.action-setup-node %}
  with:
    node-version: '14'
    cache: 'yarn'
- run: yarn
- run: yarn test
```

El siguiente ejemplo guarda las dependencias en caché para pnpm (v6.10+).

```yaml{:copy}
{% data reusables.actions.actions-not-certified-by-github-comment %}

# NOTE: pnpm caching support requires pnpm version >= 6.10.0

steps:
- uses: {% data reusables.actions.action-checkout %}
- uses: pnpm/action-setup@646cdf48217256a3d0b80361c5a50727664284f2
  with:
    version: 6.10.0
- uses: {% data reusables.actions.action-setup-node %}
  with:
    node-version: '14'
    cache: 'pnpm'
- run: pnpm install
- run: pnpm test
```

Si tienes un requisito personalizado o necesitas controles más exactos para almacenar en caché, puedes utilizar la [acción `cache`](https://github.com/marketplace/actions/cache). Para obtener más información, consulta la sección "[Almacenar las dependencias en caché para agilizar los flujos de trabajo](/actions/using-workflows/caching-dependencies-to-speed-up-workflows)".

{% endif %}

## Construir y probar tu código

Puedes usar los mismos comandos que usas de forma local para construir y probar tu código. Por ejemplo, si ejecutas `npm run build` para ejecutar pasos de construcción definidos en tu archivo *package.json* y `npm test` para ejecutar tu conjunto de pruebas, añadirías esos comandos en tu archivo de flujo de trabajo.

```yaml{:copy}
steps:
- uses: {% data reusables.actions.action-checkout %}
- name: Use Node.js
  uses: {% data reusables.actions.action-setup-node %}
  with:
    node-version: '12.x'
- run: npm install
- run: npm run build --if-present
- run: npm test
```

## Empaquetar datos de flujo de trabajo como artefactos

Puedes guardar los artefactos de tus pasos de construcción y prueba para verlos después de que se complete un trabajo. Por ejemplo, es posible que debas guardar los archivos de registro, los vaciados de memoria, los resultados de las pruebas o las capturas de pantalla. Para obtener más información, consulta "[Conservar datos de flujo de trabajo mediante artefactos](/actions/automating-your-workflow-with-github-actions/persisting-workflow-data-using-artifacts)."

## Publicar en registros de paquetes

Puedes configurar tu flujo de trabajo para que publique tu paquete Node.js en un registro de paquete después de que se aprueben tus pruebas de CI. Para obtener más información acerca de la publicación a npm y {% data variables.product.prodname_registry %}, consulta [Publicar paquetes Node.js](/actions/automating-your-workflow-with-github-actions/publishing-nodejs-packages)."
