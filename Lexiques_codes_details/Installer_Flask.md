# Guide d'Installation et Configuration d'un Serveur Flask

## 1. Installation des Dépendances

### Étape 1 : Créer l'environnement virtuel
```bash
# Créer l'environnement
python -m venv venv

# Activer l'environnement
# Windows:
venv\Scripts\activate
# Linux/Mac:
source venv/bin/activate
```

### Étape 2 : Installer les packages essentiels
```bash
# Packages principaux
pip install Flask==3.0.0
pip install SQLAlchemy==2.0.0
pip install Flask-SQLAlchemy==3.1.1
pip install python-dotenv==1.0.0

# Sécurité
pip install Flask-Login==0.6.3
pip install Flask-Bcrypt==1.0.1

# Base de données
pip install Flask-Migrate==4.0.5

# Utilitaires
pip install Flask-Mail==0.9.1
pip install Pillow==10.1.0
pip install Flask-Cors==4.0.0
```

## 2. Configuration de Base

### Structure du Projet
```
projet/
│
├── app/
│   ├── __init__.py      # Configuration Flask
│   ├── models/          # Modèles de données
│   ├── routes/          # Routes de l'application
│   └── templates/       # Templates HTML
│
├── config.py           # Configuration
├── .env               # Variables d'environnement
└── run.py             # Point d'entrée
```

### Configuration de Flask (config.py)
```python
import os
from dotenv import load_dotenv

load_dotenv()

class Config:
    # Configuration de base
    SECRET_KEY = os.getenv('SECRET_KEY', 'votre-cle-secrete')
    
    # Base de données
    SQLALCHEMY_DATABASE_URI = os.getenv('DATABASE_URL', 'sqlite:///app.db')
    SQLALCHEMY_TRACK_MODIFICATIONS = False
    
    # Email (optionnel)
    MAIL_SERVER = os.getenv('MAIL_SERVER', 'smtp.gmail.com')
    MAIL_PORT = int(os.getenv('MAIL_PORT', '587'))
    MAIL_USE_TLS = True
    MAIL_USERNAME = os.getenv('MAIL_USERNAME')
    MAIL_PASSWORD = os.getenv('MAIL_PASSWORD')
```

### Variables d'Environnement (.env)
```plaintext
SECRET_KEY=votre-cle-secrete-ici
DATABASE_URL=sqlite:///app.db
FLASK_ENV=development
FLASK_DEBUG=1
```

### Initialisation de l'Application (app/__init__.py)
```python
from flask import Flask
from flask_sqlalchemy import SQLAlchemy
from flask_login import LoginManager
from flask_mail import Mail
from config import Config

# Initialisation des extensions
db = SQLAlchemy()
login_manager = LoginManager()
mail = Mail()

def create_app():
    app = Flask(__name__)
    app.config.from_object(Config)
    
    # Initialisation des extensions
    db.init_app(app)
    login_manager.init_app(app)
    mail.init_app(app)
    
    # Enregistrement des blueprints
    from app.routes import main, auth
    app.register_blueprint(main.bp)
    app.register_blueprint(auth.bp)
    
    return app
```

### Point d'Entrée (run.py)
```python
from app import create_app

app = create_app()

if __name__ == '__main__':
    app.run(debug=True)
```

## 3. Utilisation des Extensions

### Base de Données (app/models/user.py)
```python
from app import db
from flask_login import UserMixin

class User(UserMixin, db.Model):
    id = db.Column(db.Integer, primary_key=True)
    username = db.Column(db.String(64), unique=True, nullable=False)
    email = db.Column(db.String(120), unique=True, nullable=False)
    password_hash = db.Column(db.String(128))
```

### Routes (app/routes/main.py)
```python
from flask import Blueprint, jsonify
from app.models import User

bp = Blueprint('main', __name__)

@bp.route('/')
def index():
    return jsonify({'message': 'Serveur opérationnel'})
```

## 4. Initialisation de la Base de Données

```python
# Dans un terminal Python
from app import create_app, db
app = create_app()
with app.app_context():
    db.create_all()
```

## 5. Démarrage du Serveur

```bash
# Dans le terminal
python run.py
```

## Descriptions des Dépendances

- **Flask** : Framework web léger pour Python
- **SQLAlchemy** : ORM pour la gestion de la base de données
- **Flask-Login** : Gestion de l'authentification utilisateur
- **Flask-Mail** : Envoi d'emails
- **Flask-Migrate** : Gestion des migrations de base de données
- **Flask-Bcrypt** : Hashage sécurisé des mots de passe
- **Python-dotenv** : Chargement des variables d'environnement

## Points Importants

1. **Sécurité**
   - Protégez toujours les mots de passe avec Bcrypt
   - Utilisez des variables d'environnement pour les secrets
   - Activez CSRF protection pour les formulaires

2. **Base de Données**
   - Faites des sauvegardes régulières
   - Utilisez les migrations pour les changements
   - Validez toujours les entrées utilisateur

3. **Déploiement**
   - Désactivez le mode debug en production
   - Utilisez un serveur WSGI (Gunicorn)
   - Configurez un proxy inverse (Nginx)
