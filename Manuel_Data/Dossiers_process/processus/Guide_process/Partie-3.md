# Partie 3 : Gestion des Produits
## Structure et Modèles de Données

### Description Générale
Cette partie gère :
- Le catalogue de produits
- Les catégories
- Les marques d'imprimantes
- Les images de produits
- La gestion des stocks

### 3.1 Structure des Dossiers
```
backend/
├── app/
│   ├── products/
│   │   ├── __init__.py
│   │   ├── models.py       # Modèles de produits et catégories
│   │   ├── routes.py       # Routes API produits
│   │   ├── schemas.py      # Schémas de validation
│   │   └── utils.py        # Utilitaires
│   └── static/
│       └── uploads/
│           └── products/    # Images des produits
```

### 3.2 Modèles de Données

```python
# backend/app/products/models.py

from datetime import datetime
from app import db
from slugify import slugify

class Category(db.Model):
    """Modèle pour les catégories de produits.
    
    Attributs:
        id (int): Identifiant unique
        name (str): Nom de la catégorie
        slug (str): Slug pour l'URL
        description (str): Description de la catégorie
        image_url (str): URL de l'image de la catégorie
        parent_id (int): ID de la catégorie parente (pour sous-catégories)
        created_at (datetime): Date de création
        updated_at (datetime): Date de dernière modification
    """
    
    __tablename__ = 'categories'

    id = db.Column(db.Integer, primary_key=True)
    name = db.Column(db.String(100), nullable=False)
    slug = db.Column(db.String(100), unique=True, nullable=False)
    description = db.Column(db.Text)
    image_url = db.Column(db.String(255))
    parent_id = db.Column(db.Integer, db.ForeignKey('categories.id'))
    created_at = db.Column(db.DateTime, default=datetime.utcnow)
    updated_at = db.Column(db.DateTime, default=datetime.utcnow, onupdate=datetime.utcnow)

    # Relations
    products = db.relationship('Product', backref='category', lazy=True)
    subcategories = db.relationship(
        'Category',
        backref=db.backref('parent', remote_side=[id]),
        lazy=True
    )

    def __init__(self, name, description=None, image_url=None, parent_id=None):
        self.name = name
        self.slug = slugify(name)
        self.description = description
        self.image_url = image_url
        self.parent_id = parent_id

    def to_dict(self):
        """Convertit la catégorie en dictionnaire."""
        return {
            'id': self.id,
            'name': self.name,
            'slug': self.slug,
            'description': self.description,
            'image_url': self.image_url,
            'parent_id': self.parent_id,
            'created_at': self.created_at.isoformat(),
            'updated_at': self.updated_at.isoformat()
        }

class Brand(db.Model):
    """Modèle pour les marques d'imprimantes.
    
    Attributs:
        id (int): Identifiant unique
        name (str): Nom de la marque
        slug (str): Slug pour l'URL
        description (str): Description de la marque
        logo_url (str): URL du logo de la marque
        website (str): Site web officiel
        created_at (datetime): Date de création
    """
    
    __tablename__ = 'brands'

    id = db.Column(db.Integer, primary_key=True)
    name = db.Column(db.String(100), nullable=False)
    slug = db.Column(db.String(100), unique=True, nullable=False)
    description = db.Column(db.Text)
    logo_url = db.Column(db.String(255))
    website = db.Column(db.String(255))
    created_at = db.Column(db.DateTime, default=datetime.utcnow)

    # Relations
    products = db.relationship('Product', backref='brand', lazy=True)

    def __init__(self, name, description=None, logo_url=None, website=None):
        self.name = name
        self.slug = slugify(name)
        self.description = description
        self.logo_url = logo_url
        self.website = website

    def to_dict(self):
        """Convertit la marque en dictionnaire."""
        return {
            'id': self.id,
            'name': self.name,
            'slug': self.slug,
            'description': self.description,
            'logo_url': self.logo_url,
            'website': self.website,
            'created_at': self.created_at.isoformat()
        }

class Product(db.Model):
    """Modèle pour les produits (cartouches d'imprimante).
    
    Attributs:
        id (int): Identifiant unique
        sku (str): Référence unique du produit
        name (str): Nom du produit
        slug (str): Slug pour l'URL
        description (str): Description détaillée
        short_description (str): Description courte
        price (float): Prix de vente
        stock_quantity (int): Quantité en stock
        min_stock_level (int): Niveau minimum de stock
        category_id (int): ID de la catégorie
        brand_id (int): ID de la marque
        is_active (bool): Produit actif/inactif
        created_at (datetime): Date de création
        updated_at (datetime): Date de dernière modification
    """
    
    __tablename__ = 'products'

    id = db.Column(db.Integer, primary_key=True)
    sku = db.Column(db.String(50), unique=True, nullable=False)
    name = db.Column(db.String(200), nullable=False)
    slug = db.Column(db.String(200), unique=True, nullable=False)
    description = db.Column(db.Text)
    short_description = db.Column(db.String(500))
    price = db.Column(db.Float, nullable=False)
    stock_quantity = db.Column(db.Integer, default=0)
    min_stock_level = db.Column(db.Integer, default=5)
    category_id = db.Column(db.Integer, db.ForeignKey('categories.id'), nullable=False)
    brand_id = db.Column(db.Integer, db.ForeignKey('brands.id'), nullable=False)
    is_active = db.Column(db.Boolean, default=True)
    created_at = db.Column(db.DateTime, default=datetime.utcnow)
    updated_at = db.Column(db.DateTime, default=datetime.utcnow, onupdate=datetime.utcnow)

    # Relations
    images = db.relationship('ProductImage', backref='product', lazy=True, cascade='all, delete-orphan')
    specifications = db.relationship('ProductSpecification', backref='product', lazy=True, cascade='all, delete-orphan')

    def __init__(self, sku, name, price, category_id, brand_id, **kwargs):
        self.sku = sku
        self.name = name
        self.slug = slugify(name)
        self.price = price
        self.category_id = category_id
        self.brand_id = brand_id
        
        # Attributs optionnels
        self.description = kwargs.get('description')
        self.short_description = kwargs.get('short_description')
        self.stock_quantity = kwargs.get('stock_quantity', 0)
        self.min_stock_level = kwargs.get('min_stock_level', 5)
        self.is_active = kwargs.get('is_active', True)

    def to_dict(self):
        """Convertit le produit en dictionnaire."""
        return {
            'id': self.id,
            'sku': self.sku,
            'name': self.name,
            'slug': self.slug,
            'description': self.description,
            'short_description': self.short_description,
            'price': self.price,
            'stock_quantity': self.stock_quantity,
            'min_stock_level': self.min_stock_level,
            'category': self.category.to_dict(),
            'brand': self.brand.to_dict(),
            'is_active': self.is_active,
            'images': [img.to_dict() for img in self.images],
            'specifications': [spec.to_dict() for spec in self.specifications],
            'created_at': self.created_at.isoformat(),
            'updated_at': self.updated_at.isoformat()
        }

class ProductImage(db.Model):
    """Modèle pour les images de produits.
    
    Attributs:
        id (int): Identifiant unique
        product_id (int): ID du produit associé
        url (str): URL de l'image
        alt (str): Texte alternatif
        is_primary (bool): Image principale ou non
        position (int): Position dans la galerie
        created_at (datetime): Date de création
    """
    
    __tablename__ = 'product_images'

    id = db.Column(db.Integer, primary_key=True)
    product_id = db.Column(db.Integer, db.ForeignKey('products.id'), nullable=False)
    url = db.Column(db.String(255), nullable=False)
    alt = db.Column(db.String(200))
    is_primary = db.Column(db.Boolean, default=False)
    position = db.Column(db.Integer, default=0)
    created_at = db.Column(db.DateTime, default=datetime.utcnow)

    def to_dict(self):
        """Convertit l'image en dictionnaire."""
        return {
            'id': self.id,
            'url': self.url,
            'alt': self.alt,
            'is_primary': self.is_primary,
            'position': self.position,
            'created_at': self.created_at.isoformat()
        }

class ProductSpecification(db.Model):
    """Modèle pour les spécifications techniques des produits.
    
    Attributs:
        id (int): Identifiant unique
        product_id (int): ID du produit associé
        name (str): Nom de la spécification
        value (str): Valeur de la spécification
        unit (str): Unité de mesure (optionnel)
        position (int): Position dans la liste
    """
    
    __tablename__ = 'product_specifications'

    id = db.Column(db.Integer, primary_key=True)
    product_id = db.Column(db.Integer, db.ForeignKey('products.id'), nullable=False)
    name = db.Column(db.String(100), nullable=False)
    value = db.Column(db.String(200), nullable=False)
    unit = db.Column(db.String(50))
    position = db.Column(db.Integer, default=0)

    def to_dict(self):
        """Convertit la spécification en dictionnaire."""
        return {
            'id': self.id,
            'name': self.name,
            'value': self.value,
            'unit': self.unit,
            'position': self.position
        }
```

### 3.3 Schémas de Validation

```python
# backend/app/products/schemas.py

from marshmallow import Schema, fields, validate

class CategorySchema(Schema):
    """Schéma de validation pour les catégories."""
    
    id = fields.Int(dump_only=True)
    name = fields.Str(required=True, validate=validate.Length(min=2, max=100))
    description = fields.Str()
    image_url = fields.Str()
    parent_id = fields.Int(allow_none=True)

class BrandSchema(Schema):
    """Schéma de validation pour les marques."""
    
    id = fields.Int(dump_only=True)
    name = fields.Str(required=True, validate=validate.Length(min=2, max=100))
    description = fields.Str()
    logo_url = fields.Str()
    website = fields.Str()

class ProductImageSchema(Schema):
    """Schéma de validation pour les images de produits."""
    
    id = fields.Int(dump_only=True)
    url = fields.Str(required=True)
    alt = fields.Str()
    is_primary = fields.Bool()
    position = fields.Int()

class ProductSpecificationSchema(Schema):
    """Schéma de validation pour les spécifications produits."""
    
    id = fields.Int(dump_only=True)
    name = fields.Str(required=True, validate=validate.Length(min=2, max=100))
    value = fields.Str(required=True)
    unit = fields.Str()
    position = fields.Int()

class ProductSchema(Schema):
    """Schéma de validation pour les produits."""
    
    id = fields.Int(dump_only=True)
    sku = fields.Str(required=True, validate=validate.Length(min=3, max=50))
    name = fields.Str(required=True, validate=validate.Length(min=3, max=200))
    description = fields.Str()
    short_description = fields.Str(validate=validate.Length(max=500))
    price = fields.Float(required=True, validate=validate.Range(min=0))
    stock_quantity = fields.Int()
    min_stock_level = fields.Int()
    category_id = fields.Int(required=True)
    brand_id = fields.Int(required=True)
    is_active = fields.Bool()
    images = fields.Nested(ProductImageSchema, many=True)
    specifications = fields.Nested(ProductSpecificationSchema, many=True)
```

# Partie 3 : Gestion des Produits
## Routes API Produits

### 3.4 Routes API (backend/app/products/routes.py)

```python
# backend/app/products/routes.py

from flask import Blueprint, request, jsonify
from flask_jwt_extended import jwt_required, get_jwt_identity
from app import db
from app.products.models import Product, Category, Brand, ProductImage
from app.products.schemas import ProductSchema, CategorySchema, BrandSchema
from app.auth.models import User
from app.products.utils import save_product_image, delete_product_image

# Création du Blueprint
products_bp = Blueprint('products', __name__, url_prefix='/api/products')

# Initialisation des schémas
product_schema = ProductSchema()
products_schema = ProductSchema(many=True)
category_schema = CategorySchema()
categories_schema = CategorySchema(many=True)
brand_schema = BrandSchema()
brands_schema = BrandSchema(many=True)

# =========================================
# Routes pour les produits
# =========================================

@products_bp.route('/', methods=['GET'])
def get_products():
    """Récupère la liste des produits avec filtres optionnels.
    
    Query Params:
        category (str): Filtre par catégorie
        brand (str): Filtre par marque
        search (str): Recherche dans le nom/description
        sort (str): Tri par champ
        page (int): Numéro de page
        limit (int): Nombre d'éléments par page
    """
    try:
        # Récupération des paramètres
        category = request.args.get('category')
        brand = request.args.get('brand')
        search = request.args.get('search')
        sort = request.args.get('sort', 'name')
        page = int(request.args.get('page', 1))
        limit = int(request.args.get('limit', 20))

        # Construction de la requête
        query = Product.query.filter_by(is_active=True)

        # Application des filtres
        if category:
            query = query.join(Category).filter(Category.slug == category)
        
        if brand:
            query = query.join(Brand).filter(Brand.slug == brand)
        
        if search:
            search_term = f"%{search}%"
            query = query.filter(
                (Product.name.ilike(search_term)) |
                (Product.description.ilike(search_term)) |
                (Product.sku.ilike(search_term))
            )

        # Tri
        if sort.startswith('-'):
            query = query.order_by(getattr(Product, sort[1:]).desc())
        else:
            query = query.order_by(getattr(Product, sort).asc())

        # Pagination
        pagination = query.paginate(page=page, per_page=limit, error_out=False)
        products = pagination.items

        # Préparation de la réponse
        response = {
            'items': products_schema.dump(products),
            'total': pagination.total,
            'pages': pagination.pages,
            'current_page': page
        }

        return jsonify(response), 200

    except Exception as e:
        return jsonify({'error': str(e)}), 500

@products_bp.route('/<string:slug>', methods=['GET'])
def get_product(slug):
    """Récupère les détails d'un produit par son slug."""
    try:
        product = Product.query.filter_by(slug=slug, is_active=True).first_or_404()
        return jsonify(product_schema.dump(product)), 200

    except Exception as e:
        return jsonify({'error': str(e)}), 500

@products_bp.route('/', methods=['POST'])
@jwt_required()
def create_product():
    """Crée un nouveau produit (admin seulement)."""
    try:
        # Vérification des droits admin
        current_user_id = get_jwt_identity()
        user = User.query.get(current_user_id)
        
        if not user or not user.is_admin:
            return jsonify({'error': 'Accès non autorisé'}), 403

        # Validation des données
        data = request.get_json()
        errors = product_schema.validate(data)
        if errors:
            return jsonify({'error': errors}), 400

        # Création du produit
        product = Product(
            sku=data['sku'],
            name=data['name'],
            price=data['price'],
            category_id=data['category_id'],
            brand_id=data['brand_id'],
            description=data.get('description'),
            short_description=data.get('short_description'),
            stock_quantity=data.get('stock_quantity', 0),
            min_stock_level=data.get('min_stock_level', 5)
        )

        # Ajout des spécifications
        if 'specifications' in data:
            for spec in data['specifications']:
                product.specifications.append(
                    ProductSpecification(**spec)
                )

        db.session.add(product)
        db.session.commit()

        return jsonify(product_schema.dump(product)), 201

    except Exception as e:
        db.session.rollback()
        return jsonify({'error': str(e)}), 500

@products_bp.route('/<int:product_id>', methods=['PUT'])
@jwt_required()
def update_product(product_id):
    """Met à jour un produit existant (admin seulement)."""
    try:
        # Vérification des droits admin
        current_user_id = get_jwt_identity()
        user = User.query.get(current_user_id)
        
        if not user or not user.is_admin:
            return jsonify({'error': 'Accès non autorisé'}), 403

        # Récupération du produit
        product = Product.query.get_or_404(product_id)

        # Validation des données
        data = request.get_json()
        errors = product_schema.validate(data, partial=True)
        if errors:
            return jsonify({'error': errors}), 400

        # Mise à jour des champs
        for field in ['name', 'description', 'short_description', 'price', 
                     'stock_quantity', 'min_stock_level', 'category_id', 
                     'brand_id', 'is_active']:
            if field in data:
                setattr(product, field, data[field])

        # Mise à jour des spécifications
        if 'specifications' in data:
            # Suppression des anciennes spécifications
            product.specifications = []
            
            # Ajout des nouvelles spécifications
            for spec in data['specifications']:
                product.specifications.append(
                    ProductSpecification(**spec)
                )

        db.session.commit()

        return jsonify(product_schema.dump(product)), 200

    except Exception as e:
        db.session.rollback()
        return jsonify({'error': str(e)}), 500

@products_bp.route('/<int:product_id>/images', methods=['POST'])
@jwt_required()
def upload_product_image(product_id):
    """Ajoute une image à un produit."""
    try:
        # Vérification des droits admin
        current_user_id = get_jwt_identity()
        user = User.query.get(current_user_id)
        
        if not user or not user.is_admin:
            return jsonify({'error': 'Accès non autorisé'}), 403

        # Vérification du produit
        product = Product.query.get_or_404(product_id)

        # Vérification du fichier
        if 'image' not in request.files:
            return jsonify({'error': 'Aucune image fournie'}), 400

        file = request.files['image']
        
        if not file.filename:
            return jsonify({'error': 'Nom de fichier invalide'}), 400

        # Sauvegarde de l'image
        image_url = save_product_image(file)

        # Création de l'entrée dans la base
        image = ProductImage(
            product_id=product_id,
            url=image_url,
            alt=request.form.get('alt', product.name),
            is_primary=not bool(product.images),  # Premier = image principale
            position=len(product.images)
        )

        db.session.add(image)
        db.session.commit()

        return jsonify({
            'message': 'Image ajoutée avec succès',
            'image': image.to_dict()
        }), 201

    except Exception as e:
        db.session.rollback()
        return jsonify({'error': str(e)}), 500

@products_bp.route('/<int:product_id>/images/<int:image_id>', methods=['DELETE'])
@jwt_required()
def delete_image(product_id, image_id):
    """Supprime une image d'un produit."""
    try:
        # Vérification des droits admin
        current_user_id = get_jwt_identity()
        user = User.query.get(current_user_id)
        
        if not user or not user.is_admin:
            return jsonify({'error': 'Accès non autorisé'}), 403

        # Vérification de l'image
        image = ProductImage.query.filter_by(
            id=image_id, 
            product_id=product_id
        ).first_or_404()

        # Suppression du fichier
        delete_product_image(image.url)

        # Suppression de l'entrée
        db.session.delete(image)
        db.session.commit()

        return jsonify({
            'message': 'Image supprimée avec succès'
        }), 200

    except Exception as e:
        db.session.rollback()
        return jsonify({'error': str(e)}), 500

# =========================================
# Routes pour les catégories
# =========================================

@products_bp.route('/categories', methods=['GET'])
def get_categories():
    """Récupère la liste des catégories."""
    try:
        categories = Category.query.all()
        return jsonify(categories_schema.dump(categories)), 200

    except Exception as e:
        return jsonify({'error': str(e)}), 500

@products_bp.route('/categories', methods=['POST'])
@jwt_required()
def create_category():
    """Crée une nouvelle catégorie (admin seulement)."""
    try:
        # Vérification des droits admin
        current_user_id = get_jwt_identity()
        user = User.query.get(current_user_id)
        
        if not user or not user.is_admin:
            return jsonify({'error': 'Accès non autorisé'}), 403

        # Validation des données
        data = request.get_json()
        errors = category_schema.validate(data)
        if errors:
            return jsonify({'error': errors}), 400

        # Création de la catégorie
        category = Category(**data)
        
        db.session.add(category)
        db.session.commit()

        return jsonify(category_schema.dump(category)), 201

    except Exception as e:
        db.session.rollback()
        return jsonify({'error': str(e)}), 500

# =========================================
# Routes pour les marques
# =========================================

@products_bp.route('/brands', methods=['GET'])
def get_brands():
    """Récupère la liste des marques."""
    try:
        brands = Brand.query.all()
        return jsonify(brands_schema.dump(brands)), 200

    except Exception as e:
        return jsonify({'error': str(e)}), 500

@products_bp.route('/brands', methods=['POST'])
@jwt_required()
def create_brand():
    """Crée une nouvelle marque (admin seulement)."""
    try:
        # Vérification des droits admin
        current_user_id = get_jwt_identity()
        user = User.query.get(current_user_id)
        
        if not user or not user.is_admin:
            return jsonify({'error': 'Accès non autorisé'}), 403

        # Validation des données
        data = request.get_json()
        errors = brand_schema.validate(data)
        if errors:
            return jsonify({'error': errors}), 400

        # Création de la marque
        brand = Brand(**data)
        
        db.session.add(brand)
        db.session.commit()

        return jsonify(brand_schema.dump(brand)), 201

    except Exception as e:
        db.session.rollback()
        return jsonify({'error': str(e)}), 500
```

