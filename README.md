# PostgreSQL and PostgREST with Docker Compose: a lab for teaching databases and REST APIs

A ready-made **Postgres + PostgREST** lab for classrooms, from the
[MitsuruTech Learning Platform](https://mitsurutech.co.uk/learning), where schools and
colleges give every student their own databases, containers and terminal in the browser.
Each student gets their own PostgreSQL and their own [PostgREST](https://postgrest.org),
which turns every table in the database into a REST API. They work in a terminal beside the
two, which has `psql` and `curl`.

The whole lab is one `docker-compose` file, `compose.yaml`. It runs the same on your own
machine:

```
docker compose up
```

## Using it on the platform

Make a task of type **Lab**, choose **In a GitHub repository** and press **Use the
sample**, or fork this repository and point the lab at your fork to change it. Publish, and
every student's copy starts from the commit you published. The help manual's
[guide to container labs](https://mitsurutech.co.uk/help/labs) covers the rest: what a
compose file may say, how much memory a copy gets, and how teachers watch and take over a
student's terminal.

No college account yet? [Your first year is free](https://mitsurutech.co.uk/pricing) for up
to 50 students, and [signing up](https://mitsurutech.co.uk/signup) takes an email address.

## The lesson it was written for

Open the lab and, in the terminal (new to the shell? Start with the
[Linux terminal guide](https://mitsurutech.co.uk/help/linux-terminal)):

1. **Talk to the database by name.** Every service is reachable by its name on the lab's
   own network, as it would be in production:

   ```
   psql -h db -U shop
   ```

   The password is `shop`. Make a table and put something in it:

   ```sql
   create table notes (id serial primary key, body text not null);
   insert into notes (body) values ('hello');
   notify pgrst, 'reload schema';
   ```

   The last line asks PostgREST to look at the database again, since it read it once when
   it started and the table did not exist then.

2. **Use the API.** Leave `psql` with `\q`:

   ```
   curl http://api:3000/notes
   curl -X POST -H 'Content-Type: application/json' -d '{"body":"from curl"}' http://api:3000/notes
   curl 'http://api:3000/notes?order=id.desc'
   ```

3. **See what lasts.** Write one file in your home folder and one outside it:

   ```
   echo kept > ~/kept.txt
   echo gone > /tmp/gone.txt
   ```

   Stop the lab and start it again. The notes are still there, because the database keeps
   its files in the named volume `pgdata`, and so is `~/kept.txt`, because the platform saves
   your home folder too. `/tmp/gone.txt` is not: anything outside a volume goes when a
   container stops. That is not a quirk of the platform: it is how containers behave in
   production, and the reason every real deployment decides what goes in a volume.

4. **Back it up and restore it.**

   ```
   pg_dump -h db -U shop shop > ~/backup.sql
   psql -h db -U shop -c 'drop table notes'
   psql -h db -U shop -f ~/backup.sql
   curl http://api:3000/notes
   ```

5. **Open it in your browser.** On the lab's page, under **In your browser**, press **A web
   API over every table** and add `/notes` to the address. It is the same API, reached from
   outside the lab, and only you and your teachers can open it. From Postman or `curl` on
   your own computer, send the lab's token, shown on the same page, in the `X-Lab-Token`
   header.

## What the file shows

| In `compose.yaml` | Why it is there |
| --- | --- |
| `volumes: pgdata` | the only part of `db` that lasts |
| `healthcheck` on `db` | so `api` starts once Postgres answers, not merely once it has started |
| `depends_on: condition: service_healthy` | the other half of that |
| `deploy.resources.limits.memory` | each service's share of the memory one copy may have |
| `expose: 3000` | the API's port, inside the lab's network; nothing is published on the machine |
| `labels: lab.title` | what the lab's page calls each service |
| `labels: lab.open` | a port the lab's page links to, so it opens in a browser |

Images only, nothing built: a lab can also build a service from a `Dockerfile` in its
repository, which the platform does on GitHub when the lab is published.

## More labs

- [lab-git-gitea](https://github.com/mitsurutech/lab-git-gitea): Git and Gitea, to clone,
  commit and push to a Git server of your own and see it in the web interface.
- [lab-mysql-phpmyadmin](https://github.com/mitsurutech/lab-mysql-phpmyadmin): MySQL, with
  phpMyAdmin already signed in to it.

## About

Made by [Mitsuru Technologies](https://mitsurutech.co.uk) for the
[MitsuruTech Learning Platform](https://mitsurutech.co.uk/learning): Python, Java, Go,
Node, SQL and Linux in the browser for schools and colleges, with container labs, web apps
deployed from GitHub, AWS's S3, DynamoDB and SQS with no AWS account, and quizzes that mark
themselves. Try code with no account in the
[playground](https://mitsurutech.co.uk/playground).

## Licence

MIT. Copy it, change it, teach with it.
