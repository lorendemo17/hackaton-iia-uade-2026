M20 · ¿Este jugador hace trampa?
Dominio: Seguridad e integridad en videojuegos mediante IA
Integrantes: Lorenzo Fernández y Thomas Kong
Grupo: M20
Pista: Neurona (perceptrón con pesos y sesgo)
IA declarada en la ficha: Claude
Herramientas de apoyo: ChatGPT y Claude (detallar el uso real de cada una en la bitácora)
1. Formulación y PEAS
La pregunta. Para una cuenta de un shooter competitivo observada durante una semana, ¿utiliza aimbot?
- SÍ (1): la cuenta presenta indicios de trampa y se deriva a revisión manual.
- NO (0): el sistema no detecta indicios suficientes para derivarla automáticamente.
PEAS	Nuestro sistema
Medida de rendimiento (P)	Clasificar correctamente las cuentas, reduciendo falsos positivos y falsos negativos.
Entorno (E)	Shooter competitivo multijugador, estadísticas semanales de las cuentas y proceso de moderación.
Actuadores (A)	Emitir una clasificación SÍ/NO y solicitar revisión manual cuando la salida sea SÍ; no banear automáticamente.
Sensores (S)	Porcentaje de impactos a la cabeza, tiempo de reacción (ms) y horas totales de la cuenta.


Objetivo y métrica. El objetivo es ayudar a detectar tramposos sin acusar injustamente a jugadores legítimos. Evaluamos el desempeño con exactitud, precisión, sensibilidad y especificidad. No son lo mismo: las métricas resumen los errores observados, mientras que el objetivo incluye las consecuencias de cada decisión. Con pocos datos, una métrica alta no garantiza buen funcionamiento real.
2. El sistema
Cómo decide
Usamos StandardScaler y un Perceptron de scikit-learn (random_state=42, max_iter=10000). Primero se estandarizan las entradas usando solo el conjunto de entrenamiento. Luego el perceptrón calcula:
\[
z=w_1x'_1+w_2x'_2+w_3x'_3+b
\]
- Si z >= 0, predice SÍ (1).
- Si z < 0, predice NO (0).
En la versión documentada, los pesos aproximados son (1,4979; −1,5232; −1,7629) y el sesgo es −4. Corresponden a entradas estandarizadas y deben coincidir con los valores que imprima el cuaderno definitivo.
Variables que usa y por qué
Variable	Motivo
Impactos a la cabeza (%)	Puede aportar información sobre precisión inusual, pero por sí sola no prueba trampas.
Tiempo de reacción (ms)	Puede aportar información sobre respuestas anormalmente rápidas.
Horas totales de la cuenta	Ofrece contexto sobre la experiencia del jugador, sin probar inocencia ni culpabilidad.


