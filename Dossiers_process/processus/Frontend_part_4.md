# Frontend FMP - Partie 4 : Espace Client et Authentification

## Structure des Dossiers
```
frontend/
├── pages/
│   ├── account.html        # Espace client
│   ├── login.html         # Page de connexion
│   └── register.html      # Page d'inscription
├── static/
│   ├── css/
│   │   ├── style.css      # Styles globaux
│   │   ├── auth.css       # Styles authentification
│   │   └── account.css    # Styles espace client
│   └── js/
│       ├── auth.js        # Logique authentification
│       └── account.js     # Logique espace client
```

## 1. Page de Connexion (login.html)
```html
<!DOCTYPE html>
<html lang="fr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>FMP - Connexion</title>
    <link rel="stylesheet" href="../static/css/style.css">
    <link rel="stylesheet" href="../static/css/auth.css">
</head>
<body>
    <header>
        <!-- Navigation standard -->
    </header>

    <main class="auth-container">
        <div class="auth-box">
            <div class="auth-header">
                <h1>Connexion</h1>
                <p>Bienvenue sur FMP</p>
            </div>

            <form id="login-form" class="auth-form">
                <!-- Message d'erreur -->
                <div id="error-message" class="error-message" style="display: none;"></div>

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

                <!-- Mot de passe -->
                <div class="form-group">
                    <label for="password">Mot de passe</label>
                    <div class="password-input">
                        <input 
                            type="password" 
                            id="password" 
                            name="password" 
                            required>
                        <button type="button" class="toggle-password">
                            <i class="icon-eye"></i>
                        </button>
                    </div>
                </div>

                <!-- Options de connexion -->
                <div class="form-options">
                    <label class="remember-me">
                        <input type="checkbox" name="remember">
                        <span>Se souvenir de moi</span>
                    </label>
                    <a href="forgot-password.html" class="forgot-password">
                        Mot de passe oublié ?
                    </a>
                </div>

                <!-- Bouton de connexion -->
                <button type="submit" class="auth-button">
                    <span>Se connecter</span>
                </button>
            </form>

            <!-- Lien d'inscription -->
            <div class="auth-footer">
                <p>Pas encore de compte ?</p>
                <a href="register.html" class="register-link">
                    Créer un compte
                </a>
            </div>
        </div>
    </main>

    <footer>
        <!-- Footer standard -->
    </footer>

    <script src="../static/js/api.js"></script>
    <script src="../static/js/auth.js"></script>
</body>
</html>
```

## 2. Styles d'Authentification (auth.css)
```css
/* Conteneur principal */
.auth-container {
    min-height: calc(100vh - 140px);
    display: flex;
    align-items: center;
    justify-content: center;
    padding: 2rem;
    background: #f8f9fa;
}

/* Boîte de connexion */
.auth-box {
    background: white;
    border-radius: 8px;
    box-shadow: 0 2px 10px rgba(0,0,0,0.1);
    padding: 2rem;
    width: 100%;
    max-width: 400px;
}

.auth-header {
    text-align: center;
    margin-bottom: 2rem;
}

.auth-header h1 {
    font-size: 1.8rem;
    color: #2c3e50;
    margin-bottom: 0.5rem;
}

.auth-header p {
    color: #666;
}

/* Formulaire */
.auth-form {
    display: flex;
    flex-direction: column;
    gap: 1.5rem;
}

.form-group {
    display: flex;
    flex-direction: column;
    gap: 0.5rem;
}

.form-group label {
    font-weight: 500;
    color: #2c3e50;
}

.form-group input {
    padding: 0.8rem;
    border: 1px solid #ddd;
    border-radius: 4px;
    font-size: 1rem;
    transition: border-color 0.3s;
}

.form-group input:focus {
    border-color: #3498db;
    outline: none;
}

/* Input mot de passe avec icône */
.password-input {
    position: relative;
}

.toggle-password {
    position: absolute;
    right: 1rem;
    top: 50%;
    transform: translateY(-50%);
    background: none;
    border: none;
    color: #666;
    cursor: pointer;
}

/* Options de connexion */
.form-options {
    display: flex;
    justify-content: space-between;
    align-items: center;
    font-size: 0.9rem;
}

.remember-me {
    display: flex;
    align-items: center;
    gap: 0.5rem;
}

.forgot-password {
    color: #3498db;
    text-decoration: none;
}

/* Bouton de connexion */
.auth-button {
    background: #3498db;
    color: white;
    border: none;
    padding: 1rem;
    border-radius: 4px;
    font-size: 1rem;
    cursor: pointer;
    transition: background 0.3s;
}

.auth-button:hover {
    background: #2980b9;
}

/* Pied de page du formulaire */
.auth-footer {
    text-align: center;
    margin-top: 2rem;
    padding-top: 2rem;
    border-top: 1px solid #eee;
}

.register-link {
    color: #3498db;
    text-decoration: none;
    font-weight: 500;
    display: block;
    margin-top: 0.5rem;
}

/* Message d'erreur */
.error-message {
    background: #fee;
    color: #e74c3c;
    padding: 1rem;
    border-radius: 4px;
    font-size: 0.9rem;
    margin-bottom: 1rem;
}

/* Responsive */
@media (max-width: 480px) {
    .auth-box {
        padding: 1.5rem;
    }

    .form-options {
        flex-direction: column;
        gap: 1rem;
        align-items: flex-start;
    }
}
```

## 3. Logique d'Authentification (auth.js)
```javascript
document.addEventListener('DOMContentLoaded', () => {
    setupLoginForm();
    setupPasswordToggle();
});

// Configuration du formulaire de connexion
function setupLoginForm() {
    const loginForm = document.getElementById('login-form');
    const errorMessage = document.getElementById('error-message');

    loginForm.addEventListener('submit', async (e) => {
        e.preventDefault();
        
        // Récupérer les données du formulaire
        const formData = new FormData(loginForm);
        const credentials = {
            email: formData.get('email'),
            password: formData.get('password'),
            remember: formData.get('remember') ? true : false
        };

        try {
            // Appel API pour la connexion
            const response = await fetch('/api/auth/login', {
                method: 'POST',
                headers: {
                    'Content-Type': 'application/json'
                },
                body: JSON.stringify(credentials)
            });

            const data = await response.json();

            if (response.ok) {
                // Stocker le token
                localStorage.setItem('authToken', data.token);
                
                // Rediriger vers la page précédente ou l'accueil
                const returnUrl = new URLSearchParams(window.location.search).get('returnUrl');
                window.location.href = returnUrl || '/';
            } else {
                // Afficher l'erreur
                showError(data.message || 'Erreur de connexion');
            }
        } catch (error) {
            showError('Erreur de connexion au serveur');
        }
    });
}

// Affichage des erreurs
function showError(message) {
    const errorMessage = document.getElementById('error-message');
    errorMessage.textContent = message;
    errorMessage.style.display = 'block';

    // Masquer après 5 secondes
    setTimeout(() => {
        errorMessage.style.display = 'none';
    }, 5000);
}

// Gestion de l'affichage/masquage du mot de passe
function setupPasswordToggle() {
    const toggleButton = document.querySelector('.toggle-password');
    const passwordInput = document.getElementById('password');

    toggleButton.addEventListener('click', () => {
        const type = passwordInput.getAttribute('type') === 'password' ? 'text' : 'password';
        passwordInput.setAttribute('type', type);
        
        // Changer l'icône
        const icon = toggleButton.querySelector('i');
        icon.className = type === 'password' ? 'icon-eye' : 'icon-eye-off';
    });
}

// Vérification du token d'authentification
function checkAuth() {
    const token = localStorage.getItem('authToken');
    return token !== null;
}

// Déconnexion
function logout() {
    localStorage.removeItem('authToken');
    window.location.href = '/login.html';
}

// Ajout du token aux requêtes API
function getAuthHeaders() {
    const token = localStorage.getItem('authToken');
    return token ? {
        'Authorization': `Bearer ${token}`
    } : {};
}
```

## 1. Page Espace Client (account.html)
```html
<!DOCTYPE html>
<html lang="fr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>FMP - Mon Compte</title>
    <link rel="stylesheet" href="../static/css/style.css">
    <link rel="stylesheet" href="../static/css/account.css">
</head>
<body>
    <header>
        <!-- Navigation standard -->
    </header>

    <main class="account-container">
        <!-- Menu latéral -->
        <aside class="account-sidebar">
            <div class="user-info">
                <div class="user-avatar">
                    <img id="user-avatar" src="../static/images/default-avatar.png" alt="Avatar">
                </div>
                <div class="user-details">
                    <h3 id="user-name">Chargement...</h3>
                    <p id="user-email">Chargement...</p>
                </div>
            </div>

            <nav class="account-nav">
                <ul>
                    <li><a href="#profile" class="active" data-section="profile">Mon Profil</a></li>
                    <li><a href="#orders" data-section="orders">Mes Commandes</a></li>
                    <li><a href="#addresses" data-section="addresses">Mes Adresses</a></li>
                    <li><a href="#settings" data-section="settings">Paramètres</a></li>
                    <li><a href="#" id="logout-btn" class="logout">Déconnexion</a></li>
                </ul>
            </nav>
        </aside>

        <!-- Contenu principal -->
        <div class="account-content">
            <!-- Section Profil -->
            <section id="profile-section" class="account-section active">
                <h2>Mon Profil</h2>
                <form id="profile-form" class="profile-form">
                    <div class="form-group">
                        <label for="firstname">Prénom</label>
                        <input type="text" id="firstname" name="firstname" required>
                    </div>
                    <div class="form-group">
                        <label for="lastname">Nom</label>
                        <input type="text" id="lastname" name="lastname" required>
                    </div>
                    <div class="form-group">
                        <label for="phone">Téléphone</label>
                        <input type="tel" id="phone" name="phone">
                    </div>
                    <div class="form-group">
                        <label for="email">Email</label>
                        <input type="email" id="email" name="email" required readonly>
                    </div>
                    <button type="submit" class="save-button">
                        Enregistrer les modifications
                    </button>
                </form>
            </section>

            <!-- Section Commandes -->
            <section id="orders-section" class="account-section">
                <h2>Mes Commandes</h2>
                <div class="orders-list" id="orders-list">
                    <!-- Liste des commandes ajoutée via JS -->
                </div>
            </section>

            <!-- Section Adresses -->
            <section id="addresses-section" class="account-section">
                <h2>Mes Adresses</h2>
                <div class="addresses-grid" id="addresses-grid">
                    <!-- Adresses ajoutées via JS -->
                </div>
                <button id="add-address-btn" class="add-button">
                    Ajouter une adresse
                </button>
            </section>

            <!-- Section Paramètres -->
            <section id="settings-section" class="account-section">
                <h2>Paramètres</h2>
                <div class="settings-content">
                    <!-- Changement de mot de passe -->
                    <div class="settings-block">
                        <h3>Changer le mot de passe</h3>
                        <form id="password-form" class="settings-form">
                            <div class="form-group">
                                <label for="current-password">Mot de passe actuel</label>
                                <input type="password" id="current-password" name="current-password" required>
                            </div>
                            <div class="form-group">
                                <label for="new-password">Nouveau mot de passe</label>
                                <input type="password" id="new-password" name="new-password" required>
                            </div>
                            <div class="form-group">
                                <label for="confirm-password">Confirmer le mot de passe</label>
                                <input type="password" id="confirm-password" name="confirm-password" required>
                            </div>
                            <button type="submit" class="save-button">
                                Changer le mot de passe
                            </button>
                        </form>
                    </div>

                    <!-- Notifications -->
                    <div class="settings-block">
                        <h3>Notifications</h3>
                        <div class="notifications-settings">
                            <label class="toggle-switch">
                                <input type="checkbox" id="email-notifications">
                                <span class="toggle-slider"></span>
                                <span class="toggle-label">Notifications par email</span>
                            </label>
                            <label class="toggle-switch">
                                <input type="checkbox" id="order-notifications">
                                <span class="toggle-slider"></span>
                                <span class="toggle-label">Suivi de commandes</span>
                            </label>
                        </div>
                    </div>
                </div>
            </section>
        </div>
    </main>

    <footer>
        <!-- Footer standard -->
    </footer>

    <!-- Modals -->
    <div id="address-modal" class="modal">
        <div class="modal-content">
            <h3>Ajouter une adresse</h3>
            <form id="address-form">
                <!-- Contenu du formulaire d'adresse -->
            </form>
        </div>
    </div>

    <script src="../static/js/api.js"></script>
    <script src="../static/js/account.js"></script>
</body>
</html>
```

## 2. Styles de l'Espace Client (account.css)
```css
/* Layout principal */
.account-container {
    max-width: 1200px;
    margin: 2rem auto;
    padding: 0 2rem;
    display: grid;
    grid-template-columns: 250px 1fr;
    gap: 2rem;
}

/* Sidebar */
.account-sidebar {
    background: white;
    border-radius: 8px;
    box-shadow: 0 2px 5px rgba(0,0,0,0.1);
    padding: 1.5rem;
}

.user-info {
    display: flex;
    align-items: center;
    gap: 1rem;
    margin-bottom: 2rem;
    padding-bottom: 1.5rem;
    border-bottom: 1px solid #eee;
}

.user-avatar img {
    width: 60px;
    height: 60px;
    border-radius: 50%;
    object-fit: cover;
}

.user-details h3 {
    margin: 0;
    font-size: 1.1rem;
    color: #2c3e50;
}

.user-details p {
    margin: 0.25rem 0 0;
    font-size: 0.9rem;
    color: #666;
}

/* Navigation */
.account-nav ul {
    list-style: none;
    padding: 0;
    margin: 0;
}

.account-nav a {
    display: block;
    padding: 0.8rem 1rem;
    color: #2c3e50;
    text-decoration: none;
    border-radius: 4px;
    transition: background 0.3s;
}

.account-nav a:hover {
    background: #f8f9fa;
}

.account-nav a.active {
    background: #3498db;
    color: white;
}

.account-nav .logout {
    color: #e74c3c;
    margin-top: 1rem;
    border-top: 1px solid #eee;
    padding-top: 1rem;
}

/* Sections de contenu */
.account-section {
    display: none;
    background: white;
    border-radius: 8px;
    box-shadow: 0 2px 5px rgba(0,0,0,0.1);
    padding: 2rem;
}

.account-section.active {
    display: block;
}

.account-section h2 {
    margin: 0 0 2rem;
    color: #2c3e50;
}

/* Formulaires */
.form-group {
    margin-bottom: 1.5rem;
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

.save-button {
    background: #3498db;
    color: white;
    border: none;
    padding: 0.8rem 1.5rem;
    border-radius: 4px;
    cursor: pointer;
    transition: background 0.3s;
}

.save-button:hover {
    background: #2980b9;
}

/* Liste des commandes */
.orders-list {
    display: flex;
    flex-direction: column;
    gap: 1rem;
}

.order-card {
    border: 1px solid #eee;
    border-radius: 4px;
    padding: 1.5rem;
}

.order-header {
    display: flex;
    justify-content: space-between;
    margin-bottom: 1rem;
}

/* Grid d'adresses */
.addresses-grid {
    display: grid;
    grid-template-columns: repeat(auto-fill, minmax(250px, 1fr));
    gap: 1.5rem;
    margin-bottom: 2rem;
}

.address-card {
    border: 1px solid #eee;
    border-radius: 4px;
    padding: 1.5rem;
    position: relative;
}

/* Toggles pour les paramètres */
.toggle-switch {
    display: flex;
    align-items: center;
    gap: 1rem;
    margin: 1rem 0;
}

.toggle-slider {
    width: 50px;
    height: 24px;
    background: #ddd;
    border-radius: 12px;
    position: relative;
    cursor: pointer;
    transition: background 0.3s;
}

.toggle-slider:before {
    content: '';
    position: absolute;
    width: 20px;
    height: 20px;
    background: white;
    border-radius: 50%;
    top: 2px;
    left: 2px;
    transition: transform 0.3s;
}

input:checked + .toggle-slider {
    background: #3498db;
}

input:checked + .toggle-slider:before {
    transform: translateX(26px);
}

/* Responsive */
@media (max-width: 768px) {
    .account-container {
        grid-template-columns: 1fr;
    }

    .account-sidebar {
        position: relative;
        z-index: 1;
    }

    .account-nav ul {
        display: flex;
        overflow-x: auto;
        padding-bottom: 1rem;
    }

    .account-nav a {
        white-space: nowrap;
    }
}
```

# Frontend FMP - Suite de la Partie 4

## 1. Logique de l'Espace Client (account.js)
```javascript
// Initialisation
document.addEventListener('DOMContentLoaded', () => {
    checkAuthentication();
    loadUserProfile();
    setupNavigation();
    setupForms();
});

// Vérification de l'authentification
function checkAuthentication() {
    if (!localStorage.getItem('authToken')) {
        window.location.href = 'login.html';
    }
}

// Chargement du profil utilisateur
async function loadUserProfile() {
    try {
        const response = await fetch('/api/user/profile', {
            headers: {
                'Authorization': `Bearer ${localStorage.getItem('authToken')}`
            }
        });

        if (!response.ok) throw new Error('Erreur de chargement du profil');

        const user = await response.json();
        updateProfileDisplay(user);
        
    } catch (error) {
        showNotification('Erreur de chargement du profil', 'error');
    }
}

// Mise à jour de l'affichage du profil
function updateProfileDisplay(user) {
    document.getElementById('user-name').textContent = `${user.firstname} ${user.lastname}`;
    document.getElementById('user-email').textContent = user.email;
    
    // Remplir le formulaire de profil
    document.getElementById('firstname').value = user.firstname;
    document.getElementById('lastname').value = user.lastname;
    document.getElementById('phone').value = user.phone || '';
    document.getElementById('email').value = user.email;
}

// Configuration de la navigation
function setupNavigation() {
    const navLinks = document.querySelectorAll('.account-nav a[data-section]');
    
    navLinks.forEach(link => {
        link.addEventListener('click', (e) => {
            e.preventDefault();
            const sectionId = link.getAttribute('data-section');
            switchSection(sectionId);
        });
    });
}

// Changement de section
function switchSection(sectionId) {
    // Mettre à jour les classes actives
    document.querySelectorAll('.account-nav a').forEach(link => {
        link.classList.remove('active');
    });
    document.querySelector(`[data-section="${sectionId}"]`).classList.add('active');

    // Afficher la section correspondante
    document.querySelectorAll('.account-section').forEach(section => {
        section.classList.remove('active');
    });
    document.getElementById(`${sectionId}-section`).classList.add('active');

    // Charger les données si nécessaire
    if (sectionId === 'orders') loadOrders();
    if (sectionId === 'addresses') loadAddresses();
}

// Configuration des formulaires
function setupForms() {
    setupProfileForm();
    setupPasswordForm();
    setupAddressForm();
    setupNotificationSettings();
}

// Gestion du formulaire de profil
function setupProfileForm() {
    const profileForm = document.getElementById('profile-form');
    
    profileForm.addEventListener('submit', async (e) => {
        e.preventDefault();
        
        const formData = new FormData(profileForm);
        const userData = {
            firstname: formData.get('firstname'),
            lastname: formData.get('lastname'),
            phone: formData.get('phone')
        };

        try {
            const response = await fetch('/api/user/profile', {
                method: 'PUT',
                headers: {
                    'Content-Type': 'application/json',
                    'Authorization': `Bearer ${localStorage.getItem('authToken')}`
                },
                body: JSON.stringify(userData)
            });

            if (!response.ok) throw new Error('Erreur de mise à jour du profil');

            showNotification('Profil mis à jour avec succès', 'success');
            
        } catch (error) {
            showNotification('Erreur lors de la mise à jour du profil', 'error');
        }
    });
}

// Gestion du changement de mot de passe
function setupPasswordForm() {
    const passwordForm = document.getElementById('password-form');
    
    passwordForm.addEventListener('submit', async (e) => {
        e.preventDefault();
        
        const formData = new FormData(passwordForm);
        
        // Vérifier que les nouveaux mots de passe correspondent
        if (formData.get('new-password') !== formData.get('confirm-password')) {
            showNotification('Les mots de passe ne correspondent pas', 'error');
            return;
        }

        try {
            const response = await fetch('/api/user/password', {
                method: 'PUT',
                headers: {
                    'Content-Type': 'application/json',
                    'Authorization': `Bearer ${localStorage.getItem('authToken')}`
                },
                body: JSON.stringify({
                    currentPassword: formData.get('current-password'),
                    newPassword: formData.get('new-password')
                })
            });

            if (!response.ok) throw new Error('Erreur de changement de mot de passe');

            showNotification('Mot de passe modifié avec succès', 'success');
            passwordForm.reset();
            
        } catch (error) {
            showNotification('Erreur lors du changement de mot de passe', 'error');
        }
    });
}

// Chargement des commandes
async function loadOrders() {
    const ordersList = document.getElementById('orders-list');
    ordersList.innerHTML = '<div class="loading">Chargement des commandes...</div>';

    try {
        const response = await fetch('/api/user/orders', {
            headers: {
                'Authorization': `Bearer ${localStorage.getItem('authToken')}`
            }
        });

        if (!response.ok) throw new Error('Erreur de chargement des commandes');

        const orders = await response.json();
        displayOrders(orders);
        
    } catch (error) {
        ordersList.innerHTML = '<div class="error">Erreur de chargement des commandes</div>';
    }
}

// Affichage des commandes
function displayOrders(orders) {
    const ordersList = document.getElementById('orders-list');
    ordersList.innerHTML = '';

    if (orders.length === 0) {
        ordersList.innerHTML = '<div class="empty-state">Aucune commande à afficher</div>';
        return;
    }

    orders.forEach(order => {
        const orderCard = createOrderCard(order);
        ordersList.appendChild(orderCard);
    });
}

// Création d'une carte de commande
function createOrderCard(order) {
    const orderCard = document.createElement('div');
    orderCard.className = 'order-card';
    orderCard.innerHTML = `
        <div class="order-header">
            <div class="order-info">
                <h3>Commande #${order.id}</h3>
                <p class="order-date">${new Date(order.date).toLocaleDateString()}</p>
            </div>
            <div class="order-status ${order.status.toLowerCase()}">
                ${order.status}
            </div>
        </div>
        <div class="order-content">
            <div class="order-items">
                ${order.items.map(item => `
                    <div class="order-item">
                        <img src="${item.image}" alt="${item.name}">
                        <div class="item-details">
                            <h4>${item.name}</h4>
                            <p>Quantité: ${item.quantity}</p>
                            <p>Prix: ${item.price.toFixed(2)}€</p>
                        </div>
                    </div>
                `).join('')}
            </div>
            <div class="order-total">
                <p>Total: ${order.total.toFixed(2)}€</p>
            </div>
        </div>
        <div class="order-actions">
            <button onclick="downloadInvoice(${order.id})">
                Télécharger la facture
            </button>
        </div>
    `;
    return orderCard;
}

// Notification utilisateur
function showNotification(message, type = 'info') {
    const notification = document.createElement('div');
    notification.className = `notification ${type}`;
    notification.textContent = message;

    document.body.appendChild(notification);

    // Animation d'entrée
    setTimeout(() => notification.classList.add('show'), 100);

    // Suppression automatique
    setTimeout(() => {
        notification.classList.remove('show');
        setTimeout(() => notification.remove(), 300);
    }, 3000);
}

// Déconnexion
document.getElementById('logout-btn').addEventListener('click', (e) => {
    e.preventDefault();
    localStorage.removeItem('authToken');
    window.location.href = 'login.html';
});
```

## 2. Page d'Inscription (register.html)
```html
<!DOCTYPE html>
<html lang="fr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>FMP - Inscription</title>
    <link rel="stylesheet" href="../static/css/style.css">
    <link rel="stylesheet" href="../static/css/auth.css">
</head>
<body>
    <header>
        <!-- Navigation standard -->
    </header>

    <main class="auth-container">
        <div class="auth-box">
            <div class="auth-header">
                <h1>Créer un compte</h1>
                <p>Rejoignez FMP et profitez de tous nos services</p>
            </div>

            <form id="register-form" class="auth-form">
                <!-- Message d'erreur -->
                <div id="error-message" class="error-message" style="display: none;"></div>

                <!-- Nom et prénom -->
                <div class="form-row">
                    <div class="form-group">
                        <label for="firstname">Prénom</label>
                        <input type="text" id="firstname" name="firstname" required>
                    </div>
                    <div class="form-group">
                        <label for="lastname">Nom</label>
                        <input type="text" id="lastname" name="lastname" required>
                    </div>
                </div>

                <!-- Email -->
                <div class="form-group">
                    <label for="email">Email</label>
                    <input type="email" id="email" name="email" required>
                </div>

                <!-- Mot de passe -->
                <div class="form-group">
                    <label for="password">Mot de passe</label>
                    <div class="password-input">
                        <input type="password" id="password" name="password" required>
                        <button type="button" class="toggle-password">
                            <i class="icon-eye"></i>
                        </button>
                    </div>
                    <div class="password-requirements">
                        <p>Le mot de passe doit contenir :</p>
                        <ul>
                            <li data-requirement="length">8 caractères minimum</li>
                            <li data-requirement="uppercase">Une majuscule</li>
                            <li data-requirement="lowercase">Une minuscule</li>
                            <li data-requirement="number">Un chiffre</li>
                            <li data-requirement="special">Un caractère spécial</li>
                        </ul>
                    </div>
                </div>

                <!-- Confirmation mot de passe -->
                <div class="form-group">
                    <label for="confirm-password">Confirmer le mot de passe</label>
                    <input type="password" id="confirm-password" name="confirm-password" required>
                </div>

                <!-- Conditions d'utilisation -->
                <div class="form-group">
                    <label class="checkbox-label">
                        <input type="checkbox" name="terms" required>
                        <span>J'accepte les <a href="#">conditions d'utilisation</a></span>
                    </label>
                </div>

                <!-- Newsletter -->
                <div class="form-group">
                    <label class="checkbox-label">
                        <input type="checkbox" name="newsletter">
                        <span>Je souhaite recevoir la newsletter</span>
                    </label>
                </div>

                <!-- Bouton d'inscription -->
                <button type="submit" class="auth-button">
                    Créer mon compte
                </button>
            </form>

            <!-- Lien de connexion -->
            <div class="auth-footer">
                <p>Déjà un compte ?</p>
                <a href="login.html" class="login-link">
                    Se connecter
                </a>
            </div>
        </div>
    </main>

    <footer>
        <!-- Footer standard -->
    </footer>

    <script src="../static/js/api.js"></script>
    <script src="../static/js/register.js"></script>
</body>
</html>
```

Je peux maintenant continuer avec :
1. La logique d'inscription (register.js)
2. Les styles supplémentaires pour l'inscription
3. La validation des formulaires

# Frontend FMP - Partie 4 : Inscription et Validation

## 1. Logique d'Inscription (register.js)
```javascript
document.addEventListener('DOMContentLoaded', () => {
    setupRegistrationForm();
    setupPasswordValidation();
    setupPasswordToggle();
});

// Configuration du formulaire d'inscription
function setupRegistrationForm() {
    const form = document.getElementById('register-form');
    
    form.addEventListener('submit', async (e) => {
        e.preventDefault();
        
        // Vérifier la validation du formulaire
        if (!validateForm()) return;

        const formData = new FormData(form);
        const userData = {
            firstname: formData.get('firstname'),
            lastname: formData.get('lastname'),
            email: formData.get('email'),
            password: formData.get('password'),
            newsletter: formData.get('newsletter') ? true : false
        };

        try {
            const response = await fetch('/api/auth/register', {
                method: 'POST',
                headers: {
                    'Content-Type': 'application/json'
                },
                body: JSON.stringify(userData)
            });

            const data = await response.json();

            if (response.ok) {
                // Sauvegarder le token
                localStorage.setItem('authToken', data.token);
                
                // Rediriger vers la page de confirmation
                window.location.href = 'register-success.html';
            } else {
                showError(data.message || 'Erreur lors de l\'inscription');
            }
        } catch (error) {
            showError('Erreur de connexion au serveur');
        }
    });
}

// Validation du formulaire
function validateForm() {
    const form = document.getElementById('register-form');
    let isValid = true;

    // Validation du mot de passe
    const password = form.querySelector('#password').value;
    const confirmPassword = form.querySelector('#confirm-password').value;

    if (password !== confirmPassword) {
        showError('Les mots de passe ne correspondent pas');
        isValid = false;
    }

    // Validation des exigences du mot de passe
    if (!isPasswordValid(password)) {
        showError('Le mot de passe ne respecte pas les exigences');
        isValid = false;
    }

    // Validation des conditions d'utilisation
    if (!form.querySelector('[name="terms"]').checked) {
        showError('Vous devez accepter les conditions d\'utilisation');
        isValid = false;
    }

    return isValid;
}

// Validation en temps réel du mot de passe
function setupPasswordValidation() {
    const passwordInput = document.getElementById('password');
    const requirements = document.querySelectorAll('.password-requirements li');

    passwordInput.addEventListener('input', () => {
        const password = passwordInput.value;

        // Vérifier chaque exigence
        requirements.forEach(requirement => {
            const type = requirement.getAttribute('data-requirement');
            const isValid = checkPasswordRequirement(password, type);
            
            requirement.classList.toggle('valid', isValid);
            requirement.classList.toggle('invalid', !isValid);
        });
    });
}

// Vérification des exigences du mot de passe
function checkPasswordRequirement(password, type) {
    const requirements = {
        length: password.length >= 8,
        uppercase: /[A-Z]/.test(password),
        lowercase: /[a-z]/.test(password),
        number: /[0-9]/.test(password),
        special: /[!@#$%^&*]/.test(password)
    };

    return requirements[type];
}

// Vérification complète du mot de passe
function isPasswordValid(password) {
    return checkPasswordRequirement(password, 'length') &&
           checkPasswordRequirement(password, 'uppercase') &&
           checkPasswordRequirement(password, 'lowercase') &&
           checkPasswordRequirement(password, 'number') &&
           checkPasswordRequirement(password, 'special');
}

// Affichage/masquage du mot de passe
function setupPasswordToggle() {
    const toggleButtons = document.querySelectorAll('.toggle-password');
    
    toggleButtons.forEach(button => {
        button.addEventListener('click', () => {
            const input = button.previousElementSibling;
            const type = input.getAttribute('type') === 'password' ? 'text' : 'password';
            input.setAttribute('type', type);
            
            // Changer l'icône
            const icon = button.querySelector('i');
            icon.className = type === 'password' ? 'icon-eye' : 'icon-eye-off';
        });
    });
}

// Gestion des erreurs
function showError(message) {
    const errorDiv = document.getElementById('error-message');
    errorDiv.textContent = message;
    errorDiv.style.display = 'block';

    // Masquer après 5 secondes
    setTimeout(() => {
        errorDiv.style.display = 'none';
    }, 5000);
}
```

## 2. Styles supplémentaires pour l'inscription (auth.css)
```css
/* Styles spécifiques à l'inscription */

/* Layout formulaire */
.form-row {
    display: grid;
    grid-template-columns: 1fr 1fr;
    gap: 1rem;
}

/* Requirements du mot de passe */
.password-requirements {
    margin-top: 0.5rem;
    font-size: 0.9rem;
    color: #666;
}

.password-requirements ul {
    list-style: none;
    padding: 0;
    margin: 0.5rem 0;
}

.password-requirements li {
    display: flex;
    align-items: center;
    margin: 0.25rem 0;
}

.password-requirements li::before {
    content: '•';
    color: #ddd;
    margin-right: 0.5rem;
    transition: color 0.3s;
}

.password-requirements li.valid::before {
    content: '✓';
    color: #27ae60;
}

.password-requirements li.invalid::before {
    content: '×';
    color: #e74c3c;
}

/* Checkboxes stylisées */
.checkbox-label {
    display: flex;
    align-items: flex-start;
    gap: 0.5rem;
    cursor: pointer;
}

.checkbox-label input[type="checkbox"] {
    width: 18px;
    height: 18px;
    margin-top: 0.2rem;
}

.checkbox-label span {
    font-size: 0.9rem;
    color: #666;
}

.checkbox-label a {
    color: #3498db;
    text-decoration: none;
}

.checkbox-label a:hover {
    text-decoration: underline;
}

/* Message de succès */
.success-message {
    text-align: center;
    padding: 2rem;
    background: #d4edda;
    border-radius: 8px;
    margin: 2rem 0;
}

.success-message h2 {
    color: #155724;
    margin-bottom: 1rem;
}

.success-message p {
    color: #2c3e50;
    margin-bottom: 1.5rem;
}

.success-actions {
    display: flex;
    justify-content: center;
    gap: 1rem;
}

/* Responsive */
@media (max-width: 480px) {
    .form-row {
        grid-template-columns: 1fr;
    }
}
```

## 3. Page de Succès (register-success.html)
```html
<!DOCTYPE html>
<html lang="fr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>FMP - Inscription réussie</title>
    <link rel="stylesheet" href="../static/css/style.css">
    <link rel="stylesheet" href="../static/css/auth.css">
</head>
<body>
    <header>
        <!-- Navigation standard -->
    </header>

    <main class="auth-container">
        <div class="auth-box">
            <div class="success-message">
                <i class="icon-check-circle"></i>
                <h2>Inscription réussie !</h2>
                <p>
                    Votre compte a été créé avec succès. Un email de confirmation
                    a été envoyé à votre adresse email.
                </p>
                <div class="success-actions">
                    <a href="account.html" class="primary-button">
                        Accéder à mon compte
                    </a>
                    <a href="products.html" class="secondary-button">
                        Voir les produits
                    </a>
                </div>
            </div>
        </div>
    </main>

    <footer>
        <!-- Footer standard -->
    </footer>
</body>
</html>
```

## Points Clés de la Validation

1. **Validation des Champs**
   - Vérification des champs requis
   - Validation du format email
   - Vérification de la force du mot de passe
   - Correspondance des mots de passe

2. **Sécurité**
   - Hashage du mot de passe côté serveur
   - Protection contre les injections
   - Validation des tokens
   - Gestion des sessions

3. **Expérience Utilisateur**
   - Feedback en temps réel
   - Messages d'erreur clairs
   - Indicateurs visuels
   - Redirection intelligente

La partie 4 est maintenant complète avec :
- Système d'authentification
- Espace client
- Gestion du profil
- Inscription sécurisée
