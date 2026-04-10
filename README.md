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

## Conceptos clave

### ¿Qué es Make?

Make es una herramienta que automatiza la construcción de programas. Lee un archivo
llamado `Makefile` (con la M mayúscula) y determina qué partes del proyecto deben
recompilarse basándose en las **dependencias** entre archivos y sus **fechas de modificación**.
Si nada cambió, Make no hace nada innecesariamente.

### Estructura de una regla

```makefile
target : dependencia1 dependencia2
	comando1
	comando2
```

> **Importante:** los comandos de una regla deben ir precedidos por un **tabulador** (`Tab`),
> no por espacios.

- **target**: nombre del archivo que se quiere generar (o nombre lógico de la acción).
- **dependencia**: archivos o targets que deben existir/estar actualizados antes de ejecutar la regla.
- **comando**: instrucción de shell que se ejecuta para construir el target.

### Variables (macros)

```makefile
CC     := gcc
CFLAGS := -Wall

programa: main.c
	$(CC) $(CFLAGS) main.c -o programa
```

Las variables se referencian con `$(NOMBRE)`. Permiten cambiar el compilador o las flags
en un solo lugar.

### Targets especiales

| Target  | Uso habitual                                  |
|---------|-----------------------------------------------|
| `all`   | Target por defecto; agrupa lo que se construye |
| `clean` | Elimina los archivos generados                 |

### `.PHONY`

Cuando un target **no corresponde a un archivo real** (como `all` o `clean`), se lo declara
con `.PHONY` para evitar conflictos si existiera un archivo con ese nombre:

```makefile
.PHONY: all clean
```

### Variables automáticas

Útiles en reglas de patrón:

| Variable | Significado                                  |
|----------|----------------------------------------------|
| `$@`     | El nombre del target                         |
| `$<`     | El primer prerequisito                       |
| `$^`     | Todos los prerequisitos                      |

### Reglas de patrón

Permiten definir una regla genérica para compilar cualquier `.c` en su `.o`:

```makefile
%.o : %.c
	$(CC) $(CFLAGS) -c $< -o $@
```

---

## Ejercicios

### Ejercicio 1 — Makefile básico con gcc (25 pts)

Completá el archivo `ejercicio1/Makefile` para compilar `suma.c`.

**TODOs:**
- **TODO 1:** Definí la variable `CC` con el valor `gcc`.
- **TODO 2:** Definí la variable `CFLAGS` con el valor `-Wall`.
- **TODO 3:** Escribí el comando de compilación usando `$(CC)`, `$(CFLAGS)`, `suma.c` y `-o $(PROGRAMA)`.
- **TODO 4:** Escribí el comando `rm -f $(PROGRAMA)` en la regla `clean`.

**Verificación manual:**
```bash
cd ejercicio1
make
echo "3 4" | ./suma    # → 3 + 4 = 7
make clean             # elimina el ejecutable
```

---

### Ejercicio 2 — Makefile con Flex (25 pts)

Completá el archivo `ejercicio2/Makefile` para compilar `scanner2.l` usando Flex.

El pipeline es:
```
scanner2.l  →[flex]→  lex.yy.c  →[gcc]→  scanner2
```

**TODOs:**
- **TODO 1:** Ejecutá Flex sobre `scanner2.l` para generar `lex.yy.c`.
- **TODO 2:** Compilá `lex.yy.c` con `gcc` para generar el ejecutable `scanner2`.
- **TODO 3:** Eliminá el ejecutable y `lex.yy.c` en la regla `clean`.

**Verificación manual:**
```bash
cd ejercicio2
make
echo "42" | ./scanner2    # → Numero: 42
make clean
```

---

### Ejercicio 3 — Makefile con Flex + Bison (25 pts)

Completá el archivo `ejercicio3/Makefile` para construir una calculadora a partir de
`scanner3.l` y `parser3.y`. Los archivos fuente ya están completos; solo tenés que
escribir el Makefile.

El pipeline es:
```
parser3.y   →[bison -d]→  parser3.tab.c + parser3.tab.h
scanner3.l  →[flex]→      lex.yy.c
lex.yy.c + parser3.tab.c  →[gcc]→  calc3
```

**TODOs:**
- **TODO 1:** Ejecutá `bison -d` sobre `parser3.y`.
- **TODO 2:** Ejecutá `flex` sobre `scanner3.l`.
- **TODO 3:** Compilá `lex.yy.c` y `parser3.tab.c` con `gcc` para generar `calc3`.
- **TODO 4:** Eliminá todos los archivos generados en la regla `clean`.

**Verificación manual:**
```bash
cd ejercicio3
make
echo "3 + 4" | ./calc3    # → = 7
make clean
```

---

### Ejercicio 4 — Makefile avanzado: múltiples archivos (15 pts)

Completá el archivo `ejercicio4/Makefile` para compilar un proyecto con dos archivos `.c`:
`main.c` y `operaciones.c`. Debés usar **reglas de patrón** y **variables automáticas**.

**TODOs:**
- **TODO 1:** Definí `SRCS` listando `main.c` y `operaciones.c`.
- **TODO 2:** Definí `OBJS` usando sustitución: `$(SRCS:.c=.o)`.
- **TODO 3:** Escribí el comando de linkeo usando `$(CC)`, `$^` y `-o $(PROGRAMA)`.
- **TODO 4:** Escribí la regla de patrón `%.o : %.c` con su comando de compilación usando `$<` y `$@`.
- **TODO 5:** Declará `all` y `clean` como `.PHONY`.

**Verificación manual:**
```bash
cd ejercicio4
make
echo "5 3" | ./programa
# → Suma: 8
# → Resta: 2
# → Producto: 15
make clean
```

---

## Preguntas de reflexión

Respondé en este `README.md` reemplazando cada `???` con la opción correcta.

**P1.** Si ejecutás `make` dos veces seguidas sin modificar ningún archivo,
¿qué hace make en la segunda ejecución?
Opciones: `RECOMPILA` / `NO_RECOMPILA` / `DA_ERROR`

```
P1=???
```

**P2.** El target `clean` en los Makefiles de este laboratorio,
¿genera un archivo llamado `clean`?
Opciones: `SI` / `NO`

```
P2=???
```

**P3.** ¿Para qué sirve declarar un target como `.PHONY`?
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
