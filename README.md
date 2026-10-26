# ShopEase — Full-Stack E-Commerce Application

A complete e-commerce platform with:
- **Frontend:** React 18 + Vite, React Router DOM, Bootstrap 5, custom CSS
- **Backend:** Spring Boot 3 (Java 17), Spring Data JPA, Spring Security (JWT), Spring Web, Validation, MySQL, Lombok, DevTools
- **Payments:** Razorpay integration (test mode)
- **Admin Panel:** Product & order management dashboard
- **API Testing:** Postman collection included

---

## 📁 Project Structure

```
ecommerce-project/
├── backend/                    # Spring Boot app — open in Eclipse as "Existing Maven Project"
│   ├── pom.xml
│   └── src/main/java/com/ecommerce/
│       ├── entity/             # Product, User, Order, OrderItem, Role
│       ├── repository/         # Spring Data JPA repositories
│       ├── dto/                 # Request/response DTOs
│       ├── security/           # JWT filter, util, UserDetailsService
│       ├── service/             # Business logic + Razorpay service
│       ├── controller/          # REST controllers
│       ├── config/               # SecurityConfig, DataSeeder
│       └── exception/            # Global exception handler
│   └── src/main/resources/
│       ├── application.properties
│       └── data.sql             # Seeds 12 products automatically
│
├── frontend/                   # React + Vite app — open in VS Code
│   └── src/
│       ├── pages/               # Home, Shop, ProductDetail, Cart, Checkout, Login, Register, MyOrders
│       ├── pages/admin/         # Admin Dashboard, Product & Order management
│       ├── components/          # Navbar, Footer, ProductCard, ProtectedRoute, Loader
│       ├── context/             # AuthContext, CartContext
│       ├── services/            # Axios API layer
│       └── styles/custom.css    # Custom theming on top of Bootstrap
│
├── postman/
│   └── Ecommerce.postman_collection.json
│
└── README.md
```

---

## 1. Backend Setup (Eclipse)

