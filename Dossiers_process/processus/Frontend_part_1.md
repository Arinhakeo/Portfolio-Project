# Structure Frontend - FMP

## Structure des Dossiers Frontend
```
frontend/
├── static/
│   ├── css/
│   │   ├── style.css        # Styles généraux
│   │   └── responsive.css   # Styles pour le responsive design
│   ├── js/
│   │   ├── main.js         # JavaScript principal
│   │   ├── cart.js         # Logique du panier
│   │   └── api.js          # Appels à l'API
│   └── images/             # Images du site
└── pages/
    ├── index.html          # Page d'accueil
    ├── products.html       # Liste des produits
    ├── product.html        # Page détail produit
    ├── cart.html          # Panier
    └── account.html       # Espace client
```

## Page d'accueil (index.html)
```html
<!DOCTYPE html>
<html lang="fr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>FMP - For My Printer</title>
    <link rel="stylesheet" href="../static/css/style.css">
    <link rel="stylesheet" href="../static/css/responsive.css">
</head>
<body>
    <header>
        <nav class="main-nav">
            <div class="logo">
                <h1>FMP</h1>
            </div>
            <ul class="nav-links">
                <li><a href="index.html">Accueil</a></li>
                <li><a href="products.html">Cartouches</a></li>
                <li><a href="cart.html">Panier <span id="cart-count">0</span></a></li>
                <li><a href="account.html">Mon Compte</a></li>
            </ul>
        </nav>
    </header>

    <main>
        <!-- Section Hero -->
        <section class="hero">
            <h1>For My Printer</h1>
            <p>Trouvez la cartouche parfaite pour votre imprimante</p>
            <a href="products.html" class="cta-button">Voir nos produits</a>
        </section>

        <!-- Catégories populaires -->
        <section class="categories">
            <h2>Nos Catégories</h2>
            <div class="category-grid">
                <div class="category-card">
                    <img src="../static/images/hp.jpg" alt="HP">
                    <h3>HP</h3>
                </div>
                <div class="category-card">
                    <img src="../static/images/canon.jpg" alt="Canon">
                    <h3>Canon</h3>
                </div>
                <div class="category-card">
                    <img src="../static/images/epson.jpg" alt="Epson">
                    <h3>Epson</h3>
                </div>
            </div>
        </section>

        <!-- Produits en vedette -->
        <section class="featured-products">
            <h2>Produits Populaires</h2>
            <div id="products-container" class="products-grid">
                <!-- Les produits seront injectés ici via JavaScript -->
            </div>
        </section>
    </main>

    <footer>
        <div class="footer-content">
            <div class="footer-section">
                <h3>À Propos</h3>
                <p>FMP est votre solution pour toutes vos cartouches d'imprimante.</p>
            </div>
            <div class="footer-section">
                <h3>Contact</h3>
                <p>Email: contact@fmp.com</p>
                <p>Tél: 01 23 45 67 89</p>
            </div>
        </div>
        <div class="footer-bottom">
            <p>&copy; 2024 FMP - For My Printer. Tous droits réservés.</p>
        </div>
    </footer>

    <script src="../static/js/api.js"></script>
    <script src="../static/js/main.js"></script>
</body>
</html>
```

