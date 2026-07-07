# SPEC 01 — Pantallas visuales de Arcade Vault (MVP)

> **Status:** Aprobado
> **Depends on:** —
> **Date:** 2026-07-03
> **Objective:** Portar las 5 pantallas visuales de Arcade Vault (Biblioteca, Detalle, Reproductor, Login, Salón de la Fama) desde `references/templates/` a rutas reales de Next.js App Router, sin implementar ningún juego funcional.

## Scope

**In:**

- 4 pantallas enrutadas con App Router: Biblioteca (`/`), Detalle de juego (`/game/[id]`), Reproductor (`/game/[id]/play`), Salón de la Fama (`/hall-of-fame`).
- `Nav` compartido (enlaces desktop + menú hamburguesa móvil) y footer, visibles en todas las rutas, con resaltado de sección activa según la ruta actual.
- Botón "Iniciar Sesión" en el Nav visible por fidelidad visual, sin acción real (no navega, no hay ruta `/login` ni formulario).
- Un usuario simulado fijo y constante (ej. `{ name: "PLAYER1" }`) usado únicamente para renderizar las variantes visuales de "sesión iniciada": la fila "TU MEJOR MARCA" en Salón de la Fama y el nombre por defecto en el Reproductor. No está conectado al Nav ni a ningún flujo de login.
- Datos mock (8 juegos, generador de puntuaciones `seededScores`, lista de jugadores) portados a TypeScript.
- Reproductor con HUD (jugador, puntuación, vidas, nivel), simulación animada de puntuación (incremento aleatorio por intervalo, subida de nivel), pausa, modal de fin de juego con guardado de puntuación en `localStorage` (`av_scores`) — todo puramente visual/decorativo, sin lógica de juego real.
- Salón de la Fama con tabs por juego, podio (top 3) y tabla de 12 posiciones, incluyendo la fila fija "tu mejor marca" descrita arriba.
- Reutilización íntegra del tema visual ya portado en `app/globals.css` (colores neón, tipografías pixel/mono, animaciones, breakpoints responsive).

**Out of scope (para futuros specs):**

- Pantalla de Login/registro (`auth.jsx` del template) y cualquier autenticación real (OAuth, hashing, base de datos, sesión real).
- Lógica real de cualquiera de los 8 juegos (colisiones, controles, física).
- Sistema de créditos/moneda real (el contador "CRÉDITOS · 03" queda estático).
- Multijugador real (p. ej. "Duelo Pixel" a dos jugadores).
- Sonido / audio.
- Tests automatizados.

## Data model

Ubicación: `lib/games.ts` (nuevo archivo, tipado en TypeScript, sin dependencias de React).

```ts
export interface Game {
  id: string;
  title: string;
  short: string;
  long: string;
  cat: "ARCADE" | "PUZZLE" | "SHOOTER" | "VERSUS";
  cover: string; // clase CSS del cover art, ej. "cover-bricks"
  color: "cyan" | "magenta" | "yellow" | "green";
  best: number;
  plays: string;
}

export const CATS = ["TODOS", "ARCADE", "PUZZLE", "SHOOTER", "VERSUS"] as const;

export interface ScoreRow {
  rank: number;
  name: string;
  score: number;
  date: string;
}

// seededScores(seed, count) — misma generación pseudoaleatoria determinista del template
export function seededScores(seed: number, count?: number): ScoreRow[];
```

Constante de usuario simulado (vive junto a los componentes que la usan, no en `lib/games.ts`):

```ts
const MOCK_USER = { name: "PLAYER1" } as const;
```

Forma persistida en `localStorage` bajo la clave `av_scores` (array, se hace `push` de una entrada por cada partida guardada en el Reproductor):

```ts
interface SavedScoreEntry {
  game: string;  // Game.id
  score: number;
  name: string;  // iniciales ingresadas en el modal de fin de juego
  at: number;    // Date.now()
}
```

Convenciones:

