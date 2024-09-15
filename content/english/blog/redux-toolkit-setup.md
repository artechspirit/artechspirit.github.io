---
title: "The Most Effective and Simple Way to Implement Redux Toolkit Query in Next.js with CRUD Operations"
meta_title: "The Most Effective and Simple Way to Implement Redux Toolkit Query in Next.js with CRUD Operations"
description: "Redux Toolkit Query (RTK Query) is an advanced data fetching and caching tool that integrates seamlessly with Redux, making it a great choice for managing server state. Combined with Next.js’s App Router and Tailwind CSS, you can create a robust, scalable, and beautifully styled application with minimal effort."
date: 2024-09-15
image: "/images/blog-post-2.webp"
categories: ["JavaScript", "Frontend Development"]
author: "Beta Priyoko"
tags: ["html", "css", "JavaScript", "Redux Toolkit Query", "React Hook", "Next.js"]
draft: false
---
When building modern web applications, efficiency and simplicity are crucial. Redux Toolkit Query (RTK Query) simplifies data fetching and caching, making it an excellent choice for managing server state. Coupled with Next.js’s App Router and Tailwind CSS, you can create a scalable, well-styled application with minimal effort.

In this blog post, we’ll demonstrate how to implement RTK Query in a Next.js project using the App Router, perform CRUD operations with the FakeStore API, and style the application with Tailwind CSS. This guide is designed to be practical, modular, and easy to follow, ensuring clean and maintainable code.

## Background

Next.js is a powerful React framework that supports server-side rendering, static site generation, and more. Redux Toolkit Query is part of Redux Toolkit, designed to simplify data fetching and caching. Tailwind CSS is a utility-first CSS framework that makes it easy to create responsive and customizable designs.

The FakeStore API provides a straightforward way to simulate interactions with a store, making it ideal for our CRUD operations.

## Step-by-Step Guide

### 1. **Setting Up the Project**

First, set up a Next.js project with JavaScript and install the necessary dependencies:

```bash
npx create-next-app my-app
cd my-app
npm install @reduxjs/toolkit react-redux daisyui
npm install tailwindcss@latest postcss@latest autoprefixer@latest
npx tailwindcss init -p
```

Configure Tailwind CSS by editing `tailwind.config.js`:

```js
module.exports = {
  content: [
    './app/**/*.{js,jsx}',
    './components/**/*.{js,jsx}',
    './pages/**/*.{js,jsx}',
  ],
  theme: {
    extend: {},
  },
  plugins: [require('daisyui')],
};
```

Add Tailwind CSS to `styles/globals.css`:

```css
@tailwind base;
@tailwind components;
@tailwind utilities;
```

### 2. **Setting Up Redux Toolkit Query**

Create an API slice using RTK Query. This will serve as our interface to the FakeStore API.

Create `app/store/api.js`:

```js
import { createApi, fetchBaseQuery } from '@reduxjs/toolkit/query/react';

export const fakeStoreApi = createApi({
  reducerPath: 'fakeStoreApi',
  baseQuery: fetchBaseQuery({ baseUrl: 'https://fakestoreapi.com' }),
  endpoints: (builder) => ({
    getProducts: builder.query({
      query: () => '/products',
    }),
    getProductById: builder.query({
      query: (id) => `/products/${id}`,
    }),
    addProduct: builder.mutation({
      query: (product) => ({
        url: '/products',
        method: 'POST',
        body: product,
      }),
    }),
    updateProduct: builder.mutation({
      query: ({ id, ...product }) => ({
        url: `/products/${id}`,
        method: 'PUT',
        body: product,
      }),
    }),
    deleteProduct: builder.mutation({
      query: (id) => ({
        url: `/products/${id}`,
        method: 'DELETE',
      }),
    }),
  }),
});

export const {
  useGetProductsQuery,
  useGetProductByIdQuery,
  useAddProductMutation,
  useUpdateProductMutation,
  useDeleteProductMutation,
} = fakeStoreApi;
```

### 3. **Setting Up Redux Store**

Create the Redux store and configure it in `app/store/index.js`:

```js
import { configureStore } from '@reduxjs/toolkit';
import { fakeStoreApi } from './api';

export const store = configureStore({
  reducer: {
    [fakeStoreApi.reducerPath]: fakeStoreApi.reducer,
  },
  middleware: (getDefaultMiddleware) =>
    getDefaultMiddleware().concat(fakeStoreApi.middleware),
});

export const { Provider } = require('react-redux');
export const { useSelector, useDispatch } = require('react-redux');
```

Provide the store in `app/layout.js`:

```jsx
import { Provider } from 'react-redux';
import { store } from './store';

export default function RootLayout({ children }) {
  return (
    <html lang="en">
      <head />
      <body>
        <Provider store={store}>{children}</Provider>
      </body>
    </html>
  );
}
```

### 4. **Implementing CRUD Pages**

Create pages for listing, viewing, adding, and editing products.

- **List Products (`app/products/page.js`)**:

```jsx
import { useGetProductsQuery } from '../store/api';
import Link from 'next/link';

export default function ProductsPage() {
  const { data, error, isLoading } = useGetProductsQuery();

  if (isLoading) return <p>Loading...</p>;
  if (error) return <p>Error loading products</p>;

  return (
    <div className="container mx-auto p-4">
      <h1 className="text-2xl font-bold mb-4">Products</h1>
      <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4 gap-6">
        {data?.map((product) => (
          <div key={product.id} className="border p-4 rounded">
            <h2 className="text-lg font-semibold">{product.title}</h2>
            <p>${product.price}</p>
            <Link href={`/products/${product.id}`}>
              <button className="mt-2 bg-blue-500 text-white py-1 px-2 rounded">View</button>
            </Link>
          </div>
        ))}
      </div>
    </div>
  );
}
```

- **Product Details (`app/products/[id]/page.js`)**:

```jsx
import { useGetProductByIdQuery } from '../../store/api';
import { useRouter } from 'next/router';

export default function ProductDetailPage({ params }) {
  const { id } = params;
  const { data, error, isLoading } = useGetProductByIdQuery(id);

  if (isLoading) return <p>Loading...</p>;
  if (error) return <p>Error loading product</p>;

  return (
    <div className="container mx-auto p-4">
      <h1 className="text-2xl font-bold">{data?.title}</h1>
      <p className="mt-2">{data?.description}</p>
      <p className="mt-2 text-lg font-semibold">${data?.price}</p>
    </div>
  );
}
```

- **Add Product (`app/products/add/page.js`)**:

```jsx
import { useState } from 'react';
import { useAddProductMutation } from '../../store/api';

export default function AddProductPage() {
  const [addProduct] = useAddProductMutation();
  const [formData, setFormData] = useState({
    title: '',
    price: '',
    description: '',
    image: '',
    category: '',
  });

  const handleChange = (e) => {
    setFormData({ ...formData, [e.target.name]: e.target.value });
  };

  const handleSubmit = async (e) => {
    e.preventDefault();
    await addProduct(formData);
  };

  return (
    <form onSubmit={handleSubmit} className="container mx-auto p-4">
      <h1 className="text-2xl font-bold mb-4">Add New Product</h1>
      <input
        name="title"
        value={formData.title}
        onChange={handleChange}
        placeholder="Title"
        className="border p-2 w-full mb-4"
      />
      {/* Repeat for other fields */}
      <button type="submit" className="bg-green-500 text-white py-2 px-4 rounded">
        Submit
      </button>
    </form>
  );
}
```

- **Edit Product (`app/products/[id]/edit/page.js`)**:

```jsx
import { useState, useEffect } from 'react';
import { useRouter } from 'next/router';
import { useGetProductByIdQuery, useUpdateProductMutation } from '../../store/api';

export default function EditProductPage({ params }) {
  const { id } = params;
  const { data, error, isLoading } = useGetProductByIdQuery(id);
  const [updateProduct] = useUpdateProductMutation();
  const [formData, setFormData] = useState({
    title: '',
    price: '',
    description: '',
    image: '',
    category: '',
  });

  useEffect(() => {
    if (data) {
      setFormData(data);
    }
  }, [data]);

  const handleChange = (e) => {
    setFormData({ ...formData, [e.target.name]: e.target.value });
  };

  const handleSubmit = async (e) => {
    e.preventDefault();
    await updateProduct({ id, ...formData });
  };

  if (isLoading) return <p>Loading...</p>;
  if (error) return <p>Error loading product</p>;

  return (
    <form onSubmit={handleSubmit} className="container mx-auto p-4">
      <h1 className="text-2xl font-bold mb-4">Edit Product</h1>
      <input
        name="title"
        value={formData.title}
        onChange={handleChange}
        placeholder

="Title"
        className="border p-2 w-full mb-4"
      />
      {/* Repeat for other fields */}
      <button type="submit" className="bg-blue-500 text-white py-2 px-4 rounded">
        Update
      </button>
    </form>
  );
}
```

- **Delete Product (Button in `ProductDetailPage`)**:

Add a delete button in the product detail page:

```jsx
import { useDeleteProductMutation } from '../../store/api';

export default function ProductDetailPage({ params }) {
  const { id } = params;
  const { data, error, isLoading } = useGetProductByIdQuery(id);
  const [deleteProduct] = useDeleteProductMutation();

  const handleDelete = async () => {
    await deleteProduct(id);
    // Redirect or show a success message
  };

  if (isLoading) return <p>Loading...</p>;
  if (error) return <p>Error loading product</p>;

  return (
    <div className="container mx-auto p-4">
      <h1 className="text-2xl font-bold">{data?.title}</h1>
      <p className="mt-2">{data?.description}</p>
      <p className="mt-2 text-lg font-semibold">${data?.price}</p>
      <button
        onClick={handleDelete}
        className="mt-4 bg-red-500 text-white py-2 px-4 rounded"
      >
        Delete
      </button>
    </div>
  );
}
```

### 5. **Final Touches and Best Practices**

- **Reusability:** Create reusable components for forms and buttons to avoid code duplication.
- **Error Handling:** Implement robust error handling to improve user experience and provide feedback.
- **Optimistic Updates:** Consider using optimistic updates for a smoother user experience when performing mutations.

## Conclusion

Integrating Redux Toolkit Query with Next.js using the App Router and combining it with Tailwind CSS provides a powerful and elegant solution for managing server state and building user interfaces. By following this guide, you’ll be able to create a highly efficient and maintainable CRUD application with minimal boilerplate code.

Feel free to experiment with the provided code and customize it further to suit your needs. Happy coding!