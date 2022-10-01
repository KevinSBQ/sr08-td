# sr08-td

## Exercice 2 : BD

1. Récupérer une image postgres (SGBD)
```
docker pull postgres
```

2. Lancer un conteneur BD (En suivant les indications disponibles sur le lien Docker Hub de l’image)
```
docker run --name mypsql -p 5432:5432 -e POSTGRES_PASSWORD=MyPassword -d postgres
```
Le container mypsql a été crée et est en cours d'execution.

3. Lancer un conteneur adminer (database manager)
```
docker pull adminer
docker run --name myadminer --link mypsql:db -p 8080:8080 adminer
```
Le container myadminer a été crée et est en cours d'execution.

4. Connecter à votre base de données via adminer
Accéder au navigateur, aller à http://localhost:8080, puis saisir les champs :
| Système | PostgreSQL |
| ------- | ------- |
| Serveur | db |
| Utilisateur | postgres |
| Mot de pasee | MyPassword |
| Base de données | postgres |

5. Utiliser adminer pour créer, modifier votre base de données

6. Afficher les logs, la consommation des ressources de vos conteneurs
```
docker logs mypsql
docker stats mypsql
```

7. Donner le fichier docker-compose pour faciliter la configuration de toute cette partie

Copier les codes ci-dessous et les stocker dans un ficher nommé config.yml
```
version: '3.1'
services:
  db:
    image: postgres
    restart: always
    ports:
      - 5432:5432
    environment:
      POSTGRES_PASSWORD: example

  adminer:
    image: adminer
    restart: always
    ports:
      - 8080:8080
```
Lancer la ligne de commande ci-dessous dans la console
```
docker-compose -f config.yml up
```
Le sql est accesible à http://localhost:8080

## Exercice 3 proxy web

1. Proposer une solution de reverse proxy via docker-compose qui permet de

a. créer deux siteweb (simples : web1 et web2) lancés sur deux serveurs web (Nginx ou apache) différents

Installer nginx
```
docker pull nginx
```

Stocker les codes ci-dessous dans un fichier appelé creation_site.yml
```
version: '3.1'
services:
  web1:
    image: nginx 
    volumes:
      - ./web1:/usr/share/nginx/html
  web2:
    image: nginx
    volumes:
      - ./web2:/usr/share/nginx/html
``` 
Lancer la ligne de commande ci-dessous dans la console
```
docker-compose -f creation_site.yml up
```
Deux répertoire : web1 et web2 ont été crée dans le chemin courant.
Créer des fichiers index.html dans ces deux répertoire et mettre des textes dedans, par exemple, dans web1/index.html:
```
Hello World Web1
```
Dans web2/index.html:
```
Hello World Web2
```
Relancer la ligne de commande ci-dessous dans la console pour mettre à jours ces dernières modifications:
```
docker-compose -f creation_site.yml up
```

b. créer un serveur de proxy qui va rediriger les requête vers le site 1 ou site 2 selon l’url : site1.localhost ou site2.localhost.

Créer un répertoire proxy-conf puis créer un ficher default.conf et stocker les codes ci-dessous dedans:
```
server {
    listen 80;
    server_name site1.localhost;

    location / {
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for; proxy_set_header X-Forwarded-Proto $scheme;
        proxy_buffering off;
        proxy_request_buffering off;
        proxy_http_version 1.1;
        proxy_intercept_errors on;
        proxy_passa http://web1:80;
    }
}

server {
    listen 80;
    server_name site2.localhost;

    location / {
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for; proxy_set_header X-Forwarded-Proto $scheme;
        proxy_buffering off;
        proxy_request_buffering off;
        proxy_http_version 1.1;
        proxy_intercept_errors on;
        proxy_pass http://web2:80;
    }
}
```
Stocker les codes ci-dessous dans un fichier appelé creation_proxy.yml (dans le répertoire où se trouve répertoire proxy-conf)
```
version: '3.1'
services:
  proxy:
    image: nginx
    volumes:
      - ./proxy-conf:/etc/nginx/conf.d
    ports:
      - 80:80
``` 
Lancer la ligne de commande ci-dessous dans une autre console (la dernière peut-être bloqué si on ne l'a pas lancé en arrière plan)
```
docker-compose -f creation_proxy.yml up
```
Aller à http://localhost http://site1.localhost et http://site2.localhost pour voir les résultats.