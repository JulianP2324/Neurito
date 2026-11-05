# Informe de laboratorio · Neurito 2.0

**Objetivo.** Comprobar cómo cambia el desempeño de Neurito al entrenar su red neuronal y al modificar su tamaño, la tasa de aprendizaje y el momento en que debe saltar.

## Cómo hice la prueba

Entrené cada configuración con 5.000 ejemplos sintéticos y medí la precisión con 1.500 ejemplos distintos. Para comparar el desempeño en el juego, probé 30 obstáculos por medición con las mismas velocidades para todos los modelos. Usé la física y la detección de colisiones del juego en modo automático, sin abrir la ventana. Los resultados corresponden a una ejecución por configuración.

## 1. ¿Qué pasó sin entrenar?

Neurito **chocó 30 veces y no esquivó ningún obstáculo**. Su precisión inicial fue del 45,00 %. En esta etapa la red aún tenía sus pesos iniciales.

## 2. ¿Qué cambió cada 100 épocas?

Usé 12 neuronas ocultas, una tasa de aprendizaje de 0,8 y una ventana de salto de 7 a 13 fotogramas.

| Épocas | Pérdida | Precisión de prueba | Esquivados | Choques |
| ---: | ---: | ---: | ---: | ---: |
| 0 | 0,7118 | 45,00 % | 0 | 30 |
| 100 | 0,2547 | 91,33 % | 28 | 2 |
| 200 | 0,2269 | 92,20 % | 30 | 0 |
| 300 | 0,2072 | 92,33 % | 30 | 0 |
| 400 | 0,1847 | 92,47 % | 30 | 0 |
| 500 | 0,1545 | 92,67 % | 30 | 0 |
| 600 | 0,1219 | 95,07 % | 30 | 0 |
| 700 | 0,0970 | 96,07 % | 30 | 0 |
| 800 | 0,0805 | 97,13 % | 30 | 0 |
| 900 | 0,0691 | 97,53 % | 30 | 0 |
| 1.000 | 0,0610 | 98,07 % | 30 | 0 |
| 1.100 | 0,0547 | 98,40 % | 30 | 0 |
| 1.200 | 0,0498 | 98,60 % | 30 | 0 |

A las 200 épocas ya esquivaba los 30 obstáculos de la prueba. Seguir entrenando redujo la pérdida y mejoró la precisión sobre ejemplos nuevos, mientras el resultado en la pista se mantuvo.

## 3. ¿Qué pasó al cambiar las neuronas ocultas?

Después de 1.200 épocas, obtuve **93,33 %** de precisión con 3 neuronas, **98,60 %** con 12 y **98,80 %** con 24. Las tres redes esquivaron los 30 obstáculos. En esta ejecución, pasar de 12 a 24 neuronas mejoró la precisión por solo 0,20 puntos porcentuales y produjo el mismo resultado en la pista.

## 4. ¿Qué efecto tuvo la tasa de aprendizaje?

| Tasa | Pérdida a 100 épocas | Pérdida a 400 épocas | Pérdida a 1.200 épocas | Precisión final | Esquivados / choques |
| ---: | ---: | ---: | ---: | ---: | ---: |
| 0,01 | 0,6455 | 0,5390 | 0,4085 | 87,07 % | 13 / 17 |
| 0,8 | 0,2547 | 0,1847 | 0,0498 | 98,60 % | 30 / 0 |
| 1,2 | 0,2610 | 0,1471 | 0,0332 | 99,13 % | 30 / 0 |

Con 0,01 los ajustes fueron más lentos. Con 1,2 la pérdida era ligeramente mayor que con 0,8 a las 100 épocas, pero fue menor a las 400 y a las 1.200. Ambas tasas terminaron sin choques en esta prueba.

## 5. ¿Qué ventanas funcionaron con la física del juego?

Cambié el intervalo en la regla de etiquetado y también en la generación de ejemplos positivos. Probé tanto la red entrenada como la regla aplicada directamente.

| Ventana de salto | Precisión de la red frente a sus etiquetas | Red: esquivados / choques | Regla directa: esquivados / choques |
| --- | ---: | ---: | ---: |
| 7 a 13 fotogramas | 98,60 % | 30 / 0 | 30 / 0 |
| 9 a 15 fotogramas | 98,33 % | 30 / 0 | 30 / 0 |
| 2 a 5 fotogramas | 96,73 % | 30 / 0 | 30 / 0 |
| 30 a 36 fotogramas | 98,40 % | 0 / 30 | 0 / 30 |

Las primeras tres ventanas fueron compatibles con la física para las velocidades probadas. Con la última, Neurito saltó demasiado pronto y ya estaba demasiado bajo cuando llegó el obstáculo. La regla directa también chocó las 30 veces.

## Pregunta central

**¿Por qué Neurito puede tener buena precisión y aun así chocar?**

La precisión cuenta cuántas predicciones coinciden con las etiquetas. Un choque depende de que Neurito actúe en el momento adecuado. Con la ventana de 30 a 36 fotogramas, la red acertó el 98,40 % de las etiquetas y chocó las 30 veces porque las etiquetas enseñaban a saltar demasiado pronto. Por eso evalué tanto los aciertos en los datos como los resultados en la pista.
