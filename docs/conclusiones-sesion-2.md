# Conclusiones — Sesión 2

janobox@DESKTOP-IM3TV61:~/proyectos/taller-CEDCCDOFA01$ cat docs/conclusiones-sesion-2.md
## Parte 0. Preparación

### Pruebas iniciales

Se ejecutó `make test` desde el contenedor Docker de herramientas, obteniendo los siguientes resultados:

| Servicio | Pruebas aprobadas | Pruebas fallidas | Duración |
|---|---:|---:|---:|
| incidents | 82 | 0 | 2,76 s |
| catalog | 28 | 0 | 671 ms |
| notifications | No desglosadas | 0 | 0,007 s |

Los tres servicios completaron sus pruebas sin fallos.

### Pruebas de mutación

Se ejecutó `tarea/mutar.sh services/catalog/src/shifts.ts` con los siguientes resultados:

- Mutantes generados: 5
- Mutantes muertos: 1
- Mutantes sobrevivientes: 4
- Puntaje de mutación: 20 %

### Cobertura de catalog

Se ejecutó `npm run coverage` y se obtuvieron los siguientes resultados para `shifts.ts`:

- Cobertura de sentencias: 80 %
- Cobertura de ramas: 100 %
- Cobertura de funciones: 80 %
- Cobertura de líneas: 80 %
- Línea sin cubrir: 15

La cobertura global de líneas de catalog fue de 66,14 %.

### Observación inicial

Aunque todas las pruebas pasan y `shifts.ts` alcanza un 100 % de cobertura de ramas, el puntaje de mutación es de solo 20 %. Esto evidencia que ejecutar código durante una prueba no implica necesariamente detectar todos los errores o cambios en su comportamiento.

Estos valores se utilizarán como línea base para comparar los resultados posteriores a la incorporación de la prueba y la corrección del bug.


## Parte 1. Un bug, de rojo a verde

### Incorporación de una prueba para detectar el bug

Se aplicó el parche `01-prueba-que-falla.patch`, que modifica `services/catalog/tests/shifts.test.ts` agregando cuatro líneas para comprobar que el último día de un turno está incluido en su período de vigencia.

### Resultados de las pruebas antes de la corrección

| Indicador | Parte 0 | Parte 1 (nueva prueba) |
|---|---:|---:|
| Pruebas aprobadas | 28 | 28 |
| Pruebas fallidas | 0 | 1 |
| Total de pruebas | 28 | 29 |
| Duración | 671 ms | 665 ms |

La prueba `isActiveOn > cubre el último día del turno` falló porque la función devolvió `false` cuando se esperaba `true` para la fecha `2026-10-31`.

El resultado demuestra que existía un error funcional que no era detectado por las pruebas originales. La incorporación de una prueba adicional permitió evidenciar el defecto.

El estado rojo no representa un retroceso en la calidad de las pruebas, sino una mejora en su capacidad para detectar un comportamiento incorrecto.


### Resultados después de la corrección

Se aplicó el parche `02-correccion.patch`, modificando la función `isActiveOn` del archivo `services/catalog/src/shifts.ts`.

La corrección reemplazó la comparación estricta `day < shift.end_date` por `day <= shift.end_date`, incluyendo el último día del turno dentro de su período de vigencia.

Al ejecutar nuevamente `npm test`, se obtuvieron los siguientes resultados:

- Pruebas aprobadas: 29.
- Pruebas fallidas: 0.
- Duración: 957 ms.

Antes de la corrección, 28 pruebas pasaban y una fallaba. Después de la corrección, las 29 pruebas pasan.

**Conclusión:** un CI verde significa que las verificaciones configuradas han pasado, pero no garantiza la ausencia de errores. Antes de incorporar la nueva prueba, el código contenía un defecto funcional que no era detectado.

Separar la prueba y la corrección en commits independientes permitió demostrar primero la existencia del error y posteriormente comprobar que la modificación lo resolvió.


## Parte 2. ¿Las pruebas detectan cambios?

### Resultados de las pruebas de mutación

Se repitió el análisis de mutación sobre `services/catalog/src/shifts.ts`, luego de incorporar la nueva prueba y corregir el error.

