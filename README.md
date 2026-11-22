# DevCarmelo

Blog personal de [Carmelo](https://devcarmelo.github.io/), creado con Astro y basado en el tema AstroPaper. El sitio publica artículos y tutoriales sobre Linux, Debian y otros temas técnicos.

**Sitio:** https://devcarmelo.github.io/blog/

## Características

- Contenido escrito en Markdown o MDX.
- Interfaz en español.
- Modo claro y oscuro.
- Buscador estático con Pagefind.
- Tabla de contenidos en los artículos.
- RSS, sitemap y páginas de archivos.
- Etiquetas y paginación.
- Imagen Open Graph dinámica.
- Diseño adaptable y optimizado para el rendimiento.

## Tecnologías

- [Astro](https://astro.build/)
- [TypeScript](https://www.typescriptlang.org/)
- [Tailwind CSS](https://tailwindcss.com/)
- [Pagefind](https://pagefind.app/)
- [pnpm](https://pnpm.io/)
- [GitHub Actions](https://github.com/features/actions)
- [GitHub Pages](https://pages.github.com/)

## Requisitos

- Node.js 24 o una versión compatible
- pnpm 11

## Desarrollo local

Instala las dependencias:

```bash
pnpm install
```

Inicia el servidor de desarrollo:

```bash
pnpm dev
```

El sitio estará disponible en `http://localhost:4321/blog/`.

## Comandos disponibles

Ejecuta estos comandos desde la raíz del proyecto:

| Comando             | Acción                                                                                            |
| :------------------ | :------------------------------------------------------------------------------------------------ |
| `pnpm install`      | Instala las dependencias.                                                                         |
| `pnpm dev`          | Inicia el servidor de desarrollo.                                                                 |
| `pnpm build`        | Comprueba tipos, genera el sitio, crea el índice Pagefind y copia el índice a `public/pagefind/`. |
| `pnpm preview`      | Sirve localmente la compilación generada.                                                         |
| `pnpm sync`         | Genera los tipos de Astro.                                                                        |
| `pnpm lint`         | Ejecuta ESLint sobre el proyecto.                                                                 |
| `pnpm format:check` | Comprueba el formato con Prettier.                                                                |
| `pnpm format`       | Aplica el formato con Prettier.                                                                   |

## Crear un artículo

Los artículos se encuentran en `src/content/posts/`. Cada archivo debe incluir información básica en el encabezado YAML:

```md
---
title: Título del artículo
author: Carmelo
pubDatetime: 2026-11-22T14:36:53Z
featured: false
draft: false
tags:
  - Linux
  - tutorial
description: Resumen breve del artículo.
---

Contenido del artículo...
```

Durante el desarrollo puedes asignar `draft: true` para excluir temporalmente una entrada de la versión publicada.

## Estructura del proyecto

```text
/
├── public/                 # Recursos públicos e índice generado por Pagefind
├── src/
│   ├── assets/             # Imágenes e iconos
│   ├── components/         # Componentes reutilizables de Astro
│   ├── content/
│   │   ├── pages/          # Páginas Markdown
│   │   └── posts/          # Artículos del blog
│   ├── i18n/               # Traducciones y formatos
│   ├── layouts/            # Plantillas de página y artículo
│   ├── pages/              # Rutas del sitio
│   ├── styles/             # Estilos globales
│   └── utils/              # Utilidades
├── astro-paper.config.ts   # Configuración del sitio
├── astro.config.ts         # Configuración de Astro
├── compose.yaml            # Entorno de desarrollo con Docker Compose
└── Dockerfile              # Compilación y servidor nginx
```

## Docker

El entorno de desarrollo también puede ejecutarse con Docker Compose:

```bash
docker compose up
```

El servidor quedará publicado en `http://localhost:4321`.

Para generar una imagen de producción con nginx:

```bash
docker build -t devcarmelo-blog .
docker run --rm -p 8080:80 devcarmelo-blog
```

La imagen servirá el resultado de la compilación en `http://localhost:8080`.

## Despliegue

El repositorio utiliza GitHub Actions para construir y publicar el sitio en GitHub Pages. El flujo de despliegue se ejecuta automáticamente al subir cambios a la rama `master`, y también puede iniciarse manualmente desde la pestaña **Actions**.
