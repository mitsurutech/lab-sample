# PostgreSQL und PostgREST mit Docker Compose: ein Lab für den Unterricht zu Datenbanken und REST-APIs

[English](README.md) | [Français](README.fr.md) | [Español](README.es.md) | [日本語](README.ja.md) | [Deutsch](README.de.md)

Ein fertiges Lab mit **Postgres + PostgREST** für den Unterricht, von der
[MitsuruTech Learning Platform](https://mitsurutech.co.uk/de/learning), auf der Schulen und
Colleges allen Lernenden eigene Datenbanken, Container und ein Terminal im Browser geben.
Alle Lernenden erhalten ein eigenes PostgreSQL und ein eigenes [PostgREST](https://postgrest.org),
das jede Tabelle der Datenbank in eine REST-API verwandelt. Sie arbeiten in einem Terminal neben den
beiden, das `psql` und `curl` hat.

Das ganze Lab ist eine einzige `docker-compose`-Datei, `compose.yaml`. Es läuft auf Ihrem eigenen
Rechner genauso:

```
docker compose up
```

## Auf der Plattform verwenden

Erstellen Sie eine Aufgabe vom Typ **Lab**, wählen Sie **In einem GitHub-Repository** und klicken Sie auf **Das Beispiel
verwenden**, oder forken Sie dieses Repository und richten Sie das Lab auf Ihren Fork, um es zu ändern. Veröffentlichen Sie, und
jede Kopie der Lernenden startet von dem Commit, den Sie veröffentlicht haben. Das Hilfehandbuch hat [eine Seite zu diesem Lab](https://mitsurutech.co.uk/de/help/lab-database-api) und einen
[Leitfaden zu Container-Labs](https://mitsurutech.co.uk/de/help/labs), der den Rest erklärt: was in einer
Compose-Datei stehen darf, wie viel Speicher eine Kopie erhält und wie Lehrkräfte das Terminal von Lernenden
beobachten und übernehmen.

Noch kein Konto für Ihr College? [Ihr erstes Jahr ist kostenlos](https://mitsurutech.co.uk/de/pricing) für bis
zu 50 Lernende, und für die [Registrierung](https://mitsurutech.co.uk/de/signup) genügt eine E-Mail-Adresse.

## Die Lektion, für die es geschrieben wurde

Öffnen Sie das Lab und arbeiten Sie im Terminal (neu in der Shell? Beginnen Sie mit dem
[Leitfaden zum Linux-Terminal](https://mitsurutech.co.uk/de/help/linux-terminal)):

1. **Sprechen Sie die Datenbank über ihren Namen an.** Jeder Dienst ist im eigenen Netzwerk des Labs
   über seinen Namen erreichbar, wie er es auch im Produktivbetrieb wäre:

   ```
   psql -h db -U shop
   ```

   Das Passwort ist `shop`. Legen Sie eine Tabelle an und schreiben Sie etwas hinein:

   ```sql
   create table notes (id serial primary key, body text not null);
   insert into notes (body) values ('hello');
   notify pgrst, 'reload schema';
   ```

   Die letzte Zeile bittet PostgREST, sich die Datenbank erneut anzusehen, denn es hat sie beim Start
   einmal gelesen, und damals gab es die Tabelle noch nicht.

2. **Nutzen Sie die API.** Verlassen Sie `psql` mit `\q`:

   ```
   curl http://api:3000/notes
   curl -X POST -H 'Content-Type: application/json' -d '{"body":"from curl"}' http://api:3000/notes
   curl 'http://api:3000/notes?order=id.desc'
   ```

3. **Sehen Sie nach, was bleibt.** Schreiben Sie eine Datei in Ihren Home-Ordner und eine außerhalb davon:

   ```
   echo kept > ~/kept.txt
   echo gone > /tmp/gone.txt
   ```

   Stoppen Sie das Lab und starten Sie es wieder. Die Notizen sind noch da, weil die Datenbank ihre
   Dateien im benannten Volume `pgdata` hält, und `~/kept.txt` ebenfalls, weil die Plattform auch
   Ihren Home-Ordner speichert. `/tmp/gone.txt` dagegen nicht: Alles außerhalb eines Volumes verschwindet, wenn ein
   Container stoppt. Das ist keine Eigenheit der Plattform: So verhalten sich Container auch im
   Produktivbetrieb, und deshalb legt jedes echte Deployment fest, was in ein Volume kommt.

4. **Sichern und wiederherstellen.**

   ```
   pg_dump -h db -U shop shop > ~/backup.sql
   psql -h db -U shop -c 'drop table notes'
   psql -h db -U shop -f ~/backup.sql
   curl http://api:3000/notes
   ```

5. **Öffnen Sie es in Ihrem Browser.** Klicken Sie auf der Seite des Labs unter **In Ihrem Browser** auf **A web
   API over every table** und hängen Sie `/notes` an die Adresse an. Es ist dieselbe API, von
   außerhalb des Labs erreicht, und nur Sie und Ihre Lehrkräfte können sie öffnen. Senden Sie aus Postman oder mit `curl` auf
   Ihrem eigenen Rechner das Token des Labs, das auf derselben Seite angezeigt wird, im Header `X-Lab-Token`.

## Was die Datei zeigt

| In `compose.yaml` | Warum es da ist |
| --- | --- |
| `volumes: pgdata` | der einzige Teil von `db`, der bleibt |
| `healthcheck` bei `db` | damit `api` startet, sobald Postgres antwortet, und nicht schon, sobald es gestartet ist |
| `depends_on: condition: service_healthy` | die andere Hälfte davon |
| `deploy.resources.limits.memory` | der Anteil jedes Dienstes an dem Speicher, den eine Kopie haben darf |
| `expose: 3000` | der Port der API, im Netzwerk des Labs; nichts wird auf der Maschine veröffentlicht |
| `labels: lab.title` | wie die Seite des Labs jeden Dienst nennt |
| `labels: lab.open` | ein Port, auf den die Seite des Labs verlinkt, damit er sich im Browser öffnet |

Nur Images, nichts wird gebaut: Ein Lab kann einen Dienst auch aus einem `Dockerfile` in seinem
Repository bauen, was die Plattform auf GitHub erledigt, wenn das Lab veröffentlicht wird.

## Weitere Labs

- [lab-git-gitea](https://github.com/mitsurutech/lab-git-gitea): Git und Gitea, um auf einen eigenen Git-Server zu klonen,
  zu committen und zu pushen und das Ergebnis in der Weboberfläche zu sehen.
- [lab-mysql-phpmyadmin](https://github.com/mitsurutech/lab-mysql-phpmyadmin): MySQL, mit
  einem bereits angemeldeten phpMyAdmin.

## Über uns

Erstellt von [Mitsuru Technologies Ltd](https://mitsurutech.co.uk/de) (eingetragen in England und Wales, Handelsregisternummer 16755264) für die
[MitsuruTech Learning Platform](https://mitsurutech.co.uk/de/learning): Python, Java, Go,
Node, SQL und Linux im Browser für Schulen und Colleges, mit Container-Labs, aus GitHub bereitgestellten
Web-Apps, S3, DynamoDB und SQS von AWS ohne AWS-Konto und Quizzen, die sich
selbst bewerten. Probieren Sie Code ohne Konto im
[Playground](https://mitsurutech.co.uk/de/playground) aus.

## Lizenz

Dieses Werk steht unter der Lizenz [CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/). Sie dürfen es zum Lernen und Lehren weitergeben und bearbeiten, wenn Sie Mitsuru Technologies Ltd nennen, es nicht kommerziell nutzen und alles, was Sie daraus machen, unter derselben Lizenz weitergeben. Der vollständige Text steht in der Datei `LICENSE`.
