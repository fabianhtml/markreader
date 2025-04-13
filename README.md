# MarkReader

> _Un lector de Markdown simple_

[Demo](https://markreader.reshape.so)

`markreader` nació buscando mejorar la experiencia de lectura en dispositivos móviles y de escritorio de documentos como PDFs, así como de tener un archivo personal que también pueda compartir con _LLMs_ para diferentes propósitos.

Los PDFs son incómodos de leer, pero hoy con IAs (como Gemini 1.5) convertirlos en Markdown toma segundos. También lo uso para guardar transcripciones de YouTube, consolidar documentación técnica, guardar reportes de _deep research_,  o cualquier texto interesante encontrado en línea (muchas veces mal formateado), etc.

## 🎨 Características

- Diseño simple y minimalista
- Soporte para temas claro y oscuro
- Resaltado de sintaxis para bloques de código
- Tipografía optimizada para lectura
- Opción de cambiar de fuente

## 🛠️ Tecnología Utilizada

El núcleo de `markreader` está construido con **Astro**, un framework moderno para generar sitios web rápidos y optimizados, ideal para manejar contenido estático como archivos Markdown. El proyecto utiliza **Node.js** y **pnpm** para la gestión de dependencias y scripts. Está configurado para un despliegue sencillo en **Cloudflare Pages**, utilizando la integración oficial de Astro para Cloudflare.

## 🚀 Estructura del Proyecto

Tiene una estrtucura muy simple.

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
| `pnpm install`            | Instala dependencias                             |
| `pnpm run dev`            | Inicia servidor de desarrollo en `localhost:4321`|
| `pnpm run build`          | Construye el sitio para producción en `./dist/`  |
| `pnpm run preview`        | Vista previa local de la build                   |

## 🌩️ Despliegue en Cloudflare Pages

La aplicación está desplegada en: 
- **[https://4ef65337.markreader.pages.dev](https://4ef65337.markreader.pages.dev)** (URL de Cloudflare Pages)
- **[https://markreader.reshape.so](https://markreader.reshape.so)** (Dominio personalizado)

### Configuración Previa

1. Asegúrate de tener una cuenta en Cloudflare
2. Actualiza el archivo `wrangler.toml` con tu `account_id` y `zone_id` (si tienes un dominio personalizado)

### Despliegue Manual

1. Construye la aplicación:

   ```bash
   pnpm run build
   ```

2. Despliega en Cloudflare Pages:

   ```bash
   wrangler pages deploy dist
   ```

### Despliegue Automático con GitHub Actions

Este proyecto incluye configuración para despliegue automático a Cloudflare Pages usando GitHub Actions. Cada vez que haces push a la rama `main`, tu aplicación se construye y despliega automáticamente.

#### Configuración de GitHub Actions

1. **Haz fork o clona este repositorio en tu cuenta de GitHub**

2. **Configura los secretos necesarios en tu repositorio de GitHub**:
   - Ve a tu repositorio en GitHub
   - Navega a Settings > Secrets and variables > Actions
   - Añade los siguientes secretos:
     - `CLOUDFLARE_API_TOKEN`: Tu token de API de Cloudflare con permisos para Pages
     - `CLOUDFLARE_ACCOUNT_ID`: El ID de tu cuenta de Cloudflare

3. **Asegúrate de que el flujo de trabajo utilice pnpm**:
   - Revisa el archivo `.github/workflows/deploy.yml` para asegurarte de que utiliza pnpm en lugar de npm
   - Actualiza los comandos según sea necesario:
     ```yaml
     - name: Instalar dependencias
       run: pnpm install
     
     - name: Construir aplicación
       run: pnpm run build
     ```

4. **Cómo obtener los valores para los secretos**:

   Para `CLOUDFLARE_API_TOKEN`:
   - Inicia sesión en el [Dashboard de Cloudflare](https://dash.cloudflare.com)
   - Ve a "Mi perfil" > "API Tokens"
   - Crea un token personalizado con permisos:
     - Account > Cloudflare Pages > Edit

   Para `CLOUDFLARE_ACCOUNT_ID`:
   - El ID de tu cuenta se encuentra en la URL del dashboard de Cloudflare
   - Formato: `https://dash.cloudflare.com/abcdef1234567890abcdef1234567890`
   - El string alfanumérico largo es tu Account ID

5. **Verifica el despliegue**:
   - Haz un pequeño cambio en tu repositorio
   - Haz commit y push a la rama main
   - Ve a la pestaña "Actions" en tu repositorio para ver el progreso
   - Una vez completado, tu aplicación estará disponible en Cloudflare Pages

El archivo de configuración de GitHub Actions se encuentra en `.github/workflows/deploy.yml`.

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

## Añadir Nuevos archivos Markdown

Para añadir nuevos archivos Markdown:

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