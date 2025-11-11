// Simple Mini Shop app
// Data and app logic
const productsEl = document.getElementById('products');
const cartBtn = document.getElementById('cartBtn');
const cartCountEl = document.getElementById('cartCount');
const cartDrawer = document.getElementById('cartDrawer');
const closeCart = document.getElementById('closeCart');
const cartItemsEl = document.getElementById('cartItems');
const cartQtyEl = document.getElementById('cartQty');
const cartTotalEl = document.getElementById('cartTotal');
const clearCartBtn = document.getElementById('clearCart');
const checkoutBtn = document.getElementById('checkout');
const searchInput = document.getElementById('search');
const categoryFilter = document.getElementById('categoryFilter');
const sortSelect = document.getElementById('sortSelect');

const productTpl = document.getElementById('productTpl');
const cartItemTpl = document.getElementById('cartItemTpl');

// Example product data
const products = [
  { id: 'p1', title: 'Vintage Watch', price: 79.99, category: 'Accessories', desc: 'Classic leather strap watch.', img: 'https://picsum.photos/id/1005/600/400' },
  { id: 'p2', title: 'Wireless Headphones', price: 129.00, category: 'Electronics', desc: 'Noise-cancelling over-ear headphones.', img: 'https://picsum.photos/id/180/600/400' },
  { id: 'p3', title: 'Minimalist Backpack', price: 59.50, category: 'Bags', desc: 'Water-resistant everyday backpack.', img: 'https://picsum.photos/id/1018/600/400' },
  { id: 'p4', title: 'Scented Candle', price: 14.25, category: 'Home', desc: 'Soy wax with calming scent.', img: 'https://picsum.photos/id/1080/600/400' },
  { id: 'p5', title: 'Sneakers', price: 89.99, category: 'Footwear', desc: 'Comfortable urban sneakers.', img: 'https://picsum.photos/id/21/600/400' },
  { id: 'p6', title: 'Smart Lamp', price: 39.99, category: 'Electronics', desc: 'Mood lamp with app control.', img: 'https://picsum.photos/id/1062/600/400' }
];

let cart = loadCart();

// populate category filter
function initCategories(){
  const cats = Array.from(new Set(products.map(p => p.category)));
  for(const c of cats){
    const opt = document.createElement('option');
    opt.value = c;
    opt.textContent = c;
    categoryFilter.appendChild(opt);
  }
}

// render product cards
function renderProducts(list){
  productsEl.innerHTML = '';
  const frag = document.createDocumentFragment();
  for(const p of list){
    const node = productTpl.content.cloneNode(true);
    const card = node.querySelector('.product-card');
    node.querySelector('.product-img').src = p.img;
    node.querySelector('.product-img').alt = p.title;
    node.querySelector('.product-title').textContent = p.title;
    node.querySelector('.product-desc').textContent = p.desc;
    node.querySelector('.product-price').textContent = `$${p.price.toFixed(2)}`;
    const btn = node.querySelector('.add-btn');
    btn.addEventListener('click', ()=> addToCart(p.id));
    card.addEventListener('keydown', (e)=>{
      if(e.key === 'Enter') addToCart(p.id);
    });
    frag.appendChild(node);
  }
  productsEl.appendChild(frag);
}

// cart logic
function addToCart(productId){
  const p = products.find(x => x.id === productId);
  if(!p) return;
  const existing = cart.find(i => i.id === productId);
  if(existing) existing.qty += 1;
  else cart.push({ id: p.id, title: p.title, price: p.price, img: p.img, qty: 1 });
  saveCart();
  updateCartUI();
  openCart();
}

function removeFromCart(productId){
  cart = cart.filter(i => i.id !== productId);
  saveCart();
  updateCartUI();
}

function changeQty(productId, delta){
  const item = cart.find(i => i.id === productId);
  if(!item) return;
  item.qty += delta;
  if(item.qty < 1) removeFromCart(productId);
  saveCart();
  updateCartUI();
}

