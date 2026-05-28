# Troubleshooting

Problèmes fréquents rencontrés lors de la mise en place de ce projet et leurs solutions.

---

## Fail2Ban ne bannit pas depuis localhost

**Symptôme :** Vous testez depuis votre propre machine et aucun ban ne se déclenche, même après des centaines de requêtes.

**Cause :** C'est un comportement intentionnel. Fail2Ban place `127.0.0.1` dans sa liste blanche (`ignoreip`) par défaut — bannir sa propre machine serait contre-productif.

**Solution :** Testez depuis une autre machine sur le même réseau local. Si vous n'en avez pas sous la main, une VM locale ou un second appareil (téléphone en partage de connexion désactivé, par exemple) font l'affaire. Voir la section [Tester Fail2Ban](setup.md#tester-fail2ban) dans le guide de mise en place.

---

## `docker compose` introuvable

**Symptôme :** `docker compose up` retourne `command not found` ou `unknown command`.

**Cause :** Vous avez une ancienne version de Docker qui utilise le binaire séparé `docker-compose` (avec tiret) plutôt que le plugin intégré.

**Solution :** Remplacez `docker compose` par `docker-compose` dans toutes les commandes. Pour vérifier votre version :

```bash
docker --version
docker compose version    # Plugin v2
docker-compose --version  # Binaire v1
```

---

## Le jail `nginx-limit-req` n'apparaît pas dans `fail2ban-client status`

**Symptôme :** `docker exec -it fail2ban fail2ban-client status` ne liste pas `nginx-limit-req`.

**Causes possibles et vérifications :**

1. **Le fichier n'est pas au bon endroit ou mal nommé.** Il doit être dans `fail2ban/config/jail.d/` et porter l'extension `.local` (pas `.conf`).

2. **Fail2Ban n'a pas été rechargé** après la création du fichier. Lancez :
   ```bash
   docker exec -it fail2ban fail2ban-client reload
   ```

3. **Une erreur de syntaxe dans le fichier.** Consultez les logs pour voir le message d'erreur :
   ```bash
   tail -f fail2ban/log/fail2ban.log
   ```

---

## Les bans sont ajoutés mais n'ont aucun effet

**Symptôme :** `fail2ban-client status nginx-limit-req` affiche bien des IPs bannies, mais ces IPs peuvent encore accéder au site.

**Cause :** Sans la directive `chain=DOCKER-USER`, Fail2Ban ajoute ses règles dans la chaîne `INPUT` d'iptables. Cette chaîne filtre le trafic destiné directement à l'hôte, mais pas le trafic routé vers les conteneurs Docker — le ban est créé mais sans effet sur le port `8080`.

**Solution :** Vérifiez que votre fichier `nginx-limit-req.local` contient bien :

```ini
action = iptables-allports[name=nginx-limit, chain=DOCKER-USER]
```

Rechargez ensuite Fail2Ban et débannissez les IPs existantes pour repartir proprement :

```bash
docker exec -it fail2ban fail2ban-client reload
docker exec -it fail2ban fail2ban-client unban --all
```

---

## Les timestamps des logs semblent décalés

**Symptôme :** Fail2Ban ne déclenche pas de ban alors que des erreurs s'accumulent dans `error.log`, ou les bans semblent se déclencher en retard.

**Cause :** Le fuseau horaire du conteneur Nginx ne correspond pas à celui du conteneur Fail2Ban. Fail2Ban compare les timestamps des logs à son horloge interne pour calculer `findtime` — un décalage peut faire rater des dépassements de seuil.

**Solution :** Assurez-vous que `TZ=` dans `docker-compose.yml` correspond à votre fuseau horaire réel, et que le `Dockerfile` Nginx installe bien `tzdata` :

```dockerfile
RUN apk add --no-cache tzdata
```

Redémarrez les conteneurs après toute modification de timezone :

```bash
docker compose down && docker compose up -d
```

---

## Fail2Ban ne trouve pas le fichier de log

**Symptôme :** `fail2ban-client status nginx-limit-req` affiche `File list: (empty)` ou une erreur dans les logs Fail2Ban indiquant que le fichier est introuvable.

**Cause :** Le volume `./nginx/logs:/remotelogs/nginx:ro` n'est pas correctement monté, ou le dossier `nginx/logs/` n'existait pas au moment du `docker compose up`.

**Solution :**

```bash
# Vérifier que le dossier existe sur l'hôte
ls nginx/logs/

# Si vide ou inexistant, le créer et redémarrer
mkdir -p nginx/logs
docker compose down && docker compose up -d
```

Vérifiez aussi que le fichier `error.log` est bien créé par Nginx après le premier démarrage :

```bash
ls -la nginx/logs/
```