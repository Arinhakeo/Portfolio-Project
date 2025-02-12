### 1. Routes Flask avec Blueprint

#### **Fichier : `backend/app/routes/products.py`**

```python
# Importation des outils Flask pour créer des routes, retourner des réponses JSON, et gérer les requêtes HTTP.
from flask import Blueprint, jsonify, request
# Importation du modèle "Product" pour interagir avec les données des produits.
from app.models.product import Product
# Importation de l'objet "db" pour gérer les opérations de base de données.
from app import db

# Création du Blueprint pour organiser les routes relatives aux produits.
products = Blueprint('products', __name__)

# Définition d'une route pour obtenir la liste des produits.
@products.route('/api/products', methods=['GET'])
def get_products():
    # Récupération de tous les produits à partir de la base de données.
    products = Product.query.all()
    return jsonify([{  # Conversion des produits en JSON pour les retourner au client.
        'id': p.id,  # ID unique du produit.
        'name': p.name,  # Nom du produit.
        'price': p.price,  # Prix du produit.
        'description': p.description  # Description du produit.
    } for p in products])  # Liste compréhension pour transformer chaque produit en dictionnaire.

# Route pour obtenir les détails d'un produit spécifique.
@products.route('/api/products/<int:id>', methods=['GET'])
def get_product(id):
    product = Product.query.get_or_404(id)  # Recherche du produit par ID; renvoie une erreur 404 si non trouvé.
    return jsonify({  # Conversion du produit trouvé en dictionnaire JSON.
        'id': product.id,  # ID unique du produit.
        'name': product.name,  # Nom du produit.
        'price': product.price,  # Prix du produit.
        'description': product.description  # Description du produit.
    })
```

---

### 2. Initialisation de l'Application et Enregistrement des Blueprints

#### **Fichier : `backend/app/__init__.py`**

```python
# Initialisation de l'application Flask.
def create_app():
    app = Flask(__name__)
    app.config.from_object(Config)  # Chargement des configurations depuis l'objet Config.
    
    db.init_app(app)  # Initialisation de l'objet de base de données avec l'application Flask.
    
    # Importation et enregistrement des blueprints pour modulariser les routes.
    from app.routes.products import products
    app.register_blueprint(products)  # Enregistrement du Blueprint "products" dans l'application Flask.
    
    return app  # Retourne l'application Flask configurée.
```

---

### 3. Modèle de Base de Données pour les Produits

#### **Fichier : `backend/app/models/product.py`**

```python
from app import db
from datetime import datetime

class Product(db.Model):  # Classe représentant le modèle "Product" pour les produits en base de données.
    id = db.Column(db.Integer, primary_key=True)  # Identifiant unique du produit.
    name = db.Column(db.String(100), nullable=False)  # Nom du produit (obligatoire).
    price = db.Column(db.Float, nullable=False)  # Prix du produit (obligatoire).
    description = db.Column(db.Text)  # Description du produit (facultative).
    printer_model = db.Column(db.String(50))  # Modèle d'imprimante compatible (facultatif).
    stock = db.Column(db.Integer, default=0)  # Quantité en stock, par défaut 0.
    created_at = db.Column(db.DateTime, default=datetime.utcnow)  # Date de création automatique du produit.

    def to_dict(self):
        # Méthode pour convertir un produit en dictionnaire JSON.
        return {
            'id': self.id,
            'name': self.name,
            'price': self.price,
            'description': self.description,
            'printer_model': self.printer_model,
            'stock': self.stock
        }
```

---

### 4. Initialisation de la Base de Données

#### **Fichier : `backend/init_db.py`**

```python
from app import create_app, db
from app.models.product import Product

app = create_app()

def init_db():
    with app.app_context():  # Création d'un contexte d'application pour accéder à la base de données.
        db.create_all()  # Création de toutes les tables définies par les modèles SQLAlchemy.
        
        # Ajout de données de test pour initialiser la base de données.
        test_product = Product(
            name="Cartouche HP 304 Noire",
            price=19.99,
            description="Cartouche d'encre noire pour HP",
            printer_model="HP DeskJet 3750",
            stock=10
        )
        
        db.session.add(test_product)  # Ajout du produit au contexte de session.
        db.session.commit()  # Validation des modifications dans la base de données.

if __name__ == '__main__':
    init_db()  # Exécution de la fonction d'initialisation lorsque le script est lancé directement.
```

