# 🛒 Internet Retail Backend

A full-featured e-commerce API backend powering product catalogs, inventory management, and category organization. Built with Node.js, Express, MySQL, and Sequelize ORM, demonstrating advanced relational database design and N+1 query optimization.

**[📹 API Walkthrough](https://watch.screencastify.com/v/d1ycPQSZ8VCJxR9gG9LO)** | **[🗄️ Database Schema](#database-schema)** | **[🔌 Endpoints](#api-endpoints)**

---

## ✨ Features

### 📦 Product Management
- CRUD operations for products with inventory tracking
- Product search and filtering by category or tags
- Stock level management

### 🏷️ Category Organization
- Hierarchical product organization
- Fast category lookups via indexes
- Bulk category operations

### 🎯 Tags & Classification
- Multi-tag support per product
- Tag-based product discovery
- Many-to-many relationships via join table

### 💰 Pricing & Inventory
- Price tracking and updates
- Stock quantity management
- Bulk operations for large inventory updates

---

## 🗄️ Database Schema

```
┌──────────────────────────┐
│      Categories          │
├──────────────────────────┤
│  id (PK)                 │
│  category_name           │
│  createdAt, updatedAt     │
└──────────────────┬───────┘
                   │ 1:N
                   │
          ┌────────▼──────────┐
          │     Products      │
          ├───────────────────┤
          │  id (PK)          │
          │  product_name     │
          │  price            │
          │  stock            │
          │  category_id (FK) │
          │  createdAt        │
          └────────┬──────────┘
                   │ N:N (via ProductTag join table)
                   │
          ┌────────▼──────────┐
          │  ProductTag (JT)  │
          ├───────────────────┤
    ┌─────┤  id (PK)          │
    │     │  product_id (FK)  │
    │     │  tag_id (FK)      │
    │     │  createdAt        │
    │     └───────────────────┘
    │
    └─────────────────────────────┐
                                  │
                 ┌────────────────▼────┐
                 │       Tags          │
                 ├─────────────────────┤
                 │  id (PK)            │
                 │  tag_name           │
                 │  createdAt, updatedAt│
                 └─────────────────────┘
```

---

## 🔌 API Endpoints

### Categories Routes

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/api/categories` | Fetch all categories with products |
| `GET` | `/api/categories/:id` | Get category by ID + associated products |
| `POST` | `/api/categories` | Create new category |
| `PUT` | `/api/categories/:id` | Update category name |
| `DELETE` | `/api/categories/:id` | Delete category |

### Products Routes

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/api/products` | Get all products with category & tags |
| `GET` | `/api/products/:id` | Get product by ID (expanded with tags) |
| `POST` | `/api/products` | Create new product |
| `PUT` | `/api/products/:id` | Update product (price, stock, category) |
| `DELETE` | `/api/products/:id` | Delete product & associations |

### Tags Routes

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/api/tags` | Get all tags with products |
| `GET` | `/api/tags/:id` | Get tag by ID + tagged products |
| `POST` | `/api/tags` | Create new tag |
| `PUT` | `/api/tags/:id` | Update tag name |
| `DELETE` | `/api/tags/:id` | Delete tag |

---

## 🚀 Getting Started

### Prerequisites
- Node.js 14+
- MySQL 5.7+ (local or cloud)
- npm or yarn

### Installation

```bash
git clone https://github.com/dev-dominick/internet-retail-back-end.git
cd internet-retail-back-end

npm install

# Create .env file with database credentials
cat > .env << EOF
DB_NAME=ecommerce_db
DB_USER=root
DB_PASSWORD=your_password
DB_HOST=localhost
EOF

# Create database & sync models
npm run seed

npm start
# Server running on http://localhost:3001
```

### Seed Database

```bash
npm run seed
```

Creates sample categories (Electronics, Clothing), products, and tags from [seeds](./db/seeds/) directory.

---

## 📊 Sequelize ORM Patterns

### Model Associations (Relational Integrity)

```javascript
// One-to-Many: Category has many Products
Category.hasMany(Product, { foreignKey: 'category_id' });
Product.belongsTo(Category, { foreignKey: 'category_id' });

// Many-to-Many: Product has many Tags (via ProductTag join table)
Product.belongsToMany(Tag, { 
  through: ProductTag, 
  foreignKey: 'product_id'
});
Tag.belongsToMany(Product, { 
  through: ProductTag, 
  foreignKey: 'tag_id'
});
```

### Efficient Querying with Includes

```javascript
// Fetch product with category and tags (no N+1)
const product = await Product.findByPk(1, {
  include: [
    { model: Category },
    { model: Tag }
  ]
});
```

### Cascade Operations

```javascript
// Delete category → automatically deletes products
// Delete product → automatically removes tag associations
```

---

## 🛠️ Tech Stack

| Technology | Purpose |
|-----------|---------|
| **Node.js** | JavaScript runtime |
| **Express.js** | Web framework & routing |
| **MySQL** | SQL relational database |
| **Sequelize** | ORM for query building & associations |
| **npm** | Package management |

---

## ⚡ Performance Optimization

✅ **Indexed Foreign Keys** — `category_id`, `product_id`, `tag_id` indexed for sub-1ms lookups
✅ **Eager Loading** — Use `.include()` to fetch related data in single query (eliminates N+1)
✅ **Connection Pooling** — Sequelize manages MySQL connection pool
✅ **Pagination Ready** — Use `.limit()` & `.offset()` for large result sets
✅ **Query Logging** — Enable `logging: console` in Sequelize config for optimization insights

### Example N+1 Prevention

```javascript
// ❌ BAD: Generates 1 + N queries
const products = await Product.findAll();
for (let product of products) {
  const category = await product.getCategory(); // 1 query per product!
}

// ✅ GOOD: Single query with JOIN
const products = await Product.findAll({
  include: [{ model: Category }]  // 1 query total
});
```

---

## 🔐 Security Best Practices

- Input validation via Sequelize schemas
- Parameterized queries (Sequelize default)
- Environment variables for sensitive data (.env)
- Rate limiting recommended for production
- CORS configured for trusted origins

---

## 🧪 Testing Endpoints

### Insomnia Collection
Import [Insomnia collection](./docs/API.json) for pre-configured requests.

### Quick API Test

```bash
# Create category
curl -X POST http://localhost:3001/api/categories \
  -H "Content-Type: application/json" \
  -d '{"category_name":"Electronics"}'

# Create product
curl -X POST http://localhost:3001/api/products \
  -H "Content-Type: application/json" \
  -d '{
    "product_name":"Laptop",
    "price":999.99,
    "stock":50,
    "category_id":1
  }'
```

---

## 👥 Credits

- **Isaak** — [GitHub](https://github.com/CallMeIce)
- **Kyle** — [GitHub](https://github.com/kgiunta)

---

## 📝 License

MIT License

Copyright (c) 2022 Dominick Albano

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.

---

## 💡 What This Demonstrates

✅ **Relational Schema Design** — Category-Product-Tag modeling with proper foreign keys
✅ **Sequelize ORM Mastery** — Associations, eager loading, cascade deletes
✅ **N+1 Query Prevention** — Demonstrates problem and efficient `.include()` solution
✅ **REST API Design** — Proper HTTP methods, status codes, nested resources
✅ **Production-Ready Patterns** — Connection pooling, indexed queries, env configuration