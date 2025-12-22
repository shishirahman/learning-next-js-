 **Module 3: The "Back End" (Data Fetching & Server Actions)**।

এই মডিউলটা আপনার জীবনের অনেক কষ্ট কমিয়ে দিবে। আপনি যেহেতু Node/Express জানেন, আপনি অভ্যস্ত `app.get('/api/users')` বা `axios.get()` এসব ব্যবহারে।

Next.js এ আমরা এসব ছাড়াই কাজ করব। বিশ্বাস হচ্ছে না? চলুন দেখি।

---

**Topic 1: ডাটা ফেচিং (GET Request) - `useEffect` এর দিন শেষ!**

**আগে (React + Vite):**
আগে ডাটা লোড করতে হলে `useEffect` লাগত, `useState` লাগত, আবার লোডিং স্টেট হ্যান্ডেল করা লাগত।

**এখন (Next.js Server Component):**
যেহেতু আমাদের কম্পোনেন্ট সার্ভারেই রান হচ্ছে, আমরা সরাসরি কম্পোনেন্টের ভেতর ডাটাবেস বা API কল করতে পারি। `useEffect` এর কোনো দরকার নাই!

**কোড উদাহরণ:**
আমরাjsonplaceholder থেকে কিছু ডামি ইউজার এর ডাটা আনব।

```tsx
// src/app/users/page.tsx

// 1. কম্পোনেন্ট অবশ্যই async হতে হবে
export default async function UsersPage() {
  
  // 2. সরাসরি fetch কল! (কোনো useEffect নাই)
  // Next.js সার্ভার সাইডে এই রিকোয়েস্ট পাঠাবে
  const res = await fetch("https://jsonplaceholder.typicode.com/users");
  const users = await res.json();

  return (
    <div className="p-10">
      <h1 className="text-2xl font-bold mb-5">ইউজার লিস্ট</h1>
      <ul>
        {users.map((user: any) => (
          <li key={user.id} className="mb-2 p-2 bg-gray-100 rounded">
            {user.name} - <span className="text-blue-500">{user.email}</span>
          </li>
        ))}
      </ul>
    </div>
  );
}
```
দেখলেন? কোড অর্ধেক হয়ে গেছে! এটা সার্ভারে রেন্ডার হয়ে ডাটা সহ HTML হয়ে ব্রাউজারে আসবে। তাই এসইও (SEO) এর জন্য এটা সেরা।

---

**Topic 2: সার্ভার অ্যাকশন (POST/Form Submit) - API Route এর দিন শেষ!**

Express এ ফর্ম সাবমিট করতে হলে আপনি একটা API route বানাতেন (`app.post('/api/submit')`), তারপর ফ্রন্টএন্ড থেকে `axios.post` করতেন।

Next.js এ **Server Actions** দিয়ে আমরা জাভাস্ক্রিপ্ট ফাংশনের মতোই ব্যাকএন্ড লজিক লিখব।

**নিয়ম:** ফাংশনের শুরুতে `"use server"` লিখতে হবে।

চলুন একটা ফর্ম বানাই যেখানে ইউজার নাম সাবমিট করবে, আর আমরা সেটা রিসিভ করব।

```tsx
// src/app/actions/page.tsx

// এটা সার্ভার কম্পোনেন্ট
export default function ActionPage() {

  // এই ফাংশনটা সার্ভারে রান হবে! ব্রাউজারে না।
  async function createKey(formData: FormData) {
    "use server"; // এই লাইনটা বলছে এটা সার্ভার অ্যাকশন

    const name = formData.get("username");
    console.log("সার্ভারে ডাটা পেয়েছি:", name);
    
    // এখানে আপনি ডাটাবেসে সেভ করার কোড লিখবেন (ভবিষ্যতে)
    // যেমন: await db.user.create({ name })
  }

  return (
    <div className="p-10">
      <h1 className="mb-4">নতুন ইউজার তৈরি করুন</h1>
      
      {/* action এর মধ্যে সরাসরি সার্ভার ফাংশন দিয়ে দিলাম! */}
      <form action={createKey} className="flex gap-2">
        <input 
          type="text" 
          name="username" 
          placeholder="আপনার নাম লিখুন" 
          className="border p-2 text-black"
        />
        <button type="submit" className="bg-blue-600 text-white p-2 rounded">
          সাবমিট
        </button>
      </form>
    </div>
  );
}
```

**ম্যাজিকটা বুঝলেন?**
কোনো `useState` নাই, কোনো `axios` নাই, কোনো API Route নাই। ফর্ম সাবমিট করলে Next.js অটোমেটিক ডাটা সার্ভারে পাঠাবে এবং `createKey` ফাংশনটা এক্সিকিউট করবে। `console.log` টা আপনার VS Code এর টার্মিনালে দেখাবে।

---

**আপনার এখনকার কাজ (Homework):**

১. `src/app/users` ফোল্ডার বানিয়ে `page.tsx` খুলুন। উপরের **Topic 1** এর কোডটা বসিয়ে দেখুন ডাটা লোড হয় কিনা।
২. `src/app/form` ফোল্ডার বানিয়ে `page.tsx` খুলুন।
৩. সেখানে একটা ফর্ম বানান (ইনপুট: টাইটেল, বডি)।
৪. ফর্মের জন্য একটা **Server Action** লিখুন।
৫. ফর্ম সাবমিট করলে টার্মিনালে টাইটেল আর বডি প্রিন্ট করে দেখান।

(নোট: ফর্ম সাবমিট করার পর পেজ রিফ্রেশ বা কিছু হবে না, শুধু VS Code টার্মিনালে লগ দেখলেই বুঝবেন কাজ হয়েছে)।

কাজ শেষ হলে রিপ্লাই দিন: **"Backend Logic Done"**। এরপর আমরা **Module 4** এ আসল ডাটাবেস কানেক্ট করব!
