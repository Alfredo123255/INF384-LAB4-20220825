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
- Job y etapa:
- Problema que causa en el pipeline:
- Link al commit:
- Corrección (una o más: mover de etapa / dejar como informativo / eliminar / ajustar configuración / modificar tests):
- Evidencia (dato observado, con link al run o fragmento de la salida local):
### Defecto 3
- Job y etapa:
- Problema que causa en el pipeline:
- Link al commit:
- Corrección (una o más: mover de etapa / dejar como informativo / eliminar / ajustar configuración / modificar tests):
- Evidencia (dato observado, con link al run o fragmento de la salida local):
### Defecto 4
- Job y etapa:
- Problema que causa en el pipeline:
- Link al commit:
- Corrección (una o más: mover de etapa / dejar como informativo / eliminar / ajustar configuración / modificar tests):
- Evidencia (dato observado, con link al run o fragmento de la salida local):
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
