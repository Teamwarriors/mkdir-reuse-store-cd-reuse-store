<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Reuse Store</title>
  <script src="https://cdn.jsdelivr.net/npm/react@18.2.0/umd/react.production.min.js"></script>
  <script src="https://cdn.jsdelivr.net/npm/react-dom@18.2.0/umd/react-dom.production.min.js"></script>
  <script src="https://cdn.jsdelivr.net/npm/babel-standalone@7.22.5/babel.min.js"></script>
  <script src="https://cdn.tailwindcss.com"></script>
</head>
<body>
  <div id="root"></div>
  <script type="text/babel">
    const { useState, useEffect, useMemo } = React;

    // Sample base64 image (placeholder, small size)
    const placeholderImage = "data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAACgAAAAoCAYAAACM/rhtAAAAAXNSR0IArs4c6QAAAERlWElmTU0AKgAAAAgAAYdpAAQAAAABAAAAGgAAAAAAA6ABAAMAAAABAAEAAKACAAQAAAABAAAAKKADAAQAAAABAAAAKAAAAAD3b3RyAAAA8ElEQVRYCe2WsQ3CMAxE7wIVKCAeABUoIB4AFShwXqO8C2QDuVkmS8nQ2dmZ2SS/3SS7k+w9yQLgA+BXC4A3gH8tAN4A/jUAeAP41wLgDeBfC4A3gH8tAN4A/jUAeAP41wLgDeBfC4A3gH8tAN4A/jUAeAP41wLgDeBfC4A3gH8tAN4A/jUAeAP41wLgDeBfC4A3gH8tAN4A/jUAeAP41wLgDeBfC4A3gH8tAN4A/jUAeAP41wLgDeBfC4A3gH8tAN4A/jUAeAP41wLgDeBfC4A3gH8tAN4A/jUAeAP41wLgDeA3C4A3gL8BfgExvD6vBZwUAAAAAElFTkSuQmCC";

    // Generate 10 sample products
    const initialProducts = Array.from({ length: 10 }, (_, i) => ({
      id: i + 1,
      name: `Thrift Item ${i + 1}`,
      category: ["Clothing", "Electronics", "Books", "Furniture"][i % 4],
      price: (10 + i * 5).toFixed(2),
      image: placeholderImage,
      description: `Quality thrift ${["shirt", "gadget", "novel", "chair"][i % 4]} in great condition.`,
    }));

    // Simulate 100 products
    const allProducts = Array.from({ length: 100 }, (_, i) => ({
      ...initialProducts[i % 10],
      id: i + 1,
      name: `Thrift Item ${i + 1}`,
    }));

    // Navigation Component
    const Navbar = ({ cartItems, setPage, isLoggedIn, setIsLoggedIn, setIsAdmin }) => {
      const handleLogout = () => {
        setIsLoggedIn(false);
        setIsAdmin(false);
        localStorage.removeItem('isLoggedIn');
        localStorage.removeItem('isAdmin');
        setPage('home');
      };

      return (
        <nav className="bg-blue-600 text-white p-4 flex justify-between items-center" aria-label="Main navigation">
          <h1
            className="text-2xl font-bold cursor-pointer"
            onClick={() => setPage('home')}
            role="button"
            tabIndex={0}
            onKeyDown={e => e.key === 'Enter' && setPage('home')}
          >
            Reuse Store
          </h1>
          <div className="space-x-4">
            <button onClick={() => setPage('home')} className="hover:underline focus:outline-none focus:ring-2 focus:ring-white">
              Home
            </button>
            <button onClick={() => setPage('categories')} className="hover:underline focus:outline-none focus:ring-2 focus:ring-white">
              Categories
            </button>
            <button onClick={() => setPage('cart')} className="hover:underline focus:outline-none focus:ring-2 focus:ring-white">
              Cart ({cartItems.reduce((sum, item) => sum + item.quantity, 0)})
            </button>
            {isLoggedIn ? (
              <button onClick={handleLogout} className="hover:underline focus:outline-none focus:ring-2 focus:ring-white">
                Logout
              </button>
            ) : (
              <button onClick={() => setPage('login')} className="hover:underline focus:outline-none focus:ring-2 focus:ring-white">
                Login
              </button>
            )}
            <button onClick={() => setPage('admin')} className="hover:underline focus:outline-none focus:ring-2 focus:ring-white">
              Admin
            </button>
          </div>
        </nav>
      );
    };

    // Home Page
    const HomePage = ({ products, addToCart }) => (
      <main className="p-6">
        <h2 className="text-3xl font-bold mb-6">Welcome to Reuse Store</h2>
        <div className="grid grid-cols-1 sm:grid-cols-2 md:grid-cols-4 gap-6">
          {products.slice(0, 8).map(product => (
            <article key={product.id} className="border p-4 rounded-lg shadow hover:shadow-lg">
              <img src={product.image} alt={product.name} className="w-full h-32 object-cover mb-4" loading="lazy" />
              <h3 className="text-lg font-semibold">{product.name}</h3>
              <p className="text-gray-600">${product.price}</p>
              <button
                onClick={() => addToCart(product)}
                className="mt-2 bg-blue-600 text-white px-4 py-2 rounded hover:bg-blue-700 focus:outline-none focus:ring-2 focus:ring-blue-500"
              >
                Add to Cart
              </button>
            </article>
          ))}
        </div>
      </main>
    );

    // Categories Page
    const CategoriesPage = ({ products, addToCart }) => {
      const categories = [...new Set(products.map(p => p.category))];
      return (
        <main className="p-6">
          <h2 className="text-3xl font-bold mb-6">Categories</h2>
          {categories.map(category => (
            <section key={category} className="mb-8">
              <h3 className="text-2xl font-semibold mb-4">{category}</h3>
              <div className="grid grid-cols-1 sm:grid-cols-2 md:grid-cols-4 gap-6">
                {products.filter(p => p.category === category).slice(0, 4).map(product => (
                  <article key={product.id} className="border p-4 rounded-lg shadow hover:shadow-lg">
                    <img src={product.image} alt={product.name} className="w-full h-32 object-cover mb-4" loading="lazy" />
                    <h4 className="text-lg font-semibold">{product.name}</h4>
                    <p className="text-gray-600">${product.price}</p>
                    <button
                      onClick={() => addToCart(product)}
                      className="mt-2 bg-blue-600 text-white px-4 py-2 rounded hover:bg-blue-700 focus:outline-none focus:ring-2 focus:ring-blue-500"
                    >
                      Add to Cart
                    </button>
                  </article>
                ))}
              </div>
            </section>
          ))}
        </main>
      );
    };

    // Cart Page
    const CartPage = ({ cartItems, removeFromCart, proceedToCheckout }) => {
      const total = useMemo(
        () => cartItems.reduce((sum, item) => sum + parseFloat(item.price) * item.quantity, 0).toFixed(2),
        [cartItems]
      );

      return (
        <main className="p-6">
          <h2 className="text-3xl font-bold mb-6">Your Cart</h2>
          {cartItems.length === 0 ? (
            <p>Your cart is empty.</p>
          ) : (
            <div>
              {cartItems.map(item => (
                <article key={item.id} className="flex items-center border-b py-4">
                  <img src={item.image} alt={item.name} className="w-16 h-16 object-cover mr-4" loading="lazy" />
                  <div className="flex-1">
                    <h3 className="text-lg font-semibold">{item.name}</h3>
                    <p className="text-gray-600">${item.price} x {item.quantity}</p>
                  </div>
                  <button
                    onClick={() => removeFromCart(item.id)}
                    className="text-red-600 hover:underline focus:outline-none focus:ring-2 focus:ring-red-500"
                    aria-label={`Remove ${item.name} from cart`}
                  >
                    Remove
                  </button>
                </article>
              ))}
              <div className="mt-6">
                <p className="text-xl font-semibold">Total: ${total}</p>
                <button
                  onClick={proceedToCheckout}
                  className="mt-4 bg-green-600 text-white px-6 py-3 rounded hover:bg-green-700 focus:outline-none focus:ring-2 focus:ring-green-500"
                >
                  Proceed to Checkout
                </button>
              </div>
            </div>
          )}
        </main>
      );
    };

    // Payment Gateway (Mock)
    const PaymentPage = ({ cartItems, setPage, setCartItems }) => {
      const [paymentStatus, setPaymentStatus] = useState('');

      const handlePayment = () => {
        setPaymentStatus('Processing...');
        setTimeout(() => {
          setPaymentStatus('Payment Successful!');
          setCartItems([]);
          localStorage.removeItem('cartItems');
          setTimeout(() => setPage('home'), 2000);
        }, 2000);
      };

      const total = useMemo(
        () => cartItems.reduce((sum, item) => sum + parseFloat(item.price) * item.quantity, 0).toFixed(2),
        [cartItems]
      );

      return (
        <main className="p-6 max-w-md mx-auto">
          <h2 className="text-3xl font-bold mb-6">Checkout</h2>
          <section className="border p-4 rounded-lg">
            <h3 className="text-xl font-semibold mb-4">Order Summary</h3>
            {cartItems.map(item => (
              <div key={item.id} className="flex justify-between mb-2">
                <span>{item.name} x {item.quantity}</span>
                <span>${(parseFloat(item.price) * item.quantity).toFixed(2)}</span>
              </div>
            ))}
            <p className="text-lg font-semibold mt-4">Total: ${total}</p>
          </section>
          <section className="mt-6">
            <h3 className="text-xl font-semibold mb-4">Payment Details</h3>
            <input
              type="text"
              placeholder="Card Number"
              className="w-full p-2 mb-4 border rounded focus:outline-none focus:ring-2 focus:ring-blue-500"
              aria-label="Card number"
            />
            <div className="flex space-x-4 mb-4">
              <input
                type="text"
                placeholder="MM/YY aload"
                className="w-1/2 p-2 border rounded focus:outline-none focus:ring-2 focus:ring-blue-500"
                aria-label="Expiration date"
              />
              <input
                type="text"
                placeholder="CVC"
                className="w-1/2 p-2 border rounded focus:outline-none focus:ring-2 focus:ring-blue-500"
                aria-label="CVC code"
              />
            </div>
            <button
              onClick={handlePayment}
              className="w-full bg-green-600 text-white px-6 py-3 rounded hover:bg-green-700 focus:outline-none focus:ring-2 focus:ring-green-500"
              disabled={paymentStatus === 'Processing...'}
            >
              {paymentStatus === 'Processing...' ? 'Processing...' : 'Pay Now'}
            </button>
            {paymentStatus && (
              <p className={`mt-4 text-center ${paymentStatus === 'Payment Successful!' ? 'text-green-600' : 'text-gray-600'}`}>
                {paymentStatus}
              </p>
            )}
          </section>
        </main>
      );
    };

    // Login Page
    const LoginPage = ({ setIsLoggedIn, setIsAdmin, setPage }) => {
      const [username, setUsername] = useState('');
      const [password, setPassword] = useState('');
      const [error, setError] = useState('');

      const handleLogin = () => {
        // Mock authentication (replace with backend in production)
        if (username === 'admin' && password === 'admin123') {
          setIsLoggedIn(true);
          setIsAdmin(true);
          localStorage.setItem('isLoggedIn', 'true');
          localStorage.setItem('isAdmin', 'true');
          setPage('admin');
        } else if (username === 'user' && password === 'user123') {
          setIsLoggedIn(true);
          setIsAdmin(false);
          localStorage.setItem('isLoggedIn', 'true');
          localStorage.setItem('isAdmin', 'false');
          setPage('home');
        } else {
          setError('Invalid username or password');
        }
      };

      return (
        <main className="p-6 max-w-md mx-auto">
          <h2 className="text-3xl font-bold mb-6">Login</h2>
          {error && <p className="text-red-600 mb-4">{error}</p>}
          <input
            type="text"
            placeholder="Username"
            value={username}
            onChange={e => setUsername(e.target.value)}
            className="w-full p-2 mb-4 border rounded focus:outline-none focus:ring-2 focus:ring-blue-500"
            aria-label="Username"
          />
          <input
            type="password"
            placeholder="Password"
            value={password}
            onChange={e => setPassword(e.target.value)}
            className="w-full p-2 mb-4 border rounded focus:outline-none focus:ring-2 focus:ring-blue-500"
            aria-label="Password"
          />
          <button
            onClick={handleLogin}
            className="w-full bg-blue-600 text-white px-6 py-3 rounded hover:bg-blue-700 focus:outline-none focus:ring-2 focus:ring-blue-500"
          >
            Login
          </button>
        </main>
      );
    };

    // Admin Page
    const AdminPage = ({ products, setProducts, isAdmin }) => {
      const [newProduct, setNewProduct] = useState({
        name: '',
        category: '',
        price: '',
        image: placeholderImage,
        description: '',
      });
      const [error, setError] = useState('');

      if (!isAdmin) return <main className="p-6">Access Denied</main>;

      const addProduct = () => {
        if (!newProduct.name || !newProduct.category || !newProduct.price || isNaN(newProduct.price) || parseFloat(newProduct.price) <= 0) {
          setError('Please fill all fields with valid data (price must be positive)');
          return;
        }
        const product = {
          id: products.length + 1,
          ...newProduct,
          price: parseFloat(newProduct.price).toFixed(2),
        };
        setProducts([...products, product]);
        setNewProduct({ name: '', category: '', price: '', image: placeholderImage, description: '' });
        setError('');
      };

      const deleteProduct = id => {
        setProducts(products.filter(p => p.id !== id));
      };

      return (
        <main className="p-6">
          <h2 className="text-3xl font-bold mb-6">Admin Panel</h2>
          <section className="mb-8">
            <h3 className="text-2xl font-semibold mb-4">Add New Product</h3>
            {error && <p className="text-red-600 mb-4">{error}</p>}
            <input
              type="text"
              placeholder="Name"
              value={newProduct.name}
              onChange={e => setNewProduct({ ...newProduct, name: e.target.value })}
              className="w-full p-2 mb-4 border rounded focus:outline-none focus:ring-2 focus:ring-blue-500"
              aria-label="Product name"
            />
            <input
              type="text"
              placeholder="Category"
              value={newProduct.category}
              onChange={e => setNewProduct({ ...newProduct, category: e.target.value })}
              className="w-full p-2 mb-4 border rounded focus:outline-none focus:ring-2 focus:ring-blue-500"
              aria-label="Product category"
            />
            <input
              type="number"
              placeholder="Price"
              value={newProduct.price}
              onChange={e => setNewProduct({ ...newProduct, price: e.target.value })}
              className="w-full p-2 mb-4 border rounded focus:outline-none focus:ring-2 focus:ring-blue-500"
              aria-label="Product price"
              min="0"
              step="0.01"
            />
            <input
              type="text"
              placeholder="Description"
              value={newProduct.description}
              onChange={e => setNewProduct({ ...newProduct, description: e.target.value })}
              className="w-full p-2 mb-4 border rounded focus:outline-none focus:ring-2 focus:ring-blue-500"
              aria-label="Product description"
            />
            <button
              onClick={addProduct}
              className="bg-blue-600 text-white px-6 py-3 rounded hover:bg-blue-700 focus:outline-none focus:ring-2 focus:ring-blue-500"
            >
              Add Product
            </button>
          </section>
          <section>
            <h3 className="text-2xl font-semibold mb-4">Manage Products</h3>
            <div className="grid grid-cols-1 sm:grid-cols-2 md:grid-cols-3 gap-6">
              {products.map(product => (
                <article key={product.id} className="border p-4 rounded-lg shadow">
                  <img src={product.image} alt={product.name} className="w-full h-32 object-cover mb-4" loading="lazy" />
                  <h4 className="text-lg font-semibold">{product.name}</h4>
                  <p className="text-gray-600">${product.price}</p>
                  <button
                    onClick={() => deleteProduct(product.id)}
                    className="mt-2 text-red-600 hover:underline focus:outline-none focus:ring-2 focus:ring-red-500"
                    aria-label={`Delete ${product.name}`}
                  >
                    Delete
                  </button>
                </article>
              ))}
            </div>
          </section>
        </main>
      );
    };

    // Main App Component
    const App = () => {
      const [page, setPage] = useState('home');
      const [cartItems, setCartItems] = useState(() => {
        return JSON.parse(localStorage.getItem('cartItems')) || [];
      });
      const [products, setProducts] = useState(allProducts);
      const [isLoggedIn, setIsLoggedIn] = useState(localStorage.getItem('isLoggedIn') === 'true');
      const [isAdmin, setIsAdmin] = useState(localStorage.getItem('isAdmin') === 'true');

      useEffect(() => {
        localStorage.setItem('cartItems', JSON.stringify(cartItems));
      }, [cartItems]);

      const addToCart = product => {
        setCartItems(prev => {
          const existing = prev.find(item => item.id === product.id);
          if (existing) {
            return prev.map(item =>
              item.id === product.id ? { ...item, quantity: item.quantity + 1 } : item
            );
          }
          return [...prev, { ...product, quantity: 1 }];
        });
      };

      const removeFromCart = id => {
        setCartItems(prev => prev.filter(item => item.id !== id));
      };

      const proceedToCheckout = () => {
        if (!isLoggedIn) {
          setPage('login');
        } else if (cartItems.length > 0) {
          setPage('payment');
        }
      };

      return (
        <div className="min-h-screen bg-gray-100">
          <Navbar
            cartItems={cartItems}
            setPage={setPage}
            isLoggedIn={isLoggedIn}
            setIsLoggedIn={setIsLoggedIn}
            setIsAdmin={setIsAdmin}
          />
          {page === 'home' && <HomePage products={products} addToCart={addToCart} />}
          {page === 'categories' && <CategoriesPage products={products} addToCart={addToCart} />}
          {page === 'cart' && (
            <CartPage
              cartItems={cartItems}
              removeFromCart={removeFromCart}
              proceedToCheckout={proceedToCheckout}
            />
          )}
          {page === 'payment' && (
            <PaymentPage cartItems={cartItems} setPage={setPage} setCartItems={setCartItems} />
          )}
          {page === 'login' && (
            <LoginPage setIsLoggedIn={setIsLoggedIn} setIsAdmin={setIsAdmin} setPage={setPage} />
          )}
          {page === 'admin' && (
            <AdminPage products={products} setProducts={setProducts} isAdmin={isAdmin} />
          )}
        </div>
      );
    };

    // Render the app
    ReactDOM.render(<App />, document.getElementById('root'));
  </script>
</body>
</html>
