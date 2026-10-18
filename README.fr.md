# PostgreSQL et PostgREST avec Docker Compose : un lab pour enseigner les bases de données et les API REST

[English](README.md) | [Français](README.fr.md) | [日本語](README.ja.md) | [Español](README.es.md)

Un lab **Postgres + PostgREST** prêt à l’emploi pour la classe, issu de la
[plateforme d’apprentissage MitsuruTech](https://mitsurutech.co.uk/fr/learning), où les écoles et
les établissements donnent à chaque élève ses propres bases de données, conteneurs et terminal dans le navigateur.
Chaque élève dispose de son propre PostgreSQL et de son propre [PostgREST](https://postgrest.org),
qui transforme chaque table de la base en API REST. Il travaille dans un terminal placé à côté
des deux, qui offre `psql` et `curl`.

Tout le lab tient dans un seul fichier `docker-compose`, `compose.yaml`. Il fonctionne de la même
façon sur votre propre machine :

```
docker compose up
```

## L’utiliser sur la plateforme

Créez une tâche « Lab de conteneurs », choisissez « Dans un dépôt GitHub » et appuyez sur « Utiliser
l’exemple », ou dupliquez (fork) ce dépôt et faites pointer le lab vers votre copie pour le modifier. Publiez, et
la copie de chaque élève part du commit que vous avez publié. Le manuel d’aide a
[une page pour ce lab](https://mitsurutech.co.uk/fr/help/lab-database-api) et un
[guide des labs de conteneurs](https://mitsurutech.co.uk/fr/help/labs) qui couvre le reste : ce qu’un
fichier compose peut dire, la mémoire dont dispose une copie, et comment les enseignants regardent et
prennent la main sur le terminal d’un élève.

Pas encore de compte d’établissement ? [La première année est gratuite](https://mitsurutech.co.uk/fr/pricing) jusqu’à
50 élèves, et [l’inscription](https://mitsurutech.co.uk/fr/signup) ne demande qu’une adresse e-mail.

## La leçon pour laquelle il a été écrit

Ouvrez le lab et, dans le terminal (nouveau dans le shell ? Commencez par le
[guide du terminal Linux](https://mitsurutech.co.uk/fr/help/linux-terminal)) :

1. **Parler à la base de données par son nom.** Chaque service est joignable par son nom sur le
   réseau propre du lab, comme il le serait en production :

   ```
   psql -h db -U shop
   ```

   Le mot de passe est `shop`. Créez une table et mettez-y quelque chose :

   ```sql
   create table notes (id serial primary key, body text not null);
   insert into notes (body) values ('hello');
   notify pgrst, 'reload schema';
   ```

   La dernière ligne demande à PostgREST de relire la base, car il l’a lue une seule fois à
   son démarrage et la table n’existait pas à ce moment-là.

2. **Utiliser l’API.** Quittez `psql` avec `\q` :

   ```
   curl http://api:3000/notes
   curl -X POST -H 'Content-Type: application/json' -d '{"body":"from curl"}' http://api:3000/notes
   curl 'http://api:3000/notes?order=id.desc'
   ```

3. **Voir ce qui dure.** Écrivez un fichier dans votre dossier personnel et un autre en dehors :

   ```
   echo kept > ~/kept.txt
   echo gone > /tmp/gone.txt
   ```

   Arrêtez le lab et redémarrez-le. Les notes sont toujours là, parce que la base garde
   ses fichiers dans le volume nommé `pgdata`, et `~/kept.txt` aussi, parce que la plateforme
   enregistre aussi votre dossier personnel. `/tmp/gone.txt` a disparu : tout ce qui est
   en dehors d’un volume s’en va quand un conteneur s’arrête. Ce n’est pas une bizarrerie de la
   plateforme : c’est ainsi que se comportent les conteneurs en production, et la raison pour
   laquelle tout vrai déploiement décide de ce qui va dans un volume.

4. **La sauvegarder et la restaurer.**

   ```
   pg_dump -h db -U shop shop > ~/backup.sql
   psql -h db -U shop -c 'drop table notes'
   psql -h db -U shop -f ~/backup.sql
   curl http://api:3000/notes
   ```

5. **L’ouvrir dans votre navigateur.** Sur la page du lab, sous « Dans votre navigateur », appuyez sur
   **A web API over every table** et ajoutez `/notes` à l’adresse. C’est la même API, atteinte depuis
   l’extérieur du lab, et seuls vous et vos enseignants peuvent l’ouvrir. Depuis Postman ou `curl` sur
   votre propre ordinateur, envoyez le jeton du lab, affiché sur la même page, dans l’en-tête
   `X-Lab-Token`.

## Ce que montre le fichier

| Dans `compose.yaml` | Pourquoi c’est là |
| --- | --- |
| `volumes: pgdata` | la seule partie de `db` qui dure |
| `healthcheck` sur `db` | pour que `api` démarre quand Postgres répond, et pas seulement quand il a démarré |
| `depends_on: condition: service_healthy` | l’autre moitié de cela |
| `deploy.resources.limits.memory` | la part de chaque service dans la mémoire qu’une copie peut avoir |
| `expose: 3000` | le port de l’API, à l’intérieur du réseau du lab ; rien n’est publié sur la machine |
| `labels: lab.title` | comment la page du lab nomme chaque service |
| `labels: lab.open` | un port vers lequel la page du lab fait un lien, pour qu’il s’ouvre dans un navigateur |

Des images seulement, rien de construit : un lab peut aussi construire un service à partir d’un `Dockerfile` de son
dépôt, ce que la plateforme fait sur GitHub quand le lab est publié.

## Autres labs

- [lab-git-gitea](https://github.com/mitsurutech/lab-git-gitea) : Git et Gitea, pour cloner,
  valider (commit) et pousser (push) vers un serveur Git bien à vous et le voir dans l’interface web.
- [lab-mysql-phpmyadmin](https://github.com/mitsurutech/lab-mysql-phpmyadmin) : MySQL, avec
  phpMyAdmin déjà connecté.

## À propos

Réalisé par [Mitsuru Technologies Ltd](https://mitsurutech.co.uk/fr) (société immatriculée en Angleterre et au pays de Galles, numéro 16755264) pour la
[plateforme d’apprentissage MitsuruTech](https://mitsurutech.co.uk/fr/learning) : Python, Java, Go,
Node, SQL et Linux dans le navigateur pour les écoles et les établissements, avec des labs de conteneurs, des applications web
déployées depuis GitHub, S3, DynamoDB et SQS d’AWS sans compte AWS, et des quiz qui se corrigent
tout seuls. Essayez du code sans compte dans le
[bac à sable](https://mitsurutech.co.uk/fr/playground).

## Licence

Cette œuvre est sous licence [CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/). Vous pouvez la partager et l’adapter pour apprendre et enseigner, à condition de citer Mitsuru Technologies Ltd, de ne pas l’utiliser commercialement et de partager tout ce que vous en tirez sous la même licence. Le texte complet est dans le fichier `LICENSE`.
