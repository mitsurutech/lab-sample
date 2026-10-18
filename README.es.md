# PostgreSQL y PostgREST con Docker Compose: un laboratorio para enseñar bases de datos y API REST

[English](README.md) | [Français](README.fr.md) | [日本語](README.ja.md) | [Español](README.es.md)

Un laboratorio **Postgres + PostgREST** listo para el aula, de la
[plataforma de aprendizaje MitsuruTech](https://mitsurutech.co.uk/es/learning), donde las escuelas y
los centros dan a cada alumno sus propias bases de datos, contenedores y terminal en el navegador.
Cada alumno tiene su propio PostgreSQL y su propio [PostgREST](https://postgrest.org),
que convierte cada tabla de la base de datos en una API REST. Trabaja en un terminal junto a
los dos, que tiene `psql` y `curl`.

Todo el laboratorio es un solo archivo `docker-compose`, `compose.yaml`. Funciona igual en tu
propio equipo:

```
docker compose up
```

## Usarlo en la plataforma

Crea una tarea «Laboratorio de contenedores», elige «En un repositorio de GitHub» y pulsa «Usar el
ejemplo», o bifurca (fork) este repositorio y apunta el laboratorio a tu bifurcación para cambiarlo. Publica, y
la copia de cada alumno parte del commit que publicaste. El manual de ayuda tiene
[una página para este laboratorio](https://mitsurutech.co.uk/es/help/lab-database-api) y una
[guía de laboratorios de contenedores](https://mitsurutech.co.uk/es/help/labs) que cubre el resto: lo que un
archivo compose puede decir, cuánta memoria recibe una copia y cómo los profesores ven y toman
el control del terminal de un alumno.

¿Aún no tienes una cuenta de centro? [El primer año es gratis](https://mitsurutech.co.uk/es/pricing) para hasta
50 alumnos, y [registrarse](https://mitsurutech.co.uk/es/signup) solo pide una dirección de correo electrónico.

## La lección para la que se escribió

Abre el laboratorio y, en el terminal (¿es tu primera vez con la shell? Empieza por la
[guía del terminal de Linux](https://mitsurutech.co.uk/es/help/linux-terminal)):

1. **Habla con la base de datos por su nombre.** Cada servicio es accesible por su nombre en la
   red propia del laboratorio, como lo sería en producción:

   ```
   psql -h db -U shop
   ```

   La contraseña es `shop`. Crea una tabla y mete algo en ella:

   ```sql
   create table notes (id serial primary key, body text not null);
   insert into notes (body) values ('hello');
   notify pgrst, 'reload schema';
   ```

   La última línea le pide a PostgREST que vuelva a mirar la base de datos, porque la leyó una
   sola vez al arrancar y entonces la tabla no existía.

2. **Usa la API.** Sal de `psql` con `\q`:

   ```
   curl http://api:3000/notes
   curl -X POST -H 'Content-Type: application/json' -d '{"body":"from curl"}' http://api:3000/notes
   curl 'http://api:3000/notes?order=id.desc'
   ```

3. **Mira qué perdura.** Escribe un archivo en tu carpeta personal y otro fuera de ella:

   ```
   echo kept > ~/kept.txt
   echo gone > /tmp/gone.txt
   ```

   Detén el laboratorio y vuelve a iniciarlo. Las notas siguen ahí, porque la base de datos guarda
   sus archivos en el volumen con nombre `pgdata`, y también `~/kept.txt`, porque la plataforma
   guarda además tu carpeta personal. `/tmp/gone.txt` ya no está: todo lo que queda fuera de un
   volumen desaparece cuando se detiene un contenedor. No es una rareza de la plataforma: así se
   comportan los contenedores en producción, y por eso todo despliegue real decide qué va en un volumen.

4. **Haz una copia de seguridad y restáurala.**

   ```
   pg_dump -h db -U shop shop > ~/backup.sql
   psql -h db -U shop -c 'drop table notes'
   psql -h db -U shop -f ~/backup.sql
   curl http://api:3000/notes
   ```

5. **Ábrelo en tu navegador.** En la página del laboratorio, en «En tu navegador», pulsa
   **A web API over every table** y añade `/notes` a la dirección. Es la misma API, a la que se
   llega desde fuera del laboratorio, y solo tú y tus profesores pueden abrirla. Desde Postman o `curl` en
   tu propio ordenador, envía el token del laboratorio, que se muestra en la misma página, en la
   cabecera `X-Lab-Token`.

## Qué muestra el archivo

| En `compose.yaml` | Por qué está |
| --- | --- |
| `volumes: pgdata` | la única parte de `db` que perdura |
| `healthcheck` en `db` | para que `api` arranque cuando Postgres responde, y no solo cuando ha arrancado |
| `depends_on: condition: service_healthy` | la otra mitad de eso |
| `deploy.resources.limits.memory` | la parte de cada servicio en la memoria que puede tener una copia |
| `expose: 3000` | el puerto de la API, dentro de la red del laboratorio; no se publica nada en el equipo |
| `labels: lab.title` | cómo llama la página del laboratorio a cada servicio |
| `labels: lab.open` | un puerto al que enlaza la página del laboratorio, para que se abra en un navegador |

Solo imágenes, nada que construir: un laboratorio también puede construir un servicio a partir de un `Dockerfile` de su
repositorio, lo que la plataforma hace en GitHub cuando se publica el laboratorio.

## Más laboratorios

- [lab-git-gitea](https://github.com/mitsurutech/lab-git-gitea): Git y Gitea, para clonar,
  hacer commit y push a un servidor Git propio y verlo en la interfaz web.
- [lab-mysql-phpmyadmin](https://github.com/mitsurutech/lab-mysql-phpmyadmin): MySQL, con
  phpMyAdmin con la sesión ya iniciada.

## Acerca de

Hecho por [Mitsuru Technologies Ltd](https://mitsurutech.co.uk/es) (registrada en Inglaterra y Gales, número de empresa 16755264) para la
[plataforma de aprendizaje MitsuruTech](https://mitsurutech.co.uk/es/learning): Python, Java, Go,
Node, SQL y Linux en el navegador para escuelas y centros, con laboratorios de contenedores, aplicaciones web
desplegadas desde GitHub, S3, DynamoDB y SQS de AWS sin cuenta de AWS, y cuestionarios que se corrigen
solos. Prueba código sin cuenta en el
[entorno de pruebas](https://mitsurutech.co.uk/es/playground).

## Licencia

Esta obra está bajo licencia [CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/). Puedes compartirla y adaptarla para aprender y enseñar, si citas a Mitsuru Technologies Ltd, no la usas comercialmente y compartes lo que hagas con ella bajo la misma licencia. El texto completo está en el archivo `LICENSE`.
