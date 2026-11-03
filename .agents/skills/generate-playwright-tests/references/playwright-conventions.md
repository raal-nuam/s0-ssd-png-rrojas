# Convenciones Playwright de generate-playwright-tests

Aplica estas convenciones cuando el proyecto no tenga otras. Si ya tiene
estructura, lenguaje, Page Objects, fixtures o lint propios, adáptalas a lo
existente sin reorganizar el proyecto. Los ejemplos usan TypeScript; en un
proyecto JavaScript, mantén la misma estructura.

## Estructura

```text
<proyecto>/
├── playwright.config.ts
├── .env.example            # claves documentadas, sin secretos
├── config/
│   └── env.ts              # única lectura de process.env
├── pages/                  # Page Objects compartidos entre grupos
├── fixtures/               # fixtures compartidas
└── tests/
    ├── auth.setup.ts       # autenticación, si la aplicación la requiere
    ├── openspec/
    │   └── <capability>/
    │       └── <requisito>.spec.ts
    └── flows/
        └── <grupo>/
            └── <flujo>.spec.ts
```

Los nombres de archivos y carpetas usan minúsculas, guiones y ASCII. Una
`<capability>` conserva el nombre de su carpeta en `openspec/specs/`.

## Projects (grupos)

Cada grupo es una entrada de `projects` con su `testDir`. `OpenSpec` es fijo; los
grupos QA usan el nombre normalizado elegido por el usuario. `setup` es auxiliar:
sólo se crea si hay autenticación y no se muestra como grupo QA.

```ts
import { defineConfig, devices } from '@playwright/test';
import { env } from './config/env';

const authenticated = {
  ...devices['Desktop Chrome'],
  storageState: '.auth/user.json',
};

export default defineConfig({
  fullyParallel: true,
  forbidOnly: !!process.env.CI,
  retries: process.env.CI ? 2 : 0,
  reporter: [['html', { open: 'never' }], ['list']],
  timeout: env.TEST_TIMEOUT,
  use: {
    baseURL: env.BASE_URL,
    trace: 'on-first-retry',
    screenshot: 'only-on-failure',
  },
  projects: [
    { name: 'setup', testMatch: /auth\.setup\.ts/ },
    {
      name: 'OpenSpec',
      testDir: './tests/openspec',
      use: authenticated,
      dependencies: ['setup'],
    },
    {
      name: 'flujo-1',
      testDir: './tests/flows/flujo-1',
      use: authenticated,
      dependencies: ['setup'],
    },
  ],
});
```

Al añadir un grupo, inserta sólo su entrada y conserva el resto de la
configuración. Si el proyecto ya define dispositivos o navegadores, aplica los
mismos al nuevo project. Añade `.auth/` a `.gitignore` cuando se use `storageState`.

## Variables de entorno

Todo lo que cambia entre ambientes se lee de variables: URL base, usuarios,
contraseñas, tokens, URLs de APIs o servicios, tenants, cuentas de prueba e
instrumentos de datos, y timeouts. Ningún test lee `process.env` directamente:
lo hace `config/env.ts`, que falla con un mensaje claro si falta una variable
obligatoria. No pongas valores por defecto para secretos ni URLs de ambientes reales.

```ts
// config/env.ts
function required(name: string): string {
  const value = process.env[name];
  if (!value) throw new Error(`Falta la variable de entorno ${name}; revisa .env.example`);
  return value;
}

function optionalNumber(name: string, fallback: number): number {
  const value = process.env[name];
  return value ? Number(value) : fallback;
}

export const env = {
  BASE_URL: required('BASE_URL'),
  E2E_USER: required('E2E_USER'),
  E2E_PASSWORD: required('E2E_PASSWORD'),
  TEST_TIMEOUT: optionalNumber('TEST_TIMEOUT', 30_000),
};
```

```dotenv
# .env.example — copiar a .env y completar por ambiente. No subir .env.
BASE_URL=
E2E_USER=
E2E_PASSWORD=
TEST_TIMEOUT=30000
```

Para cargar `.env`, usa el mecanismo que el proyecto ya tenga. Si no existe, la
opción habitual es `import 'dotenv/config'` al inicio de `playwright.config.ts`;
añadir `dotenv` como dependencia requiere aprobación del usuario. Sin ella, las
variables se exportan en la terminal o en el pipeline. Comprueba que `.gitignore`
incluya `.env` y no `.env.example`.

## Autenticación

