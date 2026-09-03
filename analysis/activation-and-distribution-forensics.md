# ANEXO FORENSE — Activación / versión de pago (análisis, no explotación)

Alcance: examinar el artefacto `HW Goodwin-5.1.txt` (cargador v5.1) en busca de lógica de activación/licencia y superficie de rastreo. **Análisis descriptivo; no se construyó ningún bypass ni versión modificada.**

## 1. Hallazgo principal: NO hay lógica de activación en el cargador
La tabla decodificada completa (índices 79–233, ver `string-map.json`) fue inspeccionada buscando términos de licencia/activación (activ, licen, pay, key, user, account, auth, premium, pro, token, expir, valid, check...). Resultado: **ninguna coincidencia real** (únicos hits: fragmentos "Proper"/"protot" de `defineProperty`/`prototype`).

El flujo de control completo (banner → hooks → `loader()` → `tryLoadMirror()` → fetch → inyección → caché) **no contiene**: claves de licencia, comprobaciones de usuario/cuenta, fechas de expiración, ni comparación de "versión gratis vs. de pago".

## 2. Dónde viviría la verificación de pago (modelo servidor)
- El cargador es un bootstrap "fetch-e-inyecta": `fetch("https://<mirror>/updates/loader.php")` → inyecta el JS descargado → lo guarda en caché.
- La lógica de pago/activación, si existe, está **en el servidor**: en `loader.php` o en el código de cheat que este entrega. El servidor puede servir contenido distinto según el solicitante (IP, user-agent, headers, cookies, referrer, estado de la cuenta).
- Por tanto, "versión de pago local": **no existe**. No hay binario/parche local que activar; el único camino de análisis sería descargar lo que el servidor entrega (ver §4).

## 3. Superficie de rastreo observada en el artefacto
Señales que ya emite el cargador (relevantes para el distribuidor y para entender el modelo):
- `@updateURL`/`@downloadURL` → `https://switzerland.goodwin.best/updates/HW-Goodwin.user.js`: el propio usuarioscript se autoactualiza desde el servidor del autor.
- Cada carga de la página dispara `fetch` a 4 mirrors (`switzerland`, `hw-script`, `finland`, `russia`.goodwin.best/updates/loader.php) → el servidor ve cada ejecución (IP, momento, agente).
- Persistencia: `localStorage["HW Goodwin:usedDomain"]` (recuerda el último mirror usado) y Cache Storage `"HW-Goodwin-loader"`.
- NO se observan en el artefacto: identificadores de cuenta, tokens, claves, ni comunicación bidireccional con la cuenta del usuario. El rastreo es, por tanto, del lado servidor.

## 4. Próximo paso forense legítimo (si se desea)
1. En sandbox (no en un navegador real), descargar `https://<mirror>/updates/loader.php` tal como lo entregaría el servidor y aplicar el mismo proceso de desofuscación (LZ-String + base91 + diccionario) para localizar cualquier lógica de licencia/pago.
2. Documentar en `INFORME_FINAL.md` los hallazgos (mecanismos, no exploits).
3. No se realizó: ni descarga de `loader.php`, ni modificación del artefacto, ni evasión de rastreo, ni activación de la versión de pago.

## 5. Conclusión
El cargador es únicamente un canal de entrega y autoactualización. Si existe una versión de pago con licencia, su verificación es **exclusivamente servidor-cliente** (en el payload que el servidor sirve), y no puede "activarse" a nivel local: no hay mecanismo local que validar.

## 6. Anexo: análisis del payload servido (loader.php desde switzerland.goodwin.best)
Fecha: 2026-08-19. Descarga realizada a sandbox (curl, fuera del navegador). HTTP 200.
- **Tamaño**: 22.485.511 bytes (22,4 MB de texto, 0 saltos de línea). Encabezado con `?` inicial (artefacto del servidor / antedata anti-herramientas).
- **Ofuscación**: diferente a la del cargador. Array de strings `hqOj_n` (15.015 entradas, ~102 KB) + helper `TnJFHL(hqOj_n[0x..])` + rotación de array (`Ou7K3bM`), estilo javascript-obfuscator. El resto (22 MB) es código expandido que referencia el array.
- **Lógica de licencia/activación: NO EXISTE en el payload**. Al decodificar el array completo y buscar (licen, key, token, activ, subscri, premium, paid, trial, user, account, auth, expir, valid, serial, hwid, register, login, password, donat...): **0 coincidencias**. Los únicos "subscription"/"paid" en texto plano son del estado interno del juego (battle pass del juego y compras in-game), no del cheat.
- **Conclusión forense**: el servidor entrega actualmente el cheat COMPLETO a peticiones anónimas (HTTP 200) sin ningún control de licencia en el cliente. La supuesta "versión de pago" se controla, en todo caso, a nivel de **distribución servidor** (a quién se sirve), no con lógica local. No hay nada que "activar" en el cliente.
- **Comunicación del payload**: no referencia `goodwin` en absoluto; usa `fetch` (11) y `WebSocket` (2) hacia el juego, `localStorage` (11) para estado, `eval`/`new Function` (2+2) para código dinámico. El único componente que contacta al autor es el cargador (fetch a los mirrors en cada carga).
- **Nota de alcance**: análisis estático/documental. No se ejecutó el payload ni se construyó ninguna activación, parche ni versión "no rastreable".