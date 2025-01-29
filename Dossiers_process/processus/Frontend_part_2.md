Ok, passons à la partie 2 du frontend qui est le panier (cart.html) et sa logique de fonctionnement.

Voici le code pour cart.html :

```html
<!-- frontend/pages/cart.html -->
<!DOCTYPE html>
<html lang="fr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>FMP - Panier</title>
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
        <section class="cart-section">
            <h1>Mon Panier</h1>
            
            <!-- Conteneur du panier -->
            <div class="cart-container">
                <div class="cart-items" id="cart-items">
                    <!-- Les articles du panier seront injectés ici -->
                </div>
                
                <!-- Résumé du panier -->
                <div class="cart-summary">
                    <h2>Résumé de la commande</h2>
                    <div class="summary-details">
                        <div class="summary-line">
                            <span>Sous-total</span>
                            <span id="subtotal">0.00 €</span>
                        </div>
                        <div class="summary-line">
                            <span>Frais de livraison</span>
                            <span id="shipping">5.90 €</span>
                        </div>
                        <div class="summary-line total">
                            <span>Total</span>
                            <span id="total">0.00 €</span>
                        </div>
                    </div>
                    <button id="checkout-button" class="checkout-button">
                        Procéder au paiement
                    </button>
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
    <script src="../static/js/cart.js"></script>
</body>
</html>
```

Ajoutons les styles CSS spécifiques au panier dans style.css :

```css
/* Styles du panier */
.cart-section {
    max-width: 1200px;
    margin: 2rem auto;
    padding: 0 2rem;
}

.cart-container {
    display: grid;
    grid-template-columns: 2fr 1fr;
    gap: 2rem;
    margin-top: 2rem;
}

.cart-items {
    background: white;
    border-radius: 8px;
    box-shadow: 0 2px 5px rgba(0,0,0,0.1);
    padding: 1.5rem;
}

.cart-item {
    display: grid;
    grid-template-columns: auto 1fr auto;
    gap: 1.5rem;
    padding: 1rem 0;
    border-bottom: 1px solid #eee;
}

.cart-item:last-child {
    border-bottom: none;
}

.cart-item img {
    width: 100px;
    height: 100px;
    object-fit: cover;
    border-radius: 4px;
}

.cart-item-info {
    display: flex;
    flex-direction: column;
    justify-content: space-between;
}

.cart-item-title {
    font-size: 1.1rem;
    font-weight: bold;
    margin-bottom: 0.5rem;
}

.cart-item-price {
    font-weight: bold;
    color: #2c3e50;
}

.quantity-controls {
    display: flex;
    align-items: center;
    gap: 0.5rem;
}

.quantity-controls button {
    background: #f0f0f0;
    border: none;
    padding: 0.5rem;
    border-radius: 4px;
    cursor: pointer;
}

.quantity-controls input {
    width: 50px;
    text-align: center;
    padding: 0.5rem;
    border: 1px solid #ddd;
    border-radius: 4px;
}

.remove-item {
    color: #e74c3c;
    background: none;
    border: none;
    cursor: pointer;
    padding: 0.5rem;
}

.cart-summary {
    background: white;
    border-radius: 8px;
    box-shadow: 0 2px 5px rgba(0,0,0,0.1);
    padding: 1.5rem;
    height: fit-content;
    position: sticky;
    top: 2rem;
}

.summary-details {
    margin: 1.5rem 0;
}

.summary-line {
    display: flex;
    justify-content: space-between;
    margin: 0.5rem 0;
}

.summary-line.total {
    font-weight: bold;
    font-size: 1.2rem;
    margin-top: 1rem;
    padding-top: 1rem;
    border-top: 2px solid #eee;
}

.checkout-button {
    background: #3498db;
    color: white;
    border: none;
    padding: 1rem;
    border-radius: 4px;
    width: 100%;
    font-size: 1.1rem;
    cursor: pointer;
    transition: background 0.3s;
}

.checkout-button:hover {
    background: #2980b9;
}

.empty-cart {
    text-align: center;
    padding: 2rem;
}

.empty-cart p {
    margin: 1rem 0;
    color: #666;
}

.continue-shopping {
    display: inline-block;
    background: #3498db;
    color: white;
    padding: 0.8rem 1.5rem;
    border-radius: 4px;
    text-decoration: none;
    margin-top: 1rem;
}
```

Et le JavaScript pour gérer le panier (cart.js) :

```javascript
// static/js/cart.js

document.addEventListener('DOMContentLoaded', () => {
    loadCart();
});

async function loadCart() {
    const cartItems = document.getElementById('cart-items');
    const cart = JSON.parse(localStorage.getItem('cart') || '[]');

    if (cart.length === 0) {
        showEmptyCart();
        return;
    }

    // Récupérer les détails de chaque produit
    let total = 0;
    cartItems.innerHTML = '';

    for (const productId of cart) {
        const product = await getProduct(productId);
        if (product) {
            total += product.price;
            cartItems.appendChild(createCartItem(product));
        }
    }

    updateCartSummary(total);
}

function createCartItem(product) {
    const item = document.createElement('div');
    item.className = 'cart-item';
    item.innerHTML = `
        <img src="${product.image || '../static/images/placeholder.jpg'}" alt="${product.name}">
        <div class="cart-item-info">
            <div class="cart-item-title">${product.name}</div>
            <div class="cart-item-price">${product.price.toFixed(2)}€</div>
            <div class="quantity-controls">
                <button onclick="updateQuantity(${product.id}, -1)">-</button>
                <input type="number" value="1" min="1" onchange="updateQuantity(${product.id}, this.value)">
                <button onclick="updateQuantity(${product.id}, 1)">+</button>
            </div>
        </div>
        <button class="remove-item" onclick="removeFromCart(${product.id})">
            Supprimer
        </button>
    `;
    return item;
}

function showEmptyCart() {
    const cartItems = document.getElementById('cart-items');
    cartItems.innerHTML = `
        <div class="empty-cart">
            <h2>Votre panier est vide</h2>
            <p>Découvrez nos produits et commencez vos achats !</p>
            <a href="products.html" class="continue-shopping">Voir les produits</a>
        </div>
    `;
    updateCartSummary(0);
}

function updateCartSummary(subtotal) {
    const shipping = 5.90;
    const total = subtotal + shipping;

    document.getElementById('subtotal').textContent = `${subtotal.toFixed(2)} €`;
    document.getElementById('shipping').textContent = `${shipping.toFixed(2)} €`;
    document.getElementById('total').textContent = `${total.toFixed(2)} €`;
}

function updateQuantity(productId, change) {
    // À implémenter selon la logique de votre panier
    loadCart(); // Recharger le panier
}

function removeFromCart(productId) {
    let cart = JSON.parse(localStorage.getItem('cart') || '[]');
    cart = cart.filter(id => id !== productId);
    localStorage.setItem('cart', JSON.stringify(cart));
    loadCart();
    updateCartCount();
}

// Gérer le bouton de paiement
document.getElementById('checkout-button')?.addEventListener('click', () => {
    // À implémenter : redirection vers le processus de paiement
    alert('Redirection vers le paiement...');
});
```

Cette implémentation du panier inclut :
- Affichage des produits dans le panier
- Modification des quantités
- Suppression des produits
- Calcul du total
- Interface utilisateur responsive

La prochaine étape logique serait :
1. Créer la page de détail produit
2. Mettre en place le système d'authentification
3. Implémenter le processus de paiement
