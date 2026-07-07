# SPEC 03 — Home como página principal y Nav superior actualizado

> **Status:** Aprobado
> **Depends on:** [01-pantallas-visuales-mvp](./01-pantallas-visuales-mvp.md), [02-login-registro-visual](./02-login-registro-visual.md)
> **Date:** 2026-07-07
> **Objective:** Portar la pantalla Home (`home.jsx`) como página principal (`/`) de Arcade Vault, mover la Biblioteca actual a `/games`, y actualizar el Nav superior con las pestañas Inicio / Biblioteca / Salón de la Fama / Acerca de (esta última visible pero inerte, sin ruta todavía), según el diseño de `references/templates/home-about/`.

## Scope

**In:**

- Nueva página Home portada a `app/page.tsx` (ruta `/`, reemplaza el contenido actual): hero con silhouettes flotantes decorativas, sección "¿Por qué Arcade Vault?" (grid de 4 features), vitrina "Juegos disponibles ahora" (6 tarjetas mini desde `GAMES` de `lib/games.ts`), sección de stats, sección "Actividad en vivo" (ticker de últimas puntuaciones + top 5 jugadores, datos hardcodeados igual que el template), sección de precios (plan único + FAQ), CTA final. Incluye el hook de reveal-on-scroll (`IntersectionObserver`) portado de `home.jsx`.
- La Biblioteca actual (buscador, chips, grid de `GameCard`) se mueve de `app/page.tsx` a `app/games/page.tsx` (ruta `/games`), sin cambios de comportamiento.
- Todos los CTAs de Home mapeados a rutas reales ya existentes:
  - "EXPLORAR JUEGOS", "VER TODOS LOS JUEGOS →", "INSERTAR MONEDA →" → `/games`.
  - "CREAR CUENTA", "EMPEZAR GRATIS →" → `/login`.
  - Click en una mini-card de la vitrina → `/game/[id]`.
  - "VER SALÓN →" → `/hall-of-fame`.
- `components/Nav.tsx` actualizado con 4 pestañas en este orden: **Inicio** (`/`), **Biblioteca** (`/games`), **Salón de la Fama** (`/hall-of-fame`), **Acerca de** (visible, sin `href` ni `onClick` — inerte, igual que se hizo con "Iniciar Sesión" antes de la spec 02). Mismo cambio replicado en el panel móvil.
- Lógica de sección activa actualizada: Inicio activo en `/`; Biblioteca activo en `/games` y `/game/*`; Salón activo en `/hall-of-fame`; Acerca de nunca activo (no tiene ruta).
- Portar a `app/globals.css` únicamente las clases CSS de `home-about/styles.css` que usa `home.jsx` (hero, silos, mini-card/rail, feature-grid, stats, activity/ticker/top-list, pricing/faq, CTA final, `reveal`/`reveal.in`, section-head/title/rule).

**Out of scope (para futuros specs):**

- Pantalla About (`about.jsx`): misión, highlights y formulario de contacto — la pestaña "Acerca de" queda inerte hasta ese spec futuro.
- Clases CSS exclusivas de About (`about-*`, `contact-*`, `ab-*`, `gp-*`, `dp-*`, `terminal-success`, `highlight*`) — no se portan todavía.
- Datos reales de actividad/ranking (el ticker y el top de jugadores siguen siendo mock hardcodeado, igual que el template).
- Conectar `MOCK_USER` al Nav o a cualquier estado de sesión real.
- Tests automatizados.

## Data model

Esta feature no introduce datos nuevos compartidos — la vitrina de juegos reutiliza `GAMES` de `lib/games.ts` (ya existente), y los arrays de "últimas puntuaciones" / "top jugadores" son constantes decorativas locales al componente Home (igual que en el template), sin persistencia ni tipos propios. Se omite esta sección.

## Implementation plan

