

React JS এ রাউটিং করার জন্য আমাদের `react-router-dom` ইন্সটল করতে হতো, `Routes`, `Route` এসব সেটআপ করতে হতো। অনেক ঝামেলার কাজ।

Next.js এ রাউটিং পানির মতো সহজ। একে বলা হয় **File-system based Routing**। অর্থাৎ, আপনি ফোল্ডার বানাবেন, রাউট অটোমেটিক তৈরি হয়ে যাবে।

চলুন ধাপে ধাপে দেখি।

---

**Step 1: বেসিক রাউটিং (Basic Routing)**

আপনার `src/app` ফোল্ডারের ভেতর যে কোনো ফোল্ডার বানালে সেটাই একটা URL হয়ে যাবে। কিন্তু শর্ত হলো, ওই ফোল্ডারের ভেতর `page.tsx` ফাইল থাকতে হবে।

ধরুন আপনি `/about` এবং `/contact` পেজ বানাতে চান।

স্ট্রাকচার হবে এরকম:
1. `src/app/about/page.tsx`  -> এটা ব্রাউজারে হবে `localhost:3000/about`
2. `src/app/contact/page.tsx` -> এটা ব্রাউজারে হবে `localhost:3000/contact`

কোড দেখতে একদম সাধারণ রিয়েক্ট কম্পোনেন্টের মতোই:
```tsx
// src/app/about/page.tsx
export default function AboutPage() {
  return <h1>আমাদের সম্পর্কে জানুন</h1>;
}
```

---

**Step 2: নেস্টেড রাউটিং (Nested Routing)**

ধরুন আপনি ব্লগের ভেতর পোস্ট দেখাতে চান (`/blog/first-post`)।
তাহলে ফোল্ডার বানাবেন এভাবে: `src/app/blog/first-post/page.tsx`।

---

**Step 3: ডাইনামিক রাউটিং (Dynamic Routing)** - *খুব গুরুত্বপূর্ণ*

ধরুন আপনার ১০০টা প্রোডাক্ট আছে। আপনি তো আর ১০০টা ফোল্ডার বানাবেন না। আপনি চাইবেন `/products/1`, `/products/2` এভাবে ইউআরএল কাজ করুক।

এর জন্য ফোল্ডারের নাম দিতে হবে থার্ড ব্র্যাকেট দিয়ে `[]`। যেমন: `[id]` বা `[slug]`।

স্ট্রাকচার: `src/app/products/[id]/page.tsx`

Next.js 15 এ `params` হ্যান্ডেল করা একটু ট্রিকি (আগের ভার্সনের চেয়ে ভিন্ন), কারণ এখন `params` একটা `Promise`। তাই এটাকে `await` করতে হয়।

উদাহরণ:
```tsx
// src/app/products/[id]/page.tsx
// ইউজার যদি /products/500 এ হিট করে, তাহলে id হবে 500

export default async function ProductDetails({ params }: { params: Promise<{ id: string }> }) {
  // Next.js 15 এ params কে await করতে হয়
  const { id } = await params;

  return (
    <div className="p-10">
      <h1 className="text-2xl font-bold">প্রোডাক্ট ডিটেইলস</h1>
      <p>আপনি এখন প্রোডাক্ট নাম্বার: {id} দেখছেন।</p>
    </div>
  );
}
```
*(নোট: `async` ফাংশন ব্যবহার করেছি কারণ এটা সার্ভার কম্পোনেন্ট, আর Next.js 15 এ এটাই নিয়ম)*

---

**Step 4: এক পেজ থেকে অন্য পেজে যাওয়া (Navigation)**

HTML এ আমরা `<a>` ট্যাগ ব্যবহার করতাম। React/Next.js এ ভুলেও `<a>` ট্যাগ ব্যবহার করবেন না পেজ চেঞ্জ করার জন্য। কারণ এতে পুরো পেজ রিফ্রেশ হয় (Slow)।

এর বদলে Next.js দেয় `<Link>` কম্পোনেন্ট।

```tsx
import Link from "next/link";

export default function Navbar() {
  return (
    <nav className="flex gap-4 p-4 bg-gray-200">
      <Link href="/" className="text-blue-600">হোম</Link>
      <Link href="/about" className="text-blue-600">এবাউট</Link>
      <Link href="/products/10" className="text-blue-600">প্রোডাক্ট ১০</Link>
    </nav>
  );
}
```

---

**আপনার এখনকার কাজ (Homework):**

১. `src/app` এর ভেতর `about` নামে একটা ফোল্ডার বানান এবং সেখানে `page.tsx` তৈরি করে কিছু লিখুন।
২. `src/app` এর ভেতর `blog` নামে একটা ফোল্ডার বানান। এর ভেতর `[id]` নামে একটা ফোল্ডার বানান এবং তার ভেতর `page.tsx` দিন।
৩. `blog/[id]/page.tsx` ফাইলে ডাইনামিক `id` টা প্রিন্ট করে দেখান (মনে রাখবেন, `await params` করতে হবে)।
৪. হোম পেজে (`app/page.tsx`) ৩টি লিংক তৈরি করুন:
   - About Page
   - Blog 1
   - Blog 2
৫. লিংকগুলোতে ক্লিক করে চেক করুন পেজ রিফ্রেশ ছাড়া নেভিগেশন হচ্ছে কিনা।