- `Game.id` es el slug usado en la URL (`/game/[id]`).
- `seededScores` es determinista: mismo `seed` produce siempre las mismas filas (se usa `id.length * N + M` como semilla en Detalle y Salón, igual que el template).

## Implementation plan

1. Crear `lib/games.ts`: tipos `Game`/`ScoreRow`, arreglo `GAMES` (8 juegos), `CATS`, `PLAYERS` y `seededScores()`, portados de `references/templates/data.jsx`. Verificación manual: `npx tsc --noEmit` sin errores.
2. Crear `components/GameCard.tsx` (tarjeta con tilt al hover, portada, badge de mejor puntuación, botón "JUGAR") portando la lógica de `GameCard` en `biblioteca.jsx`.
3. Reescribir `app/page.tsx` como la pantalla Biblioteca: buscador, chips de categoría, grid de `GameCard`, estado vacío "NO HAY RESULTADOS". Verificación: `npm run dev`, `/` muestra el grid completo y el filtro funciona.
4. Crear `components/Nav.tsx` (client component): logo, links Biblioteca/Salón de la Fama con resaltado activo vía `usePathname`, contador de créditos estático, botón "Iniciar Sesión" inactivo, menú hamburguesa móvil con backdrop. Añadirlo a `app/layout.tsx` junto con un footer fijo. Verificación: el Nav aparece en `/` y resalta "Biblioteca".
5. Crear `app/game/[id]/page.tsx` para el Detalle: portada, tags, descripción, stat-strip, tabla de mejores puntuaciones (`seededScores`), botones "JUGAR AHORA" / "VOLVER AL VAULT". Manejar `id` inexistente con `notFound()`. Verificación: navegar desde una tarjeta muestra el detalle correcto.
6. Crear `app/game/[id]/play/page.tsx` para el Reproductor: HUD, arena CRT decorativa, simulación de puntuación por intervalo, pausa, subida de nivel, modal de fin de juego con guardado en `localStorage` (`av_scores`) usando `MOCK_USER.name` como nombre por defecto. Verificación: jugar, pausar, terminar y guardar puntuación sin errores de consola.
7. Crear `app/hall-of-fame/page.tsx`: tabs por juego, podio top 3, tabla de 12 filas, fila fija "TU MEJOR MARCA" usando `MOCK_USER`. Verificación: cambiar de tab actualiza podio y tabla.
8. Pasada final de QA visual: recorrer las 4 rutas en el navegador comparando contra `references/templates/Arcade Vault.html`, revisar breakpoints móviles (<840px, <900px, <720px) y confirmar `npm run lint` + `npx tsc --noEmit` sin errores.

## Acceptance criteria

- [ ] `npm run dev` levanta sin errores y `/` muestra la Biblioteca con buscador, chips de categoría y grid de 8 juegos.
- [ ] Escribir en el buscador o seleccionar una categoría filtra el grid; si no hay resultados se muestra "NO HAY RESULTADOS".
- [ ] Click en una tarjeta o en su botón "JUGAR" navega a `/game/[id]` mostrando portada, tags, descripción, stats y tabla de mejores puntuaciones para ese juego.
- [ ] Un `id` de juego inexistente en `/game/[id]` responde con la página 404 de Next.js.
- [ ] "JUGAR AHORA" navega a `/game/[id]/play`, muestra el HUD (jugador, puntuación, vidas, nivel) y la puntuación sube sola cada ~220ms.
- [ ] "PAUSA" detiene el incremento de puntuación y muestra el overlay "EN PAUSA"; "REANUDAR" lo retoma.
- [ ] "FIN" abre el modal de fin de juego con la puntuación final e input de iniciales.
- [ ] Guardar la puntuación la persiste en `localStorage` (`av_scores`) y muestra el toast "PUNTUACIÓN GUARDADA".
- [ ] `/hall-of-fame` muestra tabs por juego, podio (2º/1º/3º) y tabla de 12 filas; cambiar de tab actualiza podio y tabla.
- [ ] La fila "TU MEJOR MARCA" aparece siempre en `/hall-of-fame` usando el usuario simulado fijo.
- [ ] El Nav resalta la sección activa según la ruta actual, muestra "Iniciar Sesión" sin acción real, y el menú hamburguesa funciona en viewport móvil (<840px).
- [ ] No existe la ruta `/login` ni ningún formulario de autenticación en la aplicación.
- [ ] `npx tsc --noEmit` y `npm run lint` pasan sin errores.
- [ ] No hay errores en la consola del navegador al navegar entre las 4 pantallas.

