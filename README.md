# Red-market
Sales
// RedSalle Marketplace - Single-file React App (MVP demo) // Branding: red & white // Purpose: buyer-facing marketplace where sellers list products and the platform takes 5% commission on each sale. // This is a frontend demo built with React + Tailwind CSS. It's intended as a preview/shell. Backend responsibilities are described below.

/* Backend (required for production):

Users: auth (JWT), roles: buyer, seller, admin

Listings: Create / Update / Delete product, with seller_id

Orders: create order, capture payment

Payments: integrate Stripe Connect (platform takes 5% application_fee_amount on each charge) OR use separate transfers

Webhooks: handle Stripe payment events and payout to sellers

Commission: enforce commission server-side when creating payouts

Database: Postgres with tables users, products, orders, payouts


Key endpoints (examples): POST /api/auth/login POST /api/auth/register GET /api/products POST /api/products (seller only) POST /api/orders -> create Stripe PaymentIntent server-side with application_fee_amount = Math.round(price * 0.05 * 100) POST /api/webhooks/stripe

Notes on Stripe Connect:

Use Express server with stripe-node.

For marketplace where you hold funds, consider Connect with 'direct charges' or 'destination charges'; application_fee_amount collects platform fee.

Comply with KYC for sellers.


Deployment:

Frontend: Vercel

Backend: Render / Heroku / Railway

DB: Managed Postgres (Render, Supabase)


This demo front-end uses local state and mock data to illustrate flows. */

import React, { useState } from 'react';

export default function App() { const [products, setProducts] = useState([ { id: 1, title: 'Handmade Scarf', price: 250.0, seller: 'Nadia' }, { id: 2, title: 'Vintage Lamp', price: 780.0, seller: 'Omar' }, { id: 3, title: 'Art Print A4', price: 120.0, seller: 'Sara' }, ]);

const [cart, setCart] = useState([]); const [view, setView] = useState('market'); const [form, setForm] = useState({ title: '', price: '', seller: '' });

function addToCart(p) { setCart(prev => [...prev, p]); }

function removeFromCart(idx) { setCart(prev => prev.filter((_, i) => i !== idx)); }

function total() { return cart.reduce((s, p) => s + p.price, 0); }

function platformFee(amount) { // 5% platform commission return +(amount * 0.05).toFixed(2); }

function checkout() { const subtotal = total(); const fee = platformFee(subtotal); const totalPayable = +(subtotal + fee).toFixed(2);

// In production: call backend to create Stripe PaymentIntent and pass application_fee_amount = Math.round(fee * 100)
alert(`Checkout summary:\nSubtotal: ${subtotal} EGP\nRedSalle fee (5%): ${fee} EGP\nTotal payable: ${totalPayable} EGP\n\nThis demo does not process real payments.`);

}

function createListing(e) { e.preventDefault(); const next = { id: Date.now(), title: form.title, price: parseFloat(form.price), seller: form.seller || 'Unknown' }; setProducts(prev => [next, ...prev]); setForm({ title: '', price: '', seller: '' }); setView('market'); }

return ( <div className="min-h-screen bg-white text-gray-900"> <header className="bg-red-600 text-white p-4"> <div className="max-w-5xl mx-auto flex items-center justify-between"> <div className="flex items-center gap-3"> <div className="w-10 h-10 rounded-md bg-white flex items-center justify-center text-red-600 font-bold">RS</div> <h1 className="text-xl font-semibold">RedSalle Marketplace</h1> </div> <nav className="flex gap-3"> <button onClick={() => setView('market')} className="px-3 py-2 rounded hover:bg-red-500">Market</button> <button onClick={() => setView('sell')} className="px-3 py-2 rounded hover:bg-red-500">Sell</button> <button onClick={() => setView('cart')} className="px-3 py-2 rounded hover:bg-red-500">Cart ({cart.length})</button> </nav> </div> </header>

<main className="max-w-5xl mx-auto p-6">
    {view === 'market' && (
      <section>
        <h2 className="text-2xl mb-4">Products</h2>
        <div className="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 gap-6">
          {products.map(p => (
            <div key={p.id} className="border rounded p-4 shadow-sm">
              <div className="h-40 bg-red-50 rounded flex items-center justify-center text-red-600 font-semibold">Image</div>
              <h3 className="mt-3 font-semibold">{p.title}</h3>
              <p className="text-sm text-gray-600">Seller: {p.seller}</p>
              <p className="mt-2 font-bold">{p.price} EGP</p>
              <div className="mt-3 flex gap-2">
                <button onClick={() => addToCart(p)} className="flex-1 bg-red-600 text-white px-3 py-2 rounded">Add to cart</button>
              </div>
            </div>
          ))}
        </div>
      </section>
    )}

    {view === 'sell' && (
      <section>
        <h2 className="text-2xl mb-4">Create a listing</h2>
        <form onSubmit={createListing} className="max-w-md">
          <label className="block mb-2">Title
            <input value={form.title} onChange={e => setForm({...form, title: e.target.value})} required className="w-full border p-2 rounded mt-1" />
          </label>
          <label className="block mb-2">Price (EGP)
            <input value={form.price} onChange={e => setForm({...form, price: e.target.value})} required type="number" step="0.01" className="w-full border p-2 rounded mt-1" />
          </label>
          <label className="block mb-2">Seller name
            <input value={form.seller} onChange={e => setForm({...form, seller: e.target.value})} className="w-full border p-2 rounded mt-1" />
          </label>
          <button type="submit" className="mt-3 bg-red-600 text-white px-4 py-2 rounded">Publish</button>
        </form>

        <div className="mt-8 text-sm text-gray-700">
          <p><strong>Platform fee:</strong> RedSalle takes 5% of each sale. For example, on a 100 EGP sale the platform fee is 5 EGP.</p>
          <p className="mt-2">In production the listing creation should be authenticated and saved to a backend service.</p>
        </div>
      </section>
    )}

    {view === 'cart' && (
      <section>
        <h2 className="text-2xl mb-4">Cart</h2>
        {cart.length === 0 ? (
          <p>Your cart is empty.</p>
        ) : (
          <div className="space-y-4">
            {cart.map((c, i) => (
              <div key={i} className="flex items-center justify-between border p-3 rounded">
                <div>
                  <div className="font-semibold">{c.title}</div>
                  <div className="text-sm text-gray-600">Seller: {c.seller}</div>
                </div>
                <div className="text-right">
                  <div className="font-bold">{c.price} EGP</div>
                  <button onClick={() => removeFromCart(i)} className="text-sm mt-2 underline">Remove</button>
                </div>
              </div>
            ))}

            <div className="border-t pt-4">
              <div className="flex justify-between mb-2"><span>Subtotal</span><span>{total()} EGP</span></div>
              <div className="flex justify-between mb-2"><span>RedSalle fee (5%)</span><span>{platformFee(total())} EGP</span></div>
              <div className="flex justify-between font-bold text-lg"><span>Total</span><span>{(total() + platformFee(total())).toFixed(2)} EGP</span></div>
              <div className="mt-4 flex gap-2">
                <button onClick={checkout} className="bg-red-600 text-white px-4 py-2 rounded">Checkout</button>
                <button onClick={() => setCart([])} className="px-4 py-2 border rounded">Clear</button>
              </div>
            </div>
          </div>
        )}
      </section>
    )}
  </main>

  <footer className="bg-gray-50 border-t p-4 text-center text-sm">
    <div className="max-w-5xl mx-auto">© {new Date().getFullYear()} RedSalle — marketplace platform (5% commission)</div>
  </footer>
</div>

); }

