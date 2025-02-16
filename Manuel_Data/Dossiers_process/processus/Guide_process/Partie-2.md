# Guide de Mise en Place du Projet FMP
## Partie 2 : Système d'Authentification

### Introduction
Dans cette partie, nous allons mettre en place le système d'authentification complet de l'application FMP. Ce système permettra aux utilisateurs de s'inscrire, se connecter et gérer leur compte.

### Table des matières
1. [Configuration initiale](#1-configuration-initiale)
2. [Modèle utilisateur](#2-modèle-utilisateur)
3. [Routes d'authentification](#3-routes-dauthentification)
4. [Tests](#4-tests)
5. [Frontend](#5-frontend)

## 1. Configuration initiale

### 1.1 Mise à jour des dépendances
Commençons par installer les packages nécessaires pour l'authentification. Créez ou mettez à jour votre fichier `requirements.txt` :

```txt
# backend/requirements.txt

# Dépendances principales pour l'authentification
Flask-JWT-Extended==4.5.2     # Gestion des JWT (JSON Web Tokens)
Flask-Bcrypt==1.0.1          # Hashage des mots de passe
email-validator==2.0.0.post2  # Validation des emails

# Sécurité supplémentaire
PyJWT==2.8.0                 # Manipulation directe des JWT si nécessaire
```

Pour installer ces dépendances :
```bash
pip install -r backend/requirements.txt
```

### 1.2 Configuration JWT
Configuration des tokens JWT dans `config.py` :

```python
# backend/config.py

import os
from datetime import timedelta

class Config:
    """Configuration de base de l'application Flask."""
    
    # Configuration JWT
    JWT_SECRET_KEY = os.environ.get('JWT_SECRET_KEY') or 'votre-clé-secrète-dev'
    JWT_ACCESS_TOKEN_EXPIRES = timedelta(hours=1)  # Durée de validité du token
    JWT_REFRESH_TOKEN_EXPIRES = timedelta(days=30)  # Durée du refresh token
```

## 2. Modèle utilisateur

Le modèle utilisateur est la pierre angulaire du système d'authentification. Il définit la structure des données utilisateur et les méthodes associées.

### 2.1 Création du modèle User

```python
# backend/app/models/user.py

from datetime import datetime
from app import db
from werkzeug.security import generate_password_hash, check_password_hash

class User(db.Model):
    """Modèle pour les utilisateurs de l'application.
    
    Attributs:
        id (int): Identifiant unique de l'utilisateur
        email (str): Email de l'utilisateur (unique)
        firstname (str): Prénom de l'utilisateur
        lastname (str): Nom de l'utilisateur
        password_hash (str): Hash du mot de passe
        created_at (datetime): Date de création du compte
        is_active (bool): Statut du compte (actif/inactif)
        is_admin (bool): Droits administrateur
    """
    
    __tablename__ = 'users'

    id = db.Column(db.Integer, primary_key=True)
    email = db.Column(db.String(120), unique=True, nullable=False)
    firstname = db.Column(db.String(50), nullable=False)
    lastname = db.Column(db.String(50), nullable=False)
    password_hash = db.Column(db.String(256), nullable=False)
    created_at = db.Column(db.DateTime, default=datetime.utcnow)
    is_active = db.Column(db.Boolean, default=True)
    is_admin = db.Column(db.Boolean, default=False)

    def __init__(self, email, firstname, lastname, password=None):
        """Initialise un nouvel utilisateur.
        
        Args:
            email (str): Email de l'utilisateur
            firstname (str): Prénom de l'utilisateur
            lastname (str): Nom de l'utilisateur
            password (str, optional): Mot de passe en clair
        """
        self.email = email.lower()
        self.firstname = firstname
        self.lastname = lastname
        if password:
            self.set_password(password)

    def set_password(self, password):
        """Hash et stocke le mot de passe."""
        self.password_hash = generate_password_hash(password)

    def check_password(self, password):
        """Vérifie si le mot de passe correspond au hash.
        
        Returns:
            bool: True si le mot de passe correspond, False sinon
        """
        return check_password_hash(self.password_hash, password)

    def to_dict(self):
        """Convertit l'utilisateur en dictionnaire pour l'API.
        
        Returns:
            dict: Données de l'utilisateur
        """
        return {
            'id': self.id,
            'email': self.email,
            'firstname': self.firstname,
            'lastname': self.lastname,
            'created_at': self.created_at.isoformat(),
            'is_admin': self.is_admin
        }
```

## 3. Routes d'authentification

Les routes d'authentification gèrent l'inscription, la connexion et la gestion du compte utilisateur.

### 3.1 Blueprint d'authentification

```python
# backend/app/routes/auth.py

from flask import Blueprint, request, jsonify
from flask_jwt_extended import (
    create_access_token,
    create_refresh_token,
    get_jwt_identity,
    jwt_required
)
from app import db
from app.models.user import User
from email_validator import validate_email, EmailNotValidError

auth_bp = Blueprint('auth', __name__, url_prefix='/api/auth')

@auth_bp.route('/register', methods=['POST'])
def register():
    """Enregistrement d'un nouvel utilisateur.
    
    Attends un JSON avec:
        - email: Email de l'utilisateur
        - password: Mot de passe
        - firstname: Prénom
        - lastname: Nom
    
    Returns:
        JSON: Données utilisateur et token d'accès
    """
    try:
        data = request.get_json()

        # Validation des données
        if not all(k in data for k in ['email', 'password', 'firstname', 'lastname']):
            return jsonify({'error': 'Tous les champs sont requis'}), 400

        # Validation de l'email
        try:
            valid = validate_email(data['email'])
            email = valid.email
        except EmailNotValidError as e:
            return jsonify({'error': str(e)}), 400

        # Vérification si l'email existe déjà
        if User.query.filter_by(email=email).first():
            return jsonify({'error': 'Cet email est déjà utilisé'}), 400

        # Création de l'utilisateur
        user = User(
            email=email,
            firstname=data['firstname'],
            lastname=data['lastname']
        )
        user.set_password(data['password'])

        # Sauvegarde en base de données
        db.session.add(user)
        db.session.commit()

        # Création des tokens
        access_token = create_access_token(identity=user.id)
        refresh_token = create_refresh_token(identity=user.id)

        return jsonify({
            'message': 'Inscription réussie',
            'user': user.to_dict(),
            'access_token': access_token,
            'refresh_token': refresh_token
        }), 201

    except Exception as e:
        db.session.rollback()
        return jsonify({'error': str(e)}), 500
```


### 3.2 Route de Connexion

La route de connexion permet aux utilisateurs de se connecter avec leur email et mot de passe. Voici l'implémentation détaillée :

```python
# backend/app/routes/auth.py (suite)

@auth_bp.route('/login', methods=['POST'])
def login():
    """Route de connexion utilisateur.
    
    Attends un JSON avec:
        - email: Email de l'utilisateur
        - password: Mot de passe
    
    Returns:
        JSON: Données utilisateur et tokens d'accès/rafraîchissement
    """
    try:
        data = request.get_json()

        # Validation des données reçues
        if not data or not data.get('email') or not data.get('password'):
            return jsonify({
                'error': 'Email et mot de passe requis'
            }), 400

        # Recherche de l'utilisateur
        user = User.query.filter_by(email=data['email'].lower()).first()

        # Vérification de l'utilisateur et du mot de passe
        if not user or not user.check_password(data['password']):
            return jsonify({
                'error': 'Email ou mot de passe incorrect'
            }), 401

        # Vérification du statut du compte
        if not user.is_active:
            return jsonify({
                'error': 'Ce compte a été désactivé'
            }), 401

        # Création des tokens
        access_token = create_access_token(identity=user.id)
        refresh_token = create_refresh_token(identity=user.id)

        # Retour de la réponse
        return jsonify({
            'message': 'Connexion réussie',
            'user': user.to_dict(),
            'access_token': access_token,
            'refresh_token': refresh_token
        }), 200

    except Exception as e:
        return jsonify({'error': str(e)}), 500
```

### 3.3 Route de Récupération du Profil

Cette route permet à un utilisateur connecté de récupérer ses informations de profil :

```python
@auth_bp.route('/me', methods=['GET'])
@jwt_required()
def get_profile():
    """Récupère le profil de l'utilisateur connecté.
    
    Nécessite un token JWT valide dans le header Authorization.
    
    Returns:
        JSON: Données du profil utilisateur
    """
    try:
        # Récupération de l'ID utilisateur depuis le token
        current_user_id = get_jwt_identity()
        
        # Recherche de l'utilisateur en base
        user = User.query.get(current_user_id)
        
        if not user:
            return jsonify({
                'error': 'Utilisateur non trouvé'
            }), 404

        # Retour des données utilisateur
        return jsonify(user.to_dict()), 200

    except Exception as e:
        return jsonify({'error': str(e)}), 500
```

### 3.4 Route de Modification du Profil

Cette route permet à un utilisateur de mettre à jour ses informations personnelles :

```python
@auth_bp.route('/me', methods=['PUT'])
@jwt_required()
def update_profile():
    """Met à jour le profil de l'utilisateur connecté.
    
    Attends un JSON avec les champs à modifier:
        - firstname (optionnel): Nouveau prénom
        - lastname (optionnel): Nouveau nom
        - current_password (requis pour changer le mot de passe)
        - new_password (optionnel): Nouveau mot de passe
    
    Returns:
        JSON: Données du profil mises à jour
    """
    try:
        current_user_id = get_jwt_identity()
        user = User.query.get(current_user_id)
        
        if not user:
            return jsonify({
                'error': 'Utilisateur non trouvé'
            }), 404

        data = request.get_json()
        
        # Mise à jour des champs de base
        if 'firstname' in data:
            user.firstname = data['firstname']
        if 'lastname' in data:
            user.lastname = data['lastname']
            
        # Gestion du changement de mot de passe
        if 'new_password' in data:
            # Vérification du mot de passe actuel
            if not data.get('current_password'):
                return jsonify({
                    'error': 'Mot de passe actuel requis'
                }), 400
                
            if not user.check_password(data['current_password']):
                return jsonify({
                    'error': 'Mot de passe actuel incorrect'
                }), 401
                
            # Mise à jour du mot de passe
            user.set_password(data['new_password'])

        # Sauvegarde des modifications
        db.session.commit()

        return jsonify({
            'message': 'Profil mis à jour avec succès',
            'user': user.to_dict()
        }), 200

    except Exception as e:
        db.session.rollback()
        return jsonify({'error': str(e)}), 500
```

### 3.5 Route de Rafraîchissement du Token

Cette route permet de renouveler un token d'accès expiré en utilisant le refresh token :

```python
@auth_bp.route('/refresh', methods=['POST'])
@jwt_required(refresh=True)
def refresh_token():
    """Renouvelle le token d'accès en utilisant le refresh token.
    
    Nécessite un refresh token valide dans le header Authorization.
    
    Returns:
        JSON: Nouveau token d'accès
    """
    try:
        current_user_id = get_jwt_identity()
        
        # Vérification que l'utilisateur existe toujours
        user = User.query.get(current_user_id)
        if not user or not user.is_active:
            return jsonify({
                'error': 'Compte utilisateur invalide'
            }), 401

        # Création d'un nouveau token d'accès
        new_access_token = create_access_token(identity=current_user_id)
        
        return jsonify({
            'access_token': new_access_token
        }), 200

    except Exception as e:
        return jsonify({'error': str(e)}), 500
```

### 3.6 Route de Déconnexion

Cette route permet de gérer la déconnexion de l'utilisateur :

```python
@auth_bp.route('/logout', methods=['POST'])
@jwt_required()
def logout():
    """Gère la déconnexion de l'utilisateur.
    
    Dans une implémentation JWT, la déconnexion se fait côté client
    en supprimant le token, mais cette route peut être utilisée
    pour des actions supplémentaires (logging, invalidation de token, etc.)
    
    Returns:
        JSON: Message de confirmation
    """
    try:
        # Note: Dans une implémentation plus complexe, on pourrait
        # ajouter le token à une liste noire ici
        
        return jsonify({
            'message': 'Déconnexion réussie'
        }), 200

    except Exception as e:
        return jsonify({'error': str(e)}), 500
```
# Intégration des Routes d'Authentification

## 1. Intégration dans l'Application Flask

### 1.1 Modification du fichier __init__.py

Nous allons d'abord modifier le fichier principal de l'application pour intégrer les routes d'authentification.

```python
# backend/app/__init__.py

"""
Ce fichier est le point d'entrée de l'application.
Il initialise Flask et toutes les extensions nécessaires.
"""

from flask import Flask
from flask_sqlalchemy import SQLAlchemy
from flask_jwt_extended import JWTManager
from flask_cors import CORS
from config import config

# Initialisation des extensions
db = SQLAlchemy()
jwt = JWTManager()

def create_app(config_name='default'):
    """
    Crée et configure l'application Flask.
    
    Args:
        config_name (str): Nom de la configuration à utiliser
                          ('development', 'production', 'testing', 'default')
    
    Returns:
        Flask: L'application Flask configurée
    """
    app = Flask(__name__)
    
    # Application de la configuration
    app.config.from_object(config[config_name])
    
    # Initialisation des extensions avec l'application
    db.init_app(app)
    jwt.init_app(app)
    CORS(app)
    
    # Enregistrement des blueprints
    from app.auth import auth_bp
    app.register_blueprint(auth_bp, url_prefix='/api/auth')
    
    # Configuration des gestionnaires d'erreurs JWT
    @jwt.expired_token_loader
    def expired_token_callback(jwt_header, jwt_payload):
        """Gère les tokens expirés."""
        return {
            'status': 401,
            'sub_status': 42,
            'message': 'Le token a expiré'
        }, 401
    
    @jwt.invalid_token_loader
    def invalid_token_callback(error):
        """Gère les tokens invalides."""
        return {
            'status': 401,
            'sub_status': 43,
            'message': 'Token invalide'
        }, 401
    
    @jwt.unauthorized_loader
    def missing_token_callback(error):
        """Gère l'absence de token."""
        return {
            'status': 401,
            'sub_status': 44,
            'message': 'Token manquant'
        }, 401
        
    return app
```

### 1.2 Création du Blueprint d'Authentification

Créez un nouveau fichier pour regrouper toutes les fonctionnalités d'authentification :

```python
# backend/app/auth/__init__.py

"""
Module d'authentification.
Contient toutes les fonctionnalités liées à l'authentification.
"""

from flask import Blueprint

# Création du blueprint
auth_bp = Blueprint('auth', __name__)

# Import des routes (après la création du blueprint pour éviter les imports circulaires)
from . import routes
```

### 1.3 Création du Fichier de Démarrage

Créez le point d'entrée de l'application :

```python
# backend/run.py

"""
Point d'entrée de l'application.
Lance le serveur de développement.
"""

import os
from app import create_app

# Création de l'application
app = create_app(os.getenv('FLASK_CONFIG') or 'default')

if __name__ == '__main__':
    # Configuration du serveur de développement
    debug = os.getenv('FLASK_DEBUG', 'true').lower() == 'true'
    host = os.getenv('FLASK_HOST', '127.0.0.1')
    port = int(os.getenv('FLASK_PORT', 5000))
    
    # Lancement du serveur
    app.run(
        host=host,
        port=port,
        debug=debug
    )
```

## 2. Mise en Place des Tests

### 2.1 Configuration des Tests

Créez un fichier de configuration pour les tests :

```python
# backend/tests/conftest.py

"""
Configuration des tests.
Contient les fixtures pytest utilisées dans les tests.
"""

import pytest
from app import create_app, db
from app.auth.models import User

@pytest.fixture
def app():
    """Crée une instance de l'application pour les tests."""
    app = create_app('testing')
    
    # Création du contexte d'application
    with app.app_context():
        db.create_all()  # Création des tables
        yield app  # Fournit l'application aux tests
        db.session.remove()  # Nettoyage de la session
        db.drop_all()  # Suppression des tables

@pytest.fixture
def client(app):
    """Crée un client de test."""
    return app.test_client()

@pytest.fixture
def test_user(app):
    """Crée un utilisateur de test."""
    user = User(
        email='test@example.com',
        firstname='Test',
        lastname='User'
    )
    user.set_password('password123')
    
    with app.app_context():
        db.session.add(user)
        db.session.commit()
    
    return user
```

## 3. Interface Utilisateur

### 3.1 Page de Connexion (login.html)

Créez le fichier HTML pour la page de connexion :

```html
<!-- frontend/pages/login.html -->
<!DOCTYPE html>
<html lang="fr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>FMP - Connexion</title>
    <!-- Styles -->
    <link rel="stylesheet" href="../static/css/main.css">
    <link rel="stylesheet" href="../static/css/auth.css">
</head>
<body>
    <div class="auth-container">
        <div class="auth-box">
            <h2>Connexion</h2>
            <!-- Formulaire de connexion -->
            <form id="login-form" class="auth-form">
                <!-- Message d'erreur -->
                <div id="error-message" class="error-message" style="display: none;"></div>
                
                <!-- Email -->
                <div class="form-group">
                    <label for="email">Email</label>
                    <input type="email" id="email" name="email" required>
                </div>
                
                <!-- Mot de passe -->
                <div class="form-group">
                    <label for="password">Mot de passe</label>
                    <input type="password" id="password" name="password" required>
                </div>
                
                <!-- Options -->
                <div class="form-options">
                    <label>
                        <input type="checkbox" name="remember">
                        Se souvenir de moi
                    </label>
                    <a href="#" class="forgot-password">Mot de passe oublié ?</a>
                </div>
                
                <!-- Bouton de connexion -->
                <button type="submit" class="auth-button">Se connecter</button>
            </form>
            
            <!-- Liens -->
            <div class="auth-links">
                <p>Pas encore de compte ? <a href="register.html">S'inscrire</a></p>
            </div>
        </div>
    </div>

    <!-- Scripts -->
    <script src="../static/js/auth.js"></script>
</body>
</html>
```
# Styles et JavaScript pour l'Authentification

## 1. Styles CSS
Créez les fichiers de styles suivants :

### 1.1 Styles Principaux (main.css)
```css
/* frontend/static/css/main.css */

/* Reset et styles de base */
* {
    margin: 0;
    padding: 0;
    box-sizing: border-box;
}

body {
    font-family: 'Arial', sans-serif;
    line-height: 1.6;
    background-color: #f5f5f5;
    color: #333;
}

/* Variables CSS globales */
:root {
    --primary-color: #2c3e50;
    --secondary-color: #3498db;
    --error-color: #e74c3c;
    --success-color: #2ecc71;
    --text-color: #333;
    --border-radius: 4px;
    --box-shadow: 0 2px 5px rgba(0,0,0,0.1);
}
```

### 1.2 Styles d'Authentification (auth.css)
```css
/* frontend/static/css/auth.css */

/* Conteneur principal */
.auth-container {
    min-height: 100vh;
    display: flex;
    justify-content: center;
    align-items: center;
    padding: 20px;
    background: linear-gradient(135deg, #f5f7fa 0%, #c3cfe2 100%);
}

/* Boîte d'authentification */
.auth-box {
    background: white;
    padding: 2rem;
    border-radius: 8px;
    box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
    width: 100%;
    max-width: 400px;
}

.auth-box h2 {
    color: var(--primary-color);
    text-align: center;
    margin-bottom: 1.5rem;
    font-size: 1.8rem;
}

/* Formulaire */
.auth-form {
    display: flex;
    flex-direction: column;
    gap: 1rem;
}

.form-group {
    display: flex;
    flex-direction: column;
    gap: 0.5rem;
}

.form-group label {
    color: var(--text-color);
    font-size: 0.9rem;
    font-weight: 500;
}

.form-group input {
    padding: 0.8rem;
    border: 1px solid #ddd;
    border-radius: var(--border-radius);
    font-size: 1rem;
    transition: border-color 0.3s, box-shadow 0.3s;
}

.form-group input:focus {
    outline: none;
    border-color: var(--secondary-color);
    box-shadow: 0 0 0 2px rgba(52, 152, 219, 0.2);
}

/* Options du formulaire */
.form-options {
    display: flex;
    justify-content: space-between;
    align-items: center;
    font-size: 0.9rem;
}

.form-options label {
    display: flex;
    align-items: center;
    gap: 0.5rem;
}

.forgot-password {
    color: var(--secondary-color);
    text-decoration: none;
}

.forgot-password:hover {
    text-decoration: underline;
}

/* Bouton de soumission */
.auth-button {
    background: var(--secondary-color);
    color: white;
    padding: 0.8rem;
    border: none;
    border-radius: var(--border-radius);
    font-size: 1rem;
    cursor: pointer;
    transition: background 0.3s;
}

.auth-button:hover {
    background: #2980b9;
}

/* Liens supplémentaires */
.auth-links {
    margin-top: 1.5rem;
    text-align: center;
    font-size: 0.9rem;
}

.auth-links a {
    color: var(--secondary-color);
    text-decoration: none;
}

.auth-links a:hover {
    text-decoration: underline;
}

/* Messages d'erreur */
.error-message {
    background: #fde8e8;
    color: var(--error-color);
    padding: 0.8rem;
    border-radius: var(--border-radius);
    margin-bottom: 1rem;
    font-size: 0.9rem;
    display: none;
}

/* Responsive design */
@media (max-width: 480px) {
    .auth-box {
        padding: 1.5rem;
    }

    .form-options {
        flex-direction: column;
        gap: 0.5rem;
        align-items: flex-start;
    }
}
```

## 2. JavaScript pour la Gestion du Formulaire

### 2.1 JavaScript d'Authentification (auth.js)
```javascript
// frontend/static/js/auth.js

/**
 * Gestion du formulaire de connexion
 */
document.addEventListener('DOMContentLoaded', function() {
    const loginForm = document.getElementById('login-form');
    const errorMessage = document.getElementById('error-message');
    
    if (loginForm) {
        loginForm.addEventListener('submit', async function(e) {
            e.preventDefault();
            
            try {
                // Récupération des données du formulaire
                const formData = {
                    email: this.email.value.trim(),
                    password: this.password.value
                };
                
                // Validation basique
                if (!validateForm(formData)) {
                    return;
                }
                
                // Appel à l'API
                const response = await fetch('/api/auth/login', {
                    method: 'POST',
                    headers: {
                        'Content-Type': 'application/json'
                    },
                    body: JSON.stringify(formData)
                });
                
                const data = await response.json();
                
                if (!response.ok) {
                    throw new Error(data.error || 'Erreur de connexion');
                }
                
                // Stockage du token
                localStorage.setItem('token', data.token);
                localStorage.setItem('user', JSON.stringify(data.user));
                
                // Redirection
                window.location.href = '/';
                
            } catch (error) {
                showError(error.message);
            }
        });
    }
});

/**
 * Valide les données du formulaire
 * @param {Object} formData - Données du formulaire
 * @returns {boolean} - Validité du formulaire
 */
function validateForm(formData) {
    // Validation de l'email
    if (!formData.email) {
        showError('L\'email est requis');
        return false;
    }
    
    if (!isValidEmail(formData.email)) {
        showError('Email invalide');
        return false;
    }
    
    // Validation du mot de passe
    if (!formData.password) {
        showError('Le mot de passe est requis');
        return false;
    }
    
    return true;
}

/**
 * Valide le format de l'email
 * @param {string} email - Email à valider
 * @returns {boolean} - Validité de l'email
 */
function isValidEmail(email) {
    const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
    return emailRegex.test(email);
}

/**
 * Affiche un message d'erreur
 * @param {string} message - Message d'erreur à afficher
 */
function showError(message) {
    const errorDiv = document.getElementById('error-message');
    errorDiv.textContent = message;
    errorDiv.style.display = 'block';
    
    // Cache le message après 3 secondes
    setTimeout(() => {
        errorDiv.style.display = 'none';
    }, 3000);
}
```
# Interface Utilisateur - Page d'Inscription
## 3.4 Page d'Inscription

### Description
La page d'inscription permet aux nouveaux utilisateurs de créer un compte. Elle inclut :
- Un formulaire avec validation
- Des messages d'erreur en temps réel
- Une vérification de la force du mot de passe
- Des liens vers la page de connexion

### 3.4.1 HTML - Formulaire d'Inscription
```html
<!-- frontend/pages/register.html -->
<!DOCTYPE html>
<html lang="fr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>FMP - Inscription</title>
    <!-- Import des styles -->
    <link rel="stylesheet" href="../static/css/main.css">
    <link rel="stylesheet" href="../static/css/auth.css">
    <link rel="stylesheet" href="../static/css/register.css">
</head>
<body>
    <div class="auth-container">
        <div class="auth-box register-box">
            <!-- En-tête -->
            <h2>Créer un compte</h2>
            <p class="auth-subtitle">Rejoignez FMP pour accéder à nos services</p>

            <!-- Formulaire d'inscription -->
            <form id="register-form" class="auth-form">
                <!-- Message d'erreur -->
                <div id="error-message" class="error-message" style="display: none;"></div>

                <!-- Nom et Prénom sur la même ligne -->
                <div class="form-row">
                    <!-- Prénom -->
                    <div class="form-group">
                        <label for="firstname">Prénom</label>
                        <input 
                            type="text" 
                            id="firstname" 
                            name="firstname" 
                            required
                            autocomplete="given-name">
                        <small class="form-hint">2 caractères minimum</small>
                    </div>

                    <!-- Nom -->
                    <div class="form-group">
                        <label for="lastname">Nom</label>
                        <input 
                            type="text" 
                            id="lastname" 
                            name="lastname" 
                            required
                            autocomplete="family-name">
                        <small class="form-hint">2 caractères minimum</small>
                    </div>
                </div>

                <!-- Email -->
                <div class="form-group">
                    <label for="email">Email</label>
                    <input 
                        type="email" 
                        id="email" 
                        name="email" 
                        required
                        autocomplete="email">
                    <small class="form-hint">Sera utilisé pour la connexion</small>
                </div>

                <!-- Mot de passe -->
                <div class="form-group">
                    <label for="password">Mot de passe</label>
                    <div class="password-input-group">
                        <input 
                            type="password" 
                            id="password" 
                            name="password" 
                            required>
                        <button type="button" class="toggle-password">
                            <span class="eye-icon">👁️</span>
                        </button>
                    </div>
                    <!-- Indicateur de force du mot de passe -->
                    <div class="password-strength">
                        <div class="strength-meter">
                            <div class="meter-section"></div>
                            <div class="meter-section"></div>
                            <div class="meter-section"></div>
                            <div class="meter-section"></div>
                        </div>
                        <span class="strength-text">Force du mot de passe</span>
                    </div>
                </div>

                <!-- Confirmation du mot de passe -->
                <div class="form-group">
                    <label for="password-confirm">Confirmez le mot de passe</label>
                    <input 
                        type="password" 
                        id="password-confirm" 
                        name="password-confirm" 
                        required>
                </div>

                <!-- Conditions d'utilisation -->
                <div class="form-group terms">
                    <label class="checkbox-label">
                        <input 
                            type="checkbox" 
                            name="terms" 
                            required>
                        <span>J'accepte les <a href="/terms.html">conditions d'utilisation</a></span>
                    </label>
                </div>

                <!-- Newsletter -->
                <div class="form-group terms">
                    <label class="checkbox-label">
                        <input 
                            type="checkbox" 
                            name="newsletter">
                        <span>Je souhaite recevoir la newsletter</span>
                    </label>
                </div>

                <!-- Bouton d'inscription -->
                <button type="submit" class="auth-button">
                    Créer mon compte
                </button>
            </form>

            <!-- Liens -->
            <div class="auth-links">
                <p>Déjà un compte ? <a href="login.html">Se connecter</a></p>
            </div>
        </div>
    </div>

    <!-- Scripts -->
    <script src="../static/js/register.js"></script>
</body>
</html>
```

### 3.4.2 CSS - Styles spécifiques à l'inscription
```css
/* frontend/static/css/register.css */

/* Styles spécifiques à la page d'inscription */
.register-box {
    max-width: 500px; /* Plus large que la page de connexion */
}

/* Style pour la mise en page sur deux colonnes */
.form-row {
    display: grid;
    grid-template-columns: 1fr 1fr;
    gap: 1rem;
}

/* Groupe de mot de passe avec icône */
.password-input-group {
    position: relative;
    display: flex;
    align-items: center;
}

.toggle-password {
    position: absolute;
    right: 10px;
    background: none;
    border: none;
    cursor: pointer;
    padding: 5px;
}

/* Indicateur de force du mot de passe */
.password-strength {
    margin-top: 0.5rem;
}

.strength-meter {
    display: flex;
    gap: 5px;
    margin-bottom: 5px;
}

.meter-section {
    height: 4px;
    flex: 1;
    background: #ddd;
    border-radius: 2px;
    transition: background-color 0.3s;
}

/* Classes de force du mot de passe */
.weak .meter-section:nth-child(1) {
    background: #e74c3c;
}

.medium .meter-section:nth-child(1),
.medium .meter-section:nth-child(2) {
    background: #f39c12;
}

.strong .meter-section:nth-child(1),
.strong .meter-section:nth-child(2),
.strong .meter-section:nth-child(3) {
    background: #3498db;
}

.very-strong .meter-section {
    background: #2ecc71;
}

.strength-text {
    font-size: 0.8rem;
    color: #666;
}

/* Style des cases à cocher */
.checkbox-label {
    display: flex;
    align-items: flex-start;
    gap: 0.5rem;
    font-size: 0.9rem;
}

.checkbox-label input[type="checkbox"] {
    margin-top: 3px;
}

/* Messages d'aide */
.form-hint {
    font-size: 0.8rem;
    color: #666;
    margin-top: 0.25rem;
}

/* Responsive */
@media (max-width: 600px) {
    .form-row {
        grid-template-columns: 1fr;
    }

    .register-box {
        padding: 1.5rem;
    }
}
```
# Interface Utilisateur - JavaScript d'Inscription
## 3.4.3 JavaScript - Gestion du Formulaire d'Inscription

### Description
Le script gère :
- La validation en temps réel des champs
- L'évaluation de la force du mot de passe
- L'affichage/masquage du mot de passe
- La soumission du formulaire
- La gestion des erreurs

```javascript
// frontend/static/js/register.js

/**
 * Gestionnaire du formulaire d'inscription
 */
document.addEventListener('DOMContentLoaded', function() {
    // Récupération des éléments du DOM
    const registerForm = document.getElementById('register-form');
    const passwordInput = document.getElementById('password');
    const passwordConfirmInput = document.getElementById('password-confirm');
    const togglePasswordBtn = document.querySelector('.toggle-password');
    const strengthMeter = document.querySelector('.strength-meter');
    const strengthText = document.querySelector('.strength-text');

    // Gestion de l'affichage/masquage du mot de passe
    if (togglePasswordBtn) {
        togglePasswordBtn.addEventListener('click', function() {
            const type = passwordInput.type === 'password' ? 'text' : 'password';
            passwordInput.type = type;
            this.querySelector('.eye-icon').textContent = type === 'password' ? '👁️' : '👁️‍🗨️';
        });
    }

    // Validation en temps réel du mot de passe
    if (passwordInput) {
        passwordInput.addEventListener('input', function() {
            const strength = checkPasswordStrength(this.value);
            updatePasswordStrengthIndicator(strength);
        });
    }

    // Validation en temps réel de la confirmation du mot de passe
    if (passwordConfirmInput) {
        passwordConfirmInput.addEventListener('input', function() {
            validatePasswordMatch(passwordInput.value, this.value);
        });
    }

    // Gestion de la soumission du formulaire
    if (registerForm) {
        registerForm.addEventListener('submit', async function(e) {
            e.preventDefault();
            
            // Validation finale avant envoi
            if (!validateForm(this)) {
                return;
            }

            try {
                const formData = {
                    firstname: this.firstname.value.trim(),
                    lastname: this.lastname.value.trim(),
                    email: this.email.value.trim(),
                    password: this.password.value,
                    newsletter: this.newsletter.checked
                };

                const response = await fetch('/api/auth/register', {
                    method: 'POST',
                    headers: {
                        'Content-Type': 'application/json'
                    },
                    body: JSON.stringify(formData)
                });

                const data = await response.json();

                if (!response.ok) {
                    throw new Error(data.error || 'Erreur lors de l\'inscription');
                }

                // Stockage du token et redirection
                localStorage.setItem('token', data.token);
                localStorage.setItem('user', JSON.stringify(data.user));
                
                // Redirection vers la page d'accueil
                window.location.href = '/';

            } catch (error) {
                showError(error.message);
            }
        });
    }
});

/**
 * Vérifie la force du mot de passe
 * @param {string} password - Mot de passe à vérifier
 * @returns {Object} - Score et feedback sur la force du mot de passe
 */
function checkPasswordStrength(password) {
    let score = 0;
    let feedback = [];

    // Longueur minimale
    if (password.length >= 8) {
        score++;
    } else {
        feedback.push('8 caractères minimum');
    }

    // Présence de majuscules
    if (/[A-Z]/.test(password)) {
        score++;
    } else {
        feedback.push('Au moins 1 majuscule');
    }

    // Présence de minuscules
    if (/[a-z]/.test(password)) {
        score++;
    } else {
        feedback.push('Au moins 1 minuscule');
    }

    // Présence de chiffres
    if (/\d/.test(password)) {
        score++;
    } else {
        feedback.push('Au moins 1 chiffre');
    }

    // Présence de caractères spéciaux
    if (/[^A-Za-z0-9]/.test(password)) {
        score++;
    } else {
        feedback.push('Au moins 1 caractère spécial');
    }

    return {
        score: score,
        feedback: feedback
    };
}

/**
 * Met à jour l'indicateur de force du mot de passe
 * @param {Object} strength - Résultat de checkPasswordStrength
 */
function updatePasswordStrengthIndicator(strength) {
    const strengthMeter = document.querySelector('.strength-meter');
    const strengthText = document.querySelector('.strength-text');

    // Suppression des classes existantes
    strengthMeter.className = 'strength-meter';

    // Ajout de la nouvelle classe selon le score
    if (strength.score === 0) {
        strengthMeter.classList.add('very-weak');
        strengthText.textContent = 'Très faible';
    } else if (strength.score <= 2) {
        strengthMeter.classList.add('weak');
        strengthText.textContent = 'Faible';
    } else if (strength.score <= 3) {
        strengthMeter.classList.add('medium');
        strengthText.textContent = 'Moyen';
    } else if (strength.score <= 4) {
        strengthMeter.classList.add('strong');
        strengthText.textContent = 'Fort';
    } else {
        strengthMeter.classList.add('very-strong');
        strengthText.textContent = 'Très fort';
    }

    // Affichage du feedback si nécessaire
    if (strength.feedback.length > 0) {
        strengthText.textContent += ` (${strength.feedback.join(', ')})`;
    }
}

/**
 * Valide la correspondance des mots de passe
 * @param {string} password - Mot de passe original
 * @param {string} confirmation - Confirmation du mot de passe
 */
function validatePasswordMatch(password, confirmation) {
    const confirmInput = document.getElementById('password-confirm');
    const matchFeedback = document.querySelector('.password-match-feedback');

    if (password !== confirmation) {
        confirmInput.classList.add('error');
        if (!matchFeedback) {
            const feedback = document.createElement('small');
            feedback.className = 'password-match-feedback error-text';
            feedback.textContent = 'Les mots de passe ne correspondent pas';
            confirmInput.parentNode.appendChild(feedback);
        }
    } else {
        confirmInput.classList.remove('error');
        if (matchFeedback) {
            matchFeedback.remove();
        }
    }
}

/**
 * Valide l'ensemble du formulaire
 * @param {HTMLFormElement} form - Formulaire à valider
 * @returns {boolean} - True si le formulaire est valide
 */
function validateForm(form) {
    // Validation du prénom et nom
    if (form.firstname.value.trim().length < 2) {
        showError('Le prénom doit contenir au moins 2 caractères');
        return false;
    }

    if (form.lastname.value.trim().length < 2) {
        showError('Le nom doit contenir au moins 2 caractères');
        return false;
    }

    // Validation de l'email
    const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
    if (!emailRegex.test(form.email.value.trim())) {
        showError('Adresse email invalide');
        return false;
    }

    // Validation du mot de passe
    const passwordStrength = checkPasswordStrength(form.password.value);
    if (passwordStrength.score < 3) {
        showError('Le mot de passe est trop faible');
        return false;
    }

    // Validation de la confirmation du mot de passe
    if (form.password.value !== form['password-confirm'].value) {
        showError('Les mots de passe ne correspondent pas');
        return false;
    }

    // Validation des conditions d'utilisation
    if (!form.terms.checked) {
        showError('Vous devez accepter les conditions d\'utilisation');
        return false;
    }

    return true;
}

/**
 * Affiche un message d'erreur
 * @param {string} message - Message d'erreur à afficher
 */
function showError(message) {
    const errorDiv = document.getElementById('error-message');
    errorDiv.textContent = message;
    errorDiv.style.display = 'block';

    // Masquage automatique après 3 secondes
    setTimeout(() => {
        errorDiv.style.display = 'none';
    }, 3000);
}
```
# Interface Utilisateur - Page de Profil
## 3.5 Page de Profil Utilisateur

### Description
La page de profil permet à l'utilisateur de :
- Voir ses informations personnelles
- Modifier ses données
- Changer son mot de passe
- Gérer ses préférences
- Voir son historique de commandes

### 3.5.1 HTML - Page de Profil
```html
<!-- frontend/pages/profile.html -->
<!DOCTYPE html>
<html lang="fr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>FMP - Mon Profil</title>
    <!-- Import des styles -->
    <link rel="stylesheet" href="../static/css/main.css">
    <link rel="stylesheet" href="../static/css/profile.css">
</head>
<body>
    <!-- En-tête avec navigation -->
    <header class="main-header">
        <nav class="main-nav">
            <div class="logo">
                <a href="/">FMP</a>
            </div>
            <div class="nav-links">
                <a href="/products.html">Produits</a>
                <a href="/cart.html">Panier</a>
                <a href="/profile.html" class="active">Mon Compte</a>
            </div>
        </nav>
    </header>

    <main class="profile-container">
        <!-- Menu latéral -->
        <aside class="profile-sidebar">
            <nav class="profile-nav">
                <ul>
                    <li>
                        <a href="#profile" class="active" data-tab="profile">
                            Informations personnelles
                        </a>
                    </li>
                    <li>
                        <a href="#orders" data-tab="orders">
                            Mes commandes
                        </a>
                    </li>
                    <li>
                        <a href="#security" data-tab="security">
                            Sécurité
                        </a>
                    </li>
                    <li>
                        <a href="#preferences" data-tab="preferences">
                            Préférences
                        </a>
                    </li>
                </ul>
            </nav>
        </aside>

        <!-- Contenu principal -->
        <div class="profile-content">
            <!-- Section Informations Personnelles -->
            <section id="profile" class="profile-section active">
                <h2>Informations personnelles</h2>
                <form id="profile-form" class="profile-form">
                    <div class="form-row">
                        <div class="form-group">
                            <label for="firstname">Prénom</label>
                            <input type="text" id="firstname" name="firstname">
                        </div>
                        <div class="form-group">
                            <label for="lastname">Nom</label>
                            <input type="text" id="lastname" name="lastname">
                        </div>
                    </div>

                    <div class="form-group">
                        <label for="email">Email</label>
                        <input type="email" id="email" name="email">
                    </div>

                    <div class="form-group">
                        <label for="phone">Téléphone</label>
                        <input type="tel" id="phone" name="phone">
                    </div>

                    <button type="submit" class="save-button">
                        Enregistrer les modifications
                    </button>
                </form>
            </section>

            <!-- Section Commandes -->
            <section id="orders" class="profile-section">
                <h2>Mes commandes</h2>
                <div class="orders-list">
                    <!-- Les commandes seront injectées ici via JavaScript -->
                    <div class="no-orders-message" style="display: none;">
                        Vous n'avez pas encore de commande.
                    </div>
                </div>
            </section>

            <!-- Section Sécurité -->
            <section id="security" class="profile-section">
                <h2>Sécurité</h2>
                <form id="password-form" class="profile-form">
                    <div class="form-group">
                        <label for="current-password">Mot de passe actuel</label>
                        <input type="password" id="current-password" name="current-password">
                    </div>

                    <div class="form-group">
                        <label for="new-password">Nouveau mot de passe</label>
                        <input type="password" id="new-password" name="new-password">
                        <div class="password-strength">
                            <div class="strength-meter">
                                <div class="meter-section"></div>
                                <div class="meter-section"></div>
                                <div class="meter-section"></div>
                                <div class="meter-section"></div>
                            </div>
                        </div>
                    </div>

                    <div class="form-group">
                        <label for="confirm-password">Confirmer le mot de passe</label>
                        <input type="password" id="confirm-password" name="confirm-password">
                    </div>

                    <button type="submit" class="save-button">
                        Changer le mot de passe
                    </button>
                </form>
            </section>

            <!-- Section Préférences -->
            <section id="preferences" class="profile-section">
                <h2>Préférences</h2>
                <form id="preferences-form" class="profile-form">
                    <div class="form-group">
                        <label class="checkbox-label">
                            <input type="checkbox" name="newsletter">
                            Recevoir la newsletter
                        </label>
                    </div>

                    <div class="form-group">
                        <label class="checkbox-label">
                            <input type="checkbox" name="order_updates">
                            Notifications de commande par email
                        </label>
                    </div>

                    <div class="form-group">
                        <label class="checkbox-label">
                            <input type="checkbox" name="promo_updates">
                            Recevoir les offres promotionnelles
                        </label>
                    </div>

                    <button type="submit" class="save-button">
                        Enregistrer les préférences
                    </button>
                </form>
            </section>
        </div>
    </main>

    <!-- Scripts -->
    <script src="../static/js/profile.js"></script>
</body>
</html>
```

### 3.5.2 CSS - Styles de la page profil
```css
/* frontend/static/css/profile.css */

/* Layout principal */
.profile-container {
    display: grid;
    grid-template-columns: 250px 1fr;
    gap: 2rem;
    max-width: 1200px;
    margin: 2rem auto;
    padding: 0 1rem;
}

/* Barre latérale */
.profile-sidebar {
    background: white;
    border-radius: 8px;
    box-shadow: 0 2px 4px rgba(0,0,0,0.1);
}

.profile-nav ul {
    list-style: none;
    padding: 0;
}

.profile-nav a {
    display: block;
    padding: 1rem;
    color: #333;
    text-decoration: none;
    transition: background-color 0.3s, color 0.3s;
}

.profile-nav a:hover,
.profile-nav a.active {
    background-color: #f0f7ff;
    color: var(--primary-color);
    border-right: 3px solid var(--primary-color);
}

/* Contenu principal */
.profile-content {
    background: white;
    border-radius: 8px;
    box-shadow: 0 2px 4px rgba(0,0,0,0.1);
    padding: 2rem;
}

/* Sections */
.profile-section {
    display: none;
}

.profile-section.active {
    display: block;
}

.profile-section h2 {
    margin-bottom: 1.5rem;
    color: var(--primary-color);
}

/* Formulaires */
.profile-form {
    max-width: 600px;
}

.form-row {
    display: grid;
    grid-template-columns: 1fr 1fr;
    gap: 1rem;
    margin-bottom: 1rem;
}

.form-group {
    margin-bottom: 1.5rem;
}

.form-group label {
    display: block;
    margin-bottom: 0.5rem;
    color: #555;
}

.form-group input[type="text"],
.form-group input[type="email"],
.form-group input[type="password"],
.form-group input[type="tel"] {
    width: 100%;
    padding: 0.8rem;
    border: 1px solid #ddd;
    border-radius: 4px;
    transition: border-color 0.3s, box-shadow 0.3s;
}

.form-group input:focus {
    border-color: var(--primary-color);
    box-shadow: 0 0 0 2px rgba(52, 152, 219, 0.2);
    outline: none;
}

/* Boutons */
.save-button {
    background: var(--primary-color);
    color: white;
    border: none;
    padding: 0.8rem 1.5rem;
    border-radius: 4px;
    cursor: pointer;
    transition: background-color 0.3s;
}

.save-button:hover {
    background: #2980b9;
}

/* Liste des commandes */
.orders-list {
    display: grid;
    gap: 1rem;
}

.order-card {
    border: 1px solid #ddd;
    border-radius: 4px;
    padding: 1rem;
}

.order-header {
    display: flex;
    justify-content: space-between;
    margin-bottom: 1rem;
}

.order-status {
    padding: 0.25rem 0.5rem;
    border-radius: 4px;
    font-size: 0.9rem;
}

.status-pending {
    background: #fff3cd;
    color: #856404;
}

.status-shipped {
    background: #d4edda;
    color: #155724;
}

/* Responsive */
@media (max-width: 768px) {
    .profile-container {
        grid-template-columns: 1fr;
    }

    .profile-sidebar {
        margin-bottom: 1rem;
    }

    .profile-nav ul {
        display: flex;
        overflow-x: auto;
    }

    .profile-nav a {
        white-space: nowrap;
    }

    .form-row {
        grid-template-columns: 1fr;
    }
}
```
# Interface Utilisateur - JavaScript de la Page Profil
## 3.5.3 JavaScript - Gestion du Profil Utilisateur

### Description
Ce script gère :
- La navigation entre les onglets
- Le chargement des données utilisateur
- La mise à jour du profil
- Le changement de mot de passe
- L'historique des commandes
- Les préférences utilisateur

```javascript
// frontend/static/js/profile.js

/**
 * Gestion de la page profil utilisateur
 */
document.addEventListener('DOMContentLoaded', function() {
    // Initialisation
    initializeTabs();
    loadUserProfile();
    setupFormListeners();
    loadOrderHistory();

    // Vérification de l'authentification
    if (!isAuthenticated()) {
        window.location.href = '/login.html';
        return;
    }
});

/**
 * Vérifie si l'utilisateur est authentifié
 * @returns {boolean} Statut d'authentification
 */
function isAuthenticated() {
    const token = localStorage.getItem('token');
    return !!token;
}

/**
 * Initialise la navigation par onglets
 */
function initializeTabs() {
    const tabLinks = document.querySelectorAll('.profile-nav a');
    const sections = document.querySelectorAll('.profile-section');

    tabLinks.forEach(link => {
        link.addEventListener('click', (e) => {
            e.preventDefault();
            
            // Retrait des classes actives
            tabLinks.forEach(l => l.classList.remove('active'));
            sections.forEach(s => s.classList.remove('active'));
            
            // Activation de l'onglet cliqué
            link.classList.add('active');
            const targetId = link.getAttribute('data-tab');
            document.getElementById(targetId).classList.add('active');
            
            // Mise à jour de l'URL
            history.pushState(null, '', `#${targetId}`);
        });
    });

    // Gestion du retour navigateur
    window.addEventListener('popstate', () => {
        const hash = window.location.hash.slice(1) || 'profile';
        const targetLink = document.querySelector(`[data-tab="${hash}"]`);
        if (targetLink) {
            targetLink.click();
        }
    });

    // Activation de l'onglet initial
    const initialHash = window.location.hash.slice(1) || 'profile';
    const initialLink = document.querySelector(`[data-tab="${initialHash}"]`);
    if (initialLink) {
        initialLink.click();
    }
}

