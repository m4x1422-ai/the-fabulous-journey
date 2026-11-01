# تعليمات العمل على موقع QMFS (للـ Gemini)

> انسخ هذا الملف باسم `GEMINI.md` إلى **جذر ريبو qmfs**. الـ Gemini CLI يقرأه تلقائيًا.
> (لو تستخدم أداة ثانية: سمّه `AGENTS.md` أو الصقه أول رسالة في المحادثة.)
> املأ كل مكان فيه `<...>` قبل ما تعطيه للـ Gemini.

---

## 0. قواعد أمان (أهم قسم، اقرأه أول)

1. **ممنوع تكتب أي مفتاح أو توكن أو كلمة سر داخل أي ملف في الريبو**، ولا في الكومنتات ولا في الـ commit messages ولا في الـ PR. الأسرار تُقرأ فقط من متغيرات البيئة (Environment Variables) أو من `wrangler secret`.
2. لا تسوي commit لأي ملف `.env` أو `.dev.vars` أو `*.pem` أو `*.key`. تأكد إنها موجودة في `.gitignore`.
3. لو لقيت سر مكشوف في الكود أو في تاريخ git: **وقّف وبلغني** عشان أغيّره (rotate). حذفه من الملف لحاله ما يكفي.
4. إذا الأمر مدمّر أو ما ينعكس (حذف، force-push، تغيير DNS، حذف مشروع) **اسألني قبل التنفيذ** حتى لو كنت تملك الصلاحية.
5. لا تعطِ نفسك صلاحيات أكثر من اللي أعطيتك. لو تحتاج صلاحية إضافية، اطلبها مني وبيّن السبب.

---

## 1. معلومات المشروع

| البند | القيمة |
|---|---|
| اسم الموقع | QMFS |
| الدومين الرئيسي | `<مثال: qmfs.example.com>` |
| ريبو GitHub | `<owner>/<repo>` |
| الفرع الرئيسي | `main` |
| نوع المشروع | `<HTML ثابت / React / Next.js / Astro / ...>` |
| أمر التثبيت | `<npm ci>` |
| أمر البناء | `<npm run build>` |
| مجلد المخرجات | `<dist / build / out / . >` |
| الاستضافة | Cloudflare `<Pages / Workers>` |
| اسم مشروع Cloudflare | `<project-name>` |
| Cloudflare Account ID | `<account-id>` (هذا ليس سرًا، لكن لا تنشره بدون داعي) |
| Workers / APIs مرتبطة | `<اسم الـ worker ورابطه، أو "لا يوجد">` |
| قواعد بيانات / تخزين | `<D1 / KV / R2 / لا يوجد>` |

---

## 2. كيف تحصل على صلاحياتي (أنا أجهزها لك)

أنت ما تقدر "تدخل بحسابي" بنفسك. أنا أعطيك وصول **محدود** بإحدى الطرق التالية، وأنت تتحقق منه بالخطوات في القسم 3.

### GitHub
- الطريقة الأفضل: `gh auth login` (أنا أسجل الدخول مرة وحدة في الترمنل).
- أو متغير بيئة `GH_TOKEN` يحتوي **Fine-grained Personal Access Token** محصور على ريبو QMFS فقط، بصلاحيات:
  - Contents: Read & Write
  - Pull requests: Read & Write
  - Actions: Read
  - Metadata: Read
- لا تحتاج صلاحيات إدارة الريبو (Administration) ولا حذف.

### Cloudflare
- الطريقة 1: `npx wrangler login` (تسجيل دخول عبر المتصفح).
- الطريقة 2 (أفضل للأتمتة): متغيرات بيئة:
  - `CLOUDFLARE_API_TOKEN` ← **API Token محدود**، مو الـ Global API Key.
  - `CLOUDFLARE_ACCOUNT_ID`
- الصلاحيات المطلوبة للتوكن (أعطيها حسب الحاجة فقط):
  - Account → Cloudflare Pages: Edit
  - Account → Workers Scripts: Edit
  - Zone → Zone: Read
  - Zone → DNS: Read (Edit فقط لو طلبت مني تعديل DNS)
  - Zone → Zone Settings: Read (Edit لو نحتاج تغيير إعدادات SSL/Headers)
  - Account → D1 / KV / R2: حسب الاستخدام الفعلي

---

## 3. أول شيء تسويه: تحقق من الوصول (بدون تعديل أي شيء)

شغّل هذي الأوامر بالترتيب وقل لي النتيجة. ما تكمل لو أي واحد فشل.

```bash
# GitHub
gh auth status
gh repo view <owner>/<repo>
git remote -v && git status && git branch -a

# Cloudflare
npx wrangler whoami
npx wrangler pages project list      # لو الاستضافة Pages
npx wrangler deployments list        # لو Workers

# المشروع
git log --oneline | head -10
ls -la
```

بعدها اكتب لي ملخص قصير: وش لقيت، وش الصلاحيات الفعلية، وش اللي ناقص.

---

## 4. طريقة العمل

