# Frontend FMP - Partie 3 : Détail Produit
## Guide Complet de Développement

### Structure des fichiers pour cette partie
```
frontend/
├── pages/
│   └── product.html          # Page détail produit
├── static/
│   ├── css/
│   │   ├── style.css        # Styles globaux
│   │   └── product.css      # Styles spécifiques au produit
│   └── js/
│       ├── api.js           # Fonctions API
│       └── product.js       # Logique du détail produit
```

### 1. HTML - product.html
```html
<!DOCTYPE html>
<html lang="fr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>FMP - Détail Produit</title>
    <!-- Import des styles -->
    <link rel="stylesheet" href="../static/css/style.css">
    <link rel="stylesheet" href="../static/css/product.css">
</head>
<body>
    <!-- En-tête avec navigation -->
    <header>
        <nav class="main-nav">
            <div class="logo">
                <a href="index.html">
                    <img src="../static/images/logo.png" alt="FMP Logo">
                </a>
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
        <!-- Fil d'Ariane -->
        <div class="breadcrumb">
            <a href="index.html">Accueil</a> >
            <a href="products.html">Cartouches</a> >
            <span id="product-breadcrumb">Nom du produit</span>
        </div>

        <!-- Section principale du produit -->
        <section class="product-detail">
            <!-- Galerie d'images -->
            <div class="product-gallery">
                <div class="main-image">
                    <img id="main-product-image" src="" alt="">
                    <div class="zoom-lens"></div>
                </div>
                <div class="thumbnail-images" id="thumbnail-container">
                    <!-- Miniatures ajoutées en JS -->
                </div>
            </div>

            <!-- Informations produit -->
            <div class="product-info">
                <h1 id="product-name"></h1>
                
                <!-- Méta-informations -->
                <div class="product-meta">
                    <span>Référence : <span id="product-ref"></span></span>
                    <span>Marque : <span id="product-brand"></span></span>
                    <div class="product-rating">
                        <div class="stars"></div>
                        <span class="reviews-count"></span>
                    </div>
                </div>

                <!-- Prix et stock -->
                <div class="product-price-section">
                    <div class="product-price" id="product-price"></div>
                    <div class="stock-status" id="stock-status"></div>
                    <div class="delivery-info">
                        <i class="icon-truck"></i>
                        Livraison gratuite dès 49€
                    </div>
                </div>

                <!-- Options d'achat -->
                <div class="purchase-options">
                    <div class="quantity-selector">
                        <button class="quantity-btn minus">-</button>
                        <input type="number" value="1" min="1" max="99" id="quantity-input">
                        <button class="quantity-btn plus">+</button>
                    </div>
                    <button class="add-to-cart-btn" id="add-to-cart">
                        <i class="icon-cart"></i>
                        Ajouter au panier
                    </button>
                    <button class="wishlist-btn" id="wishlist-btn">
                        <i class="icon-heart"></i>
                    </button>
                </div>

                <!-- Description et caractéristiques -->
                <div class="product-tabs">
                    <div class="tabs-header">
                        <button class="tab-btn active" data-tab="description">
                            Description
                        </button>
                        <button class="tab-btn" data-tab="specifications">
                            Caractéristiques
                        </button>
                        <button class="tab-btn" data-tab="reviews">
                            Avis clients
                        </button>
                    </div>

                    <div class="tabs-content">
                        <div id="description" class="tab-content active"></div>
                        <div id="specifications" class="tab-content"></div>
                        <div id="reviews" class="tab-content"></div>
                    </div>
                </div>
            </div>
        </section>

        <!-- Produits similaires -->
        <section class="related-products">
            <h2>Produits similaires</h2>
            <div class="products-grid" id="related-products"></div>
        </section>
    </main>

    <footer>
        <!-- Contenu du footer -->
    </footer>

    <!-- Scripts -->
    <script src="../static/js/api.js"></script>
    <script src="../static/js/product.js"></script>
</body>
</html>
```