/**
 * Charge les données du profil utilisateur
 */
async function loadUserProfile() {
    try {
        const response = await fetch('/api/auth/me', {
            headers: {
                'Authorization': `Bearer ${localStorage.getItem('token')}`
            }
        });

        if (!response.ok) {
            throw new Error('Erreur de chargement du profil');
        }

        const userData = await response.json();
        
        // Remplissage du formulaire
        document.getElementById('firstname').value = userData.firstname;
        document.getElementById('lastname').value = userData.lastname;
        document.getElementById('email').value = userData.email;
        document.getElementById('phone').value = userData.phone || '';

        // Remplissage des préférences
        if (userData.preferences) {
            document.querySelector('[name="newsletter"]').checked = userData.preferences.newsletter;
            document.querySelector('[name="order_updates"]').checked = userData.preferences.order_updates;
            document.querySelector('[name="promo_updates"]').checked = userData.preferences.promo_updates;
        }

    } catch (error) {
        showError('Erreur lors du chargement du profil');
        console.error(error);
    }
}

/**
 * Configure les écouteurs d'événements des formulaires
 */
function setupFormListeners() {
    // Formulaire de profil
    const profileForm = document.getElementById('profile-form');
    if (profileForm) {
        profileForm.addEventListener('submit', handleProfileUpdate);
    }

    // Formulaire de mot de passe
    const passwordForm = document.getElementById('password-form');
    if (passwordForm) {
        passwordForm.addEventListener('submit', handlePasswordChange);
    }

    // Formulaire de préférences
    const preferencesForm = document.getElementById('preferences-form');
    if (preferencesForm) {
        preferencesForm.addEventListener('submit', handlePreferencesUpdate);
    }
}

