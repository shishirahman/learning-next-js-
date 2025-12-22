

স্বাগতম **Module 4: The Full Stack Ecosystem (Database & Prisma)**-এ।

আমরা এখন **Prisma ORM** ব্যবহার করব। আপনি যেহেতু Node/Express এ কাজ করেছেন, হয়তো Mongoose বা Sequelize এর নাম শুনেছেন। Prisma বর্তমানে Next.js এর সাথে স্ট্যান্ডার্ড হিসেবে চলে। এটা দিয়ে আমরা SQL কোড না লিখেই জাভাস্ক্রিপ্ট দিয়ে ডাটাবেস কন্ট্রোল করব।

শেখার সুবিধার্থে আমরা এখন **SQLite** (লোকাল ফাইল ডাটাবেস) ব্যবহার করব, যাতে আপনার এখনই কোথাও সাইন-আপ করতে না হয়। প্রোডাকশনে শুধু কনফিগারেশন চেঞ্জ করে দিলেই এটা PostgreSQL হয়ে যাবে।

চলুন শুরু করি।

---

**Step 1: Prisma ইন্সটলেশন (Install)**

VS Code এর টার্মিনাল ওপেন করুন (সার্ভার রানিং থাকলে `Ctrl + C` দিয়ে থামান)। তারপর নিচের কমান্ডগুলো দিন:

```bash
npm install prisma --save-dev
npm install @prisma/client
```

ইন্সটল হলে Prisma ইনিশিয়ালাইজ করুন (আমরা SQLite ব্যবহার করব):

```bash
npx prisma init --datasource-provider sqlite
```

এখন দেখবেন আপনার ফোল্ডারে `prisma` নামে একটা নতুন ফোল্ডার এবং `.env` ফাইল তৈরি হয়েছে।

---

**Step 2: ডাটাবেস স্কিমা ডিজাইন (Schema Design)**

`prisma/schema.prisma` ফাইলটি ওপেন করুন। এখানে আমরা বলে দিব আমাদের ডাটাবেসে কি কি টেবিল থাকবে। নিচের কোডটুকু ফাইলের শেষে যোগ করুন:

```prisma
// prisma/schema.prisma

model Post {
  id        String   @id @default(cuid()) // অটোমেটিক ইউনিক আইডি
  title     String
  content   String?
  createdAt DateTime @default(now())
}
```
আমরা `Post` নামে একটা টেবিল বানালাম যার `id`, `title`, `content` আর `createdAt` কলাম থাকবে।

---

**Step 3: ডাটাবেস তৈরি করা (Migration)**

আমরা স্কিমা বানালাম, কিন্তু ডাটাবেস এখনো তৈরি হয়নি। নিচের কমান্ড দিলে Prisma অটোমেটিক একটা `dev.db` ফাইল বানাবে এবং টেবিল সেটআপ করে দিবে।

```bash
npx prisma migrate dev --name init
```
টার্মিনালে সব সবুজ টিক চিহ্ন আসলে বুঝবেন কাজ হয়েছে।

---

**Step 4: Prisma ক্লায়েন্ট সেটআপ (Best Practice)**

Next.js এ ডেভেলপমেন্ট মোডে হট-রিলোড হয়, তাই বারবার ডাটাবেস কানেকশন ওপেন হয়ে এরর দিতে পারে। এটা সলভ করার জন্য একটা স্পেশাল ফাইল বানাতে হয়।

১. `src` ফোল্ডারের ভেতর `lib` নামে একটা ফোল্ডার বানান।
২. `src/lib/db.ts` নামে একটা ফাইল বানান এবং নিচের কোডটি পেস্ট করুন:

```ts
// src/lib/db.ts
import { PrismaClient } from '@prisma/client';

const globalForPrisma = global as unknown as { prisma: PrismaClient };

export const db = globalForPrisma.prisma || new PrismaClient();

if (process.env.NODE_ENV !== 'production') globalForPrisma.prisma = db;
```
ব্যাস! এখন পুরো অ্যাপে আমরা এই `db` ভেরিয়েবল দিয়েই ডাটাবেস এক্সেস করব।