## Styles CSS (style.css)
```css
/* Reset CSS */
* {
    margin: 0;
    padding: 0;
    box-sizing: border-box;
}

body {
    font-family: Arial, sans-serif;
    line-height: 1.6;
    color: #333;
}

/* Header & Navigation */
header {
    background: #2c3e50;
    padding: 1rem 0;
    position: sticky;
    top: 0;
    z-index: 100;
}

.main-nav {
    max-width: 1200px;
    margin: 0 auto;
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding: 0 2rem;
}

.logo h1 {
    color: white;
    font-size: 1.8rem;
}

.nav-links {
    display: flex;
    list-style: none;
    gap: 2rem;
}

.nav-links a {
    color: white;
    text-decoration: none;
    font-size: 1.1rem;
    transition: color 0.3s;
}

.nav-links a:hover {
    color: #3498db;
}

/* Hero Section */
.hero {
    background: linear-gradient(rgba(0,0,0,0.5), rgba(0,0,0,0.5)), url('../images/hero-bg.jpg');
    background-size: cover;
    background-position: center;
    color: white;
    text-align: center;
    padding: 4rem 2rem;
}

.hero h1 {
    font-size: 3rem;
    margin-bottom: 1rem;
}

.cta-button {
    display: inline-block;
    background: #3498db;
    color: white;
    padding: 1rem 2rem;
    border-radius: 5px;
    text-decoration: none;
    margin-top: 1rem;
    transition: background 0.3s;
}

.cta-button:hover {
    background: #2980b9;
}

/* Grilles */
.category-grid, .products-grid {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
    gap: 2rem;
    padding: 2rem;
    max-width: 1200px;
    margin: 0 auto;
}

/* Cartes */
.category-card {
    background: white;
    border-radius: 8px;
    overflow: hidden;
    box-shadow: 0 2px 5px rgba(0,0,0,0.1);
    transition: transform 0.3s;
}

.category-card:hover {
    transform: translateY(-5px);
}

.category-card img {
    width: 100%;
    height: 200px;
    object-fit: cover;
}

/* Footer */
footer {
    background: #2c3e50;
    color: white;
    padding: 2rem 0;
    margin-top: 2rem;
}

.footer-content {
    max-width: 1200px;
    margin: 0 auto;
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
    gap: 2rem;
    padding: 0 2rem;
}

.footer-bottom {
    text-align: center;
    margin-top: 2rem;
    padding-top: 2rem;
    border-top: 1px solid rgba(255,255,255,0.1);
}
```

La suite logique serait de créer la page des produits (products.html) et son JavaScript associé pour afficher les produits depuis l'API.

Créons d'abord products.html :

```html
<!-- frontend/pages/products.html -->
<!DOCTYPE html>
<html lang="fr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>FMP - Nos Cartouches</title>
    <link rel="stylesheet" href="../static/css/style.css">
    <link rel="stylesheet" href="../static/css/responsive.css">
</head>
<body>
    <header>
        <nav class="main-nav">
            <div class="logo">
                <h1>FMP</h1>
            </div>
            <ul class="nav-links">
                <li><a href="index.html">Accueil</a></li>
                <li><a href="products.html">Cartouches</a></li>
                <li><a href="cart.html">Panier <span id="cart-count">0</span></a></li>
                <li><a href="account.html">Mon Compte</a></li>
            </ul>
        </nav>
    </header>

    <main>
        <section class="products-section">
            <div class="filters">
                <h2>Filtres</h2>
                <div class="filter-group">
                    <h3>Marque</h3>
                    <label><input type="checkbox" name="brand" value="hp"> HP</label>
                    <label><input type="checkbox" name="brand" value="canon"> Canon</label>
                    <label><input type="checkbox" name="brand" value="epson"> Epson</label>
                </div>
                <div class="filter-group">
                    <h3>Prix</h3>
                    <input type="range" id="price-range" min="0" max="100" step="5">
                    <div class="price-inputs">
                        <input type="number" id="min-price" placeholder="Min">
                        <input type="number" id="max-price" placeholder="Max">
                    </div>
                </div>
            </div>

            <div class="products-container">
                <div class="products-header">
                    <h1>Nos Cartouches</h1>
                    <div class="sort-options">
                        <select id="sort-select">
                            <option value="price-asc">Prix croissant</option>
                            <option value="price-desc">Prix décroissant</option>
                            <option value="name-asc">Nom A-Z</option>
                            <option value="name-desc">Nom Z-A</option>
                        </select>
                    </div>
                </div>

                <div id="products-grid" class="products-grid">
                    <!-- Les produits seront injectés ici via JavaScript -->
                </div>
            </div>
        </section>
    </main>

    <footer>
        <div class="footer-content">
            <div class="footer-section">
                <h3>À Propos</h3>
                <p>FMP est votre solution pour toutes vos cartouches d'imprimante.</p>
            </div>
            <div class="footer-section">
                <h3>Contact</h3>
                <p>Email: contact@fmp.com</p>
                <p>Tél: 01 23 45 67 89</p>
            </div>
        </div>
        <div class="footer-bottom">
            <p>&copy; 2024 FMP - For My Printer. Tous droits réservés.</p>
        </div>
    </footer>

    <script src="../static/js/api.js"></script>
    <script src="../static/js/products.js"></script>
</body>
</html>
```

Et ajoutons le CSS spécifique aux produits dans style.css :

