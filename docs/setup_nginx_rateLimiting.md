# Setup — Nginx Rate Limiting

Ce document explique uniquement la partie **Nginx Rate Limiting** du projet. Le but est de limiter le nombre de requêtes envoyées par une même IP afin de protéger le serveur contre le spam ou les attaques de type flood.

---

# 1. Créer les fichiers Nginx

```bash mkdir -p nginx/logs touch nginx/nginx.conf nginx/Dockerfile ```

---

# 2. Dockerfile Nginx

## `nginx/Dockerfile`

```dockerfile
FROM nginx:alpine

RUN apk add --no-cache tzdata

EXPOSE 80
```

Le package `tzdata` permet d'avoir des logs synchronisés avec l'heure du serveur.

---

# 3. Configuration du Rate Limiting

## `nginx/nginx.conf`

```nginx
user nginx;

events {
    worker_connections 1024;
}

http {

    access_log /var/log/nginx/access.log;
    error_log  /var/log/nginx/error.log warn;

    # Limite principale :
    # 5 requêtes/seconde par IP
    limit_req_zone $binary_remote_addr zone=zone_main:10m rate=5r/s;

    server {

        listen 80;

        root /usr/share/nginx/html;

        location / {

    # burst=10 :
    # accepte un petit pic temporaire
    limit_req zone=zone_main burst=10 nodelay;

    index index.html;
    }
    }
}
```

### Explication rapide

| Paramètre   | Rôle                                           |
| ----------- | ---------------------------------------------- |
| `rate=5r/s` | Maximum 5 requêtes/seconde par IP              |
| `burst=10`  | Tolère un petit pic de requêtes                |
| `nodelay`   | Bloque immédiatement si la limite est dépassée |

Quand la limite est dépassée, Nginx retourne :

```text 
503 Service Unavailable
```

---

# 4. Docker Compose

Dans `docker-compose.yml` :

```yaml
services:

    web:
    build: ./nginx

    container_name: nginx-server

    ports:
    - "8080:80"

    volumes:
    - ./nginx/nginx.conf:/etc/nginx/nginx.conf:ro
    - ./nginx/logs:/var/log/nginx
    - ./app:/usr/share/nginx/html
```

---

# 5. Lancer le projet

```bash
docker compose up -d 
```

Le site sera accessible sur :

```text 
http://localhost:8080
```

---

# 6. Tester le Rate Limiting

Recharger la page très rapidement (`F5` en rafale).

Ou utiliser :

```
  for i in $(seq 1 50); do
    curl -I http://localhost:8080
  done 
```

Après plusieurs requêtes rapides, certaines réponses deviennent :

```text 
503 Service Unavailable
```

---

# 7. Vérifier les logs

```bash 
cat nginx/logs/error.log
```

Vous devriez voir :

```text 
limiting requests
```

Cela confirme que le rate limiting fonctionne correctement.
