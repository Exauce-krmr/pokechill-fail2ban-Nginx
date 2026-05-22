<img width="1075" height="935" alt="screenshot-2026-05-22_08-38-23" src="https://github.com/user-attachments/assets/ba1004e4-78b2-47ee-b5ae-7272da4c0b6d" />
<figcaption>Image du jeu en cours de fonctionnement</figcaption>

# Pokechill-Fail2ban-Nginx

## C'est quoi ?
Notre projet consiste à reprendre un projet GitHub (ici, PokeChill) et à y intégrer Fail 2 Ban ainsi que le rate limiting de Nginx.

### Fail 2 Ban
Fail 2 Ban est un outil de sécurité qui analyse les logs afin de détecter des comportements suspects (par exemple: trop de requêtes dans un court laps de temps).
S’il en repère un, il bannit l’adresse IP ayant un comportement suspect pendant une durée déterminée (dans notre cas, 1 heure).

### Nginx Rate Limiting
Le rate limiting de Nginx est également un outil de sécurité.
Il limite le nombre de requêtes d’un utilisateur et, si le quota de requêtes est dépassé, les requêtes de cette adresse IP ne reçoivent plus de réponse.
Cependant, comparé à Fail 2 Ban, le blocage ne dure pas longtemps (moins d’une seconde).

## Architecture du projet

 pokechill-fail2ban-Nginx
├──  app
│   ├──  font
│   │   ├──  Megrim.ttf
│   │   └──  WinkySans.ttf
│   ├──  img
│   │   ├──  bg
│   │   ├──  decor
│   │   ├──  icons
│   │   ├──  items
│   │   ├──  pkmn
│   │   ├──  resources
│   │   ├──  ribbons
│   │   └──  trainers
│   ├──  index.html
│   ├──  scripts
│   │   ├──  areasDictionary.js
│   │   ├──  decor.js
│   │   ├──  dictionarySearch.js
│   │   ├──  explore.js
│   │   ├──  fuse.js
│   │   ├──  'Fuse.js License.txt'
│   │   ├──  HackTimer.js
│   │   ├──  'Hacktimer License.txt'
│   │   ├──  itemDictionary.js
│   │   ├──  moveDictionary.js
│   │   ├──  pkmnDictionary.js
│   │   ├──  PR
│   │   ├──  save.js
│   │   ├──  script.js
│   │   ├──  shop.js
│   │   ├──  teamPreviews.js
│   │   ├──  teams.js
│   │   └──  tooltip.js
│   └──  styles.css
├──  docker-compose.yml
├──  docs
│   ├──  img
│   ├──  setup_fail2ban.md
│   └──  setup_nginx_rateLimiting.md
├──  fail2ban
│   └──  config
│       ├──  fail2ban
│       └──  log
├──  nginx
│   ├──  Dockerfile
│   ├──  logs
│   │   ├──  access.log
│   │   └──  error.log
│   └── 󱁻 nginx.conf
└── 󰂺 README.md

## Comment résaliser ce projet ?

### Prérequis
  - docker
  - git

### Mise en place de l'espace de travail

Créer un répertoire pokechill-fail2ban-Nginx (Nous allons travailler dedans)
```bash
    mkdir pokechill-fail2ban-Nginx
```
Entrez dans ce dossier nouvellement creer :
```bash
    cd pokechill-fail2ban-Nginx
```

Et copier le projet `play-pokechill.github.io` sous le nom de app
```bash
    git clone https://github.com/play-pokechill/play-pokechill.github.io.git app 
```

Toujour dans `pokechill-fail2ban-Nginx` (votre répertoire principale)

Créer les dossiers pour la conf + les logs Nginx et fail2ban
```bash
  mkdir -p nginx/logs fail2ban
```

### Configuration de Nginx

Nous allons créer le fichier de configuration de nginx avec la fonctionalite de rate limiting

```bash
    cd nginx && touch nginx.conf
``` 

Ensuite éditer `nginx.conf` avec votre éditeur préférer. 
_**Le contenue du fichier [ICI](github.com/Exauce-krmr/pokechill-fail2ban-Nginx/blob/main/nginx/nginx.conf)**_

Nous allons aussi creer un Dockerfile pour personaliser l'image de Nginx

```bash
    touch Dockerfile
``` 
_**Le contenue du fichier [ICI](github.com/Exauce-krmr/pokechill-fail2ban-Nginx/blob/main/nginx/nginx.conf)**_

### Création du docker-compose.yml

Creer un docker-compose.yml à la racine du projet (pokechill-fail2ban-Nginx)

```bash
    touch docker-compose.yml
``` 
_**Le contenue du fichier [ICI](github.com/Exauce-krmr/pokechill-fail2ban-Nginx/blob/main/nginx/nginx.conf)**_

Vous pouvez desormais executer le projet

```
    docker compose up 
```
Si sa ne marche pas ragarder [ce fichier](). Vous y trouverer des potentiel solution à ce problème

En faisant cela du contenu devrait apparaitre dans `fail2ban`

Dans le dossier `fail2ban` vous allez trouver un dossier se nommant egalement `fail2ban` et un s'appelant `log`

La configuration ce trouve dans `fail2ban`. Allez dedans.


## Resultat Final
<img width="1075" height="935" alt="screenshot-2026-05-22_08-38-23" src="https://github.com/user-attachments/assets/ba1004e4-78b2-47ee-b5ae-7272da4c0b6d" />
<figcaption>Image du jeu en cours de fonctionnement</figcaption>

### Ban Rate limiting:
<img width="1059" height="985" alt="screenshot-2026-05-22_08-44-43" src="https://github.com/user-attachments/assets/2f571eaa-aeef-46b6-821a-24e7d9f1d41d" />
<figcaption>Image du navigateur après le bannissement par le rate limiting de Nginx</figcaption>

### Ban Fail2Ban:
<img width="928" height="647" alt="screenshot-2026-05-22_09-20-16" src="https://github.com/user-attachments/assets/73737e75-ba51-469d-bcde-70bed781dcb9" />
<figcaption>Image du navigateur après le bannissement par Fail2Ban</figcaption>

### Containers
<img width="1198" height="85" alt="screenshot-2026-05-22_08-36-20" src="https://github.com/user-attachments/assets/1fca5a27-5b9b-4524-9a39-b5d2a747e9b6" />
<figcaption>Image des conteneurs en cours d’exécution</figcaption>
