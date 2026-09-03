# INFORME FINAL — Desofuscación de "HW Goodwin-5.1.txt"

## 1. Acceso al sistema de archivos (prueba)
Se comprobó acceso de lectura al directorio `C:\H` y se obtuvo el listado: contiene exactamente un archivo, `HW Goodwin-5.1.txt`, con los siguientes atributos:
- Tamaño: 39.759 bytes; 18 líneas; la línea 18 ocupa 34.953 caracteres.
- Contenido final: `})();` — estructura IIFE. El archivo **no fue modificado** (evidencia forense intacta).

## 2. Archivo encontrado
Es un **userscript de Tampermonkey** llamado **"HW Goodwin" v5.1** (autores: Goodwin & ZingerY), para el juego Hero Wars (Хроники Хаоса):
- `@match https://www.hero-wars.com/*` y `https://apps-1701433570146040.apps.fbsbx.com/*`
- `@run-at document-start`, `@grant none`
- `@updateURL`/`@downloadURL`: `https://switzerland.goodwin.best/updates/HW-Goodwin.user.js`
- Descripción: "Упрощает и автоматизирует многие аспекты игры Хроники Хаоса" (simplifica/automatiza el juego).
- Este archivo es el **cargador/actualizador**: descarga el código de cheat real desde 4 mirrors e lo inyecta en la página.