/**
 * Gère la mise à jour du profil
 * @param {Event} e - Événement de soumission
 */
async function handleProfileUpdate(e) {
    e.preventDefault();

    try {
        const formData = {
            firstname: document.getElementById('firstname').value.trim(),
            lastname: document.getElementById('lastname').value.trim(),
            email: document.getElementById('email').value.trim(),
            phone: document.getElementById('phone').value.trim()
        };

        const response = await fetch('/api/auth/profile', {
            method: 'PUT',
            headers: {
                'Content-Type': 'application/json',
                'Authorization': `Bearer ${localStorage.getItem('token')}`
            },
            body: JSON.stringify(formData)
        });

        if (!response.ok) {
            throw new Error('Erreur lors de la mise à jour du profil');
        }

        showSuccess('Profil mis à jour avec succès');

    } catch (error) {
        showError(error.message);
    }
}

/**
 * Gère le changement de mot de passe
 * @param {Event} e - Événement de soumission
 */
async function handlePasswordChange(e) {
    e.preventDefault();

    const currentPassword = document.getElementById('current-password').value;
    const newPassword = document.getElementById('new-password').value;
    const confirmPassword = document.getElementById('confirm-password').value;

    // Validation basique
    if (newPassword !== confirmPassword) {
        showError('Les mots de passe ne correspondent pas');
        return;
    }

    try {
        const response = await fetch('/api/auth/password', {
            method: 'PUT',
            headers: {
                'Content-Type': 'application/json',
                'Authorization': `Bearer ${localStorage.getItem('token')}`
            },
            body: JSON.stringify({
                current_password: currentPassword,
                new_password: newPassword
            })
        });

        if (!response.ok) {
            const data = await response.json();
            throw new Error(data.error || 'Erreur lors du changement de mot de passe');
        }

        // Réinitialisation du formulaire
        e.target.reset();
        showSuccess('Mot de passe modifié avec succès');

    } catch (error) {
        showError(error.message);
    }
}