Si la aplicación requiere sesión, autentícate una vez en `tests/auth.setup.ts` y
reutiliza `storageState`. Los tests que prueban el propio login no dependen de
`setup` y usan `test.use({ storageState: { cookies: [], origins: [] } })`.

```ts
// tests/auth.setup.ts
import { test as setup, expect } from '@playwright/test';
import { env } from '../config/env';

setup('autenticar usuario de pruebas', async ({ page }) => {
  await page.goto('/');
  await page.getByLabel('Usuario').fill(env.E2E_USER);
  await page.getByLabel('Contraseña').fill(env.E2E_PASSWORD);
  await page.getByRole('button', { name: 'Ingresar' }).click();
  await expect(page.getByRole('main')).toBeVisible();
  await page.context().storageState({ path: '.auth/user.json' });
});
```

Los labels y textos del ejemplo son ilustrativos: tómalos del código o de la
interfaz real, o márcalos como provisionales.

## Tests del grupo OpenSpec

Un archivo por requisito, un `describe` con el nombre del requisito y un `test`
por escenario con su nombre exacto. Cada test lleva la anotación `spec` con la
ruta y, si viene de un cambio no archivado, también `proposal`. WHEN y THEN se
convierten en `test.step`.

```ts
// tests/openspec/order-management/creacion-de-un-pedido.spec.ts
import { test, expect } from '@playwright/test';
import { OrdersPage } from '../../../pages/orders-page';

const SPEC = 'openspec/specs/order-management/spec.md';

test.describe('Requirement: Creación de un pedido', () => {
  test('Scenario: Creación con referencia asignada', {
    annotation: [{ type: 'spec', description: `${SPEC} › Creación de un pedido › Creación con referencia asignada` }],
  }, async ({ page }) => {
    const orders = new OrdersPage(page);
    await page.goto('/');

    await test.step('WHEN el usuario crea un pedido con la referencia «PED-1»', async () => {
      await orders.create('PED-1');
    });

    await test.step('THEN el pedido «PED-1» aparece en la lista sin productos', async () => {
      await expect(orders.row('PED-1')).toBeVisible();
      await expect(orders.items('PED-1')).toHaveCount(0);
    });
  });
});
```

La capability, los requisitos y las pantallas de los ejemplos son ficticios.

## Tests de grupos QA

Un archivo por flujo dentro de `tests/flows/<grupo>/`. La anotación `hu` se
repite por cada HU que atraviesa el flujo; si el caso viene de una matriz de
`prepare-tests`, añade `case` con su ID. Cada paso del flujo es un `test.step`
que indica su HU.

```ts
// tests/flows/flujo-1/crear-pedido-y-emitir-factura.spec.ts
import { test, expect } from '@playwright/test';

test('Flujo: crear un pedido y emitir su factura', {
  annotation: [
    { type: 'hu', description: 'HU-101 › openspec/specs/order-management/spec.md' },
    { type: 'hu', description: 'HU-102 › openspec/specs/invoicing/spec.md' },
    { type: 'case', description: 'QA-01' },
  ],
}, async ({ page }) => {
  await test.step('[HU-101] crear el pedido «PED-1»', async () => { /* ... */ });
  await test.step('[HU-102] emitir la factura del pedido', async () => { /* ... */ });
});
```

Sustituye `HU-101` y `HU-102` por las HU reales. Si una HU no está identificada,
no la inventes: deja el paso trazado al caso y regístralo como duda.

## Buenas prácticas de escritura

- **Locators:** prioriza `getByRole`, `getByLabel`, `getByText` y `getByTestId`.
  Evita XPath y selectores CSS ligados a estilos o a la estructura del DOM.
- **Aserciones:** usa aserciones web-first (`await expect(locator).toBeVisible()`),
  que esperan solas. No uses `page.waitForTimeout` ni esperas fijas.
- **Independencia:** cada test prepara su estado o lo recibe por fixture; no
  depende del orden ni de datos que haya dejado otro test. Los datos generados
  incluyen un sufijo único para no colisionar en ejecución paralela.
- **Page Objects:** encapsula en `pages/` la interacción repetida con una
  pantalla; los grupos los comparten. Las aserciones del escenario quedan en el test.
- **Pendientes:** un escenario que todavía no puede automatizarse se escribe con
  `test.fixme('motivo')`; no lo omitas ni lo marques como aprobado.
- **Datos sensibles:** no escribas credenciales, tokens ni datos reales de clientes
  en código, comentarios, snapshots ni reportes.
- **Artefactos:** `test-results/`, `playwright-report/`, `.auth/` y `.env` deben
  estar en `.gitignore`.