| Indicador | Antes | Después | Variación |
|---|---:|---:|---:|
| Mutantes generados | 5 | 4 | -1 |
| Mutantes muertos | 1 | 1 | 0 |
| Mutantes sobrevivientes | 4 | 3 | -1 |
| Puntaje de mutación | 20 % | 25 % | +5 puntos porcentuales |

Si bien el puntaje aumentó, no se incrementó la cantidad absoluta de mutantes detectados. La mejora porcentual también responde a una reducción en el total de mutantes generados.

### Cobertura después de la corrección

La cobertura de `shifts.ts` permaneció sin cambios:

- Sentencias: 80 %.
- Ramas: 100 %.
- Funciones: 80 %.
- Líneas: 80 %.
- Línea sin cubrir: 15.

La cobertura global de líneas de catalog fue de 66,14 %, igual que antes de la corrección.

### Cobertura de los otros servicios

| Servicio | Cobertura total |
|---|---:|
| incidents | 77 % |
| notifications | 70,3 % |

En incidents se identificaron módulos con baja cobertura, entre ellos `main.py` (0 %), `repository.py` (32 %) y `clients.py` (40 %).

### Interpretación

**¿Cambió la cobertura?**

No. La cobertura de `shifts.ts` permaneció en 80 % de líneas y 100 % de ramas, a pesar de que se incorporó una prueba que permitió detectar un error funcional previamente ignorado.

**¿Estaba cubierto el código del bug cuando existía?**

Sí. La lógica defectuosa se encontraba en una línea ejecutada por las pruebas existentes. Sin embargo, estas no comprobaban adecuadamente el comportamiento en el límite final del turno.

**¿Una línea cubierta está verificada?**

No necesariamente. La cobertura indica que una línea se ejecutó durante las pruebas, pero no garantiza que existan aserciones capaces de detectar todos sus comportamientos incorrectos.

**¿Qué significan los mutantes sobrevivientes?**

Representan modificaciones que no provocaron fallos en las pruebas ejecutadas. Esto puede señalar escenarios no verificados o limitaciones en la evaluación.

Se observó una limitación concreta en el script `mutar.sh`: al existir dos operadores `<=` en una misma línea, modifica solamente la primera coincidencia. Por ello, el mutante sobreviviente de la línea 8 modifica el límite inicial del turno y no el límite final.

**¿Qué solicitaría al equipo?**

Incorporar pruebas explícitas para los límites inicial y final de los turnos, mejorar los casos de solapamiento y revisar los mutantes sobrevivientes. También sería conveniente mejorar el script de mutación para evaluar cada aparición de un operador de manera independiente.

### Conclusión

Las pruebas de cobertura y mutación entregan información complementaria. La nueva prueba demostró un error real y permitió verificar su corrección, aunque la cobertura de código no aumentó. Esto confirma que la calidad de una batería de pruebas depende tanto de los escenarios y aserciones utilizados como de las métricas cuantitativas.


## Parte 3. Análisis estático

### Resultados de `km score`

El proyecto obtuvo una calificación **C (71,1 puntos)**, sobre 23 archivos y 2.679 líneas de código.

La dimensión con menor puntuación fue la complejidad cognitiva, con 34,0 puntos y calificación F--. Esta dimensión representa el 32 % de la evaluación del proyecto.

Los tres archivos con peor puntuación fueron:

| Archivo | Puntaje | Calificación |
|---|---:|---|
| services/incidents/src/incidents/api.py | 42,1 | F- |
| services/notifications/server.go | 51,8 | F |
| services/catalog/src/app.ts | 52,6 | F |

La complejidad cognitiva constituye la principal oportunidad de mejora del proyecto, ya que puede dificultar la comprensión, las pruebas y el mantenimiento.

### Resultados de `km smells`

Se identificaron 147 hallazgos en 10 archivos:

- 125 números mágicos.
- 17 problemas de parámetros.
- 5 funciones largas.
- 0 TODO.
- 0 hallazgos de comentarios.

