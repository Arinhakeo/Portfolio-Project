# Introduction à Docker et Guide d'Installation Complet

## Partie 1 : Comprendre Docker

### Qu'est-ce que Docker ?
Docker est une plateforme qui vous permet de "conteneuriser" vos applications. En termes simples, cela signifie que vous pouvez :
- Empaqueter votre application et tout ce dont elle a besoin pour fonctionner dans un conteneur portable
- Déplacer facilement cette application d'un ordinateur à un autre
- La faire fonctionner partout de la même manière

### Les avantages de Docker
1. **Portabilité** :
   - Fonctionne partout de la même façon
   - Pas de problème de "ça marche sur mon ordinateur mais pas ailleurs"

2. **Isolation** :
   - Chaque conteneur est isolé des autres
   - Contient ses propres :
     * Bibliothèques
     * Dépendances
     * Configuration

3. **Légèreté** :
   - Démarre rapidement
   - S'arrête rapidement
   - Utilise peu de ressources

4. **Scalabilité** :
   - Peut faire fonctionner plusieurs copies de la même application
   - Facile à gérer avec Docker Compose

## Partie 2 : Le Projet

### Objectif du Projet
Nous allons créer une infrastructure complète comprenant :
1. Un proxy inverse (reverse proxy)
2. Un équilibreur de charge (load balancer)
3. Deux serveurs d'application
4. Un serveur front-end

### Fonctionnement
1. **Point d'Entrée Unique** :
   - Un seul serveur principal qui reçoit tout le trafic
   - Agit comme proxy inverse et équilibreur de charge

2. **Gestion des Requêtes** :
   - Pour le contenu statique (front-end) :
     * Redirige vers le serveur front-end
     * Renvoie le contenu au client
   - Pour les requêtes API :
     * Utilise l'algorithme Round Robin
     * Répartit entre les deux serveurs API

### Explication du Round Robin
C'est comme un tourniquet qui distribue les requêtes à tour de rôle :
- Requête 1 → Serveur A
- Requête 2 → Serveur B
- Requête 3 → Serveur C
- Requête 4 → Retour au Serveur A
Et ainsi de suite...

## Partie 3 : Guide d'Installation Détaillé

### Windows
1. **Prérequis** :
   - Windows 10 64-bit: Pro, Enterprise, ou Education
   - Activation de la virtualisation dans le BIOS

2. **Installation** :
   ```bash
   # 1. Télécharger Docker Desktop
   - Aller sur https://www.docker.com/products/docker-desktop
   - Cliquer sur "Download for Windows"

   # 2. Installation
   - Double-cliquer sur le fichier .exe téléchargé
   - Suivre l'assistant d'installation
   - Redémarrer votre ordinateur

   # 3. Vérification
   - Ouvrir PowerShell
   - Taper : docker --version
   ```

### macOS
1. **Prérequis** :
   - macOS 10.15 ou plus récent
   - Processeur Intel ou Apple Silicon

2. **Installation** :
   ```bash
   # 1. Télécharger Docker Desktop
   - Aller sur https://www.docker.com/products/docker-desktop
   - Cliquer sur "Download for Mac"

   # 2. Installation
   - Double-cliquer sur le fichier .dmg
   - Glisser Docker dans Applications
   - Lancer Docker depuis Applications

   # 3. Vérification
   - Ouvrir Terminal
   - Taper : docker --version
   ```

### Linux (Ubuntu)
1. **Installation via Terminal** :
   ```bash
   # Mise à jour du système
   sudo apt-get update
   sudo apt-get upgrade -y

   # Installation des prérequis
   sudo apt-get install -y \
       apt-transport-https \
       ca-certificates \
       curl \
       gnupg \
       lsb-release

   # Ajout de la clé GPG Docker
   curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

   # Configuration du repository
   echo \
     "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
     $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

   # Installation de Docker
   sudo apt-get update
   sudo apt-get install -y docker-ce docker-ce-cli containerd.io

   # Ajout de l'utilisateur au groupe docker
   sudo usermod -aG docker $USER

   # Vérification
   docker --version
   ```

### Chromebook
⚠️ Note importante : Sur Chromebook, vous n'aurez accès qu'au moteur Docker, pas à Docker Desktop. Il est recommandé d'utiliser une machine Windows, Mac ou Linux.

## Partie 4 : Premiers Pas avec Docker

### Vérification de l'Installation
```bash
# Vérifier la version
docker --version

# Vérifier que Docker fonctionne
docker run hello-world
```

### Commandes Essentielles
```bash
# Lister les images
docker images

# Lister les conteneurs en cours d'exécution
docker ps

# Lister tous les conteneurs (même arrêtés)
docker ps -a

# Arrêter un conteneur
docker stop [ID_ou_NOM]

# Supprimer un conteneur
docker rm [ID_ou_NOM]
```

### Pour Aller Plus Loin
- Documentation officielle : https://docs.docker.com/
- Docker Hub (bibliothèque d'images) : https://hub.docker.com/
- Tutoriels interactifs : https://www.docker.com/play-with-docker
