# Estructuras_de_datos_y_laboratorio
Laboratorio 1: Matriz de 100.000 x 100.000 en disco

Estudiante: Zareth Mariana Vega Sanchez
Curso: Estructuras de Datos y Laboratorio

Objetivo

Escribir una matriz de 100.000 x 100.000 celdas en disco duro y mostrar la matriz creada, resolviendo tres problemas que aparecen al trabajar con datos que no caben en memoria:

- Consumo excesivo de RAM: la matriz completa (10.000.000.000 celdas) nunca se genera ni se carga entera en memoria.
- Escritura lenta a disco: se evita escribir celda por celda; se escribe por bloques de filas de forma secuencial.
- Optimización en la manipulación, creación, almacenamiento y lectura: el almacenamiento usa el mínimo de espacio posible por celda, y la
  lectura/verificación se hace por acceso aleatorio, sin recorrer el archivo completo.

# Archivos del repositorio
Laboratorio1.ipynb	Script único con la solución: escritura de la matriz en disco y función para mostrarla.
README.md	Este documento.

# Cómo funciona Laboratorio1.ipynb

# 1. Por qué no se puede cargar la matriz en RAM
100.000 x 100.000 = 10.000.000.000 celdas. Incluso con el tipo de dato más pequeño posible (1 bit por celda), la matriz completa pesa:
10.000.000.000 bits / 8 = 1.250.000.000 bytes = 1.16 GB -> Peso final del archivo

Con un tipo de dato normal como int32 (4 bytes) pesaría 37.3 GB. Ningún equipo tiene esa cantidad de RAM libre para mantener la matriz completa como un array en memoria, así que la matriz se trata siempre como un archivo en disco, nunca como una variable completa en RAM.

# 2. Estrategia de escritura: bloques + empaquetado a bits

La función escribir_matriz_bits_separada:

Crea el archivo en disco con np.memmap, que asocia el archivo directamente a un array de NumPy sin necesidad de tenerlo todo cargado en memoria y es el sistema operativo gestiona qué partes están en RAM en cada momento.
Recorre la matriz por bloques de filas con TAMANO_BLOQUE = 1_000, generando y escribiendo solo un bloque a la vez. Esto resuelve el consumo de RAM: en ningún momento existen más de 1.000 filas en memoria.
Cada celda (0 o 1) se empaqueta a nivel de bit con np.packbits, de forma que 8 celdas ocupan 1 solo byte en disco, en vez de 1 byte o 4 bytes por celda. Esto es lo que reduce el archivo de 37 GB a 1.16 GB.
Cada fila termina con un byte separador, es decir 10, el carácter \n en ASCII, lo que permite identificar dónde empieza y termina cada fila dentro del archivo binario.
La escritura se hace por bloque completo, es decir, de forma secuencial y en lotes, no celda por celda. Escribir de a una celda con acceso aleatorio sería mucho más lento en disco; escribir bloques contiguos aprovecha que el acceso secuencial es significativamente más rápido.

Al final, el tamaño del archivo se calcula así:

bytes por fila = (100.000 / 8) + 1 = 12.501 bytes  (12.500 de datos + 1 del separador)
tamaño total   = 100.000 filas x 12.501 bytes = 1.16 GB

Las "dimensiones reales en disco" que imprime el script (100000, 12501) son bytes por fila, no celdas por fila, la matriz lógica sigue siendo 100.000 x 100.000, solo que cada fila de 100.000 valores ocupa 12.501 bytes en el archivo en vez de 100.000 o 400.000.

# 3. Cómo se muestra la matriz creada (verificación del contenido)

La función mostrar_filas_bloques es la forma de comprobar que el archivo generado contiene lo que se espera, sin leer el archivo completo:

Calcula el desplazamiento (offset) exacto de cada fila dentro del archivo: offset = fila * bytes_por_fila_en_disco.
Usa seek() para saltar directamente a esa posición (acceso aleatorio), en vez de recorrer el archivo desde el inicio.
Lee únicamente los bytes de esa fila, ignorando el byte separador.
Desempaqueta los bits de vuelta a valores 0/1 y los imprime separados por coma, una fila completa por línea.

Solo se decodifican unas pocas filas (FILAS_A_MOSTRAR = 5 por defecto), no las 100.000: cada fila completa como texto ocupa 200 KB (100.000 números + comas), así que mostrar las 100.000 filas generaría 20 GB de texto, esto anularía la reducción de tamaño lograda al empaquetar a bits, y no aporta la evidencia suficiente.

Otras formas de verificar el archivo generado:

Tamaño en disco: el propio script imprime os.path.getsize(ruta) / (1024**3) al terminar de escribir, para confirmar que el archivo pesa lo esperado 1.16 GB.
Consultar una celda puntual cualquiera, por ejemplo, la última fila usando la misma lógica de seek + desempaquetado de bits que usa mostrar_filas_bloques, sin necesidad de leer el archivo completo.
