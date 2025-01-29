## **1. Serveur SQL (PostgreSQL)**

### **Méthode A : Installation Locale**
1. **Installer PostgreSQL** :
   - Téléchargez PostgreSQL depuis [le site officiel](https://www.postgresql.org/).
   - Suivez les étapes d'installation et configurez un utilisateur admin (souvent appelé `postgres`).

2. **Démarrer le Serveur PostgreSQL** :
   - Sur Windows : Utilisez `pgAdmin` ou démarrez le service via l'application PostgreSQL.
   - Sur Mac/Linux : Démarrez PostgreSQL avec la commande :
     ```bash
     sudo service postgresql start
     ```

3. **Créer une Base de Données** :
   - Accédez à `psql` (outil en ligne de commande) :
     ```bash
     psql -U postgres
     ```
   - Créez une base de données :
     ```sql
     CREATE DATABASE mydatabase;
     ```

4. **Configurer Flask pour PostgreSQL** :
   - Dans votre fichier de configuration Flask :
     ```python
     SQLALCHEMY_DATABASE_URI = 'postgresql://username:password@localhost/mydatabase'
     ```

---

### **Méthode B : Docker**
1. **Installer Docker** :
   - Téléchargez Docker depuis [docker.com](https://www.docker.com/).

2. **Lancer un Conteneur PostgreSQL** :
   ```bash
   docker run --name my_postgres -e POSTGRES_USER=postgres -e POSTGRES_PASSWORD=password -d -p 5432:5432 postgres
   ```

3. **Se Connecter depuis Flask** :
   - Mettez à jour la configuration pour utiliser l'hôte Docker :
     ```python
     SQLALCHEMY_DATABASE_URI = 'postgresql://postgres:password@localhost:5432/mydatabase'
     ```

---

## **2. Serveur SQL (MySQL)**

### **Méthode A : Installation Locale**
1. **Installer MySQL** :
   - Téléchargez MySQL depuis [mysql.com](https://www.mysql.com/).
   - Configurez un utilisateur root et un mot de passe.

2. **Démarrer le Serveur MySQL** :
   - Sur Windows : Utilisez le panneau de contrôle MySQL.
   - Sur Linux/Mac :
     ```bash
     sudo service mysql start
     ```

3. **Créer une Base de Données** :
   - Accédez à l'interface MySQL :
     ```bash
     mysql -u root -p
     ```
   - Créez une base de données :
     ```sql
     CREATE DATABASE mydatabase;
     ```

4. **Configurer Flask pour MySQL** :
   - Installez le connecteur MySQL pour SQLAlchemy :
     ```bash
     pip install pymysql
     ```
   - Configurez l'URI dans Flask :
     ```python
     SQLALCHEMY_DATABASE_URI = 'mysql+pymysql://username:password@localhost/mydatabase'
     ```

---

### **Méthode B : Docker**
1. **Lancer un Conteneur MySQL** :
   ```bash
   docker run --name my_mysql -e MYSQL_ROOT_PASSWORD=password -d -p 3306:3306 mysql
   ```

2. **Configurer l’Accès depuis Flask** :
   - Adaptez la configuration Flask pour le conteneur MySQL.

---

## **3. Serveur SQL Lite (SQLite)**

SQLite est intégré et ne nécessite pas de serveur séparé. Idéal pour le développement ou les petits projets.

### Configuration dans Flask :
1. Installez SQLite :
   ```bash
   sudo apt install sqlite3  # Sur Linux
   ```
2. Configurez Flask pour utiliser un fichier SQLite :
   ```python
   SQLALCHEMY_DATABASE_URI = 'sqlite:///mydatabase.db'
   ```

---

## **4. Serveur NoSQL (MongoDB)**

### **Méthode A : Installation Locale**
1. **Installer MongoDB** :
   - Téléchargez MongoDB depuis [mongodb.com](https://www.mongodb.com/).
   - Configurez le chemin des données et démarrez MongoDB :
     ```bash
     sudo service mongod start
     ```

2. **Configurer Flask pour MongoDB** :
   - Installez l'extension Flask pour MongoDB :
     ```bash
     pip install flask-pymongo
     ```
   - Configurez MongoDB dans votre application Flask :
     ```python
     from flask_pymongo import PyMongo

     app.config['MONGO_URI'] = "mongodb://localhost:27017/mydatabase"
     mongo = PyMongo(app)
     ```

---

### **Méthode B : Docker**
1. **Lancer un Conteneur MongoDB** :
   ```bash
   docker run --name my_mongo -d -p 27017:27017 mongo
   ```

2. **Configurer Flask pour le Conteneur** :
   - Modifiez l'URI pour pointer vers le conteneur Docker :
     ```python
     app.config['MONGO_URI'] = "mongodb://localhost:27017/mydatabase"
     ```

---

## **5. Serveur Redis (Cache)**

Redis est utilisé comme base de données en mémoire ou système de cache.

### **Méthode A : Installation Locale**
1. **Installer Redis** :
   - Téléchargez Redis depuis [redis.io](https://redis.io/).
   - Démarrez le serveur :
     ```bash
     redis-server
     ```

2. **Configurer Flask pour Redis** :
   - Installez l’extension Redis pour Flask :
     ```bash
     pip install flask-redis
     ```
   - Configurez Redis dans Flask :
     ```python
     from flask_redis import FlaskRedis

     app.config['REDIS_URL'] = "redis://localhost:6379/0"
     redis_client = FlaskRedis(app)
     ```

---

### **Méthode B : Docker**
1. **Lancer un Conteneur Redis** :
   ```bash
   docker run --name my_redis -d -p 6379:6379 redis
   ```

2. **Configurer Flask pour le Conteneur** :
   - Modifiez l’URL Redis dans la configuration.

---

## **6. Serveurs avec ORM (Django ou SQLAlchemy)**

Si vous utilisez un ORM comme SQLAlchemy ou Django ORM, le processus de connexion est souvent le même. Voici quelques points spécifiques :

- **SQLAlchemy** :
  - Configurez simplement l’URI de connexion dans Flask comme mentionné ci-dessus.
  
- **Django** :
  - Utilisez le fichier `settings.py` pour définir la configuration de la base de données.

---

## Synthèse

| **Type de Serveur** | **Options**                              | **Configuration**                                                                 |
|----------------------|------------------------------------------|-----------------------------------------------------------------------------------|
| PostgreSQL           | Local/Docker                            | URI : `postgresql://username:password@localhost:5432/dbname`                     |
| MySQL                | Local/Docker                            | URI : `mysql+pymysql://username:password@localhost:3306/dbname`                  |
| SQLite               | Aucun Serveur                           | Fichier local : `sqlite:///mydatabase.db`                                        |
| MongoDB              | Local/Docker                            | URI : `mongodb://localhost:27017/dbname`                                         |
| Redis                | Local/Docker                            | URI : `redis://localhost:6379/0`                                                 |
