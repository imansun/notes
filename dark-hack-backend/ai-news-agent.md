# AI News Agent

یک ایجنت خودکار برای جمع‌آوری، تحلیل و انتشار اخبار هوش مصنوعی در پورتفولیو.

## معماری

```
RSS Sources (8 AI sites)
       |
       v
ArticleAgentService.runAINewsAgent()
       |
       v
AiService.ask() --> OpenCode Server (port 4097)
       |
       v
parseAiNewsResponse() --> JSON
       |
       v
PostsService.create() --> MySQL (posts table)
       |
       v
ArticleSource (article_sources table) برای جلوگیری از تکرار
```

## اندپوینت‌ها

| Method | Path | Auth | توضیح |
|--------|------|------|--------|
| POST | `/api/article-agent/run-ai-news` | JWT admin | اجرای دستی ایجنت |
| GET | `/api/article-agent/history` | JWT admin | تاریخچه پردازش‌ها |

## زمانبندی خودکار

- فایل: `src/tasks/tasks.service.ts`
- کرون: `EVERY_6_HOURS` (هر ۶ ساعت)
- تایم‌زون: `Asia/Tehran`
- خطاها لاگ و catch می‌شوند

## منابع RSS

1. TechCrunch AI
2. VentureBeat AI
3. MIT Technology Review AI
4. Ars Technica AI
5. Google AI Blog
6. OpenAI Blog
7. Meta AI Blog
8. DeepMind Blog

## ساختار خروجی AI

```json
{
  "title_fa": "عنوان فارسی",
  "title_en": "English Title",
  "content_fa": "محتوای کامل فارسی (Markdown)",
  "content_en": "Full English content (Markdown)",
  "excerpt_fa": "خلاصه فارسی",
  "excerpt_en": "English excerpt",
  "tags": "AI, Deep Learning, LLM",
  "imageUrl": "...",
  "sourceUrl": "..."
}
```

## فایل‌های تغییر کرده

- `src/article-agent/article-agent.service.ts` — متدهای جدید + سورس‌ها + پرامپت
- `src/article-agent/article-agent.controller.ts` — اندپوینت `run-ai-news`
- `src/tasks/tasks.service.ts` **[جدید]** — کرون جاب
- `src/tasks/tasks.module.ts` **\[جدید\]** — ماژول تسک‌ها
- `src/app.module.ts` — import TasksModule
- `package.json` — اضافه شدن `@nestjs/schedule`

## نصب

```sh
npm install @nestjs/schedule
```

ساخته شده توسط AI Agent در تاریخ ۲۰۲۶-۰۷-۰۷