## Decisions

- **Sí:** rutas reales de Next.js App Router con slugs en inglés (`/game/[id]`, `/game/[id]/play`, `/hall-of-fame`) en vez del router hash-based del template. Razón: convención del proyecto (App Router only, sin `pages/`) y navegación nativa.
- **No:** pantalla de Login/registro en esta spec. Razón: el usuario decidió no implementar autenticación todavía; se retoma en un spec futuro junto con auth real.
- **Sí:** mantener el botón "Iniciar Sesión" visible en el Nav pero sin acción real. Razón: fidelidad visual del diseño de referencia sin necesitar la pantalla ni lógica de auth.
- **Sí:** usuario simulado fijo (`MOCK_USER`, ej. `PLAYER1`) para mostrar las variantes "con sesión" en Salón de la Fama y Reproductor. Razón: esas variantes son parte del diseño visual a portar, y sin esto quedarían sin cubrir hasta que exista auth real.
- **Sí:** `localStorage` (`av_scores`) para persistir puntuaciones guardadas en el Reproductor. Razón: MVP sin backend, ya validado visualmente en el template; no requiere sesión de usuario para funcionar.
- **Sí:** organización híbrida de componentes — compartidos (`Nav`, `GameCard`) en `components/`, pantallas específicas co-localizadas en su carpeta de ruta bajo `app/`. Razón: evita una carpeta `components/` masiva mientras reutiliza piezas comunes.
- **Sí:** datos mock (`GAMES`, `seededScores`, `PLAYERS`) viven en `lib/games.ts` tipados con TypeScript. Razón: separar datos de UI y aprovechar el path alias `@/lib/games`.
- **Sí:** mantener la simulación animada de puntuación en el Reproductor tal como está en el template. Razón: es decorativa (no hay lógica de juego real), y removerla quitaría fidelidad visual sin necesidad.
- **No:** implementar lógica real de los 8 juegos. Razón: explícitamente fuera de alcance por el usuario.
- **No:** tests automatizados. Razón: no se pidió; la verificación es manual vía dev server.

## Risks

| Riesgo                                                                 | Mitigación                                                                                                   |
| ----------------------------------------------------------------------| ---------------------------------------------------------------------------------------------------------- |
| Next.js 16.2.10 puede manejar `params` de rutas dinámicas (`[id]`) distinto a versiones anteriores (ej. como Promise). | Consultar `node_modules/next/dist/docs/01-app/` antes de implementar `app/game/[id]/page.tsx` y `app/game/[id]/play/page.tsx`. |
| `localStorage` deshabilitado (modo privado) rompería el guardado de puntuaciones. | La app sigue funcionando; el guardado simplemente no persiste entre recargas (fallar en silencio, sin crashear la UI). |
| Sin pantalla de Login, algún componente portado podría referenciar `user`/`onLogin` que ya no existen. | Al portar `salon.jsx` y `reproductor.jsx`, reemplazar toda referencia a `user` por la constante `MOCK_USER` fija. |

## What is **not** in this spec

- Pantalla de Login/registro y cualquier autenticación real (será otra spec).
- Lógica real de cualquiera de los 8 juegos (colisiones, controles, física).
- Sistema de créditos/moneda real.
- Multijugador real (ej. "Duelo Pixel" a dos jugadores).
- Sonido / audio.
- Tests automatizados.

Cada uno de estos, si se implementa, irá en su propia spec.
