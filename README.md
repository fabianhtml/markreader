# MarkReader

Un lector minimalista de documentos Markdown con soporte para temas claro y oscuro.

## 🚀 Estructura del Proyecto

```text
/
├── public/
│   └── favicon.svg
├── src/
│   ├── content/
│   │   └── documentos en markdown (.md)
│   ├── layouts/
│   │   ├── Layout.astro
│   │   └── MarkdownLayout.astro
│   ├── pages/
│   │   └── index.astro
│   └── styles/
│       └── global.css
├── .cloudflare/
│   └── workers-site/
│       └── index.js
├── wrangler.toml
└── package.json
```

## 🧞 Comandos

Todos los comandos se ejecutan desde la raíz del proyecto, desde una terminal:

| Comando                   | Acción                                           |
| :------------------------ | :----------------------------------------------- |
| `npm install`             | Instala dependencias                             |
| `npm run dev`             | Inicia servidor de desarrollo en `localhost:4321`|
| `npm run build`           | Construye el sitio para producción en `./dist/`  |
| `npm run preview`         | Vista previa local de la build                   |

## 🌩️ Despliegue en Cloudflare Pages

La aplicación está desplegada en: 
- **[https://4ef65337.markreader.pages.dev](https://4ef65337.markreader.pages.dev)** (URL de Cloudflare Pages)
- **[https://markreader.reshape.so](https://markreader.reshape.so)** (Dominio personalizado)

### Configuración Previa

1. Asegúrate de tener una cuenta en Cloudflare
2. Actualiza el archivo `wrangler.toml` con tu `account_id` y `zone_id` (si tienes un dominio personalizado)

### Pasos para el Despliegue

1. Construye la aplicación:
   ```bash
   npm run build
   ```

2. Despliega en Cloudflare Pages:
   ```bash
   wrangler pages deploy dist
   ```

### Configuración de Dominio Personalizado

Para configurar un dominio personalizado (como `markreader.reshape.so`):

1. **En el Panel de Cloudflare**:
   - Inicia sesión en [dash.cloudflare.com](https://dash.cloudflare.com)
   - Selecciona tu dominio principal (ej: `reshape.so`)
   - Ve a la sección "DNS"

2. **Añade un Registro CNAME**:
   - Haz clic en "Add record" (Agregar registro)
   - Tipo: CNAME
   - Nombre: `markreader` (para crear `markreader.reshape.so`)
   - Destino: La URL de Cloudflare Pages (ej: `4ef65337.markreader.pages.dev`)
   - Proxy: Activado (naranja)
   - Guarda el registro

3. **Configura el Dominio en Pages**:
   - Ve a la sección "Pages" en Cloudflare
   - Selecciona tu proyecto "markreader"
   - Haz clic en "Custom domains" (Dominios personalizados)
   - Añade `markreader.reshape.so` como dominio personalizado
   - Sigue las instrucciones para completar la configuración

4. **Actualiza wrangler.toml** (opcional):
   - Añade la configuración de dominio personalizado en tu archivo `wrangler.toml`:
   ```toml
   [env.production]
   routes = [
     { pattern = "markreader.reshape.so", custom_domain = true }
   ]
   ```

5. **Espera la Propagación DNS**:
   - Los cambios pueden tardar hasta 24 horas, aunque con Cloudflare suele ser más rápido
   - Verifica que el certificado SSL esté activo

## 🔧 Mantenimiento

### Actualización de la Aplicación

Para actualizar la aplicación después de realizar cambios:

1. Realiza tus modificaciones en el código
2. Construye la aplicación:
   ```bash
   npm run build
   ```
3. Despliega los cambios:
   ```bash
   wrangler pages deploy dist
   ```

### Añadir Nuevos Documentos Markdown

Para añadir nuevos documentos:

1. Crea un archivo Markdown en la carpeta `src/content/`
2. Asegúrate de incluir el frontmatter con título, autor y fecha (opcional):
   ```markdown
   ---
   title: Título del post
   author: Autor del post
   date: 2025-03-21
   description: Una breve descripción del post
   ---

   Contenido del post en Markdown...
   ```
3. Construye y despliega la aplicación

### Solución de Problemas Comunes

- **Error de compilación**: Verifica la sintaxis de tus archivos Markdown
- **Problemas con el tema**: Asegúrate de que los estilos en `global.css` sean correctos
- **Errores en el despliegue**: Verifica que `wrangler.toml` tenga la configuración correcta

## 🎨 Características

- Diseño minimalista enfocado en la legibilidad
- Soporte para temas claro y oscuro
- Resaltado de sintaxis para bloques de código
- Tipografía optimizada para lectura
- Almacenamiento de preferencia de tema