## 3. Técnicas de ofuscación (5 capas)
1. **LZ-String**: biblioteca integrada (compressToBase64/decompressFromBase64/compress/decompress/decompressFromUTF16).
2. **Diccionario comprimido**: 226 palabras separadas por `|`, empaquetadas con `decompressFromUTF16`; el acceso es `b(0xN)` → palabra N; `b(0x0)` = `"undefined"`.
3. **Base91 de múltiples alfabetos**: **20 alfabetos únicos de 91 caracteres**, uno por ámbito. El alfabeto global es: `|8~96/y()_,%{<.4!AMoW$v#LzighJrP`Z72@BTkNO=dsSV+R][C;EmtD">F0:wIcalf^xYeGuQ}35nX1&p?KHjqUb*`
4. **Matriz `f[]` de 234 entradas** (207 referencias `b(0x..)` + 27 literales base91). La posición en `f` **no coincide** con el índice `0xNN` (p. ej., `f[79]=b(0x3c)`).
5. **Caché perezosa compartida**: `e={}`; `function d(c){return typeof e[c]===b(0x0)?e[c]=decoder(f[c]):e[c]}`.
El algoritmo base91 fue reimplementado byte a byte y validado contra el obfuscador.

## 4. Mapa de identificadores
Se documentó en `identifier-map.json`. Resumen:
- `a`→LZString; `b`→diccionario; `c`/`d`→decoder/caché base91 global; `e`→caché compartida; `f`→strings codificados; `g`→globalThis.
- En el IIFE principal: `h`→`loader()`, `g()`→`injectScript()`, `aa`→`tryLoadMirror()`, `V`→mirrors rotados, `W`→índice del mirror, `X`→clave de caché `"https://goodwin.local/updates/loader.php"`, `Z`→mirror actual (`V[W]`), `ab`→URL del mirror, `ac`→AbortController, `ah`→chunks, `ap`→código descargado.
- Nota: el `Z` exterior (Last-Modified de la copia en caché) es **código muerto** (sombreado por el `Z` interno de `aa()`).

## 5. Cadenas decodificadas
Tabla completa índice→valor→confianza en **`string-map.json`** (índices 79–233; ejecutadas en sandbox = verdad, las demás confirmadas por alfabeto único y contexto). Mensajes clave:
- `"HW Goodwin script loader launched"`; `"HW Goodwin loaded from"`; `"HW Goodwin using a cached copy"`;
- `"HW Goodwin all mirrors are unavailable, searching for cache."`;
- `"HW Goodwin no mirrors available and no cache"`; `"HW Goodwin: mirror X failed or hung"`;
- Claves/nombres: caché `"HW-Goodwin-loader"`, `localStorage["HW Goodwin:usedDomain"]`, URL `https://goodwin.local/updates/loader.php`, `"application/javascript"`, `"Last-Modified"`, `"last-modified"`, `"no-cache"`, `"Content-Type"`, `"toUTCString"`.

## 6. Flujo de ejecución
1. Banner en consola.
2. **Hook 1**: `Object.defineProperty(Object.prototype,"game.model.GameModelStart",{ set(c){window.Game=this; setInterval(... si c.prototype.start, guarda window.gameStartRun, sustituye start por envoltorio que captura window.gameStartInst y guarda this["game.model.GameModelStart_"]=c, 1ms) }, get(){return this["game.model.GameModelStart_"]} })`.
3. **Hook 2**: `Object.defineProperty(Object.prototype,"__class__",{ set(c){ if(window.Game&&typeof c.j==="string"&&c.j!==""&&!window.Game[c.j]) window.Game[c.j]=c; this["__class_s_"]=c }, get(){return this["__class_s_"]} })` (registro de clases del juego por id).
4. `injectScript(c)`: `createElement("script")`, `textContent=c`, `(document.head||document.documentElement).appendChild(el)`.
5. `loader()`: abre caché, rota 4 mirrors [switzerland, hw-script, finland, russia].goodwin.best empezando por el último usado, calcula `X`.
6. `tryLoadMirror()`: si `W>=4` → usa copia en caché o `window.gameStartRun.call(window.gameStartInst)` (reinicio del juego, no `location.reload`). Sino: `fetch("https://<mirror>/updates/loader.php",{method:"GET",cache:"no-cache",signal})`, timeouts 5s (0x1388) y guard 3s (0xbb8), lee body por chunks, `blob.text()`, log, `localStorage.setItem`, inyecta el código, `cache.put` con Content-Type y Last-Modified; en error: log + `W++` + recursión.
7. `loader()` se invoca al final del IIFE.

## 7. Comportamiento de red y almacenamiento (observado en sandbox)
- 4 URLs de fetch (una por mirror): `https://switzerland.goodwin.best/updates/loader.php`, `https://hw-script.goodwin.best/updates/loader.php`, `https://finland.goodwin.best/updates/loader.php`, `https://russia.goodwin.best/updates/loader.php`.
- Escrituras: Cache Storage `"HW-Goodwin-loader"` (clave `https://goodwin.local/updates/loader.php`) y `localStorage["HW Goodwin:usedDomain"]`.
- Inyección de JavaScript remoto en el documento.
- Sin `eval`; un único `new Function` dentro de `findGlobalThis` (`new Function("return this")()`). Sin acceso a cookies.

## 8. Riesgos de seguridad
- **Ejecución de código remoto**: descarga JS arbitrario de 4 dominios de terceros y lo inyecta en la página; la copia se guarda en caché y se reutiliza si los mirrors fallan.
- Persistencia: Cache Storage + localStorage (rotación de mirrors).
- Monkey-patching de clases internas del juego (`prototype.start`, registro de clases por `__class__`).
- El servidor del distribuidor controla el código inyectado (riesgo de supply-chain). El propio script descargado (loader.php) no se analizó: no se ejecutó su contenido.

## 9. Archivos creados en `C:\H\deobfuscated\`
- `decode.js`, `decode2.js`: sandboxes (Proxy sobre `e`; modos fail/ok; logging incremental).
- `pure_decode.js`: decoder determinista (LZ-UTF16 + base91 + 20 alfabetos).
- `stringtable_fail.json`, `stringtable_ok.json`, `table_pure.json`: tablas ejecutadas/puras.
- `string-map.json`: **tabla definitiva completa** (índice→valor→confianza).
- `identifier-map.json`: mapa de identificadores.
- `deobfuscated.js`: **reconstrucción legible y ejecutable** (biblioteca LZ-String verbatim + datos reales + lógica legible).
- `DEOBFUSCATION_CHECKPOINT.md`: checkpoint persistente.
- Logs de evidencia: `consolelog_ok.log`, `reclog_ok.log`, `async_errors.log`.

## 10. Verificación
- La reconstrucción `deobfuscated.js` se ejecutó en sandbox con los mismos stubs y reprodujo **exactamente** el comportamiento original: banner, fetch al primer mirror con `{method:"GET",cache:"no-cache",signal}`, `localStorage.setItem("HW Goodwin:usedDomain", ...)`, `cache.put` con `Content-Type: application/javascript` y `Last-Modified`, y ambos hooks funcionando (`window.Game`, `gameStartRun`, `gameStartInst`, registro de clases).
- Cross-check puro vs. ejecutado: 92/99 iniciales; los 7 empates restantes resueltos y confirmados por decodificación independiente con todos los alfabetos (p. ej. 190→"abort", 123→"ent", 210→"from", 213→"ify", 223→"get", 106-108→"game.model.GameModelStart_", 148→"max").

## 11. Incertidumbres
- Índices 106–108, 121–123, 148–150, 160–161: no se ejecutaron en sandbox (rutas no tomadas), pero se confirmaron por alfabeto único y contexto sintáctico (confianza ALTA).
- El propósito exacto del reinicio `window.gameStartRun.call(window.gameStartInst)` es inferido del contexto (reinicio del arranque del juego) — no probado con el juego real.
- El contenido del script descargado (loader.php) es desconocido: **no se descargó ni ejecutó**.
- Índices 0–78 de `f[]` no se referencian nunca en el código (entradas muertas).

## 12. Próximos pasos sugeridos
1. Si se desea analizar el código de cheat real, descargar `https://<mirror>/updates/loader.php` y repetir el mismo proceso de sandbox (sin ejecutarlo en un navegador real).
2. Ampliar el sandbox con un DOM simulado más completo (jsdom) para ejercitar las ramas de caché (success y fallback) y el getter de `gameStartRun.call`.
3. Redactar un informe para el usuario final del juego sobre los riesgos de este script.

## 13. Notas forenses
- El original `C:\H\HW Goodwin-5.1.txt` permanece intacto (solo lectura de datos).
- El "cuelgue" inicial del modo OK fue un **bug del stub de prueba** (el reader devolvía `done:false` siempre), no un bucle infinito del payload; resuelto y documentado en el checkpoint.
- Toda decodificación se realizó con lógica reimplementada; el payload nunca se ejecutó fuera de la sandbox con stubs.

## 14. Conclusión
"HW Goodwin" es un cargador de cheat para Hero Wars que se autoactualiza descargando e inyectando código remoto desde mirrors propios, con persistencia en caché y localStorage, y que engancha internos del juego. El archivo analizado es legítimo en su propósito (cargar el cheat) pero de **riesgo alto** por la ejecución de código remoto no auditado.
