# FMP (For My Printer) - Documentation Technique

## Table des matières
- [Vue d'ensemble](#vue-densemble)
- [Architecture](#architecture)
- [Structure des dossiers](#structure-des-dossiers)
- [Technologies utilisées](#technologies-utilisées)
- [Guide d'installation](#guide-dinstallation)
- [Workflow de développement](#workflow-de-développement)

## Vue d'ensemble

FMP est une plateforme e-commerce spécialisée dans la vente de cartouches d'imprimantes. 
Le projet utilise une architecture simple avec Python (Flask) pour le backend et HTML/CSS/JavaScript pour le frontend.

## Architecture 

### Structure des dossiers
```
FMP/
├── docs/                  # Documentation
├── backend/              # API Python/Flask
└── frontend/             # Interface utilisateur HTML/JS/CSS
```

### Backend (Python/Flask)
```
backend/
├── app/
│   ├── __init__.py       # Configuration Flask
│   ├── routes/           # Points d'entrée API
│   ├── models/           # Modèles de données
│   ├── static/           # Fichiers statiques
│   └── templates/        # Templates email
├── tests/                # Tests unitaires
├── config.py             # Configuration
├── requirements.txt      # Dépendances
└── run.py               # Point d'entrée
```

### Frontend (HTML/JS/CSS)
```
frontend/
├── static/
│   ├── css/             # Styles CSS
│   ├── js/              # Scripts JavaScript
│   └── images/          # Images et assets
├── pages/               # Pages HTML
└── components/          # Composants réutilisables
```

## Technologies utilisées

### Frontend
- HTML5 : Structure des pages
- CSS3 : Styles et mise en page
- JavaScript : Interactivité et appels API
- LocalStorage : Stockage du panier

### Backend
- Python 3.8+ : Langage principal
- Flask : Framework web
- SQLite : Base de données
- JWT : Authentification

## Guide d'installation

### Prérequis
1. Python 3.8 ou supérieur
2. Éditeur de code (VS Code recommandé)
3. Git pour le versioning

### Installation

1. Cloner le projet :
```bash
git clone <URL_DU_REPO>
cd FMP
```

2. Configurer l'environnement Python :
```bash
# Créer l'environnement virtuel
python -m venv venv

# Activer l'environnement (Windows)
venv\Scripts\activate
# OU (Linux/Mac)
source venv/bin/activate

# Installer les dépendances
pip install -r backend/requirements.txt
```

3. Configuration de la base de données :
```python
# backend/config.py
import os

class Config:
    SECRET_KEY = 'votre-clé-secrète'
    SQLALCHEMY_DATABASE_URI = 'sqlite:///fmp.db'
    UPLOAD_FOLDER = 'static/uploads'
```

## Workflow de développement

### 1. Structure Git
- `main` : Code production
- `develop` : Développement
- `feature/*` : Nouvelles fonctionnalités

### 2. Commandes utiles
```bash
# Nouvelle branche
git checkout -b feature/ma-fonctionnalite

# Sauvegarder les modifications
git add .
git commit -m "Description"
git push origin feature/ma-fonctionnalite
```

### 3. Ordre de développement recommandé

1. Backend :
   - Configuration Flask
   - Modèles de données
   - Routes API
   - Tests

2. Frontend :
   - Structure HTML
   - Styles CSS de base
   - JavaScript pour les appels API
   - Panier et checkout

## Sécurité

### Mesures importantes
1. Validation des données (côtés client et serveur)
2. Protection contre les injections SQL
3. Authentification sécurisée
4. Validation des fichiers uploadés
5. Protection CSRF

### Configuration sécurité Flask
```python
# Configuration de base sécurisée
from flask import Flask
from flask_talisman import Talisman  # Sécurité headers HTTP

app = Flask(__name__)
Talisman(app)

# Limite taille upload
app.config['MAX_CONTENT_LENGTH'] = 16 * 1024 * 1024  # 16 MB max
```

## Exemples de code

### Backend (Python/Flask)
```python
# Routes produits (backend/app/routes/products.py)
from flask import Blueprint, jsonify
from app.models import Product

products = Blueprint('products', __name__)

@products.route('/api/products')
def get_products():
    products = Product.query.all()
    return jsonify([{
        'id': p.id,
        'name': p.name,
        'price': p.price,
        'image': p.image_url
    } for p in products])
```

### Frontend (JavaScript)
```javascript
// Appels API (frontend/static/js/api.js)
async function getProducts() {
    try {
        const response = await fetch('/api/products');
        return await response.json();
    } catch (error) {
        console.error('Erreur:', error);
        return [];
    }
}

// Gestion panier
function addToCart(product) {
    let cart = JSON.parse(localStorage.getItem('cart') || '[]');
    cart.push(product);
    localStorage.setItem('cart', JSON.stringify(cart));
}

