# Frontend Migration Analysis: Sprint 2 → Sprint 3

> วันที่: 2026-05-28  
> วิเคราะห์จาก: `group5-ecommerce-sprint2` (frontend ที่ทำไว้แล้ว) เทียบกับ Frontend Agent Instructions

---

## ภาพรวมของสองโปรเจก

| | Sprint 2 | Sprint 3 |
|---|---|---|
| **ประเภท** | Frontend (React/Vite) | Backend (Express/MongoDB) + **Frontend ใหม่** |
| **สถานะ** | มี UI ครบ แต่ใช้ mock data | Backend skeleton เปล่า, Frontend ยังไม่มี |
| **Auth** | Hardcode `useState` ใน NavBar | JWT HttpOnly Cookie |
| **ข้อมูลสินค้า** | `src/data/products.js` | จะดึงจาก API จริง |
| **Cart** | Context + mock initial data | Context + ไม่มี initial data |

---

## โครงสร้าง Folder ที่ต้องการ (Instructions) vs Sprint 2

```
Instructions (เป้าหมาย)         Sprint 2 (มีอยู่แล้ว)
──────────────────────────────────────────────────────
src/
├── assets/                     src/assets/              ✅ ตรงกัน
├── components/                 src/components/ui/        ✅ มีครบมาก
├── contexts/                   src/context/              ⚠️  ชื่อต่างกัน (s)
├── hooks/                      src/context/useCart.js    ⚠️  อยู่ผิดที่
├── pages/                      src/components/screens/   ⚠️  ชื่อและ path ต่างกัน
├── services/                   ❌ ไม่มีเลย
├── utils/                      ❌ ไม่มีเลย
├── App.jsx                     src/App.jsx               ✅ มี
└── main.jsx                    src/main.jsx              ✅ มี
```

---

## ส่วนที่ Copy มาได้เลย (แทบไม่ต้องเปลี่ยน)

### 1. UI Components (`src/components/ui/`)
Sprint 2 มี shadcn-style components ครบมาก:
- `button.jsx`, `input.jsx`, `card.jsx`, `table.jsx`, `badge.jsx`
- `dialog.jsx`, `select.jsx`, `textarea.jsx`, `skeleton.jsx`
- `pagination.jsx`, `tabs.jsx`, `dropdown-menu.jsx` ฯลฯ

**Action:** Copy โฟลเดอร์ `src/components/ui/` ทั้งหมดมาตรงๆ

### 2. หน้า Home (`DHomeScreen.jsx`)
เนื้อหาส่วนใหญ่เป็น static UI ไม่ได้ดึง API:
- Hero section, Feature cards, Article section, CTA

**Action:** Copy มาเป็น `src/pages/HomePage.jsx` แล้วเปลี่ยนชื่อไฟล์ ไม่ต้องแก้ logic

### 3. หน้า Login/Register UI Shell
UI ที่ออกแบบไว้สวยงามแล้ว Form validation เบื้องต้นก็มีแล้ว

**Action:** Copy มาเป็น `src/pages/LoginPage.jsx`, `src/pages/RegisterPage.jsx`  
ต้องเปลี่ยนแค่ `handleSubmit` ให้เรียก `authService.login()` แทน `alert()`

### 4. Layout Components
- `NavBar.jsx` — โครงสร้างดี แต่ต้องแก้ส่วน auth
- `DFooter.jsx` — Static ทั้งหมด copy ได้เลย

### 5. Tailwind + Vite Config
Sprint 2 ใช้ Tailwind CSS v4 + Vite เหมือนกันทุกอย่าง

**Action:** Copy `vite.config.js`, `index.css`, `tailwind.config` มาตรงๆ

### 6. Package Dependencies
Sprint 2 มี dependencies ที่ต้องใช้ต่อครบแล้ว:
```
react-router-dom, lucide-react, tailwindcss, react-hook-form, zod ฯลฯ
```

---

## ส่วนที่ Copy มาได้แต่ต้องปรับ (Adaptation Required)

### 1. `CartContext.jsx` → `src/contexts/CartContext.jsx`

**ปัญหาที่ต้องแก้:**
```js
// Sprint 2: เริ่มต้นด้วย mock data
import { cartItems as initialCart } from '@/data/cart'
const [items, setItems] = useState(initialCart)  // ❌ ต้องเปลี่ยน

// Sprint 3: เริ่มต้น empty
const [items, setItems] = useState([])  // ✅
```

**Action:** ลบ import `@/data/cart` ออก เปลี่ยน initial state เป็น `[]`  
Logic ทั้งหมด (`addItem`, `removeItem`, `incrementQty`, `decrementQty`) ใช้ได้เลย ไม่ต้องแก้

