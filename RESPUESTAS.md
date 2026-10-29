# Respuestas · Laboratorio 4
Alumno: [20220825]
Presupuesto: [11] min  Métrica DORA: [change failure rate]

## Parte 1 · Diagnóstico
### Defecto 1
- Job y etapa: El job con el defecto es "smoke" en la etapa deploy
- Problema que causa en el pipeline: smoke arranca apenas termina el job "built-push, en paralelo con el job "deploy", por lo que solo puede probar la versión anterior en Lamda pero no la imagen recien construida.
- Link al commit: https://github.com/Alfredo123255/INF384-LAB4-20220825/commit/628d8822783bcf9b78c4875905ccb76a790af342
- Corrección (una o más: mover de etapa / dejar como informativo / eliminar / ajustar configuración / modificar tests): Mover de etapa: smoke pasa de estar paralelo a la etapa deploy (needs: build-push) a la etapa post-deploy (needs: deploy), para que pruebe la versión recién desplegada.
Ajustar configuración: se agrega EXPECTED_COMMIT: ${{ github.sha }} al paso Smoke, para que smoke.sh compare el commit que responde /version con el commit del run.
- Evidencia (dato observado, con link al run o fragmento de la salida local): https://github.com/Alfredo123255/INF384-LAB4-20220825/actions/runs/40956515299/job/123738676838 el job smoke terminó en verde en 33 s. En el paso Smoke, el bloque env solo trae APP, AWS_REGION, SLO_P95, credenciales y BASE_URL; no incluye EXPECTED_COMMIT. El log solo muestra:
  salud: {"ok":true,"version":"1.0.0","commit":"49ba49ebee84f4c9f8b6943031595970311a3bb6"}
  saldo: {"usuario":"ana","saldo":1000}
  No aparece la línea "commit:", así que smoke.sh se saltó la verificación de la versión desplegada.
### Defecto 2
- Job y etapa: Job sca, etapa pre-build
- Problema que causa en el pipeline: Trivy se ejecuta con include dev-deps, asi que analiza las dependencias de desarrollo que no viajan en la imagen porque el dockerfile las omite por lo que el control esta probando algo que no se despliega por lo que si alguna de esas herramientas de DEV tiene una CVE alta el pipeline se detiene por una vulnerabilidad no alcanzable en producción y el reporte se llena de ruido.
- Link al commit: https://github.com/Alfredo123255/INF384-LAB4-20220825/commit/367cd1e5ccf6e00a8a8dbc3636e0b601196d7784
- Corrección (una o más: mover de etapa / dejar como informativo / eliminar / ajustar configuración / modificar tests):
- Eliminar --include-dev-deps para que sca solo analice dependencias de produccion
- Evidencia (dato observado, con link al run o fragmento de la salida local): en el run base https://github.com/Alfredo123255/INF384-LAB4-20220825/actions/runs/40956515299 el paso Trivy del job sca se ejecutó con "--include-dev-deps" y su Report Summary muestra:
  │ package-lock.json │ npm │ 0 │
  Es decir, analizó el lock file completo. En package-lock.json, [84] de 122 paquetes tienen "dev": true: el 69 % de lo que analiza sca no llega a la imagen.
### Defecto 3
- Job y etapa: Job integration etapa pre-build
- Problema que causa en el pipeline: los test de integracion asumen que la base arranca con los saldos semilla y dependen del orden en que se ejecutan, pero la base es un archivo que no se borra antes de correr. SI el archivo ya existe, por un reintento los test fallan aunque el codigo este bien.
- Link al commit: https://github.com/Alfredo123255/INF384-LAB4-20220825/commit/f112ce33282e3ed21bc8e206152189452ada9e44
- Corrección (una o más: mover de etapa / dejar como informativo / eliminar / ajustar configuración / modificar tests):
  BOrrar /tmp/wallet.db del job antes de ejecutar los test, para que cada corrida parta de los usuarios semilla