---

**Step 5: ডাটাবেসে ডাটা সেভ করা (Server Action + DB)**

এখন চলুন আপনার আগের `src/app/form/page.tsx` ফাইলে ফিরে যাই। আগে যেখানে `console.log` ছিল, সেখানে এখন সত্যি সত্যি ডাটাবেসে সেভ করব।

কোড আপডেট করুন:

```tsx
// src/app/form/page.tsx
import { db } from "@/lib/db"; // আমাদের বানানো db কানেকশন
import { revalidatePath } from "next/cache"; // পেজ রিফ্রেশ করার জন্য

export default async function FormPage() {
  
  // আমাদের সার্ভার অ্যাকশন
  async function createPost(formData: FormData) {
    "use server";

    const title = formData.get("title") as string;
    const content = formData.get("content") as string;

    // ১. ডাটাবেসে সেভ করা
    await db.post.create({
      data: {
        title: title,
        content: content,
      },
    });

    console.log("ডাটাবেসে সেভ হয়েছে!");

    // ২. পেজটা রিফ্রেশ করা যাতে নতুন ডাটা দেখা যায় (Cache Clear)
    revalidatePath("/form");
  }

  // ৩. ডাটাবেস থেকে সব পোস্ট নিয়ে আসা (Display Data)
  const posts = await db.post.findMany({
    orderBy: { createdAt: "desc" },
  });

  return (
    <div className="p-10 max-w-2xl mx-auto">
      <h1 className="text-3xl font-bold mb-6">পোস্ট করুন</h1>

      <form action={createPost} className="flex flex-col gap-4 mb-10 border p-5 rounded">
        <input name="title" placeholder="টাইটেল" className="border p-2 text-black" required />
        <textarea name="content" placeholder="কি ভাবছেন?" className="border p-2 text-black" required />
        <button className="bg-green-600 text-white p-2 rounded">পোস্ট করুন</button>
      </form>

      <h2 className="text-xl font-bold mb-4">সকল পোস্ট:</h2>
      <div className="space-y-4">
        {posts.map((post) => (
          <div key={post.id} className="border p-4 rounded bg-gray-50 text-black">
            <h3 className="font-bold">{post.title}</h3>
            <p>{post.content}</p>
          </div>
        ))}
      </div>
    </div>
  );
}
```

---

**কি কি হচ্ছে এখানে?**
১. ইউজার ফর্ম সাবমিট করছে -> `createPost` অ্যাকশন কল হচ্ছে।
২. `db.post.create` দিয়ে ডাটাবেসে সেভ হচ্ছে।
৩. `revalidatePath` দিয়ে Next.js কে বলা হচ্ছে, "এই পেজের ডাটা চেঞ্জ হয়েছে, নতুন করে লোড করো"।
৪. `db.post.findMany` দিয়ে নিচে অটোমেটিক লিস্ট আপডেট হয়ে যাচ্ছে।

---

**আপনার এখনকার কাজ (Homework):**

১. উপরে যেভাবে বলেছি, Prisma ইন্সটল এবং সেটআপ করুন।
২. `src/lib/db.ts` ফাইলটি তৈরি করুন।
৩. `src/app/form/page.tsx` আপডেট করে ডাটা সেভ এবং রিড (Create & Read) করার সিস্টেমটা চালু করুন।
৪. ব্রাউজারে ফর্মে কিছু লিখে সাবমিট করুন এবং দেখুন নিচে সেটা সাথে সাথে দেখাচ্ছে কিনা।

কাজ শেষ হলে এবং ডাটাবেস কাজ করলে রিপ্লাই দিন: **"Database Connected"**।

তারপর আমরা যাব আমাদের শেষ মডিউলে (Deployment) এবং আপনাকে ফাইনাল টিপস দিব!
