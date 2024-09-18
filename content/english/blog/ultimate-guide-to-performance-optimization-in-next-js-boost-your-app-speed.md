---
title: "Ultimate Guide to Performance Optimization in Next.js: Boost Your App Speed"
meta_title: "Blog | Next.js Performance Optimization: Boost Speed with SSG, SSR, SWR, and More"
description: "Learn how to build a scalable and secure CRUD API from scratch using MongoDB, Mongoose, and Express.js. This guide covers user authentication with JWT, login features with middleware, and a clean, maintainable folder structure that follows the DRY principle."
date: 2024-09-18
image: "/images/next-js-optimization.png"
categories: ["Next.js",
  "Web Development",
  "Performance Optimization",
  "Frontend Development",
  "React.js"]
author: "Beta Priyoko"
tags: [ "Next.js performance",
  "Static Site Generation (SSG)",
  "Incremental Static Regeneration (ISR)",
  "Server-Side Rendering (SSR)",
  "Dynamic imports",
  "Code splitting",
  "Image optimization",
  "SWR",
  "Tailwind CSS",
  "PurgeCSS",
  "Tree shaking",
  "JavaScript optimization",
  "CSS optimization",
  "Preloading assets",
  "Font optimization",
  "Google Lighthouse"]
draft: false
---
**Introduction:**

Building a high-performance web application is crucial for a positive user experience and better search engine rankings. With **Next.js**, a powerful React framework, performance optimization becomes simpler through built-in features like **Static Site Generation (SSG)**, **Server-Side Rendering (SSR)**, **Image Optimization**, and more. This comprehensive guide will walk you through various ways to optimize your Next.js app for speed, efficiency, and scalability.

### 1. **Leverage Static Site Generation (SSG) and Incremental Static Regeneration (ISR)**

#### Static Site Generation (SSG):
SSG is one of the most powerful ways to pre-render your Next.js pages at build time. By generating static HTML pages, your app delivers blazing-fast performance. SSG is perfect for pages that don’t require frequent updates and can be built once and served to all users.

##### Example of SSG with `getStaticProps`:
```typescript
export async function getStaticProps() {
  const res = await fetch('https://api.example.com/products');
  const products = await res.json();

  return {
    props: {
      products,
    },
  };
}
```
In this example, the `getStaticProps` function fetches data at build time, resulting in pre-rendered HTML that significantly improves load times.

##### Incremental Static Regeneration (ISR):
For pages that need to be updated frequently, ISR allows you to regenerate specific pages in the background at runtime without rebuilding the entire app. You can configure how often to regenerate by using the `revalidate` option.

###### Example of ISR:
```typescript
export async function getStaticProps() {
  const res = await fetch('https://api.example.com/products');
  const products = await res.json();

  return {
    props: {
      products,
    },
    revalidate: 60, // Revalidate every 60 seconds
  };
}
```

With ISR, you get the best of both worlds: fast static pages and dynamic updates.

#### 2. **Utilize Server-Side Rendering (SSR) for Dynamic Content**

Sometimes, you’ll need to fetch data at each request rather than at build time. This is where **Server-Side Rendering (SSR)** comes in handy. It’s useful for dynamic pages that depend on user-specific data, such as user dashboards or pages that change frequently.

###### Example of SSR with `getServerSideProps`:
```typescript
export async function getServerSideProps() {
  const res = await fetch('https://api.example.com/user');
  const user = await res.json();

  return {
    props: {
      user,
    },
  };
}
```

While SSR is powerful, it can add overhead, as the server needs to generate HTML for each request. Use SSR wisely for dynamic content and avoid overusing it on static pages to maximize performance.

#### 3. **Optimize JavaScript with Dynamic Imports and Code Splitting**

Next.js allows you to split your JavaScript bundle into smaller chunks using **dynamic imports**, improving load times for large modules. Code splitting reduces the initial download size by loading only the code needed for a specific page or component.

###### Example of Dynamic Import:
```typescript
import dynamic from 'next/dynamic';

const HeavyComponent = dynamic(() => import('../components/HeavyComponent'), {
  loading: () => <p>Loading...</p>,
});

export default function Home() {
  return (
    <div>
      <HeavyComponent />
    </div>
  );
}
```

This approach delays the loading of `HeavyComponent` until it’s actually needed, improving initial page load performance.

#### 4. **Image Optimization Using the Next.js `<Image />` Component**