/**
 * Charge l'historique des commandes
 */
async function loadOrderHistory() {
    try {
        const response = await fetch('/api/orders/history', {
            headers: {
                'Authorization': `Bearer ${localStorage.getItem('token')}`
            }
        });

        if (!response.ok) {
            throw new Error('Erreur de chargement des commandes');
        }

        const orders = await response.json();
        
        const ordersList = document.querySelector('.orders-list');
        const noOrdersMessage = document.querySelector('.no-orders-message');

        if (orders.length === 0) {
            noOrdersMessage.style.display = 'block';
            return;
        }

        // Génération des cartes de commande
        ordersList.innerHTML = orders.map(order => `
            <div class="order-card">
                <div class="order-header">
                    <div class="order-info">
                        <strong>Commande #${order.id}</strong>
                        <span>${new Date(order.created_at).toLocaleDateString()}</span>
                    </div>
                    <span class="order-status status-${order.status.toLowerCase()}">
                        ${getStatusLabel(order.status)}
                    </span>
                </div>
                <div class="order-total">
                    Total: ${order.total.toFixed(2)} €
                </div>
            </div>
        `).join('');

    } catch (error) {
        showError('Erreur lors du chargement des commandes');
        console.error(error);
    }
}

/**
 * Obtient le libellé d'un statut de commande
 * @param {string} status - Code du statut
 * @returns {string} Libellé du statut
 */
