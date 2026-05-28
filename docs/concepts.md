# Concepts — Rate Limiting Nginx & Fail2Ban

Ce document explique comment fonctionnent le Rate Limiting Nginx et Fail2Ban, ce qu'on a mis en place concrètement dans ce projet, et comment les deux mécanismes se complètent.

---

## Table des matières

- [Nginx Rate Limiting](#nginx-rate-limiting)
- [Fail2Ban](#fail2ban)
- [Comment les deux se complètent](#comment-les-deux-se-complètent)

---

## Nginx Rate Limiting

### Principe général

Le Rate Limiting est une fonctionnalité native de Nginx. Elle limite le nombre de requêtes qu'une adresse IP peut envoyer sur une période donnée, **directement au niveau du serveur web**, avant même que la requête n'atteigne l'application.

Nginx maintient une zone mémoire partagée qui associe chaque IP à un compteur de requêtes. Si une IP dépasse le seuil défini :

- La requête est immédiatement rejetée par Nginx.
- L'application backend ne reçoit rien.
- Le client reçoit une réponse `503 Service Unavailable`.
- Nginx écrit une ligne d'erreur dans son `error.log` — c'est cette ligne que Fail2Ban va surveiller.

Le blocage est instantané et temporaire : il ne dure que le temps du dépassement.

### Cas d'usage typiques

- Ralentir les attaques par brute force sur un formulaire de login.
- Limiter les pics de trafic soudains (scraping, bots).
- Empêcher qu'un seul client monopolise les ressources du serveur.
- Protéger l'application en amont, sans modifier son code.

### Ce qu'on a configuré dans ce projet

On a défini deux zones de limitation dans `nginx/nginx.conf`, car les besoins ne sont pas les mêmes selon le type de ressource :

```nginx
limit_req_zone $binary_remote_addr zone=zone_main:10m rate=5r/s;
```
→ Pages HTML : max **5 requêtes/seconde** par IP. Une navigation normale ne dépasse jamais ça.

```nginx
limit_req_zone $binary_remote_addr zone=zone_assets:10m rate=30r/s;
```
→ Assets statiques (images, scripts, fonts) : max **30 requêtes/seconde** par IP. Une seule page PokeChill charge des dizaines d'assets en parallèle, donc une zone plus permissive est nécessaire.

La taille `10m` alloue 10 Mo de mémoire partagée — suffisant pour suivre environ 160 000 IPs simultanément.

Ces zones sont ensuite appliquées sur les routes correspondantes :

```nginx
location / {
    limit_req zone=zone_main burst=10 nodelay;
}

location ~* ^/(img|scripts|font)/ {
    limit_req zone=zone_assets burst=200 nodelay;
}
```

| Paramètre | Rôle |
|---|---|
| `burst=10` | Tolère jusqu'à 10 requêtes supplémentaires en rafale avant de bloquer |
| `nodelay` | Traite la rafale immédiatement, sans mise en file d'attente |
| Au-delà du burst | `503` + ligne écrite dans `error.log` → Fail2Ban peut agir |

Le `burst` élevé sur les assets (`burst=200`) est intentionnel : le chargement initial d'une page PokeChill génère de nombreuses requêtes simultanées pour les sprites, scripts et polices.

---

## Fail2Ban

### Principe général

Fail2Ban est un outil de sécurité qui tourne en arrière-plan et **agit au niveau du firewall** (via `iptables` ou `nftables`), contrairement au Rate Limiting qui agit au niveau logiciel.

Son principe : analyser des fichiers de logs, détecter des comportements suspects via des filtres (expressions régulières), et bannir automatiquement les IPs fautives en ajoutant une règle firewall.

### Les trois éléments de Fail2Ban

| Élément | Rôle |
|---|---|
| **Logs** | Les fichiers surveillés — ici `error.log` de Nginx |
| **Filtres** | Des expressions régulières qui identifient les lignes suspectes |
| **Jails** | La configuration qui relie un filtre à des logs et définit les seuils de déclenchement |

### Fail2Ban peut surveiller bien plus que le rate limiting

Dans ce projet on utilise Fail2Ban pour détecter les dépassements de rate limit Nginx, mais ce n'est qu'un cas d'usage parmi d'autres. On aurait pu configurer des jails pour :

- Les tentatives de connexion SSH échouées.
- Les erreurs `404` répétées (scan de chemins).
- Les requêtes malformées ou les tentatives d'injection.
- N'importe quel pattern détectable dans un fichier de log.

C'est la flexibilité des filtres qui rend Fail2Ban généraliste.

### Ce qu'on a configuré dans ce projet

**Fonctionnement dans notre infrastructure :**

1. Nginx rejette une requête pour dépassement de rate limit → écrit une ligne `limiting requests` dans `error.log`.
2. Fail2Ban détecte cette ligne grâce au filtre `nginx-limit-req` (filtre intégré à Fail2Ban, aucune configuration à écrire).
3. Fail2Ban compte les occurrences pour chaque IP dans la fenêtre `findtime`.
4. Si le seuil `maxretry` est atteint → règle `iptables` ajoutée → l'IP est bannie au niveau réseau.
5. Toute connexion depuis cette IP est rejetée avant même d'atteindre Nginx, pendant toute la durée du `bantime`.

**Fichier `fail2ban/config/jail.d/nginx-limit-req.local` :**

```ini
[nginx-limit-req]
enabled  = true
port     = 8080
filter   = nginx-limit-req
logpath  = /remotelogs/nginx/error.log
maxretry = 5
findtime = 600
bantime  = 1h

# Cible la chaîne DOCKER-USER d'iptables
# Sans ça, le ban est ajouté dans INPUT et n'a aucun effet sur le trafic Docker
action = iptables-allports[name=nginx-limit, chain=DOCKER-USER]
```

| Paramètre | Valeur | Signification |
|---|---|---|
| `filter` | `nginx-limit-req` | Filtre intégré à Fail2Ban, détecte les lignes `limiting requests` |
| `logpath` | `/remotelogs/nginx/error.log` | Le `error.log` Nginx monté en volume dans le conteneur Fail2Ban |
| `maxretry` | `5` | 5 dépassements de rate limit → ban |
| `findtime` | `600` | Ces 5 erreurs doivent survenir en moins de 10 minutes |
| `bantime` | `1h` | L'IP est bannie pendant 1 heure |
| `chain` | `DOCKER-USER` | Chaîne iptables interceptant le trafic avant qu'il n'atteigne les conteneurs Docker |

---

## Comment les deux se complètent

| | Nginx Rate Limiting | Fail2Ban |
|---|---|---|
| **Où ça agit** | Au niveau de Nginx (logiciel) | Au niveau du firewall (réseau) |
| **Durée du blocage** | Instantané, fraction de seconde | 1 heure (configurable) |
| **Déclenchement** | Quota de requêtes dépassé | Seuil d'erreurs dans les logs atteint |
| **Réponse côté client** | `503 Service Unavailable` | Connexion refusée / timeout |
| **Ce qu'il protège** | L'application, les ressources serveur | L'infrastructure réseau |

Les deux fonctionnent en cascade :

```
Requête entrante
      │
      ▼
┌─────────────┐    quota ok     ┌─────────────┐
│    Nginx    │ ─────────────── │ Application │
│ Rate Limit  │                 └─────────────┘
└─────────────┘
      │ quota dépassé
      ▼
  503 + ligne dans error.log
      │
      ▼
┌─────────────┐
│  Fail2Ban   │  surveille error.log en continu
│             │
│  5 erreurs  │
│  en 10 min  │ ──────────────── ban iptables 1h
└─────────────┘
      │
      ▼
Connexion refusée au niveau réseau
(l'IP n'atteint même plus Nginx)
```

Nginx réagit immédiatement pour chaque requête excessive. Fail2Ban observe le comportement dans la durée et bannit les IPs qui persistent — les empêchant même d'atteindre Nginx lors des tentatives suivantes.