### 3.5 Enregistrement des Routes

Pour enregistrer ces routes, nous devons mettre à jour le fichier `__init__.py` :

```python
# backend/app/products/__init__.py

from flask import Blueprint

products_bp = Blueprint('products', __name__)

from . import routes  # Import des routes
```

Puis dans l'application principale :

```python
# backend/app/__init__.py

def create_app(config_name='default'):
    # ...
    
    # Import et enregistrement du blueprint products
    from app.products import products_bp
    app.register_blueprint(products_bp, url_prefix='/api/products')
    
    # ...
    return app
```

# Partie 3 : Gestion des Produits
## Interface d'Administration

### 3.9 Structure des Fichiers Admin
```
frontend/
├── admin/
│   ├── pages/
│   │   ├── products/
│   │   │   ├── index.html     # Liste des produits
│   │   │   ├── create.html    # Création de produit
│   │   │   └── edit.html      # Édition de produit
│   │   ├── categories/
│   │   │   ├── index.html     # Gestion des catégories
│   │   │   └── edit.html      # Édition catégorie
│   │   └── brands/
│   │       ├── index.html     # Gestion des marques
│   │       └── edit.html      # Édition marque
│   ├── static/
│   │   ├── css/
│   │   │   ├── admin.css      # Styles admin
│   │   │   └── forms.css      # Styles formulaires
│   │   └── js/
│   │       ├── products.js     # Gestion produits
│   │       ├── categories.js   # Gestion catégories
│   │       └── brands.js       # Gestion marques
│   └── index.html             # Dashboard admin
```

### 3.10 Page Liste des Produits
```html
<!-- frontend/admin/pages/products/index.html -->
<!DOCTYPE html>
<html lang="fr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>FMP Admin - Gestion des Produits</title>
    <link rel="stylesheet" href="../../static/css/admin.css">
    <link rel="stylesheet" href="../../static/css/forms.css">
    
    <!-- Vérification des droits admin -->
    <script type="module">
        import { session } from '../../../static/js/session.js';
        
        // Redirection si non admin
        const userData = session.getUserData();
        if (!userData?.is_admin) {
            window.location.href = '/login.html';
        }
    </script>
</head>
<body>
    <!-- Menu latéral admin -->
    <aside class="admin-sidebar">
        <div class="admin-brand">
            <h1>FMP Admin</h1>
        </div>
        <nav class="admin-nav">
            <ul>
                <li>
                    <a href="../dashboard.html">
                        <i class="icon dashboard-icon"></i>
                        Dashboard
                    </a>
                </li>
                <li>
                    <a href="./index.html" class="active">
                        <i class="icon products-icon"></i>
                        Produits
                    </a>
                </li>
                <li>
                    <a href="../categories/index.html">
                        <i class="icon categories-icon"></i>
                        Catégories
                    </a>
                </li>
                <li>
                    <a href="../brands/index.html">
                        <i class="icon brands-icon"></i>
                        Marques
                    </a>
                </li>
                <li>
                    <a href="../orders/index.html">
                        <i class="icon orders-icon"></i>
                        Commandes
                    </a>
                </li>
                <li>
                    <a href="../users/index.html">
                        <i class="icon users-icon"></i>
                        Utilisateurs
                    </a>
                </li>
            </ul>
        </nav>
    </aside>

    <!-- Contenu principal -->
    <main class="admin-main">
        <!-- En-tête -->
        <header class="admin-header">
            <div class="header-content">
                <h2>Gestion des Produits</h2>
                <a href="./create.html" class="btn btn-primary">
                    Nouveau Produit
                </a>
            </div>
        </header>

        <!-- Filtres et recherche -->
        <section class="filters-section">
            <div class="search-box">
                <input 
                    type="text" 
                    id="search-input" 
                    placeholder="Rechercher un produit..."
                >
            </div>
            
            <div class="filters">
                <select id="category-filter">
                    <option value="">Toutes les catégories</option>
                </select>
                
                <select id="brand-filter">
                    <option value="">Toutes les marques</option>
                </select>
                
                <select id="stock-filter">
                    <option value="">Tous les stocks</option>
                    <option value="in_stock">En stock</option>
                    <option value="low_stock">Stock bas</option>
                    <option value="out_of_stock">Rupture de stock</option>
                </select>
            </div>
        </section>

        <!-- Liste des produits -->
        <section class="products-list">
            <table class="admin-table">
                <thead>
                    <tr>
                        <th>Image</th>
                        <th>SKU</th>
                        <th>Nom</th>
                        <th>Catégorie</th>
                        <th>Marque</th>
                        <th>Prix</th>
                        <th>Stock</th>
                        <th>Statut</th>
                        <th>Actions</th>
                    </tr>
                </thead>
                <tbody id="products-table-body">
                    <!-- Les produits seront injectés ici -->
                </tbody>
            </table>

            <!-- Pagination -->
            <div class="pagination" id="pagination">
                <!-- La pagination sera injectée ici -->
            </div>
        </section>
    </main>

    <!-- Scripts -->
    <script type="module" src="../../static/js/products.js"></script>
</body>
</html>
```

### 3.11 JavaScript de Gestion des Produits
```javascript
// frontend/admin/static/js/products.js

import { session } from '../../../static/js/session.js';
import { notifications } from '../../../static/js/notifications.js';

class ProductsManager {
    constructor() {
        this.currentPage = 1;
        this.itemsPerPage = 20;
        this.filters = {
            search: '',
            category: '',
            brand: '',
            stock: ''
        };
        
        this.initialize();
    }

    async initialize() {
        // Chargement des données initiales
        await this.loadCategories();
        await this.loadBrands();
        await this.loadProducts();
        
        // Mise en place des écouteurs d'événements
        this.setupEventListeners();
    }

    setupEventListeners() {
        // Recherche
        const searchInput = document.getElementById('search-input');
        searchInput.addEventListener('input', this.debounce(() => {
            this.filters.search = searchInput.value;
            this.currentPage = 1;
            this.loadProducts();
        }, 300));

        // Filtres
        const filters = ['category', 'brand', 'stock'];
        filters.forEach(filter => {
            const element = document.getElementById(`${filter}-filter`);
            element.addEventListener('change', () => {
                this.filters[filter] = element.value;
                this.currentPage = 1;
                this.loadProducts();
            });
        });
    }

    async loadCategories() {
        try {
            const response = await fetch('/api/products/categories');
            const categories = await response.json();
            
            const select = document.getElementById('category-filter');
            categories.forEach(category => {
                const option = document.createElement('option');
                option.value = category.slug;
                option.textContent = category.name;
                select.appendChild(option);
            });
        } catch (error) {
            console.error('Erreur chargement catégories:', error);
            notifications.create({
                type: 'error',
                title: 'Erreur',
                message: 'Impossible de charger les catégories'
            });
        }
    }

    async loadBrands() {
        try {
            const response = await fetch('/api/products/brands');
            const brands = await response.json();
            
            const select = document.getElementById('brand-filter');
            brands.forEach(brand => {
                const option = document.createElement('option');
                option.value = brand.slug;
                option.textContent = brand.name;
                select.appendChild(option);
            });
        } catch (error) {
            console.error('Erreur chargement marques:', error);
            notifications.create({
                type: 'error',
                title: 'Erreur',
                message: 'Impossible de charger les marques'
            });
        }
    }

    async loadProducts() {
        try {
            // Construction de l'URL avec les filtres
            const queryParams = new URLSearchParams({
                page: this.currentPage,
                limit: this.itemsPerPage,
                ...this.filters
            });

            const response = await fetch(`/api/products?${queryParams}`);
            const data = await response.json();
            
            this.renderProducts(data.items);
            this.renderPagination(data);
            
        } catch (error) {
            console.error('Erreur chargement produits:', error);
            notifications.create({
                type: 'error',
                title: 'Erreur',
                message: 'Impossible de charger les produits'
            });
        }
    }

    renderProducts(products) {
        const tbody = document.getElementById('products-table-body');
        tbody.innerHTML = products.map(product => `
            <tr>
                <td>
                    <img 
                        src="${product.images[0]?.url || '/static/images/no-image.png'}" 
                        alt="${product.name}"
                        class="product-thumbnail"
                    >
                </td>
                <td>${product.sku}</td>
                <td>${product.name}</td>
                <td>${product.category.name}</td>
                <td>${product.brand.name}</td>
                <td>${product.price.toFixed(2)} €</td>
                <td>
                    <span class="stock-badge ${this.getStockClass(product.stock_quantity)}">
                        ${product.stock_quantity}
                    </span>
                </td>
                <td>
                    <span class="status-badge ${product.is_active ? 'active' : 'inactive'}">
                        ${product.is_active ? 'Actif' : 'Inactif'}
                    </span>
                </td>
                <td class="actions">
                    <button 
                        class="btn btn-icon" 
                        onclick="window.location.href='./edit.html?id=${product.id}'"
                        title="Modifier"
                    >
                        <i class="icon edit-icon"></i>
                    </button>
                    <button 
                        class="btn btn-icon" 
                        onclick="productsManager.toggleProductStatus(${product.id})"
                        title="${product.is_active ? 'Désactiver' : 'Activer'}"
                    >
                        <i class="icon ${product.is_active ? 'disable-icon' : 'enable-icon'}"></i>
                    </button>
                </td>
            </tr>
        `).join('');
    }

    renderPagination(data) {
        const pagination = document.getElementById('pagination');
        const totalPages = Math.ceil(data.total / this.itemsPerPage);
        
        // Création des boutons de pagination
        let html = '';
        
        // Bouton précédent
        html += `
            <button 
                class="pagination-btn"
                ${this.currentPage === 1 ? 'disabled' : ''}
                onclick="productsManager.goToPage(${this.currentPage - 1})"
            >
                Précédent
            </button>
        `;
        
        // Pages
        for (let i = 1; i <= totalPages; i++) {
            if (
                i === 1 || 
                i === totalPages || 
                (i >= this.currentPage - 2 && i <= this.currentPage + 2)
            ) {
                html += `
                    <button 
                        class="pagination-btn ${i === this.currentPage ? 'active' : ''}"
                        onclick="productsManager.goToPage(${i})"
                    >
                        ${i}
                    </button>
                `;
            } else if (
                i === this.currentPage - 3 || 
                i === this.currentPage + 3
            ) {
                html += '<span class="pagination-ellipsis">...</span>';
            }
        }
        
        // Bouton suivant
        html += `
            <button 
                class="pagination-btn"
                ${this.currentPage === totalPages ? 'disabled' : ''}
                onclick="productsManager.goToPage(${this.currentPage + 1})"
            >
                Suivant
            </button>
        `;
        
        pagination.innerHTML = html;
    }

    async toggleProductStatus(productId) {
        try {
            const response = await fetch(`/api/products/${productId}`, {
                method: 'PUT',
                headers: {
                    'Content-Type': 'application/json',
                    'Authorization': `Bearer ${session.getToken()}`
                },
                body: JSON.stringify({
                    is_active: !document.querySelector(`tr[data-id="${productId}"]`).classList.contains('active')
                })
            });

            if (!response.ok) {
                throw new Error('Erreur lors de la modification du statut');
            }

            // Rechargement des produits
            await this.loadProducts();
            
            notifications.create({
                type: 'success',
                title: 'Succès',
                message: 'Statut du produit modifié'
            });

        } catch (error) {
            console.error('Erreur modification statut:', error);
            notifications.create({
                type: 'error',
                title: 'Erreur',
                message: error.message
            });
        }
    }

    getStockClass(quantity) {
        if (quantity <= 0) return 'out-of-stock';
        if (quantity <= 5) return 'low-stock';
        return 'in-stock';
    }

    goToPage(page) {
        this.currentPage = page;
        this.loadProducts();
    }

    debounce(func, wait) {
        let timeout;
        return function executedFunction(...args) {
            const later = () => {
                clearTimeout(timeout);
                func(...args);
            };
            clearTimeout(timeout);
            timeout = setTimeout(later, wait);
        };
    }
}

// Initialisation
const productsManager = new ProductsManager();
// Rendre l'instance accessible globalement
window.productsManager = productsManager;

```

# Suite de l'Interface d'Administration

// Styles CSS pour l'interface d'administration

### 3.12 Styles Admin
```css
/* frontend/admin/static/css/admin.css */

/* Layout principal */
body {
    display: grid;
    grid-template-columns: 250px 1fr;
    min-height: 100vh;
    background: #f5f7fa;
}

/* Sidebar */
.admin-sidebar {
    background: #2c3e50;
    color: white;
    padding: 1rem;
    position: fixed;
    top: 0;
    left: 0;
    bottom: 0;
    width: 250px;
    overflow-y: auto;
}

.admin-brand {
    padding: 1rem;
    margin-bottom: 2rem;
    border-bottom: 1px solid rgba(255, 255, 255, 0.1);
}

.admin-brand h1 {
    font-size: 1.5rem;
    color: white;
    margin: 0;
}

.admin-nav ul {
    list-style: none;
    padding: 0;
    margin: 0;
}

.admin-nav li {
    margin-bottom: 0.5rem;
}

.admin-nav a {
    display: flex;
    align-items: center;
    padding: 0.8rem 1rem;
    color: rgba(255, 255, 255, 0.8);
    text-decoration: none;
    border-radius: 4px;
    transition: all 0.3s;
}

.admin-nav a:hover,
.admin-nav a.active {
    background: rgba(255, 255, 255, 0.1);
    color: white;
}

.admin-nav .icon {
    margin-right: 0.8rem;
    width: 20px;
    height: 20px;
}

/* Contenu principal */
.admin-main {
    margin-left: 250px;
    padding: 2rem;
}

.admin-header {
    background: white;
    padding: 1rem 2rem;
    margin: -2rem -2rem 2rem;
    box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

.header-content {
    display: flex;
    justify-content: space-between;
    align-items: center;
}

/* Filtres */
.filters-section {
    background: white;
    padding: 1rem;
    border-radius: 8px;
    margin-bottom: 2rem;
    box-shadow: 0 2px 4px rgba(0, 0, 0, 0.05);
}

.search-box {
    margin-bottom: 1rem;
}

.search-box input {
    width: 100%;
    padding: 0.8rem;
    border: 1px solid #ddd;
    border-radius: 4px;
    font-size: 1rem;
}

.filters {
    display: flex;
    gap: 1rem;
}

.filters select {
    padding: 0.8rem;
    border: 1px solid #ddd;
    border-radius: 4px;
    font-size: 1rem;
    min-width: 150px;
}

/* Table des produits */
.admin-table {
    width: 100%;
    background: white;
    border-radius: 8px;
    overflow: hidden;
    box-shadow: 0 2px 4px rgba(0, 0, 0, 0.05);
}

.admin-table th {
    background: #f8f9fa;
    padding: 1rem;
    text-align: left;
    font-weight: 600;
    color: #2c3e50;
}

.admin-table td {
    padding: 1rem;
    border-top: 1px solid #eee;
}

.product-thumbnail {
    width: 50px;
    height: 50px;
    object-fit: cover;
    border-radius: 4px;
}

/* Badges de statut */
.stock-badge {
    display: inline-block;
    padding: 0.25rem 0.5rem;
    border-radius: 4px;
    font-size: 0.875rem;
}

.stock-badge.in-stock {
    background: #e8f5e9;
    color: #2e7d32;
}

.stock-badge.low-stock {
    background: #fff3e0;
    color: #f57c00;
}

.stock-badge.out-of-stock {
    background: #ffebee;
    color: #c62828;
}

.status-badge {
    display: inline-block;
    padding: 0.25rem 0.5rem;
    border-radius: 4px;
    font-size: 0.875rem;
}

.status-badge.active {
    background: #e3f2fd;
    color: #1565c0;
}

.status-badge.inactive {
    background: #f5f5f5;
    color: #616161;
}

/* Actions */
.actions {
    display: flex;
    gap: 0.5rem;
}

.btn-icon {
    width: 32px;
    height: 32px;
    padding: 0;
    display: flex;
    align-items: center;
    justify-content: center;
    border-radius: 4px;
    border: none;
    background: none;
    cursor: pointer;
    transition: background 0.3s;
}

.btn-icon:hover {
    background: #f5f5f5;
}

/* Pagination */
.pagination {
    display: flex;
    justify-content: center;
    gap: 0.5rem;
    margin-top: 2rem;
}

.pagination-btn {
    padding: 0.5rem 1rem;
    border: 1px solid #ddd;
    background: white;
    border-radius: 4px;
    cursor: pointer;
    transition: all 0.3s;
}

.pagination-btn:hover:not(:disabled) {
    background: #f5f5f5;
    border-color: #ccc;
}

.pagination-btn.active {
    background: #3498db;
    color: white;
    border-color: #3498db;
}

.pagination-btn:disabled {
    opacity: 0.5;
    cursor: not-allowed;
}

.pagination-ellipsis {
    padding: 0.5rem;
    color: #666;
}

/* Boutons */
.btn {
    padding: 0.8rem 1.5rem;
    border-radius: 4px;
    border: none;
    font-size: 1rem;
    cursor: pointer;
    transition: all 0.3s;
}

.btn-primary {
    background: #3498db;
    color: white;
}

.btn-primary:hover {
    background: #2980b9;
}

/* Formulaires d'édition */
.form-section {
    background: white;
    padding: 2rem;
    border-radius: 8px;
    box-shadow: 0 2px 4px rgba(0, 0, 0, 0.05);
    margin-bottom: 2rem;
}

.form-grid {
    display: grid;
    grid-template-columns: repeat(2, 1fr);
    gap: 1.5rem;
}

.form-group {
    margin-bottom: 1.5rem;
}

.form-group label {
    display: block;
    margin-bottom: 0.5rem;
    font-weight: 500;
}

.form-group input,
.form-group textarea,
.form-group select {
    width: 100%;
    padding: 0.8rem;
    border: 1px solid #ddd;
    border-radius: 4px;
    font-size: 1rem;
}

.form-group textarea {
    min-height: 150px;
    resize: vertical;
}

/* Images produit */
.product-images {
    display: grid;
    grid-template-columns: repeat(auto-fill, minmax(120px, 1fr));
    gap: 1rem;
    margin-top: 1rem;
}

.image-item {
    position: relative;
    border-radius: 4px;
    overflow: hidden;
}

.image-item img {
    width: 100%;
    height: 120px;
    object-fit: cover;
}

.image-actions {
    position: absolute;
    top: 0;
    right: 0;
    padding: 0.5rem;
    background: rgba(0, 0, 0, 0.5);
    border-radius: 0 0 0 4px;
}

.image-actions button {
    background: none;
    border: none;
    color: white;
    cursor: pointer;
    padding: 0.25rem;
}

/* Responsive */
@media (max-width: 1024px) {
    .form-grid {
        grid-template-columns: 1fr;
    }
}

@media (max-width: 768px) {
    body {
        grid-template-columns: 1fr;
    }

    .admin-sidebar {
        display: none;
    }

    .admin-main {
        margin-left: 0;
    }

    .filters {
        flex-direction: column;
    }

    .admin-table {
        display: block;
        overflow-x: auto;
    }
}
```

