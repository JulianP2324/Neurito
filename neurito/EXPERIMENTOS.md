Experimentos

1. Dos necesidades fijas, una variable. La acción cambia justo cuando la prioridad que varía cruza a la más alta de las otras dos.

Fijas	Varía	Cambio real	Cambio de la red
Hambre 0,3 · Energía 0,5 (cansancio 0,5)	Aburrimiento	DORMIR→JUGAR entre 0,50 y 0,55	entre 0,45 y 0,50
Energía 0,5 · Aburrimiento 0,4	Hambre	DORMIR→COMER entre 0,45 y 0,50	igual
Hambre 0,3 · Aburrimiento 0,4	Energía	DORMIR→JUGAR entre 0,60 y 0,65	entre 0,55 y 0,60

La red aprendió la frontera con un error de aproximadamente 0,05. El cambio es gradual en Softmax: con el aburrimiento en 0,4, 0,5 y 0,6, la probabilidad de JUGAR pasa de 0,13 a 0,51 y a 0,88.

2. Prioridades casi iguales. Con hambre, cansancio y aburrimiento en (0,50, 0,50, 0,50), Softmax da [0,35, 0,33, 0,31], casi un tercio para cada acción. Con (0,52, 0,50, 0,48) da [0,48, 0,32, 0,20], y una diferencia de 0,02 ya inclina el resultado. Lo mismo ocurre con los tres valores en 0,1 o en 0,9, así que Softmax depende de las diferencias y no del nivel absoluto. Con diferencias claras da [1, 0, 0].

Cuanto menor es el margen entre las dos prioridades más altas, menor es la confianza y el acierto (muestra de prueba):

Margen	Confianza media	Acierto
< 0,02	0,53	90 %
0,05–0,10	0,77	100 %
> 0,20	≥ 0,99	100 %

Los empates exactos los decide np.argmax por orden de índice (COMER antes que DORMIR antes que JUGAR). Esa regla arbitraria no se puede aprender.

3. Capa oculta (1200 épocas, lr 0,1).

Neuronas	Seed 42: pérdida / acierto	Promedio de 10 semillas: acierto (mín.)
3	0,162 / 96,3 %	78,9 % ± 17 (60,5 %)
6	0,123 / 99,4 %	97,3 % ± 1,8 (92,5 %)
10	0,126 / 99,3 %	98,7 % ± 0,5 (97,8 %)
20	0,122 / 98,8 %	99,0 % ± 0,3 (98,6 %)

Con 3 neuronas, 5 de 10 semillas se quedan atascadas cerca de 61 %. Más neuronas dan estabilidad, no mucha más precisión. El 98,8 % de H=20 frente al 99,4 % de H=6 no es una diferencia real: con 1000 ejemplos de prueba el error de muestreo es de unos ±0,3 puntos.

4. Épocas y tasa de aprendizaje.

Épocas (lr 0,1)	Pérdida	Acierto
50	0,94	60,6 %
200	0,50	91,8 %
600	0,19	98,6 %
1200	0,12	99,4 %
6000	0,05	99,6 %

Mejora y luego se estanca: la pérdida sigue bajando, pero el acierto se queda en 99,5–99,7 %.

lr (1200 épocas)	Pérdida	Acierto	Comportamiento
0,001	1,03	41,8 %	casi no aprende
0,01	0,74	82,9 %	lento
0,1	0,12	99,4 %	estable
0,5	0,048	99,6 %	el mejor
1–2	0,045–0,056	98,7 % / 97,3 %	la pérdida sube y baja (unas 550 subidas)
≥ 5	1,1 a 14,5	~34 %	inestable y colapsa a nivel de azar

Con lr bajo, solo cambia la velocidad: lr 0,01 con 12000 épocas da lo mismo que lr 0,1 con 1200. Con lr alto, en cambio, se vuelve inestable. Además, con lr=1 la pérdida es menor que con lr=0,5 pero el acierto es peor, así que ambas métricas no siempre coinciden.

5. Una cuarta acción (ej. BAÑAR). No la corrí; es lo que exigiría el código:

Datos: si depende de una nueva necesidad (suciedad), X pasa a 4 columnas y W1 a 4×H. Si depende de las existentes, la entrada no cambia.
Etiquetas: una regla o datos reales que digan cuándo gana, por ejemplo argmax([hambre, 1-energía, aburrimiento, suciedad]). Necesitas suficientes ejemplos de la clase nueva, sobre todo cerca de las fronteras con las otras.
Red: W2 pasa a H×4, b2 a 1×4, Softmax sobre 4 salidas, y hay que reentrenar desde cero.
Aplicación: agregar la acción a actions, su texto, sus sprites, su tecla y su caso en la regla de NEUTRAL.
Evaluación: si la acción es poco frecuente, medir el recall por clase.
Explicación de resultados

1. Pesos y sesgos. Los pesos (W1, W2) multiplican cada entrada y dicen cuánto y en qué sentido influye cada necesidad en cada neurona. Los sesgos (b1, b2) se suman al final y desplazan el umbral de activación sin depender de las entradas. Tu red tiene 45 parámetros: 18+6+18+3. Los pesos se inicializan al azar para que las neuronas no sean idénticas, y los sesgos en cero.

2. Cansancio = 1 − energía. Las tres cantidades deben medir urgencia en el mismo sentido: más alto significa más necesidad de actuar. Hambre y aburrimiento ya van así, pero energía alta significa descansado, o sea menos ganas de dormir. Invertirla pone las tres en una escala comparable para el argmax. La red solo recibe la energía, así que tiene que aprender el peso negativo y el “1” mediante el sesgo; ahí se ve para qué sirve.

3. ReLU y Softmax. ReLU es max(0, z): deja pasar lo positivo y anula lo negativo. Aporta no linealidad; sin ella, dos capas serían una sola transformación lineal. Una neurona que siempre da negativo no aprende más (“muerta”). Softmax convierte las 3 salidas en probabilidades que suman 1, conservando el orden y amplificando diferencias. Con entropía cruzada su gradiente es simplemente p − y (delta2 en tu código). Un detalle por razonamiento, no probado: esta regla es un argmax de funciones lineales, así que en teoría ni haría falta la capa oculta. Por eso tampoco mejora mucho al agrandarla.

4. Por qué una precisión alta engaña.

Mide acuerdo con tu regla sintética, no si el comportamiento es bueno, y el test sale de la misma distribución.
Esconde la frontera: el 99,2 % global (muestra de 200 mil casos) baja a 87 % en casi-empates (margen < 0,02), que son el 6 % de los casos. En tu test los 6 errores tenían margen medio de 0,005.
Con clases desbalanceadas es peor: tus clases están parejas (34/32/34 %, base 34 %), pero una cuarta acción rara, del 5 %, haría que un modelo que nunca la predice tuviera 95 %. La matriz de confusión y el recall por clase lo muestran.
No mide el estado NEUTRAL, que es una regla de pantalla sin etiqueta.
Ignora la confianza: en aciertos es 0,90 y en errores 0,51.
Una sola corrida engaña: con 3 neuronas, la seed 42 da 96 % y otras semillas dan 60 %.