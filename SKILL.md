---
name: code-annotator
description: Adds rich, intent-focused comments to source code in Hebrew or English. Reads code at the architecture level — understands what each function does, how files and functions connect, and which scopes, tags, or contracts other code relies on — then writes file headers, per-function comments, scope dividers, and cross-reference notes. Use this skill whenever the user asks to comment code, document code, explain a file, annotate functions, add headers, mark sections/scopes, or asks "תוסיף הערות", "תתעד את הקוד", "תסביר מה כל פונקציה עושה", or similar — in any language (Dart, TypeScript, JavaScript, PHP, Python, etc.). Trigger even if the user does not say the word "skill".
license: MIT
metadata:
  display_name:
    en: Code Annotator
    he: מתעד קוד
  display_description:
    en: Adds rich, intent-focused comments to code in Hebrew or English — file headers, per-function notes, named scope dividers, and cross-reference contracts — without changing any logic.
    he: מוסיף הערות עשירות וברורות לקוד בעברית או באנגלית — כותרות לקבצים, הערות לכל פונקציה, מחיצות סקופ עם שם, וחוזים בין קבצים — בלי לשנות אף שורת לוגיקה.
  categories:
    - developer-tools
    - documentation
---

# Code Annotator

Add explanatory comments to code without changing a single line of logic. The goal is comments that a new developer could read to understand **what** the code does, **why** it does it that way, and **how** it connects to the rest of the codebase — not comments that merely restate the syntax.

## Golden rules (never break these)

1. **Never change code behavior.** Only add or improve comments and whitespace around them. Do not rename, reorder, refactor, or "fix" anything. If you spot a real bug, mention it to the user separately — do not silently change it.
2. **Never delete working code.** Comments are additive.
3. **Don't restate the obvious.** `i++ // increment i` is noise. Comment intent, contracts, and non-obvious decisions — not self-evident syntax.
4. **Preserve and upgrade existing comments.** If a comment already exists, keep its meaning; improve clarity only if it's wrong or unclear. Don't duplicate it.
5. **Match the file's comment syntax.** `//` for Dart/TS/JS/PHP/C++, `#` for Python/Ruby/shell, `<!-- -->` for HTML, `///` for Dart doc comments where appropriate.

## Choosing the language (Hebrew vs English)

Decide per file, in this order:
1. If the file already has comments, **match their language**.
2. If there are no comments, use the language the user requested in their prompt.
3. If still ambiguous, ask the user once: "עברית או אנגלית להערות?" and apply the answer to the whole task.

Keep code identifiers, API names, and error strings exactly as they are — only the prose of the comments is translated.

## Workflow

### Step 1 — Understand before writing
Before commenting anything, build a mental map:
- Read the target file(s) fully.
- Trace the connections: what does this file import/require? What functions does it call elsewhere? What calls **into** it? (grep for the function/class names across the codebase.)
- Identify the contracts: parameter order that matters, return shapes other code depends on, side effects, required call order, auth/security assumptions, global state touched.

Only once you understand how the file fits into the whole do you start writing comments. This is what separates this skill from line-by-line description.

### Step 2 — File header
At the top of each file, add a header block:

```
// ─── <filename> — <one-line purpose in chosen language> ──────────────────────
// <1–3 lines: what this file is responsible for>
// <connections: who calls it / what it calls — e.g. "נקרא על ידי updateTicketStatus() ב-admin_dashboard.js">
```

Use box-drawing dashes (`─`, U+2500) to pad the divider to a consistent width (~78 cols). Keep it tidy and aligned.

### Step 3 — Scope / section dividers
Split the file into logical scopes and label each one with a named divider:

```
// ─── <scope name> ───────────────────────────────────────────────────────────
// <why this scope exists / what it groups>
```

Examples of scopes: "בדיקת method", "איסוף פרמטרים", "עדכון בDB", "Helpers", "State setup", "Render". Name them by intent, not by mechanics.

### Step 4 — Per-function comments
Above every function/method, explain:
- **What** it does (one line, intent-level).
- **Why / when** it's used, if not obvious.
- **Contract notes** that callers must know: parameter-order gotchas, required preconditions, what it returns, side effects, what happens on error.
- **Cross-references**: who calls this, what it depends on. E.g. `// נקרא על ידי TaskRepository.submit() — מצפה ל-userId כבר מאומת`.