function getStatusLabel(status) {
    const labels = {
        'PENDING': 'En attente',
        'PROCESSING': 'En traitement',
        'SHIPPED': 'Expédiée',
        'DELIVERED': 'Livrée',
        'CANCELLED': 'Annulée'
    };
    return labels[status] || status;
}

/**
 * Affiche un message de succès
 * @param {string} message - Message à afficher
 */
function showSuccess(message) {
    const successDiv = document.createElement('div');
    successDiv.className = 'success-message';
    successDiv.textContent = message;
    
    document.querySelector('.profile-content').prepend(successDiv);
    
    setTimeout(() => {
        successDiv.remove();
    }, 3000);
}

/**
 * Affiche un message d'erreur
 * @param {string} message - Message d'erreur
 */
function showError(message) {
    const errorDiv = document.createElement('div');
    errorDiv.className = 'error-message';
    errorDiv.textContent = message;
    
    document.querySelector('.profile-content').prepend(errorDiv);
    
    setTimeout(() => {
        errorDiv.remove();
    }, 3000);
}
```

# Interface Utilisateur - Récupération de Mot de Passe
## 3.6 Système de Récupération de Mot de Passe

### Description
Le système de récupération comprend deux pages :
1. Demande de réinitialisation (envoi d'email)
2. Réinitialisation du mot de passe (avec token)

### 3.6.1 Page de Demande de Réinitialisation
```html
<!-- frontend/pages/forgot-password.html -->
<!DOCTYPE html>
<html lang="fr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>FMP - Mot de passe oublié</title>
    <link rel="stylesheet" href="../static/css/main.css">
    <link rel="stylesheet" href="../static/css/auth.css">
</head>
<body>
    <div class="auth-container">
        <div class="auth-box">
            <h2>Mot de passe oublié</h2>
            <p class="auth-subtitle">
                Entrez votre adresse email pour recevoir un lien de réinitialisation
            </p>

            <!-- Formulaire de demande -->
            <form id="forgot-password-form" class="auth-form">
                <!-- Message de statut -->
                <div id="status-message" class="status-message" style="display: none;"></div>

                <!-- Email -->
                <div class="form-group">
                    <label for="email">Email</label>
                    <input 
                        type="email" 
                        id="email" 
                        name="email" 
                        required 
                        autocomplete="email">
                </div>

                <!-- Bouton d'envoi -->
                <button type="submit" class="auth-button">
                    Envoyer le lien
                </button>
            </form>

            <!-- Liens -->
            <div class="auth-links">
                <p>
                    <a href="login.html">Retour à la connexion</a>
                </p>
            </div>
        </div>
    </div>

    <script src="../static/js/forgot-password.js"></script>
</body>
</html>
```

### 3.6.2 Page de Réinitialisation
```html
<!-- frontend/pages/reset-password.html -->
<!DOCTYPE html>
<html lang="fr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>FMP - Réinitialisation du mot de passe</title>
    <link rel="stylesheet" href="../static/css/main.css">
    <link rel="stylesheet" href="../static/css/auth.css">
</head>
<body>
    <div class="auth-container">
        <div class="auth-box">
            <h2>Réinitialisation du mot de passe</h2>
            <p class="auth-subtitle">
                Choisissez votre nouveau mot de passe
            </p>

            <!-- Formulaire de réinitialisation -->
            <form id="reset-password-form" class="auth-form">
                <!-- Message de statut -->
                <div id="status-message" class="status-message" style="display: none;"></div>

                <!-- Nouveau mot de passe -->
                <div class="form-group">
                    <label for="password">Nouveau mot de passe</label>
                    <div class="password-input-group">
                        <input 
                            type="password" 
                            id="password" 
                            name="password" 
                            required>
                        <button type="button" class="toggle-password">
                            <span class="eye-icon">👁️</span>
                        </button>
                    </div>
                    <!-- Indicateur de force -->
                    <div class="password-strength">
                        <div class="strength-meter">
                            <div class="meter-section"></div>
                            <div class="meter-section"></div>
                            <div class="meter-section"></div>
                            <div class="meter-section"></div>
                        </div>
                        <span class="strength-text">Force du mot de passe</span>
                    </div>
                </div>

                <!-- Confirmation -->
                <div class="form-group">
                    <label for="password-confirm">Confirmer le mot de passe</label>
                    <input 
                        type="password" 
                        id="password-confirm" 
                        name="password-confirm" 
                        required>
                </div>

                <!-- Bouton de validation -->
                <button type="submit" class="auth-button">
                    Réinitialiser le mot de passe
                </button>
            </form>
        </div>
    </div>

    <script src="../static/js/reset-password.js"></script>
</body>
</html>
```

### 3.6.3 Styles spécifiques
```css
/* Ajouts au fichier frontend/static/css/auth.css */

/* Messages de statut */
.status-message {
    padding: 1rem;
    border-radius: 4px;
    margin-bottom: 1rem;
    font-size: 0.9rem;
}

.status-message.success {
    background-color: #d4edda;
    color: #155724;
    border: 1px solid #c3e6cb;
}

.status-message.error {
    background-color: #f8d7da;
    color: #721c24;
    border: 1px solid #f5c6cb;
}

.status-message.info {
    background-color: #cce5ff;
    color: #004085;
    border: 1px solid #b8daff;
}

/* Sous-titre */
.auth-subtitle {
    text-align: center;
    color: #666;
    margin-bottom: 1.5rem;
    font-size: 0.9rem;
}

/* Animation de chargement */
.loading {
    position: relative;
}

.loading::after {
    content: '';
    position: absolute;
    top: 50%;
    left: 50%;
    width: 20px;
    height: 20px;
    margin: -10px 0 0 -10px;
    border: 2px solid #f3f3f3;
    border-top: 2px solid #3498db;
    border-radius: 50%;
    animation: spin 1s linear infinite;
}

@keyframes spin {
    0% { transform: rotate(0deg); }
    100% { transform: rotate(360deg); }
}
```

### 3.6.4 JavaScript - Demande de Réinitialisation
```javascript
// frontend/static/js/forgot-password.js

document.addEventListener('DOMContentLoaded', function() {
    const form = document.getElementById('forgot-password-form');
    const statusMessage = document.getElementById('status-message');

    if (form) {
        form.addEventListener('submit', async function(e) {
            e.preventDefault();

            try {
                // Désactivation du formulaire pendant la requête
                form.classList.add('loading');
                form.querySelector('button').disabled = true;

                const email = this.email.value.trim();

                const response = await fetch('/api/auth/forgot-password', {
                    method: 'POST',
                    headers: {
                        'Content-Type': 'application/json'
                    },
                    body: JSON.stringify({ email })
                });

                const data = await response.json();

                if (!response.ok) {
                    throw new Error(data.error || 'Une erreur est survenue');
                }

                // Affichage du message de succès
                showStatus('success', 
                    'Un email contenant les instructions de réinitialisation ' +
                    'vous a été envoyé. Vérifiez votre boîte de réception.'
                );

                // Réinitialisation du formulaire
                form.reset();

            } catch (error) {
                showStatus('error', error.message);
            } finally {
                // Réactivation du formulaire
                form.classList.remove('loading');
                form.querySelector('button').disabled = false;
            }
        });
    }
});

/**
 * Affiche un message de statut
 * @param {string} type - Type de message ('success', 'error', 'info')
 * @param {string} message - Contenu du message
 */
function showStatus(type, message) {
    const statusDiv = document.getElementById('status-message');
    statusDiv.className = `status-message ${type}`;
    statusDiv.textContent = message;
    statusDiv.style.display = 'block';

    if (type === 'success') {
        setTimeout(() => {
            statusDiv.style.display = 'none';
        }, 5000);
    }
}
```

### 3.6.5 JavaScript - Réinitialisation
```javascript
// frontend/static/js/reset-password.js

