# Tarea sesión 2: CI de calidad

## Parte 0. Preparación

Se preparó el ambiente de trabajo utilizando Git y Docker y se ejecutaron las pruebas de los tres servicios.

Los resultados iniciales de Catalog fueron:

- 28 pruebas aprobadas y ninguna fallida.
- Cobertura general de líneas: 66,14 %.
- Cobertura de shifts.ts: 80 %, quedando sin cubrir la línea 15.
- Mutación: 5 mutantes, de los cuales 1 fue detectado y 4 sobrevivieron.
- Porcentaje de mutación detectada: 20 %.

Las pruebas de Incidents y Notifications también se ejecutaron correctamente.

Lo que más llamó mi atención en esta primera etapa fue que, a pesar de que todas las pruebas pasaban, cuatro de los cinco cambios introducidos artificialmente en el código no fueron detectados.

## Parte 1. Un bug, de rojo a verde

Se aplicó el primer parche, que incorporaba una prueba para verificar que un turno estuviera activo durante su último día.

Al ejecutar las pruebas, se obtuvieron los siguientes resultados:

- Situación inicial: 28 pruebas aprobadas y ninguna fallida.
- Con la nueva prueba: 28 aprobadas y 1 fallida.
- Después de aplicar la corrección: 29 aprobadas y ninguna fallida.

El problema estaba en la comparación de la fecha de término del turno, que no consideraba el último día como parte del periodo vigente.

La prueba y la corrección se registraron en commits separados:

- 9040e32: incorporación de la prueba.
- ac388a1: corrección del error.

Este ejercicio permitió comprender que un CI en verde no necesariamente significa que el sistema esté libre de errores, sino que cumple con las pruebas que se han definido hasta ese momento.

Considero importante incorporar primero la prueba y luego la corrección, porque permite comprobar que la prueba efectivamente detecta el problema y que posteriormente el cambio lo resuelve.

## Parte 2. ¿Las pruebas detectan cambios?

Durante esta etapa se incorporó también una prueba adicional para verificar la selección del turno de menor identificador cuando existen varios turnos activos.

Los resultados obtenidos fueron:

| Indicador | Inicial | Final |
|---|---:|---:|
| Pruebas aprobadas de Catalog | 28 | 30 |
| Cobertura general de Catalog | 66,14 % | 66,66 % |
| Cobertura de shifts.ts | 80 % | 100 % |
| Mutantes detectados | 1 | 1 |
| Mutantes sobrevivientes | 4 | 3 |
| Porcentaje de mutación detectada | 20 % | 25 % |

También se midió la cobertura de los otros servicios:

- Incidents: 77 %, con 82 pruebas aprobadas en 12,81 segundos.
- Notifications: 70,3 %, con ejecución exitosa de las pruebas.

Si bien la cobertura de shifts.ts llegó al 100 %, todavía quedaron tres mutantes que las pruebas no fueron capaces de detectar.

Además, el aumento del porcentaje de mutación detectada, de 20 % a 25 %, se explica por la disminución del número total de mutantes y no por un aumento de los mutantes detectados.

La línea donde se encontraba el error del último día del turno ya estaba cubierta inicialmente. Esto demuestra que ejecutar una línea de código durante una prueba no significa necesariamente que se esté verificando correctamente su comportamiento.

Solicitaría al equipo responsable incorporar pruebas que consideren especialmente los límites de las fechas y las distintas condiciones que pueden presentarse entre turnos.

## Parte 3. Análisis estático

Se ejecutaron los comandos km score, km smells y km infra smells.

### Calidad general del código

El proyecto obtuvo una nota de 71,1 puntos, equivalente a una clasificación C. Se analizaron 23 archivos y 2.679 líneas de código.

Los tres archivos con menor puntuación fueron:

| Archivo | Puntaje | Nota |
|---|---:|---|
| incidents/api.py | 42,1 | F- |
| notifications/server.go | 51,8 | F |
| catalog/app.ts | 52,6 | F |

La dimensión con menor puntuación fue la complejidad cognitiva, con 34 puntos. Esto indica que existen partes del código que podrían resultar difíciles de comprender y mantener.

