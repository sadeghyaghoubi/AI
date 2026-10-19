# راهنمای ساده پیاده‌سازی و کار با JEV

> این فایل یک راهنمای خلاصه و عملی برای شروع کار با JEV است.

---

# 1. JEV چطور کار می‌کند؟

به‌جای تولید متن، روی **تصمیم‌گیری ساختاریافته** تمرکز دارد.

در ساده‌ترین حالت شما دو چیز به آن می‌دهید:

```text
State = اطلاعاتی که باید بررسی شود
Questions = سؤال‌هایی که JEV باید درباره آن اطلاعات جواب دهد
```

و JEV جواب ساختاریافته برمی‌گرداند.

ساختار کلی:

```text
Input
  ↓
State
  ↓
JEV
  ↓
Questions
  ↓
Decision
```

مثلاً:

```text
پیام مشتری:
"سایت من از دیشب باز نمی‌شود و فروش ما متوقف شده."

سؤال:
این پیام مربوط به کدام بخش است؟

گزینه‌ها:
- sales
- support
- finance
```

خروجی ممکن است چیزی شبیه این باشد:

```json
{
  "choice": "support",
  "confidence": 0.94
}
```

---

# 2. سه نوع سؤال اصلی در JEV

JEV سه نوع سؤال اصلی دارد:

## Choice

برای انتخاب بین چند گزینه.

مثال:

```text
این پیام مربوط به کدام بخش است؟

Sales
Support
Finance
```

---

## Score

برای امتیازدهی یا درجه‌بندی.

مثلاً:

```text
میزان عصبانیت مشتری چقدر است؟

0 = آرام
1 = ناراضی
2 = بسیار عصبانی
```

---

## Noul

برای سؤال‌هایی که جوابشان به شکل احتمال درست/غلط است.

مثلاً:

```text
آیا این پیام فوری است؟
```

خروجی مثلاً:

```text
0.91
```

یعنی JEV حدود 91٪ احتمال می‌دهد پیام فوری باشد.

---

# 3. برای شروع چه چیزهایی لازم داریم؟

برای استفاده مستقیم از JEV معمولاً این مراحل را داریم:

1. ساخت حساب TypeSafe
2. دریافت API Key
3. ارسال درخواست به API
4. خواندن پاسخ
5. استفاده از پاسخ در برنامه یا Workflow

API اصلی:

```text
POST https://api.typesafe.ai/v1/systemone
```

Header:

```text
Authorization: Bearer YOUR_API_KEY
Content-Type: application/json
```

بهتر است API Key را داخل کد قرار ندهید و آن را در Environment Variable نگه دارید.

مثلاً:

```bash
export TYPESAFE_API_KEY="your-api-key"
```

---

# 4. اولین مثال ساده

فرض کنیم می‌خواهیم یک پیام مشتری را دسته‌بندی کنیم.

## Request

```json
{
  "model": "jev-latest",
  "state": "سایت ما از صبح قطع شده و مشتری‌ها نمی‌توانند خرید کنند.",
  "questions": {
    "department": {
      "type": "choice",
      "instructions": "این درخواست مربوط به کدام بخش است؟",
      "criteria": {
        "sales": "فروش و خرید",
        "support": "مشکل فنی و پشتیبانی",
        "finance": "پرداخت، فاکتور یا مسائل مالی"
      }
    }
  }
}
```

خروجی ممکن است شبیه این باشد:

```json
{
  "answers": {
    "department": {
      "type": "choice",
      "choice": "support",
      "confidence": 0.95
    }
  }
}
```

پس در برنامه می‌توانیم بگوییم:

```text
اگر department = support
    → Workflow پشتیبانی اجرا شود
```

---

# 5. تست با curl

یک روش ساده برای تست JEV:

```bash
curl -X POST https://api.typesafe.ai/v1/systemone \
  -H "Authorization: Bearer $TYPESAFE_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "jev-latest",
    "state": "مشتری میگوید سایت من قطع شده و فروش ندارم.",
    "questions": {
      "department": {
        "type": "choice",
        "instructions": "این درخواست مربوط به کدام بخش است؟",
        "criteria": {
          "sales": "فروش",
          "support": "پشتیبانی فنی",
          "finance": "امور مالی"
        }
      }
    }
  }'
```

---

# 6. استفاده از JEV در Python

ابتدا SDK را نصب می‌کنیم:

```bash
pip install typesafe-sdk
```

API Key:

```bash
export TYPESAFE_API_KEY="your-api-key"
```

مثال:

```python
from typesafe_sdk import Choice, TypeSafeClient

message = "سایت من از دیشب باز نمی‌شود."

with TypeSafeClient(model="jev-latest") as client:

    response = client.system_one(

        state=message,

        questions={

            "department": Choice(

                instructions="این پیام مربوط به کدام بخش است؟",

                criteria={
                    "sales": "فروش",
                    "support": "مشکلات فنی",
                    "finance": "پرداخت و امور مالی"
                }

            )

        }

    )

print(response.choices["department"].choice)
```

خروجی مثلاً:

```text
support
```

---

# 7. پیاده‌سازی JEV در n8n

برای n8n ساده‌ترین روش استفاده از:

```text
HTTP Request Node
```

است.

Workflow ساده:

```text
Webhook
   ↓
HTTP Request
   ↓
JEV API
   ↓
Switch
   ↓
Workflow مناسب
```

---

# 8. تنظیم HTTP Request در n8n

داخل n8n یک Node با نام:

```text
HTTP Request
```

بسازید.

تنظیمات:

```text
Method:
POST

URL:
https://api.typesafe.ai/v1/systemone
```

Headers:

```text
Authorization
Bearer YOUR_API_KEY
```

و:

```text
Content-Type
application/json
```

---

# 9. Body در n8n

فرض کنیم Webhook یک فیلد به نام:

```text
message
```

دریافت می‌کند.

Body:

```json
{
  "model": "jev-latest",
  "state": "{{$json.message}}",
  "questions": {
    "department": {
      "type": "choice",
      "instructions": "این پیام مربوط به کدام بخش است؟",
      "criteria": {
        "sales": "فروش",
        "support": "مشکل فنی",
        "finance": "امور مالی"
      }
    }
  }
}
```

JEV پیام را دریافت و دسته‌بندی می‌کند.

---

# 10. استفاده از Switch در n8n

بعد از HTTP Request یک:

```text
Switch Node
```

اضافه می‌کنیم.

مقدار مورد بررسی:

```text
{{$json.answers.department.choice}}
```

Routeها:

```text
sales
support
finance
```

در نتیجه:

```text
                     ┌── Sales Workflow
                     │
Webhook → JEV → Switch ── Support Workflow
                     │
                     └── Finance Workflow
```

---

# 11. مثال کاربردی SEO

فرض کنیم یک Keyword داریم:

```text
خرید جارو رباتیک شیائومی
```

می‌خواهیم Intent را تشخیص دهیم.

Request:

```json
{
  "model": "jev-latest",
  "state": "خرید جارو رباتیک شیائومی",
  "questions": {
    "intent": {
      "type": "choice",
      "instructions": "Search Intent این کلمه کلیدی چیست؟",
      "criteria": {
        "informational": "کاربر دنبال اطلاعات است",
        "commercial": "کاربر در حال مقایسه و بررسی قبل از خرید است",
        "transactional": "کاربر قصد خرید دارد",
        "navigational": "کاربر دنبال سایت یا برند مشخص است"
      }
    }
  }
}
```

خروجی احتمالی:

```text
transactional
```

بعد n8n می‌تواند تصمیم بگیرد:

```text
transactional
      ↓
Product / Category Workflow

informational
      ↓
Blog Workflow
```

---

# 12. Workflow پیشنهادی SEO

```text
Google Sheet
      ↓
Keyword
      ↓
JEV
      ↓
Intent Detection
      ↓
Switch
      ↓
┌───────────────────────┐
│ Informational         │ → Blog
│ Commercial            │ → Comparison Content
│ Transactional         │ → Product / Category
│ Navigational          │ → Brand Page
└───────────────────────┘
```

---

# 13. مثال Lead Qualification

فرض کنیم فرم سایت یک Lead ایجاد کرده است.

اطلاعات:

```json
{
  "name": "Ali",
  "message": "برای طراحی سایت فروشگاهی و سئو قیمت میخواهم."
}
```

می‌توانیم از JEV بپرسیم:

```text
آیا این کاربر Lead واقعی و مناسب فروش است؟
```

با Noul:

```json
{
  "qualified_lead": {
    "type": "noul",
    "instructions": "آیا این پیام نشان می‌دهد کاربر واقعاً قصد خرید خدمات دارد؟"
  }
}
```

مثلاً:

```text
qualified_lead = 0.89
```

بعد در n8n:

```text
اگر بیشتر از 0.8 بود
      ↓
ارسال به CRM / Sales

اگر کمتر بود
      ↓
Lead Nurturing
```

> عدد 0.8 فقط یک Threshold نمونه است. مقدار مناسب باید با داده واقعی پروژه تست و تنظیم شود.

---

# 14. ترکیب JEV و GPT

JEV قرار نیست جای GPT را بگیرد.

بهتر است هرکدام کاری که برایش مناسب است انجام دهد.

