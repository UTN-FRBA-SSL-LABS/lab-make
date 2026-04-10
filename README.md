# Laboratorio: Make y Makefile

## Objetivo

Familiarizarse con la herramienta **Make** y la escritura de **Makefiles** para automatizar
la compilación de proyectos en C que usan `gcc`, `flex` y `bison`.

## Requisitos

- `make`
- `gcc`
- `flex`
- `bison`

En Ubuntu/Debian: `sudo apt-get install -y make gcc flex bison`

---

## ¿Qué problema resuelve Make?

Cuando el proyecto crece, compilar a mano se vuelve tedioso y propenso a errores.
Por ejemplo, para compilar un proyecto con Flex y Bison hay que recordar y ejecutar
tres comandos en el orden correcto cada vez que algo cambia:

```bash
bison -d parser.y
flex scanner.l
gcc lex.yy.c parser.tab.c -o mi_programa
```

Make automatiza esto: con un solo comando (`make`) ejecuta los pasos necesarios,
y además es inteligente — si un archivo no cambió desde la última compilación,
**no lo recompila**. Eso ahorra tiempo en proyectos grandes.

---

## Conceptos clave

### El archivo Makefile

Make lee las instrucciones de un archivo llamado `Makefile` (con M mayúscula).
Ese archivo vive en la misma carpeta que el código fuente. Al ejecutar `make` en
esa carpeta, Make lee el Makefile y construye el proyecto.

### Estructura de una regla

La unidad básica de un Makefile es la **regla**:

```makefile
target : dependencia1 dependencia2
	comando1
	comando2
```

- **target**: el nombre del archivo que queremos generar, o el nombre lógico de una acción.
- **dependencias**: los archivos (u otros targets) que deben existir y estar actualizados
  para que este target pueda construirse. Make compara las fechas de modificación: si alguna
  dependencia es más nueva que el target, re-ejecuta los comandos.
- **comandos**: las instrucciones de shell que producen el target. Pueden ser uno o varios.

> **Importante:** los comandos deben ir precedidos por un **tabulador** (`Tab`), no por espacios.
> Este es uno de los errores más frecuentes al escribir Makefiles por primera vez.

**Ejemplo concreto:**

```makefile
suma: suma.c
	gcc suma.c -o suma
```

Esto le dice a Make: _"para tener `suma`, necesito `suma.c`; si `suma.c` cambió (o si
`suma` no existe), ejecutá `gcc suma.c -o suma`"_.

### Variables

Las variables (también llamadas macros) permiten definir un valor una sola vez
y reutilizarlo en varias reglas. Se declaran con `:=` y se usan con `$(NOMBRE)`:

```makefile
CC     := gcc
CFLAGS := -Wall

suma: suma.c
	$(CC) $(CFLAGS) suma.c -o suma
```

La ventaja es inmediata: si mañana queremos compilar con `clang` en lugar de `gcc`,
solo cambiamos una línea (`CC := clang`) y todo el Makefile se actualiza solo.

Las variables más comunes en proyectos C son:

| Variable | Uso convencional |
|----------|-----------------|
| `CC` | El compilador de C a usar (normalmente `gcc`) |
| `CFLAGS` | Flags de compilación (p. ej. `-Wall`, `-g`, `-O2`) |
| `SRCS` | Lista de archivos fuente `.c` |
| `OBJS` | Lista de archivos objeto `.o` |

### El target `all`

Por convención, el primer target del Makefile es el que se ejecuta cuando
escribís simplemente `make` sin argumentos. Se suele llamar `all` y lista
como dependencias todo lo que se quiere construir:

```makefile
all: suma

suma: suma.c
	$(CC) suma.c -o suma
```

### El target `clean`

Por convención, el target `clean` elimina todos los archivos generados por la
compilación (ejecutables, objetos, archivos temporales de Flex/Bison), dejando
el directorio como si nunca hubiéramos compilado:

```makefile
clean:
	rm -f suma lex.yy.c
```

`clean` no tiene dependencias porque no necesita ningún archivo para poder borrarlo.

### `.PHONY`

Make asume que los targets son nombres de archivos. Si existiera un archivo
llamado `clean` en el directorio, Make lo vería como "ya construido" y no
ejecutaría la regla. Para evitar ese problema, declaramos los targets que
**no son archivos reales** con `.PHONY`:

```makefile
.PHONY: all clean
```

