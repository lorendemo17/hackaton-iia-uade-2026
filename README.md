M20 · ¿Este jugador hace trampa?
Dominio: Seguridad e integridad en videojuegos mediante IA
Integrantes: Lorenzo Fernández Thomas Kong
Código del grupo: M20
Pista: Neurona (perceptrón con pesos y sesgo)
IA declarada en la ficha: Claude. Herramientas realmente utilizadas: Claude y Chat GPT (por tema de tokens).
Estado del documento: Se verificaron las métricas del modelo descrito mediante ejecución local con M20.csv. Falta completar los datos del equipo, confirmar que el cuaderno subido a GitHub ejecuta el mismo modelo y registrar las corridas reales del evaluador.

1. Formulación y PEAS
Pregunta y respuesta
Pregunta: Para una cuenta de un shooter competitivo observada durante una semana, ¿usa trampas (aimbot)?
Salida: SÍ (1) = el sistema clasifica a la cuenta como tramposa y la deriva a revisión manual; NO (0) = no se detectan indicios suficientes para derivarla automáticamente.
Objetivo versus métricas
El objetivo operativo es priorizar para revisión manual las cuentas con comportamiento compatible con aimbot, procurando no perjudicar a jugadores honestos y no dejar pasar tramposos. Las métricas (exactitud, precisión, sensibilidad y especificidad) son indicadores para evaluar esta capacidad; una buena métrica en pocos casos no equivale a cumplir el objetivo en un videojuego real.
PEAS
Elemento	Descripción
P — Performance / Medida de desempeño	Identificar correctamente cuentas sospechosas, limitar falsos positivos que perjudican a jugadores legítimos y falsos negativos que afectan la integridad competitiva. Una decisión positiva inicia revisión, no baneo automático.
E — Environment / Entorno	Shooter competitivo multijugador; cuentas de jugadores observadas durante una semana; sistema de moderación y revisión humana.
A — Actuators / Actuadores	Emitir SÍ o NO; marcar una cuenta para revisión manual cuando la respuesta sea SÍ. No ejecutar sanciones automáticas.
S — Sensors / Sensores	Porcentaje de disparos a la cabeza entre los impactos; tiempo de reacción en milisegundos; horas totales de la cuenta.


El sistema solo utiliza información disponible al decidir: las tres estadísticas del caso. No usa la etiqueta resultado_real para predecir nuevos casos; esa columna sirve exclusivamente para entrenar y evaluar.
2. El sistema
Datos y división obligatoria
El archivo M20.csv contiene 20 casos sintéticos, una cuenta observada durante una semana por caso. Los casos 1 a 14 son de entrenamiento y los casos 15 a 20, de prueba. No cambiamos la división.

Entradas: Disparos a la cabeza (% de los impactos), Tiempo de reacción (ms) y Horas en la cuenta (h totales).
Modelo
Se utiliza StandardScaler seguido de un Perceptron de scikit-learn con random_state=42 y max_iter=10000. El escalado se ajusta solo con entrenamiento para evitar fuga de información del conjunto de prueba. El perceptrón aprende pesos y sesgo, calcula una suma ponderada y aplica una decisión binaria:
\[
z = w_1 x'_1+w_2 x'_2+w_3 x'_3+b,\qquad \hat y=\begin{cases}1 & z\ge 0\\0 & z<0\end{cases}
\]
Aquí, las variables x' son las tres entradas estandarizadas, no sus valores originales. Para la ejecución verificada, los pesos aproximados sobre esas entradas fueron (1,4979; -1,5232; -1,7629) y el sesgo -4. El peso de la precisión es positivo, mientras que los de reacción y horas son negativos: con los demás valores fijos, más precisión eleva la activación, y mayores valores de reacción y horas la reducen. Estos pesos describen este modelo entrenado, no una ley general sobre el comportamiento de jugadores.
Reproducción en Google Colab
1. Abrir Google Colab y cargar M20_anticheat.ipynb.
2. Subir M20.csv al entorno de Colab (por ejemplo, con files.upload()).
3. Ejecutar todas las celdas en orden. Se necesitan pandas y scikit-learn.
4. Verificar que el cuaderno carga el CSV, divide por conjunto, entrena el perceptrón con entrenamiento, imprime las métricas de ambos conjuntos y permite ingresar un caso nuevo mediante la función analizar_jugador(cabeza, reaccion, horas).
Caso nuevo de demostración: analizar_jugador(85, 140, 200) produce SÍ (1) en el modelo verificado y debe generar la indicación de revisión manual. La entrada del caso nuevo no incluye la respuesta real, porque esta es desconocida en el momento de decidir.
3. Medición
Se evaluó el modelo con los datos de M20.csv, usando los casos de entrenamiento para ajustar el modelo y los seis de prueba para medir las predicciones fuera de entrenamiento.
Matriz de confusión: entrenamiento (14 casos)
Resultado real / Predicción	NO (0)	SÍ (1)
NO (0)	12 (VN)	0 (FP)
SÍ (1)	0 (FN)	2 (VP)