1. **لا تشتغل مباشرة على `main`.** سوِّ فرع جديد: `git switch -c <type>/<وصف-قصير>` (مثل `fix/security-headers`).
2. غيّر أقل قدر ممكن لكل مهمة، وخلّ كل commit له غرض واحد.
3. قبل أي رفع: شغّل البناء والفحوصات محليًا (`<أمر البناء>` و `<أمر الاختبارات إن وجد>`).
4. ارفع الفرع وافتح **Pull Request**، واشرح فيه: وش تغيّر، ليش، وكيف جرّبته.
5. اختبر على **Preview Deployment** (Cloudflare يعطي رابط معاينة لكل فرع) قبل الدمج.
6. **لا تدمج في `main` ولا تنشر على الـ production إلا بعد موافقتي الصريحة.**
7. بعد النشر: افتح الموقع الحقيقي وتأكد إنه شغّال (الصفحة الرئيسية + الصفحات المهمة + الـ API).

### أوامر النشر (عدّلها حسب مشروعك)

```bash
# Pages
npm run build
npx wrangler pages deploy <مجلد المخرجات> --project-name=<project-name> --branch=<اسم-الفرع>   # معاينة
# production فقط بعد موافقتي: --branch=main

# Workers
npx wrangler deploy --dry-run      # افحص أول
npx wrangler deploy                # بعد موافقتي
```

لو عندنا GitHub Actions أو ربط تلقائي بين GitHub وCloudflare، **الأفضل نعتمد عليه** بدل الرفع اليدوي. تحقق أول هل هو موجود (`.github/workflows/` أو إعدادات Pages → Builds).

---

## 5. قائمة سد الثغرات (ابدأ بالتدقيق، ولا تغيّر شيء قبل ما تعرض علي النتائج)

### الكود والريبو
- [ ] فحص أسرار مكشوفة في الملفات وتاريخ git (`gitleaks` أو `git log -p | grep -i -E "key|token|secret|password"`).
- [ ] `.gitignore` يغطي `.env*` و`.dev.vars` و`node_modules`.
- [ ] تحديث الاعتماديات وفحصها: `npm audit` (لا تسوي `--force` بدون ما تسألني).
- [ ] أي سكربت أو CSS خارجي (CDN) يُحمَّل مع `integrity` (SRI) و`crossorigin`، أو ننقله محليًا.
- [ ] لا يوجد `innerHTML` أو `eval` على مدخلات غير موثوقة (XSS).
- [ ] أي نماذج أو API لها تحقق من المدخلات (validation) وحماية من الإساءة (rate limiting).

### Cloudflare والـ Headers
- [ ] SSL/TLS على **Full (strict)**، و**Always Use HTTPS** مفعّل، و**HSTS** مفعّل.
- [ ] Security headers عبر ملف `_headers` (Pages) أو الـ Worker:
  - `Content-Security-Policy` (ابدأ بوضع Report-Only ثم شدّد)
  - `X-Content-Type-Options: nosniff`
  - `X-Frame-Options: DENY` (أو `frame-ancestors` في CSP)
  - `Referrer-Policy: strict-origin-when-cross-origin`
  - `Permissions-Policy` يمنع الكاميرا/المايك/الموقع إذا ما نحتاجها
- [ ] CORS في الـ Workers مقيّد على دومين الموقع فقط، مو `*`.
- [ ] الأسرار في الـ Workers عبر `wrangler secret put` وليس في `wrangler.toml`.
- [ ] مراجعة سجلات DNS (لا توجد سجلات قديمة تشير لخدمات ما نستخدمها، خطر subdomain takeover). **قراءة فقط**، أي تعديل بعد موافقتي.
- [ ] مراجعة WAF / Bot Fight Mode / Rate Limiting rules المتاحة في الخطة.
- [ ] تأكد إن روابط الـ preview مو مفتوحة للعموم لو فيها محتوى حساس (Cloudflare Access).

بعد الفحص اكتب تقرير: **الثغرة، الخطورة (عالية/متوسطة/منخفضة)، الإصلاح المقترح، وهل يحتاج موافقتي**. ثم ننفذ بالترتيب من الأخطر.

---

## 6. ممنوع بدون إذن صريح مني

- `git push --force` أو إعادة كتابة التاريخ على `main`.
- حذف فروع، ريبوهات، مشاريع Pages، Workers، قواعد بيانات، أو ملفات R2/KV.
- تعديل أو حذف سجلات DNS، أو نقل الدومين، أو تغيير nameservers.
- إنشاء أو تدوير أو حذف API tokens.
- تغيير إعدادات الفوترة أو الخطة.
- تنفيذ migrations على قاعدة بيانات الـ production.
- نشر مباشر على الـ production.
- تثبيت اعتماديات جديدة غير معروفة أو تشغيل سكربتات من الإنترنت (`curl ... | sh`).

---

## 7. أسلوب التواصل

- رد بالعربي، وبشكل مختصر وواضح، وبدون مصطلحات معقدة إلا لما تحتاج.
- قبل أي مهمة كبيرة: اكتب خطتك في 3-6 نقاط وانتظر موافقتي.
- بعد كل مهمة: لخّص وش تغيّر، وفي أي ملفات، وكيف أتحقق، ورابط الـ PR أو الـ preview.
- لو ما تعرف أو ما أنت متأكد، قل ذلك واسأل. لا تخمّن في الإعدادات الحساسة.
- لو فشل أمر، اعرض لي رسالة الخطأ الفعلية، ولا تقل "تم" إلا لما تتأكد فعليًا.

---

## 8. ملاحظات خاصة بالمشروع (عبّيها أنت)

- `<أي قيود: مثلاً "الموقع ثابت بدون build"، "لا تستخدم frameworks"، "التصميم RTL">`
- `<صفحات أو ميزات حساسة ما تتغير>`
- `<أشياء معروف إنها مكسورة أو قيد العمل>`
