# Conclusiones sesión 2

## Parte 0

1.	Se auditaron 61 paquetes, de los cuales se detectaron 4 vulnerabilidades, 2 moderadas y 2 altas.
2.	Al realizar el “make test” en Python (pytest) pasan 82 pruebas, 0 con errores y se demoró 2,46 segundos. Y en TypeScript (vitest) pasan 28 pruebas y se demoró 1,54 segundos.
3.	Al correr el script que cambia el código los resultados son los siguientes Mutantes: 5  - muerto: 1 -  sobreviven: 4
4.	Al medir la cobertura del catalog el porcentaje de líneas es del 80% quedando 15 líneas sin cubrir.


## Parte 1

1. Cambió un archivo se realizaron 4 inserciones.
2. De los test pasaron 18 y 1 falló. Fallo el test para: Cubre el último día delturno.
3. Se sube la prueba a github.
4. De las 4 pruebas, hay 2 verdes, 1 gris y 1 roja. La que fallo "catalog-service" falla en la prueba "cubre el último día del turno".
5. Cambió la función isActiveOn() ahora es "<= shift.end_date" en vez de "< shift.end_date".
6. Se corren la pruebas y las 29 pasan.
7. Sew realiza el push y el pull request y todos los job ahora están en verde. 

## Parte 2

1. Se ejecuta el script para cambiar el código, los resultados son los siguientes Mutantes: 4 - muerto: 1 - sobreviven: 3. En comparación con la parte 0 hay 1 mutante menos y sobreviven 3 en vez de 4.
    ¿Mutantes sobreviven que significa para el CI? Significa que un cambio que produzca error o incorrecto en esas lineas llegaria a main con todas las pruebas en verde.
    ¿Que le pedirías al equipo dueño? Que elaboren pruebas mas concretas para los mutantes que sobreviven, es decir que hagan pruebas de casos bordes para que fallen con esos cambios.
2. Al medir la cobertura del catalog el porcentaje de lineas es del 80% quedando 15 líneas sin cubrir, no se ven cambios respecto de la parte 0 y es lo que se espera, dado que solo cambio < por <= en la misma línea. 
    ¿Está cubierto el código cuando el bug existía? Si, la linea ya la ejecutaban las pruebas antiguas. Todo estaba en verde con el bug dentro del código.
    Una línea cubierta ¿está verificada? No,cubierta significa que un test pasó por ahi y verificada significa que alg´un trest falla si esa linea se "rompe". Las pruebas antiguas ejecutaba isActiveOn, pero ninguna comprobaba el ultimo día del turno, justo donde estaba el error.
3. El total del servicio incidents es 925, falló en 215, por lo que cubre un 77%. El notificacions tiene un 70.3% de cobertura. 

## Parte 3


LA NOTA DEL PROYECTO ES C 71.1
1. km score: 
    Cognitive complexity score = 34 Grade: F--
    Duplication score = 91.9 Grade: A
    Identation Complexity score: 87 Grade: A-
    Halstead Effort score = 82.9 Grade: B
    File size score = 95.8 Grade: A+

2. km smells 
### Los tres archivos con más hallazgos (`km smells`)

| # | Archivo | Total | magic | long | param | Tipo dominante |
|---:|---|---:|---:|---:|---:|---|
| 1 | services/incidents/src/incidents/api.py | 79 | 78 | 1 | 0 | magic |
| 2 | services/catalog/src/app.ts | 23 | 22 | 1 | 0 | magic |
| 3 | services/incidents/src/incidents/repository.py | 14 | 3 | 0 | 11 | param |

Tipo de hallazgo más frecuente: **magic**, con 125 de 147 hallazgos (cerca del 85 %).                                147    125     5     17     0     0

3. 
### `km infra smells`: riesgo real o ruido

| Hallazgo | Veredicto | Justificación |
|---|---|---|
| `compose.yaml:9` y `:25`, `POSTGRES_PASSWORD` | Ruido (si es de desarrollo) | Es la base de datos local del compose y un valor tipo `postgres` no protege nada real. |
| `compose.yaml:57` y `:83`, `NOTIFICATIONS_API_KEY` | Riesgo real (si parece una clave verdadera) | Una API key en el repositorio queda en el historial para siempre y sirve contra cualquier entorno que la use. |
| `compose.yaml:84`, `ADMIN_TOKEN` | Riesgo real | Un token de administración filtrado da acceso privilegiado, aunque hoy sea de prueba. |
| `ci.yml:24, 40, 41, 50, 62, 63, 70, 80` (acciones oficiales de GitHub) | Ruido relativo | Son acciones oficiales de GitHub, con riesgo bajo de que alguien reescriba la etiqueta `v4`. |
| `ci.yml:25`, `astral-sh/setup-uv@v6` | Riesgo real | Es una acción de terceros: si se compromete la etiqueta, corre código ajeno en el CI con acceso a los secretos. |

## Parte 4

 1. 
 | Medición | Total | Moderadas | Altas |
|---|---:|---:|---:|
| `npm audit` (todas las dependencias) | 4 | 2 | 2 |
| `npm audit --omit=dev` (solo producción) | 3 | 2 | 1 |

