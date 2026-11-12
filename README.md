# Firmalo Developers

Sitio público de [Mintlify](https://www.mintlify.com/docs) con las guías de la API de integración de Firmalo y su referencia técnica. Este archivo no se publica (ver `.mintignore`).

```text
docs.json              Configuración, tabs y navegación
index.mdx              Introducción
quickstart.mdx         Inicio rápido: crear y consultar un documento
authentication.mdx     X-API-Key y errores de autenticación
idempotency.mdx        Idempotency-Key, reintentos y vigencia
errors.mdx             Formato de error, estados y códigos
webhooks.mdx           Contrato público de webhooks salientes; la fuente es
                       signature-server/src/outbound-webhooks/README.md
logo/ y favicon.png    Marca, copiada de signature-site/public/brand
```

## Referencia API

La pestaña **Referencia API** no tiene archivos aquí: Mintlify lee en cada despliegue el contrato que publica el backend en `https://<api-producción>/api/v1/integration/docs-json`. Ese contrato sólo contiene las rutas `/api/v1/integration/...`, así que un endpoint nuevo aparece al redesplegar este sitio, sin cambios en este repositorio.

- La URL está en `navigation.tabs[1].openapi` de `docs.json`. Mientras no se defina el host de producción apunta a `https://api.firmalo.example`, que no existe: el build falla hasta reemplazarla.
- Mintlify sólo acepta URLs `https://` en producción.
- El playground está desactivado (`api.playground.display: "none"`): la referencia es de consulta.
- El contrato no declara `servers`, así que la referencia muestra las rutas sin host.

## Agregar una página

1. Crea el `.mdx` con `title` y `description` en el frontmatter.
2. Agrega su ruta, sin extensión, al grupo que corresponda en `docs.json`.
3. Revísala en local.

En `.mdx`, escribe entre comillas invertidas todo lo que lleve `<` o `{`, como `fma_live_<publicId>_<secret>`: fuera de código se interpreta como JSX.

## Vista local

```bash
npx mint dev --port 3333    # http://localhost:3333
npx mint validate           # valida docs.json y el OpenAPI remoto
npx mint broken-links
```

Para probar contra un backend local, cambia temporalmente la URL de `openapi` a `http://localhost:3000/api/v1/integration/docs-json` y agrega `--local-schema` a los comandos. No subas ese cambio.

## Publicación

Se hace una vez desde el [panel de Mintlify](https://app.mintlify.com), con permisos de administración en la organización de GitHub.

1. Crea el proyecto en Mintlify y conecta `data-lo/signature-documentation` con su GitHub App; `docs.json` está en la raíz.
2. Elige la rama de despliegue.
3. En **Custom domain**, configura el dominio público y crea el `CNAME` que indica el panel.
4. Comprueba en el panel que el despliegue termine bien y que la pestaña **Referencia API** liste los cinco endpoints.
