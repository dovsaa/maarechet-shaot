# הגדרת Firebase לסנכרון בין מורים

## מה זה נותן?
כל מורה שפותח את `index.html` רואה את אותם הנתונים בזמן אמת.
שינוי שמורה אחד עושה (עריכת שיעור, הוספת מורה, קביעת אילוצים) — מופיע אצל כולם תוך שניות.

---

## שלב 1 — יצירת פרויקט Firebase (חינמי)

1. כנסו ל-[https://console.firebase.google.com](https://console.firebase.google.com)
2. לחצו **"Add project"** → בחרו שם (למשל `school-schedule-2024`) → צרו
3. בתפריט השמאלי: **Build → Realtime Database**
4. לחצו **"Create database"**
   - בחרו מיקום: `europe-west1` (קרוב לישראל)
   - התחילו ב-**Test mode** (מאפשר קריאה/כתיבה לכולם ל-30 יום)

---

## שלב 2 — העתקת הגדרות לאפליקציה

1. בתפריט Firebase: **Project settings** (גלגל שיניים למעלה)
2. גללו לסוף הדף → **"Your apps"** → לחצו **</>** (Web app)
3. תנו שם (למשל `schedule`) → **Register app**
4. יופיע קוד כזה — **העתיקו את הערכים**:

```javascript
const firebaseConfig = {
  apiKey: "AIzaSy...",
  authDomain: "school-schedule-2024.firebaseapp.com",
  databaseURL: "https://school-schedule-2024-default-rtdb.europe-west1.firebasedatabase.app",
  projectId: "school-schedule-2024",
  storageBucket: "school-schedule-2024.appspot.com",
  messagingSenderId: "123456789",
  appId: "1:123456789:web:abc123"
};
```

5. פתחו את `index.html` בעורך טקסט (Notepad / VS Code)
6. חפשו `FIREBASE_CONFIG` — תמצאו בלוק כזה בתחילת ה-`<script>`:

```javascript
const FIREBASE_CONFIG = {
  apiKey:            "YOUR_API_KEY",
  authDomain:        "YOUR_PROJECT_ID.firebaseapp.com",
  ...
};
```

7. החליפו כל ערך `"YOUR_..."` בערך האמיתי מ-Firebase
8. שמרו את הקובץ

---

## שלב 3 — שיתוף הקובץ עם מורים

**אפשרות א — Google Drive / OneDrive (פשוטה)**
- העלו את `index.html` לתיקייה משותפת
- כל מורה מוריד את הקובץ ופותח עם דפדפן
- ⚠️ כל שינוי מחייב שמירה מחדש ב-Drive

**אפשרות ב — שרת קבצים (מומלצת)**
- שמרו ב-IIS / nginx / Apache בתוך הרשת הבית-ספרית
- כל מורה ניגש דרך URL קבוע — הקובץ תמיד עדכני

**אפשרות ג — GitHub Pages (חינמי, קל)**
1. צרו repo ב-GitHub
2. העלו את `index.html`
3. Settings → Pages → Branch: main → Save
4. קבלו URL קבוע כמו `https://yourname.github.io/schedule/`

---

## שלב 4 — הגנה על הנתונים (חובה לפני שיתוף!)

⚠️ **חשוב:** Firebase נוצר ב-"Test mode" (פתוח לכולם). חובה להחליף את הכללים לפני שמשתפים את הקישור!

### שלב 4א — הפעלת Anonymous Authentication
1. בתפריט Firebase: **Build → Authentication**
2. לחצו **"Get started"**
3. **"Sign-in method"** → **Anonymous** → הפעל → **Save**

### שלב 4ב — עדכון כללי האבטחה
1. **Realtime Database → Rules**
2. החליפו בכללים הבאים:

```json
{
  "rules": {
    "school-schedule": {
      ".read": "auth != null",
      ".write": "auth != null"
    }
  }
}
```

כללים אלו מאפשרים גישה **רק לדפדפנים שפתחו את index.html** (הם מתחברים אנונימית אוטומטית). מישהו שינסה לגשת לבסיס הנתונים ישירות — יחסם.

> **מה זה אומר בפועל:** כל מורה שפותח את הקובץ עם דפדפן מקבל אסימון אנונימי אוטומטי. אין צורך בהתחברות עם שם משתמש/סיסמה. האבטחה מונעת גישה של בוטים וכלים חיצוניים.

### לאחר עדכון הכללים — הוסיפו Anonymous Auth ל-index.html
פתחו את index.html בעורך טקסט, חפשו את שורת Firebase database SDK והוסיפו מתחתיה:
```html
<script src="https://www.gstatic.com/firebasejs/9.23.0/firebase-auth-compat.js"></script>
```
ובתוך `initFirebase()`, לפני `fbDb.ref('.info/connected')`, הוסיפו:
```js
firebase.auth(app).signInAnonymously().catch(e => console.warn('Auth:', e));
```

---

## אינדיקטור חיבור

בפינה העליונה של האפליקציה יופיע:
- 🟢 **מחובר לענן** — הנתונים מסונכרנים
- 🔴 **מנותק** — עובד מקומית, יסתנכרן עם חזרת החיבור
- ⚠️ **Firebase לא מוגדר** — עוד לא הכנסתם את ה-Config

---

## שאלות נפוצות

**ש: מה קורה אם שני מורים ערכו בו-זמנית?**
הנתון האחרון שנשמר מנצח. אין עדיין נעילה אופטימיסטית.

**ש: האם הנתונים נשמרים אם הדפדפן סגור?**
כן — ב-localStorage מקומי וגם ב-Firebase.

**ש: כמה זה עולה?**
Firebase Spark (חינמי) מאפשר 1GB נתונים ו-10GB/חודש להורדה — יותר מספיק.