El tipo más frecuente fue `magic`, que representa aproximadamente el 85 % de los hallazgos.

El archivo `services/incidents/src/incidents/api.py` concentró 79 hallazgos, seguido por `services/catalog/src/app.ts`, con 23.

### Resultados de `km infra smells`

Se identificaron 14 hallazgos en dos archivos:

- 5 secretos escritos directamente en `compose.yaml`.
- 9 referencias a GitHub Actions sin fijación mediante SHA en `.github/workflows/ci.yml`.

#### Evaluación de riesgos

| Hallazgo | Evaluación | Justificación |
|---|---|---|
| Contraseñas PostgreSQL literales | Riesgo condicionado | Pueden corresponder al entorno local del taller, pero no deberían utilizarse en producción. |
| API key de notifications | Riesgo real si está activa | Su exposición podría permitir accesos no autorizados. |
| Token administrativo literal | Riesgo potencialmente alto | Podría facilitar el uso indebido de funciones administrativas si es válido. |
| GitHub Actions sin SHA fijo | Riesgo preventivo real | Utilizar etiquetas de versión no garantiza la inmutabilidad del código ejecutado por el pipeline. |

### Conclusión

El análisis estático entregó información que no aparece en las pruebas funcionales.

Aunque los servicios pasan sus pruebas, existen problemas de mantenibilidad, especialmente relacionados con la complejidad cognitiva y el uso de números mágicos.

Además, los controles de infraestructura identificaron posibles exposiciones de credenciales y riesgos de cadena de suministro.

Los hallazgos requieren triaje según su contexto y severidad. No todos implican vulnerabilidades explotables, pero tampoco deben descartarse simplemente porque el CI esté en verde.



## Parte 4. Dependencias y secretos

### Auditoría de catalog

Se ejecutaron `npm audit` y `npm audit --omit=dev`.

La auditoría completa detectó 4 vulnerabilidades: 2 altas y 2 moderadas.

Al excluir dependencias de desarrollo, permanecieron 3 vulnerabilidades: 1 alta (`zod`) y 2 moderadas (`hono` y `@hono/node-server`).

La vulnerabilidad alta de `vite` pertenece a dependencias de desarrollo y no aparece en la auditoría de producción.

### Auditoría de incidents

Se ejecutó `pip-audit` utilizando las dependencias exportadas por `uv`.

La herramienta reportó 4 entradas en 2 paquetes:

- `certifi 2026.7.22`: identificador PYSEC-2026-4320, reportado dos veces. Versión corregida: 2026.10.18.
- `opentelemetry-api 1.45.0`: CVE-2026-114603 y CVE-2026-126360. Versiones corregidas: 1.45.1 y 1.45.2, respectivamente.

Se identificó un registro duplicado para `certifi`, lo que debe considerarse al interpretar el número de vulnerabilidades.

### Auditoría de notifications

`govulncheck` reportó 37 vulnerabilidades que afectan rutas alcanzables desde el código, asociadas a la biblioteca estándar de Go 1.23.12.

Adicionalmente, se encontraron 8 vulnerabilidades en paquetes importados y 15 en módulos requeridos que no parecen ser alcanzables desde el código analizado.

La resolución requiere evaluar la actualización de la versión de Go utilizada para compilar y ejecutar el servicio.

A diferencia de las dependencias administradas por npm o uv, los problemas de la biblioteca estándar no se corrigen simplemente actualizando un paquete individual del proyecto.

### Detección de secretos

Se ejecutó `gitleaks detect --redact -v`.

Se analizaron 8 commits y se identificaron 7 hallazgos en los siguientes archivos:

- `tarea/03-secreto.patch`: 1.
- `services/notifications/README.md`: 2.
- `services/incidents/src/incidents/config.py`: 1.
- `compose.yaml`: 2.
- `services/notifications/server.go`: 1.

El hallazgo en `tarea/03-secreto.patch` corresponde a una credencial ficticia preparada para comprobar el funcionamiento del gate.

Los demás hallazgos deben evaluarse antes de considerarlos secretos operativos.

### Tabla de triaje

