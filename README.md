# Pokechill-Fail2ban-Nginx

<img width="1075" height="935" src="docs/img/screenshotGame1.png" />
<figcaption>Image du jeu en cours de fonctionnement sur l'accueil</figcaption>

---

## Table des matières

- [C'est quoi ?](#cest-quoi-)
    - [Fail2Ban](#fail2ban)
    - [Nginx Rate Limiting](#nginx-rate-limiting)
- [Architecture du projet](#architecture-du-projet)
- [Comment réaliser ce projet ?](#comment-réaliser-ce-projet-)
    - [Prérequis](#prérequis)
    - [Mise en place de l'espace de travail](#mise-en-place-de-lespace-de-travail)
    - [Configuration de Nginx](#configuration-de-nginx)
    - [Création du docker-compose.yml](#création-du-docker-composeyml)
    - [Comment savoir si tout fonctionne ?](#comment-savoir-si-tout-fonctionne-)
        - [Nginx Rate Limiter](#nginx-rate-limiter)
        - [Fail2Ban](#fail2ban-1)
- [Résultat Final](#résultat-final)
    - [Ban Rate Limiting](#ban-rate-limiting)
    - [Ban Fail2Ban](#ban-fail2ban)
    - [Containers](#containers)

---

## C'est quoi ?

Notre projet consiste à reprendre un projet GitHub (ici, PokeChill) et à y intégrer Fail2Ban ainsi que le rate limiting de Nginx.

### Fail2Ban

Fail2Ban est un outil de sécurité qui analyse les logs afin de détecter des comportements suspects (par exemple : trop de requêtes dans un court laps de temps).
S'il en repère un, il bannit l'adresse IP ayant un comportement suspect pendant une durée déterminée (dans notre cas, 1 heure).

### Nginx Rate Limiting

Le rate limiting de Nginx est également un outil de sécurité.
Il limite le nombre de requêtes d'un utilisateur et, si le quota de requêtes est dépassé, les requêtes de cette adresse IP ne reçoivent plus de réponse.
Cependant, comparé à Fail2Ban, le blocage ne dure pas longtemps (moins d'une seconde).

---

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

---

## Comment réaliser ce projet ?

### Prérequis

- Docker
- Git

### Mise en place de l'espace de travail

Créer un répertoire `pokechill-fail2ban-Nginx` (nous allons travailler dedans) :

```bash
mkdir pokechill-fail2ban-Nginx
```

Entrer dans ce dossier nouvellement créé :

```bash
cd pokechill-fail2ban-Nginx
```

Copier le projet `play-pokechill.github.io` sous le nom de `app` :

```bash
git clone https://github.com/play-pokechill/play-pokechill.github.io.git app
```

Toujours dans `pokechill-fail2ban-Nginx` (votre répertoire principal), créer les dossiers pour la configuration et les logs Nginx et Fail2Ban :

```bash
mkdir -p nginx/logs fail2ban/config fail2ban/log
```

---

### Configuration de Nginx

Créer le fichier de configuration de Nginx avec la fonctionnalité de rate limiting :

```bash
cd nginx && touch nginx.conf
```

Ensuite, éditer `nginx.conf` avec votre éditeur préféré.
**Le contenu du fichier [ICI](nginx/nginx.conf)**

Créer également un `Dockerfile` pour personnaliser l'image de Nginx :

```bash
touch Dockerfile
```

**Le contenu du fichier [ICI](nginx/Dockerfile)**

---

### Création du docker-compose.yml

Créer un `docker-compose.yml` à la racine du projet (`pokechill-fail2ban-Nginx`) :

```bash
touch docker-compose.yml
```

**Le contenu du fichier [ICI](docker-compose.yml)**

Vous pouvez désormais exécuter le projet :

```bash
docker compose up
```

En faisant cela, du contenu devrait apparaître dans `fail2ban`.

Dans le dossier `fail2ban`, vous allez trouver un dossier `config` et un dossier `log`.

La configuration se trouve dans `config`. Entrer dedans, puis dans le dossier `jail.d`. Il contient toute la configuration par défaut.

> **⚠️ INFO :** Toute modification à l'intérieur des fichiers par défaut sera réinitialisée au redémarrage.

```bash
cd fail2ban/config/jail.d
```

Ajouter le fichier `nginx-limit-req.local` :

```bash
touch nginx-limit-req.local
```

**Le contenu du fichier [ICI](fail2ban/config/jail.d/nginx-limit-req.local)**

Une fois le fichier créé, recharger la configuration pour l'appliquer *(attention, le projet doit être lancé !)* :

```bash
docker exec -it fail2ban fail2ban-client reload
```

Vérifier que la configuration est bien appliquée :

```bash
docker exec -it fail2ban fail2ban-client status nginx-limit-req
```

---

### Comment savoir si tout fonctionne ?

Pour savoir si votre projet fonctionne, vous pouvez le tester avec les moyens ci-dessous :

#### Nginx Rate Limiter

Rechargez la page plusieurs fois rapidement dans votre navigateur. Vous obtiendrez un écran **503 - Temporarily Banned**.

#### Fail2Ban

> **⚠️ IMPORTANT :** Tester Fail2Ban en local ne sert à rien, car Fail2Ban ne bannit pas `localhost`. Il est nécessaire de tester depuis une autre machine ou via une adresse IP externe.

Pour simuler des connexions depuis l'extérieur, nous utilisons `ufw` (Uncomplicated Firewall).

Si `ufw` n'est pas installé sur votre machine, installez-le d'abord :

```bash
sudo apt install ufw
```

Une fois installé, voici les commandes utiles :

| Commande | Description |
|---|---|
| `sudo ufw allow 8080/tcp` | Autorise les connexions entrantes sur le port 8080 |
| `ufw status` | Affiche l'état et les règles actives de ufw |
| `sudo ufw delete allow 8080/tcp` | Supprime la règle autorisant le port 8080 |

Pour recharger la configuration de Fail2Ban, vérifier son statut ou débannir des adresses IP :

```bash
# Recharger la configuration
docker exec -it fail2ban fail2ban-client reload

# Vérifier le statut du jail nginx-limit-req
docker exec -it fail2ban fail2ban-client status nginx-limit-req

# Débannir une adresse IP spécifique
docker exec -it fail2ban fail2ban-client set nginx-limit-req unbanip <ADRESSE_IP>

# Débannir toutes les adresses IP
docker exec -it fail2ban fail2ban-client unban --all
```

---

## Résultat Final

<img width="1075" height="935" src="docs/img/screenshotGame2.png" />
<figcaption>Image du jeu en cours de fonctionnement sur la page Travel</figcaption>

### Ban Rate Limiting

<img width="1059" height="985" src="docs/img/banRateLimiting.png" />
<figcaption>Image du navigateur après le bannissement par le rate limiting de Nginx</figcaption>

### Ban Fail2Ban

<img width="928" height="647" src="docs/img/banFail2Ban.png" />
<figcaption>Image du navigateur après le bannissement par Fail2Ban</figcaption>

### Containers

<img width="1198" height="85" src="docs/img/dockerContainers.png" />
<figcaption>Image des conteneurs en cours d'exécution</figcaption>