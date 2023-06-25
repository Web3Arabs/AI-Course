# بناء بوت دردشة بإستخدام OpenAI

في هذا الدرس، سنبني بوت محادثة بسيط للغاية باستخدام واجهة برمجة تطبيقات OpenAI. سنطلب فقط من المستخدم إدخال بيانات، وإرسالها إلى واجهة برمجة التطبيقات، والحصول على الاستجابة التي يرسلها مرة أخرى.

## التقنيات

سنستخدم في هذا الدرس إطار العمل Nextjs ولغة TypeScript وإطار Tailwind.

تأكد من تثبيت Nodejs بالفعل على جهازك. إذا لم تقم بتنزيله يمكنك تنزيله <a href="https://nodejs.org/en/download" target="_blank">من هنا</a>

سنستخدم أحدث إصدار من Nextjs، والذي يتضمن استخدام موجه التطبيقات. إنه مختلف تمامًا عن الإصدار الذي قد تكون معتادًا عليه حاليًا.

تحتاج أيضًا إلى مفتاح OpenAI API. إذا لم يكن لديك، فإحصل على واحدة <a href="https://platform.openai.com/account/api-keys" target="_blank">من هنا</a>

قم بإستدعاء مشروع Nextjs من خلال إضافة هذا في terminal:

```bash
npx create-next-app@latest
```

والاستمرار في تتبع الخطوات وتثبيت اي package قد يطلبها منك.

اسم المشروع: **chatbot**

تأكد من أن إعدادات مشروعك تبدو بهذا الشكل:
<img src="https://www.web3arabs.com/courses/ai/chatbot/next-chatbot.png"/>

سنقوم بتثبيت مكونات OpenAI في المشروع من خلال إضافة هذا في terminal:

```bash
npm install openai
```

قم بتثبيت حزمة dotenv حتى تتمكن من إستيراد ملف **env.**

```bash
npm install dotenv
```

الان قم بإنشاء ملف **env.** في المشروع الذي قمنا بإنشائه واضف **secret key** الخاص بك

```bash
OPENAI_API_KEY="sk-yoursecretkey"
```

> يمكنك الحصول على secret key الخاص بك <a href="https://platform.openai.com/account/api-keys" target="_blank">من هنا</a>

## الواجهة الأمامية (Front-end)

بعد إن قمنا بإستدعاء مشروع Nextjs وإعداد بيئة العمل الذي سنعمل عليها دعونا نبدء في البناء

إذهب إلى الملف **src/app/globals.css** وقم بإضافة هذا الكود بعد ان تقوم بإزالة الكود السابق

```css
@tailwind base;
@tailwind components;
@tailwind utilities;
```

إذهب إلى ملف **src/app/page.tsx** وقم بإزالة كل الكود الذي ستراه واضف هذا الكود وقم بإتباع كل سطر كما هو موضح وظيفة كل شيء متواجد في الكود.

```TSX
// في الصفحة فيجب ان نجعلها تُعرض من جانب العميل وليس السيرفر useState بما اننا سنقوم بإضافة
"use client";
import { useState } from "react";

export default function Home() {
  /**
   * عبارة عن مجموعة من الكائنات لمساعدة روبوت المحادثة على تتبع محفوظات محادثته مع مستخدم معين
   * ستتم إعادة تعيينه عند إعادة التحميل
   * content و role : كل كائن له خاصيتان
   */
  const [messages, setMessages] = useState([
    {
      role: "assistant",
      content: "مرحباً، كيف يمكننا مساعدتك؟",
    },
  ]);
  // المدخلات التي سيعطيها المستخدم للبوت
  const [theTextarea, setTheTextarea] = useState("");
  // OpenAI ستظهر للمستخدم نص اثناء الإنتظار لرد
  const [isLoading, setIsLoading] = useState(false);

  /**
   * OpenAI تقوم هذه الدالة بالإتصال برد
   */
  const getResponse = async () => {
    // يظهر للمستخدم انه ما زال البوت يقوم بالرد وعليه الإنتظار حتى ينتهي
    setIsLoading(true);
    // فيها messages مصفوفة مؤقتة ووضع جميع عناصر مصفوفة
    let mess = messages;
    /**
     * ندفع مدخلات المستخدم إلى هذه المصفوفة المؤقتة ونضبط رسائلنا
     * لقد قمنا بفعل هذا لأنه ليس من الجيد دفع البيانات الى المصفوفة الأساسية
     * فلذلك قمنا بإنشاء مصفوفة مؤقتة
     */
    mess.push({ role: "user", content: theTextarea });
    // نقوم بإرجاع خانة إدخال المحتوى من جاب المستخدم فارغة
    setTheTextarea("");

    // الخاص بنا الذي لم نقوم بإنشائه بعد API نقوم بإرسال طلب الى
    const response = await fetch("/api", {
      method: "POST",
      headers: {
        "Content-Type": "application/json",
      },

      body: JSON.stringify({ messages }),
    });

    // output نقوم بطلب إرجاع البيانات والمخرجات
    const data = await response.json();
    const { output } = data;

    // التي تحتوي ايضاً على بيانات الادخال messages نقوم بإضافة بيانات الاخراج التي ارسلت لنا من البوت الى مصفوفة
    setMessages((prevMessages) => [...prevMessages, output]);
    // والتي تعني ان البوت قد انتهى من كتابة رده false نقوم بإرجاعها الى
    setIsLoading(false);
  }

  return (
    <main dir="rtl" className="m-2">
      <div>
        <div>
          {/* messages إستدعاء جميع البيانات من مصفوفة */}
          {messages.map((e) => {
            return <div key={e.content}>{e.content}</div>
          })}

          {/* فيظهر للمستخدم بأن البوت ما زال يقوم بالرد true قيمتها isLoading في حال كان */}
          {isLoading ? <div className="text-red-900">جاري الرد...</div> : ""}
        </div>
        {
          /**
           * theTextarea لإستقبال البيانات من المستخدم وتخزينها في المتغيرtextarea قمنا بإنشاء خانة
           * اثناء النقر عليه getResponse ثم قمنا بإنشار زر يتصل بالدالة
           */
        }
        <div className="flex items-center mt-6">
          <textarea
            value={theTextarea}
            onChange={(event) => setTheTextarea(event.target.value)}
            className="w-72 text-black bg-gray-200 px-2 rouded"
          />
          <button
            onClick={getResponse}
            className="bg-red-800 text-white mr-5 px-6 py-3 rounded-sm"
          >
            إرسال
          </button>
        </div>
      </div>
    </main>
  );
}
```

