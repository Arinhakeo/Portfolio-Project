# Frontend FMP - Partie 5 : Processus de Commande et Paiement

## Sommaire
1. [Structure des fichiers](#structure-des-fichiers)
2. [Page de validation du panier](#page-de-validation-du-panier)
3. [Processus de commande](#processus-de-commande)
4. [Système de paiement](#système-de-paiement)
5. [Confirmation de commande](#confirmation-de-commande)

## Structure des fichiers
```
frontend/
├── pages/
│   ├── cart.html           # Panier détaillé
│   ├── checkout.html       # Processus de commande
│   └── confirmation.html   # Confirmation commande
├── static/
│   ├── css/
│   │   ├── cart.css       # Styles panier
│   │   └── checkout.css   # Styles commande
│   └── js/
│       ├── cart.js        # Logique panier
│       ├── checkout.js    # Logique commande
│       └── payment.js     # Intégration paiement
```

## 1. Page de Checkout (checkout.html)
```html
<!DOCTYPE html>
<html lang="fr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>FMP - Paiement</title>
    <link rel="stylesheet" href="../static/css/style.css">
    <link rel="stylesheet" href="../static/css/checkout.css">
</head>
<body>
    <!-- En-tête simplifié pour le processus de commande -->
    <header class="checkout-header">
        <div class="logo">
            <a href="index.html">
                <img src="../static/images/logo.png" alt="FMP Logo">
            </a>
        </div>
        <!-- Étapes du processus de commande -->
        <div class="checkout-steps">
            <div class="step active" data-step="1">
                <span class="step-number">1</span>
                <span class="step-label">Adresse</span>
            </div>
            <div class="step" data-step="2">
                <span class="step-number">2</span>
                <span class="step-label">Livraison</span>
            </div>
            <div class="step" data-step="3">
                <span class="step-number">3</span>
                <span class="step-label">Paiement</span>
            </div>
            <div class="step" data-step="4">
                <span class="step-number">4</span>
                <span class="step-label">Confirmation</span>
            </div>
        </div>
    </header>

    <main class="checkout-container">
        <!-- Formulaire de commande -->
        <form id="checkout-form" class="checkout-form">
            <!-- 1. Section Adresse -->
            <section id="address-section" class="checkout-section active">
                <h2>Adresse de livraison</h2>
                
                <!-- Adresses sauvegardées -->
                <div id="saved-addresses" class="saved-addresses">
                    <!-- Injecté via JavaScript -->
                </div>

                <!-- Nouvelle adresse -->
                <div id="new-address-form" class="new-address">
                    <h3>Nouvelle adresse</h3>
                    <div class="form-grid">
                        <div class="form-group">
                            <label for="firstname">Prénom</label>
                            <input type="text" id="firstname" name="firstname" required>
                        </div>
                        <div class="form-group">
                            <label for="lastname">Nom</label>
                            <input type="text" id="lastname" name="lastname" required>
                        </div>
                        <div class="form-group full-width">
                            <label for="address">Adresse</label>
                            <input type="text" id="address" name="address" required>
                        </div>
                        <div class="form-group">
                            <label for="postal_code">Code postal</label>
                            <input type="text" id="postal_code" name="postal_code" required>
                        </div>
                        <div class="form-group">
                            <label for="city">Ville</label>
                            <input type="text" id="city" name="city" required>
                        </div>
                        <div class="form-group">
                            <label for="phone">Téléphone</label>
                            <input type="tel" id="phone" name="phone" required>
                        </div>
                    </div>
                </div>
            </section>

            <!-- 2. Section Livraison -->
            <section id="shipping-section" class="checkout-section">
                <h2>Mode de livraison</h2>
                
                <div class="shipping-options">
                    <!-- Option Standard -->
                    <div class="shipping-option">
                        <input type="radio" id="standard-shipping" name="shipping_method" value="standard" checked>
                        <label for="standard-shipping">
                            <div class="shipping-info">
                                <span class="shipping-name">Livraison Standard</span>
                                <span class="shipping-delay">3-5 jours ouvrés</span>
                            </div>
                            <span class="shipping-price">5.90 €</span>
                        </label>
                    </div>

                    <!-- Option Express -->
                    <div class="shipping-option">
                        <input type="radio" id="express-shipping" name="shipping_method" value="express">
                        <label for="express-shipping">
                            <div class="shipping-info">
                                <span class="shipping-name">Livraison Express</span>
                                <span class="shipping-delay">1-2 jours ouvrés</span>
                            </div>
                            <span class="shipping-price">9.90 €</span>
                        </label>
                    </div>
                </div>
            </section>

            <!-- 3. Section Paiement -->
            <section id="payment-section" class="checkout-section">
                <h2>Paiement</h2>
                
                <!-- Méthodes de paiement -->
                <div class="payment-methods">
                    <div class="payment-method active">
                        <input type="radio" id="card-payment" name="payment_method" value="card" checked>
                        <label for="card-payment">
                            <i class="icon-credit-card"></i>
                            Carte bancaire
                        </label>
                    </div>
                    <!-- Formulaire carte bancaire -->
                    <div id="card-form" class="payment-form">
                        <div class="form-group">
                            <label for="card-number">Numéro de carte</label>
                            <div class="card-input">
                                <input type="text" id="card-number" name="card_number" placeholder="1234 5678 9012 3456" maxlength="19">
                                <div class="card-icons">
                                    <i class="icon-visa"></i>
                                    <i class="icon-mastercard"></i>
                                </div>
                            </div>
                        </div>
                        <div class="form-row">
                            <div class="form-group">
                                <label for="card-expiry">Date d'expiration</label>
                                <input type="text" id="card-expiry" name="card_expiry" placeholder="MM/AA" maxlength="5">
                            </div>
                            <div class="form-group">
                                <label for="card-cvv">CVV</label>
                                <input type="text" id="card-cvv" name="card_cvv" placeholder="123" maxlength="3">
                            </div>
                        </div>
                        <div class="form-group">
                            <label for="card-name">Nom sur la carte</label>
                            <input type="text" id="card-name" name="card_name" placeholder="JEAN DUPONT">
                        </div>
                    </div>
                </div>
            </section>
        </form>

        <!-- Résumé de la commande -->
        <aside class="order-summary">
            <h3>Résumé de la commande</h3>
            
            <!-- Liste des articles -->
            <div id="order-items" class="order-items">
                <!-- Injecté via JavaScript -->
            </div>

            <!-- Totaux -->
            <div class="order-totals">
                <div class="subtotal">
                    <span>Sous-total</span>
                    <span id="subtotal-amount">0.00 €</span>
                </div>
                <div class="shipping">
                    <span>Livraison</span>
                    <span id="shipping-amount">5.90 €</span>
                </div>
                <div class="total">
                    <span>Total</span>
                    <span id="total-amount">0.00 €</span>
                </div>
            </div>

            <!-- Bouton de validation -->
            <button type="button" id="validate-step" class="validate-button">
                Continuer
            </button>
        </aside>
    </main>

    <script src="../static/js/checkout.js"></script>
    <script src="../static/js/payment.js"></script>
</body>
</html>
```

## 2. Styles du Processus de Commande (checkout.css)
```css
/* Layout principal */
.checkout-container {
    display: grid;
    grid-template-columns: 1fr 350px;
    gap: 2rem;
    max-width: 1200px;
    margin: 2rem auto;
    padding: 0 2rem;
}

/* En-tête du checkout */
.checkout-header {
    background: white;
    padding: 1rem 0;
    box-shadow: 0 2px 4px rgba(0,0,0,0.1);
    position: sticky;
    top: 0;
    z-index: 100;
}

/* Étapes du processus */
.checkout-steps {
    display: flex;
    justify-content: center;
    gap: 2rem;
    margin-top: 2rem;
}

.step {
    display: flex;
    align-items: center;
    gap: 0.5rem;
    color: #666;
}

.step.active {
    color: #3498db;
}

.step-number {
    width: 24px;
    height: 24px;
    border-radius: 50%;
    background: #f0f0f0;
    display: flex;
    align-items: center;
    justify-content: center;
    font-weight: bold;
}

.step.active .step-number {
    background: #3498db;
    color: white;
}

/* Sections de checkout */
.checkout-section {
    display: none;
    background: white;
    border-radius: 8px;
    box-shadow: 0 2px 5px rgba(0,0,0,0.1);
    padding: 2rem;
}

.checkout-section.active {
    display: block;
}

/* Formulaires */
.form-grid {
    display: grid;
    grid-template-columns: 1fr 1fr;
    gap: 1rem;
}

.form-group {
    margin-bottom: 1.5rem;
}

.form-group.full-width {
    grid-column: 1 / -1;
}

.form-group label {
    display: block;
    margin-bottom: 0.5rem;
    color: #2c3e50;
}

.form-group input {
    width: 100%;
    padding: 0.8rem;
    border: 1px solid #ddd;
    border-radius: 4px;
    font-size: 1rem;
}

/* Options de livraison */
.shipping-options {
    display: flex;
    flex-direction: column;
    gap: 1rem;
}

.shipping-option {
    border: 1px solid #ddd;
    border-radius: 4px;
    padding: 1rem;
    cursor: pointer;
}

.shipping-option input[type="radio"] {
    display: none;
}

.shipping-option label {
    display: flex;
    justify-content: space-between;
    align-items: center;
    cursor: pointer;
}

/* Paiement */
.payment-methods {
    display: flex;
    flex-direction: column;
    gap: 1rem;
}

.card-input {
    position: relative;
}

.card-icons {
    position: absolute;
    right: 1rem;
    top: 50%;
    transform: translateY(-50%);
    display: flex;
    gap: 0.5rem;
}

/* Résumé de commande */
.order-summary {
    background: white;
    border-radius: 8px;
    box-shadow: 0 2px 5px rgba(0,0,0,0.1);
    padding: 1.5rem;
    height: fit-content;
    position: sticky;
    top: 2rem;
}

.order-items {
    margin: 1.5rem 0;
    max-height: 300px;
    overflow-y: auto;
}

.order-totals {
    border-top: 1px solid #eee;
    padding-top: 1rem;
}

.order-totals > div {
    display: flex;
    justify-content: space-between;
    margin-bottom: 0.5rem;
}

.total {
    font-weight: bold;
    font-size: 1.2rem;
    margin-top: 1rem;
    padding-top: 1rem;
    border-top: 1px solid #eee;
}

/* Bouton de validation */
.validate-button {
    width: 100%;
    background: #3498db;
    color: white;
    border: none;
    padding: 1rem;
    border-radius: 4px;
    font-size: 1rem;
    cursor: pointer;
    transition: background 0.3s;
}

.validate-button:hover {
    background: #2980b9;
}

/* Responsive */
@media (max-width: 768px) {
    .checkout-container {
        grid-template-columns: 1fr;
    }

    .form-grid {
        grid-template-columns: 1fr;
    }
}
```

# Frontend FMP - Partie 5 : Logique de Commande

## 1. Processus de Commande (checkout.js)
```javascript
// Gestion du processus de commande
document.addEventListener('DOMContentLoaded', () => {
    initCheckout();
    loadCartItems();
    setupEventListeners();
});

// Variables globales pour le processus
let currentStep = 1;
let orderData = {
    items: [],
    address: null,
    shipping: null,
    payment: null,
    totals: {
        subtotal: 0,
        shipping: 5.90,
        total: 0
    }
};

// Initialisation du processus
function initCheckout() {
    // Vérifier si le panier n'est pas vide
    const cart = JSON.parse(localStorage.getItem('cart') || '[]');
    if (cart.length === 0) {
        window.location.href = 'cart.html';
        return;
    }

    // Vérifier l'authentification
    if (!localStorage.getItem('authToken')) {
        window.location.href = `login.html?returnUrl=${encodeURIComponent(window.location.href)}`;
        return;
    }

    // Charger les adresses sauvegardées
    loadSavedAddresses();
}

// Chargement des articles du panier
async function loadCartItems() {
    const cart = JSON.parse(localStorage.getItem('cart') || '[]');
    const itemsContainer = document.getElementById('order-items');
    
    try {
        // Récupérer les détails des produits
        const items = await Promise.all(cart.map(async (item) => {
            const response = await fetch(`/api/products/${item.id}`);
            const product = await response.json();
            return {
                ...product,
                quantity: item.quantity
            };
        }));

        // Mettre à jour orderData
        orderData.items = items;
        
        // Afficher les articles
        itemsContainer.innerHTML = items.map(item => `
            <div class="order-item">
                <img src="${item.image}" alt="${item.name}">
                <div class="item-details">
                    <h4>${item.name}</h4>
                    <p>Quantité: ${item.quantity}</p>
                    <p class="item-price">${(item.price * item.quantity).toFixed(2)} €</p>
                </div>
            </div>
        `).join('');

        // Mettre à jour les totaux
        updateTotals();
        
    } catch (error) {
        console.error('Erreur chargement articles:', error);
        showError('Erreur lors du chargement des articles');
    }
}

// Mise à jour des totaux
function updateTotals() {
    const subtotal = orderData.items.reduce((total, item) => 
        total + (item.price * item.quantity), 0);
    
    orderData.totals = {
        subtotal,
        shipping: orderData.shipping === 'express' ? 9.90 : 5.90,
        total: subtotal + (orderData.shipping === 'express' ? 9.90 : 5.90)
    };

    // Mettre à jour l'affichage
    document.getElementById('subtotal-amount').textContent = 
        `${orderData.totals.subtotal.toFixed(2)} €`;
    document.getElementById('shipping-amount').textContent = 
        `${orderData.totals.shipping.toFixed(2)} €`;
    document.getElementById('total-amount').textContent = 
        `${orderData.totals.total.toFixed(2)} €`;
}

// Configuration des écouteurs d'événements
function setupEventListeners() {
    // Gestion des étapes
    document.getElementById('validate-step').addEventListener('click', validateCurrentStep);
    
    // Gestion du mode de livraison
    document.querySelectorAll('input[name="shipping_method"]').forEach(input => {
        input.addEventListener('change', (e) => {
            orderData.shipping = e.target.value;
            updateTotals();
        });
    });
}

// Validation de l'étape courante
async function validateCurrentStep() {
    const isValid = await validateStep(currentStep);
    
    if (isValid) {
        if (currentStep < 3) {
            // Passer à l'étape suivante
            moveToStep(currentStep + 1);
        } else {
            // Dernière étape : traiter le paiement
            processPayment();
        }
    }
}

// Validation des étapes
async function validateStep(step) {
    switch (step) {
        case 1: // Adresse
            return validateAddressStep();
        case 2: // Livraison
            return validateShippingStep();
        case 3: // Paiement
            return validatePaymentStep();
        default:
            return false;
    }
}

// Validation de l'adresse
function validateAddressStep() {
    const form = document.querySelector('#address-section form');
    if (!form.checkValidity()) {
        form.reportValidity();
        return false;
    }

    // Récupérer les données de l'adresse
    const formData = new FormData(form);
    orderData.address = Object.fromEntries(formData.entries());
    return true;
}

// Validation de la livraison
function validateShippingStep() {
    const shippingMethod = document.querySelector('input[name="shipping_method"]:checked');
    if (!shippingMethod) {
        showError('Veuillez sélectionner un mode de livraison');
        return false;
    }

    orderData.shipping = shippingMethod.value;
    return true;
}

// Validation du paiement
async function validatePaymentStep() {
    const cardForm = document.getElementById('card-form');
    if (!cardForm.checkValidity()) {
        cardForm.reportValidity();
        return false;
    }

    try {
        // Tokeniser la carte avec Stripe
        const token = await createPaymentToken();
        orderData.payment = { token };
        return true;
    } catch (error) {
        showError('Erreur de validation du paiement');
        return false;
    }
}

// Navigation entre les étapes
function moveToStep(step) {
    // Mettre à jour l'interface
    document.querySelectorAll('.checkout-section').forEach(section => {
        section.classList.remove('active');
    });
    document.querySelector(`[data-step="${step}"]`).classList.add('active');

    // Mettre à jour les indicateurs d'étape
    document.querySelectorAll('.step').forEach(stepEl => {
        const stepNum = parseInt(stepEl.getAttribute('data-step'));
        stepEl.classList.toggle('active', stepNum <= step);
    });

    // Mettre à jour le texte du bouton
    const validateButton = document.getElementById('validate-step');
    validateButton.textContent = step === 3 ? 'Payer' : 'Continuer';

    currentStep = step;
}

// Gestion des erreurs
function showError(message) {
    const notification = document.createElement('div');
    notification.className = 'error-notification';
    notification.textContent = message;

    document.body.appendChild(notification);
    setTimeout(() => notification.remove(), 5000);
}
```

## 2. Intégration du Paiement (payment.js)
```javascript
// Configuration de Stripe
let stripe;
let elements;

document.addEventListener('DOMContentLoaded', () => {
    initializeStripe();
    setupCardValidation();
});

// Initialisation de Stripe
async function initializeStripe() {
    stripe = Stripe('votre_clé_publique_stripe');
    elements = stripe.elements();

    // Créer les éléments de carte
    const cardNumber = elements.create('cardNumber');
    const cardExpiry = elements.create('cardExpiry');
    const cardCvc = elements.create('cardCvc');

    // Monter les éléments
    cardNumber.mount('#card-number');
    cardExpiry.mount('#card-expiry');
    cardCvc.mount('#card-cvv');

    // Gérer les erreurs de validation
    [cardNumber, cardExpiry, cardCvc].forEach(element => {
        element.addEventListener('change', ({error}) => {
            const displayError = document.getElementById('card-errors');
            if (error) {
                displayError.textContent = error.message;
            } else {
                displayError.textContent = '';
            }
        });
    });
}

// Création du token de paiement
async function createPaymentToken() {
    try {
        const {token, error} = await stripe.createToken('card');
        if (error) {
            throw new Error(error.message);
        }
        return token;
    } catch (error) {
        throw new Error('Erreur lors de la création du token de paiement');
    }
}

// Traitement du paiement
async function processPayment() {
    // Afficher l'indicateur de chargement
    showLoadingIndicator();

    try {
        // Créer la commande
        const order = await createOrder();
        
        // Traiter le paiement
        const response = await fetch('/api/payment/process', {
            method: 'POST',
            headers: {
                'Content-Type': 'application/json',
                'Authorization': `Bearer ${localStorage.getItem('authToken')}`
            },
            body: JSON.stringify({
                orderId: order.id,
                paymentToken: orderData.payment.token.id,
                amount: orderData.totals.total * 100 // En centimes
            })
        });

        if (!response.ok) throw new Error('Erreur de paiement');

        const result = await response.json();
        
        // Rediriger vers la page de confirmation
        window.location.href = `confirmation.html?orderId=${order.id}`;
        
    } catch (error) {
        showError('Erreur lors du traitement du paiement');
        hideLoadingIndicator();
    }
}

// Création de la commande
async function createOrder() {
    try {
        const response = await fetch('/api/orders', {
            method: 'POST',
            headers: {
                'Content-Type': 'application/json',
                'Authorization': `Bearer ${localStorage.getItem('authToken')}`
            },
            body: JSON.stringify({
                items: orderData.items.map(item => ({
                    productId: item.id,
                    quantity: item.quantity
                })),
                shipping: orderData.shipping,
                address: orderData.address,
                totals: orderData.totals
            })
        });

        if (!response.ok) throw new Error('Erreur de création de commande');

        return await response.json();
        
    } catch (error) {
        throw new Error('Erreur lors de la création de la commande');
    }
}
```

## 3. Page de Confirmation (confirmation.html)
```html
<!DOCTYPE html>
<html lang="fr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>FMP - Confirmation de commande</title>
    <link rel="stylesheet" href="../static/css/style.css">
    <link rel="stylesheet" href="../static/css/confirmation.css">
</head>
<body>
    <header>
        <!-- En-tête standard -->
    </header>

    <main class="confirmation-container">
        <div class="confirmation-box">
            <div class="confirmation-icon">
                <i class="icon-check-circle"></i>
            </div>
            
            <h1>Commande confirmée !</h1>
            <p class="confirmation-number">
                Commande n° <span id="order-number"></span>
            </p>
            
            <div class="confirmation-details">
                <p>Un e-mail de confirmation a été envoyé à <span id="customer-email"></span></p>
                <p>Vous pouvez suivre votre commande dans votre espace client.</p>
            </div>

            <div class="order-summary">
                <h2>Récapitulatif de la commande</h2>
                
                <!-- Détails de livraison -->
                <div class="summary-section">
                    <h3>Livraison</h3>
                    <div id="delivery-address"></div>
                    <div id="shipping-method"></div>
                </div>

                <!-- Articles commandés -->
                <div class="summary-section">
                    <h3>Articles</h3>
                    <div id="ordered-items"></div>
                </div>

                <!-- Totaux -->
                <div class="summary-section totals">
                    <div class="total-line">
                        <span>Sous-total</span>
                        <span id="subtotal"></span>
                    </div>
                    <div class="total-line">
                        <span>Livraison</span>
                        <span id="shipping-cost"></span>
                    </div>
                    <div class="total-line grand-total">
                        <span>Total</span>
                        <span id="total"></span>
                    </div>
                </div>
            </div>

            <div class="confirmation-actions">
                <a href="account.html?tab=orders" class="primary-button">
                    Suivre ma commande
                </a>
                <a href="products.html" class="secondary-button">
                    Continuer mes achats
                </a>
            </div>
        </div>
    </main>

    <footer>
        <!-- Pied de page standard -->
    </footer>

    <script src="../static/js/confirmation.js"></script>
</body>
</html>
```

## 4. Styles de la Confirmation (confirmation.css)
```css
/* Conteneur principal */
.confirmation-container {
    max-width: 800px;
    margin: 4rem auto;
    padding: 0 2rem;
}

/* Boîte de confirmation */
.confirmation-box {
    background: white;
    border-radius: 8px;
    box-shadow: 0 2px 10px rgba(0,0,0,0.1);
    padding: 2rem;
    text-align: center;
}

/* Icône de confirmation */
.confirmation-icon {
    font-size: 4rem;
    color: #27ae60;
    margin-bottom: 1rem;
}

/* Numéro de commande */
.confirmation-number {
    font-size: 1.2rem;
    color: #666;
    margin: 1rem 0;
}

/* Détails de la confirmation */
.confirmation-details {
    margin: 2rem 0;
    padding: 1rem;
    background: #f8f9fa;
    border-radius: 4px;
}

/* Récapitulatif de la commande */
.order-summary {
    margin: 3rem 0;
    text-align: left;
}

.summary-section {
    margin: 2rem 0;
    padding-bottom: 1rem;
    border-bottom: 1px solid #eee;
}

.summary-section h3 {
    color: #2c3e50;
    margin-bottom: 1rem;
}

/* Lignes de total */
.total-line {
    display: flex;
    justify-content: space-between;
    margin: 0.5rem 0;
    font-size: 1.1rem;
}

.grand-total {
    font-weight: bold;
    font-size: 1.3rem;
    margin-top: 1rem;
    padding-top: 1rem;
    border-top: 1px solid #eee;
}

/* Articles commandés */
.ordered-item {
    display: flex;
    gap: 1rem;
    margin: 1rem 0;
    padding: 1rem;
    background: #f8f9fa;
    border-radius: 4px;
}

.ordered-item img {
    width: 80px;
    height: 80px;
    object-fit: cover;
    border-radius: 4px;
}

.item-details {
    flex: 1;
}

.item-price {
    font-weight: bold;
    color: #2c3e50;
}

/* Actions de confirmation */
.confirmation-actions {
    margin-top: 3rem;
    display: flex;
    gap: 1rem;
    justify-content: center;
}

.primary-button,
.secondary-button {
    padding: 1rem 2rem;
    border-radius: 4px;
    text-decoration: none;
    font-weight: 500;
    transition: background 0.3s;
}

.primary-button {
    background: #3498db;
    color: white;
}

.secondary-button {
    background: #f8f9fa;
    color: #2c3e50;
}

.primary-button:hover {
    background: #2980b9;
}

.secondary-button:hover {
    background: #e9ecef;
}

/* Responsive */
@media (max-width: 768px) {
    .confirmation-container {
        margin: 2rem auto;
    }

    .confirmation-actions {
        flex-direction: column;
    }

    .ordered-item {
        flex-direction: column;
        align-items: center;
        text-align: center;
    }
}
```

## 2. Logique de Confirmation (confirmation.js)
```javascript
document.addEventListener('DOMContentLoaded', () => {
    loadOrderDetails();
    clearCart();
});

// Chargement des détails de la commande
async function loadOrderDetails() {
    try {
        // Récupérer l'ID de commande depuis l'URL
        const urlParams = new URLSearchParams(window.location.search);
        const orderId = urlParams.get('orderId');

        if (!orderId) {
            window.location.href = 'index.html';
            return;
        }

        // Charger les détails de la commande
        const order = await fetchOrderDetails(orderId);
        
        // Mettre à jour l'interface
        updateConfirmationPage(order);
        
    } catch (error) {
        console.error('Erreur chargement commande:', error);
        showError('Erreur lors du chargement des détails de la commande');
    }
}

// Récupération des détails de commande
async function fetchOrderDetails(orderId) {
    const response = await fetch(`/api/orders/${orderId}`, {
        headers: {
            'Authorization': `Bearer ${localStorage.getItem('authToken')}`
        }
    });

    if (!response.ok) throw new Error('Erreur de chargement de la commande');

    return await response.json();
}

// Mise à jour de la page de confirmation
function updateConfirmationPage(order) {
    // Numéro de commande
    document.getElementById('order-number').textContent = order.id;
    
    // Email client
    document.getElementById('customer-email').textContent = order.customer.email;
    
    // Adresse de livraison
    updateDeliveryAddress(order.shippingAddress);
    
    // Mode de livraison
    updateShippingMethod(order.shipping);
    
    // Articles commandés
    updateOrderedItems(order.items);
    
    // Totaux
    updateOrderTotals(order.totals);
}

// Mise à jour de l'adresse de livraison
function updateDeliveryAddress(address) {
    const addressElement = document.getElementById('delivery-address');
    addressElement.innerHTML = `
        <div class="delivery-address">
            <p>${address.firstname} ${address.lastname}</p>
            <p>${address.address}</p>
            <p>${address.postal_code} ${address.city}</p>
            <p>${address.phone}</p>
        </div>
    `;
}

// Mise à jour du mode de livraison
function updateShippingMethod(shipping) {
    const shippingElement = document.getElementById('shipping-method');
    const methodName = shipping.method === 'standard' ? 'Livraison Standard' : 'Livraison Express';
    const delay = shipping.method === 'standard' ? '3-5 jours ouvrés' : '1-2 jours ouvrés';
    
    shippingElement.innerHTML = `
        <div class="shipping-method">
            <p>${methodName}</p>
            <p>Délai estimé : ${delay}</p>
        </div>
    `;
}

// Mise à jour des articles commandés
function updateOrderedItems(items) {
    const itemsContainer = document.getElementById('ordered-items');
    
    itemsContainer.innerHTML = items.map(item => `
        <div class="ordered-item">
            <img src="${item.image}" alt="${item.name}">
            <div class="item-details">
                <h4>${item.name}</h4>
                <p>Quantité: ${item.quantity}</p>
                <p class="item-price">${(item.price * item.quantity).toFixed(2)} €</p>
            </div>
        </div>
    `).join('');
}

// Mise à jour des totaux
function updateOrderTotals(totals) {
    document.getElementById('subtotal').textContent = 
        `${totals.subtotal.toFixed(2)} €`;
    document.getElementById('shipping-cost').textContent = 
        `${totals.shipping.toFixed(2)} €`;
    document.getElementById('total').textContent = 
        `${totals.total.toFixed(2)} €`;
}

// Vider le panier
function clearCart() {
    localStorage.removeItem('cart');
}

// Affichage des erreurs
function showError(message) {
    const notification = document.createElement('div');
    notification.className = 'error-notification';
    notification.textContent = message;

    document.body.appendChild(notification);
    setTimeout(() => notification.remove(), 5000);
}
```

## 3. Envoi d'Email de Confirmation (backend)
```python
# backend/app/services/email_service.py

from flask_mail import Message
from app import mail
from jinja2 import Template

def send_order_confirmation(order):
    """Envoie l'email de confirmation de commande"""
    try:
        # Charger le template
        with open('app/templates/emails/order_confirmation.html', 'r') as file:
            template = Template(file.read())

        # Rendre le template avec les données de la commande
        html_content = template.render(
            order_id=order.id,
            customer_name=f"{order.customer.firstname} {order.customer.lastname}",
            order_items=order.items,
            shipping_address=order.shipping_address,
            totals=order.totals
        )

        # Créer le message
        msg = Message(
            subject=f'FMP - Confirmation de votre commande #{order.id}',
            recipients=[order.customer.email],
            html=html_content
        )

        # Envoyer l'email
        mail.send(msg)
        return True
        
    except Exception as e:
        print(f"Erreur envoi email: {str(e)}")
        return False
```

## 4. Template Email de Confirmation
```html
<!-- backend/app/templates/emails/order_confirmation.html -->
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <style>
        body {
            font-family: Arial, sans-serif;
            line-height: 1.6;
            color: #333;
            max-width: 600px;
            margin: 0 auto;
            padding: 20px;
        }
        .header {
            text-align: center;
            margin-bottom: 30px;
        }
        .order-details {
            background: #f8f9fa;
            padding: 20px;
            border-radius: 4px;
            margin-bottom: 20px;
        }
        .items-table {
            width: 100%;
            border-collapse: collapse;
            margin: 20px 0;
        }
        .items-table th,
        .items-table td {
            padding: 10px;
            border-bottom: 1px solid #ddd;
            text-align: left;
        }
        .total {
            text-align: right;
            font-weight: bold;
            margin-top: 20px;
        }
        .footer {
            text-align: center;
            margin-top: 30px;
            padding-top: 20px;
            border-top: 1px solid #ddd;
            color: #666;
        }
    </style>
</head>
<body>
    <div class="header">
        <h1>Confirmation de commande</h1>
        <p>Merci pour votre commande !</p>
    </div>

    <div class="order-details">
        <h2>Commande #{{ order_id }}</h2>
        <p>Bonjour {{ customer_name }},</p>
        <p>Nous avons bien reçu votre commande et nous la préparons avec soin.</p>
    </div>

    <h3>Articles commandés</h3>
    <table class="items-table">
        <thead>
            <tr>
                <th>Produit</th>
                <th>Quantité</th>
                <th>Prix</th>
            </tr>
        </thead>
        <tbody>
            {% for item in order_items %}
            <tr>
                <td>{{ item.name }}</td>
                <td>{{ item.quantity }}</td>
                <td>{{ "%.2f"|format(item.price * item.quantity) }} €</td>
            </tr>
            {% endfor %}
        </tbody>
    </table>

    <div class="total">
        <p>Sous-total : {{ "%.2f"|format(totals.subtotal) }} €</p>
        <p>Livraison : {{ "%.2f"|format(totals.shipping) }} €</p>
        <p>Total : {{ "%.2f"|format(totals.total) }} €</p>
    </div>

    <div class="order-details">
        <h3>Adresse de livraison</h3>
        <p>{{ shipping_address.firstname }} {{ shipping_address.lastname }}</p>
        <p>{{ shipping_address.address }}</p>
        <p>{{ shipping_address.postal_code }} {{ shipping_address.city }}</p>
    </div>

    <div class="footer">
        <p>Pour toute question, n'hésitez pas à nous contacter.</p>
        <p>L'équipe FMP</p>
    </div>
</body>
</html>
```

Cela complète la Partie 5 avec toutes les fonctionnalités nécessaires pour :
1. Le processus de commande
2. Le paiement sécurisé
3. La confirmation de commande
4. Les emails de confirmation