# Partie 3 : Gestion des Produits
## 3.13 Page de Création/Édition de Produit

### Description
Cette section comprend :
- Le formulaire de création/édition de produit
- La gestion des images
- La gestion des spécifications
- Les validations de formulaire

### 3.13.1 HTML du Formulaire
```html
<!-- frontend/admin/pages/products/edit.html -->
<!DOCTYPE html>
<html lang="fr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>FMP Admin - Édition Produit</title>
    <link rel="stylesheet" href="../../static/css/admin.css">
    <link rel="stylesheet" href="../../static/css/forms.css">
    
    <!-- Vérification admin -->
    <script type="module">
        import { session } from '../../../static/js/session.js';
        const userData = session.getUserData();
        if (!userData?.is_admin) {
            window.location.href = '/login.html';
        }
    </script>
</head>
<body>
    <!-- Menu latéral admin (comme dans index.html) -->
    <aside class="admin-sidebar">
        <!-- ... contenu du menu ... -->
    </aside>

    <!-- Contenu principal -->
    <main class="admin-main">
        <!-- En-tête -->
        <header class="admin-header">
            <div class="header-content">
                <h2 id="page-title">Nouveau Produit</h2>
                <div class="header-actions">
                    <button class="btn btn-secondary" onclick="history.back()">
                        Annuler
                    </button>
                    <button class="btn btn-primary" onclick="productEditor.saveProduct()">
                        Enregistrer
                    </button>
                </div>
            </div>
        </header>

        <!-- Formulaire principal -->
        <form id="product-form" class="form-section" onsubmit="return false;">
            <!-- Informations de base -->
            <div class="form-grid">
                <div class="form-group">
                    <label for="sku">Référence (SKU)*</label>
                    <input 
                        type="text" 
                        id="sku" 
                        name="sku" 
                        required
                        placeholder="Ex: HP-LAS-001"
                    >
                    <small class="form-hint">
                        La référence doit être unique
                    </small>
                </div>

                <div class="form-group">
                    <label for="name">Nom du produit*</label>
                    <input 
                        type="text" 
                        id="name" 
                        name="name" 
                        required
                        placeholder="Ex: Cartouche HP LaserJet Pro"
                    >
                </div>

                <div class="form-group">
                    <label for="category_id">Catégorie*</label>
                    <select id="category_id" name="category_id" required>
                        <option value="">Sélectionner une catégorie</option>
                    </select>
                </div>

                <div class="form-group">
                    <label for="brand_id">Marque*</label>
                    <select id="brand_id" name="brand_id" required>
                        <option value="">Sélectionner une marque</option>
                    </select>
                </div>

                <div class="form-group">
                    <label for="price">Prix (€)*</label>
                    <input 
                        type="number" 
                        id="price" 
                        name="price" 
                        step="0.01" 
                        min="0" 
                        required
                        placeholder="0.00"
                    >
                </div>

                <div class="form-group">
                    <label for="stock_quantity">Quantité en stock*</label>
                    <input 
                        type="number" 
                        id="stock_quantity" 
                        name="stock_quantity" 
                        min="0" 
                        required
                        placeholder="0"
                    >
                </div>

                <div class="form-group">
                    <label for="min_stock_level">Niveau minimum de stock</label>
                    <input 
                        type="number" 
                        id="min_stock_level" 
                        name="min_stock_level" 
                        min="0" 
                        placeholder="5"
                    >
                </div>

                <div class="form-group">
                    <label class="checkbox-label">
                        <input 
                            type="checkbox" 
                            id="is_active" 
                            name="is_active" 
                            checked
                        >
                        Produit actif
                    </label>
                </div>
            </div>

            <!-- Description -->
            <div class="form-group">
                <label for="short_description">Description courte</label>
                <input 
                    type="text" 
                    id="short_description" 
                    name="short_description"
                    maxlength="500"
                    placeholder="Brève description du produit"
                >
                <small class="form-hint">
                    Maximum 500 caractères
                </small>
            </div>

            <div class="form-group">
                <label for="description">Description détaillée</label>
                <textarea 
                    id="description" 
                    name="description"
                    rows="6"
                    placeholder="Description complète du produit"
                ></textarea>
            </div>
        </form>

        <!-- Gestion des images -->
        <section class="form-section">
            <h3>Images du produit</h3>
            
            <!-- Zone de drop des images -->
            <div 
                id="image-dropzone" 
                class="image-dropzone"
                ondrop="productEditor.handleImageDrop(event)"
                ondragover="productEditor.handleDragOver(event)"
                ondragleave="productEditor.handleDragLeave(event)"
            >
                <div class="dropzone-content">
                    <i class="icon upload-icon"></i>
                    <p>Glissez des images ici ou</p>
                    <input 
                        type="file" 
                        id="image-input" 
                        multiple 
                        accept="image/*" 
                        style="display: none;"
                        onchange="productEditor.handleImageSelect(event)"
                    >
                    <button 
                        type="button" 
                        class="btn btn-secondary"
                        onclick="document.getElementById('image-input').click()"
                    >
                        Parcourir
                    </button>
                </div>
            </div>

            <!-- Liste des images -->
            <div id="product-images" class="product-images">
                <!-- Les images seront injectées ici -->
            </div>
        </section>

        <!-- Spécifications -->
        <section class="form-section">
            <h3>Spécifications techniques</h3>
            
            <div id="specifications-list">
                <!-- Les spécifications seront injectées ici -->
            </div>

            <button 
                type="button" 
                class="btn btn-secondary"
                onclick="productEditor.addSpecification()"
            >
                Ajouter une spécification
            </button>
        </section>
    </main>

    <!-- Template pour les spécifications -->
    <template id="specification-template">
        <div class="specification-item">
            <div class="form-grid">
                <div class="form-group">
                    <input 
                        type="text" 
                        class="spec-name" 
                        placeholder="Nom de la spécification"
                    >
                </div>
                <div class="form-group">
                    <input 
                        type="text" 
                        class="spec-value" 
                        placeholder="Valeur"
                    >
                </div>
                <div class="form-group">
                    <input 
                        type="text" 
                        class="spec-unit" 
                        placeholder="Unité (optionnel)"
                    >
                </div>
                <div class="form-group">
                    <button 
                        type="button" 
                        class="btn btn-icon delete-spec"
                        onclick="productEditor.removeSpecification(this)"
                    >
                        <i class="icon delete-icon"></i>
                    </button>
                </div>
            </div>
        </div>
    </template>

    <!-- Scripts -->
    <script type="module" src="../../static/js/product-editor.js"></script>
</body>
</html>
```

# Partie 3 : Gestion des Produits
## 3.13.2 JavaScript d'Édition de Produit

```javascript
// frontend/admin/static/js/product-editor.js

import { session } from '../../../static/js/session.js';
import { notifications } from '../../../static/js/notifications.js';

class ProductEditor {
    constructor() {
        this.productId = null;
        this.images = [];
        this.specifications = [];
        this.isNewProduct = true;
        
        this.initialize();
    }

    async initialize() {
        // Vérification si on est en mode édition
        const urlParams = new URLSearchParams(window.location.search);
        this.productId = urlParams.get('id');
        this.isNewProduct = !this.productId;

        // Mise à jour du titre
        document.getElementById('page-title').textContent = 
            this.isNewProduct ? 'Nouveau Produit' : 'Modifier Produit';

        // Chargement des données nécessaires
        await Promise.all([
            this.loadCategories(),
            this.loadBrands()
        ]);

        // Si en mode édition, charger le produit
        if (!this.isNewProduct) {
            await this.loadProduct();
        }

        // Initialisation des écouteurs d'événements
        this.setupEventListeners();
    }

    setupEventListeners() {
        // Gestion du formulaire
        const form = document.getElementById('product-form');
        form.addEventListener('submit', (e) => {
            e.preventDefault();
            this.saveProduct();
        });

        // Validation du SKU
        const skuInput = document.getElementById('sku');
        skuInput.addEventListener('blur', () => this.validateSku(skuInput.value));

        // Validation du prix
        const priceInput = document.getElementById('price');
        priceInput.addEventListener('input', () => {
            priceInput.value = parseFloat(priceInput.value).toFixed(2);
        });
    }

    async loadCategories() {
        try {
            const response = await fetch('/api/products/categories');
            const categories = await response.json();
            
            const select = document.getElementById('category_id');
            categories.forEach(category => {
                const option = document.createElement('option');
                option.value = category.id;
                option.textContent = category.name;
                select.appendChild(option);
            });
        } catch (error) {
            console.error('Erreur chargement catégories:', error);
            notifications.create({
                type: 'error',
                title: 'Erreur',
                message: 'Impossible de charger les catégories'
            });
        }
    }

    async loadBrands() {
        try {
            const response = await fetch('/api/products/brands');
            const brands = await response.json();
            
            const select = document.getElementById('brand_id');
            brands.forEach(brand => {
                const option = document.createElement('option');
                option.value = brand.id;
                option.textContent = brand.name;
                select.appendChild(option);
            });
        } catch (error) {
            console.error('Erreur chargement marques:', error);
            notifications.create({
                type: 'error',
                title: 'Erreur',
                message: 'Impossible de charger les marques'
            });
        }
    }

    async loadProduct() {
        try {
            const response = await fetch(`/api/products/${this.productId}`);
            if (!response.ok) throw new Error('Produit non trouvé');
            
            const product = await response.json();
            
            // Remplissage du formulaire
            this.fillForm(product);
            
            // Chargement des images
            this.loadImages(product.images);
            
            // Chargement des spécifications
            this.loadSpecifications(product.specifications);
            
        } catch (error) {
            console.error('Erreur chargement produit:', error);
            notifications.create({
                type: 'error',
                title: 'Erreur',
                message: 'Impossible de charger le produit'
            });
        }
    }

    fillForm(product) {
        // Remplissage des champs simples
        const fields = [
            'sku', 'name', 'price', 'stock_quantity', 'min_stock_level',
            'short_description', 'description'
        ];
        
        fields.forEach(field => {
            const input = document.getElementById(field);
            if (input) input.value = product[field] || '';
        });

        // Sélection des listes déroulantes
        document.getElementById('category_id').value = product.category_id;
        document.getElementById('brand_id').value = product.brand_id;

        // Case à cocher
        document.getElementById('is_active').checked = product.is_active;
    }

    loadImages(images) {
        this.images = images;
        this.renderImages();
    }

    loadSpecifications(specifications) {
        this.specifications = specifications;
        this.renderSpecifications();
    }

    renderImages() {
        const container = document.getElementById('product-images');
        container.innerHTML = this.images.map((image, index) => `
            <div class="image-item" data-index="${index}">
                <img src="${image.url}" alt="Image produit">
                <div class="image-actions">
                    ${image.is_primary ? `
                        <span class="primary-badge">Principal</span>
                    ` : `
                        <button 
                            type="button" 
                            onclick="productEditor.setPrimaryImage(${index})"
                            title="Définir comme image principale"
                        >
                            <i class="icon star-icon"></i>
                        </button>
                    `}
                    <button 
                        type="button" 
                        onclick="productEditor.removeImage(${index})"
                        title="Supprimer l'image"
                    >
                        <i class="icon delete-icon"></i>
                    </button>
                </div>
            </div>
        `).join('');
    }

    renderSpecifications() {
        const container = document.getElementById('specifications-list');
        container.innerHTML = this.specifications.map((spec, index) => {
            const template = document.getElementById('specification-template');
            const clone = template.content.cloneNode(true);
            
            clone.querySelector('.spec-name').value = spec.name;
            clone.querySelector('.spec-value').value = spec.value;
            clone.querySelector('.spec-unit').value = spec.unit || '';
            clone.querySelector('.specification-item').dataset.index = index;
            
            return clone.children[0].outerHTML;
        }).join('');
    }

    async validateSku(sku) {
        if (!sku) return false;

        try {
            const response = await fetch(`/api/products/validate-sku/${sku}`);
            const data = await response.json();
            
            if (!data.valid) {
                notifications.create({
                    type: 'warning',
                    title: 'SKU invalide',
                    message: 'Cette référence existe déjà'
                });
                return false;
            }
            
            return true;
            
        } catch (error) {
            console.error('Erreur validation SKU:', error);
            return false;
        }
    }

    handleImageDrop(event) {
        event.preventDefault();
        
        const files = event.dataTransfer.files;
        this.uploadImages(files);
        
        event.target.classList.remove('dragover');
    }

    handleDragOver(event) {
        event.preventDefault();
        event.target.classList.add('dragover');
    }

    handleDragLeave(event) {
        event.preventDefault();
        event.target.classList.remove('dragover');
    }

    handleImageSelect(event) {
        const files = event.target.files;
        this.uploadImages(files);
    }

    async uploadImages(files) {
        for (let file of files) {
            if (!file.type.startsWith('image/')) {
                notifications.create({
                    type: 'error',
                    title: 'Type de fichier invalide',
                    message: 'Seules les images sont acceptées'
                });
                continue;
            }

            try {
                const formData = new FormData();
                formData.append('image', file);

                const response = await fetch(`/api/products/${this.productId}/images`, {
                    method: 'POST',
                    headers: {
                        'Authorization': `Bearer ${session.getToken()}`
                    },
                    body: formData
                });

                if (!response.ok) throw new Error('Erreur upload image');

                const data = await response.json();
                this.images.push(data.image);
                this.renderImages();

            } catch (error) {
                console.error('Erreur upload:', error);
                notifications.create({
                    type: 'error',
                    title: 'Erreur',
                    message: 'Impossible d\'uploader l\'image'
                });
            }
        }
    }

    setPrimaryImage(index) {
        this.images.forEach((img, i) => {
            img.is_primary = (i === index);
        });
        this.renderImages();
    }

    removeImage(index) {
        this.images.splice(index, 1);
        this.renderImages();
    }

    addSpecification() {
        const template = document.getElementById('specification-template');
        const container = document.getElementById('specifications-list');
        container.appendChild(template.content.cloneNode(true));
    }

    removeSpecification(button) {
        const item = button.closest('.specification-item');
        item.remove();
    }

    getFormData() {
        const form = document.getElementById('product-form');
        const formData = new FormData(form);
        
        // Conversion en objet
        const data = {
            specifications: this.getSpecifications(),
            images: this.images,
            is_active: formData.get('is_active') === 'on'
        };
        
        // Ajout des autres champs
        for (let [key, value] of formData.entries()) {
            if (key !== 'is_active') {
                data[key] = value;
            }
        }
        
        return data;
    }

    getSpecifications() {
        const specs = [];
        const items = document.querySelectorAll('.specification-item');
        
        items.forEach((item, index) => {
            const name = item.querySelector('.spec-name').value;
            const value = item.querySelector('.spec-value').value;
            
            if (name && value) {
                specs.push({
                    name,
                    value,
                    unit: item.querySelector('.spec-unit').value,
                    position: index
                });
            }
        });
        
        return specs;
    }

    async saveProduct() {
        try {
            const data = this.getFormData();
            
            // Validation
            if (!this.validateForm(data)) return;
            
            const url = this.isNewProduct ? 
                '/api/products' : 
                `/api/products/${this.productId}`;
                
            const method = this.isNewProduct ? 'POST' : 'PUT';
            
            const response = await fetch(url, {
                method,
                headers: {
                    'Content-Type': 'application/json',
                    'Authorization': `Bearer ${session.getToken()}`
                },
                body: JSON.stringify(data)
            });

            if (!response.ok) throw new Error('Erreur sauvegarde produit');

            notifications.create({
                type: 'success',
                title: 'Succès',
                message: `Produit ${this.isNewProduct ? 'créé' : 'modifié'} avec succès`
            });

            // Redirection vers la liste
            setTimeout(() => {
                window.location.href = './index.html';
            }, 1500);

        } catch (error) {
            console.error('Erreur sauvegarde:', error);
            notifications.create({
                type: 'error',
                title: 'Erreur',
                message: 'Impossible de sauvegarder le produit'
            });
        }
    }

    validateForm(data) {
        // Validation des champs requis
        const requiredFields = ['sku', 'name', 'price', 'category_id', 'brand_id'];
        for (let field of requiredFields) {
            if (!data[field]) {
                notifications.create({
                    type: 'error',
                    title: 'Champs manquants',
                    message: 'Veuillez remplir tous les champs obligatoires'
                });
                return false;
            }
        }

        // Validation du prix
        if (data.price <= 0) {
            notifications.create({
                type: 'error',
                title: 'Prix invalide',
                message: 'Le prix doit être supérieur à 0'
            });
            return false;
        }

        // Validation des stocks
        if (data.stock_quantity < 0) {
            notifications.create({
                type: 'error',
                title: 'Stock invalide',
                message: 'La quantité en stock ne peut pas être négative'
            });
            return false;
        }

        return true;
    }
}

// Initialisation
const productEditor = new ProductEditor();
// Rendre l'instance accessible globalement
window.productEditor = productEditor;
```