قم بتشغيل المشروع لترى ما الذي سيظهر امامك

```bash
npm run dev
```

هذا ما سيظهر لك اثناء الذهاب الى <a href="http://localhost:3000" target="_blank">localhost:3000</a>

<img src="https://www.web3arabs.com/courses/ai/chatbot/front-end.png"/>

ستجد أنه ما زال لا يعمل!! والسبب أنه يتبقى لنا العمل على الواجهة الخلفية ويُعد أهم جزء لدينا.

## الواجهة الخلفية (Back-end)

إذهب الى مجلد **src/app** وقم بإنشاء مجلد بإسم **api** وفي هذا المجلد قم بإنشاء ملف بإسم **route.ts**

قم بنسخ هذا الكود وإضافته هناك وقم بإتباع كل سطر كما هو موضح اعلى كل سطر.

```TSX
import { NextResponse } from 'next/server';
import { Configuration, OpenAIApi } from 'openai';

/**
 * API دون مشاكل سنحتاج إلى تهيئتها بمفتاح OpenAI لنتمكن من استخدام حزمة
 * في السابق .env الذي قمنا بإضافته في ملف
*/
const configuration = new Configuration({
  apiKey: process.env.OPENAI_API_KEY,
});
const openai = new OpenAIApi(configuration);

export async function POST(req: Request, res: NextResponse) {
  // req.json() متغير يقوم اولاً بتخزين
  const body = await req.json()

  /**
   * OpenAI نقوم بإستخدام واجهة برمجة التطبيقات الخاصة ب
   * "gpt-3.5-turbo" نحدد النموذج ليكون
   * على أنها المصفوفة التي مررناها من الواجهة الأمامية messages التعامل مع الرسائل
   * بمثابة حافظة للمحادثة messages تبدو
   * بإستخدام هذا يعرف الروبوت الخاص بنا ما حدث في الماضي ويقدم إجابات وفقًا لذلك
  */
  const completion = await openai.createChatCompletion({
    model: "gpt-3.5-turbo",
    messages: body.messages,
  });
  console.log(completion.data.choices[0].message);

  // gpt-3.5-turbo يستخرج أول كائن رسالة من ردود
  const theResponse = completion.data.choices[0].message;

  // نقوم بإعادتها إلى الواجهة الامامية
  return NextResponse.json({ output: theResponse }, { status: 200 })
};
```

قم بتجربته الان على <a href="http://localhost:3000" target="_blank">localhost:3000</a>

<img src="https://www.web3arabs.com/courses/ai/chatbot/chatbot.png"/>

عمل رائع! إنه يعمل بشكل جيد 🥳🥳

كما هو الحال دائمًا، إذا كانت لديك أي أسئلة أو شعرت بالتعثر أو أردت فقط أن تقول مرحبًا، فقم بالإنضمام على <a href="https://discord.gg/ykgUvqMc4Q" target="_blank">Discord</a> وسنكون أكثر من سعداء لمساعدتك!