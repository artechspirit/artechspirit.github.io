---
title: "How to Build a Clean, Secure CRUD API with MongoDB, Mongoose, and Express.js"
meta_title: "Create a Clean & Secure CRUD API with MongoDB, Mongoose, and Express.js"
description: "Learn how to build a scalable and secure CRUD API from scratch using MongoDB, Mongoose, and Express.js. This guide covers user authentication with JWT, login features with middleware, and a clean, maintainable folder structure that follows the DRY principle."
date: 2024-09-17
image: "/images/express-mongo-node.webp"
categories: ["JavaScript", "Backend Development"]
author: "Beta Priyoko"
tags: ["JavaScript", "Node.js", "Express.js", "MongoDB", "API"]
draft: false
---
#### **Introduction:**
Building a clean, scalable, and secure API is essential for any modern web application. In this tutorial, we’ll walk through the process of creating a fully functional **CRUD API** using **MongoDB**, **Mongoose**, and **Express.js**. 

Our API will allow users to perform **Create**, **Read**, **Update**, and **Delete** operations while ensuring that the application follows best practices in terms of security, maintainability, and code clarity. We’ll also implement **JWT-based authentication** for user login, protecting certain routes with middleware to ensure secure access to sensitive data.

By the end of this guide, you’ll have a solid understanding of how to:
- Set up a MongoDB connection using Mongoose.
- Create a user model with password hashing.
- Implement authentication and authorization using JWT.
- Structure your Express.js project for long-term maintainability following the **DRY** principle.

Whether you’re working on a small project or preparing to scale your application, this step-by-step guide will help you create a reliable and secure API service from scratch.

#### **Prerequisites:**

Before starting, make sure you have the following in place:

1. **Node.js and npm**:
   Ensure that **Node.js** and **npm** (Node Package Manager) are installed on your system. You can download them from [here](https://nodejs.org/en/). Check the installation with:
   ```bash
   node -v
   npm -v
   ```

2. **Basic knowledge of JavaScript and Node.js**:
   You should have a working understanding of **JavaScript** and the basics of **Node.js** to follow along with this guide.

3. **MongoDB Atlas account or local MongoDB server**:
   You’ll need access to a MongoDB database. You can either:
   - Set up a free **MongoDB Atlas** cloud account [here](https://www.mongodb.com/cloud/atlas) and get a connection string.
   - Or install **MongoDB** locally on your system.

4. **Postman or cURL for testing APIs**:
   You can use **Postman** or **cURL** to test the API endpoints. Postman is more user-friendly and can be downloaded [here](https://www.postman.com/downloads/).

5. **Understanding of RESTful APIs**:
   Familiarity with the basics of **RESTful APIs** will be helpful. We’ll be creating REST endpoints for **CRUD** operations (Create, Read, Update, Delete).

6. **Git (Optional)**:
   For version control and managing your project, **Git** is recommended. You can install it from [here](https://git-scm.com/), and initialize a Git repository in your project folder using:
   ```bash
   git init
   ```

Once these prerequisites are set, you'll be ready to dive into building your **CRUD API** with **MongoDB**, **Mongoose**, and **Express.js**.

#### **1. Setting Up the Project**

First, let’s start by setting up a new Node.js project. You will need to install some essential packages.

##### **1.1 Initialize Node.js Project**

```bash
mkdir express-mongo-api
cd express-mongo-api
npm init -y
```

This creates a `package.json` file with default settings.

##### **1.2 Install Required Dependencies**

Install all the required packages:
- **express**: Node.js web framework.
- **mongoose**: ODM for MongoDB.
- **dotenv**: To manage environment variables.
- **jsonwebtoken**: For generating and verifying JWT tokens.
- **bcryptjs**: For hashing passwords.
- **nodemon**: Development tool that automatically restarts the server when files change (optional but useful).

```bash
npm install express mongoose dotenv bcryptjs jsonwebtoken
npm install --save-dev nodemon
```

In the `package.json`, modify the `scripts` to use **nodemon**:

```json
"scripts": {
  "start": "node server.js",
  "dev": "nodemon server.js"
}
```

#### **2. Folder Structure**

To keep the project well-organized, let's create a clear folder structure:

```javascript
├── config
│   └── db.js              # MongoDB connection setup
├── controllers
│   └── userController.js   # User registration, login, and CRUD logic
├── middlewares
│   └── authMiddleware.js   # Middleware for authentication using JWT
├── models
│   └── User.js             # Mongoose model for User
├── routes
│   └── userRoutes.js       # Route definitions for user-related operations
├── utils
│   └── generateToken.js    # Utility function to generate JWT token
├── .env                    # Environment variables (JWT secret, DB connection, etc.)
├── app.js                  # Main app setup (Express, routes, middleware)
├── server.js               # Entry point of the application
└── package.json            # Node.js project configuration
```

#### **3. MongoDB Connection (`config/db.js`)**

Create a function to connect to MongoDB using Mongoose. This file will export a function that initializes the connection to your database.

```javascript
// config/db.js
const mongoose = require('mongoose');

const connectDB = async () => {
  try {
    const conn = await mongoose.connect(process.env.MONGO_URI, {
      useNewUrlParser: true,
      useUnifiedTopology: true,
    });
    console.log(`MongoDB Connected: ${conn.connection.host}`);
  } catch (error) {
    console.error(`Error: ${error.message}`);
    process.exit(1);
  }
};

module.exports = connectDB;
```

In this file, we use `mongoose.connect` to connect to MongoDB using the connection string provided in the `.env` file.

#### **4. Environment Variables**

Create a `.env` file to store environment-specific variables:

```bash
MONGO_URI=mongodb+srv://<username>:<password>@cluster.mongodb.net/<dbname>?retryWrites=true&w=majority
JWT_SECRET=your_jwt_secret
PORT=5000
```

Make sure to replace the MongoDB connection string with your own credentials.

#### **5. User Model (`models/User.js`)**

Define the user schema for MongoDB using Mongoose. The `User` model will have fields like **name**, **email**, and **password**. We'll also include logic to hash the password before saving the user to the database.

```javascript
// models/User.js
const mongoose = require('mongoose');
const bcrypt = require('bcryptjs');

const userSchema = new mongoose.Schema({
  name: {
    type: String,
    required: true,
  },
  email: {
    type: String,
    required: true,
    unique: true,
  },
  password: {
    type: String,
    required: true,
  },
}, { timestamps: true });

// Hash password before saving
userSchema.pre('save', async function (next) {
  if (!this.isModified('password')) {
    next();
  }
  const salt = await bcrypt.genSalt(10);
  this.password = await bcrypt.hash(this.password, salt);
});

// Compare entered password with hashed password
userSchema.methods.matchPassword = async function (enteredPassword) {
  return await bcrypt.compare(enteredPassword, this.password);
};

const User = mongoose.model('User', userSchema);

module.exports = User;
```

Here we define:
- `pre('save')`: A Mongoose middleware that runs before saving a user to hash the password.
- `matchPassword`: A method to compare the entered password with the hashed password.

#### **6. JWT Token Generation Utility (`utils/generateToken.js`)**

Create a utility function to generate a JWT token for the user:

```javascript
// utils/generateToken.js
const jwt = require('jsonwebtoken');

const generateToken = (id) => {
  return jwt.sign({ id }, process.env.JWT_SECRET, {
    expiresIn: '30d',
  });
};

module.exports = generateToken;
```

This function signs the user ID with a secret key and returns a token that expires in 30 days.

#### **7. Authentication Middleware (`middlewares/authMiddleware.js`)**

Middleware to protect routes by verifying the JWT token.

```javascript
// middlewares/authMiddleware.js
const jwt = require('jsonwebtoken');
const User = require('../models/User');

const protect = async (req, res, next) => {
  let token;

  if (req.headers.authorization && req.headers.authorization.startsWith('Bearer')) {
    try {
      token = req.headers.authorization.split(' ')[1]; // Get token from header
      const decoded = jwt.verify(token, process.env.JWT_SECRET); // Verify token
      req.user = await User.findById(decoded.id).select('-password'); // Get user from token
      next();
    } catch (error) {
      res.status(401).json({ message: 'Not authorized, token failed' });
    }
  } else {
    res.status(401).json({ message: 'Not authorized, no token' });
  }
};

module.exports = protect;
```

This middleware verifies the JWT token and checks if it’s valid. It attaches the user data to the request object for further use.

#### **8. User Controller (`controllers/userController.js`)**

The controller will contain logic for user registration, login, and retrieving user profiles.

```javascript
// controllers/userController.js
const User = require('../models/User');
const generateToken = require('../utils/generateToken');

// Register new user
const registerUser = async (req, res) => {
  const { name, email, password } = req.body;

  const userExists = await User.findOne({ email });

  if (userExists) {
    return res.status(400).json({ message: 'User already exists' });
  }

  const user = await User.create({ name, email, password });

  if (user) {
    res.status(201).json({
      _id: user._id,
      name: user.name,
      email: user.email,
      token: generateToken(user._id),
    });
  } else {
    res.status(400).json({ message: 'Invalid user data' });
  }
};

// Authenticate user & get token
const loginUser = async (req, res) => {
  const { email, password } = req.body;

  const user = await User.findOne({ email });

  if (user && (await user.matchPassword(password))) {
    res.json({
      _id: user._id,
      name: user.name,
      email: user.email,
      token: generateToken(user._id),
    });
  } else {
    res.status(401).json({ message: 'Invalid email or password' });
  }
};

// Get user profile
const getUserProfile = async (req, res) => {
  const user = req.user; // Comes from middleware
  res.json({
    _id: user._id,
    name: user.name,
    email: user.email,
  });
};

module.exports = { registerUser, loginUser, getUserProfile };
```

In this file, we have:
- `registerUser`: Handles user registration and returns a JWT token.
- `loginUser`: Authenticates the user and returns a token if credentials are valid.
- `getUserProfile`: Returns the logged-in user’s profile.

#### **9. User Routes (`routes/userRoutes.js`)**

Define API routes for user registration, login, and profile retrieval:

```javascript
// routes/userRoutes.js
const express = require('express');
const router = express.Router();
const { registerUser, loginUser, getUserProfile } = require('../controllers/userController');
const { protect } = require('../middlewares/authMiddleware');

// Public routes
router.post('/register', registerUser);
router.post('/login', loginUser);

// Protected route (requires JWT)
router.get('/profile', protect, getUserProfile);

module.exports = router;
```

#### **10. Main App Setup (`app.js`)**

In `app.js`, configure the main application by setting up **Express**, connecting to the database, and adding routes.

```javascript
// app.js
const express = require('express');
const connectDB = require('./config/db');
const dotenv = require('dotenv');
const userRoutes = require('./routes/userRoutes');

// Load environment variables
dotenv.config();

// Initialize MongoDB connection
connectDB();

const app = express();
app.use(express.json()); // Middleware to parse JSON

// Routes
app.use('/api/users', userRoutes);

// Error handling for undefined routes
app.use((req, res, next) => {
  res.status(404).json({ message: 'Route not found' });
});

module.exports = app;
```

#### **11. Server Setup (`server.js`)**

Finally, create the server setup to listen on the specified port:

```javascript
// server.js
const app = require('./app');

const PORT = process.env.PORT || 5000;

app.listen(PORT, () => {
  console.log(`Server running on port ${PORT}`);
});
```

#### **12. Testing the API**

Now that everything is set up, start the development server:

```javascript
npm run dev
```

You can test the API using **Postman** or **cURL**:
- **Register**: `POST /api/users/register` with `{ "name": "John Doe", "email": "john@example.com", "password": "123456" }`
- **Login**: `POST /api/users/login` with `{ "email": "john@example.com", "password": "123456" }`
- **Profile**: `GET /api/users/profile` with the JWT token in the Authorization header (`Bearer <token>`).

This setup provides a clean, modular approach to building a **CRUD API** with user authentication using **MongoDB**, **Mongoose**, and **Express.js**. It's scalable, maintainable, and uses **JWT** for secure user authentication.