# Partie 3 : Gestion des Produits
## 3.13.3 Styles des Formulaires

### Description
Styles spécifiques pour :
- Les formulaires d'édition
- La zone de drop d'images
- Les spécifications techniques
- Les messages de validation

```css
/* frontend/admin/static/css/forms.css */

/* Structure des formulaires */
.form-section {
    background: white;
    padding: 2rem;
    border-radius: 8px;
    box-shadow: 0 2px 4px rgba(0, 0, 0, 0.05);
    margin-bottom: 2rem;
}

.form-section h3 {
    color: #2c3e50;
    margin-bottom: 1.5rem;
    font-size: 1.2rem;
}

.form-grid {
    display: grid;
    grid-template-columns: repeat(2, 1fr);
    gap: 1.5rem;
}

@media (max-width: 768px) {
    .form-grid {
        grid-template-columns: 1fr;
    }
}

/* Champs de formulaire */
.form-group {
    margin-bottom: 1.5rem;
}

.form-group label {
    display: block;
    margin-bottom: 0.5rem;
    color: #2c3e50;
    font-weight: 500;
}

.form-group input[type="text"],
.form-group input[type="number"],
.form-group input[type="email"],
.form-group input[type="password"],
.form-group textarea,
.form-group select {
    width: 100%;
    padding: 0.8rem;
    border: 1px solid #ddd;
    border-radius: 4px;
    font-size: 1rem;
    transition: border-color 0.3s, box-shadow 0.3s;
}

.form-group input:focus,
.form-group textarea:focus,
.form-group select:focus {
    border-color: #3498db;
    box-shadow: 0 0 0 2px rgba(52, 152, 219, 0.2);
    outline: none;
}

.form-group textarea {
    min-height: 150px;
    resize: vertical;
}

.form-hint {
    display: block;
    margin-top: 0.5rem;
    font-size: 0.875rem;
    color: #666;
}

/* Validation des champs */
.form-group.error input,
.form-group.error textarea,
.form-group.error select {
    border-color: #e74c3c;
}

.form-group.error .form-hint {
    color: #e74c3c;
}

.form-group.success input,
.form-group.success textarea,
.form-group.success select {
    border-color: #2ecc71;
}

/* Cases à cocher et boutons radio */
.checkbox-label,
.radio-label {
    display: flex;
    align-items: center;
    gap: 0.5rem;
    cursor: pointer;
}

.checkbox-label input[type="checkbox"],
.radio-label input[type="radio"] {
    width: 16px;
    height: 16px;
}

/* Zone de drop d'images */
.image-dropzone {
    border: 2px dashed #ddd;
    border-radius: 8px;
    padding: 2rem;
    text-align: center;
    transition: all 0.3s;
    margin-bottom: 1.5rem;
}

.image-dropzone.dragover {
    border-color: #3498db;
    background: rgba(52, 152, 219, 0.05);
}

.dropzone-content {
    display: flex;
    flex-direction: column;
    align-items: center;
    gap: 1rem;
}

.dropzone-content .upload-icon {
    font-size: 2rem;
    color: #666;
}

/* Galerie d'images */
.product-images {
    display: grid;
    grid-template-columns: repeat(auto-fill, minmax(150px, 1fr));
    gap: 1rem;
    margin-top: 1.5rem;
}

.image-item {
    position: relative;
    border-radius: 4px;
    overflow: hidden;
}

.image-item img {
    width: 100%;
    height: 150px;
    object-fit: cover;
}

.image-actions {
    position: absolute;
    top: 0;
    right: 0;
    padding: 0.5rem;
    background: rgba(0, 0, 0, 0.5);
    display: flex;
    gap: 0.5rem;
}

.image-actions button {
    background: none;
    border: none;
    color: white;
    cursor: pointer;
    padding: 0.25rem;
    border-radius: 4px;
    transition: background 0.3s;
}

.image-actions button:hover {
    background: rgba(255, 255, 255, 0.1);
}

.primary-badge {
    background: #3498db;
    color: white;
    padding: 0.25rem 0.5rem;
    border-radius: 4px;
    font-size: 0.875rem;
}

/* Spécifications */
.specification-item {
    background: #f8f9fa;
    border-radius: 4px;
    padding: 1rem;
    margin-bottom: 1rem;
}

.specification-item .form-grid {
    grid-template-columns: 2fr 2fr 1fr auto;
    align-items: start;
}

.specification-item .form-group {
    margin-bottom: 0;
}

.delete-spec {
    color: #e74c3c;
}

/* Boutons d'action */
.form-actions {
    display: flex;
    justify-content: flex-end;
    gap: 1rem;
    margin-top: 2rem;
}

/* Messages de validation */
.validation-message {
    padding: 0.5rem;
    border-radius: 4px;
    font-size: 0.875rem;
    margin-top: 0.5rem;
}

.validation-message.error {
    background: #fff5f5;
    color: #e74c3c;
    border: 1px solid #fcd2d2;
}

.validation-message.success {
    background: #f0fff4;
    color: #2ecc71;
    border: 1px solid #c6f6d5;
}

/* Champs requis */
.required-field::after {
    content: '*';
    color: #e74c3c;
    margin-left: 0.25rem;
}

/* États désactivés */
.form-group input:disabled,
.form-group textarea:disabled,
.form-group select:disabled {
    background: #f8f9fa;
    cursor: not-allowed;
}

/* Responsive */
@media (max-width: 768px) {
    .form-section {
        padding: 1rem;
    }

    .specification-item .form-grid {
        grid-template-columns: 1fr;
        gap: 1rem;
    }

    .form-actions {
        flex-direction: column;
    }

    .form-actions button {
        width: 100%;
    }
}

/* Animation de chargement */
.loading-input {
    position: relative;
    pointer-events: none;
}

.loading-input::after {
    content: '';
    position: absolute;
    top: 50%;
    right: 0.8rem;
    transform: translateY(-50%);
    width: 16px;
    height: 16px;
    border: 2px solid #ddd;
    border-top-color: #3498db;
    border-radius: 50%;
    animation: spin 1s linear infinite;
}

@keyframes spin {
    to {
        transform: translateY(-50%) rotate(360deg);
    }
}
```

## Prochaines étapes

# Partie 3 : Gestion des Produits
## 3.14 Gestion des Catégories et Marques

### Description
Cette section comprend :
- Les pages de gestion des catégories
- Les pages de gestion des marques
- Les formulaires d'édition
- La logique de gestion

### 3.14.1 Page de Gestion des Catégories
```html
<!-- frontend/admin/pages/categories/index.html -->
<!DOCTYPE html>
<html lang="fr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>FMP Admin - Gestion des Catégories</title>
    <link rel="stylesheet" href="../../static/css/admin.css">
    <link rel="stylesheet" href="../../static/css/forms.css">
    
    <!-- Vérification admin -->
    <script type="module">
        import { session } from '../../../static/js/session.js';
        const userData = session.getUserData();
        if (!userData?.is_admin) {
            window.location.href = '/login.html';
        }
    </script>
</head>
<body>
    <!-- Menu latéral admin -->
    <aside class="admin-sidebar">
        <!-- ... Menu identique aux autres pages ... -->
    </aside>

    <!-- Contenu principal -->
    <main class="admin-main">
        <!-- En-tête -->
        <header class="admin-header">
            <div class="header-content">
                <h2>Gestion des Catégories</h2>
                <button 
                    class="btn btn-primary"
                    onclick="categoriesManager.showCreateModal()"
                >
                    Nouvelle Catégorie
                </button>
            </div>
        </header>

        <!-- Liste des catégories -->
        <section class="categories-section">
            <div class="categories-tree" id="categories-tree">
                <!-- Arborescence des catégories injectée ici -->
            </div>
        </section>

        <!-- Modal de création/édition -->
        <div class="modal" id="category-modal">
            <div class="modal-content">
                <div class="modal-header">
                    <h3 id="modal-title">Nouvelle Catégorie</h3>
                    <button 
                        class="close-btn"
                        onclick="categoriesManager.hideModal()"
                    >&times;</button>
                </div>
                
                <form id="category-form" class="form-section" onsubmit="return false;">
                    <div class="form-group">
                        <label for="name">Nom de la catégorie*</label>
                        <input 
                            type="text" 
                            id="name" 
                            name="name" 
                            required
                        >
                    </div>

                    <div class="form-group">
                        <label for="description">Description</label>
                        <textarea 
                            id="description" 
                            name="description"
                            rows="3"
                        ></textarea>
                    </div>

                    <div class="form-group">
                        <label for="parent_id">Catégorie parente</label>
                        <select id="parent_id" name="parent_id">
                            <option value="">Aucune (catégorie principale)</option>
                        </select>
                    </div>

                    <div class="form-group">
                        <label for="image">Image de la catégorie</label>
                        <input 
                            type="file" 
                            id="image" 
                            name="image"
                            accept="image/*"
                        >
                        <div id="current-image" class="current-image" style="display: none;">
                            <img src="" alt="Image actuelle">
                            <button 
                                type="button" 
                                class="btn btn-danger"
                                onclick="categoriesManager.removeImage()"
                            >
                                Supprimer
                            </button>
                        </div>
                    </div>

                    <div class="form-actions">
                        <button 
                            type="button" 
                            class="btn btn-secondary"
                            onclick="categoriesManager.hideModal()"
                        >
                            Annuler
                        </button>
                        <button 
                            type="submit" 
                            class="btn btn-primary"
                            onclick="categoriesManager.saveCategory()"
                        >
                            Enregistrer
                        </button>
                    </div>
                </form>
            </div>
        </div>
    </main>

    <!-- Templates -->
    <template id="category-item-template">
        <div class="category-item">
            <div class="category-content">
                <button class="expand-btn">
                    <i class="icon chevron-icon"></i>
                </button>
                <span class="category-name"></span>
                <div class="category-actions">
                    <button class="btn btn-icon edit-btn">
                        <i class="icon edit-icon"></i>
                    </button>
                    <button class="btn btn-icon delete-btn">
                        <i class="icon delete-icon"></i>
                    </button>
                </div>
            </div>
            <div class="subcategories"></div>
        </div>
    </template>

    <!-- Scripts -->
    <script type="module" src="../../static/js/categories.js"></script>
</body>
</html>
```

### 3.14.2 JavaScript de Gestion des Catégories
```javascript
// frontend/admin/static/js/categories.js

import { session } from '../../../static/js/session.js';
import { notifications } from '../../../static/js/notifications.js';

class CategoriesManager {
    constructor() {
        this.categories = [];
        this.currentCategory = null;
        this.initialize();
    }

    async initialize() {
        await this.loadCategories();
        this.renderCategoriesTree();
        this.setupEventListeners();
    }

    async loadCategories() {
        try {
            const response = await fetch('/api/products/categories');
            const data = await response.json();
            this.categories = this.organizeCategories(data);
        } catch (error) {
            console.error('Erreur chargement catégories:', error);
            notifications.create({
                type: 'error',
                title: 'Erreur',
                message: 'Impossible de charger les catégories'
            });
        }
    }

    organizeCategories(categories) {
        // Création de la structure arborescente
        const categoryMap = {};
        const rootCategories = [];

        // Première passe : création du mapping
        categories.forEach(category => {
            categoryMap[category.id] = {
                ...category,
                children: []
            };
        });

        // Deuxième passe : organisation de l'arbre
        categories.forEach(category => {
            if (category.parent_id) {
                categoryMap[category.parent_id].children.push(categoryMap[category.id]);
            } else {
                rootCategories.push(categoryMap[category.id]);
            }
        });

        return rootCategories;
    }

    renderCategoriesTree() {
        const container = document.getElementById('categories-tree');
        container.innerHTML = this.buildCategoryHTML(this.categories);
        
        // Mise à jour du select des parents
        this.updateParentSelect();
    }

    buildCategoryHTML(categories, level = 0) {
        return categories.map(category => {
            const template = document.getElementById('category-item-template');
            const clone = template.content.cloneNode(true);
            
            // Configuration de l'élément
            const item = clone.querySelector('.category-item');
            item.dataset.id = category.id;
            item.style.marginLeft = `${level * 20}px`;
            
            // Nom de la catégorie
            item.querySelector('.category-name').textContent = category.name;
            
            // Bouton d'expansion
            const expandBtn = item.querySelector('.expand-btn');
            if (category.children.length === 0) {
                expandBtn.style.visibility = 'hidden';
            } else {
                expandBtn.addEventListener('click', () => {
                    item.classList.toggle('expanded');
                });
            }
            
            // Sous-catégories
            if (category.children.length > 0) {
                item.querySelector('.subcategories').innerHTML = 
                    this.buildCategoryHTML(category.children, level + 1);
            }
            
            // Actions
            item.querySelector('.edit-btn').addEventListener('click', () => {
                this.editCategory(category);
            });
            
            item.querySelector('.delete-btn').addEventListener('click', () => {
                this.deleteCategory(category);
            });
            
            return item.outerHTML;
            
        }).join('');
    }

    updateParentSelect() {
        const select = document.getElementById('parent_id');
        select.innerHTML = '<option value="">Aucune (catégorie principale)</option>';
        
        const addOptions = (categories, level = 0) => {
            categories.forEach(category => {
                // Ne pas ajouter la catégorie en cours d'édition
                if (this.currentCategory && category.id === this.currentCategory.id) {
                    return;
                }
                
                const option = document.createElement('option');
                option.value = category.id;
                option.textContent = '  '.repeat(level) + category.name;
                select.appendChild(option);
                
                if (category.children.length > 0) {
                    addOptions(category.children, level + 1);
                }
            });
        };
        
        addOptions(this.categories);
    }

    showCreateModal() {
        this.currentCategory = null;
        document.getElementById('modal-title').textContent = 'Nouvelle Catégorie';
        document.getElementById('category-form').reset();
        document.getElementById('current-image').style.display = 'none';
        document.getElementById('category-modal').classList.add('show');
    }

    editCategory(category) {
        this.currentCategory = category;
        document.getElementById('modal-title').textContent = 'Modifier la Catégorie';
        
        // Remplissage du formulaire
        document.getElementById('name').value = category.name;
        document.getElementById('description').value = category.description || '';
        document.getElementById('parent_id').value = category.parent_id || '';
        
        // Affichage de l'image actuelle
        if (category.image_url) {
            const imgContainer = document.getElementById('current-image');
            imgContainer.style.display = 'block';
            imgContainer.querySelector('img').src = category.image_url;
        }
        
        document.getElementById('category-modal').classList.add('show');
    }

    async deleteCategory(category) {
        if (!confirm(`Êtes-vous sûr de vouloir supprimer la catégorie "${category.name}" ?`)) {
            return;
        }

        try {
            const response = await fetch(`/api/products/categories/${category.id}`, {
                method: 'DELETE',
                headers: {
                    'Authorization': `Bearer ${session.getToken()}`
                }
            });

            if (!response.ok) throw new Error('Erreur suppression catégorie');

            notifications.create({
                type: 'success',
                title: 'Succès',
                message: 'Catégorie supprimée avec succès'
            });

            await this.loadCategories();
            this.renderCategoriesTree();

        } catch (error) {
            console.error('Erreur suppression:', error);
            notifications.create({
                type: 'error',
                title: 'Erreur',
                message: 'Impossible de supprimer la catégorie'
            });
        }
    }

    async saveCategory() {
        try {
            const form = document.getElementById('category-form');
            const formData = new FormData(form);
            
            // Ajout de l'ID si en mode édition
            if (this.currentCategory) {
                formData.append('id', this.currentCategory.id);
            }
            
            const response = await fetch(
                this.currentCategory ? 
                    `/api/products/categories/${this.currentCategory.id}` : 
                    '/api/products/categories',
                {
                    method: this.currentCategory ? 'PUT' : 'POST',
                    headers: {
                        'Authorization': `Bearer ${session.getToken()}`
                    },
                    body: formData
                }
            );

            if (!response.ok) throw new Error('Erreur sauvegarde catégorie');

            notifications.create({
                type: 'success',
                title: 'Succès',
                message: `Catégorie ${this.currentCategory ? 'modifiée' : 'créée'} avec succès`
            });

            this.hideModal();
            await this.loadCategories();
            this.renderCategoriesTree();

        } catch (error) {
            console.error('Erreur sauvegarde:', error);
            notifications.create({
                type: 'error',
                title: 'Erreur',
                message: 'Impossible de sauvegarder la catégorie'
            });
        }
    }

    removeImage() {
        if (this.currentCategory) {
            document.getElementById('current-image').style.display = 'none';
            // L'image sera supprimée lors de la sauvegarde
        }
    }

    hideModal() {
        document.getElementById('category-modal').classList.remove('show');
        this.currentCategory = null;
    }

    setupEventListeners() {
        // Fermeture du modal sur clic en dehors
        document.getElementById('category-modal').addEventListener('click', (e) => {
            if (e.target.id === 'category-modal') {
                this.hideModal();
            }
        });
    }
}

// Initialisation
const categoriesManager = new CategoriesManager();
// Rendre l'instance accessible globalement
window.categoriesManager = categoriesManager;
```

# Partie 3 : Gestion des Produits
## 3.14.3 Gestion des Marques

### Description
Cette section comprend :
- La page de liste des marques
- Le formulaire d'édition des marques
- La gestion des logos
- Les actions CRUD