- Evidencia (dato observado, con link al run o fragmento de la salida local):en el run base https://github.com/Alfredo123255/INF384-LAB4-20220825/actions/runs/40956515299 el paso Trivy del job sca se ejecutó con "--include-dev-deps" y su Report Summary muestra:
  │ package-lock.json │ npm │ 0 │
  Es decir, analizó el lock file completo. En package-lock.json, [84] de 122 paquetes tienen "dev": true: el 69 % de lo que analiza sca no llega a la imagen.
### Defecto 4
- Job y etapa: job load, etapa post-deploy
- Problema que causa en el pipeline: load y e2e dependen de smoke, entonces corren en paralelo contra la misma funcion Lambda. Las transferencias de e2e mezclan con los 30 usuarios virtuales de k6: el p95 mide load queda contamidado por trafico que no es de la prueba de carga.
- Link al commit: https://github.com/Alfredo123255/INF384-LAB4-20220825/commit/c8447247017e961ddd82d4a57583c643c9091189
- Corrección (una o más: mover de etapa / dejar como informativo / eliminar / ajustar configuración / modificar tests):
  Mover de etapa load para ejecutarce despues de e2e para que solo mida carga cuando e2e ya aprobo.
- Evidencia (dato observado, con link al run o fragmento de la salida local): el job e2e duró 46 s y terminó a las 4:20 PM (empezó ≈ 4:19:14 PM). El job load duró 12 min 9 s y terminó ≈ 4:31 PM, o sea que empezó ≈ 4:19 PM, mientras e2e todavía se ejecutaba. Los dos jobs corrieron al mismo tiempo contra la misma Function URL.
### Defecto 5
- Job y etapa: smoke, e2e y load, etapa post deploy
- Problema que causa en el pipeline: Todos los runs, de pr y de main, despliegan la misma  funcion lambda. deploy esta serializado pero las pruebas posteriores no, el deploy del run nuevo reemplaza la imagen mientras estas 3 utilizan la de la version anterior. El resultado aprueba o reprueba la version anterior no la del run actual
- Link al commit: https://github.com/Alfredo123255/INF384-LAB4-20220825/commit/43e2be06e67f5a10a2f5992de569299d98e2aefc
- Corrección (una o más: mover de etapa / dejar como informativo / eliminar / ajustar configuración / modificar tests):
- ajustar configuración. Se agrega concurrency a nivel de workflow (group: ci-${{ github.repository }}, cancel-in-progress: false) para que un run espere a que termine el anterior antes de desplegar y probar la Lambda compartida.
- Evidencia (dato observado, con link al run o fragmento de la salida local): en el run #3 https://github.com/Alfredo123255/INF384-LAB4-20220825/actions/runs/40961930282 (disparado ≈16:57, duración total 14m 7s), el job load duró 12m 9s, de ≈16:59 a ≈17:11. El workflow no tiene concurrency, solo el job deploy la tiene: cualquier push a main o PR hecho en esos 12 minutos despliega otra imagen en la misma función Lambda mientras load sigue midiendo, y el resultado del run deja de corresponder a su propia versión.

## Parte 2 · Implementación
- Link al run en verde en main: https://github.com/Alfredo123255/INF384-LAB4-20220825/actions/runs/40965169878
- Link al pull request: https://github.com/Alfredo123255/INF384-LAB4-20220825/pull/1
- Defecto de la aplicación introducido (archivo y descripción): src/wallet.js, línea 54. Se cambió la condición "cuentaOrigen.saldo < centimos" por "cuentaOrigen.saldo <= centimos". Ahora, cuando un usuario intenta transferir exactamente todo su saldo
- Job que debería detenerlo: unit. El test "transferir todo el saldo deja el origen en cero" (test/unit/wallet.test.js) transfiere el saldo exacto y espera que el origen quede en 0. También falla integration
- Link al run fallido: https://github.com/Alfredo123255/INF384-LAB4-20220825/actions/runs/40966341432