Así Make siempre ejecuta esos targets, independientemente de si existe un archivo
con ese nombre.

### Variables automáticas

Cuando tenemos muchos archivos, sería engorroso escribir el nombre de cada uno
a mano en los comandos. Make provee variables automáticas que se calculan en el
contexto de cada regla:

| Variable | Significado |
|----------|-------------|
| `$@` | El nombre del **target** de esta regla |
| `$<` | El **primer** prerequisito (primera dependencia) |
| `$^` | **Todos** los prerequisitos juntos |

Ejemplo: si la regla es `suma: main.o operaciones.o`, entonces dentro del comando:
- `$@` vale `suma`
- `$<` vale `main.o`
- `$^` vale `main.o operaciones.o`

### Reglas de patrón

Cuando tenemos varios archivos `.c` que queremos compilar a `.o`, en lugar de
escribir una regla por cada archivo podemos usar una **regla de patrón** con `%`
como comodín:

```makefile
%.o : %.c
	$(CC) $(CFLAGS) -c $< -o $@
```

El `%` hace matching con cualquier nombre. Si Make necesita construir `main.o`,
busca `main.c` y ejecuta el comando. Si necesita `operaciones.o`, busca `operaciones.c`
y ejecuta el mismo comando. Una sola regla cubre todos los casos.

La flag `-c` le indica a gcc que compile sin linkear (genera el `.o` pero no el ejecutable).

### Sustitución de variables

Para derivar la lista de `.o` a partir de la lista de `.c` automáticamente
se usa la sustitución de sufijos:

```makefile
SRCS := main.c operaciones.c
OBJS := $(SRCS:.c=.o)    # → main.o operaciones.o
```

Esto reemplaza cada `.c` por `.o` en la lista. Así si agregamos un archivo nuevo
a `SRCS`, `OBJS` se actualiza solo.

---

## Ejercicios

---

### Ejercicio 1 — Makefile básico con gcc (25 pts)

En este ejercicio vamos a escribir el Makefile más simple posible: compilar un
único archivo `.c` con `gcc`. El programa `suma.c` ya está completo; tu tarea
es escribir las instrucciones de construcción.

Abrí `ejercicio1/Makefile` y completá los cuatro TODOs.

---

#### TODO 1 — Definir la variable `CC`

```makefile
CC :=
```

`CC` es la variable estándar de Make para el **compilador de C**. Por convención
se llama `CC` (_C Compiler_). Asignale el valor `gcc`.

¿Por qué usar una variable en lugar de escribir `gcc` directamente? Porque si
en otro entorno necesitás usar `clang` o un compilador cruzado, solo cambiás
esta línea y todo el Makefile sigue funcionando sin tocar nada más.

---

#### TODO 2 — Definir la variable `CFLAGS`

```makefile
CFLAGS :=
```

`CFLAGS` (_C Flags_) contiene las opciones que le pasamos al compilador.
Asignale el valor `-Wall`.

`-Wall` activa todos los **warnings** más importantes de gcc. Los warnings no
impiden la compilación, pero señalan código potencialmente problemático
(variables sin usar, comparaciones sospechosas, etc.). Trabajar siempre con
`-Wall` es una buena práctica.

---

#### TODO 3 — Comando de compilación

```makefile
$(PROGRAMA): suma.c
	# Escribí el comando aquí
```

Este es el corazón del Makefile: el comando que convierte `suma.c` en el ejecutable.
Escribí una línea (comenzando con Tab) que invoque al compilador usando las variables
que definiste, el archivo fuente `suma.c`, y el flag `-o $(PROGRAMA)` para que
el ejecutable tenga el nombre correcto.

El comando completo debería verse así:

```
$(CC) $(CFLAGS) suma.c -o $(PROGRAMA)
```

`-o $(PROGRAMA)` le dice a gcc cómo llamar al archivo de salida. Sin este flag,
gcc generaría un ejecutable llamado `a.out` por defecto.

---

#### TODO 4 — Regla `clean`

```makefile
clean:
	# Escribí el comando aquí
```

Escribí el comando que elimina el ejecutable generado. Usá `rm -f $(PROGRAMA)`.

El flag `-f` (_force_) hace que `rm` no dé error si el archivo no existe,
lo cual es conveniente: si nunca compilaste (o ya limpiaste), `make clean`
no falla.

---

