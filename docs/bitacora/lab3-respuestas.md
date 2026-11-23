# Lab 3 — Respuestas de comprobación

Issue #3 · branch `chore/3-install-oracle-environment`

## Docker

### 1. ¿Qué diferencia hay entre una imagen y un contenedor? Usa como ejemplo lo que hiciste en los ejercicios G2 y G4.

La imagen es como la plantilla o el "molde", y el contenedor es lo que se crea a partir de ella cuando la ejecutas. En el G2, con `docker run hello-world`, Docker se descargó la imagen y creó un contenedor que imprimió el mensaje. En el G4 usé la misma imagen `alpine:3.20` dos veces y me salieron dos contenedores distintos (`prueba` y `prueba2`), así que de una imagen puedes sacar todos los contenedores que quieras.

### 2. En el Ejercicio G5 el archivo nota.txt desapareció y en el G6 no. Explica por qué.

En el G5 guardé `nota.txt` dentro del propio contenedor. Cuando lo borré con `docker rm prueba` se borró todo lo que tenía dentro, y el nuevo contenedor (`prueba2`) empezó de cero, por eso no encontraba el archivo. En el G6 lo guardé en `/datos`, que era un volumen (`datos-prueba`). El volumen está fuera del contenedor, así que aunque el contenedor se borrase (usé `--rm`), al montar el volumen en otro contenedor el archivo seguía ahí.

### 3. ¿Qué diferencia hay entre docker ps y docker ps -a, y qué significa STATUS = Exited (0)?

`docker ps` solo enseña los contenedores que están en marcha, y `docker ps -a` enseña todos, también los parados. `Exited (0)` quiere decir que el contenedor ha terminado y que lo ha hecho bien, sin errores (el 0 es el código de salida). Es lo que pasa con `hello-world` cuando termina de imprimir.

### 4. En -p 8181:8181, ¿qué número corresponde a tu equipo y cuál al contenedor? ¿Qué pasaría con -p 80:8080 en el ejercicio de nginx?

El primer número es el puerto de mi ordenador y el segundo el del contenedor. Con `-p 80:8080` no funcionaría, porque estaría mandando el tráfico al puerto 8080 del contenedor y nginx escucha en el 80. Por eso en el ejercicio usamos `-p 8080:80`: entro por el 8080 de mi equipo y llego al 80 de nginx.

### 5. ¿Por qué un contenedor de Oracle se queda en marcha y el de hello-world termina solo?

Porque un contenedor dura lo mismo que su proceso principal. `hello-world` solo imprime un texto y acaba, así que el contenedor se para. El de Oracle arranca la base de datos y se queda funcionando todo el rato, por eso sigue en `Up`.

### 6. ¿Qué es el digest de una imagen y por qué lo registramos si ya sabemos que usamos :latest?

El digest es un identificador único (`sha256:...`) que depende del contenido exacto de la imagen. `:latest` solo es una etiqueta y va cambiando cada vez que Oracle saca una versión nueva. Si guardamos el digest (el mío es `sha256:f988b0c0…`) sabemos exactamente qué versión usamos, y podríamos volver a montar el mismo entorno aunque `latest` ya sea otra cosa.

### 7. ¿Qué comando borraría realmente los datos de Oracle? ¿Por qué docker rm oralab-26ai no lo hace?

`docker volume rm oralab-26ai-data`. Los datos de la base no están dentro del contenedor, están en el volumen `oralab-26ai-data` (montado en `/opt/oracle/oradata`). `docker rm oralab-26ai` solo borra el contenedor, el volumen se queda, y si creo otro contenedor con ese volumen la base sigue estando ahí.

## Git, organización y evidencia

### 8. ¿Por qué este laboratorio se hace dentro del repositorio oracle-database-lab, con Issue, branch y Pull Request, en vez de en una carpeta aparte?

Para trabajar como se hace en una empresa: instalar el entorno también es un cambio y tiene que quedar registrado. El Issue explica qué hay que hacer, la branch evita tocar `main` directamente, cada commit guarda un paso con su evidencia y el Pull Request sirve para que un compañero lo revise antes de aceptarlo. Así cualquiera puede ver qué hice y repetirlo.

### 9. ¿Qué diferencia hay entre source 00-config.sh y bash 00-config.sh? ¿Por qué usamos source?

Con `bash` el script se ejecuta en otra shell aparte y, cuando termina, las variables desaparecen. Con `source` se ejecuta en la terminal en la que estoy, así que variables como `CONT_NAME` o `EVID` y la función `ts` se quedan disponibles. Usamos `source` porque ese archivo solo sirve para definir esas constantes y usarlas después.

### 10. Explica cada parte del nombre 20260915T091230Z_02-docker.script.log.

`20260915` es la fecha (15 de septiembre de 2026), la `T` separa la fecha de la hora, `091230` es la hora (09:12:30) y la `Z` indica que está en UTC. `02` es el número del paso, `docker` es de qué va el paso y `.script.log` dice que es una salida de terminal (si fuera de SQL sería `.spool.log`). Como empieza por la fecha, al ordenar los archivos quedan en orden cronológico.

### 11. ¿Para qué sirve .gitattributes y qué error evita?

Sirve para que los `.sh`, `.sql` y `.md` se guarden siempre con finales de línea de Linux (LF). Así se evita el error `$'\r': command not found`, que sale cuando ejecutas en Linux un script que tiene finales de línea de Windows (CRLF).

### 12. ¿Por qué en este Pull Request elegimos Create a merge commit en lugar de Squash and merge?

Porque cada commit es una parte del laboratorio y queremos que se vean todos por separado en `main`. Con Squash se juntarían en un solo commit y se perdería el paso a paso.

## Seguridad