| Hallazgo | Decisión | Justificación o fecha |
|---|---|---|
| Vulnerabilidad alta en zod | Corregir | Actualización a 4.6.7. |
| Vulnerabilidad en certifi | Corregir | Actualización a versión corregida mediante uv. |
| Vulnerabilidades en opentelemetry-api | Corregir | Actualización a versión corregida mediante uv. |
| Vulnerabilidades moderadas en Hono | Posponer | Revisar impacto y compatibilidad antes del 30-11-2026. |
| Vulnerabilidades de la biblioteca estándar Go | Posponer | Evaluar actualización del compilador, imágenes y CI antes del 30-11-2026. |
| Credencial ficticia en parche del taller | Aceptar | Forma parte del material de prueba y no constituye, por sí misma, una credencial operativa. |

### Hallazgo más difícil de decidir

El caso de Go fue el más complejo debido a la cantidad de vulnerabilidades detectadas y a que su corrección implica actualizar el compilador o runtime, reconstruir imágenes y verificar compatibilidad.

No se considera apropiado simplemente desactivar el control para obtener un CI verde.

### Diferencia entre detección y exposición real

Los resultados de las herramientas no implican automáticamente explotación. Es necesario distinguir las dependencias utilizadas en producción, la alcanzabilidad de las funciones vulnerables y la existencia de credenciales operativas.

Los controles automatizados son fundamentales, pero deben complementarse con un triaje contextualizado.


### Corrección de dependencias y validación posterior

**Catalog**

Se actualizó la dependencia `zod` a la versión `4.6.7` utilizando `npm install --save-exact`.

Antes de la actualización, `npm audit --omit=dev` reportaba tres vulnerabilidades: una alta y dos moderadas.

Después de la actualización, el mismo comando reportó dos vulnerabilidades moderadas, correspondientes a `hono` y `@hono/node-server`. La vulnerabilidad alta de producción quedó resuelta.

La ejecución de `npm test` confirmó que las 29 pruebas de catalog continúan pasando.

**Incidents**

Se ejecutó `uv lock --upgrade-package certifi --upgrade-package opentelemetry-api`, obteniendo las siguientes actualizaciones:

- `certifi`: de `2026.7.22` a `2026.11.20`.
- `opentelemetry-api`: de `1.45.0` a `1.45.2`.

Se ejecutó `uv run pytest` después de la actualización y las 82 pruebas finalizaron correctamente.

La exportación posterior mediante `uv export --frozen` confirmó que el archivo de bloqueo contiene las nuevas versiones.

**Auditoría posterior de Python:** `pip-audit` reportó `No known vulnerabilities found` después de actualizar las dependencias. Las 82 pruebas de incidents continuaron aprobando.

### Interpretación

Las actualizaciones permitieron resolver vulnerabilidades conocidas sin provocar regresiones en las pruebas existentes.

No obstante, la ausencia de fallos en las pruebas funcionales no demuestra por sí sola que todas las vulnerabilidades hayan desaparecido. Por esa razón, es necesario repetir las auditorías de seguridad después de cada actualización.


## Parte 5. El gate en el CI

### Prueba del gate de calidad

Se creó la rama `prueba-gate` y se aplicó el parche `03-secreto.patch`, que incorporó una credencial literal `CATALOG_API_KEY` en `compose.yaml`.

El Pull Request #3 ejecutó seis controles: cinco finalizaron correctamente y uno falló.

El job `calidad-y-seguridad` falló en el paso `Seguridad de la infraestructura`, ejecutando `km infra smells --since-ref "$BASE" --fail-on hard-coded-secret,empty-password`.

Se detectaron seis secretos literales en `compose.yaml`: cinco preexistentes y uno incorporado por el parche.

Esto demuestra que el gate inspecciona el archivo completo cuando ha sido modificado, no únicamente las líneas agregadas.

El job `construye-imagenes` finalizó correctamente, al igual que los cuatro jobs del CI original.

La detección de secretos interrumpió el job antes de ejecutar las auditorías de dependencias y Gitleaks.