Matriz de confusión: prueba (6 casos)
Resultado real / Predicción	NO (0)	SÍ (1)
NO (0)	5 (VN)	0 (FP)
SÍ (1)	0 (FN)	1 (VP)


Métricas verificadas
Métrica	Fórmula	Entrenamiento	Prueba
Exactitud	(VP + VN) / total	100 % (14/14)	100 % (6/6)
Precisión	VP / (VP + FP)	100 % (2/2)	100 % (1/1)
Sensibilidad	VP / (VP + FN)	100 % (2/2)	100 % (1/1)
Especificidad	VN / (VN + FP)	100 % (12/12)	100 % (5/5)


Advertencia: el conjunto de prueba tiene un solo positivo. Una predicción distinta cambiaría marcadamente los indicadores. Estos resultados no prueban un desempeño perfecto en jugadores reales ni deben extrapolarse a otras poblaciones.
4. El error más caro
Falso positivo: el sistema responde SÍ para un jugador honesto. Esto puede causarle una revisión injustificada; si alguien lo sanciona sin pruebas suficientes, puede perjudicar su reputación y generar conflictos con la comunidad. Falso negativo: el sistema responde NO para un tramposo; este puede seguir arruinando partidas y dañando la confianza de los jugadores.

Decisión de diseño: una predicción positiva no produce un baneo, sino una revisión humana. Así reducimos el daño directo de un falso positivo. También registramos la importancia de la sensibilidad para no ignorar sistemáticamente a los tramposos. No hay un costo monetario fijado en la ficha que permita afirmar cuál de los dos errores es universalmente más caro; la gravedad depende de las consecuencias operativas.
5. ¿Comparado contra qué?
Línea base trivial: predecir siempre NO (0). Como en prueba hay cinco cuentas honestas y una tramposa, logra una exactitud de 5/6 = 83,33 %, pero sensibilidad = 0 %, ya que no descubre al único tramposo.
Sistema	Exactitud en prueba	Sensibilidad en prueba
Siempre NO	83,33 %	0 %
Perceptrón	100 %	100 %


El perceptrón supera esta línea base en estos seis ejemplos. La exactitud por sí sola puede ocultar que un modelo no detecta tramposos; por eso se observan también sensibilidad, precisión y especificidad.

Checklist de evaluación: ¿objetivo distinto de la métrica? Sí. ¿Línea base? Sí. ¿Entrenamiento y prueba separados? Sí. ¿Se usa solo información disponible al decidir? Sí. ¿Matriz que cierra con cantidad de casos? Sí. ¿Consecuencias de errores explicadas? Sí. ¿Caso nuevo en vivo? Incluido en el cuaderno si se sube y ejecuta la implementación indicada.

6. Riesgo: datos → modelo → decisión → consecuencia
- Datos: los casos son sintéticos, pocos y pueden representar de forma incompleta a jugadores realmente expertos; las horas jugadas no prueban inocencia o culpabilidad.
- Modelo: un perceptrón lineal puede aprender patrones de esta muestra que no se sostengan en otros juegos, rangos o estrategias de trampa.
- Decisión: etiquetar una cuenta SÍ significa sospecha para revisión, no certeza de culpabilidad.
- Consecuencia: una falsa sospecha puede afectar a usuarios honestos; una omisión permite que continúen las trampas. Por eso recomendamos revisión humana, evidencia adicional y supervisión periódica de las tasas de error antes de cualquier sanción.
Limitación relevante: la selección de arquitectura y la inspección de resultados sobre un conjunto tan pequeño pueden influir en decisiones posteriores. No debe presentarse la prueba como una validación robusta de generalización si se ajusta el modelo repetidamente para mejorar esos mismos seis casos.

7. Bitácora

Uso de inteligencia artificial

Durante el desarrollo del proyecto utilizamos herramientas de inteligencia artificial como apoyo para comprender el problema, construir el modelo y evaluar sus resultados. Las propuestas fueron analizadas teniendo en cuenta las características del sistema anticheat y las posibles consecuencias de sus decisiones.