---

### 5. Gestion des Erreurs

#### **Fichier : `backend/app/errors.py`**

```python
from flask import jsonify

def register_error_handlers(app):
    @app.errorhandler(404)
    def not_found_error(error):
        # Gestion des erreurs 404 avec un message JSON clair.
        return jsonify({
            'error': 'Resource not found',
            'message': str(error)
        }), 404

    @app.errorhandler(500)
    def internal_error(error):
        db.session.rollback()  # Annule les transactions en cas d'erreur de base de données.
        return jsonify({
            'error': 'Internal server error',
            'message': str(error)
        }), 500
```

---

### 6. Tests

#### **Configuration des Tests : `backend/tests/conftest.py`**

```python
import pytest
from app import create_app, db

@pytest.fixture
# Initialisation de l'application Flask en mode test.
def app():
    app = create_app()
    app.config['TESTING'] = True
    app.config['SQLALCHEMY_DATABASE_URI'] = 'sqlite:///:memory:'  # Base de données en mémoire pour les tests.
    
    with app.app_context():
        db.create_all()  # Création des tables pour les tests.
        yield app
        db.session.remove()
        db.drop_all()  # Nettoyage après les tests.

@pytest.fixture
# Retourne un client de test pour simuler les requêtes HTTP.
def client(app):
    return app.test_client()
```

#### **Tests des Routes : `backend/tests/test_products.py`**

```python
def test_get_products(client):
    # Test de la route GET /api/products pour s'assurer qu'elle retourne un statut 200 et une liste de produits.
    response = client.get('/api/products')
    assert response.status_code == 200
    assert isinstance(response.json, list)  # Vérifie que la réponse est une liste JSON.
```
**


Voici une version détaillée et commentée de votre guide fondamental avec des explications exhaustives de chaque bloc de code. J'ajoute également une synthèse pour chaque partie afin de résumer le rôle de chaque élément et de faciliter votre compréhension.

---



Voici une version détaillée et commentée de votre guide fondamental avec des explications exhaustives de chaque bloc de code. J'ajoute également une synthèse pour chaque partie afin de résumer le rôle de chaque élément et de faciliter votre compréhension.

---

# **Guide Fondamental FMP - Partie 2 : Structure avancée de l'API**

## **Page 7 - Organisation des Routes et Blueprints**

### **7.1 Organisation des Routes avec Blueprint**
Les Blueprints dans Flask permettent de structurer et modulariser les routes. Cela améliore la maintenabilité et facilite le développement collaboratif.

#### **Code : Organisation des Routes**
```python
# backend/app/routes/products.py
from flask import Blueprint, jsonify, request  # Import des outils Flask pour les routes et les requêtes.
from app.models.product import Product  # Import du modèle Product pour interagir avec la base de données.
from app import db  # Import de l'objet db pour les opérations sur la base de données.

# Création du Blueprint pour regrouper toutes les routes liées aux produits.
products = Blueprint('products', __name__)

@products.route('/api/products', methods=['GET'])
def get_products():
    """
    Route pour obtenir tous les produits.
    Cette route retourne une liste de produits au format JSON.
    """
    # Récupération de tous les produits depuis la base de données.
    products = Product.query.all()
    
    # Conversion des produits en liste de dictionnaires JSON.
    return jsonify([
        {
            'id': p.id,
            'name': p.name,
            'price': p.price,
            'description': p.description
        } for p in products
    ])

@products.route('/api/products/<int:id>', methods=['GET'])
def get_product(id):
    """
    Route pour obtenir un produit spécifique via son ID.
    Si l'ID n'existe pas, une erreur 404 est retournée.
    """
    # Recherche du produit par ID. Renvoie une erreur 404 si l'ID n'existe pas.
    product = Product.query.get_or_404(id)
    
    # Conversion du produit en dictionnaire JSON.
    return jsonify({
        'id': product.id,
        'name': product.name,
        'price': product.price,
        'description': product.description
    })
```

---

### **7.2 Enregistrement des Blueprints**
Pour activer les Blueprints dans l'application, ils doivent être enregistrés dans le fichier principal `__init__.py`.