**Verificación:**
```bash
cd ejercicio1
make                   # compila suma.c y genera el ejecutable
echo "3 4" | ./suma    # → 3 + 4 = 7
echo "10 5" | ./suma   # → 10 + 5 = 15
make clean             # elimina suma
make clean             # segunda vez: no da error gracias al -f
```

---

### Ejercicio 2 — Makefile con Flex (25 pts)

Hasta ahora teníamos un único archivo `.c` y un paso de compilación. Cuando
usamos Flex, el proceso tiene **dos pasos**: primero Flex genera código C a
partir del scanner, y luego gcc compila ese código C.

El pipeline es:
```
scanner2.l  →[flex]→  lex.yy.c  →[gcc]→  scanner2
```

Abrí `ejercicio2/Makefile` y completá los tres TODOs.

---

#### TODO 1 — Invocar Flex

```makefile
$(PROGRAMA): scanner2.l
	# TODO 1: flex ...
```

El primer paso es ejecutar Flex sobre el archivo `.l`. El comando es simplemente:

```
flex scanner2.l
```

Flex lee `scanner2.l` y genera un archivo llamado `lex.yy.c` con el scanner
en código C. Este archivo no existe antes de correr Flex; Make lo produce
como parte del proceso de construcción.

Notá que `scanner2.l` figura como dependencia del target. Eso significa que
Make solo va a re-ejecutar esta regla si `scanner2.l` fue modificado desde
la última vez que se construyó el target. Si el archivo no cambió, Make
no hace nada.

---

#### TODO 2 — Compilar el código generado por Flex

```makefile
	# TODO 2: $(CC) ...
```

Una vez que Flex generó `lex.yy.c`, el segundo paso es compilarlo con gcc
para obtener el ejecutable. Escribí el comando usando `$(CC)`, el archivo
`lex.yy.c` y el flag `-o $(PROGRAMA)`.

Observá que ambos comandos (TODO 1 y TODO 2) están en la misma regla y se
ejecutan secuencialmente, de arriba a abajo. Eso garantiza que cuando gcc
intente compilar `lex.yy.c`, Flex ya lo habrá generado.

---

#### TODO 3 — Limpiar los archivos generados

```makefile
clean:
	# TODO 3: rm -f ...
```

Ahora hay dos archivos generados que conviene limpiar: el ejecutable `$(PROGRAMA)`
y el código C intermedio `lex.yy.c` que generó Flex. Escribí el comando
`rm -f` con ambos nombres.

Es buena práctica limpiar también los archivos intermedios (no solo el ejecutable)
para que `make` siempre reconstruya todo desde cero cuando se pide.

---

**Verificación:**
```bash
cd ejercicio2
make
echo "42" | ./scanner2     # → Numero: 42
echo "7" | ./scanner2      # → Numero: 7
make clean                 # elimina scanner2 y lex.yy.c
```

---

### Ejercicio 3 — Makefile con Flex + Bison (25 pts)

Este ejercicio reproduce el pipeline completo que usamos en los trabajos prácticos:
Bison genera el parser, Flex genera el scanner, y gcc los compila juntos.

Los archivos `parser3.y` y `scanner3.l` ya están completos e implementan una
calculadora simple. Tu tarea es escribir el Makefile que orquesta todo el proceso.

El pipeline es:
```
parser3.y          →[bison -d]→  parser3.tab.c
                               + parser3.tab.h
scanner3.l         →[flex]→    lex.yy.c
parser3.tab.c
+ lex.yy.c         →[gcc]→     calc3
```

Abrí `ejercicio3/Makefile` y completá los cuatro TODOs.

---

#### TODO 1 — Invocar Bison

```makefile
$(PROGRAMA): parser3.y scanner3.l
	# TODO 1: bison ...
```

El primer paso es procesar el archivo `.y` con Bison. El comando es:

```
bison -d parser3.y
```

La opción **`-d`** (_define_) es clave: le indica a Bison que, además del
archivo C con el parser (`parser3.tab.c`), genere también un **archivo de
cabecera** (`parser3.tab.h`) con las definiciones de los tokens.

¿Por qué hace falta ese header? Porque el scanner (generado por Flex) necesita
conocer los números de token que definió Bison (como `NUM`, `SUMA`, etc.) para
poder retornarlos. El scanner incluye ese `.tab.h` para tener acceso a esas
definiciones. Si no usáramos `-d`, no existiría ese archivo y la compilación fallaría.