### 2. `NavBar.jsx` → ต้องต่อกับ `AuthContext`

**ปัญหาที่ต้องแก้:**
```js
// Sprint 2: auth state hardcode อยู่ใน NavBar ❌
const [user, setUser] = useState({ name: 'สมตูน คูณร้อย', image: null })

// Sprint 3: ดึงจาก AuthContext ✅
const { user, logout } = useAuth()
```

**Action:** ลบ `useState` ที่ hardcode user ออก แล้ว import `useAuth` hook แทน

### 3. `DCatalogScreen.jsx` → `src/pages/CatalogPage.jsx`

**ปัญหาที่ต้องแก้:**
```js
// Sprint 2: ดึงจาก mock data ❌
import { categories, products } from '@/data/products'

// Sprint 3: ดึงจาก API ✅
const [products, setProducts] = useState([])
const [categories, setCategories] = useState([])
useEffect(() => {
  productService.getAllProducts(queryString).then(setProducts)
  categoryService.getAllCategories().then(setCategories)
}, [queryString])
```

Sprint 2 ใช้ `useSearchParams` อยู่แล้ว ✅ แต่ต้องเพิ่ม `loading` / `error` state  
Category filter ปัจจุบันเก็บใน local state → Sprint 3 ให้เพิ่มไว้ใน URL params ด้วย

### 4. `DProductDetailScreen.jsx` → `src/pages/ProductDetailPage.jsx`

**ปัญหาที่ต้องแก้:** ปัจจุบัน find product จาก mock array  
→ เปลี่ยนเป็น `productService.getProductById(id)` พร้อม loading/error state

### 5. `DCartCheckoutScreen.jsx` → `src/pages/CartPage.jsx`

**ปัญหาที่ต้องแก้:**
- UI ใช้ได้ดี แต่ต้องเพิ่ม form ที่อยู่จัดส่ง (address)
- `handleSubmit` ต้องเรียก `orderService.createOrder()` จริง

### 6. Admin Pages → `src/pages/admin/`

Sprint 2 มี `AdminDashboard`, `AdminProducts`, `AdminOrders`, `AdminCustomers` ครบ  
ปัจจุบันใช้ mock data ทั้งหมด → ต้องเปลี่ยนให้ดึงจาก `adminService`

**AdminProducts ที่ต้องเพิ่ม:**
- ปุ่ม Add / Edit / Delete (ปัจจุบัน read-only)
- Form popup ใช้ `dialog.jsx` ที่มีอยู่แล้ว

### 7. Routing ใน `main.jsx`

**ปัญหาที่ต้องแก้:**
```js
// Sprint 2: ไม่มี AdminRoute protection ❌
{ path: '/admin', element: <AdminLayout /> }

// Sprint 3: ต้องป้องกันด้วย AdminRoute ✅
{ path: '/admin', element: <AdminRoute><AdminLayout /></AdminRoute> }
```

---

## ส่วนที่ต้องสร้างใหม่ทั้งหมด (Create from Scratch)

### 1. `src/services/api.js` ⭐ สำคัญมาก
Fetch wrapper กลาง ต้องมี `credentials: 'include'` ทุก request

```js
// ตัวอย่าง structure
const BASE_URL = import.meta.env.VITE_API_BASE_URL

async function request(endpoint, options = {}) {
  const res = await fetch(`${BASE_URL}${endpoint}`, {
    headers: { 'Content-Type': 'application/json', ...options.headers },
    credentials: 'include',   // ส่ง cookie ทุกครั้ง
    ...options,
  })
  if (!res.ok) throw await res.json()
  return res.json()
}
```

### 2. `src/services/productService.js`
```js
getAllProducts(queryString = '')  // รองรับ ?page=1&category=xxx
getProductById(id)
createProduct(data)    // admin
updateProduct(id, data) // admin
deleteProduct(id)      // admin
```

### 3. `src/services/authService.js`
```js
login(email, password)
register(name, email, password)
logout()
getProfile()   // GET /users/profile — ใช้ใน AuthContext checkAuth
```

### 4. `src/services/orderService.js`
```js
createOrder(items, addressId, paymentMethod)
getMyOrders()
getOrderById(id)
```

### 5. `src/services/categoryService.js`
```js
getAllCategories()
```

### 6. `src/contexts/AuthContext.jsx` ⭐ สำคัญมาก
```js
// ต้องทำ:
// - checkAuthStatus() → เรียก GET /users/profile ตอน load app
// - login(email, password) → เรียก authService.login()
// - logout() → เรียก authService.logout()
// - state: user, isLoading, isAuthenticated
```

