# Prácticas mecanismos avanzados de seguridad #2

## Ejercicio 1

### Detección de anomalías en el acceso al repositorio

Libreta de práctica para los estudiantes del curso CIB-209, Temas Especiales en Seguridad de Datos y Sistemas.

## Aviso sobre los datos

Los datos son sintéticos y no corresponden a accesos reales de ninguna organización. El conjunto viene incluido en el archivo eventos_acceso.csv, junto a la libreta. Los datos ya están generados y no se calculan al ejecutar, por lo que todos los grupos ven exactamente los mismos números.

## ¿Cómo ejecutar en Binder?

1. Abrir el repositorio en Binder con el botón de abajo:

[![Binder](https://mybinder.org/badge_logo.svg)](https://mybinder.org/v2/gh/wrodriguez-cenfotec/ejercicio1_deteccion_anomalias/HEAD?urlpath=%2Fdoc%2Ftree%2Fejercicio1_deteccion_anomalias.ipynb)

2. La primera carga tarda unos minutos mientras se construye el entorno. Tener paciencia.
3. Ejecutar las celdas en orden, de arriba hacia abajo.
4. La sesión de Binder es temporal. Antes de cerrar, guardar las capturas de los resultados.

## ¿Qué técnica se usa y qué conviene recordar?

k-means es aprendizaje automático no supervisado. No se le indica qué es normal ni cuáles accesos son incidentes: agrupa por sí solo los accesos que se parecen entre sí y calcula el centro de cada grupo, que es el acceso promedio de ese grupo. En esta práctica cada grupo se llama perfil de comportamiento.

Sobre ese agrupamiento se mide la distancia de cada acceso al centro de su propio perfil. Mientras más lejos está, menos se parece a los demás accesos de su perfil. El percentil de corte define a partir de qué distancia se considera anomalía: con percentil 95 se marca el 5 por ciento de accesos más lejanos.

Vocabulario que conviene tener claro:

- Perfil: el grupo de accesos parecidos que k-means construyó por sí solo, sin etiquetas.
- Centro del perfil: el valor promedio del grupo en los cinco atributos.
- Distancia al centro: qué tan distinto es un acceso respecto de su propio grupo.
- Escalado de atributos: los cinco atributos se llevan a la misma escala antes de agrupar, para que el volumen en MB no pese más que la hora solo por tener números grandes.
- Anomalía: evento marcado por la distancia. No es lo mismo que un incidente.

## ¿Qué hace la libreta?

- Carga 2000 eventos de acceso a un repositorio documental desde eventos_acceso.csv.
- Construye los perfiles de comportamiento normal con k-means sobre la hora, el volumen descargado, la cantidad de carpetas visitadas, la marca de fuera de horario y la marca de país inusual.
- Marca como anomalía todo evento cuya distancia al centro de su perfil supere el percentil configurado.
- Muestra la tabla de perfiles construidos, el resultado de la corrida, las diez anomalías de mayor distancia y un gráfico de dispersión con los incidentes confirmados y la campaña de respaldo señalados.

## ¿Cómo se lee el gráfico de dispersión?

Un gráfico de dispersión coloca un punto por cada registro, ubicado según dos de sus atributos. Aquí cada punto es un acceso al repositorio: el eje horizontal es la hora del día en que ocurrió, de 0 a 23, y el eje vertical es el volumen descargado en MB. Un punto arriba y a la izquierda es una descarga grande en la madrugada.

Los símbolos son cuatro: punto gris, acceso que la corrida no marcó; punto naranja, acceso marcado como anomalía; círculo rojo sin relleno, uno de los seis incidentes confirmados; cuadro azul sin relleno, uno de los 30 eventos de la campaña de respaldo.

Para leerlo conviene revisar tres cosas: cuántos círculos rojos quedaron pintados de naranja, cuántos puntos naranja caen sobre cuadros azules, y cuántos puntos naranja están dentro de la nube gris, es decir, junto a accesos que nadie considera sospechosos.

Advertencia importante: el modelo trabaja con cinco atributos y el gráfico muestra solo dos. Un punto puede estar marcado por lo que ocurre en los otros tres, por ejemplo el país inusual, y por eso a veces se ve un punto naranja en medio de puntos grises.

## ¿Qué debe modificar el grupo?

Solo la celda CONFIGURACIÓN. Los parámetros son numero_de_perfiles y percentil_de_corte.

| Corrida | numero_de_perfiles | percentil_de_corte |
| --- | --- | --- |
| 1 | 3 | 95 |
| 2 | 5 | 95 |
| 3 | 5 | 99 |

Efecto de cada parámetro. numero_de_perfiles define cuántos comportamientos normales distintos se acepta que existan en la organización: si queda por debajo de la cantidad real de áreas, personas que trabajan de forma legítimamente distinta quedan forzadas al mismo perfil y aparecen como anómalas, de modo que los falsos positivos suben sin que el umbral se haya movido. percentil_de_corte define qué tan lejos del centro debe estar un acceso para ser marcado: subirlo baja el volumen de alertas y la carga del analista, pero deja pasar los incidentes de bajo perfil.

## ¿Qué se entrega?

Hoja de evidencia del ejercicio 1, con los valores de las tres corridas y tres capturas. El detalle está en el documento Practicas_Mecanismos_Avanzados_Seguridas_2.docx de la práctica.

## Referencias

- https://scikit-learn.org/stable/modules/clustering.html
- https://scikit-learn.org/stable/modules/preprocessing.html
- https://pandas.pydata.org/docs/user_guide/groupby.html
- https://matplotlib.org/stable/api/pyplot_summary.html