```css
/* Products Page Styles */
.products-section {
    max-width: 1200px;
    margin: 2rem auto;
    display: grid;
    grid-template-columns: 250px 1fr;
    gap: 2rem;
    padding: 0 2rem;
}

.filters {
    background: white;
    padding: 1.5rem;
    border-radius: 8px;
    box-shadow: 0 2px 5px rgba(0,0,0,0.1);
}

.filter-group {
    margin-bottom: 1.5rem;
}

.filter-group h3 {
    margin-bottom: 0.5rem;
}

.filter-group label {
    display: block;
    margin: 0.5rem 0;
}

.price-inputs {
    display: grid;
    grid-template-columns: 1fr 1fr;
    gap: 1rem;
    margin-top: 0.5rem;
}

.price-inputs input {
    width: 100%;
    padding: 0.5rem;
    border: 1px solid #ddd;
    border-radius: 4px;
}

.products-header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    margin-bottom: 2rem;
}

.sort-options select {
    padding: 0.5rem;
    border-radius: 4px;
    border: 1px solid #ddd;
}

.product-card {
    background: white;
    border-radius: 8px;
    overflow: hidden;
    box-shadow: 0 2px 5px rgba(0,0,0,0.1);
    transition: transform 0.3s;
}

.product-card:hover {
    transform: translateY(-5px);
}

.product-card img {
    width: 100%;
    height: 200px;
    object-fit: cover;
}

.product-info {
    padding: 1rem;
}

.product-info h3 {
    margin-bottom: 0.5rem;
}

.product-price {
    font-size: 1.2rem;
    color: #2c3e50;
    font-weight: bold;
    margin: 0.5rem 0;
}

.add-to-cart {
    background: #3498db;
    color: white;
    border: none;
    padding: 0.5rem 1rem;
    border-radius: 4px;
    cursor: pointer;
    width: 100%;
    transition: background 0.3s;
}

.add-to-cart:hover {
    background: #2980b9;
}
```

Maintenant, créons le JavaScript pour gérer l'affichage des produits (static/js/api.js) :

```javascript
// Fonctions pour interagir avec l'API
const API_URL = 'http://127.0.0.1:5000/api';

// Fonction pour récupérer tous les produits
async function getProducts() {
    try {
        const response = await fetch(`${API_URL}/products`);
        if (!response.ok) {
            throw new Error('Erreur lors de la récupération des produits');
        }
        return await response.json();
    } catch (error) {
        console.error('Erreur:', error);
        return [];
    }
}

// Fonction pour récupérer un produit spécifique
async function getProduct(id) {
    try {
        const response = await fetch(`${API_URL}/products/${id}`);
        if (!response.ok) {
            throw new Error('Produit non trouvé');
        }
        return await response.json();
    } catch (error) {
        console.error('Erreur:', error);
        return null;
    }
}
```

Et le JavaScript spécifique à la page produits (static/js/products.js) :

```javascript
document.addEventListener('DOMContentLoaded', () => {
    loadProducts();
    setupFilters();
    setupSort();
});

// Charger et afficher les produits
async function loadProducts() {
    const products = await getProducts();
    const productsGrid = document.getElementById('products-grid');
    productsGrid.innerHTML = '';

    products.forEach(product => {
        const productCard = createProductCard(product);
        productsGrid.appendChild(productCard);
    });
}

// Créer une carte produit
function createProductCard(product) {
    const card = document.createElement('div');
    card.className = 'product-card';
    card.innerHTML = `
        <img src="${product.image || '../static/images/placeholder.jpg'}" alt="${product.name}">
        <div class="product-info">
            <h3>${product.name}</h3>
            <p>${product.description}</p>
            <div class="product-price">${product.price.toFixed(2)}€</div>
            <button class="add-to-cart" onclick="addToCart(${product.id})">
                Ajouter au panier
            </button>
        </div>
    `;
    return card;
}

// Configuration des filtres
function setupFilters() {
    const filterInputs = document.querySelectorAll('.filters input');
    filterInputs.forEach(input => {
        input.addEventListener('change', () => {
            loadProducts(); // Recharger avec les filtres
        });
    });
}

// Configuration du tri
function setupSort() {
    const sortSelect = document.getElementById('sort-select');
    sortSelect.addEventListener('change', () => {
        loadProducts(); // Recharger avec le tri
    });
}

// Fonction pour ajouter au panier
function addToCart(productId) {
    // Récupérer le panier actuel du localStorage
    let cart = JSON.parse(localStorage.getItem('cart') || '[]');
    
    // Ajouter le produit
    cart.push(productId);
    
    // Sauvegarder le panier
    localStorage.setItem('cart', JSON.stringify(cart));
    
    // Mettre à jour le compteur du panier
    updateCartCount();
    
    // Feedback utilisateur
    alert('Produit ajouté au panier !');
}

// Mettre à jour le compteur du panier
function updateCartCount() {
    const cart = JSON.parse(localStorage.getItem('cart') || '[]');
    const cartCount = document.getElementById('cart-count');
    cartCount.textContent = cart.length;
}
```
La suite logique serait de créer la page des produits (products.html) et son JavaScript associé pour afficher les produits depuis l'API.