#### **Code : Enregistrement des Blueprints**
```python
# backend/app/__init__.py
def create_app():
    """
    Fonction principale pour créer et configurer l'application Flask.
    """
    app = Flask(__name__)  # Initialisation de l'application Flask.
    app.config.from_object(Config)  # Chargement de la configuration.

    db.init_app(app)  # Initialisation de la base de données avec l'application.

    # Importation et enregistrement des Blueprints
    from app.routes.products import products  # Import du Blueprint des produits.
    app.register_blueprint(products)  # Enregistrement du Blueprint pour activer les routes.

    return app  # Retourne l'application Flask configurée.
```

---

## **Page 8 - Modèles de Données**

### **8.1 Création des Modèles**
Le modèle représente la structure de la table dans la base de données et définit ses attributs ainsi que les méthodes associées.

#### **Code : Modèle Product**
```python
# backend/app/models/product.py
from app import db  # Import de l'objet db pour définir les modèles.
from datetime import datetime  # Utilisé pour ajouter une date de création.

class Product(db.Model):
    """
    Modèle représentant un produit dans la base de données.
    Chaque produit est associé à des cartouches d'imprimantes.
    """
    id = db.Column(db.Integer, primary_key=True)  # Clé primaire unique.
    name = db.Column(db.String(100), nullable=False)  # Nom du produit (obligatoire).
    price = db.Column(db.Float, nullable=False)  # Prix du produit (obligatoire).
    description = db.Column(db.Text)  # Description du produit (facultatif).
    printer_model = db.Column(db.String(50))  # Modèle d'imprimante compatible (facultatif).
    stock = db.Column(db.Integer, default=0)  # Quantité en stock (par défaut : 0).
    created_at = db.Column(db.DateTime, default=datetime.utcnow)  # Date de création par défaut.

    def to_dict(self):
        """
        Convertit un produit en dictionnaire JSON.
        Utilisé pour retourner les données au client sous forme de JSON.
        """
        return {
            'id': self.id,
            'name': self.name,
            'price': self.price,
            'description': self.description,
            'printer_model': self.printer_model,
            'stock': self.stock
        }
```

---

### **8.2 Initialisation de la Base de Données**
Une fonction d'initialisation est créée pour configurer la base de données et insérer des données de test.

#### **Code : Initialisation de la Base de Données**
```python
# backend/init_db.py
from app import create_app, db  # Import de l'application et de l'objet db.
from app.models.product import Product  # Import du modèle Product.

app = create_app()  # Création de l'application Flask.

def init_db():
    """
    Initialise la base de données.
    Crée les tables et insère des données de test.
    """
    with app.app_context():  # Création d'un contexte Flask pour accéder à la base de données.
        db.create_all()  # Création de toutes les tables définies par les modèles.

        # Ajout d'un produit de test.
        test_product = Product(
            name="Cartouche HP 304 Noire",
            price=19.99,
            description="Cartouche d'encre noire pour HP",
            printer_model="HP DeskJet 3750",
            stock=10
        )

        db.session.add(test_product)  # Ajout du produit à la session.
        db.session.commit()  # Enregistrement dans la base de données.

if __name__ == '__main__':
    init_db()  # Lancement de l'initialisation si ce fichier est exécuté directement.
```

---

## **Page 9 - Opérations CRUD**

### **9.1 Création d'un Produit**
Une route POST permet de créer un produit en recevant les données du client.

#### **Code : Création d'un Produit**
```python
@products.route('/api/products', methods=['POST'])
def create_product():
    """
    Crée un nouveau produit à partir des données reçues dans la requête.
    Retourne le produit créé en JSON avec le statut 201 (créé).
    """
    data = request.get_json()  # Récupère les données JSON de la requête.

    # Création d'un nouvel objet produit.
    new_product = Product(
        name=data['name'],
        price=data['price'],
        description=data.get('description', ''),  # Description par défaut vide.
        printer_model=data.get('printer_model', ''),  # Modèle d'imprimante par défaut vide.
        stock=data.get('stock', 0)  # Stock par défaut : 0.
    )

    db.session.add(new_product)  # Ajout du produit dans la session.
    db.session.commit()  # Validation des modifications.

    return jsonify(new_product.to_dict()), 201  # Retourne le produit en JSON avec le statut 201.
```

---