### Prerequisites
- JDK 17+
- MySQL 8+ running locally
- Eclipse IDE (with Maven support, e.g. Eclipse IDE for Enterprise Java and Web Developers) + Lombok installed (run `lombok.jar` installer against your Eclipse, or use the [Lombok plugin](https://projectlombok.org/setup/eclipse))

### Steps
1. **Import the project**
   - Eclipse → `File` → `Import` → `Maven` → `Existing Maven Projects` → select the `backend` folder.
2. **Create the database** (or let the app auto-create it — see `application.properties`, `createDatabaseIfNotExist=true`):
   ```sql
   CREATE DATABASE ecommerce_db;
   ```
3. **Check `src/main/resources/application.properties`** and update MySQL username/password if different from `root` / `root`:
   ```properties
   spring.datasource.url=jdbc:mysql://localhost:3306/ecommerce_db?createDatabaseIfNotExist=true&useSSL=false&serverTimezone=UTC
   spring.datasource.username=root
   spring.datasource.password=root
   ```
4. **Razorpay keys** are already configured (test mode):
   ```properties
   razorpay.key.id=rzp_test_T5sNXSIWAFWlYw
   razorpay.key.secret=MUVOLzE4RjfSorGORvi2wP4W
   ```
   > ⚠️ These are **test keys**. For any real deployment, move keys to environment variables and never commit real production secrets to source control.
5. **Run** `EcommerceApplication.java` as a Java Application (or `Run As → Spring Boot App`).
6. On first run:
   - `data.sql` seeds **12 products** automatically.
   - `DataSeeder.java` creates a default admin account:
     - Email: `admin@shopease.com`
     - Password: `Admin@123`
7. Backend runs at **http://localhost:8080**, all endpoints under `/api/**`.

---

## 2. Frontend Setup (VS Code)

### Prerequisites
- Node.js 18+ and npm

### Steps
1. Open the `frontend` folder in VS Code.
2. Install dependencies:
   ```bash
   cd frontend
   npm install
   ```
3. Check `.env` (already configured):
   ```
   VITE_API_BASE_URL=http://localhost:8080/api
   VITE_RAZORPAY_KEY_ID=rzp_test_T5sNXSIWAFWlYw
   ```
4. Run the dev server:
   ```bash
   npm run dev
   ```
5. App runs at **http://localhost:5173** (Vite dev server proxies `/api` calls to the backend — see `vite.config.js`).

---

## 3. Using the App

- **Shop as guest:** browse `/shop`, view product details, add to cart.
- **Register / Login:** create an account at `/register`, then `/login`.
- **Checkout:** cart → checkout → fill shipping address → pay with Razorpay (test mode).
  - Test card: `4111 1111 1111 1111`, any future expiry, any CVV, any OTP (Razorpay test mode auto-succeeds).
- **Admin:** log in with `admin@shopease.com` / `Admin@123` → a new **Admin** link appears in the navbar → manage products and orders at `/admin`.

### Payment Flow (how it's wired)
1. Frontend calls `POST /api/orders` to create the order in the database (server computes the authoritative total from live product prices).
2. Frontend calls `POST /api/payment/create-order/{orderId}` — backend asks Razorpay to create a matching order and returns `razorpayOrderId`, `amount`, `currency`, `keyId`.
3. Frontend opens Razorpay Checkout (`checkout.js`, loaded in `index.html`) with those details.
4. On success, Razorpay returns `razorpay_payment_id`, `razorpay_order_id`, `razorpay_signature` to the browser.
5. Frontend calls `POST /api/payment/verify` with those values + the local `orderId`. Backend verifies the signature using the Razorpay secret and marks the order `PAID` / `CONFIRMED`.

---

## 4. API Testing with Postman

1. Import `postman/Ecommerce.postman_collection.json` into Postman.
2. Collection variables: `baseUrl` (defaults to `http://localhost:8080/api`), `token`, `adminToken`, `productId`, `orderId`.
3. Suggested flow:
   1. **Auth → Login Admin** (or Register + Login a normal user) → copy the `token` from the response into the `token`/`adminToken` variable.
   2. **Products (Public)** → browse/search products.
   3. **Orders → Create Order** → note the returned `id`, set it as `orderId`.
   4. **Payment → Create Razorpay Order for an Order**, then **Verify Payment** (in real use, the `razorpay_*` values come from the Checkout.js success callback in the browser, not typed by hand).
   5. **Admin - Products / Admin - Orders** → manage catalog and fulfillment (requires `adminToken`).

---

## 5. Key Backend Endpoints

| Method | Endpoint | Auth | Description |
|---|---|---|---|
| POST | `/api/auth/register` | Public | Register a new user |
| POST | `/api/auth/login` | Public | Login, returns JWT |
| GET | `/api/products` | Public | List all products |
| GET | `/api/products/{id}` | Public | Get one product |
| GET | `/api/products/search?keyword=` | Public | Search products |
| POST | `/api/orders` | User | Create an order |
| GET | `/api/orders/my` | User | Get logged-in user's orders |
| POST | `/api/payment/create-order/{orderId}` | User | Create Razorpay order |
| POST | `/api/payment/verify` | User | Verify Razorpay payment signature |
| POST | `/api/admin/products` | Admin | Create product |
| PUT | `/api/admin/products/{id}` | Admin | Update product |
| DELETE | `/api/admin/products/{id}` | Admin | Delete product |
| GET | `/api/orders/admin/all` | Admin | Get all orders |
| PATCH | `/api/orders/admin/{id}/status?status=` | Admin | Update order status |

---

## 6. Tech Stack Summary

**Backend dependencies (pom.xml):** Spring Web, Spring Data JPA, Spring Security, Validation, MySQL Driver, Lombok, Spring Boot DevTools, Razorpay Java SDK, JJWT (JWT tokens).

**Frontend dependencies (package.json):** React, React Router DOM, Axios, Bootstrap 5, Bootstrap Icons, React Toastify.

---

## 7. Notes & Best Practices

- The included Razorpay keys are **test keys** — safe for development, but rotate/replace them and use environment variables before any production deployment.
- JWT secret in `application.properties` is a placeholder — replace it with a securely generated secret before deploying anywhere public.
- CORS is pre-configured for `http://localhost:5173` (Vite) and `http://localhost:3000`.
- `spring.jpa.hibernate.ddl-auto=update` will auto-create/update tables on startup — fine for development; use migrations (e.g. Flyway/Liquibase) for production.
