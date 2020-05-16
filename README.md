PAV - P4: reconocimiento y verificación del locutor
===================================================

Obtenga su copia del repositorio de la práctica accediendo a [Práctica 4](https://github.com/albino-pav/P4)
y pulsando sobre el botón `Fork` situado en la esquina superior derecha. A continuación, siga las
instrucciones de la [Práctica 2](https://github.com/albino-pav/P2) para crear una rama con el apellido de
los integrantes del grupo de prácticas, dar de alta al resto de integrantes como colaboradores del proyecto
y crear la copias locales del repositorio.

También debe descomprimir, en el directorio `PAV/P4`, el fichero [db_8mu.tgz](https://atenea.upc.edu/pluginfile.php/3145524/mod_assign/introattachment/0/spk_8mu.tgz?forcedownload=1)
con la base de datos oral que se utilizará en la parte experimental de la práctica.

Como entrega deberá realizar un *pull request* con el contenido de su copia del repositorio. Recuerde
que los ficheros entregados deberán estar en condiciones de ser ejecutados con sólo ejecutar:

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~.sh
  make release
  run_spkid mfcc train test classerr verify verifyerr
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Recuerde que, además de los trabajos indicados en esta parte básica, también deberá realizar un proyecto
de ampliación, del cual deberá subir una memoria explicativa a Atenea y los ficheros correspondientes al
repositorio de la práctica.

A modo de memoria de la parte básica, complete, en este mismo documento y usando el formato *markdown*, los
ejercicios indicados.

## Ejercicios.

### SPTK, Sox y los scripts de extracción de características.

- Analice el script `wav2lp.sh` y explique la misión de los distintos comandos, y sus opciones, involucrados
  en el *pipeline* principal (`sox`, `$X2X`, `$FRAME`, `$WINDOW` y `$LPC`).
  > Sox: Genera una señal de un cierto formato a partir de otra señal con un formato distinto, es decir, te permite canviar de formato una señal.
  
  > x2x: Te permite convertir los datos a distintos formatos.
  
  > Frame: La usamos para dividir las señales en tramas de 'n' muestras y con un desplazamiento de ventana 'm'. Ejemplo comando: sptk frame -l n -p m.
  
  >Window: multiplica las tramas usadas por la ventana de Blackman. Para indicar la longitud de muestras de la ventana usamos sptk window -l n. Siendo n el numero de muestras.
  
  > lpc: Principalmente calcula el num de coeficientes (lpc_order) primeros de predicción lineal.
  

- Explique el procedimiento seguido para obtener un fichero de formato *fmatrix* a partir de los ficheros
  de salida de SPTK (líneas 41 a 47 del script `wav2lp.sh`).
  
  >Primeramente (linea 42-43) se usan todos los comandos analizados anteriormente para procesar la señal '.wav' obteniendo los coeficientes de la predicción lineal que se almacenan en base.lp.
  
  >Seguidamente (lineas 46-47) se define el numero de columnas y el numero de filas. Principalmente lo que hacemos es almacenar en cada columna los diferentes coeficientes calculados de la predicción lineal de cada trama de la señal (que es cada fila). Por lo tanto se define el número de filas como el número de tramas de la señal.

  * ¿Por qué es conveniente usar este formato (u otro parecido)?
  > De esta manera obtenemos los datos ordenados y podemos operar con ellos facilmente.

- Escriba el *pipeline* principal usado para calcular los coeficientes cepstrales de predicción lineal
  (LPCC) en su fichero <code>scripts/wav2lpcc.sh</code>:
  ![](https://github.com/davilin98/P4/blob/Guardia-Linde/imatges/pipeline_lpcc.JPG)
- Escriba el *pipeline* principal usado para calcular los coeficientes cepstrales en escala Mel (MFCC) en
  su fichero <code>scripts/wav2mfcc.sh</code>:
    ![](https://github.com/davilin98/P4/blob/Guardia-Linde/imatges/pipeline_mfcc.JPG)

### Extracción de características.

- Inserte una imagen mostrando la dependencia entre los coeficientes 2 y 3 de las tres parametrizaciones
  para una señal de prueba.
  >LP:
  
  ![](https://github.com/davilin98/P4/blob/Guardia-Linde/imatges/lp_dispersion.png)
  
  >LPCC:
  
  ![](https://github.com/davilin98/P4/blob/Guardia-Linde/imatges/lpcc_dispersion.png)
  
  >MFCC:
  
  ![](https://github.com/davilin98/P4/blob/Guardia-Linde/imatges/mfcc_dispersion_bo.png)
  
  + ¿Cuál de ellas le parece que contiene más información?
  > Observando las graficas podríamos decir que LPCC y MFCC contienen más información ya que son una nube bastante dispersa a causa de que estan menos correlados, es decir, con una componente, nos cuesta más saber el valor de la otra.  
  > En cambio, en el lp si que se puede deducir el valor de un coeficiente conociendo el otro y entonces, uno de ellos aporta muy poca información. 
  > Si tuvieramos que quedarnos con una gráfica, eligiriamos el del MFCC porque es donde mayor dispersión se observa. 
- Usando el programa <code>pearson</code>, obtenga los coeficientes de correlación normalizada entre los
  parámetros 2 y 3, y rellene la tabla siguiente con los valores obtenidos.
  
  > Para el uso del programa Pearson hemos usado este comando : pearson work/$name/BLOCK01/SES017/*$name
  Donde $name indica si queremos lp, lpcc o mfcc

  |                        | LP   | LPCC | MFCC |
  |------------------------|:----:|:----:|:----:|
  | &rho;<sub>x</sub>[2,3] |-0.872|0.1484|0.1705|
  
  + Compare los resultados de <code>pearson</code> con los obtenidos gráficamente.
  > Los coeficientes de Pearson van de [-1,1] y nos sirven para medir el nivel de correlación que hay entre una serie de datos.
  > Los valores del Pearson nos corroboran lo que comentabamos anteriormente ya que contra más pequeño sea el valor, mayor incorrelacion en los datos. Aunque nos sorprende que el valor de LPCC sea inferior al del MFCC. 
- Según la teoría, ¿qué parámetros considera adecuados para el cálculo de los coeficientes LPCC y MFCC?
> El objetivo para el número de coeficientes es poder obtener los parámetros más importantes de la voz con el menor número de coeficientes. 
> Nº de coeficientes para el LPCC entre 8 y 12
> Nº de coeficientes para el MFCC entre 13 y 16.
### Entrenamiento y visualización de los GMM.

Complete el código necesario para entrenar modelos GMM.

- Inserte una gráfica que muestre la función de densidad de probabilidad modelada por el GMM de un locutor
  para sus dos primeros coeficientes de MFCC.
  
  ![](https://github.com/davilin98/P4/blob/Guardia-Linde/imatges/ses015.png)
  
- Inserte una gráfica que permita comparar los modelos y poblaciones de dos locutores distintos (la gŕafica
  de la página 20 del enunciado puede servirle de referencia del resultado deseado). Analice la capacidad
  del modelado GMM para diferenciar las señales de uno y otro.
  
  ![](https://github.com/davilin98/P4/blob/Guardia-Linde/imatges/ses015_ses220.png)
  
  > En esta imagen podemos observar, la gráfica de la función de densidad de los locutores SE0015 (en rojo arriba) y SE220 (en azul abajo), también se muestra la poblacion de los dos locutores. La población roja pertenece al locutor SE0015 y la azul al SE220. 
  
  > Se puede apreciar notariamente la diferencia entre las funciones de densidad de los locutores ya que tienen una forma bastante distinta. 
  
  > En la población las diferencias son menores pero se observa que cuando la función de densidad de un locutor es distinta a la población de este, observamos muchas muestras que no estan donde la función de densidad indica. Esto nos permite diferenciar a los locutores.
  
  > También comentar que la mayor densidad de probabilidad es en la parte central inferior de las gráficas ya que corresponde a los silencios, los cuales no pertenecen a ninguno de los locutores. 

### Reconocimiento del locutor.

Complete el código necesario para realizar reconociminto del locutor y optimice sus parámetros.

- Inserte una tabla con la tasa de error obtenida en el reconocimiento de los locutores de la base de datos
  SPEECON usando su mejor sistema de reconocimiento para los parámetros LP, LPCC y MFCC.

  |                        | LP   | LPCC | MFCC |
  |------------------------|:----:|:----:|:----:|
  | Tasa de error (%)      |11.08 |3.82  |1.66  |
  
  > Los parámetros que hemos usado son :
  
  > LP = 8 coeficientes; LPCC = 8 coeficientes , 12 coef.cepstrales ; MFCC = 16 coeficientes
  
  > Hemos usado lo siguiente para el GMM : Thershold = 0.00001 ; Iteraciones = 40 ; Guassianas = 40 ; Método de inicialización = aleatorio

### Verificación del locutor.

Complete el código necesario para realizar verificación del locutor y optimice sus parámetros.

- Inserte una tabla con el *score* obtenido con su mejor sistema de verificación del locutor en la tarea
  de verificación de SPEECON. La tabla debe incluir el umbral óptimo, el número de falsas alarmas y de
  pérdidas, y el score obtenido usando la parametrización que mejor resultado le hubiera dado en la tarea
  de reconocimiento.
  
            
   > El sistema de verificación del locutor que mejor nos ha funcionado es el MFCC con 16 coeficientes.
   
   > Hemos usado un umbral de 0.0001, 40 iteraciones y 50 gaussianas. 
   
   ![](https://github.com/davilin98/P4/blob/Guardia-Linde/imatges/cost.PNG)
 
### Test final y trabajo de ampliación.

- Recuerde adjuntar los ficheros `class_test.log` y `verif_test.log` correspondientes a la evaluación
  *ciega* final.
  
> Los archivos están dentro de la carpeta work

- Recuerde, también, enviar a Atenea un fichero en formato zip o tgz con la memoria con el trabajo
  realizado como ampliación, así como los ficheros `class_ampl.log` y/o `verif_ampl.log`, obtenidos como
  resultado del mismo.