### 7. `src/hooks/useAuth.js`
```js
export function useAuth() {
  const ctx = useContext(AuthContext)
  if (!ctx) throw new Error('useAuth must be inside AuthProvider')
  return ctx
}
```

### 8. `src/components/AdminRoute.jsx`
```js
// ถ้า isLoading → โชว์ spinner
// ถ้า user?.role !== 'admin' → redirect ไป /login
// ถ้าผ่าน → โชว์ children
```

### 9. `src/utils/formatCurrency.js`
```js
export function formatCurrency(amount) {
  return `฿${amount.toLocaleString()}`
}
```

### 10. `.env`
```
# Local Development
VITE_API_BASE_URL=http://localhost:5000/api

# PRODUCTION: อย่า upload ไฟล์นี้ขึ้น GitHub
# ให้ไปตั้งค่า VITE_API_BASE_URL ใน Vercel/Netlify Dashboard แทน
```

---

## สรุปงานทั้งหมดเป็น Checklist

### Phase 1: โครงสร้าง + Config
- [ ] สร้าง Vite project ใหม่ใน `frontend/` หรือ root ของ sprint3
- [ ] Copy `components/ui/` จาก sprint2
- [ ] Copy `vite.config.js`, CSS config จาก sprint2
- [ ] สร้าง `.env` + เพิ่ม `.gitignore` ให้ครอบคลุม

### Phase 2: Services Layer (ทำก่อน เพราะทุก page ต้องใช้)
- [ ] สร้าง `services/api.js` — fetch wrapper
- [ ] สร้าง `services/authService.js`
- [ ] สร้าง `services/productService.js`
- [ ] สร้าง `services/categoryService.js`
- [ ] สร้าง `services/orderService.js`

### Phase 3: Contexts + Hooks
- [ ] สร้าง `contexts/AuthContext.jsx` (ใหม่ทั้งหมด)
- [ ] Migrate `context/CartContext.jsx` → `contexts/CartContext.jsx` (แก้ initial state)
- [ ] สร้าง `hooks/useAuth.js`
- [ ] ย้าย `context/useCart.js` → `hooks/useCart.js`

### Phase 4: Pages (migrate + wire API)
- [ ] `HomePage.jsx` — copy, แทบไม่ต้องแก้
- [ ] `CatalogPage.jsx` — แก้ให้ดึง API + loading/error state
- [ ] `ProductDetailPage.jsx` — แก้ให้ดึง API
- [ ] `LoginPage.jsx` — เชื่อม authService.login()
- [ ] `RegisterPage.jsx` — เชื่อม authService.register()
- [ ] `CartPage.jsx` — เชื่อม orderService.createOrder()
- [ ] Admin pages — เชื่อม admin API ทั้งหมด

### Phase 5: Components + Guards
- [ ] แก้ `NavBar.jsx` ให้ใช้ `useAuth` แทน hardcode state
- [ ] สร้าง `AdminRoute.jsx` component

---

## ไฟล์ที่ทิ้งได้จาก Sprint 2

| ไฟล์/โฟลเดอร์ | เหตุผล |
|---|---|
| `src/data/` ทั้งโฟลเดอร์ | ข้อมูลจะมาจาก API จริง |
| `src/context/useCart.js` | ย้ายไป `hooks/useCart.js` |
| `src/tokens/theme.js` | ไม่จำเป็น |
| `src/hooks/use-mobile.js` | ไม่จำเป็นใน scope นี้ |
| หน้า `DEtc1-4Screen.jsx` | เป็น placeholder ไม่เกี่ยวกับ e-commerce |

---

## ข้อสังเกตสำคัญ

1. **Sprint 2 ใช้ `react-router-dom` v7** — Instructions บอก v6 แต่ v7 backward compatible กับ API เดิม ใช้ได้ปกติ ไม่ต้อง downgrade

2. **Sprint 2 ใช้ `components/screens/desktop/`** แทน `pages/` — ตาม Instructions ให้ rename เป็น `pages/` เพื่อให้ตรง architecture

3. **Sprint 2 มี `src/lib/utils.js`** (cn function สำหรับ shadcn) — ต้อง copy มาด้วยเสมอ เพราะ `components/ui/` ทุกไฟล์ต้องใช้

4. **Backend API routes** ที่ sprint3 ออกแบบไว้ใช้ `/api/users/profile` แต่ AuthContext Instructions บอก `/api/users/profile` — ตรงกัน ✅  
   แต่ authService ต้องดู TASKS.md: login อยู่ที่ `/auth/login`, profile อยู่ที่ `/users/me` → ต้องปรับ URL ตามนี้