Qué le pedimos a la IA
Qué aceptamos
Qué rechazamos y por qué
1 Analizar qué tipo de sistema de IA sería adecuado para detectar aimbot en videojuegos.
Utilizar un perceptrón con tres variables de entrada y una salida binaria.
Descartamos utilizar únicamente el porcentaje de disparos a la cabeza, porque un jugador legítimo puede tener una precisión elevada.
2 Proponer una implementación del modelo utilizando los datos M20.
Utilizar Python, scikit-learn y StandardScaler para entrenar un perceptrón.
Descartamos modificar la división original de los datos, porque necesitábamos mantener separados los casos de entrenamiento y prueba.
3 Explicar cómo evaluar el funcionamiento del sistema.
Utilizar matrices de confusión y calcular exactitud, precisión, sensibilidad y especificidad.
Descartamos utilizar únicamente la exactitud, porque podría ocultar falsos negativos importantes para la detección de tramposos.
4 Analizar las consecuencias de una clasificación incorrecta.
Utilizar la predicción positiva para enviar cuentas sospechosas a revisión manual.
Descartamos el baneo automático, porque un falso positivo podría perjudicar a jugadores honestos.
5 Proponer una forma de comprobar si el modelo aporta una mejora.
Comparar el perceptrón con un sistema que siempre responde NO.
Descartamos interpretar el 100 % de exactitud como prueba de funcionamiento perfecto, debido al tamaño reducido de la muestra.
Intervención humana
Como equipo, analizamos las características del problema y consideramos que detectar trampas no consiste únicamente en identificar jugadores con estadísticas elevadas.
Un ejemplo importante fue el caso 7 del conjunto de entrenamiento, que presenta un 83 % de disparos a la cabeza y un tiempo de reacción de 123 ms, pero corresponde a un jugador legítimo. Esto permitió comprender la importancia de considerar conjuntamente las tres variables.
También priorizamos evitar sanciones injustificadas. Por ese motivo, planteamos que el sistema funcionara como una herramienta de detección preliminar y no como un mecanismo de baneo automático.
Finalmente, reconocimos que los resultados obtenidos sobre veinte casos sintéticos no son suficientes para garantizar un funcionamiento correcto en un entorno competitivo real. Por lo tanto, consideramos necesaria una revisión humana y una evaluación con conjuntos de datos más amplios.
Herramientas de IA utilizadas
Herramienta: ChatGPT.
¿Para qué se utilizó?
Utilizamos ChatGPT como herramienta de apoyo para desarrollar nuestro sistema anticheat basado en inteligencia artificial.
Las consultas estuvieron relacionadas con:
- La elección de la pista de trabajo, seleccionando una neurona artificial (perceptrón).
- La interpretación del conjunto de datos M20.
- La construcción de un modelo de clasificación en Google Colab.
- La elaboración del PEAS y la documentación del proyecto.
- La evaluación del modelo mediante matrices de confusión y métricas.

Propuestas aceptadas Se propuso utilizar un perceptrón con tres variables de entrada: porcentaje de disparos a la cabeza, tiempo de reacción y horas acumuladas en la cuenta.

Propuestas rechazadas o modificadas

1. Detectar trampas únicamente mediante el porcentaje de disparos a la cabeza.

Se consideró utilizar solamente la precisión del jugador para identificar posibles tramposos. Sin embargo, se descartó porque una precisión elevada no necesariamente significa que se esté utilizando aimbot.

Por ejemplo, el caso 7 del conjunto de entrenamiento presenta un 83 % de disparos a la cabeza y no utiliza trampas. Por este motivo, se decidió considerar también el tiempo de reacción y las horas acumuladas en la cuenta.

2. Establecer reglas fijas para determinar si un jugador hace trampa.

Se consideró clasificar como sospechosos a los jugadores que superaran determinados valores de precisión y velocidad de reacción.

Esta alternativa se descartó como modelo principal porque los límites establecidos manualmente podrían generar clasificaciones incorrectas. Se prefirió utilizar un perceptrón que ajustara sus pesos a partir de los datos de entrenamiento.

3. Banear automáticamente a los jugadores clasificados como tramposos.

Se descartó la posibilidad de que una predicción positiva provocara una suspensión automática.

La decisión se fundamentó en el riesgo de falsos positivos, que podrían perjudicar a jugadores legítimos. Por eso, el sistema se orientó a identificar cuentas sospechosas y enviarlas a revisión manual.

Intervención humana

1. Selección de variables: Se analizaron las tres características disponibles en el conjunto M20 y se consideró necesario utilizarlas conjuntamente para evitar depender de un único indicador de comportamiento.

2. Elección del modelo: Se optó por trabajar con un perceptrón, considerando su relación con los contenidos estudiados en clase y la posibilidad de explicar matemáticamente su funcionamiento mediante pesos, sesgo y una función de activación.

3. Evaluación de resultados: Se estableció la importancia de analizar la matriz de confusión, prestando especial atención a los falsos positivos, debido a las consecuencias que una acusación incorrecta podría tener sobre jugadores legítimos.

4. Supervisión de las decisiones: Se determinó que las predicciones del sistema funcionaran como alertas para una revisión manual, evitando que una clasificación automática fuera suficiente para sancionar a una cuenta.

5. Reconocimiento de limitaciones: Se consideró que obtener un 100 % de exactitud sobre un conjunto de prueba de seis casos no garantiza un rendimiento equivalente en situaciones reales, por lo que sería necesario evaluar el sistema con más datos antes de implementarlo en un videojuego competitivo.

Defensa breve (sin IA)
Nuestro sistema analiza tres estadísticas disponibles de una cuenta en una semana: precisión a la cabeza, tiempo de reacción y horas totales de juego. Entrenamos un perceptrón con 14 casos etiquetados y lo evaluamos con los seis restantes, sin cambiar esa división. El modelo aplica pesos y sesgo a las entradas estandarizadas y devuelve SÍ o NO. En los datos de prueba clasificó correctamente seis casos, pero son ejemplos sintéticos y no aseguran un rendimiento igual en un entorno real. A diferencia de predecir siempre NO, que alcanza 83,33 % de exactitud pero no descubre ningún tramposo, nuestro modelo detectó el único positivo de prueba. Una respuesta SÍ significa revisión manual, no sanción automática, para reducir el daño de falsos positivos.