### 3.14.4 Page de Gestion des Marques
```html
<!-- frontend/admin/pages/brands/index.html -->
<!DOCTYPE html>
<html lang="fr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>FMP Admin - Gestion des Marques</title>
    <link rel="stylesheet" href="../../static/css/admin.css">
    <link rel="stylesheet" href="../../static/css/forms.css">
    <link rel="stylesheet" href="../../static/css/brands.css">
    
    <!-- Vérification admin -->
    <script type="module">
        import { session } from '../../../static/js/session.js';
        const userData = session.getUserData();
        if (!userData?.is_admin) {
            window.location.href = '/login.html';
        }
    </script>
</head>
<body>
    <!-- Menu latéral admin -->
    <aside class="admin-sidebar">
        <!-- ... Menu identique aux autres pages ... -->
    </aside>

    <!-- Contenu principal -->
    <main class="admin-main">
        <!-- En-tête -->
        <header class="admin-header">
            <div class="header-content">
                <h2>Gestion des Marques</h2>
                <button 
                    class="btn btn-primary"
                    onclick="brandsManager.showCreateModal()"
                >
                    Nouvelle Marque
                </button>
            </div>
        </header>

        <!-- Liste des marques -->
        <section class="brands-grid" id="brands-grid">
            <!-- Les marques seront injectées ici -->
        </section>

        <!-- Modal de création/édition -->
        <div class="modal" id="brand-modal">
            <div class="modal-content">
                <div class="modal-header">
                    <h3 id="modal-title">Nouvelle Marque</h3>
                    <button 
                        class="close-btn"
                        onclick="brandsManager.hideModal()"
                    >&times;</button>
                </div>
                
                <form id="brand-form" class="form-section" onsubmit="return false;">
                    <div class="form-group">
                        <label for="name">Nom de la marque*</label>
                        <input 
                            type="text" 
                            id="name" 
                            name="name" 
                            required
                        >
                    </div>

                    <div class="form-group">
                        <label for="description">Description</label>
                        <textarea 
                            id="description" 
                            name="description"
                            rows="3"
                        ></textarea>
                    </div>

                    <div class="form-group">
                        <label for="website">Site web</label>
                        <input 
                            type="url" 
                            id="website" 
                            name="website"
                            placeholder="https://"
                        >
                    </div>

                    <div class="form-group">
                        <label for="logo">Logo de la marque</label>
                        <input 
                            type="file" 
                            id="logo" 
                            name="logo"
                            accept="image/*"
                        >
                        <div id="current-logo" class="current-logo" style="display: none;">
                            <img src="" alt="Logo actuel">
                            <button 
                                type="button" 
                                class="btn btn-danger"
                                onclick="brandsManager.removeLogo()"
                            >
                                Supprimer
                            </button>
                        </div>
                    </div>

                    <div class="form-actions">
                        <button 
                            type="button" 
                            class="btn btn-secondary"
                            onclick="brandsManager.hideModal()"
                        >
                            Annuler
                        </button>
                        <button 
                            type="submit" 
                            class="btn btn-primary"
                            onclick="brandsManager.saveBrand()"
                        >
                            Enregistrer
                        </button>
                    </div>
                </form>
            </div>
        </div>
    </main>

    <!-- Template de carte marque -->
    <template id="brand-card-template">
        <div class="brand-card">
            <div class="brand-logo">
                <img src="" alt="">
            </div>
            <div class="brand-info">
                <h3 class="brand-name"></h3>
                <p class="brand-description"></p>
                <a href="#" class="brand-website" target="_blank">
                    <i class="icon link-icon"></i>
                    Visiter le site
                </a>
            </div>
            <div class="brand-actions">
                <button class="btn btn-icon edit-btn" title="Modifier">
                    <i class="icon edit-icon"></i>
                </button>
                <button class="btn btn-icon delete-btn" title="Supprimer">
                    <i class="icon delete-icon"></i>
                </button>
            </div>
        </div>
    </template>

    <!-- Scripts -->
    <script type="module" src="../../static/js/brands.js"></script>
</body>
</html>
```

### 3.14.5 JavaScript de Gestion des Marques
```javascript
// frontend/admin/static/js/brands.js

import { session } from '../../../static/js/session.js';
import { notifications } from '../../../static/js/notifications.js';

class BrandsManager {
    constructor() {
        this.brands = [];
        this.currentBrand = null;
        this.initialize();
    }

    async initialize() {
        await this.loadBrands();
        this.renderBrands();
        this.setupEventListeners();
    }

    async loadBrands() {
        try {
            const response = await fetch('/api/products/brands');
            this.brands = await response.json();
        } catch (error) {
            console.error('Erreur chargement marques:', error);
            notifications.create({
                type: 'error',
                title: 'Erreur',
                message: 'Impossible de charger les marques'
            });
        }
    }

    renderBrands() {
        const container = document.getElementById('brands-grid');
        container.innerHTML = this.brands.map(brand => {
            const template = document.getElementById('brand-card-template');
            const clone = template.content.cloneNode(true);
            
            // Configuration de la carte
            const card = clone.querySelector('.brand-card');
            card.dataset.id = brand.id;
            
            // Logo
            const logo = card.querySelector('.brand-logo img');
            logo.src = brand.logo_url || '/static/images/no-logo.png';
            logo.alt = `Logo ${brand.name}`;
            
            // Informations
            card.querySelector('.brand-name').textContent = brand.name;
            card.querySelector('.brand-description').textContent = 
                brand.description || 'Aucune description';
            
            // Site web
            const websiteLink = card.querySelector('.brand-website');
            if (brand.website) {
                websiteLink.href = brand.website;
            } else {
                websiteLink.style.display = 'none';
            }
            
            // Actions
            card.querySelector('.edit-btn').addEventListener('click', () => {
                this.editBrand(brand);
            });
            
            card.querySelector('.delete-btn').addEventListener('click', () => {
                this.deleteBrand(brand);
            });
            
            return clone;
            
        }).join('');
    }

    showCreateModal() {
        this.currentBrand = null;
        document.getElementById('modal-title').textContent = 'Nouvelle Marque';
        document.getElementById('brand-form').reset();
        document.getElementById('current-logo').style.display = 'none';
        document.getElementById('brand-modal').classList.add('show');
    }

    editBrand(brand) {
        this.currentBrand = brand;
        document.getElementById('modal-title').textContent = 'Modifier la Marque';
        
        // Remplissage du formulaire
        document.getElementById('name').value = brand.name;
        document.getElementById('description').value = brand.description || '';
        document.getElementById('website').value = brand.website || '';
        
        // Affichage du logo actuel
        if (brand.logo_url) {
            const logoContainer = document.getElementById('current-logo');
            logoContainer.style.display = 'block';
            logoContainer.querySelector('img').src = brand.logo_url;
        }
        
        document.getElementById('brand-modal').classList.add('show');
    }

    async deleteBrand(brand) {
        if (!confirm(`Êtes-vous sûr de vouloir supprimer la marque "${brand.name}" ?`)) {
            return;
        }

        try {
            const response = await fetch(`/api/products/brands/${brand.id}`, {
                method: 'DELETE',
                headers: {
                    'Authorization': `Bearer ${session.getToken()}`
                }
            });

            if (!response.ok) throw new Error('Erreur suppression marque');

            notifications.create({
                type: 'success',
                title: 'Succès',
                message: 'Marque supprimée avec succès'
            });

            await this.loadBrands();
            this.renderBrands();

        } catch (error) {
            console.error('Erreur suppression:', error);
            notifications.create({
                type: 'error',
                title: 'Erreur',
                message: 'Impossible de supprimer la marque'
            });
        }
    }

    async saveBrand() {
        try {
            const form = document.getElementById('brand-form');
            const formData = new FormData(form);
            
            // Ajout de l'ID si en mode édition
            if (this.currentBrand) {
                formData.append('id', this.currentBrand.id);
            }
            
            const response = await fetch(
                this.currentBrand ? 
                    `/api/products/brands/${this.currentBrand.id}` : 
                    '/api/products/brands',
                {
                    method: this.currentBrand ? 'PUT' : 'POST',
                    headers: {
                        'Authorization': `Bearer ${session.getToken()}`
                    },
                    body: formData
                }
            );

            if (!response.ok) throw new Error('Erreur sauvegarde marque');

            notifications.create({
                type: 'success',
                title: 'Succès',
                message: `Marque ${this.currentBrand ? 'modifiée' : 'créée'} avec succès`
            });

            this.hideModal();
            await this.loadBrands();
            this.renderBrands();

        } catch (error) {
            console.error('Erreur sauvegarde:', error);
            notifications.create({
                type: 'error',
                title: 'Erreur',
                message: 'Impossible de sauvegarder la marque'
            });
        }
    }

    removeLogo() {
        if (this.currentBrand) {
            document.getElementById('current-logo').style.display = 'none';
            // Le logo sera supprimé lors de la sauvegarde
        }
    }

    hideModal() {
        document.getElementById('brand-modal').classList.remove('show');
        this.currentBrand = null;
    }

    setupEventListeners() {
        // Fermeture du modal sur clic en dehors
        document.getElementById('brand-modal').addEventListener('click', (e) => {
            if (e.target.id === 'brand-modal') {
                this.hideModal();
            }
        });
    }
}

// Initialisation
const brandsManager = new BrandsManager();
// Rendre l'instance accessible globalement
window.brandsManager = brandsManager;
```

### 3.14.6 Styles Spécifiques pour les Marques
```css
/* frontend/admin/static/css/brands.css */

/* Grille des marques */
.brands-grid {
    display: grid;
    grid-template-columns: repeat(auto-fill, minmax(300px, 1fr));
    gap: 1.5rem;
    padding: 1.5rem;
}

/* Carte de marque */
.brand-card {
    background: white;
    border-radius: 8px;
    box-shadow: 0 2px 4px rgba(0, 0, 0, 0.05);
    overflow: hidden;
    transition: transform 0.3s, box-shadow 0.3s;
}

.brand-card:hover {
    transform: translateY(-5px);
    box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
}

/* Logo */
.brand-logo {
    height: 160px;
    background: #f8f9fa;
    display: flex;
    align-items: center;
    justify-content: center;
    padding: 1rem;
}

.brand-logo img {
    max-width: 100%;
    max-height: 100%;
    object-fit: contain;
}

/* Informations */
.brand-info {
    padding: 1.5rem;
}

.brand-name {
    font-size: 1.2rem;
    color: #2c3e50;
    margin-bottom: 0.5rem;
}

.brand-description {
    color: #666;
    font-size: 0.9rem;
    margin-bottom: 1rem;
    line-height: 1.4;
}

.brand-website {
    display: inline-flex;
    align-items: center;
    gap: 0.5rem;
    color: #3498db;
    text-decoration: none;
    font-size: 0.9rem;
}

.brand-website:hover {
    text-decoration: underline;
}

/* Actions */
.brand-actions {
    padding: 1rem;
    border-top: 1px solid #eee;
    display: flex;
    justify-content: flex-end;
    gap: 0.5rem;
}

/* Logo dans le formulaire */
.current-logo {
    margin-top: 1rem;
    padding: 1rem;
    background: #f8f9fa;
    border-radius: 4px;
    display: flex;
    align-items: center;
    gap: 1rem;
}

.current-logo img {
    max-height: 100px;
    width: auto;
}

/* Responsive */
@media (max-width: 768px) {
    .brands-grid {
        grid-template-columns: 1fr;
        padding: 1rem;
    }

    .brand-card {
        max-width: 100%;
    }

    .brand-logo {
        height: 120px;
    }

    .brand-info {
        padding: 1rem;
    }

    .brand-actions {
        padding: 0.8rem;
    }

    .current-logo {
        flex-direction: column;
        align-items: flex-start;
    }

    .current-logo img {
        max-width: 100%;
        height: auto;
    }
}

/* Animations */
.brand-card {
    animation: fadeIn 0.3s ease-out;
}

@keyframes fadeIn {
    from {
        opacity: 0;
        transform: translateY(10px);
    }
    to {
        opacity: 1;
        transform: translateY(0);
    }
}

# Partie 3 : Catalogue Frontend
## 3.15 Page d'Accueil

### Description
La page d'accueil comprend :
- Un en-tête avec navigation
- Un slider de bannières
- Les catégories principales
- Les produits populaires
- Les marques partenaires
- Les avantages du site

### 3.15.1 Page d'Accueil
```html
<!-- frontend/index.html -->
<!DOCTYPE html>
<html lang="fr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>FMP - For My Printer | Cartouches d'imprimantes</title>
    <meta name="description" content="FMP, votre spécialiste en cartouches d'imprimantes. Large gamme de cartouches compatibles et originales pour toutes les marques.">
    
    <!-- Styles -->
    <link rel="stylesheet" href="static/css/main.css">
    <link rel="stylesheet" href="static/css/home.css">
</head>
<body>
    <!-- En-tête -->
    <header class="main-header">
        <!-- Top Bar -->
        <div class="top-bar">
            <div class="container">
                <div class="contact-info">
                    <a href="tel:+33123456789">
                        <i class="icon phone-icon"></i>
                        01 23 45 67 89
                    </a>
                    <a href="mailto:contact@fmp.com">
                        <i class="icon mail-icon"></i>
                        contact@fmp.com
                    </a>
                </div>
                <div class="user-nav">
                    <a href="account/login.html" id="login-link">
                        <i class="icon user-icon"></i>
                        Connexion
                    </a>
                    <a href="account/register.html" id="register-link">
                        Inscription
                    </a>
                </div>
            </div>
        </div>

        <!-- Navigation Principale -->
        <nav class="main-nav">
            <div class="container">
                <div class="logo">
                    <a href="index.html">
                        <img src="static/images/logo.png" alt="FMP Logo">
                    </a>
                </div>

                <div class="search-bar">
                    <form action="search.html" method="get">
                        <input 
                            type="text" 
                            name="q" 
                            placeholder="Rechercher une cartouche..."
                            id="search-input"
                        >
                        <button type="submit">
                            <i class="icon search-icon"></i>
                        </button>
                    </form>
                </div>

                <div class="cart-widget">
                    <a href="cart.html" class="cart-link">
                        <i class="icon cart-icon"></i>
                        <span class="cart-count" id="cart-count">0</span>
                        <span class="cart-total" id="cart-total">0,00 €</span>
                    </a>
                </div>
            </div>
        </nav>

        <!-- Menu Catégories -->
        <div class="category-nav">
            <div class="container">
                <ul id="category-menu">
                    <!-- Les catégories seront injectées ici -->
                </ul>
            </div>
        </div>
    </header>

    <!-- Contenu Principal -->
    <main>
        <!-- Bannière Principale -->
        <section class="hero-slider">
            <div class="slider-container" id="main-slider">
                <div class="slide active">
                    <img src="static/images/banners/banner1.jpg" alt="Offre spéciale cartouches">
                    <div class="slide-content">
                        <h2>Offres exceptionnelles</h2>
                        <p>Jusqu'à -50% sur les cartouches compatibles</p>
                        <a href="products.html" class="cta-button">Voir les offres</a>
                    </div>
                </div>
                <!-- Autres slides -->
            </div>
            <div class="slider-controls">
                <button class="prev-btn">
                    <i class="icon chevron-left-icon"></i>
                </button>
                <button class="next-btn">
                    <i class="icon chevron-right-icon"></i>
                </button>
            </div>
        </section>

        <!-- Avantages -->
        <section class="features">
            <div class="container">
                <div class="feature-grid">
                    <div class="feature-item">
                        <i class="icon truck-icon"></i>
                        <h3>Livraison Gratuite</h3>
                        <p>Dès 49€ d'achat</p>
                    </div>
                    <div class="feature-item">
                        <i class="icon shield-icon"></i>
                        <h3>Garantie 2 Ans</h3>
                        <p>Sur tous nos produits</p>
                    </div>
                    <div class="feature-item">
                        <i class="icon return-icon"></i>
                        <h3>Retour Gratuit</h3>
                        <p>Sous 30 jours</p>
                    </div>
                    <div class="feature-item">
                        <i class="icon support-icon"></i>
                        <h3>Support Client</h3>
                        <p>5j/7 de 9h à 18h</p>
                    </div>
                </div>
            </div>
        </section>

        <!-- Catégories Populaires -->
        <section class="popular-categories">
            <div class="container">
                <h2 class="section-title">Nos Catégories</h2>
                <div class="category-grid" id="featured-categories">
                    <!-- Les catégories seront injectées ici -->
                </div>
            </div>
        </section>

        <!-- Produits en Vedette -->
        <section class="featured-products">
            <div class="container">
                <h2 class="section-title">Produits Populaires</h2>
                <div class="products-grid" id="featured-products">
                    <!-- Les produits seront injectés ici -->
                </div>
            </div>
        </section>

        <!-- Marques Partenaires -->
        <section class="brands">
            <div class="container">
                <h2 class="section-title">Nos Marques Partenaires</h2>
                <div class="brands-slider" id="brands-slider">
                    <!-- Les marques seront injectées ici -->
                </div>
            </div>
        </section>

        <!-- Newsletter -->
        <section class="newsletter">
            <div class="container">
                <div class="newsletter-content">
                    <h2>Restez informé !</h2>
                    <p>Inscrivez-vous à notre newsletter pour recevoir nos meilleures offres</p>
                    <form id="newsletter-form" onsubmit="return false;">
                        <input 
                            type="email" 
                            placeholder="Votre adresse email"
                            required
                        >
                        <button type="submit" class="cta-button">
                            S'inscrire
                        </button>
                    </form>
                </div>
            </div>
        </section>
    </main>

    <!-- Pied de page -->
    <footer class="main-footer">
        <div class="container">
            <div class="footer-grid">
                <!-- Informations -->
                <div class="footer-section">
                    <h3>À Propos de FMP</h3>
                    <p>Votre spécialiste en cartouches d'imprimantes depuis 2024.</p>
                    <div class="social-links">
                        <a href="#" target="_blank">
                            <i class="icon facebook-icon"></i>
                        </a>
                        <a href="#" target="_blank">
                            <i class="icon twitter-icon"></i>
                        </a>
                        <a href="#" target="_blank">
                            <i class="icon instagram-icon"></i>
                        </a>
                    </div>
                </div>

                <!-- Navigation -->
                <div class="footer-section">
                    <h3>Navigation</h3>
                    <ul>
                        <li><a href="products.html">Tous nos produits</a></li>
                        <li><a href="brands.html">Nos marques</a></li>
                        <li><a href="about.html">À propos</a></li>
                        <li><a href="contact.html">Contact</a></li>
                    </ul>
                </div>

                <!-- Informations Clients -->
                <div class="footer-section">
                    <h3>Informations</h3>
                    <ul>
                        <li><a href="shipping.html">Livraison</a></li>
                        <li><a href="returns.html">Retours</a></li>
                        <li><a href="faq.html">FAQ</a></li>
                        <li><a href="terms.html">CGV</a></li>
                    </ul>
                </div>

                <!-- Contact -->
                <div class="footer-section">
                    <h3>Contact</h3>
                    <ul class="contact-info">
                        <li>
                            <i class="icon map-icon"></i>
                            123 Rue du Commerce<br>
                            75001 Paris, France
                        </li>
                        <li>
                            <i class="icon phone-icon"></i>
                            01 23 45 67 89
                        </li>
                        <li>
                            <i class="icon mail-icon"></i>
                            contact@fmp.com
                        </li>
                    </ul>
                </div>
            </div>

            <!-- Copyright -->
            <div class="footer-bottom">
                <p>&copy; 2024 FMP - For My Printer. Tous droits réservés.</p>
            </div>
        </div>
    </footer>

    <!-- Scripts -->
    <script type="module" src="static/js/main.js"></script>
    <script type="module" src="static/js/home.js"></script>
</body>
</html>
```

# Partie 3 : Catalogue Frontend
## 3.15.2 Styles CSS de la Page d'Accueil

### Styles principaux
```css
/* frontend/static/css/home.css */

/* Variables CSS */
:root {
    --primary-color: #3498db;
    --secondary-color: #2ecc71;
    --text-color: #2c3e50;
    --light-gray: #f5f7fa;
    --border-color: #e1e8ed;
    --shadow: 0 2px 4px rgba(0,0,0,0.1);
}

/* En-tête */
.main-header {
    position: sticky;
    top: 0;
    z-index: 100;
    background: white;
    box-shadow: var(--shadow);
}