### Hallazgos en el código

Se identificaron 147 hallazgos distribuidos en 10 archivos:

- 125 valores incorporados directamente en el código (magic numbers).
- 17 hallazgos relacionados con la cantidad de parámetros.
- 5 hallazgos relacionados con la extensión de funciones.

El hallazgo más frecuente corresponde a los valores incorporados directamente en el código, representando aproximadamente el 85 % del total.

### Hallazgos de infraestructura

Se identificaron 20 hallazgos:

- 5 credenciales o secretos escritos directamente en compose.yaml.
- 15 referencias a acciones de GitHub que no están fijadas a un commit específico.

Respecto de los secretos, considero que representan un riesgo si corresponden a credenciales reales. En el contexto de un taller, pueden corresponder a valores de prueba, por lo que es necesario distinguir su finalidad antes de clasificarlos como un problema efectivo de seguridad.

Respecto de las acciones de GitHub, existe un riesgo potencial, ya que una referencia a una versión puede cambiar con el tiempo. Sin embargo, considero que su prioridad es menor dentro del contexto de este ejercicio.

Adicionalmente, se ejecutaron los controles de tipos, lint y compilación de Catalog, sin errores. Se observó que lint y typecheck ejecutaban el mismo comando, por lo que entregaban información similar.

## Parte 4. Dependencias y secretos

### Catalog

La primera revisión identificó cuatro vulnerabilidades:

- Dos de nivel moderado.
- Dos de nivel alto.

Las vulnerabilidades estaban asociadas a hono, @hono/node-server, vite y zod.

Se actualizaron las dependencias afectadas y se volvieron a ejecutar los controles.

El resultado final fue:

- npm audit: 0 vulnerabilidades.
- npm audit --omit=dev: 0 vulnerabilidades.
- 30 pruebas aprobadas y ninguna fallida.
- Compilación y validación de tipos sin errores.

### Incidents

Se actualizaron las dependencias certifi y opentelemetry-api.

Las versiones finales observadas fueron:

- certifi: 2026.11.20.
- opentelemetry-api: 1.45.2.

Después de la actualización, pip-audit no reportó vulnerabilidades conocidas y las pruebas se ejecutaron correctamente.

### Notifications

El análisis mediante govulncheck identificó vulnerabilidades relacionadas con componentes de Go utilizados por el servicio.

A diferencia de Catalog e Incidents, su solución requiere revisar y actualizar la versión de Go utilizada, ya que los hallazgos están asociados a componentes de su biblioteca estándar.

El control se mantuvo activo, sin ocultar sus resultados, aunque el workflow permite continuar cuando este paso falla.

### Secretos

Se ejecutó gitleaks sobre el historial del repositorio. El resultado fue de siete hallazgos en los siguientes archivos:

- tarea/03-secreto.patch: 1 hallazgo.
- compose.yaml: 2 hallazgos.
- incidents/config.py: 1 hallazgo.
- notifications/README.md: 2 hallazgos.
- notifications/server.go: 1 hallazgo.

El secreto identificado en tarea/03-secreto.patch corresponde a una clave ficticia preparada para comprobar el funcionamiento del gate.

Los siete hallazgos no necesariamente representan siete credenciales diferentes, ya que un mismo valor puede aparecer en distintos archivos o ser detectado por más de una regla.

### Tabla de triaje

| Hallazgo | Decisión | Justificación |
|---|---|---|
| Vulnerabilidad alta de zod | Corregir | Dependencia actualizada. |
| Vulnerabilidad alta de vite | Corregir | Resuelta mediante actualización. |
| Vulnerabilidades de hono | Corregir | Dependencias actualizadas. |
| Dependencias de Incidents | Corregir | Auditoría posterior sin hallazgos. |
| Vulnerabilidades de Go | Posponer | Revisar actualización antes del 15/12/2026. |
| Secreto en 03-secreto.patch | Aceptar | Clave ficticia del ejercicio. |
| Otras credenciales detectadas | Posponer | Verificar su uso antes del 15/12/2026. |

