# Pokechill — Fail2Ban & Nginx Rate Limiting

<img width="1075" height="935" src="docs/img/screenshotGame1.png" />

---

## Contexte et objectifs

Ce projet utilise [PokeChill](https://github.com/play-pokechill/play-pokechill.github.io) comme application support — un jeu web open source servi par Nginx dans un conteneur Docker. L'application elle-même n'est pas le sujet : elle sert de cible pour mettre en pratique deux mécanismes de sécurité réseau complémentaires.

**Ce que vous allez apprendre et mettre en place :**

- Le **Rate Limiting Nginx** — limiter le nombre de requêtes acceptées par IP au niveau du serveur web.
- **Fail2Ban** — surveiller les logs et bannir durablement au niveau du firewall les IPs qui abusent.

À la fin, vous aurez une infrastructure où Nginx coupe le trafic excessif en temps réel, et Fail2Ban prend le relais pour bannir durablement les IPs persistantes via `iptables`.

---

## Architecture du projet

```text
📁 pokechill-fail2ban-Nginx
├── 📁 app                        ← PokeChill (cloné depuis GitHub)
│   ├── 📁 font
│   ├── 📁 img
│   ├── 🌐 index.html
│   ├── 📁 scripts
│   └── 🎨 styles.css
├── 📁 docs
│   ├── 📁 img
│   ├── 📝 concepts.md            ← Explication Rate Limiting + Fail2Ban
│   ├── 📝 setup.md               ← Guide de mise en place complet
│   └── 📝 troubleshooting.md     ← Problèmes fréquents et solutions
├── 📁 fail2ban
│   ├── ⚙️ config
│   │   └── 📁 jail.d
│   │       └── nginx-limit-req.local
│   └── 📂 log
├── 📁 nginx
│   ├── 🐳 Dockerfile
│   ├── 📂 logs
│   │   ├── access.log
│   │   └── error.log             ← Fail2Ban surveille ce fichier
│   └── ⚙️ nginx.conf
├── 🐳 docker-compose.yml
└── 📖 README.md
```

**Flux des logs :** Nginx écrit dans `nginx/logs/error.log` → ce dossier est monté en volume dans les deux conteneurs → Fail2Ban lit les mêmes fichiers sans passer par le réseau.

---

## Documentation

| Fichier | Contenu |
|---|---|
| [docs/concepts.md](docs/concepts.md) | Comprendre le Rate Limiting Nginx et Fail2Ban — comment ils fonctionnent, ce qu'on a configuré dans ce projet, et comment ils se complètent |
| [docs/setup.md](docs/setup.md) | Créer le projet de zéro : arborescence, fichiers de config, lancement, tests |
| [docs/troubleshooting.md](docs/troubleshooting.md) | Problèmes fréquents rencontrés pendant ce projet et leurs solutions |

---

## Résultat final

<img width="1198" height="85" src="docs/img/dockerContainers.png" />
<figcaption>Les deux conteneurs en cours d'exécution</figcaption>

<img width="1075" height="935" src="docs/img/screenshotGame2.png" />
<figcaption>PokeChill en fonctionnement sur la page Travel</figcaption>