/* Top Bar */
.top-bar {
    background: var(--light-gray);
    padding: 0.5rem 0;
    font-size: 0.9rem;
}

.top-bar .container {
    display: flex;
    justify-content: space-between;
    align-items: center;
}

.contact-info {
    display: flex;
    gap: 1.5rem;
}

.contact-info a {
    display: flex;
    align-items: center;
    gap: 0.5rem;
    color: var(--text-color);
    text-decoration: none;
}

.user-nav {
    display: flex;
    gap: 1rem;
}

.user-nav a {
    color: var(--text-color);
    text-decoration: none;
    transition: color 0.3s;
}

.user-nav a:hover {
    color: var(--primary-color);
}

/* Navigation Principale */
.main-nav {
    padding: 1rem 0;
    border-bottom: 1px solid var(--border-color);
}

.main-nav .container {
    display: grid;
    grid-template-columns: auto 1fr auto;
    gap: 2rem;
    align-items: center;
}

.logo img {
    height: 50px;
    width: auto;
}

/* Barre de recherche */
.search-bar {
    max-width: 600px;
    width: 100%;
}

.search-bar form {
    display: flex;
    gap: 0.5rem;
}

.search-bar input {
    flex: 1;
    padding: 0.8rem;
    border: 2px solid var(--border-color);
    border-radius: 4px;
    font-size: 1rem;
    transition: border-color 0.3s;
}

.search-bar input:focus {
    border-color: var(--primary-color);
    outline: none;
}

.search-bar button {
    padding: 0 1.5rem;
    background: var(--primary-color);
    color: white;
    border: none;
    border-radius: 4px;
    cursor: pointer;
    transition: background 0.3s;
}

.search-bar button:hover {
    background: #2980b9;
}

/* Widget Panier */
.cart-widget {
    position: relative;
}

.cart-link {
    display: flex;
    align-items: center;
    gap: 0.5rem;
    text-decoration: none;
    color: var(--text-color);
    padding: 0.5rem 1rem;
    border-radius: 4px;
    transition: background 0.3s;
}

.cart-link:hover {
    background: var(--light-gray);
}

.cart-count {
    position: absolute;
    top: -5px;
    right: -5px;
    background: var(--secondary-color);
    color: white;
    font-size: 0.8rem;
    padding: 0.2rem 0.5rem;
    border-radius: 10px;
    min-width: 20px;
    text-align: center;
}

/* Menu Catégories */
.category-nav {
    background: var(--light-gray);
    padding: 0.5rem 0;
}

.category-nav ul {
    display: flex;
    gap: 2rem;
    list-style: none;
    margin: 0;
    padding: 0;
}

.category-nav a {
    color: var(--text-color);
    text-decoration: none;
    font-weight: 500;
    transition: color 0.3s;
}

.category-nav a:hover {
    color: var(--primary-color);
}

/* Slider Principal */
.hero-slider {
    position: relative;
    margin-bottom: 3rem;
}

.slider-container {
    position: relative;
    height: 400px;
    overflow: hidden;
}

.slide {
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    opacity: 0;
    transition: opacity 0.5s;
}

.slide.active {
    opacity: 1;
}

.slide img {
    width: 100%;
    height: 100%;
    object-fit: cover;
}

.slide-content {
    position: absolute;
    top: 50%;
    left: 10%;
    transform: translateY(-50%);
    color: white;
    text-shadow: 0 2px 4px rgba(0,0,0,0.3);
}

.slide-content h2 {
    font-size: 2.5rem;
    margin-bottom: 1rem;
}

.slider-controls button {
    position: absolute;
    top: 50%;
    transform: translateY(-50%);
    background: rgba(0,0,0,0.5);
    color: white;
    border: none;
    padding: 1rem;
    cursor: pointer;
    transition: background 0.3s;
}

.slider-controls button:hover {
    background: rgba(0,0,0,0.7);
}

.slider-controls .prev-btn {
    left: 20px;
}

.slider-controls .next-btn {
    right: 20px;
}

/* Avantages */
.features {
    padding: 3rem 0;
    background: white;
}

.feature-grid {
    display: grid;
    grid-template-columns: repeat(4, 1fr);
    gap: 2rem;
}

.feature-item {
    text-align: center;
}

.feature-item i {
    font-size: 2.5rem;
    color: var(--primary-color);
    margin-bottom: 1rem;
}

.feature-item h3 {
    margin-bottom: 0.5rem;
    color: var(--text-color);
}

.feature-item p {
    color: #666;
}

/* Catégories Populaires */
.popular-categories {
    padding: 4rem 0;
    background: var(--light-gray);
}

.section-title {
    text-align: center;
    margin-bottom: 2rem;
    color: var(--text-color);
}

.category-grid {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
    gap: 2rem;
}

/* Produits en Vedette */
.featured-products {
    padding: 4rem 0;
}

.products-grid {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
    gap: 2rem;
}

/* Marques Partenaires */
.brands {
    padding: 4rem 0;
    background: var(--light-gray);
}

.brands-slider {
    display: flex;
    gap: 2rem;
    overflow: hidden;
    padding: 1rem 0;
}

/* Newsletter */
.newsletter {
    padding: 4rem 0;
    background: var(--primary-color);
    color: white;
}

.newsletter-content {
    text-align: center;
    max-width: 600px;
    margin: 0 auto;
}

.newsletter-content h2 {
    margin-bottom: 1rem;
}

.newsletter-content p {
    margin-bottom: 2rem;
    opacity: 0.9;
}

.newsletter form {
    display: flex;
    gap: 1rem;
}

.newsletter input {
    flex: 1;
    padding: 1rem;
    border: none;
    border-radius: 4px;
    font-size: 1rem;
}

.newsletter .cta-button {
    padding: 1rem 2rem;
    background: var(--secondary-color);
    color: white;
    border: none;
    border-radius: 4px;
    cursor: pointer;
    transition: background 0.3s;
}

.newsletter .cta-button:hover {
    background: #27ae60;
}

/* Pied de page */
.main-footer {
    background: #2c3e50;
    color: white;
    padding: 4rem 0 2rem;
}

.footer-grid {
    display: grid;
    grid-template-columns: repeat(4, 1fr);
    gap: 2rem;
    margin-bottom: 3rem;
}

.footer-section h3 {
    margin-bottom: 1.5rem;
    color: white;
}

.footer-section ul {
    list-style: none;
    padding: 0;
    margin: 0;
}

.footer-section a {
    color: rgba(255,255,255,0.8);
    text-decoration: none;
    transition: color 0.3s;
}

.footer-section a:hover {
    color: white;
}

.social-links {
    display: flex;
    gap: 1rem;
    margin-top: 1rem;
}

.social-links a {
    width: 40px;
    height: 40px;
    display: flex;
    align-items: center;
    justify-content: center;
    background: rgba(255,255,255,0.1);
    border-radius: 50%;
    transition: background 0.3s;
}

.social-links a:hover {
    background: rgba(255,255,255,0.2);
}

.footer-bottom {
    text-align: center;
    padding-top: 2rem;
    border-top: 1px solid rgba(255,255,255,0.1);
}

/* Boutons CTA */
.cta-button {
    display: inline-block;
    padding: 1rem 2rem;
    background: var(--primary-color);
    color: white;
    text-decoration: none;
    border-radius: 4px;
    transition: background 0.3s;
}

.cta-button:hover {
    background: #2980b9;
}

/* Media Queries */
@media (max-width: 1024px) {
    .feature-grid {
        grid-template-columns: repeat(2, 1fr);
    }

    .footer-grid {
        grid-template-columns: repeat(2, 1fr);
    }
}

@media (max-width: 768px) {
    .main-nav .container {
        grid-template-columns: 1fr;
        gap: 1rem;
    }

    .category-nav {
        display: none;
    }

    .slider-container {
        height: 300px;
    }

    .slide-content h2 {
        font-size: 2rem;
    }

    .newsletter form {
        flex-direction: column;
    }

    .newsletter .cta-button {
        width: 100%;
    }
}

@media (max-width: 480px) {
    .top-bar .container {
        flex-direction: column;
        gap: 0.5rem;
    }

    .contact-info {
        flex-direction: column;
        align-items: center;
    }

    .feature-grid {
        grid-template-columns: 1fr;
    }

    .footer-grid {
        grid-template-columns: 1fr;
    }
}

/* Animations */
@keyframes slideIn {
    from {
        opacity: 0;
        transform: translateY(20px);
    }
    to {
        opacity: 1;
        transform: translateY(0);
    }
}

.feature-item,
.category-grid > *,
.products-grid > * {
    animation: slideIn 0.5s ease-out forwards;
}
```

# Partie 3 : Catalogue Frontend
## 3.15.3 JavaScript de la Page d'Accueil

### main.js - Fonctionnalités communes
```javascript
// frontend/static/js/main.js

import { session } from './session.js';
import { notifications } from './notifications.js';

/**
 * Gestionnaire principal du site
 */
class MainManager {
    constructor() {
        this.cartItems = [];
        this.initialize();
    }

    /**
     * Initialisation des fonctionnalités communes
     */
    async initialize() {
        // Chargement du panier
        this.loadCart();
        
        // Mise à jour de l'interface utilisateur
        this.updateUserInterface();
        
        // Mise en place des écouteurs d'événements
        this.setupEventListeners();
        
        // Chargement du menu des catégories
        await this.loadCategories();
    }

    /**
     * Charge le panier depuis le localStorage
     */
    loadCart() {
        const savedCart = localStorage.getItem('cart');
        if (savedCart) {
            try {
                this.cartItems = JSON.parse(savedCart);
                this.updateCartCount();
            } catch (error) {
                console.error('Erreur chargement panier:', error);
                this.cartItems = [];
            }
        }
    }

    /**
     * Met à jour l'interface selon l'état de connexion
     */
    updateUserInterface() {
        const isLoggedIn = session.isAuthenticated();
        const userData = session.getUserData();

        // Mise à jour des liens de connexion/inscription
        document.getElementById('login-link').style.display = 
            isLoggedIn ? 'none' : 'block';
        document.getElementById('register-link').style.display = 
            isLoggedIn ? 'none' : 'block';

        // Si connecté, ajouter le menu utilisateur
        if (isLoggedIn) {
            this.createUserMenu(userData);
        }
    }

    /**
     * Crée le menu utilisateur
     */
    createUserMenu(userData) {
        const userNav = document.querySelector('.user-nav');
        userNav.innerHTML = `
            <div class="user-menu">
                <button class="user-menu-btn">
                    <i class="icon user-icon"></i>
                    ${userData.firstname}
                </button>
                <div class="user-menu-dropdown">
                    <a href="account/profile.html">Mon Profil</a>
                    <a href="account/orders.html">Mes Commandes</a>
                    <a href="account/addresses.html">Mes Adresses</a>
                    <button onclick="mainManager.handleLogout()">
                        Déconnexion
                    </button>
                </div>
            </div>
        `;
    }

    /**
     * Charge les catégories pour le menu
     */
    async loadCategories() {
        try {
            const response = await fetch('/api/products/categories');
            const categories = await response.json();
            
            const menu = document.getElementById('category-menu');
            menu.innerHTML = categories.map(category => `
                <li>
                    <a href="products.html?category=${category.slug}">
                        ${category.name}
                    </a>
                </li>
            `).join('');
            
        } catch (error) {
            console.error('Erreur chargement catégories:', error);
        }
    }

    /**
     * Met en place les écouteurs d'événements
     */
    setupEventListeners() {
        // Formulaire de recherche
        const searchForm = document.querySelector('.search-bar form');
        if (searchForm) {
            searchForm.addEventListener('submit', (e) => {
                e.preventDefault();
                const query = document.getElementById('search-input').value.trim();
                if (query) {
                    window.location.href = `search.html?q=${encodeURIComponent(query)}`;
                }
            });
        }

        // Newsletter
        const newsletterForm = document.getElementById('newsletter-form');
        if (newsletterForm) {
            newsletterForm.addEventListener('submit', this.handleNewsletterSignup.bind(this));
        }
    }

    /**
     * Gère l'inscription à la newsletter
     */
    async handleNewsletterSignup(e) {
        e.preventDefault();
        
        const email = e.target.querySelector('input[type="email"]').value;
        
        try {
            const response = await fetch('/api/newsletter/subscribe', {
                method: 'POST',
                headers: {
                    'Content-Type': 'application/json'
                },
                body: JSON.stringify({ email })
            });

            if (!response.ok) throw new Error('Erreur inscription newsletter');

            notifications.create({
                type: 'success',
                title: 'Merci !',
                message: 'Vous êtes inscrit à notre newsletter'
            });

            e.target.reset();

        } catch (error) {
            notifications.create({
                type: 'error',
                title: 'Erreur',
                message: 'Impossible de vous inscrire à la newsletter'
            });
        }
    }

    /**
     * Met à jour le compteur du panier
     */
    updateCartCount() {
        const count = this.cartItems.reduce((total, item) => total + item.quantity, 0);
        const total = this.cartItems.reduce((sum, item) => sum + (item.price * item.quantity), 0);
        
        document.getElementById('cart-count').textContent = count;
        document.getElementById('cart-total').textContent = `${total.toFixed(2)} €`;
    }

    /**
     * Gère la déconnexion
     */
    async handleLogout() {
        await session.handleLogout();
        window.location.href = '/';
    }
}

// Initialisation
const mainManager = new MainManager();
// Rendre l'instance accessible globalement
window.mainManager = mainManager;
```

### home.js - Fonctionnalités spécifiques à la page d'accueil
```javascript
// frontend/static/js/home.js

/**
 * Gestionnaire de la page d'accueil
 */
class HomeManager {
    constructor() {
        this.currentSlide = 0;
        this.autoPlayInterval = null;
        this.initialize();
    }

    /**
     * Initialisation des fonctionnalités de la page d'accueil
     */
    async initialize() {
        // Initialisation du slider
        this.initializeSlider();
        
        // Chargement des données
        await Promise.all([
            this.loadFeaturedCategories(),
            this.loadFeaturedProducts(),
            this.loadBrands()
        ]);
    }

    /**
     * Initialise le slider principal
     */
    initializeSlider() {
        // Gestion des contrôles du slider
        const prevBtn = document.querySelector('.slider-controls .prev-btn');
        const nextBtn = document.querySelector('.slider-controls .next-btn');
        
        prevBtn.addEventListener('click', () => this.changeSlide(-1));
        nextBtn.addEventListener('click', () => this.changeSlide(1));
        
        // Démarrage de l'autoplay
        this.startAutoPlay();
        
        // Pause de l'autoplay au survol
        const slider = document.querySelector('.hero-slider');
        slider.addEventListener('mouseenter', () => this.stopAutoPlay());
        slider.addEventListener('mouseleave', () => this.startAutoPlay());
    }

    /**
     * Change la slide active
     */
    changeSlide(direction) {
        const slides = document.querySelectorAll('.slide');
        
        // Retrait de la classe active
        slides[this.currentSlide].classList.remove('active');
        
        // Calcul de la nouvelle slide
        this.currentSlide = (this.currentSlide + direction + slides.length) % slides.length;
        
        // Activation de la nouvelle slide
        slides[this.currentSlide].classList.add('active');
    }

    /**
     * Démarre l'autoplay du slider
     */
    startAutoPlay() {
        this.autoPlayInterval = setInterval(() => {
            this.changeSlide(1);
        }, 5000);
    }

    /**
     * Arrête l'autoplay du slider
     */
    stopAutoPlay() {
        if (this.autoPlayInterval) {
            clearInterval(this.autoPlayInterval);
            this.autoPlayInterval = null;
        }
    }

    /**
     * Charge les catégories mises en avant
     */
    async loadFeaturedCategories() {
        try {
            const response = await fetch('/api/products/categories/featured');
            const categories = await response.json();
            
            const container = document.getElementById('featured-categories');
            container.innerHTML = categories.map(category => `
                <a href="products.html?category=${category.slug}" class="category-card">
                    <div class="category-image">
                        <img src="${category.image_url}" alt="${category.name}">
                    </div>
                    <div class="category-info">
                        <h3>${category.name}</h3>
                        <p>${category.description}</p>
                    </div>
                </a>
            `).join('');
            
        } catch (error) {
            console.error('Erreur chargement catégories:', error);
        }
    }

    /**
     * Charge les produits mis en avant
     */
    async loadFeaturedProducts() {
        try {
            const response = await fetch('/api/products/featured');
            const products = await response.json();
            
            const container = document.getElementById('featured-products');
            container.innerHTML = products.map(product => `
                <div class="product-card">
                    <div class="product-image">
                        <img src="${product.images[0]?.url}" alt="${product.name}">
                    </div>
                    <div class="product-info">
                        <h3>${product.name}</h3>
                        <p class="product-brand">${product.brand.name}</p>
                        <p class="product-price">${product.price.toFixed(2)} €</p>
                    </div>
                    <button 
                        class="add-to-cart-btn"
                        onclick="mainManager.addToCart(${product.id})"
                    >
                        Ajouter au panier
                    </button>
                </div>
            `).join('');
            
        } catch (error) {
            console.error('Erreur chargement produits:', error);
        }
    }

    /**
     * Charge les marques partenaires
     */
    async loadBrands() {
        try {
            const response = await fetch('/api/products/brands');
            const brands = await response.json();
            
            const container = document.getElementById('brands-slider');
            container.innerHTML = brands.map(brand => `
                <div class="brand-item">
                    <img src="${brand.logo_url}" alt="${brand.name}">
                </div>
            `).join('');
            
        } catch (error) {
            console.error('Erreur chargement marques:', error);
        }
    }
}

// Initialisation
const homeManager = new HomeManager();
// Rendre l'instance accessible globalement
window.homeManager = homeManager;
```

# Partie 3 : Catalogue Frontend
## 3.16 Page Liste des Produits

### Description
Cette section comprend :
- La page de liste des produits
- Les filtres de recherche
- Le système de tri
- La pagination
- L'affichage en grille/liste

### 3.16.1 Page Liste des Produits
```html
<!-- frontend/pages/products.html -->
<!DOCTYPE html>
<html lang="fr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>FMP - Nos Cartouches d'Imprimante</title>
    <meta name="description" content="Découvrez notre large gamme de cartouches d'imprimantes. Compatibles et originales pour toutes les marques.">
    
    <!-- Styles -->
    <link rel="stylesheet" href="../static/css/main.css">
    <link rel="stylesheet" href="../static/css/products.css">