### Step 5 — Inline contract & gotcha notes
Inside functions, add short comments only where a decision is non-obvious or load-bearing. Prioritize:
- Security reasoning ("Prepared Statement מונע SQL Injection — הערכים מועברים בנפרד מהשאילתה").
- Why an error is/ isn't exposed to the user.
- Parameter order that must match something else ("סדר הפרמטרים תואם את הסדר של ? בשאילתה").
- Assumptions about state, auth, or call order that, if violated, break things.

### Step 6 — Review pass
After commenting, re-read the file as if you were new to it. Remove any comment that just echoes the code. Confirm: did you touch any logic? If yes, revert it.

## Style reference (target output)

This is the level of quality to aim for. The key thing to notice: the comments make the **cross-file integration explicit**. Each file's header names who it talks to, and the contract notes name the exact function or class on the other side — so a reader never has to guess how the pieces connect. The example below is three files of a small shopping cart: a shared helper, the code that uses it, and the stylesheet that shares a class-name contract with that code.

**File 1 — `format-price.js` (a shared utility other files depend on):**

```js
// ─── format-price.js — currency formatting helper ──────────────────────────
// Pure function, no side effects. Turns an integer amount of minor units
// (cents / agorot) into a localized price string.
// Used by: cart.js → renderTotal(). If you change the return shape here,
// update the assertion in cart.test.js too.

export function formatPrice(minorUnits, currency = 'USD') {
  // Work in minor units (integers) to avoid floating-point rounding errors.
  // Callers MUST pass whole cents — passing 9.99 instead of 999 is wrong by 100x.
  const major = minorUnits / 100;
  return new Intl.NumberFormat('en-US', { style: 'currency', currency }).format(major);
}
```

**File 2 — `cart.js` (consumes the helper above, and shares a contract with the CSS):**

```js
// ─── cart.js — shopping cart rendering ─────────────────────────────────────
// Owns the cart DOM: reads cart state, renders rows, paints the total.
// Depends on: format-price.js (formatPrice).
// Shares a contract with cart.css: toggles the .cart--empty class — the class
// name must stay identical in both files or the empty-state styling breaks.

import { formatPrice } from './format-price.js';

// ─── total ─────────────────────────────────────────────────────────────────
function renderTotal(items) {
  // sum stays in cents — formatPrice expects minor units, so we never divide here.
  const sum = items.reduce((acc, item) => acc + item.priceCents * item.qty, 0);

  // formatPrice() lives in format-price.js — we pass raw cents, not dollars.
  totalEl.textContent = formatPrice(sum);

  // .cart--empty is consumed by cart.css to hide the checkout button.
  // Rename it here and you must rename it there too — silent breakage otherwise.
  cartEl.classList.toggle('cart--empty', items.length === 0);
}
```

**File 3 — `cart.css` (annotating a non-code file: the skill understands the whole project, not just logic):**

```css
/* ─── cart.css — shopping cart styles ───────────────────────────────────────
   Styles the DOM built by cart.js. Class names here are a contract with the JS:
   cart.js toggles .cart--empty at runtime, so renaming it here silently breaks
   the empty-state behavior. Keep the two files in sync. */

/* ─── empty state ─────────────────────────────────────────────────────────── */
/* Applied when cart.js sets .cart--empty (items.length === 0).
   Hides the checkout button so an empty cart can't be submitted. */
.cart--empty .checkout-btn {
  display: none;
}
```

## Language-specific notes

- **Dart / Flutter**: use `///` doc comments above public classes, widgets, and providers so they show up in IDE tooltips; use `//` for internal scope dividers and inline notes. For Riverpod providers, note what the provider exposes and who watches it.
- **TypeScript / JS (e.g. Supabase Edge Functions)**: note the request/response shape, auth assumptions (who must be authenticated), and CORS handling. Reference shared helpers by name.
- **Python**: `#` comments for scopes; consider a short module docstring `"""..."""` for the file header instead of a `#` block.

## Scope of a run

- If the user points at **one file**, annotate that file fully.
- If the user says "the whole project" / "כל הקוד", work file by file. Confirm the order or start with the most central files (the ones most depended upon). Summarize progress so the user can stop you at any point.
- Always show the user the annotated file(s) and let them confirm the style before mass-applying across many files.
