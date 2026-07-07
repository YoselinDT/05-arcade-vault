# SPEC 02 — Login y registro (solo visual)

> **Status:** Implementado
> **Depends on:** [01-pantallas-visuales-mvp](./01-pantallas-visuales-mvp.md)
> **Date:** 2026-07-07
> **Objective:** Portar la pantalla de login/registro (`auth.jsx` del template) a la ruta `/login` de Next.js App Router, conectando el botón "Iniciar Sesión" del Nav, sin implementar autenticación real ni cambiar el estado de sesión en ninguna pantalla.

## Scope

**In:**

- 1 pantalla nueva enrutada con App Router: `/login`, con dos tabs — "INICIAR SESIÓN" y "CREAR CUENTA" — dentro de la misma card, portada de `auth.jsx`.
- Formulario de "Iniciar sesión": campos Usuario y Contraseña, botón "ENTRAR AL VAULT".
- Formulario de "Crear cuenta": campos Usuario, Correo electrónico y Contraseña, botón "CREAR Y JUGAR" (el campo de correo aparece solo en este tab, con animación `slide-in` ya existente en `globals.css`).
- Botón "JUGAR COMO INVITADO" que navega a la Biblioteca (`/`).
- Botones decorativos "GOOGLE" y "GITHUB" (sin acción al click, solo fidelidad visual).
- Ambos submits (login y registro) son puramente decorativos: no validan, no crean ningún estado de sesión, y solo redirigen a la Biblioteca (`/`) — igual de simulado que el resto del MVP.
- Botón "Iniciar Sesión" del Nav (desktop y menú móvil) pasa de inerte a navegar a `/login` usando `next/link`.
- Reutilización íntegra de las clases ya portadas en `app/globals.css` (`av-auth-wrap`, `auth-card`, `auth-header`, `auth-tabs`, `field`, `auth-divider`, `social`, `slide-in`).

**Out of scope (para futuros specs):**

- Autenticación real (OAuth con Google/GitHub, hashing de contraseñas, base de datos de usuarios, backend de sesiones).
- Cualquier estado de "sesión iniciada" que persista o afecte al Nav, Reproductor o Salón de la Fama — `MOCK_USER` sigue fijo y desconectado del flujo de login, tal como quedó definido en la spec 01.
- Validación de formulario (ni HTML ni JS) — los campos no son `required`, se puede enviar vacío.
- Recuperación de contraseña / "olvidé mi contraseña".
- Cierre de sesión (`onSignOut`) — no aplica porque no hay sesión real que cerrar.
- Tests automatizados.

## Data model

Esta feature no introduce datos nuevos — no hay estado, no hay tipos, no hay persistencia. Se omite esta sección.

## Implementation plan

1. Crear `app/login/page.tsx` (client component, sin `"use client"` extra lógica de estado más allá del tab activo): estructura `av-auth-wrap` → `auth-card` con header (logo, "ARCADE VAULT", subtítulo "ACCESO AL SISTEMA · v2.6"), tabs controlados por `useState<"in" | "up">`. Verificación: `npm run dev`, `/login` renderiza la card con el tab "INICIAR SESIÓN" activo por defecto.
2. Formulario del tab "in": campos Usuario/Contraseña, botón "ENTRAR AL VAULT" tipo submit. Formulario del tab "up": añade campo Correo electrónico (con clase `slide-in`) y botón "CREAR Y JUGAR". El `onSubmit` de ambos hace `e.preventDefault()` y navega a `/` con `useRouter().push("/")`. Verificación: cambiar de tab muestra/oculta el campo de correo; enviar cualquiera de los dos formularios redirige a la Biblioteca sin errores de consola.
3. Añadir botón "JUGAR COMO INVITADO" (navega a `/` vía `next/link` o `router.push`) y bloque "O CONTINÚA CON" con botones "GOOGLE"/"GITHUB" sin `onClick` funcional. Verificación: click en invitado navega a `/`; click en Google/Github no hace nada visible salvo el estado hover propio del `.btn`.
4. Actualizar `components/Nav.tsx`: reemplazar el `<button className="btn auth-btn">Iniciar Sesión</button>` inerte por un `Link href="/login"` con la misma clase, tanto en el nav desktop como en el link "Iniciar Sesión" del panel móvil (cerrando el menú al navegar, igual que los demás links). Verificación: click en "Iniciar Sesión" desde desktop y desde el menú hamburguesa móvil navega a `/login`.
5. Pasada final de QA visual: comparar `/login` contra `references/templates/Arcade Vault.html` (tab de login y de registro), revisar el breakpoint móvil (<840px) para la card de auth, confirmar `npm run lint` + `npx tsc --noEmit` sin errores.