</head>
<body>
    <!-- En-tête (identique à index.html) -->
    <header class="main-header">
        <!-- ... contenu de l'en-tête ... -->
    </header>

    <!-- Contenu principal -->
    <main class="products-page">
        <div class="container">
            <!-- Fil d'Ariane -->
            <nav class="breadcrumb">
                <ul>
                    <li><a href="/">Accueil</a></li>
                    <li>Cartouches</li>
                </ul>
            </nav>

            <!-- En-tête de la page -->
            <div class="page-header">
                <h1>Nos Cartouches d'Imprimante</h1>
                <div class="view-options">
                    <!-- Options d'affichage -->
                    <div class="display-mode">
                        <button 
                            class="view-btn active" 
                            data-view="grid"
                            onclick="productsManager.changeView('grid')"
                        >
                            <i class="icon grid-icon"></i>
                        </button>
                        <button 
                            class="view-btn" 
                            data-view="list"
                            onclick="productsManager.changeView('list')"
                        >
                            <i class="icon list-icon"></i>
                        </button>
                    </div>
                    <!-- Tri -->
                    <select 
                        id="sort-select"
                        onchange="productsManager.changeSort(this.value)"
                    >
                        <option value="popularity">Popularité</option>
                        <option value="price_asc">Prix croissant</option>
                        <option value="price_desc">Prix décroissant</option>
                        <option value="name_asc">Nom A-Z</option>
                        <option value="name_desc">Nom Z-A</option>
                    </select>
                </div>
            </div>

            <!-- Zone principale -->
            <div class="products-layout">
                <!-- Filtres -->
                <aside class="filters-sidebar">
                    <div class="filters-header">
                        <h2>Filtres</h2>
                        <button 
                            class="clear-filters"
                            onclick="productsManager.clearFilters()"
                        >
                            Réinitialiser
                        </button>
                    </div>

                    <!-- Prix -->
                    <div class="filter-group">
                        <h3>Prix</h3>
                        <div class="price-range">
                            <input 
                                type="range" 
                                id="price-range" 
                                min="0" 
                                max="100"
                                step="5"
                                value="100"
                                oninput="productsManager.updatePriceFilter(this.value)"
                            >
                            <div class="price-inputs">
                                <input 
                                    type="number" 
                                    id="min-price" 
                                    placeholder="Min"
                                    min="0"
                                >
                                <span>-</span>
                                <input 
                                    type="number" 
                                    id="max-price" 
                                    placeholder="Max"
                                    min="0"
                                >
                            </div>
                        </div>
                    </div>

                    <!-- Marques -->
                    <div class="filter-group">
                        <h3>Marques</h3>
                        <div class="filter-search">
                            <input 
                                type="text" 
                                placeholder="Rechercher une marque"
                                oninput="productsManager.filterBrands(this.value)"
                            >
                        </div>
                        <div class="checkbox-list" id="brands-list">
                            <!-- Les marques seront injectées ici -->
                        </div>
                    </div>

                    <!-- Catégories -->
                    <div class="filter-group">
                        <h3>Catégories</h3>
                        <div class="checkbox-list" id="categories-list">
                            <!-- Les catégories seront injectées ici -->
                        </div>
                    </div>

                    <!-- Disponibilité -->
                    <div class="filter-group">
                        <h3>Disponibilité</h3>
                        <label class="checkbox-label">
                            <input 
                                type="checkbox" 
                                id="in-stock-only"
                                onchange="productsManager.toggleInStock(this.checked)"
                            >
                            En stock uniquement
                        </label>
                    </div>
                </aside>

                <!-- Liste des produits -->
                <div class="products-content">
                    <!-- Résumé des filtres actifs -->
                    <div class="active-filters" id="active-filters">
                        <!-- Les filtres actifs seront injectés ici -->
                    </div>

                    <!-- Grille des produits -->
                    <div 
                        class="products-grid" 
                        id="products-container"
                        data-view="grid"
                    >
                        <!-- Les produits seront injectés ici -->
                    </div>

                    <!-- Message "Aucun produit" -->
                    <div 
                        class="no-products" 
                        id="no-products"
                        style="display: none;"
                    >
                        <i class="icon search-icon"></i>
                        <h3>Aucun produit trouvé</h3>
                        <p>Essayez de modifier vos filtres de recherche</p>
                        <button 
                            class="btn"
                            onclick="productsManager.clearFilters()"
                        >
                            Réinitialiser les filtres
                        </button>
                    </div>

                    <!-- Loader -->
                    <div 
                        class="products-loader" 
                        id="products-loader"
                        style="display: none;"
                    >
                        <div class="loader"></div>
                        <p>Chargement des produits...</p>
                    </div>

                    <!-- Pagination -->
                    <div class="pagination" id="pagination">
                        <!-- La pagination sera injectée ici -->
                    </div>
                </div>
            </div>
        </div>
    </main>

    <!-- Pied de page (identique à index.html) -->
    <footer class="main-footer">
        <!-- ... contenu du pied de page ... -->
    </footer>

    <!-- Templates -->
    <template id="product-card-template">
        <div class="product-card">
            <div class="product-image">
                <img src="" alt="">
                <div class="product-badges">
                    <span class="badge new-badge">Nouveau</span>
                    <span class="badge sale-badge">Promo</span>
                </div>
            </div>
            <div class="product-info">
                <h3 class="product-name">
                    <a href=""></a>
                </h3>
                <div class="product-brand"></div>
                <div class="product-price">
                    <span class="current-price"></span>
                    <span class="original-price"></span>
                    <span class="discount-tag"></span>
                </div>
                <div class="product-stock"></div>
            </div>
            <div class="product-actions">
                <button class="add-to-cart-btn">
                    Ajouter au panier
                </button>
                <button class="quick-view-btn">
                    <i class="icon eye-icon"></i>
                </button>
            </div>
        </div>
    </template>

    <!-- Scripts -->
    <script type="module" src="../static/js/main.js"></script>
    <script type="module" src="../static/js/products.js"></script>
</body>
</html>
```

# Partie 3 : Catalogue Frontend
## 3.16.2 Styles CSS de la Page Produits

```css
/* frontend/static/css/products.css */

/* Layout de la page */
.products-page {
    padding: 2rem 0;
}

/* Fil d'Ariane */
.breadcrumb {
    margin-bottom: 1.5rem;
}

.breadcrumb ul {
    display: flex;
    gap: 0.5rem;
    list-style: none;
    padding: 0;
    margin: 0;
}

.breadcrumb li:not(:last-child)::after {
    content: '/';
    margin-left: 0.5rem;
    color: #666;
}

.breadcrumb a {
    color: var(--primary-color);
    text-decoration: none;
}

.breadcrumb a:hover {
    text-decoration: underline;
}

/* En-tête de page */
.page-header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    margin-bottom: 2rem;
}

.page-header h1 {
    margin: 0;
    font-size: 1.8rem;
    color: var(--text-color);
}

/* Options d'affichage */
.view-options {
    display: flex;
    gap: 1rem;
    align-items: center;
}

.display-mode {
    display: flex;
    gap: 0.5rem;
}

.view-btn {
    width: 40px;
    height: 40px;
    display: flex;
    align-items: center;
    justify-content: center;
    border: 1px solid var(--border-color);
    border-radius: 4px;
    background: white;
    cursor: pointer;
    transition: all 0.3s;
}

.view-btn.active {
    background: var(--primary-color);
    border-color: var(--primary-color);
    color: white;
}

.view-btn:hover:not(.active) {
    background: var(--light-gray);
}

/* Select de tri */
#sort-select {
    padding: 0.5rem;
    border: 1px solid var(--border-color);
    border-radius: 4px;
    background: white;
    font-size: 0.9rem;
    min-width: 200px;
}

/* Layout produits */
.products-layout {
    display: grid;
    grid-template-columns: 280px 1fr;
    gap: 2rem;
}

/* Sidebar des filtres */
.filters-sidebar {
    background: white;
    border-radius: 8px;
    padding: 1.5rem;
    box-shadow: var(--shadow);
}

.filters-header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    margin-bottom: 1.5rem;
}

.filters-header h2 {
    font-size: 1.2rem;
    margin: 0;
}

.clear-filters {
    background: none;
    border: none;
    color: var(--primary-color);
    font-size: 0.9rem;
    cursor: pointer;
}

.clear-filters:hover {
    text-decoration: underline;
}

/* Groupes de filtres */
.filter-group {
    border-top: 1px solid var(--border-color);
    padding: 1.5rem 0;
}

.filter-group h3 {
    font-size: 1rem;
    margin: 0 0 1rem 0;
}

/* Range de prix */
.price-range {
    padding: 0 0.5rem;
}

.price-range input[type="range"] {
    width: 100%;
    margin: 1rem 0;
}

.price-inputs {
    display: flex;
    gap: 0.5rem;
    align-items: center;
}

.price-inputs input {
    width: 80px;
    padding: 0.5rem;
    border: 1px solid var(--border-color);
    border-radius: 4px;
}

/* Liste de checkboxes */
.checkbox-list {
    max-height: 200px;
    overflow-y: auto;
    padding-right: 0.5rem;
}

.checkbox-label {
    display: flex;
    align-items: center;
    gap: 0.5rem;
    padding: 0.5rem 0;
    cursor: pointer;
}

.checkbox-label input[type="checkbox"] {
    width: 16px;
    height: 16px;
}

/* Recherche dans les filtres */
.filter-search {
    margin-bottom: 1rem;
}

.filter-search input {
    width: 100%;
    padding: 0.5rem;
    border: 1px solid var(--border-color);
    border-radius: 4px;
}

/* Zone de contenu des produits */
.products-content {
    flex: 1;
}

/* Filtres actifs */
.active-filters {
    display: flex;
    flex-wrap: wrap;
    gap: 0.5rem;
    margin-bottom: 1.5rem;
}

.filter-tag {
    display: inline-flex;
    align-items: center;
    gap: 0.5rem;
    padding: 0.5rem 1rem;
    background: var(--light-gray);
    border-radius: 20px;
    font-size: 0.9rem;
}

.filter-tag button {
    background: none;
    border: none;
    color: #666;
    cursor: pointer;
    padding: 0;
    font-size: 1.2rem;
    line-height: 1;
}

/* Grille de produits */
.products-grid {
    display: grid;
    grid-template-columns: repeat(auto-fill, minmax(250px, 1fr));
    gap: 1.5rem;
    margin-bottom: 2rem;
}

/* Vue liste */
.products-grid[data-view="list"] {
    grid-template-columns: 1fr;
}

.products-grid[data-view="list"] .product-card {
    display: grid;
    grid-template-columns: 200px 1fr auto;
    gap: 1.5rem;
}

/* Carte produit */
.product-card {
    background: white;
    border-radius: 8px;
    overflow: hidden;
    transition: transform 0.3s, box-shadow 0.3s;
    box-shadow: var(--shadow);
}

.product-card:hover {
    transform: translateY(-5px);
    box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
}

.product-image {
    position: relative;
    padding-top: 100%;
}

.product-image img {
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    object-fit: cover;
}

/* Badges */
.product-badges {
    position: absolute;
    top: 1rem;
    left: 1rem;
    display: flex;
    flex-direction: column;
    gap: 0.5rem;
}

.badge {
    padding: 0.25rem 0.5rem;
    border-radius: 4px;
    font-size: 0.8rem;
    font-weight: 500;
}

.new-badge {
    background: var(--primary-color);
    color: white;
}

.sale-badge {
    background: #e74c3c;
    color: white;
}

/* Informations produit */
.product-info {
    padding: 1rem;
}

.product-name {
    margin: 0 0 0.5rem 0;
    font-size: 1rem;
}

.product-name a {
    color: var(--text-color);
    text-decoration: none;
}

.product-name a:hover {
    color: var(--primary-color);
}

.product-brand {
    color: #666;
    font-size: 0.9rem;
    margin-bottom: 0.5rem;
}

/* Prix */
.product-price {
    display: flex;
    align-items: center;
    gap: 0.5rem;
    margin-bottom: 0.5rem;
}

.current-price {
    font-size: 1.2rem;
    font-weight: 600;
    color: var(--text-color);
}

.original-price {
    font-size: 0.9rem;
    color: #999;
    text-decoration: line-through;
}

.discount-tag {
    background: #e74c3c;
    color: white;
    padding: 0.25rem 0.5rem;
    border-radius: 4px;
    font-size: 0.8rem;
}

/* Stock */
.product-stock {
    font-size: 0.9rem;
    margin-bottom: 1rem;
}

.in-stock {
    color: #27ae60;
}

.low-stock {
    color: #f39c12;
}

.out-of-stock {
    color: #e74c3c;
}

/* Actions */
.product-actions {
    padding: 1rem;
    border-top: 1px solid var(--border-color);
    display: flex;
    gap: 0.5rem;
}

.add-to-cart-btn {
    flex: 1;
    padding: 0.8rem;
    background: var(--primary-color);
    color: white;
    border: none;
    border-radius: 4px;
    cursor: pointer;
    transition: background 0.3s;
}

.add-to-cart-btn:hover {
    background: #2980b9;
}

.quick-view-btn {
    width: 40px;
    height: 40px;
    display: flex;
    align-items: center;
    justify-content: center;
    border: 1px solid var(--border-color);
    border-radius: 4px;
    background: white;
    cursor: pointer;
    transition: all 0.3s;
}

.quick-view-btn:hover {
    background: var(--light-gray);
}

/* États de chargement */
.products-loader {
    text-align: center;
    padding: 3rem 0;
}

.loader {
    width: 40px;
    height: 40px;
    border: 3px solid var(--light-gray);
    border-top-color: var(--primary-color);
    border-radius: 50%;
    animation: spin 1s linear infinite;
    margin: 0 auto 1rem;
}

@keyframes spin {
    to {
        transform: rotate(360deg);
    }
}

/* Message "Aucun produit" */
.no-products {
    text-align: center;
    padding: 3rem 0;
}

.no-products .icon {
    font-size: 3rem;
    color: #999;
    margin-bottom: 1rem;
}

.no-products h3 {
    margin-bottom: 0.5rem;
    color: var(--text-color);
}

.no-products p {
    color: #666;
    margin-bottom: 1.5rem;
}

/* Pagination */
.pagination {
    display: flex;
    justify-content: center;
    gap: 0.5rem;
    margin-top: 2rem;
}

.pagination button {
    min-width: 40px;
    height: 40px;
    padding: 0 0.8rem;
    border: 1px solid var(--border-color);
    background: white;
    border-radius: 4px;
    cursor: pointer;
    transition: all 0.3s;
}

.pagination button.active {
    background: var(--primary-color);
    color: white;
    border-color: var(--primary-color);
}

.pagination button:hover:not(.active) {
    background: var(--light-gray);
}

.pagination button:disabled {
    opacity: 0.5;
    cursor: not-allowed;
}

/* Responsive */
@media (max-width: 1024px) {
    .products-layout {
        grid-template-columns: 1fr;
    }

    .filters-sidebar {
        display: none;
        position: fixed;
        top: 0;
        left: 0;
        right: 0;
        bottom: 0;
        z-index: 1000;
        background: white;
        overflow-y: auto;
    }

    .filters-sidebar.show {
        display: block;
    }
}

@media (max-width: 768px) {
    .page-header {
        flex-direction: column;
        gap: 1rem;
        align-items: flex-start;
    }

    .products-grid[data-view="list"] .product-card {
        grid-template-columns: 1fr;
    }
}

@media (max-width: 480px) {
    .view-options {
        flex-direction: column;
        align-items: flex-start;
        gap: 0.5rem;
    }

    #sort-select {
        width: 100%;
    }
}
```

# Partie 3 : Catalogue Frontend
## 3.16.3 JavaScript de la Page Produits

```javascript
// frontend/static/js/products.js

import { notifications } from './notifications.js';

/**
 * Gestionnaire de la page produits
 */
class ProductsManager {
    constructor() {
        // État initial
        this.products = [];
        this.filteredProducts = [];
        this.currentPage = 1;
        this.itemsPerPage = 12;
        this.filters = {
            price: {
                min: null,
                max: null
            },
            brands: [],
            categories: [],
            inStock: false
        };
        this.currentView = 'grid';
        this.currentSort = 'popularity';
        
        // Initialisation
        this.initialize();
    }

    /**
     * Initialisation de la page
     */
    async initialize() {
        try {
            // Chargement des données nécessaires
            await Promise.all([
                this.loadProducts(),
                this.loadBrands(),
                this.loadCategories()
            ]);

            // Mise en place des écouteurs d'événements
            this.setupEventListeners();

            // Vérification des paramètres d'URL
            this.checkUrlParams();

            // Application des filtres initiaux
            this.applyFilters();

        } catch (error) {
            console.error('Erreur initialisation:', error);
            notifications.create({
                type: 'error',
                title: 'Erreur',
                message: 'Impossible de charger les produits'
            });
        }
    }

    /**
     * Charge les produits depuis l'API
     */
    async loadProducts() {
        const response = await fetch('/api/products');
        this.products = await response.json();
        this.filteredProducts = [...this.products];
    }

    /**
     * Charge les marques depuis l'API
     */
    async loadBrands() {
        const response = await fetch('/api/products/brands');
        const brands = await response.json();
        
        const container = document.getElementById('brands-list');
        container.innerHTML = brands.map(brand => `
            <label class="checkbox-label">
                <input 
                    type="checkbox" 
                    name="brand" 
                    value="${brand.id}"
                    onchange="productsManager.handleBrandFilter(this)"
                >
                ${brand.name}
            </label>
        `).join('');
    }

    /**
     * Charge les catégories depuis l'API
     */
    async loadCategories() {
        const response = await fetch('/api/products/categories');
        const categories = await response.json();
        
        const container = document.getElementById('categories-list');
        container.innerHTML = categories.map(category => `
            <label class="checkbox-label">
                <input 
                    type="checkbox" 
                    name="category" 
                    value="${category.id}"
                    onchange="productsManager.handleCategoryFilter(this)"
                >
                ${category.name}
            </label>
        `).join('');
    }

    /**
     * Vérifie les paramètres d'URL pour les filtres initiaux
     */
    checkUrlParams() {
        const params = new URLSearchParams(window.location.search);
        
        // Catégorie
        const category = params.get('category');
        if (category) {
            const checkbox = document.querySelector(`input[name="category"][value="${category}"]`);
            if (checkbox) {
                checkbox.checked = true;
                this.handleCategoryFilter(checkbox);
            }
        }
        
        // Prix
        const minPrice = params.get('min_price');
        const maxPrice = params.get('max_price');
        if (minPrice) document.getElementById('min-price').value = minPrice;
        if (maxPrice) document.getElementById('max-price').value = maxPrice;
        
        // Vue
        const view = params.get('view');
        if (view) this.changeView(view);
        
        // Tri
        const sort = params.get('sort');
        if (sort) {
            document.getElementById('sort-select').value = sort;
            this.currentSort = sort;
        }
    }