Images are often the largest resource on a page and can drastically slow down performance if not handled properly. Next.js’s built-in **Image Optimization** feature provides responsive images that are lazy-loaded by default, automatically optimized, and served in modern formats like WebP.

##### Example:
```typescript
import Image from 'next/image';

export default function Home() {
  return (
    <div>
      <Image
        src="/images/product.jpg"
        width={800}
        height={600}
        alt="Product Image"
      />
    </div>
  );
}
```

The `next/image` component resizes, compresses, and delivers the image in the best format for the user’s device, improving page speed and user experience.

#### 5. **Use SWR for Client-Side Data Fetching**

When fetching data on the client side, the **SWR (Stale-While-Revalidate)** library helps ensure fast and reliable data retrieval. SWR caches data and revalidates it in the background, so users get instant responses without waiting for fresh data.

###### Example using SWR:
```typescript
import useSWR from 'swr';

const fetcher = (url: string) => fetch(url).then(res => res.json());

export default function ProductList() {
  const { data, error } = useSWR('/api/products', fetcher);

  if (error) return <div>Failed to load products</div>;
  if (!data) return <div>Loading...</div>;

  return (
    <ul>
      {data.map(product => (
        <li key={product.id}>{product.name}</li>
      ))}
    </ul>
  );
}
```

With SWR, your users see cached data instantly while the app fetches updated information in the background. This drastically improves the user experience, especially in apps with frequent data changes.

#### 6. **Use Tailwind CSS and PurgeCSS for Lightweight CSS**

To reduce CSS file size, especially when using a framework like **Tailwind CSS**, it’s essential to remove unused styles in production. Tailwind’s **PurgeCSS** integration helps you achieve this by scanning your files and purging any unnecessary styles.

###### Setup in `tailwind.config.js`:
```javascript
module.exports = {
  purge: ['./pages/**/*.{js,ts,jsx,tsx}', './components/**/*.{js,ts,jsx,tsx}'],
  // Other configurations...
};
```

By purging unused CSS classes, you ensure that only the necessary styles are included in your final build, reducing CSS bloat and improving load times.

#### 7. **Reduce JavaScript and CSS Payload with Tree Shaking and Minification**

Next.js automatically performs **tree shaking** and minification, which removes unused code and compresses the remaining JavaScript and CSS. However, you can further optimize your app by:
- **Avoiding large libraries**: Import only the modules you need (e.g., using `lodash-es` instead of `lodash`).
- **Splitting components**: Dynamically load components with `React.lazy` and `Suspense` when they’re not required during initial load.

##### Example:
```typescript
import { debounce } from 'lodash-es';
```

By importing only specific methods instead of the entire library, you can significantly reduce your JavaScript bundle size.

#### 8. **Preload Critical Assets for Faster Loading**

To speed up asset loading, **preload** critical resources like fonts, images, and scripts. Preloading allows the browser to download important assets sooner, improving performance.

##### Example of Preloading Fonts:
```html
<link rel="preload" href="/fonts/custom.woff2" as="font" type="font/woff2" crossorigin="anonymous" />
```

Preloading ensures that fonts and other critical assets are loaded earlier in the page load process, reducing the risk of layout shifts and improving the user experience.


#### 9. **Optimize Fonts with `next/font`**

Next.js offers **next/font**, a built-in tool for efficiently loading Google and custom fonts. It prevents **FOIT** (Flash of Invisible Text) and **FOUT** (Flash of Unstyled Text), ensuring smooth typography rendering.

##### Example:
```typescript
import { Roboto } from 'next/font/google';

const roboto = Roboto({ subsets: ['latin'] });

export default function Home() {
  return <div className={roboto.className}>Welcome to the site!</div>;
}
```

Using `next/font`, your fonts are loaded in a way that prioritizes performance and user experience.

#### 10. **Monitor and Improve Performance with Lighthouse**

Finally, regularly monitor your app’s performance using tools like **Google Lighthouse** or **Vercel Analytics**. These tools provide metrics on **Largest Contentful Paint (LCP)**, **Time to Interactive (TTI)**, and **First Contentful Paint (FCP)**, helping you identify bottlenecks.

Run Lighthouse audits periodically to ensure your app continues to perform well as it grows in complexity.

**Conclusion:**

Optimizing the performance of your Next.js application is an ongoing process that requires attention to detail and a combination of strategies. By leveraging features like **SSG**, **ISR**, **SWR**, **dynamic imports**, **image optimization**, and more, you can ensure your app is fast, efficient, and provides a superior user experience.