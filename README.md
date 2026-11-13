# Firmalo Developers

Sitio de [Mintlify](https://www.mintlify.com/docs) con las guías de la API de integración de Firmalo y su referencia técnica. Todo lo que se publica está en `docs/`; este README queda fuera del sitio.

```text
docs/
  docs.json                 Configuración y navegación
  openapi.yaml              Contrato de la API; se genera en signature-server, no se edita a mano
  introduction.mdx          Introducción
  authentication.mdx        X-API-Key y errores de autenticación
  guides/
    getting-started.mdx     Primeros pasos: crear y consultar un documento
    errors.mdx              Formato de error, estados y códigos
    idempotency.mdx         Idempotency-Key, reintentos y vigencia
  logo/ y favicon.png       Marca, copiada de signature-site/public/brand
```

La navegación (`navigation.groups` de `docs.json`) tiene cinco secciones: **Introducción**, **Autenticación**, **Primeros pasos**, **Manejo de errores** y **Referencia de API**.

## Agregar una página

1. Crea el `.mdx` dentro de `docs/` (las guías van en `docs/guides/`) con `title` y `description` en el frontmatter.
2. Agrega su ruta, relativa a `docs/` y sin extensión, al grupo que corresponda en `docs.json`. Por ejemplo, `guides/webhooks`.
3. Revísala en local y ejecuta `npx mint broken-links`.

En `.mdx`, escribe entre comillas invertidas todo lo que lleve `<` o `{`, como `fma_live_<publicId>_<secret>`: fuera de código se interpreta como JSX.

## Referencia de API

El grupo **Referencia de API** se genera a partir de `docs/openapi.yaml` (`"openapi": "openapi.yaml"` en `docs.json`). Mintlify crea una página por operación y las agrupa por el `tag` de cada una; no hay `.mdx` que mantener a mano.

`openapi.yaml` sale de los decoradores de Swagger de `signature-server` (`src/document/integration/docs/`) y sólo contiene las rutas `/api/v1/integration/...`. Para agregar o cambiar un endpoint:

1. Cambia el controlador, el DTO o sus decoradores en `signature-server`.
2. Genera el contrato desde la raíz de `signature-server`. No necesita base de datos ni servicios:

   ```bash
   INTEGRATION_API_PUBLIC_URL=https://<host-de-la-api> npm run docs:openapi
   ```

3. Copia el resultado (`signature-server/docs/openapi.yaml`) a `docs/openapi.yaml` de este repositorio y súbelo en un PR.

Notas:

- Sin `INTEGRATION_API_PUBLIC_URL`, `servers` queda en `https://api.firmalo.example`, que no existe. Mientras no haya host público, el playground está desactivado (`api.playground.display: "none"`) y la referencia es de consulta. Con el host definido, cambia `display` a `"interactive"`.
- `npx mint validate` valida también el OpenAPI; ejecútalo después de copiarlo.

## Vista local

Desde `docs/`:

```bash
npx mint dev --port 3333    # http://localhost:3333; el puerto 3000 lo usa el backend
npx mint validate           # valida docs.json y openapi.yaml
npx mint broken-links
```

## Despliegue en Mintlify

Se configura una sola vez desde el [panel de Mintlify](https://app.mintlify.com) y requiere permisos de administración en la organización `data-lo` de GitHub.

1. Crea el proyecto en Mintlify (`npx mint signup` o desde el panel).
2. Instala la GitHub App de Mintlify con acceso a `data-lo/signature-documentation`.
3. En **Git Settings**, elige el repositorio y la rama de despliegue.
4. En esa misma página, activa la opción de que `docs.json` está en un subdirectorio e indica `/docs`, sin barra final.
5. Guarda y comprueba en el panel que el despliegue termine bien y que **Referencia de API** liste los endpoints.

### Ambiente de pruebas

Mintlify asigna al proyecto un dominio temporal, `https://<subdominio>.mintlify.site`; `npx mint status` muestra la URL exacta. Ése es el ambiente de pruebas del equipo hasta configurar un dominio propio en **Custom domain**.

### Previews por pull request

- Requieren plan **Pro o Enterprise**.
- Mintlify genera la preview de cada PR que apunta a la **rama de despliegue**. Como los PR van a `development`, usa esa rama mientras el sitio sea de pruebas; con `main`, esos PR no tendrían preview automática.
- Con la GitHub App instalada, el bot de Mintlify comenta en el PR el enlace de la preview. Si falla, los registros están en la página **Previews** del panel.
- Las URLs de preview son públicas por defecto. Restringirlas a la organización requiere Enterprise, así que no publiques ahí contenido confidencial.