function clearCart(){
  cart = [];
  saveCart();
  updateCartUI();
}

// ui updates
function updateCartUI(){
  cartCountEl.textContent = cart.reduce((s,i)=>s+i.qty,0);
  cartQtyEl.textContent = cartCountEl.textContent;
  cartTotalEl.textContent = cart.reduce((s,i)=>s + i.qty * i.price, 0).toFixed(2);

  // render items
  cartItemsEl.innerHTML = '';
  const frag = document.createDocumentFragment();
  for(const it of cart){
    const node = cartItemTpl.content.cloneNode(true);
    node.querySelector('.cart-item-img').src = it.img;
    node.querySelector('.cart-item-img').alt = it.title;
    node.querySelector('.cart-item-title').textContent = it.title;
    node.querySelector('.qty').textContent = it.qty;
    node.querySelector('.item-price').textContent = `$${(it.price * it.qty).toFixed(2)}`;
    node.querySelector('.minus').addEventListener('click', ()=> changeQty(it.id, -1));
    node.querySelector('.plus').addEventListener('click', ()=> changeQty(it.id, +1));
    node.querySelector('.remove').addEventListener('click', ()=> removeFromCart(it.id));
    frag.appendChild(node);
  }
  cartItemsEl.appendChild(frag);
}

function openCart(){
  cartDrawer.hidden = false;
  cartDrawer.setAttribute('aria-hidden', 'false');
}
function closeCartDrawer(){
  cartDrawer.hidden = true;
  cartDrawer.setAttribute('aria-hidden', 'true');
}

// persistence
function saveCart(){
  try { localStorage.setItem('mini-shop-cart', JSON.stringify(cart)); }
  catch(e){ console.warn('Could not save cart', e); }
}
function loadCart(){
  try {
    const raw = localStorage.getItem('mini-shop-cart');
    return raw ? JSON.parse(raw) : [];
  } catch(e){ return []; }
}

// search and filters
function applyFilters(){
  const q = (searchInput.value || '').trim().toLowerCase();
  const cat = categoryFilter.value;
  let list = products.filter(p => {
    const okQ = !q || (p.title + ' ' + p.desc + ' ' + p.category).toLowerCase().includes(q);
    const okC = cat === 'all' || p.category === cat;
    return okQ && okC;
  });

  const sort = sortSelect.value;
  if(sort === 'price-asc') list.sort((a,b)=> a.price - b.price);
  if(sort === 'price-desc') list.sort((a,b)=> b.price - a.price);

  renderProducts(list);
}

// checkout demo
function checkout(){
  if(cart.length === 0){ alert('Your cart is empty.'); return; }
  // simulate checkout
  alert(`Order placed. Total: $${cart.reduce((s,i)=>s + i.qty * i.price, 0).toFixed(2)}\nThank you!`);
  clearCart();
  closeCartDrawer();
}

// events
cartBtn.addEventListener('click', openCart);
closeCart.addEventListener('click', closeCartDrawer);
clearCartBtn.addEventListener('click', ()=> {
  if(confirm('Clear cart?')) clearCart();
});
checkoutBtn.addEventListener('click', checkout);

searchInput.addEventListener('input', debounce(applyFilters, 250));
categoryFilter.addEventListener('change', applyFilters);
sortSelect.addEventListener('change', applyFilters);

// helper: debounce
function debounce(fn, wait=200){
  let t;
  return (...args)=> {
    clearTimeout(t);
    t = setTimeout(()=> fn(...args), wait);
  }
}

// init app
initCategories();
applyFilters();
updateCartUI();

// expose some functions for dev debugging (optional)
window.__miniShop = { products, cart, addToCart, clearCart };D11 Mega Campaign is coming! Look at these great deals!
Product Name:  3 Piece Nikab Jilbab Skirt Set, Georgette Fabric, Adjustable Waist, Modest and Stylish Muslim Attire
Product Price:  Rs.5,906
Discount Price:  Rs.5,906
https://s.daraz.pk/s.Y7pOh?cc
