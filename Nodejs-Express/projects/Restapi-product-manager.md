Perfect! Let’s create a **REST API for Product Management** using **Node.js + Express** with a clean structure, CRUD operations, validation, and good practices. I’ll provide a **real-world example**, folder structure, and full code outline for a **portfolio-ready project**.

---

# 🔵 Project: Product Management REST API

**Objective:** Build an API to **create, read, update, delete, and list products** with proper validation and error handling.

---

## 1️⃣ Folder Structure

```
product-api/
├── node_modules/
├── src/
│   ├── controllers/
│   │   └── productController.js
│   ├── routes/
│   │   └── productRoutes.js
│   ├── models/
│   │   └── productModel.js
│   ├── middlewares/
│   │   ├── errorHandler.js
│   │   └── validateRequest.js
│   ├── utils/
│   │   └── db.js
│   └── app.js
├── package.json
└── .env
```

---

## 2️⃣ Dependencies

```bash
npm init -y
npm install express mongoose dotenv body-parser cors joi
```

* **express** → web framework
* **mongoose** → MongoDB ODM
* **dotenv** → environment variables
* **body-parser** → parse JSON
* **cors** → cross-origin requests
* **joi** → validation

---

## 3️⃣ Database Connection (`src/utils/db.js`)

```javascript
const mongoose = require("mongoose");
require("dotenv").config();

const connectDB = async () => {
    try {
        await mongoose.connect(process.env.MONGO_URI);
        console.log("MongoDB connected");
    } catch (err) {
        console.error(err);
        process.exit(1);
    }
};

module.exports = connectDB;
```

---

## 4️⃣ Product Model (`src/models/productModel.js`)

```javascript
const mongoose = require("mongoose");

const productSchema = new mongoose.Schema({
    name: { type: String, required: true, trim: true },
    description: { type: String },
    price: { type: Number, required: true, min: 0 },
    category: { type: String, required: true },
    inStock: { type: Boolean, default: true },
}, { timestamps: true });

module.exports = mongoose.model("Product", productSchema);
```

---

## 5️⃣ Validation Middleware (`src/middlewares/validateRequest.js`)

```javascript
const Joi = require("joi");

const productSchema = Joi.object({
    name: Joi.string().required(),
    description: Joi.string().allow(""),
    price: Joi.number().min(0).required(),
    category: Joi.string().required(),
    inStock: Joi.boolean()
});

const validateProduct = (req, res, next) => {
    const { error } = productSchema.validate(req.body);
    if (error) {
        return res.status(400).json({ message: error.details[0].message });
    }
    next();
};

module.exports = { validateProduct };
```

---

## 6️⃣ Product Controller (`src/controllers/productController.js`)

```javascript
const Product = require("../models/productModel");

// Create product
const createProduct = async (req, res) => {
    try {
        const product = await Product.create(req.body);
        res.status(201).json(product);
    } catch (err) {
        res.status(500).json({ message: err.message });
    }
};

// Get all products
const getProducts = async (req, res) => {
    try {
        const products = await Product.find();
        res.json(products);
    } catch (err) {
        res.status(500).json({ message: err.message });
    }
};

// Get product by ID
const getProductById = async (req, res) => {
    try {
        const product = await Product.findById(req.params.id);
        if (!product) return res.status(404).json({ message: "Product not found" });
        res.json(product);
    } catch (err) {
        res.status(500).json({ message: err.message });
    }
};

// Update product
const updateProduct = async (req, res) => {
    try {
        const product = await Product.findByIdAndUpdate(req.params.id, req.body, { new: true });
        if (!product) return res.status(404).json({ message: "Product not found" });
        res.json(product);
    } catch (err) {
        res.status(500).json({ message: err.message });
    }
};

// Delete product
const deleteProduct = async (req, res) => {
    try {
        const product = await Product.findByIdAndDelete(req.params.id);
        if (!product) return res.status(404).json({ message: "Product not found" });
        res.json({ message: "Product deleted successfully" });
    } catch (err) {
        res.status(500).json({ message: err.message });
    }
};

module.exports = { createProduct, getProducts, getProductById, updateProduct, deleteProduct };
```

---

## 7️⃣ Product Routes (`src/routes/productRoutes.js`)

```javascript
const express = require("express");
const router = express.Router();
const { createProduct, getProducts, getProductById, updateProduct, deleteProduct } = require("../controllers/productController");
const { validateProduct } = require("../middlewares/validateRequest");

router.post("/", validateProduct, createProduct);
router.get("/", getProducts);
router.get("/:id", getProductById);
router.put("/:id", validateProduct, updateProduct);
router.delete("/:id", deleteProduct);

module.exports = router;
```

---

## 8️⃣ Main App (`src/app.js`)

```javascript
const express = require("express");
const bodyParser = require("body-parser");
const cors = require("cors");
const connectDB = require("./utils/db");
const productRoutes = require("./routes/productRoutes");
require("dotenv").config();

const app = express();
connectDB();

app.use(cors());
app.use(bodyParser.json());

// Routes
app.use("/api/products", productRoutes);

// Global Error Handler
app.use((err, req, res, next) => {
    res.status(500).json({ message: err.message });
});

const PORT = process.env.PORT || 5000;
app.listen(PORT, () => console.log(`Server running on port ${PORT}`));
```

---

## 9️⃣ Test Endpoints

| Endpoint            | Method | Description       |
| ------------------- | ------ | ----------------- |
| `/api/products`     | POST   | Create a product  |
| `/api/products`     | GET    | Get all products  |
| `/api/products/:id` | GET    | Get product by ID |
| `/api/products/:id` | PUT    | Update product    |
| `/api/products/:id` | DELETE | Delete product    |

---

# ⚡ Best Practices Used

* **Folder separation** → controllers, routes, models, middleware
* **Validation** → Joi schema
* **Error handling** → centralized middleware
* **MongoDB + Mongoose** → scalable DB
* **REST principles** → clear endpoint structure

---

# 🧾 Summary

This **Product Management REST API** provides a **portfolio-ready backend** with:

* CRUD operations for products
* Validation and error handling
* Modular structure
* Easy to extend for **auth, file uploads, and background tasks**

---

If you want, I can extend this project to a **full SaaS dashboard backend** including:

* **JWT auth / role-based access**
* **File uploads for product images**
* **Event-driven notifications**
* **Background tasks for analytics / processing**

This would make it a **complete, real-world SaaS project**.

Do you want me to do that next?
