# 🍱 Home Plate — Backend

Complete Node.js + Express + Supabase backend for the Home Plate home chef delivery app.

---

## 📁 File Structure

```
homeplate-backend/
├── src/
│   ├── index.js                  ← Server entry point
│   ├── api.js                    ← Frontend API connector (copy to your React app)
│   ├── db/
│   │   ├── supabase.js           ← Supabase client
│   │   └── schema.sql            ← Run this in Supabase SQL Editor first
│   ├── middleware/
│   │   └── auth.js               ← JWT verification + error handler
│   └── routes/
│       ├── auth.js               ← Register, sign in, sign out
│       ├── chefs.js              ← Chef profiles, menus, ratings
│       ├── orders.js             ← Place, track, update orders
│       ├── payments.js           ← Razorpay integration
│       ├── feedback.js           ← Ratings & reviews
│       ├── subscriptions.js      ← Meal plans
│       └── ai.js                 ← Claude AI analysis & recommendations
├── .env.example                  ← Copy to .env and fill in values
├── package.json
└── README.md
```

---

## 🚀 Quick Start (5 Steps)

### Step 1 — Set up Supabase

1. Go to [supabase.com](https://supabase.com) → New Project
2. Open **SQL Editor** → paste the contents of `src/db/schema.sql` → Run
3. Go to **Settings → API** → copy:
   - `Project URL` → `SUPABASE_URL`
   - `service_role` key → `SUPABASE_SERVICE_KEY`
   - `anon` key → `SUPABASE_ANON_KEY`
4. Enable **Realtime** on the `orders` table:
   - Go to **Database → Replication** → enable `orders` table

### Step 2 — Configure environment

```bash
cp .env.example .env
# Fill in all values in .env
```

### Step 3 — Install & run

```bash
npm install
npm run dev        # development (with hot reload)
npm start          # production
```

### Step 4 — Connect your frontend

Copy `src/api.js` to your React project's `src/` folder.

Add to your `.env` in the React project:
```
VITE_API_URL=http://localhost:3000/api
```

Then use in your components:
```js
import { auth, chefs, orders, payments, ai } from './api.js'

// Register
const { user, token } = await auth.register({
  firstName: 'Priya', lastName: 'Sharma',
  email: 'priya@email.com', password: 'secret123',
  phone: '9876543210', address: '12 MG Road, Delhi',
  role: 'user'
})

// Get chefs
const allChefs = await chefs.getAll('dal')

// Place order
const order = await orders.place({
  chefId: 'chef-uuid',
  items: [{ id: 'item-uuid', name: 'Dal Makhani', price: 130, qty: 2, emoji: '🫕' }],
  totalAmount: 265,
  deliveryTime: '13:00'
})

// Get AI analysis
const { analysis } = await ai.userAnalysis({
  rating: 5, note: 'Amazing food!',
  specialty: 'North Indian', orderCount: 3
})
```

### Step 5 — Deploy to Railway

```bash
npm install -g @railway/cli
railway login
railway init
railway up
# Add environment variables in Railway dashboard
```

Your API will be live at: `https://homeplate-backend.up.railway.app`

---

## 📡 API Reference

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/api/auth/register` | Create account |
| POST | `/api/auth/signin` | Sign in |
| GET  | `/api/auth/me` | Get current user |
| GET  | `/api/chefs` | List all chefs (with menus) |
| GET  | `/api/chefs?preference=dal` | Filter chefs by food preference |
| GET  | `/api/chefs/:id` | Get single chef |
| POST | `/api/chefs/menu/item` | Chef adds a dish |
| DELETE | `/api/chefs/menu/item/:id` | Chef deletes a dish |
| PATCH | `/api/chefs/menu/item/:id/special` | Toggle week's special |
| POST | `/api/orders` | Place an order |
| GET  | `/api/orders/:id` | Track order status |
| GET  | `/api/orders/user/history` | User order history |
| GET  | `/api/orders/chef/active` | Chef's live orders |
| PATCH | `/api/orders/:id/status` | Chef updates order stage |
| POST | `/api/payments/create` | Create Razorpay order |
| POST | `/api/payments/verify` | Verify payment |
| POST | `/api/feedback` | Submit review |
| POST | `/api/subscriptions` | Subscribe to plan |
| POST | `/api/ai/user-analysis` | AI weekly analysis for user |
| POST | `/api/ai/chef-analysis` | AI weekly analysis for chef |
| POST | `/api/ai/recommend` | AI meal recommendations |

---

## 🔒 Authentication

All protected routes require:
```
Authorization: Bearer <token>
```

Token is returned from `/api/auth/register` and `/api/auth/signin`.

---

## ⚡ Real-time Order Tracking

Install Supabase client in your frontend:
```bash
npm install @supabase/supabase-js
```

Subscribe to live order updates:
```js
import { createClient } from '@supabase/supabase-js'
const supabase = createClient(SUPABASE_URL, SUPABASE_ANON_KEY)

const STATUS_TO_STAGE = {
  placed: 0, accepted: 1, cooking: 2,
  out_for_delivery: 3, delivered: 4
}

useEffect(() => {
  const channel = supabase.channel('order-tracking')
    .on('postgres_changes', {
      event: 'UPDATE',
      schema: 'public',
      table: 'orders',
      filter: `id=eq.${orderId}`
    }, (payload) => {
      setStage(STATUS_TO_STAGE[payload.new.status])
    })
    .subscribe()

  return () => supabase.removeChannel(channel)
}, [orderId])
```

When the chef taps "Mark as Cooking" → your backend updates `orders.status` → Supabase Realtime pushes it → user's tracking screen updates instantly. No polling needed!
