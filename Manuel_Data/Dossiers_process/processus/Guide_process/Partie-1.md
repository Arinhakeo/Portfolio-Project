# Guide de Mise en Place du Projet FMP
## Partie 1 : Configuration Initiale

### Table des matières
1. [Structure du projet](#1-structure-du-projet)
2. [Configuration de l'environnement](#2-configuration-de-lenvironnement)
3. [Configuration Backend](#3-configuration-backend)
4. [Premier test](#4-premier-test)

## 1. Structure du projet

Première étape : création de la structure des dossiers. Ouvrez votre terminal et exécutez les commandes suivantes :

```bash
# Création du dossier principal
mkdir FMP
cd FMP

# Création de la structure backend
mkdir -p backend/app/{routes,models,static,templates}
mkdir -p backend/app/static/uploads

# Création de la structure frontend
mkdir -p frontend/static/{css,js,images}
mkdir frontend/pages

# Création des fichiers Python initiaux
touch backend/config.py
touch backend/run.py
touch backend/app/__init__.py
touch backend/app/routes/__init__.py
touch backend/app/models/__init__.py
```

La structure resultante devrait être :
```
FMP/
├── backend/
│   ├── app/
│   │   ├── __init__.py
│   │   ├── routes/
│   │   │   └── __init__.py
│   │   ├── models/
│   │   │   └── __init__.py
│   │   ├── static/
│   │   │   └── uploads/
│   │   └── templates/
│   ├── config.py
│   └── run.py
└── frontend/
    ├── static/
    │   ├── css/
    │   ├── js/
    │   └── images/
    └── pages/
```

## 2. Configuration de l'environnement

### 2.1 Création de l'environnement virtuel

```bash
# Dans le dossier FMP
python3 -m venv venv

# Activation de l'environnement virtuel
# Sur Windows
venv\Scripts\activate
# Sur Linux/Mac
source venv/bin/activate
```

### 2.2 Installation des dépendances

Versions compatibles pour un projet Flask
Voici une liste de versions compatibles entre elles pour les bibliothèques couramment utilisées dans un projet Flask :

Bibliothèque	Version recommandée	Notes
Flask	2.3.2	Framework web principal.
Werkzeug	2.3.7	Utilisé par Flask pour les fonctionnalités de bas niveau.
Flask-SQLAlchemy	3.0.5	Intégration de SQLAlchemy avec Flask.
SQLAlchemy	1.4.49	ORM pour interagir avec la base de données.
Flask-Bcrypt	1.0.1	Hachage sécurisé des mots de passe.
Flask-Login	0.6.2	Gestion des sessions utilisateur et de l'authentification.
Flask-WTF	1.2.1	Intégration de WTForms pour la gestion des formulaires.
Flask-CORS	4.0.0	Gestion des en-têtes CORS pour les requêtes cross-origin.
Flask-Mail	0.9.1	Envoi d'e-mails depuis votre application Flask.
Flask-Migrate	4.0.5	Gestion des migrations de base de données avec Alembic.
Flask-RESTful	0.3.10	Création d'API RESTful.
Flask-JWT-Extended	4.5.2	Gestion des tokens JWT pour l'authentification.
python-dotenv	1.0.0	Gestion des variables d'environnement.
stripe	8.0.0	Intégration des paiements via Stripe.
Pillow	10.0.0	Manipulation d'images (pour les produits, par exemple).
Flask-Caching	2.1.0	Mise en cache pour améliorer les performances.

Créez un fichier `requirements.txt` dans le dossier backend :

```txt
# backend/requirements.txt

Flask==2.3.2
Werkzeug==2.3.7
Flask-SQLAlchemy==3.0.5
SQLAlchemy==1.4.49
Flask-Bcrypt==1.0.1
Flask-Login==0.6.2
Flask-WTF==1.2.1
Flask-CORS==4.0.0
Flask-Mail==0.9.1
Flask-Migrate==4.0.5
Flask-RESTful==0.3.10
Flask-JWT-Extended==4.5.2
python-dotenv==1.0.0
stripe==8.0.0
Pillow==10.0.0
Flask-Caching==2.1.0
```

Installation des dépendances :
```bash
pip install -r backend/requirements.txt
```

## 3. Configuration Backend

### 3.1 Configuration de base (config.py)

```python
# backend/config.py

import os
from datetime import timedelta
from dotenv import load_dotenv

# Chargement des variables d'environnement
load_dotenv()

class Config:
    """Configuration de base de l'application Flask."""
    
    # Clé secrète pour Flask
    SECRET_KEY = os.environ.get('SECRET_KEY') or 'dev-key-change-me'
    
    # Configuration de la base de données
    SQLALCHEMY_DATABASE_URI = os.environ.get('DATABASE_URL') or \
        'sqlite:///fmp.db'
    SQLALCHEMY_TRACK_MODIFICATIONS = False
    
    # Configuration JWT
    JWT_SECRET_KEY = os.environ.get('JWT_SECRET_KEY') or 'jwt-secret-dev'
    JWT_ACCESS_TOKEN_EXPIRES = timedelta(hours=1)
    
    # Configuration des uploads
    UPLOAD_FOLDER = os.path.join('app', 'static', 'uploads')
    MAX_CONTENT_LENGTH = 16 * 1024 * 1024  # 16MB max-size
```

### 3.2 Initialisation de l'application (\_\_init\_\_.py)

```python
# backend/app/__init__.py

from flask import Flask
from flask_sqlalchemy import SQLAlchemy
from flask_jwt_extended import JWTManager
from flask_cors import CORS
from config import Config

# Initialisation des extensions
db = SQLAlchemy()
jwt = JWTManager()

def create_app(config_class=Config):
    """Création et configuration de l'application Flask."""
    
    # Initialisation de Flask
    app = Flask(__name__)
    app.config.from_object(config_class)
    
    # Initialisation des extensions
    db.init_app(app)
    jwt.init_app(app)
    CORS(app)
    
    # Une route de test
    @app.route('/test')
    def test():
        return {'message': 'Le serveur FMP fonctionne !'}
    
    return app
```

### 3.3 Point d'entrée (run.py)

```python
# backend/run.py

from app import create_app

app = create_app()

if __name__ == '__main__':
    app.run(debug=True)
```

## 4. Premier test

### 4.1 Lancement du serveur

```bash
# Assurez-vous d'être dans l'environnement virtuel
# Dans le dossier backend
python run.py
```

### 4.2 Test de l'application

Ouvrez votre navigateur et accédez à :
```
http://localhost:5000/test
```

Vous devriez voir le message :
```json
{
    "message": "Le serveur FMP fonctionne !"
}
```

## Prochaines étapes

Dans la prochaine partie, nous implémenterons :
1. Le système d'authentification
2. Le modèle utilisateur
3. Les routes d'authentification
4. La page de connexion/inscription