1. Mover el contenido actual de `app/page.tsx` (Biblioteca) a `app/games/page.tsx`, sin cambios de comportamiento. Verificación: `npm run dev`, `/games` muestra la Biblioteca igual que antes (buscador, chips, grid).
2. Portar a `app/globals.css` las clases CSS de `references/templates/home-about/styles.css` que usa `home.jsx` (hero, silhouettes, `mini-card`/`mini-rail`, `feature-grid`, `home-stats`, `activity-grid`/`ticker`/`top-list`, `pricing-grid`/`faq`, CTA final, `reveal`/`reveal.in`, `section-head`/`section-title`/`section-rule`). Verificación: `npm run dev` sin errores de build; `/games`, `/login`, `/hall-of-fame`, `/game/[id]` no muestran regresiones visuales.
3. Crear `app/page.tsx` con el componente Home portado de `home.jsx`: hero + silhouettes decorativas, hook de reveal-on-scroll (`IntersectionObserver`), secciones "¿Por qué Arcade Vault?", vitrina de 6 juegos (`GAMES.slice(0, 6)` de `lib/games.ts`), stats, "Actividad en vivo" (ticker + top jugadores, arrays hardcodeados como en el template), precios + FAQ, CTA final. Todos los `onClick`/`Link` mapeados a rutas reales: vitrina y CTAs de juego → `/games` o `/game/[id]`, CTAs de cuenta → `/login`, "VER SALÓN →" → `/hall-of-fame`. Verificación: `npm run dev`, `/` muestra el Home completo, las secciones se revelan al hacer scroll, cada CTA navega a la ruta correcta.
4. Actualizar `components/Nav.tsx` (desktop y panel móvil): agregar link "Inicio" (`/`) antes de "Biblioteca"; agregar "Acerca de" al final de los links, sin `href` ni `onClick` (inerte); actualizar `isActive` — Inicio en `/`, Biblioteca en `/games` y `/game/*`, Salón en `/hall-of-fame`. Verificación: el Nav muestra las 4 pestañas en desktop y en el menú hamburguesa móvil, resalta la sección activa correctamente según la ruta, "Acerca de" no navega a ningún lado al hacer click.
5. Pasada final de QA visual: recorrer `/`, `/games`, `/game/[id]`, `/game/[id]/play`, `/hall-of-fame` y `/login` comparando contra `references/templates/home-about/arcade-vault-standalone.html`, revisar breakpoints móviles, confirmar `npm run lint` + `npx tsc --noEmit` sin errores y sin errores de consola al navegar.

## Acceptance criteria

- [ ] `npm run dev` levanta sin errores y `/` muestra el Home completo: hero con silhouettes decorativas, sección "¿Por qué Arcade Vault?", vitrina de 6 juegos, stats, "Actividad en vivo", precios + FAQ y CTA final.
- [ ] Las secciones marcadas `reveal` se animan al entrar en el viewport al hacer scroll.
- [ ] "EXPLORAR JUEGOS", "VER TODOS LOS JUEGOS →" e "INSERTAR MONEDA →" navegan a `/games`.
- [ ] "CREAR CUENTA" y "EMPEZAR GRATIS →" navegan a `/login`.
- [ ] Click en cualquier mini-card de la vitrina navega a `/game/[id]` del juego correspondiente.
- [ ] "VER SALÓN →" navega a `/hall-of-fame`.
- [ ] `/games` muestra la Biblioteca (buscador, chips, grid de 8 juegos) exactamente igual que antes de mover la ruta.
- [ ] El Nav (desktop y menú móvil) muestra 4 pestañas en orden: Inicio, Biblioteca, Salón de la Fama, Acerca de.
- [ ] El Nav resalta "Inicio" en `/`, "Biblioteca" en `/games` y en `/game/[id]`/`/game/[id]/play`, "Salón de la Fama" en `/hall-of-fame`.
- [ ] Click en "Acerca de" no navega a ninguna ruta ni produce errores de consola.
- [ ] El logo del Nav sigue navegando a `/`.
- [ ] No existe la ruta `/about` ni `/acerca-de` en la aplicación.
- [ ] `npx tsc --noEmit` y `npm run lint` pasan sin errores.
- [ ] No hay errores en la consola del navegador al navegar entre `/`, `/games`, `/game/[id]`, `/game/[id]/play`, `/hall-of-fame` y `/login`.

