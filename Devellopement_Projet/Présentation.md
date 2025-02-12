# Structure du Projet FMP (For My Printer)

## Organisation des Dossiers

```
FMP/
├── backend/
│   ├── app/
│   │   ├── __init__.py           # Initialisation de l'application Flask
│   │   ├── routes/
│   │   │   ├── __init__.py
│   │   │   ├── auth.py           # Routes authentification
│   │   │   ├── products.py       # Routes produits
│   │   │   └── orders.py         # Routes commandes
│   │   ├── models/
│   │   │   ├── __init__.py
│   │   │   ├── user.py          # Modèle utilisateur
│   │   │   ├── product.py       # Modèle produit
│   │   │   └── order.py         # Modèle commande
│   │   ├── static/
│   │   │   └── uploads/         # Images produits
│   │   └── templates/           # Templates pour les emails
│   ├── config.py                # Configuration
│   ├── requirements.txt         # Dépendances Python
│   └── run.py                   # Point d'entrée de l'application
│
└── frontend/
    ├── static/
    │   ├── css/
    │   │   ├── style.css        # Styles principaux
    │   │   └── responsive.css    # Styles responsifs
    │   ├── js/
    │   │   ├── main.js          # JavaScript principal
    │   │   ├── cart.js          # Logique du panier
    │   │   └── api.js           # Appels API
    │   └── images/              # Images du site
    ├── pages/
    │   ├── index.html           # Page d'accueil
    │   ├── products.html        # Liste des produits
    │   ├── product.html         # Page produit individuel
    │   ├── cart.html            # Panier
    │   └── account.html         # Espace utilisateur
    └── components/              # Morceaux de HTML réutilisables
        ├── header.html
        ├── footer.html
        └── product-card.html

```
## Technologies Utilisées

### Frontend
- HTML5
- CSS3
- JavaScript (Vanilla)
- Fetch API pour les appels au backend
- LocalStorage pour le panier

### Backend (Python)
- Flask (framework web léger)
- SQLite (base de données)
- SQLAlchemy (ORM)
- JWT pour l'authentification

## Installation et Configuration

1. Configuration de l'environnement Python :
```bash
# Créer un environnement virtuel
python -m venv venv

# Activer l'environnement
# Sur Windows :
venv\Scripts\activate
# Sur Linux/Mac :
source venv/bin/activate

# Installer les dépendances
pip install -r backend/requirements.txt
```

2. Structure du fichier requirements.txt :
```
Flask==2.0.1
Flask-SQLAlchemy==2.5.1
Flask-Login==0.5.0
Flask-JWT-Extended==4.2.3
Pillow==8.3.1  # Pour la gestion des images
python-dotenv==0.19.0
```

3. Configuration de la base de données (config.py) :
```python
import os
from dotenv import load_dotenv

load_dotenv()

class Config:
    SECRET_KEY = os.environ.get('SECRET_KEY') or 'votre-clé-secrète'
    SQLALCHEMY_DATABASE_URI = os.environ.get('DATABASE_URL') or 'sqlite:///fmp.db'
    SQLALCHEMY_TRACK_MODIFICATIONS = False
```

## Exemples de Code

### Backend (Python/Flask)

1. Route produits (backend/app/routes/products.py) :
```python
from flask import Blueprint, jsonify, request
from app.models.product import Product
from app import db

products = Blueprint('products', __name__)

@products.route('/api/products', methods=['GET'])
def get_products():
    products = Product.query.all()
    return jsonify([{
        'id': p.id,
        'name': p.name,
        'price': p.price,
        'description': p.description,
        'image': p.image_url
    } for p in products])
```

### Frontend (JavaScript)

1. Appel API (frontend/static/js/api.js) :
```javascript
// Fonction pour récupérer les produits
async function getProducts() {
    try {
        const response = await fetch('/api/products');
        const products = await response.json();
        return products;
    } catch (error) {
        console.error('Erreur:', error);
        return [];
    }
}

// Fonction pour gérer le panier
function addToCart(product) {
    let cart = JSON.parse(localStorage.getItem('cart') || '[]');
    cart.push(product);
    localStorage.setItem('cart', JSON.stringify(cart));
}
```

2. Affichage produits (frontend/static/js/main.js) :
```javascript
async function displayProducts() {
    const productsContainer = document.getElementById('products-container');
    const products = await getProducts();
    
    products.forEach(product => {
        const productElement = document.createElement('div');
        productElement.className = 'product-card';
        productElement.innerHTML = `
            <img src="${product.image}" alt="${product.name}">
            <h3>${product.name}</h3>
            <p>${product.price}€</p>
            <button onclick="addToCart(${JSON.stringify(product)})">
                Ajouter au panier
            </button>
        `;
        productsContainer.appendChild(productElement);
    });
}
```

## Étapes Suivantes

1. Commencer par créer la structure des dossiers
2. Mettre en place le backend Python/Flask
3. Créer les pages HTML statiques
4. Ajouter le JavaScript pour la dynamisation
5. Styliser avec CSS
6. Tester et déployer