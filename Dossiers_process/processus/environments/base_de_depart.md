# Documentation du Projet FMP - For My Printer

## Structure du Projet

```
frontend/
│
├── static/
│   ├── css/
│   │   └── style.css          # Fichier CSS pour le style de la page
│   └── js/
│       └── main.js            # Fichier JavaScript pour les fonctionnalités dynamiques
│
└── pages/
    └── index.html             # Page d'accueil de l'application
│
backend/
│
├── app/
│   ├── __init__.py            # Initialisation de l'application Flask
│
├── config.py                  # Configuration de l'application
│
└── run.py                     # Point d'entrée pour démarrer l'application
```

## 1. Page d'accueil (frontend/pages/index.html)

```html
<!DOCTYPE html>
<html lang="fr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>FMP - For My Printer</title>
    <link rel="stylesheet" href="../static/css/style.css">  <!-- Lien vers le fichier CSS -->
</head>
<body>
    <header>
        <h1>FMP - For My Printer</h1>  <!-- Titre principal de la page -->
        <nav>
            <ul>
                <li><a href="index.html">Accueil</a></li>   <!-- Lien vers la page d'accueil -->
                <li><a href="products.html">Produits</a></li>  <!-- Lien vers la page des produits -->
                <li><a href="cart.html">Panier</a></li>   <!-- Lien vers la page du panier -->
            </ul>
        </nav>
    </header>

    <main>
        <h2>Bienvenue sur FMP</h2>  <!-- Sous-titre de bienvenue -->
        <p>Votre destination pour les cartouches d'imprimante</p>  <!-- Description du service -->
    </main>

    <footer>
        <p>&copy; 2024 FMP - For My Printer</p>  <!-- Informations sur le copyright -->
    </footer>

    <script src="../static/js/main.js"></script>  <!-- Lien vers le fichier JavaScript -->
</body>
</html>
```

## 2. Styles de base (frontend/static/css/style.css)

```css
/* Reset de base */
* {
    margin: 0;  /* Supprime les marges par défaut */
    padding: 0; /* Supprime les remplissages par défaut */
    box-sizing: border-box; /* Inclut les bordures et les remplissages dans la largeur et la hauteur totales */
}

body {
    font-family: Arial, sans-serif; /* Définit la police du corps */
    line-height: 1.6; /* Définit l'espacement entre les lignes */
}

header {
    background-color: #333; /* Couleur de fond de l'en-tête */
    color: white; /* Couleur du texte dans l'en-tête */
    padding: 1rem; /* Espace intérieur de l'en-tête */
}

nav ul {
    list-style: none; /* Supprime les puces de la liste de navigation */
}

nav ul li {
    display: inline; /* Affiche les éléments de la liste en ligne */
    margin-right: 1rem; /* Espace à droite de chaque élément */
}

nav ul li a {
    color: white; /* Couleur des liens de navigation */
    text-decoration: none; /* Supprime le soulignement des liens */
}

main {
    padding: 2rem; /* Espace intérieur pour le contenu principal */
}

footer {
    background-color: #333; /* Couleur de fond du pied de page */
    color: white; /* Couleur du texte dans le pied de page */
    text-align: center; /* Centre le texte dans le pied de page */
    padding: 1rem; /* Espace intérieur du pied de page */
    position: fixed; /* Fixe le pied de page au bas de la fenêtre */
    bottom: 0; /* Positionne le pied de page en bas */
    width: 100%; /* S'assure que le pied de page occupe toute la largeur */
}
```

## 3. Configuration Flask (backend/app/__init__.py)

```python
from flask import Flask  # Importation de la classe Flask
from flask_sqlalchemy import SQLAlchemy  # Importation de SQLAlchemy pour la gestion de la base de données
from config import Config  # Importation de la configuration de l'application

db = SQLAlchemy()  # Création d'une instance de SQLAlchemy

def create_app():
    app = Flask(__name__)  # Création de l'application Flask
    app.config.from_object(Config)  # Chargement de la configuration à partir de la classe Config

    db.init_app(app)  # Initialisation de SQLAlchemy avec l'application Flask

    @app.route('/')  # Définition de la route pour la page d'accueil
    def index():
        return {'message': 'FMP API is running'}  # Renvoie un message JSON lorsque la route est accédée

    return app  # Retourne l'application Flask configurée
```

## 4. Configuration de base (backend/config.py)

```python
import os  # Importation du module os pour les interactions avec le système d'exploitation
from dotenv import load_dotenv  # Importation pour charger les variables d'environnement à partir d'un fichier .env

load_dotenv()  # Chargement des variables d'environnement

class Config:
    SECRET_KEY = os.environ.get('SECRET_KEY') or 'dev-key-change-me'  # Clé secrète pour la sécurité de l'application
    SQLALCHEMY_DATABASE_URI = 'sqlite:///fmp.db'  # URI de connexion à la base de données SQLite
    SQLALCHEMY_TRACK_MODIFICATIONS = False  # Désactive les notifications d'événements de modifications
```

## 5. Point d'entrée (backend/run.py)

```python
from app import create_app  # Importation de la fonction create_app depuis le module app

app = create_app()  # Création de l'application Flask

if __name__ == '__main__':  # Vérifie si ce script est exécuté directement
    app.run(debug=True)  # Démarre le serveur Flask en mode debug
```

## Synthèse des Fonctions

Voici un diagramme simplifié pour mieux comprendre les interactions et le flux de l'application :
```mermaid
flowchart TD
    A[Page d'accueil (index.html)] -->|Lien vers| B[Produits (products.html)]
    A -->|Lien vers| C[Panier (cart.html)]
    D[Style CSS (style.css)] --> A
    E[JavaScript (main.js)] --> A
    F[Application Flask (app/__init__.py)] -->|Route| G[Message JSON]
    H[Configuration (config.py)] -->|URI DB| F
    F --> I[Base de données SQLite]
    J[run.py] -->|Démarre| F
```

## Explications :

- **Frontend** :
  - **index.html** : La page d'accueil qui contient des liens vers d'autres pages.
  - **style.css** : Fichier CSS pour le style de la page, appliqué à l'ensemble des éléments HTML.
  - **main.js** : Fichier JavaScript pour toute fonctionnalité dynamique (non spécifié ici mais peut être utilisé pour interagir avec l'API).

- **Backend** :
  - **app/__init__.py** : Initialise l'application Flask, configure la base de données et définit les routes.
  - **config.py** : Contient les configurations de l'application, comme la clé secrète et l'URI de la base de données.
  - **run.py** : Point d'entrée du serveur qui démarre l'application Flask.

Cette documentation devrait vous aider à mieux comprendre la structure et le fonctionnement de votre application. Si vous avez besoin de détails supplémentaires ou d'éclaircissements, n'hésitez pas à demander !