El hallazgo más difícil de decidir fue el relacionado con Go, porque su solución requiere una revisión más amplia del ambiente utilizado por el servicio. Considero razonable mantenerlo temporalmente como una advertencia, pero con un plazo definido para su revisión.

## Parte 5. El gate en el CI

Se incorporó el workflow gate-calidad.yml y se configuraron como obligatorios los siguientes controles en la rama main del fork:

- calidad-y-seguridad.
- construye-imagenes.

En la rama de trabajo, ambos jobs se ejecutaron correctamente después de corregir las dependencias de Catalog e Incidents.

El workflow incorpora los siguientes controles:

- Evaluación de la calidad del código mediante km score.
- Identificación de problemas del código mediante km smells.
- Revisión de seguridad de infraestructura mediante km infra smells.
- Revisión de dependencias de Catalog mediante npm audit.
- Revisión de dependencias de Incidents mediante pip-audit.
- Revisión de dependencias de Notifications mediante govulncheck.
- Detección de secretos mediante gitleaks.
- Construcción de las imágenes de los tres servicios.

El control de dependencias de Notifications utiliza continue-on-error: true, por lo que sus hallazgos no bloquean el resultado general del job.

### Comprobación del bloqueo

Se creó una rama independiente llamada prueba-gate y se aplicó el parche que incorpora una clave ficticia en compose.yaml.

En el pull request se obtuvieron los siguientes resultados:

- Cinco controles exitosos.
- Un control fallido: calidad-y-seguridad.
- La opción de merge quedó deshabilitada bajo las reglas normales.

El paso de seguridad de infraestructura reportó 12 hallazgos, correspondientes a seis secretos escritos directamente en el código y seis acciones no fijadas a un commit.

El parche solamente agregó un secreto. La diferencia se explica porque el analizador revisó el contenido de los archivos modificados, incluyendo cinco secretos que ya existían en compose.yaml.

Por lo tanto, una futura modificación de este archivo podría generar un bloqueo debido a problemas que ya estaban presentes, aunque el nuevo cambio no incorpore otro secreto.

### Retiro del secreto

Posteriormente, se retiró la clave ficticia mediante un nuevo commit.

Sin embargo, el control calidad-y-seguridad continuó en rojo. Esta vez, el paso que falló fue el de detección de secretos mediante gitleaks.

El resultado mostró un secreto en el historial de los 12 commits analizados, asociado al commit donde se había incorporado la clave.

Esto permitió comprobar que eliminar un secreto del archivo no significa eliminarlo del historial de Git.

Si se tratara de una credencial real, sería necesario invalidarla o cambiarla y evaluar su eliminación del historial del repositorio.

La rama de prueba se mantuvo separada de la rama de trabajo y no se integró a main.

### Alcance del gate

El gate permite bloquear problemas de seguridad, vulnerabilidades que superan los criterios establecidos, deterioros en la calidad del código y errores en la construcción de imágenes.

Sin embargo, no garantiza que todas las reglas de negocio estén correctamente verificadas. Por ejemplo, un error que no esté contemplado en las pruebas podría pasar todos los controles.

Respecto de Notifications, considero aceptable que temporalmente no bloquee el CI, siempre que exista un compromiso de revisar y resolver los hallazgos dentro del plazo establecido.

## Parte 6. Contenedores

Se construyeron correctamente las imágenes de los tres servicios, tanto de manera local como mediante el CI.

### Tamaño de las imágenes

Los tamaños observados localmente fueron:

| Imagen | Uso de disco | Tamaño del contenido |
|---|---:|---:|
| pager-incidents | 266 MB | 62,5 MB |
| pager-catalog | 254 MB | 62,9 MB |
| pager-notifications | 14,7 MB | 3,31 MB |

La imagen de Notifications es considerablemente más pequeña. Esto se explica principalmente porque Go permite generar un ejecutable que requiere menos componentes para funcionar, mientras que Python y Node.js necesitan sus respectivos ambientes de ejecución y dependencias.

### Revisión de la imagen de Incidents