| Paquete | Severidad | En producción | Tipo de vulnerabilidad |
|---|---|:---:|---|
| `zod` (< 4.6.6) | Alta | Sí | Denegación de servicio por agotamiento de memoria al validar colecciones enormes |
| `vite` (8.3.0 - 8.3.2) | Alta | No (solo desarrollo) | Ejecución de comandos, lectura de archivos fuera del permitido y ejecución de scripts entre orígenes |
| `hono` (<= 4.13.15) | Moderada | Sí | Evasión de middleware, inyección de HTML y validación o protección CSRF incompleta |
| `@hono/node-server` (2.0.0 - 2.1.5) | Moderada | Sí | Denegación de servicio y evasión de control de acceso en un proxy inverso |

2. Se ven las dependencias de incidents y se puede obervar que hay 4 vulnerabilidades en las dependencias certifi y opentelemetry-api.

3. Hay 38 vulnerabilidades en la biblioteca de GOñ. Todas tienen found in: @go1.23.12. El Fixed in mas alto es go1.26.11. Según lo consultado a Claude subir a esa versión corregiria las 38 vulnerabilidades. Además, hay 7 culnerabilidades en paquetes importados y 15 en módulos requeridos que el código no llama. 

4. Se escanearon 9 commits se encontraron 7 leaks.

5. 

| # | Hallazgo | Herramienta | Severidad | Decisión | Justificación |
|--:|---|---|---|---|---|
| 1 | `zod` < 4.6.6: agotamiento de memoria al validar colecciones enormes | `npm audit` | Alta | **Corregir** | Está en producción y valida las peticiones de catalog. Se actualiza a `zod@4.6.7` (Parte 4, paso 6). |
| 2 | `certifi` 2026.7.22 (PYSEC-2026-4320) | `pip-audit` | Sin dato | **Corregir** | Es una dependencia de producción de incidents. La versión 2026.10.18 lo arregla con `uv lock --upgrade-package`. |
| 3 | `opentelemetry-api` 1.45.0 (CVE-2026-114603 y CVE-2026-126360) | `pip-audit` | Sin dato | **Corregir** | Es una dependencia de producción. La versión 1.45.2 corrige los dos avisos. |
| 4 | `vite` 8.3.0 - 8.3.2: ejecución de comandos y lectura de archivos | `npm audit` | Alta | **Aceptar** | Solo aparece sin `--omit=dev`, o sea que es de desarrollo y no llega a la imagen. Se corrige después con `npm audit fix` (sin `--force`). |
| 5 | `hono` <= 4.13.15 y `@hono/node-server` 2.0.0 - 2.1.5 | `npm audit` | Moderada | **Posponer** hasta 2026-12-15 | Requieren `--force` y subir fuera del rango declarado, con riesgo de romper catalog. Antes hay que revisar si el servicio usa `mount()`, WebSockets o `csrf()`. |
| 6 | `tarea/03-secreto.patch`: `CATALOG_API_KEY` | `gitleaks` | Falso positivo | **Aceptar** | Es el parche que la tarea usa para probar el gate en la Parte 5 y contiene un secreto de mentira a propósito. |
| 7 | `NOTIFICATIONS_API_KEY` literal en `compose.yaml` (57, 83), `config.py:10` y `server.go:15` | `gitleaks` | Riesgo real | **Corregir** | Los cuatro hallazgos tienen la misma entropía, así que probablemente es una sola clave repetida. Hay que sacarla del código, leerla de una variable de entorno y rotarla, porque queda en el historial. |
| 8 | La misma clave en `services/notifications/README.md` (21 y 23) | `gitleaks` | Bajo | **Aceptar** | Es un ejemplo de `curl` en la documentación. Se acepta solo si el valor es de ejemplo, y debe ser el valor ficticio, no la clave real del punto 7. |
| 9 | Acciones sin fijar a un commit en `ci.yml` (9 hallazgos) | `km infra smells` | Moderado | **Posponer** hasta 2026-12-15 | Las de GitHub son de bajo riesgo. La de terceros, `astral-sh/setup-uv@v6`, se debe fijar primero. |
| 10 | 38 vulnerabilidades de la biblioteca estándar de Go en notifications (`net/http`, `crypto/x509`, `crypto/tls`, `net/url`, entre otras). Found in `go1.23.12`, Fixed in hasta `go1.26.11` | `govulncheck` | Alta (alcanzables desde `main.go` y `server.go`) | **Posponer** hasta 2026-12-15 | Se corrigen subiendo la versión de Go (toolchain e imagen base) tres versiones menores, no con un comando de dependencias. Eso exige reconstruir la imagen y volver a probar. Es un servidor HTTP, así que no conviene dejarlo mucho tiempo. |
| 11 | 7 vulnerabilidades en paquetes importados y 15 en módulos requeridos que el código no llama | `govulncheck` | Baja | **Aceptar** | `govulncheck` analiza las llamadas y dice que el código no alcanza esas funciones, así que no hay ruta de explotación. Se revisan al subir Go. |


6. Se corriegieron las vulverabilidades del catalog, de las 4 vulnerabilidades, quedan 3, se corrigió la de zod. De las pruebas 29 pasan sin problemas.

7. Se actualizaron los paquetes, al corren el pytest parsaron 82 sin incidentes.

8.

## Parte 5

_(pendiente)_

## Parte 6

_(pendiente)_
