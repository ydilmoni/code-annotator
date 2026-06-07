# Code Annotator · מתעד קוד

A Claude Code / Agent Skill that adds rich, intent-focused comments to source code — in **Hebrew or English** — without changing a single line of logic.

סקיל ל-Claude Code שמוסיף הערות עשירות וממוקדות-כוונה לקוד — **בעברית או באנגלית** — בלי לשנות אף שורת לוגיקה.

**[English](#english) · [עברית](#hebrew)**

---

<a name="english"></a>

## English

### What it does

Code Annotator reads your code at the **architecture level** — it understands what each function does, how files and functions connect, and which scopes or contracts other code relies on — then writes:

- **File headers** describing each file's purpose and who calls it
- **Per-function comments** that explain intent, not just syntax
- **Named scope dividers** that group related code into labeled sections
- **Cross-reference notes** that make file-to-file integration explicit

It comments intent and contracts, never the obvious. `i++ // increment i` is noise; *why* a value is passed in cents, or which CSS class a JS file toggles at runtime, is signal worth recording.

### The golden rule

It **never changes behavior**. Comments are purely additive — no renaming, reordering, or silent "fixing." If it spots a real bug, it tells you separately.

### Language

Hebrew or English, decided per file: it matches the language of existing comments, falls back to the language you asked in, and asks once if it's ambiguous. Code identifiers and API names always stay as-is — only the prose is translated.

### Installation (Claude Code)

Copy the skill into your global skills folder so it's available in every project:

```powershell
git clone https://github.com/ydilmoni/code-annotator-claude-skill.git
New-Item -ItemType Directory -Force -Path "$HOME\.claude\skills\code-annotator"
Copy-Item "code-annotator-claude-skill\SKILL.md" "$HOME\.claude\skills\code-annotator\SKILL.md"
```

Reload your editor window, then just ask in natural language:

> Add comments to `lib/repositories/task_repository.dart`

### What the output looks like

It makes cross-file links explicit — each file's header names who it talks to, and contract notes name the exact function or class on the other side:

```js
// ─── format-price.js — currency formatting helper ──────────────────────────
// Pure function, no side effects. Turns an integer amount of minor units
// (cents) into a localized price string.
// Used by: cart.js → renderTotal(). Change the return shape here and you must
// update the assertion in cart.test.js too.
export function formatPrice(minorUnits, currency = 'USD') { /* ... */ }
```

```css
/* ─── cart.css — shopping cart styles ───────────────────────────────────────
   Class names here are a contract with the JS: cart.js toggles .cart--empty
   at runtime, so renaming it here silently breaks the empty-state behavior. */
.cart--empty .checkout-btn { display: none; }
```

### License

MIT

---

<a name="hebrew"></a>

## עברית

### מה הסקיל עושה

מתעד קוד קורא את הקוד שלך **ברמת הארכיטקטורה** — הוא מבין מה כל פונקציה עושה, איך הקבצים והפונקציות מתחברים זה לזה, ועל אילו סקופים או חוזים קוד אחר מסתמך — ואז כותב:

- **כותרות לקבצים** שמתארות את מטרת כל קובץ ומי קורא לו
- **הערות לכל פונקציה** שמסבירות כוונה, לא רק תחביר
- **מחיצות סקופ עם שם** שמקבצות קוד קשור לסקציות מסומנות
- **הערות הצלבה** שהופכות את האינטגרציה בין הקבצים למפורשת

הוא מעיר על כוונה וחוזים, אף פעם לא על המובן מאליו. `i++ // מגדיל את i` זה רעש; *למה* ערך מועבר בסנטים, או איזו מחלקת CSS קובץ JS מחליף בזמן ריצה — זה מידע ששווה לתעד.

### כלל הזהב

הוא **לעולם לא משנה התנהגות**. ההערות הן תוספת בלבד — בלי שינוי שמות, בלי סידור מחדש, בלי "תיקונים" שקטים. אם הוא מזהה באג אמיתי, הוא מציין אותו בנפרד.

### שפה

עברית או אנגלית, נקבע לכל קובץ: הוא מתאים לשפת ההערות הקיימות, נופל חזרה לשפה שבה ביקשת, ושואל פעם אחת אם זה לא ברור. מזהים בקוד ושמות API תמיד נשארים כמו שהם — רק הטקסט הפרוזאי מתורגם.

### התקנה (Claude Code)

העתק את הסקיל לתיקיית הסקילים הגלובלית כדי שיהיה זמין בכל פרויקט:

```powershell
git clone https://github.com/ydilmoni/code-annotator-claude-skill.git
New-Item -ItemType Directory -Force -Path "$HOME\.claude\skills\code-annotator"
Copy-Item "code-annotator-claude-skill\SKILL.md" "$HOME\.claude\skills\code-annotator\SKILL.md"
```

רענן את חלון העורך, ואז פשוט בקש בשפה טבעית:

> תוסיף הערות לקובץ `lib/repositories/task_repository.dart`

### איך נראה הפלט

הוא הופך את החיבורים בין הקבצים למפורשים — הכותרת של כל קובץ מציינת עם מי הוא מדבר, והערות החוזה נוקבות בשם המדויק של הפונקציה או המחלקה בצד השני:

```js
// ─── format-price.js — עזר לפורמט מטבע ──────────────────────────────────────
// פונקציה טהורה, ללא תופעות לוואי. הופכת סכום שלם ביחידות משנה (סנט)
// למחרוזת מחיר מקומית.
// נמצא בשימוש: cart.js → renderTotal(). אם משנים כאן את מבנה ההחזרה,
// צריך לעדכן גם את ה-assertion ב-cart.test.js.
export function formatPrice(minorUnits, currency = 'USD') { /* ... */ }
```

```css
/* ─── cart.css — סגנונות עגלת הקניות ─────────────────────────────────────────
   שמות המחלקות כאן הם חוזה עם ה-JS: cart.js מחליף את .cart--empty בזמן ריצה,
   ולכן שינוי שמו כאן שובר בשקט את התנהגות מצב-הריק. */
.cart--empty .checkout-btn { display: none; }
```

### רישיון

MIT