### 13. Describe las cuatro capas de la estrategia de contraseñas (Parte D) y qué pasaría si te saltas la primera.

La primera es poner `config/.env` en el `.gitignore` antes de crearlo. La segunda es `config/.env.example`, una plantilla que sí se sube pero con valores de ejemplo. La tercera es el `config/.env` de verdad, con mis contraseñas, que solo está en mi ordenador. La cuarta es que los scripts leen la contraseña de ese archivo (`source config/.env`) y nunca la escribo a mano. Si me salto la primera, al hacer `git add` se podría subir el `.env` y mi contraseña quedaría en GitHub.

### 14. ¿Por qué no escribimos la contraseña directamente en el comando docker run, aunque el script no se suba a Git?

Porque aunque el script no se suba, el comando se queda guardado en el historial de la terminal y también saldría en los logs de evidencia que sí subimos al repo, o en alguna captura. Leyéndola del `.env` no aparece en ningún sitio.

### 15. Si descubres tu contraseña en un commit ya publicado, ¿basta con borrarla en un commit nuevo? ¿Qué debes hacer?

No, porque el commit antiguo sigue en el historial y cualquiera puede verlo con `git log -p`. Lo que hay que hacer es cambiar la contraseña cuanto antes (darla por comprometida) y avisar al profesor para limpiar el historial de la branch.

## Oracle y herramientas

### 16. ¿Por qué no usamos SPOOL ni @archivo.sql con sqlplus dentro del contenedor, y qué hicimos en su lugar?

Porque `sqlplus` se ejecuta dentro del contenedor. El SPOOL guardaría el archivo dentro del contenedor y no en mi repositorio, y `@archivo.sql` buscaría el archivo dentro del contenedor, donde no existe (da el error SP2-0310). Lo que hicimos fue pasarle el SQL por la entrada estándar con `docker exec -i ... < archivo.sql` y guardar la salida con `tee`.

### 17. ¿Qué hace WHENEVER SQLERROR EXIT SQL.SQLCODE al inicio de V000 y V001, y qué pasaría sin esa línea?

Hace que el script se pare en cuanto hay un error SQL y devuelva el código de ese error. Sin esa línea seguiría ejecutando lo demás aunque algo hubiera fallado, y la base podría quedar a medias sin que te dieras cuenta.

### 18. ¿Qué es una migración y por qué V000 y V001 no se deben editar una vez aplicadas?

Una migración es un script numerado que hace un cambio en la base de datos, y se ejecutan en orden. Si cambias una que ya se ha ejecutado, las bases donde ya se aplicó no se enteran del cambio y dejarían de ser iguales. Si hay que cambiar algo, se hace en una migración nueva (V002, por ejemplo).

### 19. ¿Por qué en SQL Developer se usa el servicio FREEPDB1 y no FREE ni un SID?

Porque Oracle tiene un contenedor raíz (`FREE`) y dentro una base de datos conectable (`FREEPDB1`), que es donde están los esquemas de negocio y el usuario ALUMNO. Si me conecto a `FREE` o por SID llego a la raíz, que no es donde trabajamos.

### 20. ¿Qué aporta SQLcl frente a SQL*Plus, y por qué un DBA debe dominar ambas?

SQLcl es más moderno: tiene autocompletado, historial, formatea mejor los resultados, deja guardar conexiones y se integra con Liquibase. SQL*Plus es más antiguo, pero está en cualquier servidor Oracle, así que si solo tienes una terminal en un servidor es lo que vas a tener. Por eso hay que saber usar las dos.

## Entorno de trabajo

### 21. ¿Por qué el curso pasa de Git Bash a Ubuntu en WSL 2? Da al menos dos problemas concretos de Git Bash que desaparecen en Ubuntu.

Porque Oracle, Docker y los servidores de las empresas funcionan en Linux, y así practicamos con lo mismo que usaremos en el trabajo. Dos problemas de Git Bash: convierte rutas como `/opt/oracle` en rutas de Windows y rompe los comandos de `docker exec`, y no tiene `apt` ni herramientas como `tmux` o `ss`. Yo trabajo en macOS, que ya es tipo Unix, así que no tuve que usar WSL; solo tuve que adaptar algún detalle, como usar `lsof` porque en Mac no existe `ss`.

### 22. ¿Por qué clonamos el repositorio en ~/oracle-database-lab y no trabajamos sobre la carpeta de Windows (/mnt/c/...)? ¿Y por qué recomendamos bash frente a zsh para los scripts del curso?

Porque `/mnt/c` es el disco de Windows visto desde Linux: va mucho más lento y da problemas con los permisos y los finales de línea. En el sistema de archivos de Linux todo va más rápido y sin problemas. Los scripts se hacen en bash porque es la shell que hay en casi todos los servidores Linux, así funcionan igual para todos. Me pasó algo relacionado en este laboratorio: en zsh, `$USER:staff` se interpretó de forma rara (zsh trata `:s` como un modificador) y tuve que escribir `${USER}`.

## Control de seguridad antes del push (O.2)

- Comprobé mis tres contraseñas reales de `config/.env` (ORACLE_PWD, APP_USER_PWD y ORDS_PUBLIC_PWD).
- `git grep -n -F` no encuentra ninguna en los archivos del repositorio.
- `git log --all -p | grep -c -F` devuelve 0 para las tres, así que no están en ningún commit.
- `config/.env` no se sube a Git (`git check-ignore -v config/.env` lo confirma).
- Revisé las capturas 11 y 12 y la contraseña solo sale con puntos.
- El contenedor lo había creado con la contraseña de ejemplo de la plantilla, así que antes de instalar ORDS la cambié para SYS, SYSTEM y PDBADMIN con ALTER USER.
