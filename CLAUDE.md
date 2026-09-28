# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Qué es este proyecto

Sitio web de **ROVE Project / Henry Contreras**: plataforma de comunidad e impacto social (voluntariado, ecosistema de proyectos, membresía "Tarjeta Verde", tienda con propósito, blog, panel de impacto). Contenido y UI en **español**.

Es un **sitio estático para GitHub Pages**: sin backend, sin build, sin dependencias npm. Todo lo "dinámico" (usuarios, carrito, puntos) es una simulación en `localStorage`.

El brief del cliente está en `Estrategia web ROVE Project.pdf` (no se publica; ver `.gitignore`). Define las 9 secciones, la Tarjeta Verde como "corazón de la plataforma", y requisitos futuros (panel admin, pasarela de pagos, QR reales, Google Analytics, Meta Pixel, SEO). Consúltalo antes de añadir funcionalidades. Filosofía de diseño: cercanía, comunidad, sostenibilidad; **no** debe verse político ni institucional.

## Comandos

```bash
# Servir localmente (las rutas de GH Pages son relativas a rove/)
python -m http.server 8000 --directory rove
# → http://localhost:8000/#tarjeta-verde  (cualquier página vía hash)
```

No hay build, lint ni tests configurados. El despliegue lo hace `.github/workflows/pages.yml` al hacer push a `main`: publica **solo el contenido de `rove/`** (en GitHub: Settings → Pages → Source = "GitHub Actions").

## Arquitectura

Todo vive en **un único archivo `rove/index.html`** (~2250 líneas) con tres bloques:

1. **`<style>`** — design tokens en `:root` (paleta `--green-*`, `--sand-*`, `--copper`; fuentes Playfair Display + Inter). Las secciones CSS están delimitadas por banners `═══ NOMBRE ═══`. Breakpoints responsive al final: 1024 / 768 / 480px.
2. **HTML** — cada "página" es un `<section class="page" id="page-<nombre>">`. Solo la que tiene `.active` se muestra.
3. **`<script>`** al final — JS vanilla, sin módulos. Las funciones se exponen en `window` al final del script porque el HTML usa `onclick="..."` inline. **Si añades una función llamada desde el HTML, agrégala a esa lista.**

### Router por hash
`navigateTo(name)` activa `#page-<name>`, marca los enlaces con `data-page="<name>"` y actualiza `location.hash`. `handleHash()` corre en `DOMContentLoaded` y `hashchange`; hash desconocido → `home`. Para añadir una página: crear `<section class="page" id="page-X">` + enlaces `data-page="X"` en **ambos** menús (`#navLinks` escritorio y `#mobileMenu` móvil, que están duplicados a mano) + footer si aplica.

Páginas: `home, about, ecosystem, community, tarjeta-verde, store, blog, impact, contact`.

### Datos
- Productos (`products`) y posts (`blogPosts`) son arrays hardcodeados en el script y se renderizan con template strings → `innerHTML` (`renderStore`, `initBlog`). Las cifras de impacto están en atributos `data-count` / `data-width` del HTML.
- Claves de `localStorage`: `rove_cart`, `rove_users` (registro, **guarda la contraseña en texto plano** — es solo demo), `rove_tv_user` (sesión de Tarjeta Verde).
- Tarjeta Verde: `LEVELS` (Semilla/Raíz/Bosque/Montaña por puntos) y `BADGES`; `simulateActivity()` suma 50 pts y otorga insignias. El QR es solo un icono. `loginTV()` no valida contraseña.
- Formularios de voluntariado/contacto solo muestran un toast; no envían nada. El contacto real es WhatsApp (`wa.me/59172516464`, repetido en 3 sitios).

### Restricciones de GitHub Pages
- Solo archivos estáticos: cualquier backend (auth real, pagos, envío de formularios) requiere un servicio externo (Formspree, Supabase, Stripe Checkout, etc.).
- Usar rutas **relativas** (el sitio se sirve bajo `https://<usuario>.github.io/<repo>/`), nunca `/ruta` absoluta.
- Dependencias externas solo por CDN: Google Fonts y Font Awesome 6.5.1 (cdnjs).

## Problemas conocidos

- **Contadores y barras de impacto no se animan**: `setupScrollObserver()` crea un `IntersectionObserver` pero nunca llama a `observer.observe(...)`, así que los `data-count` se quedan en "0" y los `data-width` en 0%.
- `switchCommunityTab()` depende del global implícito `event`.
- Los datos interpolados en `innerHTML` no se escapan (hoy son datos propios, pero importa si llegan de usuarios o de una API).
- No hay imágenes reales: todo son degradados CSS + iconos Font Awesome como placeholders.