### 2. CSS - product.css
```css
/* Style pour la page détail produit */

/* Conteneur principal */
.product-detail {
    max-width: 1200px;
    margin: 2rem auto;
    padding: 0 1rem;
    display: grid;
    grid-template-columns: 1fr 1fr;
    gap: 2rem;
}

/* Fil d'Ariane */
.breadcrumb {
    max-width: 1200px;
    margin: 1rem auto;
    padding: 0 1rem;
    font-size: 0.9rem;
}

.breadcrumb a {
    color: #666;
    text-decoration: none;
}

.breadcrumb a:hover {
    color: #333;
}

/* Galerie d'images */
.product-gallery {
    position: sticky;
    top: 2rem;
}

.main-image {
    position: relative;
    width: 100%;
    border: 1px solid #eee;
    border-radius: 8px;
    overflow: hidden;
}

.main-image img {
    width: 100%;
    height: auto;
    display: block;
}

.thumbnail-images {
    display: flex;
    gap: 0.5rem;
    margin-top: 1rem;
}

.thumbnail {
    width: 80px;
    height: 80px;
    border: 1px solid #eee;
    border-radius: 4px;
    cursor: pointer;
    transition: border-color 0.3s;
}

.thumbnail:hover,
.thumbnail.active {
    border-color: #3498db;
}

/* Informations produit */
.product-info h1 {
    font-size: 2rem;
    margin-bottom: 1rem;
}

.product-meta {
    display: flex;
    flex-wrap: wrap;
    gap: 1rem;
    color: #666;
    margin-bottom: 1.5rem;
}

.product-rating {
    display: flex;
    align-items: center;
    gap: 0.5rem;
}

.stars {
    color: #f1c40f;
}

/* Prix et stock */
.product-price-section {
    margin: 2rem 0;
    padding: 1rem;
    background: #f8f9fa;
    border-radius: 8px;
}

.product-price {
    font-size: 2rem;
    font-weight: bold;
    color: #2c3e50;
}

.stock-status {
    display: inline-block;
    padding: 0.25rem 0.5rem;
    border-radius: 4px;
    font-size: 0.9rem;
    margin: 0.5rem 0;
}

.stock-status.in-stock {
    background: #27ae60;
    color: white;
}

.stock-status.out-of-stock {
    background: #e74c3c;
    color: white;
}

/* Options d'achat */
.purchase-options {
    display: flex;
    gap: 1rem;
    margin: 2rem 0;
}

.quantity-selector {
    display: flex;
    align-items: center;
    border: 1px solid #ddd;
    border-radius: 4px;
    overflow: hidden;
}

.quantity-btn {
    padding: 0.5rem 1rem;
    border: none;
    background: #f8f9fa;
    cursor: pointer;
}

.quantity-btn:hover {
    background: #e9ecef;
}

#quantity-input {
    width: 60px;
    text-align: center;
    border: none;
    border-left: 1px solid #ddd;
    border-right: 1px solid #ddd;
}

.add-to-cart-btn {
    flex: 1;
    background: #3498db;
    color: white;
    border: none;
    border-radius: 4px;
    padding: 0.5rem 1rem;
    cursor: pointer;
    transition: background 0.3s;
}

.add-to-cart-btn:hover {
    background: #2980b9;
}

/* Onglets */
.product-tabs {
    margin-top: 3rem;
}

.tabs-header {
    display: flex;
    gap: 1rem;
    border-bottom: 1px solid #ddd;
}

.tab-btn {
    padding: 1rem 2rem;
    border: none;
    background: none;
    cursor: pointer;
    font-size: 1rem;
    color: #666;
    position: relative;
}

.tab-btn.active {
    color: #333;
    font-weight: bold;
}

.tab-btn.active::after {
    content: '';
    position: absolute;
    bottom: -1px;
    left: 0;
    right: 0;
    height: 2px;
    background: #3498db;
}

.tab-content {
    display: none;
    padding: 2rem 0;
}

.tab-content.active {
    display: block;
}

/* Produits similaires */
.related-products {
    max-width: 1200px;
    margin: 4rem auto;
    padding: 0 1rem;
}

.related-products h2 {
    margin-bottom: 2rem;
}

.products-grid {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
    gap: 2rem;
}

/* Responsive design */
@media (max-width: 768px) {
    .product-detail {
        grid-template-columns: 1fr;
    }

    .product-gallery {
        position: static;
    }

    .thumbnail {
        width: 60px;
        height: 60px;
    }

    .tabs-header {
        flex-wrap: wrap;
    }

    .tab-btn {
        padding: 0.5rem 1rem;
    }
}
```

