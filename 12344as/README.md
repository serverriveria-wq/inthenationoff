# IN THE NATION (v2 · "Cristal Crepuscular")
Front-end derivado del export de InTheNation, ahora con identidad propia. Backend: Supabase `cmkzvbpdloucicnryynl`.

## Qué cambió
- **Diseño propio**: `itn/ui.css` + `itn/ui.js` se cargan en todas las páginas (fondo aurora, cristal translúcido, menú superior con icono+subtítulo, menú lateral, franja "Últimas novedades", campana de notificaciones).
- **Login solo con Google** (`login.html` → Supabase OAuth → `verify.html` guarda la sesión). `register.html` y `forgot-password.html` redirigen a login.
- **Modo Administrador**: si entras con `elnevulex@gmail.com` (tabla `admin_emails`, ya existe) aparece el panel flotante: editar textos sobre cualquier página, cambiar colores/redondeo del tema para todos, aviso global y acceso a `admin.html` (jugadores, contenido, reglas, objetos).
- **Misión de pareja omitible** (`find_partner.optional = true`; botones Omitir/Retomar en `couple.html`).
- **Ajustes / Notify** (`settings.html`): avisos del navegador, categorías, resumen sin emergentes, horas de silencio. Las **guerras avisan siempre a todos** (trigger en `wars`). Anti-spam: 1 aviso emergente cada 30 s, agrupado.

## Despliegue (en este orden)
1. Supabase SQL Editor: ejecutar `backend/migration.sql`.
2. Desplegar la función nueva: `supabase functions deploy itn --no-verify-jwt` (código en `backend/functions/itn/index.ts`). La función `api` no se modifica.
3. Auth → Providers → **Google**: activar con tus Client ID/Secret (Google Cloud Console, redirect URI: `https://cmkzvbpdloucicnryynl.supabase.co/auth/v1/callback`).
4. Auth → URL Configuration: Site URL y Redirect URLs con `https://TU-DOMINIO/verify.html`.
5. Auth → Providers → **Email**: desactivar "Enable email signups" para que Google sea realmente la única vía.
6. Subir la carpeta `inthenation.org/` a Vercel/Netlify (ya incluye los rewrites `/api/itn/*` y `/api/*`).

## v2.1 · pantallas propias + push en segundo plano
- `dashboard.html`, `market.html` y `map.html` ahora son propias (HTML + JS sin build), usando la misma API. Quedan compiladas solo páginas informativas (mundo, ayuda, reglas, tienda, etc.).
- Push con navegador cerrado: `sw.js` (service worker), suscripción en Ajustes, envío desde la función `itn` (`/push/dispatch`) cada minuto vía pg_cron. Respeta categorías, resumen y horas de silencio; las guerras siempre salen. Máx. 1 push por usuario por minuto.
- Pasos extra: ejecutar `backend/migration-push.sql` (cambiando TU_PUSH_CRON_SECRET) y cargar como secretos de la función `itn` los valores de `SECRETS-NO-SUBIR.txt (raíz del zip)` (VAPID_PUBLIC_KEY, VAPID_PRIVATE_KEY, VAPID_SUBJECT, PUSH_CRON_SECRET). **No subas ese archivo a ningún sitio público ni lo dejes en el hosting.** La clave pública ya está en `itn/ui.js`.
- iPhone/iPad: el push solo funciona con la web añadida a la pantalla de inicio.

## v3 · diseño "Azul Real" (basado en tu referencia Nerva FC)
- **Menú**: barra lateral oscura con iconos en escritorio; barra inferior en móvil. Cabecera con bandera, nombre, campana y ajustes.
- **Assets optimizados** (carpeta `itn/`): `assets.svg` = un solo archivo con 33 iconos + 16 banderas simples + marca (se usan con `<use href="/itn/assets.svg#i-home">`); `atlas.webp` = 2 ornamentos dorados + degradado en una sola imagen (10 KB); `bg.webp` = patrón geométrico (8 KB, antes 1.1 MB). Cache de 1 día en `vercel.json`.
- **Botones** planos con relieve inferior, sin bordes. **Azulejos** de menú en el panel como en la referencia.
- **index.html**: solo mapa de Rusia con nieve cayendo, botón de entrar y contadores reales (`/api/world/stats` y regiones abiertas). Sin ranking, telegramas, mensajes ni guerras. `world.html` ahora lista jugadores reales y naciones.
- **Solo Rusia abierta**: `game_config.open_countries = ["RU"]` (editable en el panel admin → game_config). El mapa bloquea el resto, `migration.sql` agrega un trigger que impide viajar a regiones cerradas, y los jugadores nuevos entran a Moscú (`/api/itn/join`).
- **Renombrado**: "Parlamentum" → "InTheNation" en textos, enlaces y nombres de archivo (misma longitud para no romper las páginas compiladas); enlaces a redes ajenas quedaron en `#`. Logo y wordmark nuevos con texto "IN THE NATION".
- Las imágenes `127536.webp` (hoja de sprites) se usaron solo como guía de estructura de atlas; no se incluyeron.

## Límites conocidos
- Faltan por reescribir las pantallas informativas compiladas (tienda, ayuda, mundo, reglas) y funciones avanzadas del juego original (política, empresas, guerra) que el backend `api` actual aún no implementa.
- El mapa usa Leaflet con teselas externas (CARTO/Esri, requieren internet y atribución). Países y regiones se dibujan como marcadores por coordenadas (no hay polígonos de fronteras en la base de datos).
- Las ediciones de texto del modo admin se guardan en `site_edits` y se reaplican al cargar; en pantallas que React vuelve a pintar pueden tardar un instante.
- Avisos con el navegador cerrado requieren service worker + VAPID (no incluido).