## Acceptance criteria

- [ ] Click en "Iniciar Sesión" del Nav (desktop) navega a `/login`.
- [ ] Click en "Iniciar Sesión" del menú hamburguesa móvil navega a `/login` y cierra el menú.
- [ ] `/login` muestra la card de auth con el tab "INICIAR SESIÓN" activo por defecto: campos Usuario y Contraseña, botón "ENTRAR AL VAULT".
- [ ] Click en el tab "CREAR CUENTA" muestra el campo adicional "Correo electrónico" (con animación de entrada) y cambia el botón a "CREAR Y JUGAR".
- [ ] Enviar cualquiera de los dos formularios (con campos vacíos o llenos, sin validación) redirige a la Biblioteca (`/`).
- [ ] Click en "JUGAR COMO INVITADO" redirige a la Biblioteca (`/`).
- [ ] Los botones "GOOGLE" y "GITHUB" se muestran con estilo `.btn.ghost` pero no ejecutan ninguna acción al click.
- [ ] No existe ningún cambio visible en el Nav, Reproductor o Salón de la Fama tras usar `/login` — `MOCK_USER` sigue fijo e independiente.
- [ ] `npx tsc --noEmit` y `npm run lint` pasan sin errores.
- [ ] No hay errores en la consola del navegador al navegar hacia/desde `/login` ni al cambiar de tab.

## Decisions

- **Sí:** ruta única `/login` con tabs internos ("Iniciar sesión" / "Crear cuenta"), en vez de dos rutas separadas. Razón: así está diseñado en `auth.jsx` del template (una sola card que cambia de formulario según el tab), y el usuario confirmó seguir el diseño de referencia tal cual.
- **No:** simular un estado de "sesión iniciada" al enviar el formulario. Razón: el usuario pidió explícitamente que el submit sea "puramente decorativo" — solo redirige a la Biblioteca, sin tocar `MOCK_USER` ni el Nav, manteniendo la separación ya establecida en la spec 01.
- **Sí:** nombre de ruta en inglés (`/login`) en vez de `/auth`. Razón: consistencia con el resto de rutas del proyecto (`/game/[id]`, `/hall-of-fame`), todas nombradas en inglés.
- **No:** validación de formulario (ni HTML ni JS). Razón: decisión explícita del usuario — el submit funciona con campos vacíos, igual que el comportamiento del template original.
- **Sí:** conectar el botón "Iniciar Sesión" del Nav (desktop y móvil) para que navegue a `/login`. Razón: sin esto la pantalla quedaría inalcanzable desde la navegación normal de la app.
- **Sí:** botones "JUGAR COMO INVITADO" navega a Biblioteca; "GOOGLE"/"GITHUB" quedan sin acción. Razón: fidelidad visual al template sin necesidad de lógica de OAuth, que está fuera de alcance.
- **No:** cierre de sesión (`onSignOut`) ni ningún estado de usuario logueado. Razón: no hay sesión real que cerrar en este spec.

## Risks

| Riesgo | Mitigación |
|---|---|
| Next.js 16.2.10 puede diferir en las APIs de navegación client-side (`useRouter`, `next/link`) respecto a versiones anteriores. | Consultar `node_modules/next/dist/docs/01-app/` antes de implementar la navegación en `app/login/page.tsx` y el cambio en `components/Nav.tsx`. |
| La clase `.btn.auth-btn` se usaba en un `<button>`; al pasar a `Link` (que renderiza un `<a>`) el estilo podría no aplicarse igual (line-height, display). | Verificar visualmente que el botón "Iniciar Sesión" se ve idéntico antes/después del cambio, tanto en desktop como en el menú móvil. |

## What is **not** in this spec

- Autenticación real (OAuth, hashing, base de datos, backend de sesiones).
- Estado de sesión que persista o afecte al Nav, Reproductor o Salón de la Fama.
- Validación de formulario.
- Recuperación de contraseña.
- Cierre de sesión.
- Tests automatizados.

Cada uno de estos, si se implementa, irá en su propia spec.