### 3. JavaScript - product.js
```javascript
// Variables globales
let currentProductId = null;
let currentProduct = null;

// Chargement initial
document.addEventListener('DOMContentLoaded', () => {
    // Récupérer l'ID du produit depuis l'URL
    const urlParams = new URLSearchParams(window.location.search);
    currentProductId = urlParams.get('id');

    if (currentProductId) {
        loadProductDetails(currentProductId);
        setupEventListeners();
    } else {
        showError('Produit non trouvé');
    }
});

// Charger les détails du produit
async function loadProductDetails(productId) {
    try {
        // Appel API
        const response = await fetch(`/api/products/${productId}`);
        if (!response.ok) throw new Error('Produit non trouvé');
        
        currentProduct = await response.json();
        
        // Mise à jour de l'interface
        updateProductDetails(currentProduct);
        updateBreadcrumb(currentProduct);
        loadRelatedProducts(currentProduct.category);
        
    } catch (error) {
        showError(error.message);
    }
}

// Mise à jour des détails du produit
function updateProductDetails(product) {
    // Titre de la page
    document.title = `FMP - ${product.name}`;
    
    // Informations principales
    document.getElementById('product-name').textContent = product.name;
    document.getElementById('product-ref').textContent = product.reference;
    document.getElementById('product-brand').textContent = product.brand;
    document.getElementById('product-price').textContent = 
        `${product.price.toFixed(2)} €`;
    
    // Image principale
    const mainImage = document.getElementById('main-product-image');
    mainImage.src = product.image;
    mainImage.alt = product.name;
    
    // Statut du stock
    updateStockStatus(product.stock);
    
    // Description et caractéristiques
    document.getElementById('description').innerHTML = product.description;
    updateSpecifications(product.specifications);
}

// Mise à jour du fil d'Ariane
function updateBreadcrumb(product) {
    document.getElementById('product-breadcrumb').textContent = product.name;
}

// Mise à jour du statut de stock
function updateStockStatus(stock) {
    const stockStatus = document.getElementById('stock-status');
    if (stock > 0) {
        stockStatus.textContent = 'En stock';
        stockStatus.className = 'stock-status in-stock';
    } else {
        stockStatus.textContent = 'Rupture de stock';
        stockStatus.className = 'stock-status out-of-stock';
    }
}

// Configuration des écouteurs d'événements
function setupEventListeners() {
    // Gestion de la quantité
    setupQuantityControls();
    
    // Gestion des onglets
    setupTabs();
    
    // Gestion du panier
    setupCartButton();
}

// Gestion de la quantité
function setupQuantityControls() {
    const minusBtn = document.querySelector('.quantity-btn.minus');
    const plusBtn = document.querySelector('.quantity-btn.plus');
    const quantityInput = document.getElementById('quantity-input');

    minusBtn.addEventListener('click', () => {
        const currentValue = parseInt(quantityInput.value);
        if (currentValue > 1) {
            quantityInput.value = currentValue - 1;
        }
    });

    plusBtn.addEventListener('click', () => {
        const currentValue = parseInt(quantityInput.value);
        if (currentValue < currentProduct.stock) {
            quantityInput.value = currentValue + 1;
        }
    });

    // Validation de l'input manuel
    quantityInput.addEventListener('change', () => {
        let value = parseInt(quantityInput.value);
        if (isNaN(value) || value < 1) {
            value = 1;
        } else if (value > currentProduct.stock) {
            value = currentProduct.stock;
        }
        quantityInput.value = value;
    });
}

// Gestion des onglets
function setupTabs() {
    const tabButtons = document.querySelectorAll('.tab-btn');
    const tabContents = document.querySelectorAll('.tab-content');

    tabButtons.forEach(button => {
        button.addEventListener('click', () => {
            // Retirer la classe active de tous les boutons
            tabButtons.forEach(btn => btn.classList.remove('active'));
            // Cacher tous les contenus
            tabContents.forEach(content => content.classList.remove('active'));

            // Activer le bouton cliqué
            button.classList.add('active');
            // Afficher le contenu correspondant
            const tabId = button.getAttribute('data-tab');
            document.getElementById(tabId).classList.add('active');
        });
    });
}

// Gestion du panier
function setupCartButton() {
    const addToCartBtn = document.getElementById('add-to-cart');
    addToCartBtn.addEventListener('click', () => {
        const quantity = parseInt(document.getElementById('quantity-input').value);
        addToCart(currentProduct, quantity);
    });
}

// Fonction d'ajout au panier
function addToCart(product, quantity) {
    // Récupérer le panier actuel
    let cart = JSON.parse(localStorage.getItem('cart') || '[]');
    
    // Vérifier si le produit est déjà dans le panier
    const existingItemIndex = cart.findIndex(item => item.id === product.id);
    
    if (existingItemIndex > -1) {
        // Mettre à jour la quantité si le produit existe déjà
        cart[existingItemIndex].quantity += quantity;
    } else {
        // Ajouter le nouveau produit
        cart.push({
            id: product.id,
            name: product.name,
            price: product.price,
            quantity: quantity,
            image: product.image
        });
    }
    
    // Sauvegarder le panier
    localStorage.setItem('cart', JSON.stringify(cart));
    
    // Mettre à jour l'affichage
    updateCartCount();
    showAddToCartConfirmation();
}

// Mise à jour du compteur panier
function updateCartCount() {
    const cart = JSON.parse(localStorage.getItem('cart') || '[]');
    const totalItems = cart.reduce((total, item) => total + item.quantity, 0);
    const cartCount = document.getElementById('cart-count');
    cartCount.textContent = totalItems;
}

// Afficher confirmation d'ajout au panier
function showAddToCartConfirmation() {
    // Créer la notification
    const notification = document.createElement('div');
    notification.className = 'cart-notification';
    notification.innerHTML = `
        <div class="notification-content">
            <i class="icon-check"></i>
            <p>Produit ajouté au panier</p>
        </div>
        <div class="notification-actions">
            <a href="cart.html" class="view-cart">Voir le panier</a>
            <button class="continue-shopping">Continuer mes achats</button>
        </div>
    `;
    
    document.body.appendChild(notification);
    
    // Animer l'entrée
    setTimeout(() => {
        notification.classList.add('show');
    }, 100);
    
    // Supprimer après 3 secondes
    setTimeout(() => {
        notification.classList.remove('show');
        setTimeout(() => {
            notification.remove();
        }, 300);
    }, 3000);
}

// Produits similaires
async function loadRelatedProducts(category) {
    try {
        // Appel API pour obtenir les produits de la même catégorie
        const response = await fetch(`/api/products?category=${category}`);
        const products = await response.json();
        
        // Filtrer pour exclure le produit actuel
        const relatedProducts = products.filter(p => p.id !== currentProduct.id);
        
        // Afficher maximum 4 produits similaires
        const container = document.getElementById('related-products');
        container.innerHTML = '';
        
        relatedProducts.slice(0, 4).forEach(product => {
            container.appendChild(createRelatedProductCard(product));
        });
        
    } catch (error) {
        console.error('Erreur chargement produits similaires:', error);
    }
}

// Créer une carte de produit similaire
function createRelatedProductCard(product) {
    const card = document.createElement('div');
    card.className = 'product-card';
    card.innerHTML = `
        <a href="product.html?id=${product.id}">
            <img src="${product.image}" alt="${product.name}">
            <div class="product-card-info">
                <h3>${product.name}</h3>
                <p class="product-card-price">${product.price.toFixed(2)} €</p>
            </div>
        </a>
    `;
    return card;
}

// Gestion des erreurs
function showError(message) {
    const errorContainer = document.createElement('div');
    errorContainer.className = 'error-container';
    errorContainer.innerHTML = `
        <div class="error-content">
            <h2>Erreur</h2>
            <p>${message}</p>
            <a href="products.html" class="error-button">
                Retour à la liste des produits
            </a>
        </div>
    `;
    
    // Remplacer le contenu principal
    const main = document.querySelector('main');
    main.innerHTML = '';
    main.appendChild(errorContainer);
}

// Fonction d'initialisation
function init() {
    updateCartCount();
    setupEventListeners();
}

// Appeler l'initialisation au chargement
document.addEventListener('DOMContentLoaded', init);
```

