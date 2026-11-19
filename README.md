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

La pestaña **Referencia API** no tiene archivos aquí: Mintlify lee en cada despliegue el contrato que publica el backend en `/api/v1/integration/docs-json`. Ese contrato sólo contiene las rutas `/api/v1/integration/...`, así que un endpoint nuevo aparece al redesplegar este sitio, sin cambios en este repositorio.

- La URL está en `navigation.tabs[1].openapi` de `docs.json` y hoy apunta a staging: `https://signature.api.staging.data-lo.online/api/v1/integration/docs-json`.
- Mintlify sólo acepta URLs `https://` en producción.
- El playground es interactivo (`api.playground.display: "interactive"`) **sólo porque la URL apunta a staging**. Las peticiones van al host que declara `servers` en el contrato, que el backend toma de `INTEGRATION_API_PUBLIC_URL` (en staging, `https://signature.api.staging.data-lo.online`); sin esa variable no hay `servers` y el playground no sabría a dónde enviar.
- Las peticiones pasan por el proxy de Mintlify (`api.playground.proxy: true`), así que `signature-server` no necesita permitir el dominio de la documentación en CORS. Si algún día se desactiva el proxy, hay que agregar a CORS exclusivamente ese dominio.
- La autenticación es el esquema `X-API-Key` del contrato: se pega una vez y se reutiliza en todos los endpoints. Usa sólo llaves de staging y no agregues llaves reales a este repositorio, a ejemplos ni a `docs.json`.
- **Producción no tiene playground.** Antes de apuntar `openapi` al contrato de producción, vuelve a poner `api.playground.display: "none"` y no definas `INTEGRATION_API_PUBLIC_URL` en el backend de producción, hasta que se apruebe la política de manejo de API keys y proxy. En producción las llaves se usan sólo desde el backend del cliente.

## Agregar una página

1. Crea el `.mdx` con `title` y `description` en el frontmatter.
2. Agrega su ruta, sin extensión, a `navigation.tabs[0].pages` de `docs.json`. Ese arreglo no usa encabezados de grupo: cada entrada es una página suelta (`"authentication"`, `"webhooks"`) o una sección desplegable (`{ "group": "Conceptos", "pages": [...] }`). Con `root`, la sección abre esa página al hacer clic, como **Introducción** con `index`.
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