مثلاً:

```text
Keyword
   ↓
JEV
   ↓
Intent
   ↓
n8n
   ↓
GPT
   ↓
Content
   ↓
WordPress
```

JEV:

```text
تصمیم می‌گیرد.
```

GPT:

```text
محتوا تولید می‌کند.
```

n8n:

```text
فرآیند را اجرا می‌کند.
```

---

# 15. مثال کامل Workflow تولید محتوا

```text
Google Sheets
      ↓
New Keyword
      ↓
JEV
      ↓
Search Intent
      ↓
Keyword Type
      ↓
Switch
      ↓
GPT
      ↓
Generate Outline
      ↓
Generate Article
      ↓
SEO Check
      ↓
WordPress Draft
```

JEV می‌تواند در چند نقطه استفاده شود:

```text
تشخیص Intent

تشخیص نوع صفحه

امتیازدهی کیفیت Lead

انتخاب Workflow

تصمیم برای Human Review
```

---

# 16. استفاده از Confidence

برای بعضی خروجی‌ها JEV مقدار Confidence می‌دهد.

مثلاً:

```text
support
confidence = 0.94
```

می‌توانیم در Workflow قانون بگذاریم:

```text
Confidence > 0.80
      ↓
Automatic Action
```

و:

```text
Confidence پایین
      ↓
GPT / Human Review
```

یک معماری خوب:

```text
JEV Decision
      ↓
Confidence Check
      ↓
 ┌───────────────┐
 ↓               ↓
High             Low
 ↓               ↓
Automation       Human / LLM
```

> Threshold ثابت و جهانی وجود ندارد؛ باید با داده واقعی خودتان تست شود.

---

# 17. ساختار پیشنهادی برای پروژه واقعی

برای شروع پیچیده نکنید.

اول یک کار کوچک انتخاب کنید.

مثلاً:

```text
تشخیص نوع پیام مشتری
```

Workflow:

```text
Webhook
   ↓
JEV
   ↓
Choice
   ↓
Switch
   ↓
Action
```

بعد از اینکه درست کار کرد، موارد دیگر را اضافه کنید.

مثلاً:

```text
Urgency
Lead Quality
Sentiment
Priority
Human Review
```

---

# 18. نکات مهم

## API Key را امن نگه دارید

API Key را:

```text
داخل Frontend
داخل JavaScript مرورگر
داخل GitHub
داخل فایل عمومی
```

قرار ندهید.

ترجیحاً از Environment Variable یا Credential در n8n استفاده کنید.

---

## JEV همیشه درست نیست

خروجی ساختاریافته است، اما تصمیم می‌تواند اشتباه باشد.

برای تصمیم‌های مهم:

```text
Confidence Check
+
Human Review
```

در نظر بگیرید.

---

## اول با داده واقعی تست کنید

مثلاً 100 پیام واقعی را بردارید.

نتیجه صحیح هرکدام را مشخص کنید.

بعد خروجی JEV را مقایسه کنید.

مثلاً:

```text
100 پیام
↓
JEV Classification
↓
Compare with correct labels
↓
Accuracy
```

بعد Threshold مناسب را انتخاب کنید.

---

# 19. خلاصه خیلی ساده

برای کار با JEV:

```text
1. API Key بگیر
2. State بفرست
3. Question تعریف کن
4. JEV تصمیم می‌گیرد
5. Response را بخوان
6. بر اساس Response عملیات انجام بده
```

معماری کلی:

```text
DATA
 ↓
JEV
 ↓
DECISION
 ↓
n8n
 ↓
ACTION
```

و اگر تولید محتوا نیاز بود:

```text
DATA
 ↓
JEV
 ↓
DECISION
 ↓
n8n
 ↓
GPT
 ↓
OUTPUT
```

---

# 20. بهترین پروژه برای شروع

برای یادگیری پیشنهاد می‌شود اولین پروژه این باشد:

```text
Webhook
↓
دریافت پیام
↓
JEV
↓
تشخیص:
Sales / Support / Finance
↓
Switch
↓
سه Workflow مختلف
```

این پروژه ساده است و تقریباً تمام منطق اصلی JEV را به شما یاد می‌دهد.

---

# منابع رسمی

Official TypeSafe Documentation:

```text
https://docs.typesafe.ai/
```

Quick Start:

```text
https://docs.typesafe.ai/introduction/quickstart
```

TypeSafe Console:

```text
https://console.typesafe.ai/
```

API Endpoint:

```text
https://api.typesafe.ai/v1/systemone
```

> اطلاعات این راهنما بر اساس مستندات TypeSafe در اکتبر 2026 تنظیم شده است.