Después de revertir el parche en otro commit, el archivo `compose.yaml` volvió a su contenido anterior. Sin embargo, en la nueva ejecución del PR #3 el job `calidad-y-seguridad` continuó en rojo, esta vez en el paso `Secretos` (Gitleaks), debido a que el secreto agregado aún existía en el historial de la rama. El job `construye-imagenes` siguió en verde. El PR #3 se cerró sin hacer merge.

El experimento evidencia que un CI funcional en verde no garantiza que una modificación cumpla los controles de seguridad. El analizador de infraestructura inspecciona archivos modificados completos (6 secretos: 5 preexistentes y 1 nuevo), mientras Gitleaks inspecciona los commits incluidos en el PR. Si el secreto fuera real, habría que revocarlo o rotarlo de inmediato y evaluar la limpieza segura del historial, coordinando los cambios con los colaboradores; una simple reversión no lo elimina del historial.

### Controles que bloquean y controles informativos

El gate bloquea deterioro de calidad en archivos cambiados (`km score`), secretos o contraseñas vacías en infraestructura modificada (`km infra smells`), vulnerabilidades altas o críticas de dependencias productivas de catalog (`npm audit`), vulnerabilidades de incidents (`pip-audit`), secretos en nuevos commits (`gitleaks`) y errores al construir las imágenes Docker. `km smells` informa hallazgos sin bloquear por sí mismo. El análisis de notifications (`govulncheck`) incluye `continue-on-error: true`, por lo que sus vulnerabilidades no bloquean el merge.

Ese tratamiento de notifications solo es defendible como una excepción temporal y documentada para el taller. En un entorno real debería existir responsable, fecha límite de corrección y una política de excepciones revisable, dado que las vulnerabilidades reportadas afectan rutas alcanzables de la biblioteca estándar de Go.

Un problema que se podría escapar es un error funcional en un caso límite no cubierto por las pruebas, como ocurrió inicialmente con el último día del turno. También pueden pasar hallazgos moderados de producción de catalog, porque el umbral bloqueante es `high`.


## Parte 6. Contenedores

### Construcción de imágenes

Se ejecutaron los comandos `docker build` correspondientes a los tres servicios desde Ubuntu/WSL, utilizando Docker Desktop.

Las tres construcciones finalizaron correctamente.

### Tamaño de las imágenes

| Imagen | Uso en disco | Tamaño del contenido |
|---|---:|---:|
| pager-incidents | 266 MB | 62,5 MB |
| pager-catalog | 252 MB | 62,2 MB |
| pager-notifications | 14,7 MB | 3,31 MB |

### Comparación

La diferencia de tamaño se explica principalmente por los entornos de ejecución incluidos en cada imagen.

`pager-incidents` incluye el entorno Python y sus dependencias, mientras que `pager-catalog` incorpora Node.js y dependencias de producción.

`pager-notifications` utiliza compilación multietapa con Go y una imagen final Distroless, lo que permite conservar un ejecutable compilado sin incluir el entorno completo de compilación.

La imagen de notifications utiliza considerablemente menos espacio que las otras dos.

### Advertencia de construcción

Docker reportó `JSONArgsRecommended` para `pager-incidents`, indicando que sería preferible utilizar la forma JSON de la instrucción `CMD` para mejorar el tratamiento de señales del sistema.

La advertencia no impidió completar la construcción.

### Revisión de reproducibilidad y ejecución

Se comprobó que el Dockerfile de `pager-incidents` utiliza una compilación multietapa. En la etapa de construcción declara `FROM python:3.12-slim AS builder` y copia la herramienta uv con `COPY --from=ghcr.io/astral-sh/uv:0.7 /uv /bin/uv`. La etapa final utiliza `FROM python:3.12-slim` y copia el resultado con `COPY --from=builder /app /app`. Las referencias fijan versiones mayores o menores, pero no fijan todos los componentes a digests inmutables: la reproducibilidad es parcial.

`docker history pager-incidents` mostró `USER nobody`, que ejecuta la aplicación sin privilegios de root. No se apreciaron credenciales en el historial de capas mostrado, aunque esto no prueba la ausencia absoluta de secretos dentro de la imagen. Se confirmó la presencia de `services/incidents/.dockerignore`. Docker también mostró la advertencia `JSONArgsRecommended` para `CMD`, que sugiere usar la forma JSON por el manejo de señales del proceso.

