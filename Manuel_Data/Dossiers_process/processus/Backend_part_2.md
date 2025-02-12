# Guide Fondamental FMP - Partie 2
## Page 7 - Structure avancée de l'API

### 7.1 Organisation des Routes avec Blueprint
Dans Flask, les Blueprints permettent d'organiser les routes par fonctionnalité.

```python
# backend/app/routes.__init__.py
from .auth import auth_bp
from .products import products_bp
from .orders import orders_bp

def init_routes(app):
    app.register_blueprint(auth_bp)
    app.register_blueprint(products_bp)
    app.register_blueprint(orders_bp)

# backend/app/routes/products.py
from flask import Blueprint, jsonify, request
from app.models.product import Product
from app import db

# Création du Blueprint pour les produits
products = Blueprint('products', __name__)

@products.route('/api/products', methods=['GET'])
def get_products():
    # Route pour obtenir tous les produits
    products = Product.query.all()
    return jsonify([{
        'id': p.id,
        'name': p.name,
        'price': p.price,
        'description': p.description
    } for p in products])

@products.route('/api/products/<int:id>', methods=['GET'])
def get_product(id):
    # Route pour obtenir un produit spécifique
    product = Product.query.get_or_404(id)
    return jsonify({
        'id': product.id,
        'name': product.name,
        'price': product.price,
        'description': product.description
    })
```

### 7.2 Enregistrement des Blueprints
Dans __init__.py, nous devons enregistrer nos Blueprints :

```python
# backend/app/__init__.py
def create_app():
    app = Flask(__name__)
    app.config.from_object(Config)
    
    db.init_app(app)
    
    # Importer et enregistrer les blueprints
    from app.routes.products import products
    app.register_blueprint(products)
    
    return app
```

## Page 8 - Modèles de Données

### 8.1 Création des Modèles
```python
# backend/app/models/product.py
from app import db
from datetime import datetime

class Product(db.Model):
    """Modèle pour les cartouches d'imprimante"""
    id = db.Column(db.Integer, primary_key=True)
    name = db.Column(db.String(100), nullable=False)
    price = db.Column(db.Float, nullable=False)
    description = db.Column(db.Text)
    printer_model = db.Column(db.String(50))
    stock = db.Column(db.Integer, default=0)
    created_at = db.Column(db.DateTime, default=datetime.utcnow)

    def to_dict(self):
        """Convertit l'objet en dictionnaire"""
        return {
            'id': self.id,
            'name': self.name,
            'price': self.price,
            'description': self.description,
            'printer_model': self.printer_model,
            'stock': self.stock
        }
```

### 8.2 Initialisation de la Base de Données
```python
# backend/init_db.py
from app import create_app, db
from app.models.product import Product

app = create_app()

def init_db():
    with app.app_context():
        # Création de toutes les tables
        print("Création des tables...")
        db.create_all()
        
        # Ajout de données de test
        print("Ajout des produits de test...")
        products = [
            Product(
                name="Cartouche HP 304 Noire",
                price=19.99,
                description="Cartouche d'encre noire pour HP DeskJet",
                printer_model="HP DeskJet 3750",
                stock=10
            ),
            Product(
                name="Cartouche HP 304 Couleur",
                price=24.99,
                description="Cartouche d'encre couleur pour HP DeskJet",
                printer_model="HP DeskJet 3750",
                stock=8
            ),
            Product(
                name="Cartouche Canon PG-540",
                price=17.99,
                description="Cartouche d'encre noire pour Canon",
                printer_model="Canon PIXMA MG3650",
                stock=15
            )
        ]
        
        for product in products:
            db.session.add(product)
        
        db.session.commit()
        print("Base de données initialisée avec succès !")

if __name__ == '__main__':
    init_db()
```

## Page 9 - Opérations CRUD

### 9.1 Création d'un Produit
```python
@products.route('/api/products', methods=['POST'])
def create_product():
    data = request.get_json()
    
    new_product = Product(
        name=data['name'],
        price=data['price'],
        description=data.get('description', ''),
        printer_model=data.get('printer_model', ''),
        stock=data.get('stock', 0)
    )
    
    db.session.add(new_product)
    db.session.commit()
    
    return jsonify(new_product.to_dict()), 201
```

### 9.2 Mise à Jour d'un Produit
```python
@products.route('/api/products/<int:id>', methods=['PUT'])
def update_product(id):
    product = Product.query.get_or_404(id)
    data = request.get_json()
    
    product.name = data.get('name', product.name)
    product.price = data.get('price', product.price)
    product.description = data.get('description', product.description)
    product.printer_model = data.get('printer_model', product.printer_model)
    product.stock = data.get('stock', product.stock)
    
    db.session.commit()
    
    return jsonify(product.to_dict())
```

## Page 10 - Gestion des Erreurs

### 10.1 Configuration des Erreurs
```python
# backend/app/errors.py
from flask import jsonify

def register_error_handlers(app):
    @app.errorhandler(404)
    def not_found_error(error):
        return jsonify({
            'error': 'Resource not found',
            'message': str(error)
        }), 404

    @app.errorhandler(500)
    def internal_error(error):
        db.session.rollback()  # En cas d'erreur de base de données
        return jsonify({
            'error': 'Internal server error',
            'message': str(error)
        }), 500
```

### 10.2 Validation des Données
```python
from flask import abort

def validate_product_data(data):
    """Valide les données d'un produit"""
    if not data.get('name'):
        abort(400, description="Le nom du produit est requis")
    if not data.get('price'):
        abort(400, description="Le prix du produit est requis")
    if data.get('price') < 0:
        abort(400, description="Le prix ne peut pas être négatif")
```

## Page 11 - Tests

Les tests sont cruciaux pour s'assurer que tout fonctionne correctement.

### 11.1 Configuration des Tests
```python
# backend/tests/conftest.py
import pytest
from app import create_app, db

@pytest.fixture
def app():
    app = create_app()
    app.config['TESTING'] = True
    app.config['SQLALCHEMY_DATABASE_URI'] = 'sqlite:///:memory:'
    
    with app.app_context():
        db.create_all()
        yield app
        db.session.remove()
        db.drop_all()

@pytest.fixture
def client(app):
    return app.test_client()
```

### 11.2 Test des Routes
```python
# backend/tests/test_products.py
def test_get_products(client):
    """Test de la route GET /api/products"""
    response = client.get('/api/products')
    assert response.status_code == 200
    assert isinstance(response.json, list)
```

## Page 12 - Prochaines Étapes

Une fois ces bases en place, nous pourrons :
1. Développer l'authentification utilisateur
2. Créer le système de panier
3. Gérer les commandes
4. Intégrer un système de paiement
