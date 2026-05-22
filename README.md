# Screenshots
<img width="923" height="933" alt="screenshot-2026-05-22_09-28-23" src="https://github.com/user-attachments/assets/8ed31089-9fe7-46bf-95b0-be4a8fd8b26a" />

# Pokechill-Fail2ban-Nginx

# C'est quoi ?
Notre projet consiste à reprendre un projet GitHub (ici, PokeChill) et à y intégrer Fail 2 Ban ainsi que le rate limiting de Nginx.

## Fail 2 Ban
Fail 2 Ban est un outil de sécurité qui analyse les logs afin de détecter des comportements suspects (par exemple: trop de requêtes dans un court laps de temps).
S’il en repère un, il bannit l’adresse IP ayant un comportement suspect pendant une durée déterminée (dans notre cas, 1 heure).

## Nginx Rate Limiting
Le rate limiting de Nginx est également un outil de sécurité.
Il limite le nombre de requêtes d’un utilisateur et, si le quota de requêtes est dépassé, les requêtes de cette adresse IP ne reçoivent plus de réponse.
Cependant, comparé à Fail 2 Ban, le blocage ne dure pas longtemps (moins d’une seconde).

# Comment résaliser ce projet ?

## Cette section sera faite par celui qui a commiter cette modification.

## Resultat Final
<img width="1075" height="935" alt="screenshot-2026-05-22_08-38-23" src="https://github.com/user-attachments/assets/ba1004e4-78b2-47ee-b5ae-7272da4c0b6d" />

# Ban Rate limiting example:
<img width="1059" height="985" alt="screenshot-2026-05-22_08-44-43" src="https://github.com/user-attachments/assets/2f571eaa-aeef-46b6-821a-24e7d9f1d41d" />

# Ban Fail2Ban example:
<img width="928" height="647" alt="screenshot-2026-05-22_09-20-16" src="https://github.com/user-attachments/assets/73737e75-ba51-469d-bcde-70bed781dcb9" />

# Containers
<img width="1198" height="85" alt="screenshot-2026-05-22_08-36-20" src="https://github.com/user-attachments/assets/1fca5a27-5b9b-4524-9a39-b5d2a747e9b6" />