document.addEventListener('DOMContentLoaded', function() {
    const form = document.getElementById('reset-password-form');
    const togglePasswordBtn = document.querySelector('.toggle-password');
    const passwordInput = document.getElementById('password');
    const statusMessage = document.getElementById('status-message');

    // Récupération du token dans l'URL
    const urlParams = new URLSearchParams(window.location.search);
    const token = urlParams.get('token');

    // Vérification de la présence du token
    if (!token) {
        showStatus('error', 'Token de réinitialisation manquant');
        form.style.display = 'none';
        return;
    }

    // Affichage/masquage du mot de passe
    if (togglePasswordBtn) {
        togglePasswordBtn.addEventListener('click', function() {
            const type = passwordInput.type === 'password' ? 'text' : 'password';
            passwordInput.type = type;
            this.querySelector('.eye-icon').textContent = 
                type === 'password' ? '👁️' : '👁️‍🗨️';
        });
    }

    // Validation en temps réel du mot de passe
    if (passwordInput) {
        passwordInput.addEventListener('input', function() {
            const strength = checkPasswordStrength(this.value);
            updatePasswordStrengthIndicator(strength);
        });
    }

    if (form) {
        form.addEventListener('submit', async function(e) {
            e.preventDefault();

            try {
                const password = this.password.value;
                const passwordConfirm = this['password-confirm'].value;

                // Validation des mots de passe
                if (password !== passwordConfirm) {
                    throw new Error('Les mots de passe ne correspondent pas');
                }

                // Vérification de la force du mot de passe
                const strength = checkPasswordStrength(password);
                if (strength.score < 3) {
                    throw new Error('Le mot de passe est trop faible');
                }

                // Envoi de la requête
                const response = await fetch('/api/auth/reset-password', {
                    method: 'POST',
                    headers: {
                        'Content-Type': 'application/json'
                    },
                    body: JSON.stringify({ 
                        token,
                        password 
                    })
                });

                const data = await response.json();

                if (!response.ok) {
                    throw new Error(data.error || 'Une erreur est survenue');
                }

                // Affichage du succès
                showStatus('success', 
                    'Votre mot de passe a été réinitialisé. ' +
                    'Redirection vers la page de connexion...'
                );

                // Redirection après 3 secondes
                setTimeout(() => {
                    window.location.href = '/login.html';
                }, 3000);

            } catch (error) {
                showStatus('error', error.message);
            }
        });
    }
});

// Fonction de vérification de la force du mot de passe
function checkPasswordStrength(password) {
    // ... (même fonction que dans register.js)
}

// Fonction de mise à jour de l'indicateur
function updatePasswordStrengthIndicator(strength) {
    // ... (même fonction que dans register.js)
}

// Fonction d'affichage des messages
function showStatus(type, message) {
    // ... (même fonction que dans forgot-password.js)
}
```

# Interface Utilisateur - Vérification d'Email
## 3.7 Système de Vérification d'Email

### Description
Le système comprend :
1. Une page de confirmation après l'inscription
2. Une page de vérification avec le token
3. Un système de renvoi d'email

### 3.7.1 Page de Confirmation d'Inscription
```html
<!-- frontend/pages/verify-email.html -->
<!DOCTYPE html>
<html lang="fr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>FMP - Vérification Email</title>
    <link rel="stylesheet" href="../static/css/main.css">
    <link rel="stylesheet" href="../static/css/auth.css">
    <link rel="stylesheet" href="../static/css/verify.css">
</head>
<body>
    <div class="auth-container">
        <div class="auth-box verify-box">
            <h2>Vérification de votre email</h2>
            
            <!-- État initial - Email envoyé -->
            <div id="verification-pending" class="verification-state">
                <div class="icon-container">
                    <img src="../static/images/email-sent.svg" alt="Email envoyé">
                </div>
                <p class="verify-message">
                    Un email de vérification a été envoyé à
                    <strong id="user-email">votre adresse email</strong>
                </p>
                <p class="verify-instructions">
                    Veuillez cliquer sur le lien dans l'email pour activer votre compte.
                </p>
                
                <!-- Minuteur pour le renvoi -->
                <div class="resend-timer">
                    <p>Vous pouvez demander un nouvel email dans</p>
                    <span id="timer">02:00</span>
                </div>
                
                <!-- Bouton de renvoi (initialement désactivé) -->
                <button id="resend-button" class="secondary-button" disabled>
                    Renvoyer l'email
                </button>
            </div>

            <!-- État de succès -->
            <div id="verification-success" class="verification-state" style="display: none;">
                <div class="icon-container success">
                    <img src="../static/images/email-verified.svg" alt="Email vérifié">
                </div>
                <h3>Email vérifié avec succès !</h3>
                <p>Votre compte est maintenant activé.</p>
                <a href="/login.html" class="auth-button">
                    Se connecter
                </a>
            </div>

            <!-- État d'erreur -->
            <div id="verification-error" class="verification-state" style="display: none;">
                <div class="icon-container error">
                    <img src="../static/images/error.svg" alt="Erreur">
                </div>
                <h3>Une erreur est survenue</h3>
                <p id="error-message">Le lien de vérification est invalide ou a expiré.</p>
                <button id="request-new-link" class="auth-button">
                    Demander un nouveau lien
                </button>
            </div>
        </div>
    </div>

    <script src="../static/js/verify-email.js"></script>
</body>
</html>
```

### 3.7.2 Styles de Vérification
```css
/* frontend/static/css/verify.css */

.verify-box {
    text-align: center;
    max-width: 450px;
}

/* Conteneur d'icône */
.icon-container {
    width: 80px;
    height: 80px;
    margin: 0 auto 1.5rem;
}

.icon-container img {
    width: 100%;
    height: 100%;
    object-fit: contain;
}

/* États de vérification */
.verification-state {
    display: none;
}

.verification-state.active {
    display: block;
}

/* Messages */
.verify-message {
    font-size: 1.1rem;
    margin-bottom: 1rem;
}

.verify-message strong {
    color: var(--primary-color);
}

.verify-instructions {
    color: #666;
    margin-bottom: 2rem;
}

/* Minuteur */
.resend-timer {
    margin: 2rem 0;
    padding: 1rem;
    background: #f8f9fa;
    border-radius: 4px;
}

.resend-timer p {
    color: #666;
    margin-bottom: 0.5rem;
}

#timer {
    font-size: 1.2rem;
    font-weight: bold;
    color: var(--primary-color);
}

/* Boutons */
.secondary-button {
    background: none;
    border: 1px solid var(--primary-color);
    color: var(--primary-color);
    padding: 0.8rem 1.5rem;
    border-radius: 4px;
    cursor: pointer;
    transition: all 0.3s;
}

.secondary-button:hover:not(:disabled) {
    background: var(--primary-color);
    color: white;
}

.secondary-button:disabled {
    border-color: #ccc;
    color: #999;
    cursor: not-allowed;
}

/* États de succès et d'erreur */
.icon-container.success img {
    animation: success-bounce 0.5s ease;
}

.icon-container.error img {
    animation: error-shake 0.5s ease;
}

@keyframes success-bounce {
    0%, 20%, 50%, 80%, 100% {
        transform: translateY(0);
    }
    40% {
        transform: translateY(-20px);
    }
    60% {
        transform: translateY(-10px);
    }
}

@keyframes error-shake {
    0%, 100% {
        transform: translateX(0);
    }
    25% {
        transform: translateX(-10px);
    }
    75% {
        transform: translateX(10px);
    }
}
```

### 3.7.3 JavaScript de Vérification
```javascript
// frontend/static/js/verify-email.js

document.addEventListener('DOMContentLoaded', function() {
    // Initialisation des éléments
    const resendButton = document.getElementById('resend-button');
    const timerDisplay = document.getElementById('timer');
    
    // Récupération du token dans l'URL
    const urlParams = new URLSearchParams(window.location.search);
    const token = urlParams.get('token');
    
    // Si on a un token, on tente de vérifier l'email
    if (token) {
        verifyEmail(token);
    } else {
        // Sinon on affiche l'état initial
        showVerificationState('pending');
        startResendTimer();
    }
    
    // Gestionnaire pour le renvoi d'email
    if (resendButton) {
        resendButton.addEventListener('click', handleResendEmail);
    }
});

/**
 * Affiche l'état de vérification approprié
 * @param {string} state - État à afficher ('pending', 'success', 'error')
 * @param {string} [message] - Message optionnel d'erreur
 */
function showVerificationState(state, message = '') {
    // Cache tous les états
    document.querySelectorAll('.verification-state').forEach(el => {
        el.style.display = 'none';
    });
    
    // Affiche l'état demandé
    const stateElement = document.getElementById(`verification-${state}`);
    if (stateElement) {
        stateElement.style.display = 'block';
    }
    
    // Met à jour le message d'erreur si nécessaire
    if (state === 'error' && message) {
        document.getElementById('error-message').textContent = message;
    }
}

/**
 * Démarre le minuteur pour le renvoi d'email
 */
function startResendTimer() {
    let timeLeft = 120; // 2 minutes
    const timerDisplay = document.getElementById('timer');
    const resendButton = document.getElementById('resend-button');
    
    const timer = setInterval(() => {
        timeLeft--;
        
        // Mise à jour de l'affichage
        const minutes = Math.floor(timeLeft / 60);
        const seconds = timeLeft % 60;
        timerDisplay.textContent = 
            `${minutes.toString().padStart(2, '0')}:${seconds.toString().padStart(2, '0')}`;
        
        // Fin du timer
        if (timeLeft <= 0) {
            clearInterval(timer);
            resendButton.disabled = false;
            timerDisplay.parentElement.style.display = 'none';
        }
    }, 1000);
}

/**
 * Vérifie le token d'email
 * @param {string} token - Token de vérification
 */
async function verifyEmail(token) {
    try {
        const response = await fetch('/api/auth/verify-email', {
            method: 'POST',
            headers: {
                'Content-Type': 'application/json'
            },
            body: JSON.stringify({ token })
        });
        
        const data = await response.json();
        
        if (!response.ok) {
            throw new Error(data.error || 'Erreur de vérification');
        }
        
        // Affichage du succès
        showVerificationState('success');
        
    } catch (error) {
        showVerificationState('error', error.message);
    }
}

/**
 * Gère le renvoi de l'email de vérification
 */
async function handleResendEmail() {
    const button = document.getElementById('resend-button');
    button.disabled = true;
    
    try {
        const response = await fetch('/api/auth/resend-verification', {
            method: 'POST',
            headers: {
                'Content-Type': 'application/json',
                'Authorization': `Bearer ${localStorage.getItem('token')}`
            }
        });
        
        const data = await response.json();
        
        if (!response.ok) {
            throw new Error(data.error || 'Erreur lors du renvoi');
        }
        
        // Réinitialisation du timer
        startResendTimer();
        document.querySelector('.resend-timer').style.display = 'block';
        
    } catch (error) {
        showVerificationState('error', error.message);
    }
}
```

# Interface Utilisateur - Système de Notifications
## 3.8 Système de Notifications en Temps Réel

### Description
Le système de notifications comprend :
1. Une classe de gestion des notifications
2. Un composant visuel pour afficher les notifications
3. Des animations et transitions
4. Une gestion de file d'attente pour les notifications multiples

### 3.8.1 HTML - Conteneur de Notifications
```html
<!-- À ajouter dans toutes les pages qui nécessitent des notifications -->
<div id="notification-container" class="notification-container"></div>
```

### 3.8.2 Styles des Notifications
```css
/* frontend/static/css/notifications.css */

/* Conteneur principal */
.notification-container {
    position: fixed;
    top: 20px;
    right: 20px;
    z-index: 1000;
    max-width: 350px;
    width: 100%;
}

/* Notification individuelle */
.notification {
    background: white;
    border-radius: 8px;
    box-shadow: 0 4px 12px rgba(0, 0, 0, 0.15);
    margin-bottom: 10px;
    padding: 16px;
    display: flex;
    align-items: flex-start;
    gap: 12px;
    animation: slideIn 0.3s ease forwards;
    position: relative;
    overflow: hidden;
}

/* Indicateur de progression */
.notification-progress {
    position: absolute;
    bottom: 0;
    left: 0;
    height: 3px;
    background: var(--primary-color);
    opacity: 0.7;
    transition: width linear;
}

/* Types de notifications */
.notification.success {
    border-left: 4px solid #2ecc71;
}

.notification.error {
    border-left: 4px solid #e74c3c;
}

.notification.warning {
    border-left: 4px solid #f1c40f;
}

.notification.info {
    border-left: 4px solid #3498db;
}

/* Icônes */
.notification-icon {
    flex-shrink: 0;
    width: 24px;
    height: 24px;
}

.notification.success .notification-icon {
    color: #2ecc71;
}

.notification.error .notification-icon {
    color: #e74c3c;
}

.notification.warning .notification-icon {
    color: #f1c40f;
}

