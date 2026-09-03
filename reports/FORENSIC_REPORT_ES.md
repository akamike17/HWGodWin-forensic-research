# Informe forense — HW Goodwin / Hero Wars

**Fecha del análisis:** 2026-08-19  
**Tipo:** investigación técnica y forense de software de automatización/trampa.

## Resumen

La investigación analizó dos componentes: el userscript cargador HW Goodwin 5.1 y el payload remoto `loader.php`. El cargador obtiene e inyecta JavaScript desde infraestructura `*.goodwin.best`; el payload contiene la funcionalidad de automatización y utiliza endpoints remotos bajo `/request/*.php`.

## Método

El análisis combinó inspección estática, decodificación de tablas de strings e instrumentación en sandbox. Las operaciones de red se sustituyeron por stubs para registrar URL, método y estructura de las llamadas sin transmitir solicitudes desde el sandbox durante esas pruebas.

## Hallazgos principales

- El cargador rota entre mirrors `switzerland`, `hw-script`, `finland` y `russia` bajo `.goodwin.best`.
- El payload construye solicitudes POST hacia `/request/<endpoint>.php`.
- Se observaron rutas asociadas con autenticación, activación, estadísticas, quiz, invasión y carga de datos/replays.
- La funcionalidad documentada por el propio payload incluye automatización de acciones del juego y características relacionadas con batallas y timing.
- La arquitectura de distribución permite al servidor remoto controlar el JavaScript que finalmente se inyecta en la sesión del juego, lo que constituye una superficie de riesgo de cadena de suministro.

## Evidencia de red

La instrumentación identificó endpoints tales como `auth`, `activateCode`, `upload/userData`, `upload/asgardReplay`, `upload/EBR`, `quiz/findAnswer`, `rating/getData` e `invasion/battle`, entre otros. El repositorio conserva un listado de observaciones de red y documentación técnica separada.

## Integridad

El paquete original de investigación utilizó SHA-256 por artefacto y una firma RSA-4096 del manifiesto. La clave privada utilizada para producir la firma no se publica. La clave pública y el procedimiento de verificación pueden publicarse independientemente para permitir comprobación de procedencia.

## Limitaciones

Las conclusiones se limitan a las muestras y al entorno documentados. Los resultados de sandbox demuestran el comportamiento observado bajo instrumentación; no deben presentarse como evidencia de acciones no observadas directamente en producción.

## Publicación responsable

El repositorio público conserva informes, hashes, metodología y evidencia no sensible. No redistribuye la clave privada ni el payload operacional reconstruido completo.
