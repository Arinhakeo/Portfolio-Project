Liste complète des dépendances de base nécessaires, avec leurs fonctions :

```plaintext
# Dépendances Principales
Flask==3.0.0                # Framework web principal
SQLAlchemy==2.0.0          # ORM pour la base de données
Flask-SQLAlchemy==3.1.1    # Intégration de SQLAlchemy avec Flask
python-dotenv==1.0.0       # Gestion des variables d'environnement

# Sécurité
Flask-Login==0.6.3         # Gestion de l'authentification
Flask-Bcrypt==1.0.1        # Hashage des mots de passe
Flask-WTF==1.2.1          # Protection CSRF et validation de formulaires

# Gestion Base de Données
alembic==1.13.1           # Migrations de base de données
Flask-Migrate==4.0.5      # Interface Flask pour Alembic

# Extensions Utiles
Flask-Mail==0.9.1         # Envoi d'emails
Pillow==10.1.0           # Manipulation d'images
Flask-Cors==4.0.0        # Gestion des CORS pour les API

# Utilitaires
requests==2.31.0         # Requêtes HTTP
itsdangerous==2.1.2     # Signature sécurisée de données
Werkzeug==3.0.1         # Utilitaires WSGI
```

Pour installer toutes ces dépendances :

1. Créez un fichier `requirements.txt` avec ce contenu

~~~plaintext requirements.txt
Flask==2.0.1
Flask-SQLAlchemy==2.5.1
Flask-Login==0.5.0
Flask-JWT-Extended==4.2.3
Flask-Cors==3.0.10
Flask-Mail==0.9.1
python-dotenv==0.19.0
Pillow==8.3.1
~~~

2. Installez-les avec :
```bash
pip install -r requirements.txt
```

3. Pour vérifier l'installation :
```bash
pip freeze
```
# 1. Installer les dépendances
pip install -r requirements.txt

# 2. Initialiser la base de données
python init_db.py

# 3. Démarrer le serveur
python run.py

Ces dépendances couvrent les besoins de base pour :
- Serveur web
- Base de données
- Authentification
- Sécurité
- Gestion des emails
- Gestion des images
- API REST

Voulez-vous que j'explique l'utilité spécifique de chaque dépendance ? Ou préférez-vous qu'on passe à leur installation et configuration ?