.notification.info .notification-icon {
    color: #3498db;
}

/* Contenu */
.notification-content {
    flex-grow: 1;
}

.notification-title {
    font-weight: 600;
    margin-bottom: 4px;
}

.notification-message {
    font-size: 0.9rem;
    color: #666;
    line-height: 1.4;
}

/* Bouton de fermeture */
.notification-close {
    background: none;
    border: none;
    color: #999;
    cursor: pointer;
    padding: 4px;
    font-size: 1.2rem;
    line-height: 1;
    transition: color 0.2s;
}

.notification-close:hover {
    color: #333;
}

/* Animations */
@keyframes slideIn {
    from {
        transform: translateX(100%);
        opacity: 0;
    }
    to {
        transform: translateX(0);
        opacity: 1;
    }
}

@keyframes slideOut {
    from {
        transform: translateX(0);
        opacity: 1;
    }
    to {
        transform: translateX(100%);
        opacity: 0;
    }
}

/* Animation de suppression */
.notification.removing {
    animation: slideOut 0.3s ease forwards;
}

/* Responsive */
@media (max-width: 480px) {
    .notification-container {
        top: 10px;
        right: 10px;
        left: 10px;
        max-width: none;
    }

    .notification {
        margin-bottom: 8px;
        padding: 12px;
    }
}
```

### 3.8.3 JavaScript - Gestionnaire de Notifications
```javascript
// frontend/static/js/notifications.js

/**
 * Gestionnaire de notifications
 * Permet d'afficher des messages de notification à l'utilisateur
 */
class NotificationManager {
    constructor() {
        this.container = null;
        this.queue = [];
        this.maxVisible = 3;
        this.currentVisible = 0;
        this.initialize();
    }

    /**
     * Initialise le gestionnaire de notifications
     */
    initialize() {
        // Création du conteneur s'il n'existe pas
        if (!document.getElementById('notification-container')) {
            this.container = document.createElement('div');
            this.container.id = 'notification-container';
            this.container.className = 'notification-container';
            document.body.appendChild(this.container);
        } else {
            this.container = document.getElementById('notification-container');
        }
    }

    /**
     * Crée une nouvelle notification
     * @param {Object} options - Options de la notification
     * @param {string} options.type - Type de notification ('success', 'error', 'warning', 'info')
     * @param {string} options.title - Titre de la notification
     * @param {string} options.message - Message de la notification
     * @param {number} [options.duration=5000] - Durée d'affichage en ms
     */
    create({ type = 'info', title, message, duration = 5000 }) {
        const notification = {
            id: Date.now(),
            type,
            title,
            message,
            duration
        };

        // Ajout à la file d'attente
        this.queue.push(notification);
        this.processQueue();
    }

    /**
     * Traite la file d'attente des notifications
     */
    processQueue() {
        // Vérifie si on peut afficher plus de notifications
        if (this.currentVisible >= this.maxVisible || this.queue.length === 0) {
            return;
        }

        const notification = this.queue.shift();
        this.show(notification);
    }

    /**
     * Affiche une notification
     * @param {Object} notification - Notification à afficher
     */
    show(notification) {
        this.currentVisible++;

        // Création de l'élément de notification
        const element = document.createElement('div');
        element.className = `notification ${notification.type}`;
        element.innerHTML = `
            <div class="notification-icon">
                ${this.getIconForType(notification.type)}
            </div>
            <div class="notification-content">
                <div class="notification-title">${notification.title}</div>
                <div class="notification-message">${notification.message}</div>
            </div>
            <button class="notification-close">&times;</button>
            <div class="notification-progress"></div>
        `;

        // Ajout au conteneur
        this.container.appendChild(element);

        // Gestion de la barre de progression
        const progress = element.querySelector('.notification-progress');
        progress.style.width = '100%';
        progress.style.transition = `width ${notification.duration}ms linear`;
        setTimeout(() => {
            progress.style.width = '0%';
        }, 50);

        // Suppression automatique
        const timeout = setTimeout(() => {
            this.remove(element, notification.id);
        }, notification.duration);

        // Gestion du bouton de fermeture
        const closeButton = element.querySelector('.notification-close');
        closeButton.addEventListener('click', () => {
            clearTimeout(timeout);
            this.remove(element, notification.id);
        });

        // Pause au survol
        element.addEventListener('mouseenter', () => {
            clearTimeout(timeout);
            progress.style.transition = 'none';
        });

        element.addEventListener('mouseleave', () => {
            const remainingWidth = progress.offsetWidth;
            const remainingTime = (remainingWidth / element.offsetWidth) * notification.duration;
            progress.style.transition = `width ${remainingTime}ms linear`;
            progress.style.width = '0%';
            
            const newTimeout = setTimeout(() => {
                this.remove(element, notification.id);
            }, remainingTime);
        });
    }

    /**
     * Supprime une notification
     * @param {HTMLElement} element - Élément à supprimer
     * @param {number} id - ID de la notification
     */
    remove(element, id) {
        element.classList.add('removing');
        
        element.addEventListener('animationend', () => {
            element.remove();
            this.currentVisible--;
            this.processQueue();
        });
    }

    /**
     * Retourne l'icône correspondant au type de notification
     * @param {string} type - Type de notification
     * @returns {string} Code HTML de l'icône
     */
    getIconForType(type) {
        const icons = {
            success: '<svg viewBox="0 0 24 24" width="24" height="24"><path fill="currentColor" d="M9 16.17L4.83 12l-1.42 1.41L9 19 21 7l-1.41-1.41L9 16.17z"/></svg>',
            error: '<svg viewBox="0 0 24 24" width="24" height="24"><path fill="currentColor" d="M19 6.41L17.59 5 12 10.59 6.41 5 5 6.41 10.59 12 5 17.59 6.41 19 12 13.41 17.59 19 19 17.59 13.41 12 19 6.41z"/></svg>',
            warning: '<svg viewBox="0 0 24 24" width="24" height="24"><path fill="currentColor" d="M12 2L1 21h22L12 2zm1 14h-2v2h2v-2zm0-6h-2v4h2v-4z"/></svg>',
            info: '<svg viewBox="0 0 24 24" width="24" height="24"><path fill="currentColor" d="M12 2C6.48 2 2 6.48 2 12s4.48 10 10 10 10-4.48 10-10S17.52 2 12 2zm1 15h-2v-6h2v6zm0-8h-2V7h2v2z"/></svg>'
        };
        
        return icons[type] || icons.info;
    }
}

// Export de l'instance
export const notifications = new NotificationManager();
```

### 3.8.4 Exemple d'utilisation
```javascript
// Importation
import { notifications } from './notifications.js';

// Exemples d'utilisation
notifications.create({
    type: 'success',
    title: 'Succès!',
    message: 'Votre profil a été mis à jour.',
    duration: 5000
});

notifications.create({
    type: 'error',
    title: 'Erreur',
    message: 'Une erreur est survenue lors de la sauvegarde.',
    duration: 8000
});

notifications.create({
    type: 'warning',
    title: 'Attention',
    message: 'Votre session va expirer dans 5 minutes.',
    duration: 10000
});

notifications.create({
    type: 'info',
    title: 'Information',
    message: 'Nouvelle version disponible!',
    duration: 4000
});
```
# Interface Utilisateur - Gestion de Session
## 3.9 Système de Gestion de Session

### Description
Le système de gestion de session comprend :
1. Un gestionnaire de session côté client
2. Un système de rafraîchissement automatique du token
3. Une gestion de la déconnexion
4. Des intercepteurs pour les requêtes API

### 3.9.1 JavaScript - Gestionnaire de Session
```javascript
// frontend/static/js/session.js

/**
 * Gestionnaire de session utilisateur
 * Gère l'authentification et les tokens
 */
class SessionManager {
    constructor() {
        this.tokenKey = 'auth_token';
        this.refreshTokenKey = 'refresh_token';
        this.userKey = 'user_data';
        this.refreshInterval = null;
        this.initialize();
    }

    /**
     * Initialise le gestionnaire de session
     */
    initialize() {
        // Vérifie si un token existe
        if (this.isAuthenticated()) {
            this.setupTokenRefresh();
            this.setupRequestInterceptors();
        }

        // Écoute les événements de stockage (pour la synchronisation multi-onglets)
        window.addEventListener('storage', (event) => {
            if (event.key === this.tokenKey) {
                // Token modifié dans un autre onglet
                if (!event.newValue) {
                    this.handleLogout();
                }
            }
        });
    }

    /**
     * Vérifie si l'utilisateur est authentifié
     * @returns {boolean} Statut d'authentification
     */
    isAuthenticated() {
        const token = localStorage.getItem(this.tokenKey);
        return !!token && !this.isTokenExpired(token);
    }

    /**
     * Vérifie si un token est expiré
     * @param {string} token - Token JWT à vérifier
     * @returns {boolean} True si le token est expiré
     */
    isTokenExpired(token) {
        try {
            const payload = JSON.parse(atob(token.split('.')[1]));
            return payload.exp * 1000 < Date.now();
        } catch (error) {
            return true;
        }
    }

    /**
     * Configure le rafraîchissement automatique du token
     */
    setupTokenRefresh() {
        if (this.refreshInterval) {
            clearInterval(this.refreshInterval);
        }

        // Rafraîchit le token toutes les 45 minutes
        this.refreshInterval = setInterval(() => {
            this.refreshToken();
        }, 45 * 60 * 1000);
    }

    /**
     * Configure les intercepteurs de requêtes
     */
    setupRequestInterceptors() {
        // Intercepte toutes les requêtes fetch
        const originalFetch = window.fetch;
        window.fetch = async (...args) => {
            const [resource, config] = args;

            // Ajoute le token aux headers si nécessaire
            if (this.isAuthenticated()) {
                config.headers = {
                    ...config.headers,
                    'Authorization': `Bearer ${this.getToken()}`
                };
            }

            try {
                const response = await originalFetch(resource, config);

                // Gestion des erreurs d'authentification
                if (response.status === 401) {
                    // Tentative de rafraîchissement du token
                    const success = await this.refreshToken();
                    if (success) {
                        // Réessaie la requête avec le nouveau token
                        config.headers['Authorization'] = `Bearer ${this.getToken()}`;
                        return originalFetch(resource, config);
                    } else {
                        // Déconnexion si le rafraîchissement échoue
                        this.handleLogout();
                    }
                }

                return response;
            } catch (error) {
                throw error;
            }
        };
    }

    /**
     * Rafraîchit le token d'accès
     * @returns {Promise<boolean>} Succès du rafraîchissement
     */
    async refreshToken() {
        try {
            const refreshToken = localStorage.getItem(this.refreshTokenKey);
            if (!refreshToken) {
                return false;
            }

            const response = await fetch('/api/auth/refresh', {
                method: 'POST',
                headers: {
                    'Content-Type': 'application/json',
                    'Authorization': `Bearer ${refreshToken}`
                }
            });

            if (!response.ok) {
                throw new Error('Échec du rafraîchissement du token');
            }

            const data = await response.json();
            this.setToken(data.token);
            return true;

        } catch (error) {
            console.error('Erreur lors du rafraîchissement du token:', error);
            return false;
        }
    }

    /**
     * Gère la connexion de l'utilisateur
     * @param {Object} data - Données de connexion (token et utilisateur)
     */
    handleLogin(data) {
        this.setToken(data.token);
        this.setRefreshToken(data.refresh_token);
        this.setUserData(data.user);
        this.setupTokenRefresh();
        this.setupRequestInterceptors();

        // Événement personnalisé pour la connexion
        window.dispatchEvent(new CustomEvent('userLogin', {
            detail: data.user
        }));
    }

    /**
     * Gère la déconnexion de l'utilisateur
     */
    async handleLogout() {
        try {
            // Appel API de déconnexion
            if (this.isAuthenticated()) {
                await fetch('/api/auth/logout', {
                    method: 'POST',
                    headers: {
                        'Authorization': `Bearer ${this.getToken()}`
                    }
                });
            }
        } catch (error) {
            console.error('Erreur lors de la déconnexion:', error);
        } finally {
            // Nettoyage local
            this.clearSession();
        }
    }

