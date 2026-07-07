# راه‌حل دائمی برای ۲۴/۷ کار کردن بک‌اند روی هاست اشتراکی

## مشکل

در هاست اشتراکی (cPanel) وقتی SSH را می‌بندی، همه فرآیندهای پس‌زمینه (از جمله PM2 و بک‌اند NestJS) kill می‌شوند. در نتیجه APIها 503 برمی‌گردانند.

## راه‌حل: Cron Job + Watchdog Script

یک کرون‌جاب هر دقیقه چک می‌کند که بک‌اند زنده است یا نه. اگر جواب نداد، خودکار ری‌استارتش می‌کند.

### ۱. اسکریپت watchdog

فایل `~/dark-hack-backend/watchdog.sh`:

```bash
#!/bin/bash
PORT=3000
APP_DIR="$HOME/dark-hack-backend"
LOG="$APP_DIR/watchdog.log"
PATH="$HOME/.npm-global/bin:/opt/alt/alt-nodejs22/root/usr/bin:/usr/bin:$PATH"

curl -sf http://127.0.0.1:$PORT/api/profile > /dev/null 2>&1 && exit 0

echo "[$(date)] Backend down. Restarting..." >> "$LOG"
pm2 kill &>/dev/null
sleep 1
cd "$APP_DIR" && pm2 start dist/main.js --name dark-hack-backend >> "$LOG" 2>&1
pm2 save >> "$LOG" 2>&1
echo "[$(date)] Restarted." >> "$LOG"
```

### ۲. کرون‌جاب

اجرای watchdog هر یک دقیقه:

```bash
crontab -e
```

و اضافه کردن این خط:

```
* * * * * /home/marmarys/dark-hack-backend/watchdog.sh >/dev/null 2>&1
```

## نحوه کار

| مرحله | توضیح |
|-------|-------|
| ۱ | کرون‌جاب هر دقیقه watchdog را اجرا می‌کند |
| ۲ | `curl` به `127.0.0.1:3000/api/profile` می‌زند |
| ۳ | اگر پاسخ ۲۰۰ گرفت → بک‌اند سالم است → خروج |
| ۴ | اگر خطا گرفت → پی‌ام‌دوی را kill و دوباره استارت می‌کند |
| ۵ | فرآیند در لاگ `watchdog.log` ثبت می‌شود |

## مزیت

- نیازی به SSH ندارید
- بعد از ری‌استارت هاست یا بسته شدن ترمینال، حداکثر ۱ دقیقه طول می‌کشد تا سرویس برگردد
- لاگ دارد برای عیب‌یابی