---

#### TODO 2 — Invocar Flex

```makefile
	# TODO 2: flex ...
```

El segundo paso es procesar `scanner3.l` con Flex para generar `lex.yy.c`.
El scanner en `scanner3.l` tiene una línea `#include "parser3.tab.h"` — por
eso este paso debe ir **después** del paso de Bison: cuando Flex procesa el
`.l`, el archivo `parser3.tab.h` ya debe existir.

---

#### TODO 3 — Compilar y linkear todo con gcc

```makefile
	# TODO 3: $(CC) ...
```

Ahora tenemos dos archivos C generados: `lex.yy.c` (el scanner) y `parser3.tab.c`
(el parser). Hay que compilarlos **juntos** en un solo comando de gcc para
generar el ejecutable `calc3`. Escribí el comando usando `$(CC)`, ambos archivos
`.c` y el flag `-o $(PROGRAMA)`.

Los dos archivos se compilan juntos porque se referencian mutuamente: el parser
llama a `yylex()` (función del scanner) y el scanner retorna tokens que el parser
definió. Separarlos en dos pasos requeriría compilar con `-c` y luego linkear,
lo cual veremos en el ejercicio 4.

---

#### TODO 4 — Limpiar todos los archivos generados

```makefile
clean:
	# TODO 4: rm -f ...
```

Ahora Bison y Flex generaron cuatro archivos intermedios: `parser3.tab.c`,
`parser3.tab.h`, `lex.yy.c`, y el ejecutable `calc3`. Escribí el comando
`rm -f` para eliminar todos ellos.

---

**Verificación:**
```bash
cd ejercicio3
make
printf '3 + 4\n'  | ./calc3    # → = 7
printf '10 - 3\n' | ./calc3    # → = 7
printf '2 * 5\n'  | ./calc3    # → = 10
make clean
```

---

### Ejercicio 4 — Makefile avanzado: múltiples archivos (15 pts)

En proyectos reales el código está dividido en varios archivos `.c`. La práctica
recomendada es compilar cada `.c` a un archivo objeto `.o` por separado, y luego
**linkear** todos los `.o` juntos en el ejecutable final.

Ventaja principal: si modificás un solo archivo, solo se recompila ese `.c` y
luego se re-linkea. Los demás `.o` se reusan tal cual. En proyectos grandes esto
ahorra mucho tiempo.

El pipeline es:
```
main.c        →[gcc -c]→  main.o
operaciones.c →[gcc -c]→  operaciones.o
main.o + operaciones.o  →[gcc]→  programa
```

Abrí `ejercicio4/Makefile` y completá los cinco TODOs.

---

#### TODO 1 — Definir `SRCS`

```makefile
SRCS :=
```

`SRCS` (_sources_) es la variable que lista todos los archivos fuente `.c` del
proyecto. Asignale los dos archivos: `main.c` y `operaciones.c`.

Esta variable es el punto central de configuración: si el proyecto crece y
agregamos un tercer archivo, solo necesitamos sumarlo aquí.

---

#### TODO 2 — Derivar `OBJS` a partir de `SRCS`

```makefile
OBJS :=
```

`OBJS` (_objects_) debe contener la lista de archivos `.o` correspondientes
a cada `.c` en `SRCS`. En lugar de escribirlos a mano, usá la **sustitución
de sufijos** de Make:

```makefile
OBJS := $(SRCS:.c=.o)
```

Esta sintaxis le dice a Make: _"tomá `SRCS` y reemplazá cada `.c` por `.o`"_.
El resultado es `main.o operaciones.o`. Así, si mañana `SRCS` tiene tres
archivos, `OBJS` se actualiza automáticamente.

---

#### TODO 3 — Comando de linkeo

```makefile
$(PROGRAMA): $(OBJS)
	# Escribí el comando aquí
```

Este target toma todos los archivos `.o` y los linkea en el ejecutable final.
Usá la variable automática **`$^`**, que vale "todos los prerequisitos", es
decir, todos los `.o`. El comando debería ser:

```
$(CC) $^ -o $(PROGRAMA)
```

¿Por qué `$^` y no escribir `main.o operaciones.o` directamente? Porque si
agregamos un archivo a `SRCS`/`OBJS`, este comando sigue siendo correcto sin
modificarlo.

---

#### TODO 4 — Regla de patrón para compilar `.c` → `.o`