    /**
     * Nettoie les données de session
     */
    clearSession() {
        localStorage.removeItem(this.tokenKey);
        localStorage.removeItem(this.refreshTokenKey);
        localStorage.removeItem(this.userKey);

        if (this.refreshInterval) {
            clearInterval(this.refreshInterval);
        }

        // Événement personnalisé pour la déconnexion
        window.dispatchEvent(new CustomEvent('userLogout'));

        // Redirection vers la page de connexion
        window.location.href = '/login.html';
    }

    // Getters et Setters
    getToken() {
        return localStorage.getItem(this.tokenKey);
    }

    setToken(token) {
        localStorage.setItem(this.tokenKey, token);
    }

    setRefreshToken(token) {
        localStorage.setItem(this.refreshTokenKey, token);
    }

    getUserData() {
        const data = localStorage.getItem(this.userKey);
        return data ? JSON.parse(data) : null;
    }

    setUserData(data) {
        localStorage.setItem(this.userKey, JSON.stringify(data));
    }
}

// Export de l'instance unique
export const session = new SessionManager();
```

### 3.9.2 HTML - Ajout au Layout Principal
```html
<!-- À ajouter dans le head de toutes les pages protégées -->
<script type="module">
    import { session } from '../static/js/session.js';
    
    // Vérification de l'authentification
    if (!session.isAuthenticated()) {
        window.location.href = '/login.html';
    }
</script>
```

### 3.9.3 JavaScript - Exemple d'Utilisation
```javascript
// Exemple dans login.js
import { session } from './session.js';
import { notifications } from './notifications.js';

async function handleLogin(credentials) {
    try {
        const response = await fetch('/api/auth/login', {
            method: 'POST',
            headers: {
                'Content-Type': 'application/json'
            },
            body: JSON.stringify(credentials)
        });

        const data = await response.json();

        if (!response.ok) {
            throw new Error(data.error);
        }

        // Gestion de la connexion
        session.handleLogin(data);

        // Notification de succès
        notifications.create({
            type: 'success',
            title: 'Connexion réussie',
            message: `Bienvenue, ${data.user.firstname}!`
        });

        // Redirection
        window.location.href = '/';

    } catch (error) {
        notifications.create({
            type: 'error',
            title: 'Erreur de connexion',
            message: error.message
        });
    }
}
```

### 3.9.4 Protection des Routes Frontend
```javascript
// frontend/static/js/route-guard.js

import { session } from './session.js';
import { notifications } from './notifications.js';

/**
 * Protège une page contre les accès non autorisés
 * @param {Object} options - Options de protection
 * @param {boolean} options.requireAuth - Authentification requise
 * @param {boolean} options.requireAdmin - Droits admin requis
 */
export function guardRoute({ requireAuth = true, requireAdmin = false } = {}) {
    // Vérifie l'authentification si nécessaire
    if (requireAuth && !session.isAuthenticated()) {
        notifications.create({
            type: 'warning',
            title: 'Accès refusé',
            message: 'Veuillez vous connecter pour accéder à cette page.'
        });
        
        // Sauvegarde la page demandée pour la redirection après connexion
        sessionStorage.setItem('redirectUrl', window.location.href);
        window.location.href = '/login.html';
        return;
    }

    // Vérifie les droits admin si nécessaire
    if (requireAdmin) {
        const userData = session.getUserData();
        if (!userData?.is_admin) {
            notifications.create({
                type: 'error',
                title: 'Accès refusé',
                message: 'Vous n\'avez pas les droits nécessaires.'
            });
            window.location.href = '/';
            return;
        }
    }
}
```

# Interface Utilisateur - Déconnexion et Gestion des Erreurs
## 3.10 Système de Déconnexion et Gestion des Erreurs

### Description
Le système comprend :
1. Une page de déconnexion
2. Un gestionnaire d'erreurs global
3. Des écrans d'erreur personnalisés
4. Un système de redirection

### 3.10.1 Page de Déconnexion
```html
<!-- frontend/pages/logout.html -->
<!DOCTYPE html>
<html lang="fr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>FMP - Déconnexion</title>
    <link rel="stylesheet" href="../static/css/main.css">
    <link rel="stylesheet" href="../static/css/auth.css">
    <link rel="stylesheet" href="../static/css/logout.css">

    <script type="module">
        import { session } from '../static/js/session.js';
        
        // Déconnexion immédiate
        session.handleLogout();
    </script>
</head>
<body>
    <div class="auth-container">
        <div class="auth-box logout-box">
            <div class="logout-animation">
                <!-- Animation de déconnexion -->
                <svg class="logout-icon" viewBox="0 0 24 24">
                    <path d="M16 17v-3H9v-4h7V7l5 5-5 5M14 2a2 2 0 012 2v2h-2V4H5v16h9v-2h2v2a2 2 0 01-2 2H5a2 2 0 01-2-2V4a2 2 0 012-2h9z"/>
                </svg>
            </div>
            <h2>Déconnexion en cours...</h2>
            <p>Vous allez être redirigé vers la page d'accueil.</p>
        </div>
    </div>
</body>
</html>
```

### 3.10.2 Styles de Déconnexion
```css
/* frontend/static/css/logout.css */

.logout-box {
    text-align: center;
    padding: 3rem 2rem;
}

.logout-animation {
    margin-bottom: 2rem;
}

.logout-icon {
    width: 64px;
    height: 64px;
    fill: var(--primary-color);
    animation: rotate-out 0.8s ease-in-out forwards;
}

@keyframes rotate-out {
    0% {
        transform: scale(1) rotate(0);
        opacity: 1;
    }
    50% {
        transform: scale(1.2) rotate(180deg);
        opacity: 0.5;
    }
    100% {
        transform: scale(0) rotate(360deg);
        opacity: 0;
    }
}

.logout-box h2 {
    color: var(--primary-color);
    margin-bottom: 1rem;
}

.logout-box p {
    color: #666;
}

/* Animation de chargement */
.loading-dots:after {
    content: '.';
    animation: dots 1.5s steps(5, end) infinite;
}

@keyframes dots {
    0%, 20% { content: '.'; }
    40% { content: '..'; }
    60% { content: '...'; }
    80%, 100% { content: ''; }
}
```

### 3.10.3 Gestionnaire d'Erreurs Global
```javascript
// frontend/static/js/error-handler.js

import { notifications } from './notifications.js';
import { session } from './session.js';

/**
 * Gestionnaire d'erreurs global
 */
class ErrorHandler {
    constructor() {
        this.initialize();
    }

    /**
     * Initialise le gestionnaire d'erreurs
     */
    initialize() {
        // Capture des erreurs non gérées
        window.addEventListener('error', this.handleGlobalError.bind(this));
        window.addEventListener('unhandledrejection', this.handlePromiseError.bind(this));

        // Intercepte les erreurs réseau
        this.setupFetchInterceptor();
    }

    /**
     * Gère les erreurs JavaScript globales
     * @param {ErrorEvent} event - Événement d'erreur
     */
    handleGlobalError(event) {
        console.error('Erreur globale:', event.error);

        notifications.create({
            type: 'error',
            title: 'Erreur',
            message: 'Une erreur inattendue est survenue.',
            duration: 8000
        });

        // Empêche l'affichage de l'erreur dans la console
        event.preventDefault();
    }

    /**
     * Gère les rejets de promesses non gérés
     * @param {PromiseRejectionEvent} event - Événement de rejet
     */
    handlePromiseError(event) {
        console.error('Promesse rejetée:', event.reason);

        notifications.create({
            type: 'error',
            title: 'Erreur',
            message: 'Une erreur est survenue lors d\'une opération asynchrone.',
            duration: 8000
        });

        event.preventDefault();
    }

    /**
     * Configure l'intercepteur de requêtes fetch
     */
    setupFetchInterceptor() {
        const originalFetch = window.fetch;
        
        window.fetch = async (...args) => {
            try {
                const response = await originalFetch(...args);

                // Gestion des erreurs HTTP
                if (!response.ok) {
                    const data = await response.json().catch(() => ({}));
                    
                    switch (response.status) {
                        case 401:
                            // Problème d'authentification
                            if (!args[0].includes('/auth/')) {
                                session.handleLogout();
                            }
                            break;

                        case 403:
                            // Accès non autorisé
                            notifications.create({
                                type: 'error',
                                title: 'Accès refusé',
                                message: 'Vous n\'avez pas les droits nécessaires.'
                            });
                            break;

                        case 404:
                            // Ressource non trouvée
                            this.showErrorPage('404');
                            break;

                        case 500:
                            // Erreur serveur
                            notifications.create({
                                type: 'error',
                                title: 'Erreur serveur',
                                message: 'Une erreur est survenue, veuillez réessayer.'
                            });
                            break;

                        default:
                            // Autres erreurs
                            notifications.create({
                                type: 'error',
                                title: 'Erreur',
                                message: data.error || 'Une erreur est survenue.'
                            });
                    }

                    throw new Error(data.error || 'Erreur réseau');
                }

                return response;

            } catch (error) {
                if (!navigator.onLine) {
                    // Erreur de connexion
                    notifications.create({
                        type: 'error',
                        title: 'Erreur de connexion',
                        message: 'Vérifiez votre connexion internet.'
                    });
                }
                throw error;
            }
        };
    }

    /**
     * Affiche une page d'erreur
     * @param {string} type - Type d'erreur (404, 500, etc.)
     */
    showErrorPage(type) {
        // Sauvegarde l'URL actuelle pour le retour
        sessionStorage.setItem('lastUrl', window.location.href);
        
        // Redirection vers la page d'erreur
        window.location.href = `/errors/${type}.html`;
    }
}

// Export de l'instance unique
export const errorHandler = new ErrorHandler();
```

### 3.10.4 Pages d'Erreur
```html
<!-- frontend/errors/404.html -->
<!DOCTYPE html>
<html lang="fr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>FMP - Page non trouvée</title>
    <link rel="stylesheet" href="../static/css/main.css">
    <link rel="stylesheet" href="../static/css/errors.css">
</head>
<body>
    <div class="error-container">
        <div class="error-content">
            <h1>404</h1>
            <h2>Page non trouvée</h2>
            <p>La page que vous recherchez n'existe pas ou a été déplacée.</p>
            
            <div class="error-actions">
                <button onclick="window.history.back()" class="secondary-button">
                    Retour
                </button>
                <a href="/" class="primary-button">
                    Accueil
                </a>
            </div>
        </div>
    </div>
</body>
</html>
```

### 3.10.5 Styles des Pages d'Erreur
```css
/* frontend/static/css/errors.css */

.error-container {
    min-height: 100vh;
    display: flex;
    justify-content: center;
    align-items: center;
    padding: 2rem;
    background: linear-gradient(135deg, #f5f7fa 0%, #c3cfe2 100%);
}

.error-content {
    background: white;
    padding: 3rem;
    border-radius: 8px;
    box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
    text-align: center;
    max-width: 500px;
    width: 100%;
}

.error-content h1 {
    font-size: 5rem;
    color: var(--primary-color);
    margin-bottom: 1rem;
    line-height: 1;
}

.error-content h2 {
    font-size: 1.5rem;
    color: #333;
    margin-bottom: 1rem;
}

.error-content p {
    color: #666;
    margin-bottom: 2rem;
}

.error-actions {
    display: flex;
    gap: 1rem;
    justify-content: center;
}

.error-actions button,
.error-actions a {
    padding: 0.8rem 1.5rem;
    border-radius: 4px;
    font-size: 1rem;
    cursor: pointer;
    text-decoration: none;
    transition: all 0.3s;
}

.primary-button {
    background: var(--primary-color);
    color: white;
    border: none;
}

.primary-button:hover {
    background: #2980b9;
}

.secondary-button {
    background: none;
    border: 1px solid var(--primary-color);
    color: var(--primary-color);
}

.secondary-button:hover {
    background: #f8f9fa;
}

/* Animation */
.error-content {
    animation: slideIn 0.5s ease-out;
}

@keyframes slideIn {
    from {
        transform: translateY(-20px);
        opacity: 0;
    }
    to {
        transform: translateY(0);
        opacity: 1;
    }
}

/* Responsive */
@media (max-width: 480px) {
    .error-content {
        padding: 2rem;
    }

    .error-content h1 {
        font-size: 4rem;
    }

    .error-actions {
        flex-direction: column;
    }
}
```