### 4. Styles complémentaires (product.css)
```css
/* Styles pour les notifications */
.cart-notification {
    position: fixed;
    bottom: -100px;
    right: 20px;
    background: white;
    box-shadow: 0 2px 10px rgba(0,0,0,0.1);
    border-radius: 8px;
    padding: 1rem;
    transition: transform 0.3s ease-out;
    z-index: 1000;
}

.cart-notification.show {
    transform: translateY(-120px);
}

.notification-content {
    display: flex;
    align-items: center;
    gap: 1rem;
    margin-bottom: 1rem;
}

.notification-actions {
    display: flex;
    gap: 1rem;
}

.view-cart {
    background: #3498db;
    color: white;
    padding: 0.5rem 1rem;
    border-radius: 4px;
    text-decoration: none;
}

.continue-shopping {
    background: none;
    border: 1px solid #3498db;
    color: #3498db;
    padding: 0.5rem 1rem;
    border-radius: 4px;
    cursor: pointer;
}

/* Styles pour les erreurs */
.error-container {
    max-width: 600px;
    margin: 4rem auto;
    text-align: center;
}

.error-content {
    background: white;
    padding: 2rem;
    border-radius: 8px;
    box-shadow: 0 2px 10px rgba(0,0,0,0.1);
}

.error-button {
    display: inline-block;
    background: #3498db;
    color: white;
    padding: 0.8rem 1.5rem;
    border-radius: 4px;
    text-decoration: none;
    margin-top: 1.5rem;
}
```

Cette partie complète la Partie 3 du frontend avec :
- La gestion complète de la quantité
- Les notifications d'ajout au panier
- L'affichage des produits similaires
- La gestion des erreurs
- Les styles complémentaires

Voulez-vous que je passe à la Partie 4 qui concerne l'espace client et l'authentification ?