### **9.2 Mise à Jour d'un Produit**
Une route PUT permet de mettre à jour un produit existant.

#### **Code : Mise à Jour d'un Produit**
```python
@products.route('/api/products/<int:id>', methods=['PUT'])
def update_product(id):
    """
    Met à jour un produit existant à partir des données reçues.
    Retourne le produit mis à jour en JSON.
    """
    product = Product.query.get_or_404(id)  # Recherche le produit par ID ou renvoie une erreur 404.
    data = request.get_json()  # Récupère les données JSON de la requête.

    # Mise à jour des champs avec les données reçues ou maintien des valeurs existantes.
    product.name = data.get('name', product.name)
    product.price = data.get('price', product.price)
    product.description = data.get('description', product.description)
    product.printer_model = data.get('printer_model', product.printer_model)
    product.stock = data.get('stock', product.stock)

    db.session.commit()  # Validation des modifications.

    return jsonify(product.to_dict())  # Retourne le produit mis à jour.
```

---

## **Page 10 - Gestion des Erreurs**

### **10.1 Configuration des Erreurs**
Les erreurs sont gérées pour fournir des messages clairs au client.

#### **Code : Gestion des Erreurs**
```python
# backend/app/errors.py
from flask import jsonify

def register_error_handlers(app):
    """
    Enregistre les gestionnaires d'erreurs pour l'application Flask.
    """
    @app.errorhandler(404)
    def not_found_error(error):
        """
        Gère les erreurs 404 (ressource introuvable).
        """
        return jsonify({
            'error': 'Resource not found',
            'message': str(error)
        }), 404

    @app.errorhandler(500)
    def internal_error(error):
        """
        Gère les erreurs 500 (erreur serveur).
        """
        db.session.rollback()  # Annule les transactions en cas d'erreur

.
        return jsonify({
            'error': 'Internal server error',
            'message': str(error)
        }), 500
```

---

### **10.2 Validation des Données**
Une fonction dédiée valide les données reçues avant de créer ou de mettre à jour un produit.

#### **Code : Validation des Données**
```python
from flask import abort

def validate_product_data(data):
    """
    Valide les données reçues pour un produit.
    Lève une erreur 400 si les données sont invalides.
    """
    if not data.get('name'):
        abort(400, description="Le nom du produit est requis")
    if not data.get('price'):
        abort(400, description="Le prix du produit est requis")
    if data.get('price') < 0:
        abort(400, description="Le prix ne peut pas être négatif")
```

---

## **Page 11 - Tests**

### **11.1 Configuration des Tests**
Les tests utilisent `pytest` pour simuler les appels à l'application.

#### **Code : Configuration des Tests**
```python
# backend/tests/conftest.py
import pytest
from app import create_app, db

@pytest.fixture
def app():
    """
    Configure l'application pour les tests.
    Utilise une base de données en mémoire pour les tests.
    """
    app = create_app()
    app.config['TESTING'] = True
    app.config['SQLALCHEMY_DATABASE_URI'] = 'sqlite:///:memory:'  # Base de données en mémoire.

    with app.app_context():
        db.create_all()  # Crée les tables pour les tests.
        yield app
        db.session.remove()  # Nettoie la session après les tests.
        db.drop_all()  # Supprime les tables.

@pytest.fixture
def client(app):
    """
    Fournit un client de test pour simuler les requêtes.
    """
    return app.test_client()
```

---

### **11.2 Test des Routes**
Les tests vérifient que les routes fonctionnent comme prévu.

#### **Code : Test des Routes**
```python
# backend/tests/test_products.py
def test_get_products(client):
    """
    Test de la route GET /api/products.
    Vérifie que le statut est 200 et que la réponse est une liste.
    """
    response = client.get('/api/products')  # Appel à la route.
    assert response.status_code == 200  # Vérifie que le statut est 200.
    assert isinstance(response.json, list)  # Vérifie que la réponse est une liste.
```

---

### **Synthèse**
- **Blueprints :** Organisent les routes par fonctionnalité.
- **Modèles :** Définissent la structure des tables dans la base de données.
- **CRUD :** Implémente les opérations (création, lecture, mise à jour, suppression).
- **Erreurs :** Gère les erreurs pour fournir des messages clairs.
- **Tests :** Vérifient que l'application fonctionne correctement.
