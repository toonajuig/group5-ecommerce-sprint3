# Backend Tasks — group5-ecommerce-sprint3

## ลำดับการทำงาน

```
#1 Project Setup
 └─ #2 Models
     ├─ #3 Auth
     │   ├─ #6 Orders
     │   └─ #7 Users
     │       └─ #8 Admin
     ├─ #4 Products ──┘
     └─ #5 Categories
```

---

## #1 Project Setup & Folder Structure

- [ ] `npm init -y`
- [ ] `npm install express mongoose dotenv bcryptjs jsonwebtoken cors`
- [ ] `npm install -D nodemon`
- [ ] สร้างโฟลเดอร์และไฟล์ตาม structure ด้านล่าง
- [ ] ตั้งค่า `.env`
- [ ] เขียน `app.js` + `server.js` เบื้องต้น
- [ ] เชื่อมต่อ MongoDB ใน `config/db.js`

### โครงสร้างไฟล์

```
group5-ecommerce-sprint3/
├── src/
│   ├── config/
│   │   └── db.js                  # เชื่อมต่อ MongoDB
│   ├── middleware/
│   │   ├── auth.js                # verifyToken, isAdmin
│   │   └── errorHandler.js        # global error handler
│   ├── models/
│   │   ├── User.js
│   │   ├── Product.js
│   │   ├── Category.js
│   │   └── Order.js
│   ├── controllers/
│   │   ├── authController.js
│   │   ├── productController.js
│   │   ├── categoryController.js
│   │   ├── orderController.js
│   │   ├── userController.js
│   │   └── adminController.js
│   ├── routes/
│   │   ├── auth.js
│   │   ├── products.js
│   │   ├── categories.js
│   │   ├── orders.js
│   │   ├── users.js
│   │   └── admin.js
│   └── app.js                     # ติดตั้ง middleware, mount routes
├── .env
├── .gitignore
├── package.json
└── server.js                      # entry point (listen port)
```

### .env

```
PORT=5000
MONGO_URI=mongodb://localhost:27017/group5-ecommerce
JWT_SECRET=your_secret_here
```

---

## #2 Models

> ต้องทำหลัง #1

- [ ] `User.js`
- [ ] `Category.js`
- [ ] `Product.js`
- [ ] `Order.js`

### User.js

```js
{
  name: String,
  email: String,          // unique
  password: String,       // bcrypt hashed
  role: enum ['user', 'admin'],
  addresses: [{
    label: String,        // 'บ้าน', 'ที่ทำงาน'
    name: String,
    address: String,
    isDefault: Boolean,
  }]
}
```

### Category.js

```js
{ name: String, icon: String }
```

### Product.js

```js
{
  name: String,
  price: Number,
  kcal: Number,
  protein: String,        // '40g'
  categoryId: ObjectId,   // ref: 'Category'
  tag: String,            // 'Best Seller', 'New', ...
  desc: String,
  imageUrl: String,
  isActive: Boolean,      // default: true
}
```

### Order.js

```js
{
  userId: ObjectId,       // ref: 'User'
  items: [{
    productId: ObjectId,  // ref: 'Product'
    name: String,
    qty: Number,
    price: Number,
  }],
  subtotal: Number,
  shippingFee: Number,    // default: 30
  total: Number,
  address: { label, name, address },
  paymentMethod: String,  // 'promptpay', 'card', ...
  status: enum ['Pending', 'Shipped', 'Delivered', 'Cancelled'],
}
```

---

## #3 Auth

> ต้องทำหลัง #2

ไฟล์: `src/routes/auth.js`, `src/controllers/authController.js`, `src/middleware/auth.js`

- [ ] `POST /auth/register` — สมัครสมาชิก
- [ ] `POST /auth/login` — เข้าสู่ระบบ คืน JWT
- [ ] `middleware/auth.js` — `verifyToken`, `isAdmin`

```
register: hash password ด้วย bcrypt → บันทึก User → คืน token
login:    หา user by email → compare password → คืน token
verifyToken: ตรวจ Authorization header → jwt.verify → req.user
isAdmin:     ตรวจ req.user.role === 'admin'
```

---

## #4 Products

> ต้องทำหลัง #2

ไฟล์: `src/routes/products.js`, `src/controllers/productController.js`

- [ ] `GET    /products` — ดึงทั้งหมด (query: `?categoryId=`)
- [ ] `GET    /products/:id` — ดึงรายการเดียว
- [ ] `POST   /products` — เพิ่มสินค้า `[verifyToken, isAdmin]`
- [ ] `PUT    /products/:id` — แก้ไขสินค้า `[verifyToken, isAdmin]`
- [ ] `DELETE /products/:id` — ลบสินค้า `[verifyToken, isAdmin]`
- [ ] seed สินค้า 20 รายการ + 5 categories จาก sprint2

---

## #5 Categories

> ต้องทำหลัง #2

ไฟล์: `src/routes/categories.js`, `src/controllers/categoryController.js`

- [ ] `GET  /categories` — ดึงหมวดหมู่ทั้งหมด
- [ ] `POST /categories` — เพิ่มหมวดหมู่ `[verifyToken, isAdmin]`
- [ ] seed 5 หมวดหมู่: อกไก่ปั่น / สลัด & Bowl / Smoothie / Meal Plan / Supplement

---

## #6 Orders

> ต้องทำหลัง #3 และ #4

ไฟล์: `src/routes/orders.js`, `src/controllers/orderController.js`

- [ ] `POST   /orders` — สร้างออเดอร์ใหม่ `[verifyToken]`
- [ ] `GET    /orders` — ดูออเดอร์ของตัวเอง `[verifyToken]`
- [ ] `GET    /orders/:id` — ดูรายละเอียด + tracking `[verifyToken]`
- [ ] `PATCH  /orders/:id/status` — อัพเดทสถานะ `[verifyToken, isAdmin]`

```
POST body: { items: [{ productId, qty }], addressId, paymentMethod }
controller lookup ราคาจาก DB เอง (ไม่ trust ราคาจาก client)
Status flow: Pending → Shipped → Delivered
```

---

## #7 Users

> ต้องทำหลัง #3

ไฟล์: `src/routes/users.js`, `src/controllers/userController.js`

- [ ] `GET    /users/me` — ดูโปรไฟล์ตัวเอง `[verifyToken]`
- [ ] `PUT    /users/me` — แก้ชื่อ/email `[verifyToken]`
- [ ] `POST   /users/me/addresses` — เพิ่มที่อยู่ `[verifyToken]`
- [ ] `PUT    /users/me/addresses/:id` — แก้ไขที่อยู่ `[verifyToken]`
- [ ] `DELETE /users/me/addresses/:id` — ลบที่อยู่ `[verifyToken]`

```
ถ้าเพิ่ม address ใหม่ isDefault: true → set address อื่นทั้งหมดเป็น false ก่อน
```

---

## #8 Admin

> ต้องทำหลัง #6 และ #7

ไฟล์: `src/routes/admin.js`, `src/controllers/adminController.js`

ทุก route ต้องผ่าน `[verifyToken, isAdmin]`

- [ ] `GET /admin/stats` — `{ totalRevenue, totalOrders, totalProducts, recentOrders[] }`
- [ ] `GET /admin/orders` — ออเดอร์ทั้งหมดทุก user
- [ ] `GET /admin/customers` — list users พร้อม orderCount, totalSpent

```
totalRevenue  → sum total ของ orders ที่ status = 'Delivered'
totalOrders   → นับ orders ทั้งหมด
totalProducts → นับ products ที่ isActive = true
recentOrders  → 5 รายการล่าสุด
```