Se ejecutó **una misma imagen, sin reconstrucción**, como `pager-a` con `-p 9001:8080` y como `pager-b` con `-e PORT=9090 -p 9002:9090`. Los endpoints `http://localhost:9001/health` y `http://localhost:9002/health` respondieron correctamente, de acuerdo con las verificaciones locales. Se detuvieron ambos contenedores usando `docker stop pager-a pager-b` (verificar en la terminal local si aún no se ha ejecutado).

El job `construye-imagenes` del workflow completó correctamente las construcciones de las tres imágenes. Reutilizar la misma imagen con distinta configuración permite promover el mismo artefacto entre entornos y disminuir diferencias entre desarrollo y producción.

## Registro de IA

Se utilizó un asistente de IA como apoyo para ordenar el trabajo en WSL/Docker, interpretar los reportes de pruebas, cobertura, mutación y análisis estático, redactar borradores de conclusiones y verificar la secuencia de Git y GitHub Actions.

- **Propuestas aceptadas:** separar los commits de prueba, corrección, dependencias y gate; registrar resultados reales; comparar cobertura con mutación; reproducir el fallo del gate con el parche previsto por el taller; documentar el triaje.
- **Propuestas corregidas:** se precisó que el aumento de 20 % a 25 % en el puntaje de mutación no implica más mutantes muertos, pues ambos análisis mataron uno; el conjunto de mutantes cambió de 5 a 4. Se aclaró también que el primer operador `<=` de la línea 8 es el que modifica el script, y no necesariamente el límite de término del turno.
- **Propuestas rechazadas:** no ejecutar `npm audit fix --force` ni desactivar controles para obtener verde, porque introduciría cambios fuera del alcance o silenciaría riesgos sin resolverlos. Tampoco interpretar el CI verde como garantía de ausencia de defectos.

La información incorporada al informe se contrastó con las salidas reales de las herramientas; el asistente no ejecutó las pruebas locales ni sustituyó la evidencia obtenida por el estudiante.

## Conclusiones finales

### 1. ¿Qué dice la distancia entre «las pruebas pasan» y «las pruebas detectan»?

En la línea base pasaban 28 pruebas de catalog, pero faltaba la comprobación del último día de turno. Al agregarla, hubo 28 pruebas aprobadas y una fallida; luego de corregir la comparación de fechas, pasaron las 29. A la vez, `shifts.ts` mantuvo 80 % de cobertura de líneas y 100 % de ramas antes y después; el puntaje de mutación pasó de 20 % a 25 % porque cambió el total de mutantes, mientras la cantidad de mutantes muertos siguió siendo uno. Por tanto, pasar las pruebas mide conformidad con los casos existentes, no exhaustividad del comportamiento ni fuerza de sus aserciones.

### 2. ¿Qué control dio más información y cuál menos?

La combinación de la nueva prueba del límite del turno y el análisis de mutación fue la más reveladora, porque permitió evidenciar un defecto real y mostrar que la cobertura elevada no implica necesariamente capacidad de detección. Como métrica aislada, la cobertura de líneas fue la menos concluyente para este defecto: permaneció idéntica incluso al incorporar una prueba que detectó el error. No significa que la cobertura carezca de utilidad, sino que necesita complementarse con pruebas de límites y validaciones de comportamiento.

En materia de seguridad, el contraste entre `km infra smells` y Gitleaks también fue valioso: el primero reportó seis secretos al analizar el archivo modificado; tras revertir el cambio, el segundo siguió detectando el secreto introducido en el historial de la rama.

### 3. Si hubiera una hora más, ¿qué agregaría al pipeline y por qué?

Agregaría primero una verificación automática y acotada de casos límite del dominio (inicio y fin inclusivos de los turnos, días inmediatamente anteriores y posteriores y turnos superpuestos) junto con una medición de mutación más completa que pruebe por separado cada aparición del operador. Esto ataca directamente la brecha observada: con 100 % de ramas cubiertas, el bug existía y varios mutantes sobrevivían. Como siguiente prioridad, dejaría planificada la actualización de Go y la eliminación controlada de la excepción de `govulncheck`, verificando luego la seguridad de las imágenes reconstruidas.


