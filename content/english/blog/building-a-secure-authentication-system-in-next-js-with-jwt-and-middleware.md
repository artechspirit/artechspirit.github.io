---
title: "Building a Secure Authentication System in Next.js with JWT and Middleware"
meta_title: "Building Secure Authentication in Next.js with JWT and Middleware"
description: "Learn how to build a secure authentication system in Next.js using JSON Web Tokens (JWT) and middleware. This guide covers the process step-by-step, ensuring a scalable, maintainable, and secure implementation."
date: 2024-09-19
image: "/images/next-js-jwt.webp"
categories: [Next.js, Security, Web Development]
author: "Beta Priyoko"
tags: [Next.js, JWT, Middleware, Authentication, Node.js, React, Security, Web Development]
draft: false
---
#### **Introduction:**
Securing your web application is essential, and building a robust authentication system is a crucial part of this. In this blog post, we'll walk through how to implement JWT-based authentication in a Next.js application. You'll learn how to secure API routes with middleware, handle login and registration, and maintain clean, maintainable code.

By the end of this tutorial, you’ll have a secure authentication system integrated into your Next.js app, making sure that only authenticated users can access certain resources.

#### **Why JWT for Authentication?**

JSON Web Tokens (JWT) offer a stateless, secure way to authenticate users. The server generates a token upon successful login, which the client stores and sends with subsequent requests. Here's why JWT is a great choice for authentication:

- **Stateless:** No session management needed on the server.
- **Compact:** The token is small and can easily be sent via HTTP headers.
- **Secure:** Tokens can be encrypted and verified using the server's secret.

#### **Setting Up Next.js for Authentication**

##### **Step 1: Initialize Your Next.js Project**

If you don’t have a Next.js project already set up, create one using the following command:

```bash
npx create-next-app@latest jwt-auth-app
cd jwt-auth-app
npm install
```

##### **Step 2: Install Required Dependencies**

You’ll need a few libraries for handling JWT and encryption:

```bash
npm install jsonwebtoken bcryptjs cookie
```

- **jsonwebtoken**: For creating and verifying JWTs.
- **bcryptjs**: For hashing passwords.
- **cookie**: For handling cookies.

#### **Step 3: Create User Authentication APIs**

In the Next.js **API routes** (`/pages/api/`), let’s start by creating the APIs for user registration and login.

##### **Register User API**

Create the file `/pages/api/auth/register.ts`:

```ts
import type { NextApiRequest, NextApiResponse } from 'next';
import bcrypt from 'bcryptjs';

interface User {
  email: string;
  password: string;
}

const users: User[] = [];  // For simplicity, using an array instead of a database.

export default async function handler(req: NextApiRequest, res: NextApiResponse) {
  if (req.method === 'POST') {
    const { email, password } = req.body;

    // Hash the password
    const hashedPassword = await bcrypt.hash(password, 10);

    // Save user with hashed password
    users.push({ email, password: hashedPassword });

    res.status(200).json({ message: 'User registered successfully' });
  } else {
    res.status(405).json({ message: 'Method Not Allowed' });
  }
}
```

##### **Login User API**

Create the file `/pages/api/auth/login.ts`:

```ts
import type { NextApiRequest, NextApiResponse } from 'next';
import bcrypt from 'bcryptjs';
import jwt from 'jsonwebtoken';
import { serialize } from 'cookie';

const SECRET_KEY = process.env.JWT_SECRET || 'your-secret-key';  // Use environment variable for production
const users = [];  // You can replace this with a database call

export default async function handler(req: NextApiRequest, res: NextApiResponse) {
  if (req.method === 'POST') {
    const { email, password } = req.body;
    const user = users.find(user => user.email === email);

    if (!user || !(await bcrypt.compare(password, user.password))) {
      return res.status(401).json({ message: 'Invalid credentials' });
    }

    // Create JWT
    const token = jwt.sign({ email: user.email }, SECRET_KEY, { expiresIn: '1h' });

    // Set the JWT in a cookie
    res.setHeader('Set-Cookie', serialize('authToken', token, {
      httpOnly: true,
      secure: process.env.NODE_ENV === 'production',
      sameSite: 'strict',
      path: '/',
      maxAge: 60 * 60,  // 1 hour
    }));

    res.status(200).json({ message: 'Login successful' });
  } else {
    res.status(405).json({ message: 'Method Not Allowed' });
  }
}
```

#### **Step 4: Securing API Routes with Middleware**

In this step, we'll create middleware to protect certain API routes. These routes will only be accessible to authenticated users.

##### **Create the Middleware**

Create a middleware file `lib/authMiddleware.ts`:

```ts
import { NextApiRequest, NextApiResponse } from 'next';
import jwt from 'jsonwebtoken';

const SECRET_KEY = process.env.JWT_SECRET || 'your-secret-key';

export const authMiddleware = (handler: any) => async (req: NextApiRequest, res: NextApiResponse) => {
  const token = req.cookies.authToken;

  if (!token) {
    return res.status(401).json({ message: 'Unauthorized' });
  }

  try {
    // Verify the token
    const decoded = jwt.verify(token, SECRET_KEY);
    req.user = decoded;
    return handler(req, res);
  } catch (err) {
    return res.status(401).json({ message: 'Invalid Token' });
  }
};
```

##### **Protect an API Route**

Let’s protect an example API route `/pages/api/protected.ts`:

```ts
import { NextApiRequest, NextApiResponse } from 'next';
import { authMiddleware } from '../../lib/authMiddleware';

const handler = (req: NextApiRequest, res: NextApiResponse) => {
  res.status(200).json({ message: `Welcome, ${req.user.email}!` });
};

export default authMiddleware(handler);
```

#### **Step 5: Implementing Logout Functionality**

Finally, you should allow users to log out by clearing the authentication cookie.

Create the file `/pages/api/auth/logout.ts`:

```ts
import { NextApiRequest, NextApiResponse } from 'next';
import { serialize } from 'cookie';

export default function handler(req: NextApiRequest, res: NextApiResponse) {
  res.setHeader('Set-Cookie', serialize('authToken', '', {
    httpOnly: true,
    secure: process.env.NODE_ENV === 'production',
    sameSite: 'strict',
    path: '/',
    maxAge: -1,  // Expire the cookie immediately
  }));

  res.status(200).json({ message: 'Logout successful' });
}
```

#### **Conclusion**

You’ve now built a simple yet secure JWT-based authentication system in Next.js. By leveraging middleware, you can protect your routes and ensure that only authenticated users can access them. Remember to use environment variables to store sensitive information like the JWT secret key.

This approach can easily be extended by integrating a real database, handling password resets, and adding additional layers of security like refresh tokens.

#### **Next Steps:**

1. Implement a proper database to store user information securely.
2. Add refresh tokens for more robust session management.
3. Secure sensitive API routes in your app, such as user profiles or payment gateways.
