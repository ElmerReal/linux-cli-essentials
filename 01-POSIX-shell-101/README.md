# POSIX shell 101

Una de las herramientas mas importantes y temidas por todos los que deseamos
introducirnos al mundo de GNU Linux es la interfaz de linea de comandos. En este
apartado encontrarás ejercicios básicos y prácticos para aprender de las POSIX
Shells.

**NOTA**: Los ejercicios que encuentras acá están enfocados en utilizar los "Shell
built-in commands" en la manera de lo posible. Hay unas pocas excepciones pero
lo haremos notar en su momento.

- [The developer friendly side 👾](#the-developer-friendly-side-)
  - [Donde todos comenzamos](#donde-todos-comenzamos)
  - [Variables](#variables)
  - [Expresiones](#expresiones)
    - [Aritméticas](#aritméticas)
    - [Lógicas](#lógicas)
    - [Interpolación de cadenas](#interpolación-de-cadenas)
  - [Condicionales](#condicionales)
    - [if](#if)
    - [case](#case)
  - [Iteradores](#iteradores)
    - [while / until](#while--until)
    - [for](#for)
  - [Scripts, funciones y parametros](#scripts-funciones-y-parametros)
- [The POSIX world 🪐](#the-posix-world-)
  - [Greeter](#greeter)
  - [Sustitución o expansión de variables](#sustitución-o-expansión-de-variables)
  - [Variables especiales expandidas por la shell](#variables-especiales-expandidas-por-la-shell)
  - [Variables de ambiente](#variables-de-ambiente)
  - [Exportar variables](#exportar-variables)
  - [Manejo de flujos de datos](#manejo-de-flujos-de-datos)
    - [Heredocs](#heredocs)
    - [Leer archivos línea a línea](#leer-archivos-línea-a-línea)
  - [Sustitución de shell y sub-shell](#sustitución-de-shell-y-sub-shell)
  - [Flujo de ejecución](#flujo-de-ejecución)
  - [Manejo de señales](#manejo-de-señales)
  - [Utilidades](#utilidades)
  - [Ejemplos](#ejemplos)
- [Otros recursos para aprender más](#otros-recursos-para-aprender-más)

## The developer friendly side 👾

### Donde todos comenzamos

```sh
# Hola mundo
echo 'Hola mundo'
```

<!-- Roll credits? -->

### Variables

```sh
# Asignación de valor
FOO='bar'
```

```sh
# Acceso al valor
echo $FOO
```

<!-- Las constantes las veremos más adelante -->

### Expresiones

#### Aritméticas

```sh
# Asignar valor númerico a N
let 'N=1+1'
echo $N
# Incrementar N en 1
let N++
echo $N
# Asignar valor númerico a M usando otra notación
M=$(( N + 5 ))
echo $M
# Disminuir M en 1
(( M-- ))
echo $M
# No necesita ser asignado para ser interpretado
echo $(( 4 * 5 ))
echo $(( N * M ))
echo $(( RANDOM % 20 )) # Un número del 1 al 20 al azár
```

Las shell solo manejan arimética de números enteros, por lo cual es recomendable
usar otras aplicaciones para este tipo de operaciones

#### Lógicas

Las POSIX shell no manejan valores lógicos de manera explicita. Es decir no hay
valores `true` o `false`.

En su lugar, se utiliza el valor que retornan otros comandos (exit code) tomando
`0` como éxitoso/verdadero y cualquier otro número como contrario.

#### Interpolación de cadenas

Para las shells todos los valores son cadenas de texto, no existen tipos explícitos.

```sh
FOO='mundo' # Comilla simple ' deshabilita interpolación
echo "Hola ${FOO}" # Comillas dobles " son requeridas para interpolación
```

En realidad esto es "expansion" o "substitution" de variables, pero veremos más
de ello adelante.

### Condicionales

#### if

```sh
FOO='No es una cadena vacía'
if [ -n "${FOO}" ]; then
  echo $FOO
else
  echo 'La variable FOO está vacía'
fi
```

#### case

```sh
FOO=$(( RANDOM % 20 ))
echo $FOO
case "${FOO}" in
  1*)
    echo 'Este número inicia con 1'
    ;;

  2*)
    echo 'Este número inicia con 2'
    ;;

  *) # Default
    echo 'Este número inicia con otro número diferente de 1 o 2'
    ;;
esac
```

### Iteradores

#### while / until

```sh
N=1
while [ $N -le 3 ]; do
  echo "Contador: ${N}"
  N=$(( $N + 1 ))
done
```

```sh
N=1
until [ $N -gt 3 ]; do
  echo "Contador: ${N}"
  N=$(( $N + 1 ))
done
```

#### for

```sh
for i in 'uno' 'dos' 'tres'; do
    echo $i
done

# Lo siguiente corre en shells que no sean sh
ARR=("cat" "dog" "mouse" "frog")
for str in ${ARR[@]}; do
  echo $str
done
```

### Scripts, funciones y parametros

Ver archivo `calculadora.sh`

## The POSIX world 🪐

### Greeter

```sh
FOO=''
echo $FOO

read -p 'Saludar a: ' FOO
echo "Hola ${FOO}"
```

### Sustitución o expansión de variables

Las POSIX shells pueden hacer multiples operaciones sobre los valores que una
variable tiene asignada.

```sh
unset UNSET_VAR
FOO='Hola mundo'

# Valores fallback o checks
echo ${UNSET_VAR-Valor defecto}
echo ${FOO+Sobre escrito cuando no está vacío}

# Substrings
echo "Substring: ${FOO:5}"
echo "Substring: ${FOO:5:1}"

# Longitud de una cadena
echo "Longitud: ${#FOO}. Cadena: ${FOO}"

# Sustitución de cadenas
FOO='1Hola mundo!1!!1!1'
echo ${FOO/1/_}   # Sustituir primer aparición
echo ${FOO//1/_}  # Sustituir todas las apariciones
echo ${FOO/#1/_}  # Sustituir solo si está al inicio
echo ${FOO/%1/_}  # Sustituir solo si está al final
```

No utilizar shell expasions para manejar valores estructurados como `json`, es
mejor delegar esto a una herramienta especializada como
[jq](https://stedolan.github.io/jq/)

### Variables especiales expandidas por la shell

- `~` es equivalente a utilizar la variable de entorno `$HOME`
- `$?` el "status code" más reciente
- `$_` el paramtetro más reciente
- `$@` expande la lista de argumentos que recibió una función o script
- `$#` es la cantidad de argumentos que recibió una función o script
- `$*` expande todos los argumentos que recibió una función o script como solo una cadena
- `$$` Identificador de proceso (PID) de la shell actual
- `$!` Identificador de proceso (PID) del proceso enviado background más reciente.
- `$0` Nombre con el que fue invocado el script actual
- `$IFS` "input field separator" determina el caracter usado para fin de cadena (o campo)

```sh
# Dan el mismo resultado
echo ~
echo $HOME

# Comprobemos diferencia entre $@ y $*
function expand_at () {
  echo "Son: $# argumentos"
  for i in "$@"; do
    echo $i
  done
}

function expand_star () {
  echo "Son: $# argumentos"
  for i in "$*"; do
    echo $i
  done
}

expand_at uno dos tres cuatro cinco
expand_star uno dos tres cuatro cinco
```

### Variables de ambiente

Hay varibles de entorno seteadas por el sistema y otras que se cargan al iniciar
"sesión". Para fines prácticos nos enfocaremos en las más utilizadas

```sh
# Que shell estoy utilizando en este momento
echo $SHELL
# Cual es el HOME de mi actual usuario
echo $HOME
# En que directorio está mi shell apuntando actualmente
echo $PWD
# En donde están todos los binarios que puedo invocar en esta shell
echo $PATH
```

Acerca de `$PATH`, esta variable la utiliza la shell para buscar los
binarios/cli que puede utilizar. Por eso cuando instalamos manualmente un
binario en un path no estandar necesitamos agregarlo a dicha variable para que
la shell lo reconozca.

```sh
# ¿Cuántos "cli" tienes en tu sistema operativo?
ls -1 $(echo $PATH | tr ':' ' ')| sort | uniq | wc -l
```

Ver más en el [manual](https://www.gnu.org/software/bash/manual/html_node/Bourne-Shell-Variables.html)

### Exportar variables

El comando `export` nos permite exportar valores para que otros proceso con el
mismo padre puedan verlo.

```sh
echo 'echo $UNA_VARIABLE' > /tmp/script_temporal.sh
chmod +x /tmp/script_temporal.sh   # Damos permisos de ejecución

# Ejecutamos nuestro script
/tmp/script_temporal.sh

# Ahora exportamos la variable que necesita y ejecutamos de nuevo
export UNA_VARIABLE='Esta variable viene de otro proceso'
/tmp/script_temporal.sh
```

Aclaraciones :
  - `chmod` **no** es un shell built-in.
  - el simbolo `>` lo estamos usando para redireccionar la salida del `echo`, se
    verá más adelante.

### Manejo de flujos de datos

Los procesos en sistemas Unix-like manejan 3 flujos de datos por defecto:

- Standard Input - STDIN
- Standard Output - STDOUT
- Standard Error - STDERR

Las POSIX shell abstraen el manejo de estos flujos por medio de los siguientes simbolos

- `>` redirige STDOUT
- `>>` agrega (append) y redirige STDOUT
- `2>` redirige el STDERR
- `2>&1` converge STDERR hacia STDOUT

- `<` redirige el STDIN
- `|` redirige STDOUT de un proceso hacia el STDIN de otro


```sh
# Por defecto las shell imprimen lo que llege a STDOUT
echo 'Hola mundo'

# Ya no veremos el mensaje al redirigir el STDOUT del echo
echo 'Hola mundo' > /tmp/ejemplo.log
# Podemos hacer "append" al usar >>
echo 'Esta es la línea 2' >> /tmp/ejemplo.log

# Podemos verificar el archivo usando cat
cat /tmp/ejemplo.log

# Estamos imprimiendo el STDIN de la subshell
echo "$(</tmp/ejemplo.log)"

# Estamos redirigiendo el STDOUT del echo hacia el STDIN de tr
echo 'Hola mundo' | tr 'o' '-'
```

Aclaración. `cat` y `tr` **no** son built-in commands.


#### Heredocs

```sh
FOO='Soporta expansiones de variables'
cat << EOF > ./archivo.txt
Esto es un heredoc :D

$FOO
EOF
```

Aclaración. `cat` **no** es un built-in command.

#### Leer archivos línea a línea

```sh
# Generaremos un archivo de ejemplo
ls -1 $(echo $PATH | tr ':' ' ')| sort | uniq | tail -10 > /tmp/comandos.log

# leeremos el archivo de ejemplo línea por línea
while read -r linea; do
  echo "Leído: $linea";
done < /tmp/comandos.log
```

### Sustitución de shell y sub-shell

```sh
# Uso antiguo
FECHA=`date`
echo $FECHA

# Uso sugerido
FECHA=$(date)
echo $FECHA
```

La sub-shell es similar, inclusive en sintaxis `()`. La diferencia es que la
sub-shell actua como solo un proceso, mientras que la sustitución como si fuera
una expasión de variable "especial"

```sh
echo $(date; echo '') | wc -l
(date; echo '' ) | wc -w
```

Aclaraciones:
  - `wc` y `date` **no** son built-in commands
  - el uso de `;` se verá en la siguiente sección

### Flujo de ejecución

Las POSIX shell te permiten manejar el flujo de como se ejecutan los procesos:

- `||` Ejecuta el siguiente proceso, cuando el primero returna un código de estado que no sea `0`
- `&&` Ejecuta el siguiente proceso, si y solo si el primero returna un código de estado `0`
- `;` Ejecuta el siguiente proceso, sin importar el código de estado del primero
- `:` Operador `noop`.


```sh
# Ejecutar el segundo cuando el primero "falle"
(exit 1) || echo ':)'

# Ejecutar el segundo cuando el primero sea exitoso
(exit 0) && echo ':)'

# Ejecutar el segundo sin importar el estado del primero
(exit 1) || echo ':)'

# "nada pasó acá"
:

# Caso especial de :
FOO='valor1'
: ${FOO:=valor2}
echo $?
echo $FOO
```

### Manejo de señales

Las POSIX shell permiten "atrapar" las señales que reciben. Por lo general esto
es útil cuando queremos hacer limpieza de elementos temporales al terminar la
ejecución de nuestro script

```sh
cat << EOF > /tmp/ejemplo-trap.sh
#!/bin/bash

setup_tmp() {
  TMP_DIR=$(mktemp -d)

  cleanup() {
    code=$?
    set +e
    trap - EXIT
    rm -rf "\${TMP_DIR}"
    exit \$code
  }
  trap cleanup INT EXIT

  export TMP_DIR
}

setup_tmp
echo "Can't touch this" > \$TMP_DIR/mc-hammer.log
cd \$TMP_DIR && pwd && ls -talh
EOF

chmod +x /tmp/ejemplo-trap.sh
/tmp/ejemplo-trap.sh
```

Aclaración. `mktemp` **no** es un built-in command.

### Utilidades

Existe una amplia variedad de "utility commands", a continuación listaremos los
de uso frecuente

- `pwd` Imprime el acutla directorio que la shell está usando (working directory)
- `cd` Cambia el "working directory" hacia el que se le pase como argumento
  - `pushd`, `popd` (no soportado por todas las shells) Similar a `cd` pero da la noción de una pila
- `history` Imprime todos los comandos que la shell ha ejecutado hasta el momento
- `alias` Permite crear "alias" a los comandos
- `getops` Es utilizado dentro de los scripts para soportar "banderas"
- `&`, `fg` y `wait` Son utilizados para enviar un proceso a background, listar procesos en background y esperar por que todos los procesos en background terminen respectivamente.
- `help` (bash only)

### Ejemplos

Puede ver estos conceptos en uso en las siguientes utilerías escritas por la
comunidad de Kubernetes Guatemala

- [Bash CNI plugin](https://github.com/adawolfs/CNI-plugin-from-Scratch/blob/main/fhcn-cni)
- [GH Candle - Sincroniza perfiles de GitHub](https://github.com/jossemarGT/gh-candle)


## Otros recursos para aprender más

- [Numeric comparation operators](https://opensource.com/article/19/10/programming-bash-logical-operators-shell-expansions#numeric-comparison-operators)
- [Shell substitutions simply explained](http://mywiki.wooledge.org/CommandSubstitution)
- [Bash manual - Shell expansions](https://www.gnu.org/software/bash/manual/bash.html#Shell-Expansions)
- [Bash manual - Special parameters](https://www.gnu.org/software/bash/manual/html_node/Special-Parameters.html)
- [Bash manual - Bash environment variables](https://www.gnu.org/software/bash/manual/html_node/Bash-Variables.html)
- [POSIX specification up-to-date](https://pubs.opengroup.org/onlinepubs/9699919799/)
- [POSIX shell language specification](https://pubs.opengroup.org/onlinepubs/9699919799/xrat/V4_xcu_chap02.html)