## Registro de uso de IA

Durante el desarrollo de la tarea utilicé ChatGPT como asistente de apoyo técnico y aprendizaje, principalmente para comprender los procedimientos, ejecutar las actividades en el orden solicitado e interpretar los resultados obtenidos.

El trabajo se realizó de manera interactiva: ejecuté personalmente los comandos en Ubuntu/WSL y Docker Desktop, compartí las salidas de terminal y los resultados de GitHub Actions, y utilicé el asistente para resolver dudas, validar lo observado y comprender las implicancias técnicas de cada actividad.

### Propuestas aceptadas

- **Orientación paso a paso:** utilicé el asistente para organizar la ejecución de la tarea según el enunciado, diferenciando los comandos que debían ejecutarse en WSL de aquellos correspondientes al contenedor de herramientas.
- **Resolución de problemas del entorno:** recibí orientación para identificar que inicialmente estaba utilizando la distribución interna `docker-desktop` en lugar de Ubuntu/WSL y configurar correctamente el entorno de trabajo.
- **Interpretación de resultados:** compartí las salidas de pruebas unitarias, cobertura, mutaciones, análisis estático y auditorías de seguridad. El asistente explicó los resultados y su relación con los objetivos del ejercicio.
- **Diagnóstico del CI:** revisamos conjuntamente los resultados de GitHub Actions para comprender por qué fallaban determinados controles y cómo interpretar los estados rojo y verde.
- **Apoyo en la documentación:** utilicé el asistente para organizar las mediciones y redactar las conclusiones, contrastándolas con la evidencia obtenida durante las ejecuciones.

### Propuestas corregidas o ajustadas

- Se ajustaron las instrucciones iniciales para trabajar correctamente en Ubuntu/WSL, después de identificar las diferencias entre este entorno y la distribución interna de Docker Desktop.
- Se precisó la interpretación de las pruebas de mutación: el puntaje aumentó de 20 % a 25 %, pero la cantidad absoluta de mutantes detectados permaneció en uno. Por lo tanto, no correspondía afirmar que las pruebas detectaban más mutantes únicamente por el aumento porcentual.
- Se revisó la interpretación del gate de seguridad. El análisis de infraestructura detectó seis secretos en un archivo modificado, aunque el parche agregaba solo uno. También se comprobó que revertir un secreto no necesariamente elimina su detección en el historial de Git.
- Se ajustaron los pasos de GitHub considerando el estado real de los Pull Requests y sus ejecuciones.

### Propuestas rechazadas o no adoptadas

- No se utilizaron resultados inventados o estimaciones para completar las métricas. Los valores documentados corresponden a las ejecuciones realizadas.
- No se desactivaron controles de seguridad ni se modificó el workflow para forzar resultados exitosos.
- No se aplicaron actualizaciones indiscriminadas como `npm audit fix --force`, priorizando las correcciones específicas solicitadas por el enunciado.
- No se asumió que todas las alertas de seguridad eran vulnerabilidades explotables; se realizó un triaje considerando severidad, contexto y alcance.

### Reflexión sobre el uso de IA

El principal aporte de la IA fue facilitar la comprensión técnica durante la ejecución práctica, más que automatizar la resolución de la tarea.

El proceso consistió en ejecutar comandos, revisar las salidas, formular preguntas, contrastar interpretaciones y tomar decisiones basadas en evidencia.

Esto permitió comprender mejor conceptos como cobertura de pruebas, análisis de mutaciones, vulnerabilidades de dependencias, secretos en el historial de Git, controles bloqueantes de CI y reproducibilidad de contenedores.

La ejecución de las pruebas, las modificaciones del repositorio y la verificación de los resultados se realizaron directamente en el entorno de trabajo. La IA se utilizó como herramienta de consulta, explicación, revisión y apoyo en la documentación, manteniendo una evaluación crítica de sus recomendaciones.
