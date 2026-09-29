# bulk-users-creator

Script de Python que registra usuarios de Linux de forma masiva a partir de `usuarios.csv`. Si la cuenta ya existe, actualiza la contraseña. Si no existe, la crea con `useradd`.

Pensado para ejecutarse en el servidor, como root. No es multiplataforma: depende de herramientas y módulos propios de Unix.

## Requisitos

- Linux con `useradd` y `passwd`
- Python 3 (usa el módulo `pwd`, que no existe en Windows)
- Permisos de root

## Uso

1. Coloca `usuarios.csv` en el mismo directorio que `main.py`.
2. Ejecuta:

```bash
sudo python3 main.py
```

El script no pide argumentos. Lee el CSV del directorio de trabajo, muestra una barra de progreso y escribe un log con la fecha y hora de inicio:

```text
2026.09.28-14.30.00.log
```

Si `usuarios.csv` no está, registra el error y termina con código de salida `1`.

Hay una pausa de 0.25 segundos entre cada fila.

## Formato del CSV

Cada línea se parte por comas. Solo se usan dos campos:

| Posición (desde 1) | Índice | Uso |
| --- | --- | --- |
| 4 | 3 | Nombre de usuario |
| 5 | 4 | Contraseña |

El resto de columnas se ignora. No hay fila de cabecera: la primera línea también se procesa como un usuario. Cada línea debe tener al menos cinco campos.

```csv
campo1,campo2,campo3,jperez,contraseña
campo1,campo2,campo3,mlopez,otra-contraseña
```

`*.csv` y `*.log` están en `.gitignore`. No subas el archivo de usuarios al repositorio: contiene contraseñas en texto plano.

## Qué hace con cada fila

1. Comprueba si el usuario existe con `pwd.getpwnam`.
2. Si existe, actualiza la contraseña ejecutando `passwd` y enviando la contraseña dos veces (contraseña y confirmación).
3. Si no existe, ejecuta:

```bash
useradd -p <contraseña> <usuario>
```

No pasa `-m`. El directorio home solo se crea si el sistema lo hace por defecto (`CREATE_HOME` en `/etc/login.defs`). El resto de opciones de la cuenta (shell, grupo, UID) quedan en los valores por defecto de `useradd`.

## Contraseñas en cuentas nuevas

Los dos caminos no tratan la contraseña igual:

- **Usuario que ya existe.** `passwd` recibe el texto plano del CSV y deja una contraseña usable.
- **Usuario nuevo.** `useradd -p` espera la contraseña **ya cifrada**, tal como la devuelve `crypt(3)`, no el texto plano. Con el valor del CSV tal cual, la cuenta se crea pero no se puede iniciar sesión con esa contraseña.

Además, `-p` deja la contraseña visible en la lista de procesos mientras corre `useradd`.

## Registro

Cada ejecución crea un archivo `AAAA.MM.DD-HH.MM.SS.log` en el directorio de trabajo. Ahí queda el inicio del proceso, si cada usuario ya existía, el alta o el fallo al crearlo, y la salida de `passwd` al actualizar una cuenta.
