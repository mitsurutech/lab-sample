# A lab: a database, and a web API in front of it

A sample lab for the [MitsuruTech Learning Platform](https://mitsurutech.co.uk). Each
student gets their own Postgres and their own [PostgREST](https://postgrest.org), which
turns every table in the database into a web API. They work in a terminal beside the two,
which has `psql` and `curl`.

The whole lab is one file, `compose.yaml`. It runs the same on your own machine:

```
docker compose up
```

## Using it on the platform

Make a task of type **Lab**, choose **In a GitHub repository** and press **Use the
sample**, or fork this repository and point the lab at your fork to change it. Publish, and
every student's copy starts from the commit you published.

## The lesson it was written for

Open the lab and, in the terminal:

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

3. **Stop the lab and start it again.** The notes are still there, because the database
   keeps its files in the named volume `pgdata`, and named volumes are what the platform
   saves between sessions.

4. **Now change something that is not in a volume.** Anything written anywhere else in a
   container is gone when the lab stops. That is not a quirk of the platform: it is how
   containers behave in production, and the reason every real deployment decides what goes
   in a volume.

## What the file shows

| In `compose.yaml` | Why it is there |
| --- | --- |
| `volumes: pgdata` | the only part of `db` that lasts |
| `healthcheck` on `db` | so `api` starts once Postgres answers, not merely once it has started |
| `depends_on: condition: service_healthy` | the other half of that |
| `deploy.resources.limits.memory` | each service's share of the memory one copy may have |
| `expose: 3000` | the API's port, inside the lab's network; nothing is published on the machine |
| `labels: lab.title` | what the lab's page calls each service |

Images only, nothing built: a lab can also build a service from a `Dockerfile` in its
repository, which the platform does on GitHub when the lab is published.

## Licence

MIT. Copy it, change it, teach with it.
