# Lab 3 — Respuestas de comprobación

Issue #3 · branch `chore/3-install-oracle-environment`

## Docker

### 1. ¿Qué diferencia hay entre una imagen y un contenedor? Usa como ejemplo lo que hiciste en los ejercicios G2 y G4.

Una imagen es una plantilla de solo lectura (sistema de archivos + metadatos); un contenedor es una instancia en ejecución de esa imagen, con su propia capa de escritura. En G2, `docker run hello-world` descargó la imagen `hello-world` y creó un contenedor a partir de ella. En G4 lancé `docker run -it --name prueba alpine:3.20 sh`: la imagen `alpine:3.20` es una sola, pero cada `docker run` creó un contenedor distinto (`prueba`, `prueba2`), cada uno con su estado y su nombre.

### 2. En el Ejercicio G5 el archivo nota.txt desapareció y en el G6 no. Explica por qué.

En G5 escribí `/tmp/nota.txt` en la capa de escritura del contenedor `prueba`; al borrarlo con `docker rm prueba` esa capa se destruyó, y `prueba2` arrancó limpio desde la imagen, así que `cat /tmp/nota.txt` no encontró nada. En G6 escribí en `/datos`, que era un volumen (`datos-prueba`) montado con `-v`: el volumen vive fuera del contenedor, así que aunque usé `--rm`, el siguiente contenedor que montó el mismo volumen leyó "dato importante".

### 3. ¿Qué diferencia hay entre docker ps y docker ps -a, y qué significa STATUS = Exited (0)?

`docker ps` muestra solo los contenedores en ejecución; `docker ps -a` muestra todos, incluidos los detenidos. `Exited (0)` significa que el proceso principal del contenedor terminó y lo hizo con código de salida 0, es decir, sin error (como `hello-world` tras imprimir su mensaje).

### 4. En -p 8181:8181, ¿qué número corresponde a tu equipo y cuál al contenedor? ¿Qué pasaría con -p 80:8080 en el ejercicio de nginx?

El formato es `-p HOST:CONTENEDOR`: el primero es el puerto de mi equipo y el segundo el del contenedor. Con `-p 80:8080` en nginx, el equipo escucharía en el 80 y reenviaría al 8080 del contenedor, pero nginx escucha en el 80 dentro del contenedor, así que no habría nada escuchando en el 8080 y la página no cargaría (conexión rechazada/reset). Además, el 80 del host es un puerto privilegiado. Lo correcto fue `-p 8080:80`.

### 5. ¿Por qué un contenedor de Oracle se queda en marcha y el de hello-world termina solo?

Un contenedor vive mientras vive su proceso principal (PID 1). En `hello-world` ese proceso imprime un mensaje y termina, así que el contenedor pasa a Exited. En Oracle, el proceso principal arranca la instancia y se queda en primer plano supervisando la base de datos indefinidamente, por eso el contenedor sigue `Up`.

### 6. ¿Qué es el digest de una imagen y por qué lo registramos si ya sabemos que usamos :latest?

El digest (`sha256:...`) es el hash del contenido de la imagen: identifica exactamente esos bytes y es inmutable. Una etiqueta como `:latest` es móvil: Oracle la reasigna a cada nueva versión. Registrar el digest (en mi caso `sha256:f988b0c0…`) permite saber qué versión exacta se usó y reproducir el entorno idéntico más adelante, aunque `:latest` ya apunte a otra.

### 7. ¿Qué comando borraría realmente los datos de Oracle? ¿Por qué docker rm oralab-26ai no lo hace?

`docker volume rm oralab-26ai-data` (o `docker rm -v` / `docker volume prune` si el volumen queda huérfano). Los datos de la base viven en el volumen con nombre `oralab-26ai-data`, montado en `/opt/oracle/oradata`; `docker rm oralab-26ai` solo elimina el contenedor y su capa de escritura, y el volumen con nombre se conserva, por lo que un contenedor nuevo con el mismo volumen recupera la base.

## Git, organización y evidencia

### 8. ¿Por qué este laboratorio se hace dentro del repositorio oracle-database-lab, con Issue, branch y Pull Request, en vez de en una carpeta aparte?

