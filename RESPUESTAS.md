# Respuestas · Laboratorio 4
Alumno: [20220825]
Presupuesto: [ ] min  Métrica DORA: [ ]

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
- Link al commit:
- Corrección (una o más: mover de etapa / dejar como informativo / eliminar / ajustar configuración / modificar tests):
  Mover de etapa load para ejecutarce despues de e2e para que solo mida carga cuando e2e ya aprobo.
- Evidencia (dato observado, con link al run o fragmento de la salida local): el job e2e duró 46 s y terminó a las 4:20 PM (empezó ≈ 4:19:14 PM). El job load duró 12 min 9 s y terminó ≈ 4:31 PM, o sea que empezó ≈ 4:19 PM, mientras e2e todavía se ejecutaba. Los dos jobs corrieron al mismo tiempo contra la misma Function URL.
### Defecto 5
- Job y etapa:
- Problema que causa en el pipeline:
- Link al commit:
- Corrección (una o más: mover de etapa / dejar como informativo / eliminar / ajustar configuración / modificar tests):
- Evidencia (dato observado, con link al run o fragmento de la salida local):

## Parte 2 · Implementación
- Link al run en verde en main:
- Link al pull request:
- Defecto de la aplicación introducido (archivo y descripción):
- Job que debería detenerlo:
- Link al run fallido:

## Parte 3 · Decisión bajo restricción
Link al run medido:

| Job | Duración (min:seg) |
|---|---|
| unit | |
| integration | |
| sast | |
| sca | |
| build-push | |
| deploy | |
| smoke | |
| e2e | |
| load | |
| **Total** | |

Resultado de load en ese run:
| Métrica | Valor |
|---|---|
| p90 | |
| p95 | |
| Máximo | |
| Tasa de error | |
| Requests por segundo | |

Conclusión sobre el comportamiento bajo carga:

SLO propuesto (percentil, valor, ventana) y justificación:

SLA propuesto y diferencia con el SLO:

Cambios para cumplir el presupuesto:
| Cambio | Job | Minutos recuperados |
|---|---|---|

Total resultante: [ ] min
Link al run en main con los cambios:

Riesgo aceptado (defecto de la aplicación concreto y efecto en la métrica DORA asignada):

Decisión sobre sast (se queda / se mueve a otra etapa o evento / informativo) y razón:

## Declaración de uso de IA
Herramientas usadas y para qué:
