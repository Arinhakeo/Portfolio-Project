# Guide Complet - Création de FMP (For My Printer)

## Structure du Projet

La structure complète du projet sera la suivante :
```
FMP/
├── backend/              # Partie serveur Python/Flask
│   ├── app/             # Code principal de l'application
│   │   ├── __init__.py  # Configuration Flask
│   │   ├── routes/      # Points d'entrée API
│   │   └── models/      # Modèles de données
│   ├── config.py        # Configuration
│   └── run.py           # Point d'entrée
└── frontend/            # Interface utilisateur
    ├── static/          # Fichiers statiques
    └── pages/           # Pages HTML
```

## 1. Installation et Configuration Initiale

### 1.1 Prérequis
```bash
# Installation des outils nécessaires
# 1. Python 3.8+ : https://www.python.org/downloads/
# 2. VS Code : https://code.visualstudio.com/
# 3. Git : https://git-scm.com/downloads

# Vérifier l'installation de Python
python --version  # Doit afficher Python 3.8 ou supérieur
```

### 1.2 Création de l'environnement
```bash
# Créer le dossier principal
mkdir FMP
cd FMP

# Créer l'environnement virtuel
python -m venv venv

# Activer l'environnement (Windows)
venv\Scripts\activate
# OU pour Linux/Mac
# source venv/bin/activate

# Installer les dépendances
pip install flask flask-sqlalchemy python-dotenv

# Sauvegarder les dépendances
pip freeze > requirements.txt
```

## 2. Structure Backend Détaillée

### 2.1 Configuration de Base (backend/config.py)
```python
import os
from dotenv import load_dotenv

# Charger les variables d'environnement du fichier .env
load_dotenv()

class Config:
    # Clé secrète pour la sécurité (sessions, tokens)
    SECRET_KEY = os.environ.get('SECRET_KEY') or 'clé-dev-temporaire'
    
    # Configuration de la base de données SQLite
    SQLALCHEMY_DATABASE_URI = 'sqlite:///fmp.db'
    
    # Désactiver le suivi des modifications de SQLAlchemy (optimisation)
    SQLALCHEMY_TRACK_MODIFICATIONS = False
    CORS_HEADERS = 'Content-Type'
    CORS_RESOURCES = {r"/api/*": {"origins": "*"}}
```

### 2.2 Point d'Entrée (backend/run.py)
```python
from app import create_app

# Créer l'instance de l'application
app = create_app()

if __name__ == '__main__':
    # Lancer le serveur en mode développement
    app.run(debug=True)
```

### 2.3 Application Flask (backend/app/__init__.py)
```python
from flask import Flask, jsonify
from flask_cors import CORS
from flask_sqlalchemy import SQLAlchemy
from config import Config

# Initialiser SQLAlchemy
db = SQLAlchemy()

def create_app():
    # Créer l'application Flask
    app = Flask(__name__)
    CORS(app)
    
    # Charger la configuration
    app.config.from_object(Config)
    
    # Initialiser la base de données avec l'application
    db.init_app(app)
    
    # Route racine pour vérifier que l'API fonctionne
    @app.route('/')
    def home():
        return jsonify({
            'message': 'Bienvenue sur l\'API de FMP',
            'status': 'online',
            'endpoints_disponibles': [
                '/test',
                '/api/products',  # Futur endpoint
                '/api/users'      # Futur endpoint
            ]
        })

    # Route de test
    @app.route('/test')
    def test():
        return jsonify({
            'message': 'API FMP fonctionnelle !',
            'status': 'success'
        })

    return app
```

### 2.4 Fichier .env (backend/.env) et requirements.txt:
```plaintext
# Variables d'environnement .env
FLASK_APP=run.py
FLASK_ENV=development
SECRET_KEY=votre-cle-secrete
DATABASE_URL=sqlite:///fmp.db
MAIL_SERVER=smtp.gmail.com
MAIL_PORT=587
MAIL_USE_TLS=True
MAIL_USERNAME=votre@email.com
MAIL_PASSWORD=votre-mot-de-passe
```
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

## 3. Explications Détaillées du Code

### 3.1 Configuration (config.py)
- `load_dotenv()` : Charge les variables d'environnement du fichier .env
- `SECRET_KEY` : Clé pour sécuriser les sessions et tokens
- `SQLALCHEMY_DATABASE_URI` : Définit l'emplacement de la base de données
- `SQLALCHEMY_TRACK_MODIFICATIONS` : Désactive une fonctionnalité non nécessaire de SQLAlchemy

### 3.2 Application (__init__.py)
- `Flask(__name__)` : Crée une nouvelle instance Flask
- `app.config.from_object(Config)` : Charge la configuration depuis la classe Config
- `db.init_app(app)` : Initialise SQLAlchemy avec notre application
- `jsonify()` : Convertit les dictionnaires Python en réponses JSON pour l'API

### 3.3 Point d'Entrée (run.py)
- `create_app()` : Crée et configure l'application Flask
- `debug=True` : Active le mode développement (rechargement automatique)

## 4. Organisation des Routes et Modèles

### 4.1 Structure des Routes
Les routes seront organisées par fonctionnalité :
```python
# backend/app/routes/products.py
from flask import Blueprint, jsonify

products = Blueprint('products', __name__)

@products.route('/api/products')
def get_products():
    return jsonify([])  # Liste vide pour l'instant
```

### 4.2 Structure des Modèles
Les modèles définissent la structure de la base de données :
```python
# backend/app/models/product.py
from app import db

class Product(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    name = db.Column(db.String(100), nullable=False)
    price = db.Column(db.Float, nullable=False)
    description = db.Column(db.Text)
```

## 5. Démarrage et Tests

### 5.1 Lancement du Serveur
```bash
cd backend
python run.py
```

### 5.2 Vérification
1. Ouvrir http://127.0.0.1:5000/ dans le navigateur
2. Vous devriez voir le message de bienvenue en JSON
3. Tester http://127.0.0.1:5000/test pour vérifier la route de test

## 6. Étapes Suivantes

1. Création des modèles de base de données
2. Mise en place des routes API complètes
3. Développement du frontend
4. Intégration frontend/backend
5. Tests et déploiement

## Points Importants

1. **Sécurité**
   - Ne jamais commiter le fichier .env
   - Toujours utiliser des variables d'environnement pour les secrets
   - Valider toutes les entrées utilisateur

2. **Organisation**
   - Un fichier par modèle
   - Une route = une responsabilité
   - Commentaires clairs et en français

3. **Bonnes Pratiques**
   - Commits réguliers et descriptifs
   - Tests avant chaque déploiement
   - Documentation des APIs
