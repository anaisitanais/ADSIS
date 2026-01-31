![Image](https://i.blogs.es/82572c/debian-logo/650_1200.jpg)
# **PRÁCTICA 1 Administración de Sistemas**
## Autores: Santiago Blasco y Anaïs Aldea, 28/1/2026
> 7b7837fa2ed64fd45224bff905643bd3ab32157dcc0ab1dc68aad5b9dc2d4d39  debian-13.3.0-amd64-DVD-1.iso es la imagen ISO que coincide con el disco instalado.
> 
**Pregunta: Busca la llamada al sistema que se emplea para ejecutar un script y explica su funcionamiento.**\
La llamada al sistema es execve().
execve(path, argv, envp) reemplaza el programa que está ejecutando el proceso actual por otro:
- path: ruta del ejecutable
- argv: argumentos
- envp: variables de entorno

Si tiene éxito, no vuelve (el proceso pasa a ser el nuevo programa). Si falla, devuelve -1 y pone errno.
Luego puedes probar a ejecutar el script con ./primer_script.sh y ocurrirá un error.

**Pregunta: ¿A qué se debe el error? ¿Cómo podría arreglarse el problema?**\
El error es un permiso denegado, porque no tenemos permisos de ejecución.
El problema podría solucionarse de 2 maneras:
1. Añadiendo permisos de ejecucución al script con chmod u+x primer_script.sh
2. Pedirle a bash que lo ejecute directamente con /bin/bash primer_script.sh.

**Pregunta: ¿Cuál es la salida del script segundo_script.sh? ¿Por qué aparecen 2 líneas en vez de una?**\
Aparecen dos lineas por como está la iteración, el for no está iterando sobre una “cadena”, sino sobre una lista de palabras.\
En Bash:
```
for i in hola mundo
do
  echo "${i}"
done
```
La parte in hola mundo se interpreta como dos elementos: hola y mundo (se separan por espacios)\
El bucle hace dos iteraciones:\
i=hola -> echo imprime hola y añade un salto de línea\
i=mundo -> echo imprime mundo y añade un salto de línea\
Por eso se ve:
> hola\
mundo
> 
Si quisiera todo en una sola línea, tendría que pasar un único elemento:
```
for i in "hola mundo"; do
  echo "$i"
done
```
o directamente echo "hola mundo".\
**Pregunta: Añade una nueva variable final_cadena que contenga la cadena: “, como estas?” e imprímela dentro del mismo comando echo.**
```
#!/bin/bash
final_cadena=", como estas?"
for i in hola mundo
do
  echo "${i}${final_cadena}"
done
```
Salida: 
> hola, como estas?\
mundo, como estas?
>
**Pregunta: ¿Se produce algún error si añadimos un subdirectorio en el directorio actual?**
```
for i in $(ls)
do
  ...
  cat "${i}"
done
```
Si en el directorio hay un subdirectorio, ls lo listará también, y el bucle intentará hacer:
> cat "nombre_del_directorio"
>  
Eso normalmente da error, porque cat espera archivos; con un directorio dice “Is a directory”.\
**Pregunta: Reescribe el último script utilizando caracters comodín, en lugar de utilizar el comando ls.**
```
#!/bin/bash
for i in *; do
  echo '-------------------'
  echo "Contenido del fichero $i:"
  cat "$i"
done
```
**Pregunta: Crea un fichero que tenga un espacio en su nombre y vuelve a ejecutar los scripts (la versión con “ls” y la versión con caracteres comodín). ¿Qué ocurre si eliminamos las comillas alrededor de ${i}, en cada caso? ¿Se te ocurre alguna explicación?**
> echo "hola" > "mi archivo.txt"
>
Luego dejamos cat $i en ambos scripts.\
Si un nombre tiene espacios (ej. mi archivo.txt), sin comillas Bash lo parte en palabras:
- En la versión con $(ls) cat $i se convierte en cat mi archivo.txt -> cat recibe 2 argumentos (mi y archivo.txt) y falla. Además hay un problema: la salida de ls se separa por espacios/nuevas líneas y
rompe los nombres con espacios incluso antes de llegar a cat, por lo que no imprime correctamente.
- En la versión con *, la salida se imprime correctamente porque * escapa los carácteres especiales (hace \\) de manera automática, por lo que lee el fichero "mi archivo.txt" todo junto e imprime su contenido correctamente.

Si eliminamos las comillas alrededor de ${i} en cada caso:
- Con ls separa las salidas en lineas distintas porque las trata como ficheros diferentes, por lo que tenemos el separador programado "--------------" dividiendo el error que indica que no existen el fichero mi y el fichero archivo.txt.
  ```
  Contenido del fichero mi:
  cat: mi: No existe el fichero o el directorio
  ----------------
  Contenido del fichero archivo.txt:
  cat: archivo.txt: No existe el fichero o el directorio
  ```
- Con * no encuentra el directorio como lo hacía anteriormente, pero llama al archivo en la misma iteración, dividiendo aún así el error en 2. Esto es porque las comillas evitan que se expanda todo menos el "$", así que al quitarlas los nombres se expanden y cat los considera como 2 elementos distintos.
  ```
  -----------------
  Contenido del fichero mi archivo.txt:
  cat: mi: No existe el fichero o el directorio
  cat: archivo.txt: No existe el fichero o el directorio
  ------------------
  ```
  
  
  
