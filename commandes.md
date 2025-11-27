
# GROUPE:   Florian BEAUCLAIR / Romain BOYER--BARNERIAS / Korawit BOONCHAT

# Etape 1:
# Creation du repertoir docker-tp3
    mkdir docker-tp3
    cd docker-tp3

# Creation des sous repertoire
    mkdir etape1
    mkdir etape2
    mkdir etape3

# Creation du reseau
# Cree un nouveau reseau Docker nomme "app-network" pour permettre la communication entre plusieurs conteneurs
    docker network create app-network

# Lancement du container PHP-FPM
# Lance en mode detache le conteneur "script", connecte au reseau app-network, qui execute l image PHP 8.2-FPM, tout en montant le dossier local ./app dans /app du conteneur
    docker container run -d --name script --network app-network 
    -v ${PWD}/app:/app 
    php:8.2-fpm

# Lancement du container Nginx
# Lance le conteneur en mode detache "http", connecte le conteneur au reseau Docker app-network
    docker container run -d --name http --network app-network 

# Redirige le port 8080 de la machine vers le port 80 du conteneur
    -p 8080:80 

# Monte le dossier local ./app dans le chemin /app du conteneur
    -v ${PWD}/app:/app 

# Monte le fichier de configuration Nginx local dans le repertoire de config du conteneur
    -v ${PWD}/nginx/default.conf:/etc/nginx/conf.d/default.conf



# ------------------------------------------------------------ #



# Etape 2:
# Construction de l'image PHP avec mysqli
    docker build -t php-mysqli .

# Lancer MariaDB
    docker container run -d --name data --network app-network 

# Monte le fichier local "create.sql" dans le dossier "/docker-entrypoint-initdb.d/" du conteneur pour qu il soit automatiquement execute a l initialisation de la base de donnees
    -v ${PWD}/app/create.sql:/docker-entrypoint-initdb.d/create.sql 

# Demande a l image MariaDB de generer automatiquement un mot de passe root aleatoire au demarrage au lieu d utiliser un mot de passe fixe
    -e MARIADB_RANDOM_ROOT_PASSWORD=yes

# mage  MariaDB 
    mariadb:latest

# Lancer PHP-FPM
    docker container run -d --name script --network app-network 

# Monte le dossier local ./app dans le chemin /app du conteneur
    -v ${PWD}/app:/app 
    php-mysqli

# Lancer Nginx
    docker container run -d --name http --network app-network 

# Redirige le port 8080 de la machine vers le port 80 du conteneur
    -p 8080:80 
    -v ${PWD}/app:/app 
# Cette option monte le fichier local "./nginx/default.conf" dans le chemin "/etc/nginx/conf.d/default.conf" du conteneur pour que Nginx utilise directement la configuration personnalisee
    -v ${PWD}/nginx/default.conf:/etc/nginx/conf.d/default.conf `



# ------------------------------------------------------------ #



# Etape 3:
# demarre en mode detache tous les conteneurs definis dans le fichier docker-compose.yml
docker-compose up -d
