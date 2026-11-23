# Lab 3 — Respuestas de comprobación

Issue #3 · branch `chore/3-install-oracle-environment`

## Docker

### 1. ¿Qué diferencia hay entre una imagen y un contenedor? Usa como ejemplo lo que hiciste en los ejercicios G2 y G4.

La imagen es como una plantilla y el contenedor es lo que se crea al ejecutarla. En el G2, `docker run hello-world` se bajó la imagen y creó un contenedor. En el G4 usé la misma imagen `alpine:3.20` y me salieron dos contenedores distintos, `prueba` y `prueba2`.

### 2. En el Ejercicio G5 el archivo nota.txt desapareció y en el G6 no. Explica por qué.

En el G5 el archivo estaba dentro del contenedor, y al borrarlo se fue con él. En el G6 lo guardé en un volumen, que está fuera del contenedor, así que aunque el contenedor se borró el archivo seguía ahí.

### 3. ¿Qué diferencia hay entre docker ps y docker ps -a, y qué significa STATUS = Exited (0)?

`docker ps` enseña solo los que están encendidos y `docker ps -a` todos, también los parados. `Exited (0)` significa que el contenedor terminó bien, sin errores.

### 4. En -p 8181:8181, ¿qué número corresponde a tu equipo y cuál al contenedor? ¿Qué pasaría con -p 80:8080 en el ejercicio de nginx?

El primero es el de mi ordenador y el segundo el del contenedor. Con `-p 80:8080` no iría, porque nginx escucha en el 80 del contenedor, no en el 8080. Por eso pusimos `-p 8080:80`.

### 5. ¿Por qué un contenedor de Oracle se queda en marcha y el de hello-world termina solo?

Porque `hello-world` solo saca un mensaje y acaba, y el contenedor se para con él. Oracle se queda ejecutando la base de datos todo el rato, así que el contenedor sigue encendido.

### 6. ¿Qué es el digest de una imagen y por qué lo registramos si ya sabemos que usamos :latest?

Es un código único (`sha256:...`) de esa versión exacta de la imagen. `latest` va cambiando con cada versión nueva, así que guardando el digest sabemos cuál usamos de verdad.

### 7. ¿Qué comando borraría realmente los datos de Oracle? ¿Por qué docker rm oralab-26ai no lo hace?

`docker volume rm oralab-26ai-data`. Los datos están en el volumen, no en el contenedor, así que `docker rm` solo borra el contenedor y los datos se quedan.

## Git, organización y evidencia

### 8. ¿Por qué este laboratorio se hace dentro del repositorio oracle-database-lab, con Issue, branch y Pull Request, en vez de en una carpeta aparte?

Para que quede todo registrado y se pueda revisar, como en una empresa. Así no toco `main` directamente, cada paso queda en un commit con su evidencia y un compañero lo revisa antes de aceptarlo.

### 9. ¿Qué diferencia hay entre source 00-config.sh y bash 00-config.sh? ¿Por qué usamos source?

Con `bash` las variables se pierden al terminar el script. Con `source` se quedan en mi terminal y las puedo seguir usando, que es justo lo que queremos con las constantes.

### 10. Explica cada parte del nombre 20260915T091230Z_02-docker.script.log.

Es la fecha (20260915), la hora (091230) en UTC (la `Z`), el número del paso (02), de qué va (docker) y el tipo de log (`.script.log`, salida de terminal).

### 11. ¿Para qué sirve .gitattributes y qué error evita?

Hace que los scripts se guarden con saltos de línea de Linux. Así no sale el error `$'\r': command not found` al ejecutar scripts hechos en Windows.

### 12. ¿Por qué en este Pull Request elegimos Create a merge commit en lugar de Squash and merge?

Para que se vean todos los commits, uno por parte. Con squash se juntarían en uno y se perdería el paso a paso.

## Seguridad

### 13. Describe las cuatro capas de la estrategia de contraseñas (Parte D) y qué pasaría si te saltas la primera.

1) Meter `config/.env` en el `.gitignore`. 2) Tener una plantilla `.env.example` sin contraseñas reales. 3) El `.env` de verdad, solo en mi ordenador. 4) Que los scripts lean la contraseña del `.env` y no escribirla nunca a mano. Si te saltas la primera, puedes acabar subiendo tu contraseña a GitHub.

### 14. ¿Por qué no escribimos la contraseña directamente en el comando docker run, aunque el script no se suba a Git?

Porque se queda en el historial de la terminal y puede salir en los logs de evidencia, que esos sí se suben.

### 15. Si descubres tu contraseña en un commit ya publicado, ¿basta con borrarla en un commit nuevo? ¿Qué debes hacer?

No, porque sigue en el historial. Hay que cambiar la contraseña y avisar al profesor para limpiar la branch.

## Oracle y herramientas

### 16. ¿Por qué no usamos SPOOL ni @archivo.sql con sqlplus dentro del contenedor, y qué hicimos en su lugar?

Porque sqlplus está dentro del contenedor: el SPOOL guardaría el archivo ahí dentro y `@archivo.sql` no encontraría el archivo. Lo que hicimos fue pasarle el SQL con `<` y guardar la salida con `tee`.

### 17. ¿Qué hace WHENEVER SQLERROR EXIT SQL.SQLCODE al inicio de V000 y V001, y qué pasaría sin esa línea?

Para el script en cuanto hay un error. Sin ella seguiría ejecutando aunque algo fallara y la base podría quedar a medias.

### 18. ¿Qué es una migración y por qué V000 y V001 no se deben editar una vez aplicadas?

Es un script numerado que hace cambios en la base, y se ejecutan en orden. Si editas uno ya aplicado, las bases dejan de ser iguales; si hay que cambiar algo se hace otra migración nueva.

### 19. ¿Por qué en SQL Developer se usa el servicio FREEPDB1 y no FREE ni un SID?

Porque `FREEPDB1` es la base donde están nuestros esquemas. `FREE` es el contenedor raíz y ahí no trabajamos.

### 20. ¿Qué aporta SQLcl frente a SQL*Plus, y por qué un DBA debe dominar ambas?

SQLcl es más cómodo: autocompletado, historial y guardar conexiones. SQL*Plus es más básico pero está en todos los servidores Oracle, así que conviene saber usar los dos.

## Entorno de trabajo

### 21. ¿Por qué el curso pasa de Git Bash a Ubuntu en WSL 2? Da al menos dos problemas concretos de Git Bash que desaparecen en Ubuntu.

Porque Oracle y Docker se usan sobre Linux en el trabajo. Git Bash cambia rutas como `/opt/...` a rutas de Windows y no tiene herramientas como `apt` o `tmux`. Yo uso Mac, que ya es parecido a Linux, así que no me hizo falta WSL.

### 22. ¿Por qué clonamos el repositorio en ~/oracle-database-lab y no trabajamos sobre la carpeta de Windows (/mnt/c/...)? ¿Y por qué recomendamos bash frente a zsh para los scripts del curso?

Porque `/mnt/c` va más lento y da problemas con permisos y saltos de línea. Y bash porque es lo que hay en casi todos los servidores, así los scripts funcionan igual para todos. A mí zsh me dio un problema con `$USER:staff` y tuve que poner `${USER}`.

## Control de seguridad antes del push (O.2)

- He buscado mis tres contraseñas del `.env` en el repo con `git grep` y en todo el historial con `git log -p`: 0 resultados.
- `config/.env` no se sube (está en el `.gitignore`).
- En las capturas la contraseña sale con puntos.
- Como el contenedor tenía la contraseña de ejemplo, la cambié antes de instalar ORDS.