Porque preparar un entorno es un cambio de infraestructura más y se trata como tal: el Issue (#3) documenta el porqué, la branch aísla el trabajo de `main` (protegida), cada commit registra un paso verificable con su evidencia y el Pull Request permite que otra persona lo revise antes de aceptarlo. Así el entorno es reproducible, auditable y queda trazado para el resto del curso.

### 9. ¿Qué diferencia hay entre source 00-config.sh y bash 00-config.sh? ¿Por qué usamos source?

`bash 00-config.sh` lo ejecuta en un proceso hijo: las variables y la función `ts` se definen allí y desaparecen al terminar. `source` lo ejecuta en la shell actual, así que `CONT_NAME`, `EVID`, `ts`, etc. quedan disponibles en la terminal y en los comandos siguientes. Lo usamos porque 00-config.sh no hace nada por sí mismo: solo define constantes para reutilizarlas.

### 10. Explica cada parte del nombre 20260915T091230Z_02-docker.script.log.

`20260915` = fecha (15/09/2026); `T` = separador fecha-hora de ISO 8601; `091230` = hora 09:12:30; `Z` = zona UTC (Zulu), sin ambigüedad de husos horarios; `_02` = número del paso en el mapa de evidencias; `docker` = descripción del paso; `.script.log` = tipo de evidencia (salida de terminal capturada; `.spool.log` sería una salida SQL). Ordenar alfabéticamente equivale a ordenar cronológicamente.

### 11. ¿Para qué sirve .gitattributes y qué error evita?

Normaliza los finales de línea: con `eol=lf` para `.sh`, `.sql` y `.md`, Git guarda y extrae esos archivos siempre con LF, independientemente del sistema. Evita el error `$'\r': command not found` al ejecutar en Linux un script guardado con finales CRLF de Windows, y diffs falsos en los que "cambian" todas las líneas.

### 12. ¿Por qué en este Pull Request elegimos Create a merge commit en lugar de Squash and merge?

Porque aquí cada commit corresponde a una Parte del laboratorio y tiene valor por sí mismo: muestra cuándo y cómo se verificó cada herramienta, junto con su evidencia. Squash los fundiría en un solo commit y se perdería ese historial paso a paso; el merge commit lo conserva en `main`.

## Seguridad

### 13. Describe las cuatro capas de la estrategia de contraseñas (Parte D) y qué pasaría si te saltas la primera.

1) `.gitignore` incluye `config/.env` (y `backups/`) antes de crear el archivo, para que Git nunca lo rastree. 2) `config/.env.example`: plantilla versionada con nombres de variables y valores ficticios. 3) `config/.env`: archivo local con los valores reales, nunca versionado. 4) Los scripts leen la contraseña con `set -a; source config/.env; set +a`, sin escribirla nunca a mano. Si te saltas la 1, un `git add .` incluiría `config/.env` y la contraseña real quedaría en el historial y, tras el push, publicada.

### 14. ¿Por qué no escribimos la contraseña directamente en el comando docker run, aunque el script no se suba a Git?

Porque el comando tecleado queda en el historial de la shell (`~/.zsh_history`, `~/.bash_history`), aparece en la evidencia grabada con `script`/`tee` (que sí se versiona), en capturas de pantalla y en la lista de procesos. Además el script dejaría de ser reutilizable y revisable. Leerla desde `config/.env` mantiene el secreto fuera de todos esos sitios.

### 15. Si descubres tu contraseña en un commit ya publicado, ¿basta con borrarla en un commit nuevo? ¿Qué debes hacer?

No: el commit antiguo sigue en el historial y cualquiera puede verlo con `git log -p`, además de clones o forks ya hechos. Hay que dar el secreto por comprometido y rotarlo de inmediato (cambiar la contraseña en la base y en `config/.env`), y después limpiar el historial (p. ej. `git filter-repo`) con ayuda del docente/equipo, avisando a quien tenga copias.

## Oracle y herramientas

### 16. ¿Por qué no usamos SPOOL ni @archivo.sql con sqlplus dentro del contenedor, y qué hicimos en su lugar?

Porque `sqlplus` se ejecuta dentro del contenedor: SPOOL escribiría el fichero en el sistema de archivos del contenedor, no en el repositorio, y `@archivo.sql` buscaría el archivo dentro del contenedor, donde no existe (SP2-0310). En su lugar enviamos el SQL desde mi equipo por la entrada estándar (`docker exec -i ... sqlplus -s ... < archivo.sql`) y guardamos la salida en el repositorio con `| tee`.

### 17. ¿Qué hace WHENEVER SQLERROR EXIT SQL.SQLCODE al inicio de V000 y V001, y qué pasaría sin esa línea?