## Parte 3 · Decisión bajo restricción
Link al run medido: https://github.com/Alfredo123255/INF384-LAB4-20220825/actions/runs/40965169878

| Job | Duración (min:seg) |
|---|---|
| unit | 0:11 |
| integration | 0:17 |
| sast | 0:20|
| sca | 0:22|
| build-push | 0:48|
| deploy | 0:20|
| smoke | 0:13|
| e2e | 0:42|
| load | 12:11|
| **Total** | 15:24|

Resultado de load en ese run:
| Métrica | Valor |
|---|---|
| p90 | 23.05 ms |
| p95 | 25.29 ms|
| Máximo | 120.68 ms|
| Tasa de error | 0%|
| Requests por segundo | 53.10|

Conclusión sobre el comportamiento bajo carga: con 30 usuarios virtuales durante 12 minutos, Monedero Andino respondio de forma estable y rapida. El 95% de los requests tardaron menos de 25.29 ms y no hubo errores. l 95 % de los requests tardó menos de 25,29 ms y no hubo errores. La diferencia entre la mediana (17,59 ms) y el p95 (25,29 ms) es pequeña, así que la latencia es consistente. El máximo de 120,68 ms es un caso aislado, probablemente un arranque en frío de una instancia nueva de Lambda. Con esta carga el sistema está muy lejos del umbral actual de 500 ms.

SLO propuesto (percentil, valor, ventana) y justificación: p95 < 50 ms, en una ventana de 28 días. El p95 medido fue 25,29 ms; 50 ms deja un margen de aproximadamente el doble para que load no falle por la variación normal entre runs, y aun así detecta si la latencia se duplica. Se configura SLO_P95: 50 en ci.yml.

SLA propuesto y diferencia con el SLO: 99,5 % de disponibilidad mensual y p95 < 200 ms al mes; si no se cumple, el cliente recibe un crédito. El SLA es un compromiso contractual con el cliente; el SLO es la meta interna del equipo. El SLO es más exigente (50 ms frente a 200 ms) para detectar y corregir problemas antes de incumplir el SLA.

Cambios para cumplir el presupuesto:
| Cambio | Job | Minutos recuperados |
| Stages de test/load/transferir.js: rampa de subida de 1m a 30s, carga constante de 10m a 4m  y rampa de bajada de 1m a 30s. Load pasa de 12 a 5 minutos. | load | ≈ 7:00 |
| SLO_P95 de 500 a 50 ms en ci.yml (no cambia la duración; ajusta el umbral que evalúa load). | load | 0:00 |


Total resultante: [ ] min
Link al run en main con los cambios: https://github.com/Alfredo123255/INF384-LAB4-20220825/actions/runs/40969359202

Riesgo aceptado (defecto de la aplicación concreto y efecto en la métrica DORA asignada): al reducir la carga constante de 10 a 4 minutos, el pipeline deja de detectar defectos que solo aparecen con carga sostenida. Ejemplo: si en src/db.js se quita el LIMIT 50 de la consulta de movimientos, /historial se vuelve más lento a medida que crece la tabla. En 4 minutos la tabla no crece lo suficiente para superar el SLO, el cambio pasa el pipeline y la latencia se degrada recién en producción. Change failure rate SUBE: más despliegues que pasan el pipeline terminan causando una falla en producción.

Decisión sobre sast (se queda / se mueve a otra etapa o evento / informativo) y razón: se queda en pre-build. Cuesta 0:20 de los 11 min (aproximadamente el 2 %) y corre en paralelo con unit, integration y sca, así que no retrasa el despliegue. Detiene antes de construir patrones inseguros, como una inyección SQL por concatenación en src/db.js. Si se quitara o pasara a informativo, esos cambios llegarían a producción y change failure rate subiría. Ahorrar 20 segundos no justifica empeorar la métrica asignada.

## Declaración de uso de IA
Herramientas usadas y para qué:
