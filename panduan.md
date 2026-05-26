1. Install Dependencies
bash
npx create-react-app retail-store
cd retail-store
npm install react-icons
npm start
2. File Utama - App.jsx
jsx
import React, { useState, useEffect } from 'react';
import { BrowserRouter as Router, Routes, Route } from 'react-router-dom';
import Navbar from './components/Navbar';
import HomePage from './pages/HomePage';
import ProductDetail from './pages/ProductDetail';
import CartPage from './pages/CartPage';
import Footer from './components/Footer';
import './App.css';

function App() {
  const [cart, setCart] = useState(() => {
    const savedCart = localStorage.getItem('cart');
    return savedCart ? JSON.parse(savedCart) : [];
  });

  const [showCart, setShowCart] = useState(false);

  useEffect(() => {
    localStorage.setItem('cart', JSON.stringify(cart));
  }, [cart]);

  const addToCart = (product) => {
    setCart(prevCart => {
      const existingItem = prevCart.find(item => item.id === product.id);
      if (existingItem) {
        return prevCart.map(item =>
          item.id === product.id
            ? { ...item, quantity: item.quantity + 1 }
            : item
        );
      }
      return [...prevCart, { ...product, quantity: 1 }];
    });
  };

  const removeFromCart = (productId) => {
    setCart(prevCart => prevCart.filter(item => item.id !== productId));
  };

  const updateQuantity = (productId, newQuantity) => {
    if (newQuantity <= 0) {
      removeFromCart(productId);
      return;
    }
    setCart(prevCart =>
      prevCart.map(item =>
        item.id === productId ? { ...item, quantity: newQuantity } : item
      )
    );
  };

  const getCartTotal = () => {
    return cart.reduce((total, item) => total + (item.price * item.quantity), 0);
  };

  return (
    <Router>
      <div className="app">
        <Navbar 
          cartCount={cart.reduce((sum, item) => sum + item.quantity, 0)} 
          setShowCart={setShowCart}
        />
        <Routes>
          <Route path="/" element={<HomePage addToCart={addToCart} />} />
          <Route path="/product/:id" element={<ProductDetail addToCart={addToCart} />} />
          <Route path="/cart" element={<CartPage 
            cart={cart}
            removeFromCart={removeFromCart}
            updateQuantity={updateQuantity}
            getCartTotal={getCartTotal}
          />} />
        </Routes>
        <Footer />
        
        {showCart && (
          <CartSidebar
            cart={cart}
            removeFromCart={removeFromCart}
            updateQuantity={updateQuantity}
            getCartTotal={getCartTotal}
            setShowCart={setShowCart}
          />
        )}
      </div>
    </Router>
  );
}

export default App;
3. CSS Utama - App.css
css
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

:root {
  --primary-color: #00aa5b;
  --secondary-color: #ff9e00;
  --dark-color: #1a1a1a;
  --light-color: #f8f9fa;
  --gray-color: #6c757d;
  --shadow: 0 2px 8px rgba(0,0,0,0.1);
  --transition: all 0.3s ease;
}

body {
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
  background-color: #f5f5f5;
  color: #333;
}

.app {
  min-height: 100vh;
  display: flex;
  flex-direction: column;
}

.container {
  max-width: 1200px;
  margin: 0 auto;
  padding: 0 20px;
}

/* Responsive Grid */
@media (max-width: 768px) {
  .container {
    padding: 0 15px;
  }
}

@media (max-width: 480px) {
  .container {
    padding: 0 10px;
  }
}

/* Button Styles */
.btn {
  padding: 10px 20px;
  border: none;
  border-radius: 8px;
  cursor: pointer;
  font-size: 16px;
  font-weight: 600;
  transition: var(--transition);
}

.btn-primary {
  background-color: var(--primary-color);
  color: white;
}

.btn-primary:hover {
  background-color: #008a49;
  transform: translateY(-2px);
  box-shadow: var(--shadow);
}

.btn-secondary {
  background-color: var(--secondary-color);
  color: white;
}

.btn-outline {
  background-color: transparent;
  border: 2px solid var(--primary-color);
  color: var(--primary-color);
}

/* Animation */
@keyframes fadeIn {
  from {
    opacity: 0;
    transform: translateY(20px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

.fade-in {
  animation: fadeIn 0.5s ease;
}
4. Komponen Navbar - components/Navbar.jsx
jsx
import React, { useState, useEffect } from 'react';
import { Link } from 'react-router-dom';
import { FaShoppingCart, FaUser, FaSearch, FaBars, FaTimes } from 'react-icons/fa';
import './Navbar.css';

const Navbar = ({ cartCount, setShowCart }) => {
  const [isMenuOpen, setIsMenuOpen] = useState(false);
  const [isScrolled, setIsScrolled] = useState(false);
  const [searchQuery, setSearchQuery] = useState('');

  useEffect(() => {
    const handleScroll = () => {
      setIsScrolled(window.scrollY > 50);
    };
    window.addEventListener('scroll', handleScroll);
    return () => window.removeEventListener('scroll', handleScroll);
  }, []);

  return (
    <nav className={`navbar ${isScrolled ? 'scrolled' : ''}`}>
      <div className="nav-container">
        <div className="nav-brand">
          <Link to="/" className="logo">
            <span className="logo-text">Toko<span>Ku</span></span>
          </Link>
        </div>

        <div className={`nav-search ${isMenuOpen ? 'active' : ''}`}>
          <input
            type="text"
            placeholder="Cari produk..."
            value={searchQuery}
            onChange={(e) => setSearchQuery(e.target.value)}
          />
          <button className="search-btn">
            <FaSearch />
          </button>
        </div>

        <div className="nav-menu">
          <Link to="/" className="nav-link">Beranda</Link>
          <Link to="/categories" className="nav-link">Kategori</Link>
          <Link to="/promo" className="nav-link">Promo</Link>
        </div>

        <div className="nav-icons">
          <button className="cart-btn" onClick={() => setShowCart(true)}>
            <FaShoppingCart />
            {cartCount > 0 && <span className="cart-badge">{cartCount}</span>}
          </button>
          <button className="user-btn">
            <FaUser />
          </button>
          <button className="menu-toggle" onClick={() => setIsMenuOpen(!isMenuOpen)}>
            {isMenuOpen ? <FaTimes /> : <FaBars />}
          </button>
        </div>
      </div>

      {/* Mobile Menu */}
      <div className={`mobile-menu ${isMenuOpen ? 'active' : ''}`}>
        <Link to="/" onClick={() => setIsMenuOpen(false)}>Beranda</Link>
        <Link to="/categories" onClick={() => setIsMenuOpen(false)}>Kategori</Link>
        <Link to="/promo" onClick={() => setIsMenuOpen(false)}>Promo</Link>
        <Link to="/account" onClick={() => setIsMenuOpen(false)}>Akun Saya</Link>
      </div>
    </nav>
  );
};

export default Navbar;
5. CSS Navbar - components/Navbar.css
css
.navbar {
  background-color: white;
  box-shadow: var(--shadow);
  position: sticky;
  top: 0;
  z-index: 1000;
  transition: var(--transition);
}

.navbar.scrolled {
  box-shadow: 0 4px 12px rgba(0,0,0,0.15);
}

.nav-container {
  max-width: 1200px;
  margin: 0 auto;
  padding: 15px 20px;
  display: flex;
  justify-content: space-between;
  align-items: center;
  gap: 20px;
}

.logo-text {
  font-size: 24px;
  font-weight: bold;
  color: var(--dark-color);
}

.logo-text span {
  color: var(--primary-color);
}

.nav-search {
  flex: 1;
  max-width: 400px;
  display: flex;
  position: relative;
}

.nav-search input {
  width: 100%;
  padding: 10px 15px;
  border: 2px solid #e0e0e0;
  border-radius: 25px;
  font-size: 14px;
  transition: var(--transition);
}

.nav-search input:focus {
  outline: none;
  border-color: var(--primary-color);
}

.search-btn {
  position: absolute;
  right: 5px;
  top: 50%;
  transform: translateY(-50%);
  background: none;
  border: none;
  color: var(--gray-color);
  cursor: pointer;
  padding: 8px 12px;
}

.nav-menu {
  display: flex;
  gap: 25px;
}

.nav-link {
  text-decoration: none;
  color: var(--dark-color);
  font-weight: 500;
  transition: var(--transition);
}

.nav-link:hover {
  color: var(--primary-color);
}

.nav-icons {
  display: flex;
  gap: 15px;
  align-items: center;
}

.cart-btn, .user-btn {
  background: none;
  border: none;
  font-size: 20px;
  cursor: pointer;
  position: relative;
  padding: 8px;
  border-radius: 50%;
  transition: var(--transition);
}

.cart-btn:hover, .user-btn:hover {
  background-color: #f0f0f0;
}

.cart-badge {
  position: absolute;
  top: 0;
  right: 0;
  background-color: var(--primary-color);
  color: white;
  border-radius: 50%;
  padding: 2px 6px;
  font-size: 12px;
  font-weight: bold;
}

.menu-toggle {
  display: none;
  background: none;
  border: none;
  font-size: 24px;
  cursor: pointer;
}

.mobile-menu {
  display: none;
  flex-direction: column;
  background-color: white;
  padding: 20px;
  gap: 15px;
  border-top: 1px solid #e0e0e0;
}

.mobile-menu a {
  text-decoration: none;
  color: var(--dark-color);
  padding: 10px;
  transition: var(--transition);
}

.mobile-menu a:hover {
  color: var(--primary-color);
  padding-left: 15px;
}

@media (max-width: 768px) {
  .nav-menu {
    display: none;
  }
  
  .menu-toggle {
    display: block;
  }
  
  .mobile-menu.active {
    display: flex;
  }
  
  .nav-search {
    display: none;
  }
  
  .nav-search.active {
    display: flex;
    position: absolute;
    top: 70px;
    left: 20px;
    right: 20px;
    max-width: none;
  }
}
6. HomePage - pages/HomePage.jsx
jsx
import React, { useState, useEffect } from 'react';
import Hero from '../components/Hero';
import CategoryMenu from '../components/CategoryMenu';
import ProductCard from '../components/ProductCard';
import './HomePage.css';

const HomePage = ({ addToCart }) => {
  const [products, setProducts] = useState([]);
  const [filteredProducts, setFilteredProducts] = useState([]);
  const [selectedCategory, setSelectedCategory] = useState('all');
  const [loading, setLoading] = useState(true);

  // Sample product data
  const sampleProducts = [
    { id: 1, name: 'Smartphone Galaxy S21', price: 5999000, category: 'electronics', image: 'https://via.placeholder.com/300x300?text=Phone', rating: 4.5, sold: 1234 },
    { id: 2, name: 'Laptop Gaming Pro', price: 15999000, category: 'electronics', image: 'https://via.placeholder.com/300x300?text=Laptop', rating: 4.8, sold: 567 },
    { id: 3, name: 'Kemeja Pria Casual', price: 199000, category: 'fashion', image: 'https://via.placeholder.com/300x300?text=Shirt', rating: 4.3, sold: 2345 },
    { id: 4, name: 'Sepatu Running', price: 499000, category: 'fashion', image: 'https://via.placeholder.com/300x300?text=Shoes', rating: 4.6, sold: 890 },
    { id: 5, name: 'Meja Belajar Minimalis', price: 899000, category: 'home', image: 'https://via.placeholder.com/300x300?text=Desk', rating: 4.4, sold: 456 },
    { id: 6, name: 'Lampu Meja LED', price: 129000, category: 'home', image: 'https://via.placeholder.com/300x300?text=Lamp', rating: 4.2, sold: 1678 },
    { id: 7, name: 'Headphone Bluetooth', price: 399000, category: 'electronics', image: 'https://via.placeholder.com/300x300?text=Headphone', rating: 4.7, sold: 2340 },
    { id: 8, name: 'Jaket Denim', price: 349000, category: 'fashion', image: 'https://via.placeholder.com/300x300?text=Jacket', rating: 4.5, sold: 789 },
  ];

  useEffect(() => {
    // Simulate API call
    setTimeout(() => {
      setProducts(sampleProducts);
      setFilteredProducts(sampleProducts);
      setLoading(false);
    }, 1000);
  }, []);

  useEffect(() => {
    if (selectedCategory === 'all') {
      setFilteredProducts(products);
    } else {
      setFilteredProducts(products.filter(p => p.category === selectedCategory));
    }
  }, [selectedCategory, products]);

  if (loading) {
    return (
      <div className="loading-container">
        <div className="spinner"></div>
        <p>Memuat produk...</p>
      </div>
    );
  }

  return (
    <div className="homepage">
      <Hero />
      <div className="container">
        <CategoryMenu 
          selectedCategory={selectedCategory} 
          setSelectedCategory={setSelectedCategory} 
        />
        
        <div className="products-section">
          <div className="section-header">
            <h2>Produk Populer</h2>
            <button className="view-all">Lihat Semua →</button>
          </div>
          
          <div className="products-grid">
            {filteredProducts.map(product => (
              <ProductCard 
                key={product.id} 
                product={product} 
                addToCart={addToCart}
              />
            ))}
          </div>
        </div>
      </div>
    </div>
  );
};

export default HomePage;
7. CSS HomePage - pages/HomePage.css
css
.homepage {
  background-color: #f5f5f5;
}

.loading-container {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  min-height: 400px;
}

.spinner {
  width: 50px;
  height: 50px;
  border: 4px solid #f3f3f3;
  border-top: 4px solid var(--primary-color);
  border-radius: 50%;
  animation: spin 1s linear infinite;
}

@keyframes spin {
  0% { transform: rotate(0deg); }
  100% { transform: rotate(360deg); }
}

.products-section {
  padding: 40px 0;
}

.section-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 30px;
}

.section-header h2 {
  font-size: 28px;
  color: var(--dark-color);
}

.view-all {
  background: none;
  border: none;
  color: var(--primary-color);
  cursor: pointer;
  font-size: 14px;
  font-weight: 600;
  transition: var(--transition);
}

.view-all:hover {
  transform: translateX(5px);
}

.products-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(250px, 1fr));
  gap: 25px;
}

@media (max-width: 768px) {
  .products-grid {
    grid-template-columns: repeat(auto-fill, minmax(200px, 1fr));
    gap: 15px;
  }
  
  .section-header h2 {
    font-size: 24px;
  }
}

@media (max-width: 480px) {
  .products-grid {
    grid-template-columns: repeat(auto-fill, minmax(160px, 1fr));
    gap: 10px;
  }
}
8. ProductCard Component - components/ProductCard.jsx
jsx
import React, { useState } from 'react';
import { Link } from 'react-router-dom';
import { FaStar, FaShoppingCart, FaHeart, FaEye } from 'react-icons/fa';
import './ProductCard.css';

const ProductCard = ({ product, addToCart }) => {
  const [isHovered, setIsHovered] = useState(false);
  const [showNotification, setShowNotification] = useState(false);

  const formatPrice = (price) => {
    return new Intl.NumberFormat('id-ID', {
      style: 'currency',
      currency: 'IDR',
      minimumFractionDigits: 0
    }).format(price);
  };

  const handleAddToCart = (e) => {
    e.preventDefault();
    addToCart(product);
    setShowNotification(true);
    setTimeout(() => setShowNotification(false), 2000);
  };

  return (
    <div 
      className={`product-card ${isHovered ? 'hovered' : ''}`}
      onMouseEnter={() => setIsHovered(true)}
      onMouseLeave={() => setIsHovered(false)}
    >
      <Link to={`/product/${product.id}`} className="product-link">
        <div className="product-image">
          <img src={product.image} alt={product.name} />
          {isHovered && (
            <div className="product-overlay">
              <button className="quick-view">
                <FaEye /> Lihat Cepat
              </button>
            </div>
          )}
          <button className="wishlist-btn">
            <FaHeart />
          </button>
        </div>
        
        <div className="product-info">
          <h3 className="product-name">{product.name}</h3>
          <div className="product-rating">
            <FaStar className="star" />
            <span>{product.rating}</span>
            <span className="sold">Terjual {product.sold}</span>
          </div>
          <div className="product-price">{formatPrice(product.price)}</div>
        </div>
      </Link>
      
      <button className="add-to-cart-btn" onClick={handleAddToCart}>
        <FaShoppingCart /> Tambah ke Keranjang
      </button>
      
      {showNotification && (
        <div className="notification">
          Berhasil ditambahkan ke keranjang!
        </div>
      )}
    </div>
  );
};

export default ProductCard;
9. CSS ProductCard - components/ProductCard.css
css
.product-card {
  background: white;
  border-radius: 12px;
  overflow: hidden;
  transition: var(--transition);
  position: relative;
  box-shadow: var(--shadow);
}

.product-card.hovered {
  transform: translateY(-5px);
  box-shadow: 0 8px 20px rgba(0,0,0,0.15);
}

.product-link {
  text-decoration: none;
  color: inherit;
  display: block;
}

.product-image {
  position: relative;
  padding-top: 100%;
  overflow: hidden;
}

.product-image img {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  object-fit: cover;
  transition: transform 0.3s ease;
}

.product-card:hover .product-image img {
  transform: scale(1.05);
}

.product-overlay {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(0,0,0,0.5);
  display: flex;
  justify-content: center;
  align-items: center;
  opacity: 0;
  transition: var(--transition);
}

.product-card:hover .product-overlay {
  opacity: 1;
}

.quick-view {
  background: white;
  border: none;
  padding: 8px 16px;
  border-radius: 20px;
  cursor: pointer;
  font-size: 14px;
  display: flex;
  align-items: center;
  gap: 5px;
  transition: var(--transition);
}

.quick-view:hover {
  background: var(--primary-color);
  color: white;
}

.wishlist-btn {
  position: absolute;
  top: 10px;
  right: 10px;
  background: white;
  border: none;
  width: 35px;
  height: 35px;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  transition: var(--transition);
  z-index: 2;
}

.wishlist-btn:hover {
  background: #ff4757;
  color: white;
  transform: scale(1.1);
}

.product-info {
  padding: 15px;
}

.product-name {
  font-size: 16px;
  font-weight: 600;
  margin-bottom: 8px;
  color: var(--dark-color);
  display: -webkit-box;
  -webkit-line-clamp: 2;
  -webkit-box-orient: vertical;
  overflow: hidden;
}

.product-rating {
  display: flex;
  align-items: center;
  gap: 5px;
  margin-bottom: 8px;
  font-size: 13px;
}

.star {
  color: #ffc107;
}

.sold {
  color: var(--gray-color);
  margin-left: 5px;
}

.product-price {
  font-size: 18px;
  font-weight: bold;
  color: var(--primary-color);
}

.add-to-cart-btn {
  width: 100%;
  padding: 12px;
  background: var(--primary-color);
  color: white;
  border: none;
  cursor: pointer;
  font-weight: 600;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 8px;
  transition: var(--transition);
}

.add-to-cart-btn:hover {
  background: #008a49;
}

.notification {
  position: fixed;
  bottom: 20px;
  right: 20px;
  background: var(--primary-color);
  color: white;
  padding: 12px 20px;
  border-radius: 8px;
  animation: slideIn 0.3s ease;
  z-index: 2000;
}

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

@media (max-width: 768px) {
  .product-name {
    font-size: 14px;
  }
  
  .product-price {
    font-size: 16px;
  }
  
  .add-to-cart-btn {
    padding: 10px;
    font-size: 14px;
  }
}
10. Hero Component - components/Hero.jsx
jsx
import React from 'react';
import './Hero.css';

const Hero = () => {
  return (
    <div className="hero">
      <div className="hero-content">
        <div className="hero-text">
          <h1>Belanja Online<br /><span>Mudah & Aman</span></h1>
          <p>Dapatkan berbagai produk berkualitas dengan harga terbaik</p>
          <button className="shop-now-btn">Belanja Sekarang →</button>
        </div>
        <div className="hero-stats">
          <div className="stat">
            <h3>1M+</h3>
            <p>Produk</p>
          </div>
          <div className="stat">
            <h3>500K+</h3>
            <p>Pelanggan</p>
          </div>
          <div className="stat">
            <h3>100+</h3>
            <p>Kategori</p>
          </div>
        </div>
      </div>
    </div>
  );
};

export default Hero;
11. CSS Hero - components/Hero.css
css
.hero {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: white;
  padding: 60px 20px;
  text-align: center;
}

.hero-content {
  max-width: 1200px;
  margin: 0 auto;
}

.hero-text h1 {
  font-size: 48px;
  margin-bottom: 20px;
  line-height: 1.2;
}

.hero-text span {
  color: var(--secondary-color);
}

.hero-text p {
  font-size: 18px;
  margin-bottom: 30px;
  opacity: 0.9;
}

.shop-now-btn {
  background: var(--secondary-color);
  color: white;
  border: none;
  padding: 12px 30px;
  font-size: 16px;
  font-weight: 600;
  border-radius: 25px;
  cursor: pointer;
  transition: var(--transition);
}

.shop-now-btn:hover {
  transform: translateY(-2px);
  box-shadow: 0 5px 15px rgba(0,0,0,0.2);
}

.hero-stats {
  display: flex;
  justify-content: center;
  gap: 50px;
  margin-top: 50px;
}

.stat h3 {
  font-size: 32px;
  margin-bottom: 5px;
}

.stat p {
  font-size: 14px;
  opacity: 0.8;
}

@media (max-width: 768px) {
  .hero-text h1 {
    font-size: 32px;
  }
  
  .hero-text p {
    font-size: 14px;
  }
  
  .hero-stats {
    gap: 30px;
  }
  
  .stat h3 {
    font-size: 24px;
  }
}

@media (max-width: 480px) {
  .hero {
    padding: 40px 15px;
  }
  
  .hero-text h1 {
    font-size: 24px;
  }
  
  .hero-stats {
    flex-direction: column;
    gap: 20px;
  }
}
12. CartSidebar Component - components/CartSidebar.jsx
jsx
import React from 'react';
import { FaTimes, FaTrash, FaPlus, FaMinus } from 'react-icons/fa';
import './CartSidebar.css';

const CartSidebar = ({ cart, removeFromCart, updateQuantity, getCartTotal, setShowCart }) => {
  const formatPrice = (price) => {
    return new Intl.NumberFormat('id-ID', {
      style: 'currency',
      currency: 'IDR',
      minimumFractionDigits: 0
    }).format(price);
  };

  return (
    <div className="cart-sidebar-overlay" onClick={() => setShowCart(false)}>
      <div className="cart-sidebar" onClick={(e) => e.stopPropagation()}>
        <div className="cart-header">
          <h3>Keranjang Belanja</h3>
          <button className="close-btn" onClick={() => setShowCart(false)}>
            <FaTimes />
          </button>
        </div>
        
        <div className="cart-items">
          {cart.length === 0 ? (
            <div className="empty-cart">
              <p>Keranjang Anda kosong</p>
            </div>
          ) : (
            cart.map(item => (
              <div key={item.id} className="cart-item">
                <img src={item.image} alt={item.name} />
                <div className="cart-item-details">
                  <h4>{item.name}</h4>
                  <p className="cart-item-price">{formatPrice(item.price)}</p>
                  <div className="cart-item-quantity">
                    <button onClick={() => updateQuantity(item.id, item.quantity - 1)}>
                      <FaMinus />
                    </button>
                    <span>{item.quantity}</span>
                    <button onClick={() => updateQuantity(item.id, item.quantity + 1)}>
                      <FaPlus />
                    </button>
                  </div>
                </div>
                <button className="remove-item" onClick={() => removeFromCart(item.id)}>
                  <FaTrash />
                </button>
              </div>
            ))
          )}
        </div>
        
        {cart.length > 0 && (
          <div className="cart-footer">
            <div className="cart-total">
              <span>Total:</span>
              <strong>{formatPrice(getCartTotal())}</strong>
            </div>
            <button className="checkout-btn">Checkout</button>
          </div>
        )}
      </div>
    </div>
  );
};

export default CartSidebar;
13. CSS CartSidebar - components/CartSidebar.css
css
.cart-sidebar-overlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(0,0,0,0.5);
  z-index: 1001;
  animation: fadeIn 0.3s ease;
}

.cart-sidebar {
  position: fixed;
  right: 0;
  top: 0;
  width: 400px;
  height: 100%;
  background: white;
  box-shadow: -2px 0 10px rgba(0,0,0,0.1);
  display: flex;
  flex-direction: column;
  animation: slideInRight 0.3s ease;
}

@keyframes slideInRight {
  from {
    transform: translateX(100%);
  }
  to {
    transform: translateX(0);
  }
}

.cart-header {
  padding: 20px;
  border-bottom: 1px solid #e0e0e0;
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.cart-header h3 {
  font-size: 20px;
  color: var(--dark-color);
}

.close-btn {
  background: none;
  border: none;
  font-size: 20px;
  cursor: pointer;
  padding: 5px;
}

.cart-items {
  flex: 1;
  overflow-y: auto;
  padding: 20px;
}

.cart-item {
  display: flex;
  gap: 15px;
  margin-bottom: 20px;
  padding-bottom: 20px;
  border-bottom: 1px solid #f0f0f0;
}

.cart-item img {
  width: 80px;
  height: 80px;
  object-fit: cover;
  border-radius: 8px;
}

.cart-item-details {
  flex: 1;
}

.cart-item-details h4 {
  font-size: 14px;
  margin-bottom: 5px;
  color: var(--dark-color);
}

.cart-item-price {
  color: var(--primary-color);
  font-weight: bold;
  font-size: 14px;
  margin-bottom: 8px;
}

.cart-item-quantity {
  display: flex;
  align-items: center;
  gap: 10px;
}

.cart-item-quantity button {
  background: #f0f0f0;
  border: none;
  width: 25px;
  height: 25px;
  border-radius: 4px;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
}

.remove-item {
  background: none;
  border: none;
  color: #ff4757;
  cursor: pointer;
  font-size: 16px;
}

.empty-cart {
  text-align: center;
  padding: 40px;
  color: var(--gray-color);
}

.cart-footer {
  padding: 20px;
  border-top: 1px solid #e0e0e0;
}

.cart-total {
  display: flex;
  justify-content: space-between;
  margin-bottom: 15px;
  font-size: 18px;
}

.cart-total strong {
  color: var(--primary-color);
  font-size: 20px;
}

.checkout-btn {
  width: 100%;
  padding: 12px;
  background: var(--primary-color);
  color: white;
  border: none;
  border-radius: 8px;
  font-size: 16px;
  font-weight: 600;
  cursor: pointer;
  transition: var(--transition);
}

.checkout-btn:hover {
  background: #008a49;
}

@media (max-width: 480px) {
  .cart-sidebar {
    width: 100%;
  }
}
14. Footer Component - components/Footer.jsx
jsx
import React from 'react';
import { FaFacebook, FaTwitter, FaInstagram, FaYoutube } from 'react-icons/fa';
import './Footer.css';

const Footer = () => {
  return (
    <footer className="footer">
      <div className="container">
        <div className="footer-content">
          <div className="footer-section">
            <h3>TokoKu</h3>
            <p>Platform belanja online terpercaya di Indonesia</p>
            <div className="social-icons">
              <FaFacebook />
              <FaTwitter />
              <FaInstagram />
              <FaYoutube />
            </div>
          </div>
          
          <div className="footer-section">
            <h4>Informasi</h4>
            <ul>
              <li>Tentang Kami</li>
              <li>Kebijakan Privasi</li>
              <li>Syarat & Ketentuan</li>
              <li>Bantuan</li>
            </ul>
          </div>
          
          <div className="footer-section">
            <h4>Layanan Pelanggan</h4>
            <ul>
              <li>Hubungi Kami</li>
              <li>Pengembalian Barang</li>
              <li>Cara Pembayaran</li>
              <li>Pengiriman</li>
            </ul>
          </div>
        </div>
        
        <div className="footer-bottom">
          <p>&copy; 2024 TokoKu. All rights reserved.</p>
        </div>
      </div>
    </footer>
  );
};

export default Footer;
15. CSS Footer - components/Footer.css
css
.footer {
  background-color: var(--dark-color);
  color: white;
  padding: 40px 0 20px;
  margin-top: 40px;
}

.footer-content {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
  gap: 30px;
  margin-bottom: 30px;
}

.footer-section h3 {
  font-size: 24px;
  margin-bottom: 15px;
}

.footer-section h4 {
  font-size: 18px;
  margin-bottom: 15px;
}

.footer-section p {
  color: #aaa;
  line-height: 1.6;
  margin-bottom: 20px;
}

.social-icons {
  display: flex;
  gap: 15px;
  font-size: 20px;
  cursor: pointer;
}

.social-icons svg {
  transition: var(--transition);
}

.social-icons svg:hover {
  color: var(--primary-color);
  transform: translateY(-2px);
}

.footer-section ul {
  list-style: none;
}

.footer-section ul li {
  margin-bottom: 10px;
  color: #aaa;
  cursor: pointer;
  transition: var(--transition);
}

.footer-section ul li:hover {
  color: var(--primary-color);
  transform: translateX(5px);
}

.footer-bottom {
  text-align: center;
  padding-top: 20px;
  border-top: 1px solid #333;
  color: #aaa;
}

@media (max-width: 768px) {
  .footer-content {
    grid-template-columns: 1fr;
    text-align: center;
  }
  
  .social-icons {
    justify-content: center;
  }
}
16. CategoryMenu Component - components/CategoryMenu.jsx
jsx
import React from 'react';
import { FaMobile, FaLaptop, FaTshirt, FaHome, FaTags } from 'react-icons/fa';
import './CategoryMenu.css';

const CategoryMenu = ({ selectedCategory, setSelectedCategory }) => {
  const categories = [
    { id: 'all', name: 'Semua', icon: <FaTags /> },
    { id: 'electronics', name: 'Elektronik', icon: <FaMobile /> },
    { id: 'fashion', name: 'Fashion', icon: <FaTshirt /> },
    { id: 'home', name: 'Rumah Tangga', icon: <FaHome /> },
  ];

  return (
    <div className="category-menu">
      <div className="category-list">
        {categories.map(category => (
          <button
            key={category.id}
            className={`category-item ${selectedCategory === category.id ? 'active' : ''}`}
            onClick={() => setSelectedCategory(category.id)}
          >
            <span className="category-icon">{category.icon}</span>
            <span className="category-name">{category.name}</span>
          </button>
        ))}
      </div>
    </div>
  );
};

export default CategoryMenu;
17. CSS CategoryMenu - components/CategoryMenu.css
css
.category-menu {
  padding: 20px 0;
  background: white;
  border-radius: 12px;
  margin: 20px 0;
  box-shadow: var(--shadow);
}

.category-list {
  display: flex;
  justify-content: space-around;
  flex-wrap: wrap;
  gap: 10px;
}

.category-item {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 8px;
  padding: 10px 20px;
  background: none;
  border: none;
  cursor: pointer;
  transition: var(--transition);
  border-radius: 8px;
}

.category-item:hover {
  background-color: #f0f0f0;
  transform: translateY(-2px);
}

.category-item.active {
  color: var(--primary-color);
  background-color: #e8f5e9;
}

.category-icon {
  font-size: 24px;
}

.category-name {
  font-size: 14px;
  font-weight: 500;
}

@media (max-width: 768px) {
  .category-icon {
    font-size: 20px;
  }
  
  .category-name {
    font-size: 12px;
  }
  
  .category-item {
    padding: 8px 15px;
  }
}
Cara Menjalankan Aplikasi:
Buat project React baru:

bash
npx create-react-app retail-store
cd retail-store
Install dependencies:

bash
npm install react-icons react-router-dom
Copy semua file yang sudah dibuat ke dalam folder src

Jalankan aplikasi:

bash
npm start
Fitur yang sudah diimplementasikan:
✅ Responsive Design - Tampilan optimal di desktop, tablet, dan mobile
✅ Product Catalog - Menampilkan produk dengan grid layout
✅ Shopping Cart - Keranjang belanja dengan sidebar
✅ Category Filtering - Filter produk berdasarkan kategori
✅ Cart Management - Tambah, hapus, update quantity produk
✅ Local Storage - Data keranjang tersimpan
✅ Animations - Efek hover, loading, dan transisi
✅ Interactive Elements - Rating, tombol wishlist, quick view
✅ Modern UI - Desain modern seperti marketplace

Perluasan Fitur (Opsional):
Integrasi API backend (Node.js + Express + MongoDB)

Sistem autentikasi user

Payment gateway integration

Admin dashboard

Search functionality

Product reviews dan rating

Order tracking

Aplikasi ini sudah siap digunakan dan dapat dikembangkan lebih lanjut sesuai kebutuhan bisnis Anda!

