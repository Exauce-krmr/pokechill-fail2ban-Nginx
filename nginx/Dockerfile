FROM nginx:alpine

# On s'assure que les fuseaux horaires sont synchronisés pour que les logs 
# correspondent à l'heure du serveur hôte (crucial pour Fail2Ban)
RUN apk add --no-cache tzdata

# On expose le port 80 (interne au conteneur)
EXPOSE 80