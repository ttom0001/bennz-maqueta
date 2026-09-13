# BENNZ — Storefront Prototype

Diseño y prototipo de tienda online para **BENNZ**, marca de streetwear de Tolhuin, Tierra del Fuego.

**Live demo:** [ttom0001.github.io/bennz-maqueta](https://ttom0001.github.io/bennz-maqueta/)

---

## Sobre el proyecto

Prototipo single-page interactivo que demuestra:

- **Hero cinemático** con 5 paneles verticales (masthead · drop reveal · producto destacado · manifiesto · CTA), snap-scroll estilo TikTok en mobile
- **Video-driven scroll** con reels reales de la marca (5 videos, versión desktop 720p + versión mobile 540p auto-servida por breakpoint)
- **Drop grid** con 12 productos reales del catálogo
- **Cubo 3D interactivo** — drag para rotar con inercia y spring physics, tap para abrir la ficha
- **PDP full-screen** con selector de talles, wishlist, gallery
- **Cart drawer** con calculador de envío por CP (7 zonas de Argentina), 7 métodos de pago (MP, GoCuotas, link, efectivo en local), retiro en Te Al 436
- **Chapter divider** editorial con marquee legible
- **Búsqueda liquid-glass** con backdrop-filter y live search
- **Micro-interacciones**: ripples, tilt 3D, skeleton loaders, FLIP transitions
- **Motion library propia** — spring physics, cero deps externas
- **Accesibilidad**: `prefers-reduced-motion` honrado, focus states, tab-index en el cubo, contrast AA

## Stack

- **HTML/CSS/JS vanilla** — cero build, cero framework
- **GSAP + ScrollTrigger** (CDN) — scroll effects
- **Lenis** (CDN) — smooth scroll
- **Google Fonts**: Fraunces, Manrope, JetBrains Mono, UnifrakturCook

Todo autocontenido en un solo archivo `index.html` + carpetas de assets. Deploy = drag & drop a cualquier host estático.

## Estructura

```
bennz-maqueta/
├── index.html          — sitio completo (single-file)
├── img/                — 13 fotos de producto
└── videos/             — 5 reels (desktop + mobile + poster JPG cada uno)
```

## Cómo abrir local

Doble click en `index.html`.

## Cómo publicar en tu propio host

Subí toda la carpeta a Vercel / Netlify / Hostinger / cualquier hosting estático. Sin build step.

---

## Identidad

- **Marca**: BENNZ · @bennz_gta
- **Local**: Te Al 436, Tolhuin, Tierra del Fuego
- **Coordenadas**: 54°30′S · 67°12′O
- **Tagline**: *Del sur al mundo — cortada acá, usada donde caiga.*