    /**
     * Met en place les écouteurs d'événements
     */
    setupEventListeners() {
        // Prix minimum
        document.getElementById('min-price').addEventListener('change', (e) => {
            this.filters.price.min = e.target.value ? Number(e.target.value) : null;
            this.applyFilters();
        });
        
        // Prix maximum
        document.getElementById('max-price').addEventListener('change', (e) => {
            this.filters.price.max = e.target.value ? Number(e.target.value) : null;
            this.applyFilters();
        });
    }

    /**
     * Applique tous les filtres actifs
     */
    applyFilters() {
        this.showLoader();
        
        // Réinitialisation
        this.filteredProducts = [...this.products];
        
        // Prix
        if (this.filters.price.min !== null) {
            this.filteredProducts = this.filteredProducts.filter(
                product => product.price >= this.filters.price.min
            );
        }
        if (this.filters.price.max !== null) {
            this.filteredProducts = this.filteredProducts.filter(
                product => product.price <= this.filters.price.max
            );
        }
        
        // Marques
        if (this.filters.brands.length > 0) {
            this.filteredProducts = this.filteredProducts.filter(
                product => this.filters.brands.includes(product.brand_id)
            );
        }
        
        // Catégories
        if (this.filters.categories.length > 0) {
            this.filteredProducts = this.filteredProducts.filter(
                product => this.filters.categories.includes(product.category_id)
            );
        }
        
        // Stock
        if (this.filters.inStock) {
            this.filteredProducts = this.filteredProducts.filter(
                product => product.stock_quantity > 0
            );
        }
        
        // Tri
        this.sortProducts();
        
        // Mise à jour de l'affichage
        this.currentPage = 1;
        this.updateDisplay();
        this.updateActiveFilters();
        this.updateUrl();
    }

    /**
     * Trie les produits selon le critère actuel
     */
    sortProducts() {
        switch (this.currentSort) {
            case 'price_asc':
                this.filteredProducts.sort((a, b) => a.price - b.price);
                break;
            case 'price_desc':
                this.filteredProducts.sort((a, b) => b.price - a.price);
                break;
            case 'name_asc':
                this.filteredProducts.sort((a, b) => a.name.localeCompare(b.name));
                break;
            case 'name_desc':
                this.filteredProducts.sort((a, b) => b.name.localeCompare(a.name));
                break;
            default: // popularity
                this.filteredProducts.sort((a, b) => b.sales - a.sales);
        }
    }

    /**
     * Met à jour l'affichage des produits
     */
    updateDisplay() {
        const container = document.getElementById('products-container');
        const start = (this.currentPage - 1) * this.itemsPerPage;
        const end = start + this.itemsPerPage;
        const pageProducts = this.filteredProducts.slice(start, end);
        
        // Affichage des produits
        if (pageProducts.length > 0) {
            container.innerHTML = pageProducts.map(product => this.createProductCard(product)).join('');
            document.getElementById('no-products').style.display = 'none';
        } else {
            container.innerHTML = '';
            document.getElementById('no-products').style.display = 'block';
        }
        
        // Mise à jour de la pagination
        this.updatePagination();
        
        this.hideLoader();
    }

    /**
     * Crée une carte produit
     */
    createProductCard(product) {
        const template = document.getElementById('product-card-template');
        const clone = template.content.cloneNode(true);
        
        // Image
        const img = clone.querySelector('.product-image img');
        img.src = product.images[0]?.url || '/static/images/no-image.png';
        img.alt = product.name;
        
        // Badges
        const badges = clone.querySelector('.product-badges');
        if (product.is_new) {
            badges.innerHTML += '<span class="badge new-badge">Nouveau</span>';
        }
        if (product.discount > 0) {
            badges.innerHTML += '<span class="badge sale-badge">-${product.discount}%</span>';
        }
        
        // Informations
        clone.querySelector('.product-name a').textContent = product.name;
        clone.querySelector('.product-name a').href = `product.html?id=${product.id}`;
        clone.querySelector('.product-brand').textContent = product.brand.name;
        
        // Prix
        const priceElement = clone.querySelector('.product-price');
        if (product.discount > 0) {
            const originalPrice = product.price;
            const discountedPrice = originalPrice * (1 - product.discount / 100);
            priceElement.innerHTML = `
                <span class="current-price">${discountedPrice.toFixed(2)} €</span>
                <span class="original-price">${originalPrice.toFixed(2)} €</span>
                <span class="discount-tag">-${product.discount}%</span>
            `;
        } else {
            priceElement.innerHTML = `
                <span class="current-price">${product.price.toFixed(2)} €</span>
            `;
        }
        
        // Stock
        const stockElement = clone.querySelector('.product-stock');
        if (product.stock_quantity <= 0) {
            stockElement.innerHTML = '<span class="out-of-stock">Rupture de stock</span>';
        } else if (product.stock_quantity <= 5) {
            stockElement.innerHTML = `<span class="low-stock">Plus que ${product.stock_quantity} en stock</span>`;
        } else {
            stockElement.innerHTML = '<span class="in-stock">En stock</span>';
        }
        
        // Action
        const addToCartBtn = clone.querySelector('.add-to-cart-btn');
        if (product.stock_quantity > 0) {
            addToCartBtn.textContent = 'Ajouter au panier';
            addToCartBtn.onclick = () => mainManager.addToCart(product);
        } else {
            addToCartBtn.textContent = 'Indisponible';
            addToCartBtn.disabled = true;
        }
        
        return clone;
    }

    /**
     * Met à jour la pagination
     */
    updatePagination() {
        const totalPages = Math.ceil(this.filteredProducts.length / this.itemsPerPage);
        const container = document.getElementById('pagination');
        
        if (totalPages <= 1) {
            container.style.display = 'none';
            return;
        }
        
        let html = `
            <button 
                ${this.currentPage === 1 ? 'disabled' : ''}
                onclick="productsManager.changePage(${this.currentPage - 1})"
            >
                Précédent
            </button>
        `;
        
        for (let i = 1; i <= totalPages; i++) {
            if (
                i === 1 || 
                i === totalPages || 
                (i >= this.currentPage - 2 && i <= this.currentPage + 2)
            ) {
                html += `
                    <button 
                        class="${i === this.currentPage ? 'active' : ''}"
                        onclick="productsManager.changePage(${i})"
                    >
                        ${i}
                    </button>
                `;
            } else if (i === this.currentPage - 3 || i === this.currentPage + 3) {
                html += '<span class="pagination-dots">...</span>';
            }
        }
        
        html += `
            <button 
                ${this.currentPage === totalPages ? 'disabled' : ''}
                onclick="productsManager.changePage(${this.currentPage + 1})"
            >
                Suivant
            </button>
        `;
        
        container.innerHTML = html;
        container.style.display = 'flex';
    }

    /**
     * Change de page
     */
    changePage(page) {
        this.currentPage = page;
        this.updateDisplay();
        window.scrollTo({
            top: 0,
            behavior: 'smooth'
        });
    }

    /**
     * Met à jour les filtres actifs
     */
    updateActiveFilters() {
        const container = document.getElementById('active-filters');
        let html = '';
        
        // Prix
        if (this.filters.price.min !== null || this.filters.price.max !== null) {
            html += `
                <div class="filter-tag">
                    Prix : ${this.filters.price.min || 0}€ - ${this.filters.price.max || '∞'}€
                    <button onclick="productsManager.clearPriceFilter()">×</button>
                </div>
            `;
        }
        
        // Marques
        this.filters.brands.forEach(brandId => {
            const brand = document.querySelector(`input[value="${brandId}"]`).parentNode.textContent.trim();
            html += `
                <div class="filter-tag">
                    ${brand}
                    <button onclick="productsManager.removeBrandFilter(${brandId})">×</button>
                </div>
            `;
        });
        
        // Catégories
        this.filters.categories.forEach(categoryId => {
            const category = document.querySelector(`input[value="${categoryId}"]`).parentNode.textContent.trim();
            html += `
                <div class="filter-tag">
                    ${category}
                    <button onclick="productsManager.removeCategoryFilter(${categoryId})">×</button>
                </div>
            `;
        });
        
        // Stock
        if (this.filters.inStock) {
            html += `
                <div class="filter-tag">
                    En stock uniquement
                    <button onclick="productsManager.toggleInStock(false)">×</button>
                </div>
            `;
        }
        
        container.innerHTML = html;
        container.style.display = html ? 'flex' : 'none';
    }

    /**
     * Met à jour l'URL avec les filtres actifs
     */
    updateUrl() {
        const params = new URLSearchParams();
        
        if (this.filters.price.min !== null) {
            params.set('min_price', this.filters.price.min);
        }
if (this.filters.price.max !== null) {
            params.set('max_price', this.filters.price.max);
        }
        
        if (this.filters.brands.length > 0) {
            params.set('brands', this.filters.brands.join(','));
        }
        
        if (this.filters.categories.length > 0) {
            params.set('categories', this.filters.categories.join(','));
        }
        
        if (this.filters.inStock) {
            params.set('in_stock', 'true');
        }
        
        params.set('view', this.currentView);
        params.set('sort', this.currentSort);
        
        // Mise à jour de l'URL sans recharger la page
        window.history.replaceState(
            {}, 
            '', 
            `${window.location.pathname}?${params.toString()}`
        );
    }

    /**
     * Change le mode d'affichage (grille/liste)
     */
    changeView(view) {
        this.currentView = view;
        
        // Mise à jour des boutons
        document.querySelectorAll('.view-btn').forEach(btn => {
            btn.classList.toggle('active', btn.dataset.view === view);
        });
        
        // Mise à jour de la grille
        document.getElementById('products-container').dataset.view = view;
        
        // Mise à jour de l'URL
        this.updateUrl();
    }

    /**
     * Change le tri des produits
     */
    changeSort(sort) {
        this.currentSort = sort;
        this.sortProducts();
        this.updateDisplay();
        this.updateUrl();
    }

    /**
     * Gère les filtres de marque
     */
    handleBrandFilter(checkbox) {
        const brandId = Number(checkbox.value);
        
        if (checkbox.checked) {
            this.filters.brands.push(brandId);
        } else {
            this.filters.brands = this.filters.brands.filter(id => id !== brandId);
        }
        
        this.applyFilters();
    }

    /**
     * Gère les filtres de catégorie
     */
    handleCategoryFilter(checkbox) {
        const categoryId = Number(checkbox.value);
        
        if (checkbox.checked) {
            this.filters.categories.push(categoryId);
        } else {
            this.filters.categories = this.filters.categories.filter(
                id => id !== categoryId
            );
        }
        
        this.applyFilters();
    }

    /**
     * Active/désactive le filtre "en stock"
     */
    toggleInStock(checked) {
        this.filters.inStock = checked;
        document.getElementById('in-stock-only').checked = checked;
        this.applyFilters();
    }

    /**
     * Supprime le filtre de prix
     */
    clearPriceFilter() {
        this.filters.price.min = null;
        this.filters.price.max = null;
        document.getElementById('min-price').value = '';
        document.getElementById('max-price').value = '';
        this.applyFilters();
    }

    /**
     * Supprime un filtre de marque
     */
    removeBrandFilter(brandId) {
        const checkbox = document.querySelector(`input[value="${brandId}"]`);
        if (checkbox) checkbox.checked = false;
        this.filters.brands = this.filters.brands.filter(id => id !== brandId);
        this.applyFilters();
    }

    /**
     * Supprime un filtre de catégorie
     */
    removeCategoryFilter(categoryId) {
        const checkbox = document.querySelector(`input[value="${categoryId}"]`);
        if (checkbox) checkbox.checked = false;
        this.filters.categories = this.filters.categories.filter(
            id => id !== categoryId
        );
        this.applyFilters();
    }

    /**
     * Réinitialise tous les filtres
     */
    clearFilters() {
        // Réinitialisation des états
        this.filters = {
            price: {
                min: null,
                max: null
            },
            brands: [],
            categories: [],
            inStock: false
        };
        
        // Réinitialisation des éléments du DOM
        document.querySelectorAll('input[type="checkbox"]').forEach(
            checkbox => checkbox.checked = false
        );
        document.getElementById('min-price').value = '';
        document.getElementById('max-price').value = '';
        
        // Application des filtres
        this.applyFilters();
    }

    /**
     * Affiche le loader
     */
    showLoader() {
        document.getElementById('products-loader').style.display = 'block';
        document.getElementById('products-container').style.opacity = '0.5';
    }

    /**
     * Cache le loader
     */
    hideLoader() {
        document.getElementById('products-loader').style.display = 'none';
        document.getElementById('products-container').style.opacity = '1';
    }
}

// Initialisation
const productsManager = new ProductsManager();
// Rendre l'instance accessible globalement
window.productsManager = productsManager;

# Partie 3 : Catalogue Frontend
## 3.17 Page Détail Produit

### Description
Cette section comprend :
- L'affichage détaillé d'un produit
- La galerie d'images
- Les spécifications techniques
- Les produits similaires
- Les options d'achat

### 3.17.1 Page Détail Produit
```html
<!-- frontend/pages/product.html -->
<!DOCTYPE html>
<html lang="fr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>FMP - Détail Produit</title>
    <meta name="description" content="Description détaillée de la cartouche d'imprimante.">
    
    <!-- Styles -->
    <link rel="stylesheet" href="../static/css/main.css">
    <link rel="stylesheet" href="../static/css/product-detail.css">
</head>
<body>
    <!-- En-tête (identique aux autres pages) -->
    <header class="main-header">
        <!-- ... contenu de l'en-tête ... -->
    </header>

    <!-- Contenu principal -->
    <main class="product-detail-page">
        <div class="container">
            <!-- Fil d'Ariane -->
            <nav class="breadcrumb">
                <ul>
                    <li><a href="/">Accueil</a></li>
                    <li><a href="/products.html">Cartouches</a></li>
                    <li><a href="#" id="category-link">Catégorie</a></li>
                    <li id="product-name">Nom du produit</li>
                </ul>
            </nav>

            <!-- Section principale -->
            <div class="product-main">
                <!-- Galerie d'images -->
                <div class="product-gallery">
                    <div class="main-image">
                        <img src="" alt="" id="main-product-image">
                        <button class="zoom-btn">
                            <i class="icon zoom-icon"></i>
                        </button>
                    </div>
                    <div class="thumbnail-list" id="image-thumbnails">
                        <!-- Les miniatures seront injectées ici -->
                    </div>
                </div>

                <!-- Informations produit -->
                <div class="product-info">
                    <h1 id="product-title">Nom du produit</h1>
                    
                    <!-- Marque et Référence -->
                    <div class="product-meta">
                        <div class="brand">
                            <span>Marque :</span>
                            <a href="#" id="brand-link">Nom de la marque</a>
                        </div>
                        <div class="reference">
                            <span>Référence :</span>
                            <span id="product-reference">REF123</span>
                        </div>
                    </div>

                    <!-- Prix et stock -->
                    <div class="product-purchase">
                        <div class="price-block">
                            <div class="current-price">
                                <span id="product-price">0.00</span> €
                            </div>
                            <div class="original-price" id="original-price-block" style="display: none;">
                                <span id="original-price">0.00</span> €
                            </div>
                            <div class="discount-tag" id="discount-tag" style="display: none;">
                                -<span id="discount-value">0</span>%
                            </div>
                        </div>

                        <div class="stock-info" id="stock-info">
                            <!-- Le statut du stock sera injecté ici -->
                        </div>

                        <!-- Options d'achat -->
                        <div class="purchase-options">
                            <div class="quantity-selector">
                                <button 
                                    class="quantity-btn minus"
                                    onclick="productDetail.decreaseQuantity()"
                                >-</button>
                                <input 
                                    type="number" 
                                    id="quantity" 
                                    value="1" 
                                    min="1"
                                    onchange="productDetail.validateQuantity()"
                                >
                                <button 
                                    class="quantity-btn plus"
                                    onclick="productDetail.increaseQuantity()"
                                >+</button>
                            </div>

                            <button 
                                class="add-to-cart-btn" 
                                id="add-to-cart-btn"
                                onclick="productDetail.addToCart()"
                            >
                                Ajouter au panier
                            </button>
                        </div>

                        <!-- Avantages -->
                        <div class="product-benefits">
                            <div class="benefit">
                                <i class="icon truck-icon"></i>
                                <span>Livraison gratuite dès 49€</span>
                            </div>
                            <div class="benefit">
                                <i class="icon shield-icon"></i>
                                <span>Garantie 2 ans</span>
                            </div>
                            <div class="benefit">
                                <i class="icon return-icon"></i>
                                <span>Retour gratuit sous 30 jours</span>
                            </div>
                        </div>
                    </div>
                </div>
            </div>

            <!-- Onglets d'information -->
            <div class="product-tabs">
                <div class="tabs-header">
                    <button 
                        class="tab-btn active" 
                        data-tab="description"
                        onclick="productDetail.switchTab('description')"
                    >
                        Description
                    </button>
                    <button 
                        class="tab-btn" 
                        data-tab="specifications"
                        onclick="productDetail.switchTab('specifications')"
                    >
                        Caractéristiques
                    </button>
                    <button 
                        class="tab-btn" 
                        data-tab="compatibility"
                        onclick="productDetail.switchTab('compatibility')"
                    >
                        Compatibilité
                    </button>
                </div>

                <div class="tabs-content">
                    <!-- Description -->
                    <div id="description" class="tab-content active">
                        <div class="description-content">
                            <!-- La description sera injectée ici -->
                        </div>
                    </div>

                    <!-- Caractéristiques -->
                    <div id="specifications" class="tab-content">
                        <table class="specs-table">
                            <tbody id="specs-list">
                                <!-- Les spécifications seront injectées ici -->
                            </tbody>
                        </table>
                    </div>

                    <!-- Compatibilité -->
                    <div id="compatibility" class="tab-content">
                        <div class="compatibility-list" id="compatibility-list">
                            <!-- La liste de compatibilité sera injectée ici -->
                        </div>
                    </div>
                </div>
            </div>

            <!-- Produits similaires -->
            <section class="similar-products">
                <h2>Produits similaires</h2>
                <div class="products-grid" id="similar-products">
                    <!-- Les produits similaires seront injectés ici -->
                </div>
            </section>
        </div>

        <!-- Modal de zoom -->
        <div class="zoom-modal" id="zoom-modal">
            <div class="zoom-content">
                <button class="close-zoom">×</button>
                <img src="" alt="" id="zoomed-image">
                <div class="zoom-thumbnails" id="zoom-thumbnails">
                    <!-- Les miniatures de zoom seront injectées ici -->
                </div>
            </div>
        </div>
    </main>

    <!-- Pied de page (identique aux autres pages) -->
    <footer class="main-footer">
        <!-- ... contenu du pied de page ... -->
    </footer>

    <!-- Scripts -->
    <script type="module" src="../static/js/main.js"></script>
    <script type="module" src="../static/js/product-detail.js"></script>
</body>
</html>
```

Je continue avec :
1. Les styles CSS de la page détail ?
2. Le JavaScript qui gère l'interactivité ?
3. Une autre partie ?