```makefile
# Escribí la regla de patrón aquí
```

Esta es la regla más poderosa del ejercicio. En lugar de escribir una regla
por cada archivo fuente, usamos el comodín `%`:

```makefile
%.o : %.c
	$(CC) $(CFLAGS) -c $< -o $@
```

Make interpreta esto como: _"para construir **cualquier** `.o`, buscá el `.c`
del mismo nombre y ejecutá este comando"_.

Desglose del comando:
- `$(CC) $(CFLAGS)` — el compilador con sus flags
- `-c` — compilar sin linkear (producir `.o`, no ejecutable)
- `$<` — el primer prerequisito, es decir, el archivo `.c` que hace match
- `-o $@` — nombrar la salida con el nombre del target, es decir, el `.o`

---

#### TODO 5 — Declarar `.PHONY`

```makefile
# Declarar targets que no son archivos
```

Agregá la declaración `.PHONY` para los targets `all` y `clean`. Si existiera
un archivo llamado `all` o `clean` en el directorio, Make pensaría que esos
targets ya están construidos y no ejecutaría sus reglas. `.PHONY` previene
ese problema indicando explícitamente que son nombres lógicos, no archivos.

---

**Verificación:**
```bash
cd ejercicio4
make
# Deberías ver que compila main.c y operaciones.c por separado, y luego los linkea
echo "5 3" | ./programa
# → Suma: 8
# → Resta: 2
# → Producto: 15

# Modificá operaciones.c (aunque sea agregando un comentario) y volvé a compilar:
# make solo recompilará operaciones.o, no main.o
make clean
```

---

## Preguntas de reflexión

Respondé en este `README.md` reemplazando cada `???` con la opción correcta.

**P1.** Si ejecutás `make` dos veces seguidas sin modificar ningún archivo entre una
ejecución y la otra, ¿qué hace Make en la segunda ejecución?

> Pensá en cómo Make decide si debe recompilar: compara las **fechas de modificación**
> del target con las de sus dependencias. Si el target es más nuevo que todas sus
> dependencias, ya está actualizado.

Opciones: `RECOMPILA` / `NO_RECOMPILA` / `DA_ERROR`

```
P1=???
```

---

**P2.** El target `clean` en los Makefiles de este laboratorio, ¿genera un archivo
llamado `clean`?

> Revisá las reglas `clean` que escribiste: ¿el comando `rm -f` crea algún archivo,
> o solo elimina? ¿Existe algún archivo `clean` después de ejecutar `make clean`?

Opciones: `SI` / `NO`

```
P2=???
```

---

**P3.** ¿Para qué sirve declarar un target como `.PHONY`?

> Imaginá que en tu carpeta existe un archivo llamado `clean`. ¿Qué pasaría si
> ejecutás `make clean` sin tener `.PHONY`? ¿Y con `.PHONY`?

Opciones: `PARA_CREAR_ARCHIVOS` / `PARA_EVITAR_CONFLICTOS_DE_NOMBRES` / `PARA_COMPILAR_MAS_RAPIDO`

```
P3=???
```

---

## Puntaje

| Criterio | Pts |
|----------|----:|
| E1. `make` genera el ejecutable | 5 |
| E1. Suma correcta (3 + 4 = 7) | 5 |
| E1. Suma correcta (10 + 5 = 15) | 5 |
| E1. `make clean` elimina el ejecutable | 5 |
| E1. Makefile usa la variable `$(CC)` | 5 |
| E2. `make` genera el ejecutable | 5 |
| E2. Reconoce número 42 | 5 |
| E2. Reconoce número 7 | 5 |
| E2. `make clean` elimina los archivos generados | 5 |
| E2. Makefile invoca `flex` | 5 |
| E3. `make` genera el ejecutable | 5 |
| E3. Calcula 3 + 4 = 7 | 5 |
| E3. Calcula 10 - 3 = 7 | 5 |
| E3. Calcula 2 * 5 = 10 | 5 |
| E3. Makefile invoca `bison` | 5 |
| E4. `make` genera el ejecutable | 5 |
| E4. Calcula Suma: 8 | 5 |
| E4. Makefile usa regla de patrón `%.o` | 5 |
| P1. Segunda ejecución sin cambios | 4 |
| P2. `clean` no genera un archivo | 3 |
| P3. Para qué sirve `.PHONY` | 3 |
| **Total** | **100** |