Créons d'abord products.html :

```html
<!-- frontend/pages/products.html -->
<!DOCTYPE html>
<html lang="fr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>FMP - Nos Cartouches</title>
    <link rel="stylesheet" href="../static/css/style.css">
    <link rel="stylesheet" href="../static/css/responsive.css">
</head>
<body>
    <header>
        <nav class="main-nav">
            <div class="logo">
                <h1>FMP</h1>
            </div>
            <ul class="nav-links">
                <li><a href="index.html">Accueil</a></li>
                <li><a href="products.html">Cartouches</a></li>
                <li><a href="cart.html">Panier <span id="cart-count">0</span></a></li>
                <li><a href="account.html">Mon Compte</a></li>
            </ul>
        </nav>
    </header>

    <main>
        <section class="products-section">
            <div class="filters">
                <h2>Filtres</h2>
                <div class="filter-group">
                    <h3>Marque</h3>
                    <label><input type="checkbox" name="brand" value="hp"> HP</label>
                    <label><input type="checkbox" name="brand" value="canon"> Canon</label>
                    <label><input type="checkbox" name="brand" value="epson"> Epson</label>
                </div>
                <div class="filter-group">
                    <h3>Prix</h3>
                    <input type="range" id="price-range" min="0" max="100" step="5">
                    <div class="price-inputs">
                        <input type="number" id="min-price" placeholder="Min">
                        <input type="number" id="max-price" placeholder="Max">
                    </div>
                </div>
            </div>

            <div class="products-container">
                <div class="products-header">
                    <h1>Nos Cartouches</h1>
                    <div class="sort-options">
                        <select id="sort-select">
                            <option value="price-asc">Prix croissant</option>
                            <option value="price-desc">Prix décroissant</option>
                            <option value="name-asc">Nom A-Z</option>
                            <option value="name-desc">Nom Z-A</option>
                        </select>
                    </div>
                </div>

                <div id="products-grid" class="products-grid">
                    <!-- Les produits seront injectés ici via JavaScript -->
                </div>
            </div>
        </section>
    </main>

    <footer>
        <div class="footer-content">
            <div class="footer-section">
                <h3>À Propos</h3>
                <p>FMP est votre solution pour toutes vos cartouches d'imprimante.</p>
            </div>
            <div class="footer-section">
                <h3>Contact</h3>
                <p>Email: contact@fmp.com</p>
                <p>Tél: 01 23 45 67 89</p>
            </div>
        </div>
        <div class="footer-bottom">
            <p>&copy; 2024 FMP - For My Printer. Tous droits réservés.</p>
        </div>
    </footer>

    <script src="../static/js/api.js"></script>
    <script src="../static/js/products.js"></script>
</body>
</html>
```

Et ajoutons le CSS spécifique aux produits dans style.css :