Usamos las tres variables de la ficha. No utilizamos resultado_real como entrada de predicción: esa columna se usa únicamente para entrenar y evaluar. Tampoco utilizamos el identificador de caso ni el conjunto como características.
Cómo se corre en vivo
Enlace al cuaderno: [https://colab.research.google.com/drive/1HigGEFGnX8PCnJKG_6RVrBvd1r54wMqE?usp=sharing]
1. Abrir M20_anticheat.ipynb en Google Colab.
2. Subir el archivo datos/M20.csv (o M20.csv, según la ruta utilizada por el código).
3. Ejecutar todas las celdas en orden.
4. Revisar las matrices y las métricas.
5. Probar una cuenta nueva con analizar_jugador(cabeza, reaccion, horas).
Dos casos inventados para demostrar la decisión
Caso inventado	Cabeza	Reacción	Horas	Salida esperada del modelo documentado
A	85 %	140 ms	200 h	SÍ (1) → revisión manual
B	40 %	350 ms	1500 h	NO (0)


Antes de entregar, ejecutar ambos ejemplos en el cuaderno final y confirmar estas salidas.
3. Medición
El archivo M20.csv contiene 20 casos: 14 de entrenamiento (casos 1–14) y 6 de prueba (casos 15–20). Se conserva esta división.

Entrenamiento (14 casos)
	Predice SÍ	Predice NO
Real SÍ	VP = 2	FN = 0
Real NO	FP = 0	VN = 12


Prueba (6 casos)
	Predice SÍ	Predice NO
Real SÍ	VP = 1	FN = 0
Real NO	FP = 0	VN = 5


Métrica	Entrenamiento	Prueba
Exactitud	100 % (14/14)	100 % (6/6)
Precisión	100 % (2/2)	100 % (1/1)
Sensibilidad	100 % (2/2)	100 % (1/1)
Especificidad	100 % (12/12)	100 % (5/5)


Entrenamiento contra prueba. En esta ejecución no hay diferencia de exactitud. Sin embargo, eso no demuestra ausencia de sobreajuste ni capacidad de generalización: la prueba contiene solo seis cuentas, de las cuales únicamente una usa trampas. Sería necesario evaluar más casos, sin reajustar el modelo sobre los seis de prueba.
4. El error más caro
- Falso positivo: se clasifica como tramposo a un jugador honesto. Podría sufrir una revisión injusta; si se sancionara sin comprobarlo, podría generarse un conflicto reputacional.
- Falso negativo: se clasifica como honesto a un tramposo. Podría seguir perjudicando partidas y afectar la confianza de la comunidad.
Prioridad de diseño: nos preocupa especialmente el falso positivo por el daño que podría producir una sanción injustificada, aunque también necesitamos sensibilidad para encontrar tramposos. Por eso la salida SÍ conduce a revisión manual, nunca a baneo automático. Esta medida reduce el daño de una acusación errónea, pero no prueba que el clasificador produzca menos falsos positivos: para afirmar eso habría que comparar su tasa de FP con otra configuración.
5. ¿Comparado contra qué?
Línea base trivial: responder siempre NO (0). En los seis casos de prueba hay cinco jugadores legítimos y un tramposo. La línea base acierta 5/6, pero no detecta al tramposo.
Sistema	Exactitud en prueba	Sensibilidad en prueba
Siempre NO	83,33 %	0 %
Perceptrón	100 %	100 %


Checklist (Clase 6)	Respuesta
¿Qué datos?	M20.csv, 14 casos para entrenamiento y 6 para prueba.
¿Qué métrica?	Exactitud, precisión, sensibilidad y especificidad.
¿Comparado contra qué?	Línea base que siempre responde NO.
¿Quién lo midió?	Los integrantes del grupo M20, mediante la ejecución del modelo en Google Colab y el cálculo de las matrices de confusión utilizando scikit-learn.


6. Riesgo: datos → modelo → decisión → consecuencia
Datos: solo 20 ejemplos sintéticos; pueden representar mal a jugadores experimentados o estrategias de aimbot diferentes.

Modelo: un perceptrón lineal puede aprender patrones que no se mantengan en otros videojuegos. Las horas de cuenta no equivalen a prueba de inocencia.

Decisión: el sistema devuelve SÍ o NO, pero un SÍ representa una alerta para revisión, no una certeza de culpabilidad.

Consecuencia: un falso positivo podría perjudicar a jugadores honestos, especialmente si se transforma en una sanción sin evidencia adicional. Un falso negativo permite que un tramposo continúe jugando.

Medidas de cuidado: revisión humana antes de sancionar, evaluación con más datos y seguimiento de tasas de error. No usar repetidamente el conjunto de prueba para ajustar el modelo.
7. Bitácora
Uso de IA
Se usó ChatGPT para analizar la ficha M20, orientar la selección de la pista, preparar el código de Colab y apoyar la redacción del PEAS y del README. 
La siguiente tabla contiene alternativas de diseño razonadas. Antes de presentarlas como decisiones históricas, el equipo debe confirmar cuáles efectivamente analizó y rechazó.
#	Consulta o decisión analizada	Propuesta adoptada	Alternativa descartada y motivo
1	¿Cómo representar las estadísticas de una cuenta?	Perceptrón con tres entradas.	Usar solo precisión: el caso 7 tiene 83 % de impactos a la cabeza y es legítimo.
2	¿Cómo separar los ejemplos?	Respetar los 14 de entrenamiento y los 6 de prueba.	Mezclar ambos conjuntos: impediría evaluar limpiamente datos no usados en el ajuste.
3	¿Cómo medir el desempeño?	Cuatro métricas y matrices de confusión.	Reportar solo exactitud: puede ocultar tramposos no detectados.
4	¿Qué hacer ante un SÍ?	Enviar a revisión manual.	Banear automáticamente: un falso positivo puede perjudicar a jugadores honestos.
5	¿Cómo interpretar un 100 %?	Explicitar las limitaciones de la muestra.	Afirmar que el modelo es infalible: solo se probaron seis cuentas.


Intervención humana del equipo
Decisiones de diseño para confirmar con los integrantes:
1. Interpretación del caso 7. Su alta precisión y su reacción rápida no lo convierten en tramposo. Esto justifica no basar el sistema en una sola estadística.
2. Selección del enfoque. Se prefiere un perceptrón porque permite explicar entradas, pesos, sesgo y clasificación binaria, contenidos trabajados en clase.
3. Control de consecuencias. Una alerta no debe convertirse automáticamente en sanción; la revisión humana protege a los jugadores frente a errores del clasificador.
4. Lectura crítica de resultados. Se reconoce que el 100 % obtenido sobre pocos casos sintéticos no garantiza desempeño real.
5. Verificación práctica pendiente.
   Para verificar el funcionamiento del sistema, se plantearon dos pruebas con jugadores nuevos, utilizando valores que no pertenecen al conjunto de datos original.

Caso A: Lorenzo Fernández

Disparos a la cabeza: 85 %

Tiempo de reacción: 140 ms

Horas de juego: 200

Resultado esperado: SÍ (1)

El sistema clasifica al jugador como posible usuario de aimbot y recomienda enviarlo a revisión manual.

Caso B: Thomas Kong

Disparos a la cabeza: 45 %

Tiempo de reacción: 320 ms

Horas de juego: 1500

Resultado esperado: NO (0)

El sistema no detecta indicios suficientes para considerar al jugador sospechoso.

Conclusión: Estas pruebas permiten comprobar cómo responde el perceptrón ante dos perfiles diferentes. También sirven para verificar que las variables se procesen correctamente y que el modelo pueda clasificar casos nuevos sin conocer previamente su resultado real.

No completar esta tabla con calificaciones o devoluciones inventadas. Cada corrida debe corresponder a una evaluación real.
Defensa breve
Nuestro sistema analiza tres estadísticas de una cuenta: precisión a la cabeza, tiempo de reacción y horas acumuladas. Entrenamos un perceptrón con 14 casos y lo probamos con otros seis. Aplica pesos y sesgo sobre entradas estandarizadas y clasifica SÍ o NO. En estos datos sintéticos acertó los seis casos de prueba; eso no garantiza que funcione igual en un videojuego real. Frente a un sistema que siempre responde NO, que logra 83,33 % de exactitud sin detectar al tramposo, nuestro modelo detectó el único positivo. Si predice SÍ, solicitamos revisión manual y no una sanción automática.
