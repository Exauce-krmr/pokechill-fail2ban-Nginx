# Pokechill-Fail2ban-Nginx
<img width="1075" height="935" src="docs/img/screenshotGame1.png" />
<figcaption>Image du jeu en cours de fonctionnement sur l'accueil</figcaption>

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

```text
📁 pokechill-fail2ban-Nginx
├── 📁 app
│   ├── 📁 font
│   ├── 📁 img
│   ├── 🌐 index.html
│   ├── 📁 scripts
│   └── 🎨 styles.css
├── 📁 docs
│   ├── 📁 img
│   ├── 📝 setup_fail2ban.md
│   └── 📝 setup_nginx_rateLimiting.md
├── 📁 fail2ban
│   ├── ⚙️ config
│   └── 📂 log
├── 📁 nginx
│   ├── 🐳 Dockerfile
│   ├── 📂 logs
│   └── ⚙️ nginx.conf
├── 🐳 docker-compose.yml
└── 📖 README.md
```

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
  mkdir -p nginx/logs fail2ban/config fail2ban/log
```

### Configuration de Nginx

Nous allons créer le fichier de configuration de nginx avec la fonctionalite de rate limiting

```bash
    cd nginx && touch nginx.conf
``` 

Ensuite éditer `nginx.conf` avec votre éditeur préférer. 
_**Le contenue du fichier [ICI](nginx/nginx.conf)**_

Nous allons aussi creer un Dockerfile pour personaliser l'image de Nginx

```bash
    touch Dockerfile
``` 
_**Le contenue du fichier [ICI](nginx/Dockerfile)**_

### Création du docker-compose.yml

Creer un docker-compose.yml à la racine du projet (pokechill-fail2ban-Nginx)

```bash
    touch docker-compose.yml
``` 
_**Le contenue du fichier [ICI](docker-compose.yml)**_
Vous pouvez desormais executer le projet

```bash
    docker compose up 
```
Si sa ne marche pas ragarder [ce fichier](). Vous y trouverer des potentiel solution à ce problème

En faisant cela du contenu devrait apparaitre dans `fail2ban`

Dans le dossier `fail2ban` vous allez trouver un dossier `config` et un s'appelant `log`

La configuration ce trouve dans `config`. Allez dedans.

Nous allons aller dans le dossier `jail.d`. Il contient tout la configuration par default.

_**INFO :** Tout modification a l'interieur des fichier par defaut se trouvera reinitialiser au redemarrage_

```bash 
    cd fail2ban/config/jail.d
```

Dedans nous allons ajouter notre `nginx-limit-req.local`
```bash
    touch nginx-limit-req.local
```
_**Le contenue du fichier [ICI](fail2ban/config/jail.d/nginx-limit-req.local)**_

Un fois le fichier creer : 

Reloader la configuraton pour l'appliquer _(Attention le projet de être lancé !)_:

```bash
  docker exec -it fail2ban fail2ban-client reload
```

Regader si la config est appliqueé
```bash 
    docker exec -it fail2ban fail2ban-client status nginx-limit-req
```

### Comment savoir si tout marche ?

Pour savoir si votre projet marche vous pouvez le tester avec les moyens si dessous :

#### Nginx rate Limiter 



▌ 572     docker exec -it fail2ban fail2ban-client reload                                                                                      │
▌ 581     docker exec -it fail2ban fail2ban-client status nginx-limit-req                                                                      │
▌ 582     docker exec -it fail2ban fail2ban-client unban --all


## Resultat Final
<img width="1075" height="935" src="docs/img/screenshotGame2.png" />
<figcaption>Image du jeu en cours de fonctionnement sur la page Travel</figcaption>

### Ban Rate limiting:
<img width="1059" height="985" src="docs/img/banRateLimiting.png" />
<figcaption>Image du navigateur après le bannissement par le rate limiting de Nginx</figcaption>

### Ban Fail2Ban:
<img width="928" height="647" src="docs/img/banFail2Ban.png" />
<figcaption>Image du navigateur après le bannissement par Fail2Ban</figcaption>

### Containers
<img width="1198" height="85" src="docs/img/dockerContainers.png" />
<figcaption>Image des conteneurs en cours d’exécution</figcaption>