```css
/* Products Page Styles */
.products-section {
    max-width: 1200px;
    margin: 2rem auto;
    display: grid;
    grid-template-columns: 250px 1fr;
    gap: 2rem;
    padding: 0 2rem;
}

.filters {
    background: white;
    padding: 1.5rem;
    border-radius: 8px;
    box-shadow: 0 2px 5px rgba(0,0,0,0.1);
}

.filter-group {
    margin-bottom: 1.5rem;
}

.filter-group h3 {
    margin-bottom: 0.5rem;
}

.filter-group label {
    display: block;
    margin: 0.5rem 0;
}

.price-inputs {
    display: grid;
    grid-template-columns: 1fr 1fr;
    gap: 1rem;
    margin-top: 0.5rem;
}

.price-inputs input {
    width: 100%;
    padding: 0.5rem;
    border: 1px solid #ddd;
    border-radius: 4px;
}

.products-header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    margin-bottom: 2rem;
}

.sort-options select {
    padding: 0.5rem;
    border-radius: 4px;
    border: 1px solid #ddd;
}

.product-card {
    background: white;
    border-radius: 8px;
    overflow: hidden;
    box-shadow: 0 2px 5px rgba(0,0,0,0.1);
    transition: transform 0.3s;
}

.product-card:hover {
    transform: translateY(-5px);
}

.product-card img {
    width: 100%;
    height: 200px;
    object-fit: cover;
}

.product-info {
    padding: 1rem;
}

.product-info h3 {
    margin-bottom: 0.5rem;
}

.product-price {
    font-size: 1.2rem;
    color: #2c3e50;
    font-weight: bold;
    margin: 0.5rem 0;
}

.add-to-cart {
    background: #3498db;
    color: white;
    border: none;
    padding: 0.5rem 1rem;
    border-radius: 4px;
    cursor: pointer;
    width: 100%;
    transition: background 0.3s;
}

.add-to-cart:hover {
    background: #2980b9;
}
```

Maintenant, créons le JavaScript pour gérer l'affichage des produits (static/js/api.js) :

```javascript
// Fonctions pour interagir avec l'API
const API_URL = 'http://localhost:5000/api';

// Fonction pour récupérer tous les produits
async function getProducts() {
    try {
        const response = await fetch(`${API_URL}/products`);
        if (!response.ok) {
            throw new Error('Erreur lors de la récupération des produits');
        }
        return await response.json();
    } catch (error) {
        console.error('Erreur:', error);
        return [];
    }
}

// Fonction pour récupérer un produit spécifique
async function getProduct(id) {
    try {
        const response = await fetch(`${API_URL}/products/${id}`);
        if (!response.ok) {
            throw new Error('Produit non trouvé');
        }
        return await response.json();
    } catch (error) {
        console.error('Erreur:', error);
        return null;
    }
}
```

Et le JavaScript spécifique à la page produits (static/js/products.js) :

```javascript
document.addEventListener('DOMContentLoaded', () => {
    loadProducts();
    setupFilters();
    setupSort();
});

// Charger et afficher les produits
async function loadProducts() {
    const products = await getProducts();
    const productsGrid = document.getElementById('products-grid');
    productsGrid.innerHTML = '';

    products.forEach(product => {
        const productCard = createProductCard(product);
        productsGrid.appendChild(productCard);
    });
}

// Créer une carte produit
function createProductCard(product) {
    const card = document.createElement('div');
    card.className = 'product-card';
    card.innerHTML = `
        <img src="${product.image || '../static/images/placeholder.jpg'}" alt="${product.name}">
        <div class="product-info">
            <h3>${product.name}</h3>
            <p>${product.description}</p>
            <div class="product-price">${product.price.toFixed(2)}€</div>
            <button class="add-to-cart" onclick="addToCart(${product.id})">
                Ajouter au panier
            </button>
        </div>
    `;
    return card;
}

// Configuration des filtres
function setupFilters() {
    const filterInputs = document.querySelectorAll('.filters input');
    filterInputs.forEach(input => {
        input.addEventListener('change', () => {
            loadProducts(); // Recharger avec les filtres
        });
    });
}

// Configuration du tri
function setupSort() {
    const sortSelect = document.getElementById('sort-select');
    sortSelect.addEventListener('change', () => {
        loadProducts(); // Recharger avec le tri
    });
}

// Fonction pour ajouter au panier
function addToCart(productId) {
    // Récupérer le panier actuel du localStorage
    let cart = JSON.parse(localStorage.getItem('cart') || '[]');
    
    // Ajouter le produit
    cart.push(productId);
    
    // Sauvegarder le panier
    localStorage.setItem('cart', JSON.stringify(cart));
    
    // Mettre à jour le compteur du panier
    updateCartCount();
    
    // Feedback utilisateur
    alert('Produit ajouté au panier !');
}

// Mettre à jour le compteur du panier
function updateCartCount() {
    const cart = JSON.parse(localStorage.getItem('cart') || '[]');
    const cartCount = document.getElementById('cart-count');
    cartCount.textContent = cart.length;
}
```