## Decisions

- **Sí:** Home reemplaza a la Biblioteca como página principal (`/`); la Biblioteca se mueve a `/games`. Razón: pedido explícito del usuario ("implementar el home como página principal").
- **Sí:** ruta `/games` en inglés en vez de `/biblioteca`. Razón: consistencia con el resto de rutas del proyecto (`/hall-of-fame`, `/game/[id]`, `/login`), todas en inglés — elegida por el usuario sobre la alternativa en español.
- **No:** implementar la pantalla About (`about.jsx`) en esta spec. Razón: decisión explícita del usuario — se agrega la pestaña "Acerca de" al Nav para fidelidad visual del diseño de referencia, pero la pantalla en sí queda para un spec futuro.
- **Sí:** pestaña "Acerca de" visible pero inerte (sin `href` ni `onClick`) mientras no exista la ruta. Razón: mismo patrón ya usado en la spec 01 para el botón "Iniciar Sesión" antes de que existiera `/login` — evita un link roto o un placeholder que confunda al usuario final.
- **Sí:** vitrina de "Juegos disponibles ahora" en Home usa `GAMES` de `lib/games.ts` (no un array separado). Razón: decisión del usuario — evita duplicar datos que ya existen y mantiene una sola fuente de verdad para los juegos.
- **Sí:** ticker de "últimas puntuaciones" y "top jugadores" quedan hardcodeados como arrays locales al componente Home, igual que en el template. Razón: decisión del usuario — no requieren conectarse a `PLAYERS`/`seededScores` de `lib/games.ts`, son puramente decorativos y así reduce alcance nuevo.
- **Sí:** se portan a `app/globals.css` solo las clases CSS que usa `home.jsx`, excluyendo las exclusivas de About (`about-*`, `contact-*`, `ab-*`, `gp-*`, `dp-*`, `terminal-success`, `highlight*`). Razón: coherente con dejar About fuera del alcance — portar esas clases ahora sería trabajo muerto hasta que exista esa spec.
- **No:** tests automatizados. Razón: no se pidió; la verificación es manual vía dev server, igual que specs 01 y 02.

## Risks

| Riesgo | Mitigación |
|---|---|
| Mover `app/page.tsx` a `app/games/page.tsx` deja `/` sin contenido momentáneamente durante el desarrollo (paso 1 del plan antes del paso 3). | Ejecutar los pasos del plan en orden; no es un estado que se despliegue, solo un intermedio local durante la implementación. |
| El Nav actual resalta "Biblioteca" en `pathname === "/"`; al mover la Biblioteca a `/games`, si se olvida actualizar `isActive`, "Biblioteca" podría quedar marcada activa en el Home o inactiva en `/games`. | Verificación explícita en el paso 4 y en los criterios de aceptación: probar la sección activa en las 6 rutas antes de dar por cerrada la spec. |
| Las ~150 líneas de CSS nuevas portadas desde `home-about/styles.css` podrían colisionar en nombre con clases ya existentes en `globals.css` (ambos archivos comparten convenciones `av-*`/`pixel`/`neon-*`). | Antes de pegar el CSS, revisar que ningún selector nuevo (`home-*`, `mini-*`, `feature-*`, `activity-*`, `pricing-*`, `pc-*`, `faq-*`, `reveal`) ya exista con otro significado en `globals.css`; en este spec ya se confirmó que no hay colisión con las clases actuales. |
| Next.js 16.2.10 puede manejar la navegación client-side (`usePathname`, `Link`) de forma distinta a versiones anteriores. | Consultar `node_modules/next/dist/docs/01-app/` antes de tocar `components/Nav.tsx` y `app/page.tsx`. |

## What is **not** in this spec

- Pantalla About (`about.jsx`) y su formulario de contacto.
- Clases CSS exclusivas de About.
- Datos reales de actividad/ranking.
- Conexión de `MOCK_USER` al Nav o a cualquier sesión real.
- Tests automatizados.

Cada uno de estos, si se implementa, irá en su propia spec.