| Criterio | Resultado |
|---|---|
| Versiones completamente fijas | Cumple parcialmente |
| Usuario sin privilegios | Cumple |
| Sin secretos visibles en el historial de capas | Cumple en la revisión realizada |
| Archivo .dockerignore | Cumple |

El Dockerfile utiliza python:3.12-slim y uv:0.7. Si bien se especifican versiones, estas no están fijadas a una versión exacta e inmutable, por lo que podrían cambiar con el tiempo.

Mediante docker inspect se comprobó que la imagen utiliza el usuario nobody, evitando su ejecución como usuario root.

En la revisión del historial de las capas no se observaron credenciales privadas visibles. Sin embargo, esta revisión no garantiza por sí sola la ausencia de secretos dentro de todos los archivos de la imagen.

También se comprobó la existencia del archivo .dockerignore.

### Ejecución en distintos puertos

Se levantaron dos contenedores utilizando la misma imagen de Incidents:

- pager-a: puerto externo 9001 e interno 8080.
- pager-b: puerto externo 9002 e interno 9090.

En ambos casos, al consultar /health desde el navegador, se obtuvo la respuesta:

{"status":"ok"}

Posteriormente, se detuvieron ambos contenedores y se comprobó que ya no estaban en ejecución.

Considero importante que una misma imagen pueda utilizarse con distintas configuraciones sin necesidad de reconstruirla, porque permite reutilizarla en diferentes ambientes y reducir el riesgo de introducir diferencias durante el despliegue.

## Registro de uso de IA

Durante el desarrollo de la tarea utilicé ChatGPT como herramienta de apoyo para comprender los comandos, interpretar los resultados y organizar las conclusiones, considerando que DevOps no es mi área de especialización.

Acepté sugerencias relacionadas con la ejecución de los controles, la incorporación de una prueba adicional y la actualización de dependencias. Algunas recomendaciones iniciales fueron complementadas tras revisar nuevamente el enunciado, particularmente las relacionadas con el análisis estático y la detección de secretos.

Las instrucciones sugeridas fueron ejecutadas y verificadas en mi ambiente de trabajo. Se mantuvieron los controles definidos en el workflow original, sin desactivarlos para obtener resultados exitosos.

La IA también apoyó a mejorar la redacción de este documento, utilizando los resultados observados durante la ejecución de la tarea.

## Conclusiones finales

### 1. ¿Qué me dice la distancia entre que las pruebas pasan y que las pruebas detectan?

Fue uno de los principales aprendizajes de la tarea.

Inicialmente, las 28 pruebas de Catalog pasaban correctamente, pero existía un error en la determinación del último día de un turno. Además, las pruebas solamente detectaban uno de los cinco mutantes generados.

Incluso después de alcanzar una cobertura del 100 % en shifts.ts, quedaron tres mutantes sobrevivientes.

Esto me permite concluir que tener muchas pruebas, o incluso una cobertura elevada, no garantiza que se estén verificando correctamente las distintas situaciones que pueden presentarse.

### 2. ¿Qué control me entregó más información y cuál menos?

El control de mutación fue el que me entregó más información, porque permitió identificar una diferencia que no era evidente al observar solamente los resultados de las pruebas y la cobertura.

También considero relevante el análisis de secretos, especialmente porque permitió comprobar que una credencial puede seguir presente en el historial de Git después de haber sido eliminada del archivo.

El control que me entregó menos información adicional fue lint de Catalog, debido a que ejecutaba el mismo comando que typecheck. Ambos controles fueron exitosos, pero entregaron información equivalente.

### 3. Si tuviera una hora más, ¿qué agregaría al pipeline y por qué?

Incorporaría un control de mutación para las funciones que contienen reglas de negocio relevantes, comenzando por shifts.ts.

Lo implementaría inicialmente como un control informativo, que permita observar cuántos cambios artificiales son detectados por las pruebas y cómo evoluciona ese resultado.

La razón es que, durante el ejercicio, la cobertura llegó al 100 % sin que aumentara la cantidad de mutantes detectados.

Considero que este control permitiría complementar las pruebas tradicionales y orientar al equipo respecto de dónde es necesario mejorar las pruebas existentes.