Hace que, ante el primer error SQL, SQL*Plus termine inmediatamente devolviendo el código del error como código de salida. Sin ella, SQL*Plus seguiría ejecutando las sentencias siguientes tras un fallo (p. ej. intentaría crear tablas en un usuario que no se creó), dejaría la base a medio provisionar y terminaría con código 0, así que el script que lo llama no se enteraría del error.

### 18. ¿Qué es una migración y por qué V000 y V001 no se deben editar una vez aplicadas?

Una migración es un script versionado y numerado que lleva el esquema de la base de un estado al siguiente, aplicado en orden. Una vez aplicada, forma parte del historial de la base: si se edita, las bases donde ya se ejecutó no reflejarían el cambio y los entornos divergirían (y herramientas como Liquibase/Flyway detectan el checksum distinto). Cualquier cambio se hace en una migración nueva (V002...).

### 19. ¿Por qué en SQL Developer se usa el servicio FREEPDB1 y no FREE ni un SID?

Oracle 26ai usa arquitectura multitenant: `FREE` es el contenedor raíz (CDB$ROOT), donde no se crean datos de aplicación, y `FREEPDB1` es la base conectable (PDB) donde están los 5 esquemas de negocio y ALUMNO. Un SID identifica la instancia, no una PDB, y te llevaría a la raíz. El nombre de servicio `FREEPDB1` conecta directamente a la base de trabajo.

### 20. ¿Qué aporta SQLcl frente a SQL*Plus, y por qué un DBA debe dominar ambas?

SQLcl es la herramienta moderna: autocompletado, historial, formato automático de resultados, conexiones guardadas (`CONNECT -save`), comandos extra (DDL, INFO) e integración con Liquibase; además se instala en el equipo y permite SPOOL local. SQL*Plus está disponible en cualquier servidor Oracle desde hace décadas: cuando solo tienes una terminal en un servidor de producción, es lo que hay. Un DBA necesita la productividad de SQLcl y la disponibilidad universal de SQL*Plus.

## Entorno de trabajo

### 21. ¿Por qué el curso pasa de Git Bash a Ubuntu en WSL 2? Da al menos dos problemas concretos de Git Bash que desaparecen en Ubuntu.

Porque Oracle, Docker y la infraestructura de las empresas corren sobre Linux, y así practicamos con los mismos comandos, rutas y permisos que en producción. Problemas de Git Bash que desaparecen: (1) convierte rutas tipo `/opt/oracle/...` en rutas de Windows y rompe argumentos de `docker exec`; (2) problemas de finales de línea CRLF y permisos de ejecución; (3) no tiene `apt` ni herramientas como `ss`, `tmux` o un `sudo` real. (En mi caso trabajo en macOS, cuya terminal nativa ya es un sistema tipo Unix; las diferencias con Linux que encontré, como la ausencia de `ss` o el grupo `staff`, las adapté en los scripts.)

### 22. ¿Por qué clonamos el repositorio en ~/oracle-database-lab y no trabajamos sobre la carpeta de Windows (/mnt/c/...)? ¿Y por qué recomendamos bash frente a zsh para los scripts del curso?

Porque `/mnt/c` es el disco de Windows accedido a través de una capa de traducción: es mucho más lento, no respeta bien los permisos de Linux (bit de ejecución) y provoca problemas de finales de línea; el sistema de archivos nativo de Linux (`/home`) es rápido y fiel. Para los scripts usamos bash porque es la shell estándar en servidores Linux y en CI, y los scripts llevan `#!/usr/bin/env bash`: así se comportan igual para todos, aunque la terminal interactiva sea zsh (como en macOS). zsh tiene diferencias sutiles (índices de arrays desde 1, expansión de modificadores como `$VAR:s`) que pueden romper un script pensado para bash.

## Control de seguridad antes del push (O.2)

- Contraseñas comprobadas: ORACLE_PWD, APP_USER_PWD y ORDS_PUBLIC_PWD (valores reales de config/.env).
- `git grep -n -F "$PWD"` → 0 coincidencias en archivos versionados, para las tres.
- `git log --all -p | grep -c -F "$PWD"` → 0 en todo el historial, para las tres.
- `config/.env` no está versionado (`git check-ignore -v config/.env` → `.gitignore:1`).
- Capturas 11 y 12 revisadas: el campo de contraseña muestra solo puntos.
- Nota: el contenedor se creó con el valor de ejemplo de la plantilla; se rotaron las contraseñas de SYS, SYSTEM y PDBADMIN con ALTER USER antes de instalar ORDS, de modo que ninguna contraseña real coincide con un valor publicado.
