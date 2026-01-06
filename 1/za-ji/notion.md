# Notion

## 🪄 Copy Notion Page as Markdown — The _Unofficial_ Manual

> “Because who has time to re‑format?”  — Someone probably

***

### 🚧 The Annoying Problem

1. **Export frenzy** – Notion’s official export makes you download ZIPs, pray to the PDF gods, **and** wait for an email.
2. **Copy‑paste chaos** – Select‑all → copy → paste gives you ~~beautiful~~ monstrosities full of invisible CSS classes.
3. **I‑just‑need‑the‑body!** – You really wanted just the meat 🥩 (the page content) in nice, clean Markdown.

_Enter our tiny Tampermonkey script dressed like a superhero wearing clipboard undies._

***

### ✨ What the Script Does

| Super‑power                     | Translation for Humans                                             |
| ------------------------------- | ------------------------------------------------------------------ |
| 🖱️ Adds a floating button      | A cute icon parks itself at the bottom‑right of every Notion page. |
| ⌨️ Alt + C hot‑key              | Because moving your mouse is sooo 2022.                            |
| 📄 Grabs only the page content  | Ignores sidebars, comments, random crumbs.                         |
| 🪄 Turns HTML ➜ **Markdown**    | Uses Turndown under the hood. Minimum fluff, maximum readability.  |
| 📋 Copies straight to clipboard | No downloads, no prompts – just paste & roll.                      |
| 🍞 Toasty feedback              | Mini toast pops tell you success or epic fail.                     |

***

### ⚙️ Installation (2 clicks & a wink)

1. Install **Tampermonkey** (Chrome / Edge / Firefox / Safari – your pick).
2. Click **`Raw`** on the userscript file or paste the code into a new script.
3. Hit **Save** → refreshed Notion tab → done.

_Extra credit:_ the script auto‑updates via `@updateURL`, so future you will thank past you.

***

### 🕹️ How to Use It

#### Option A – Button

1. Hover to the bottom‑right corner.
2. Click the 📝 icon that says **Copy MD**.
3. Watch the toast: “✅ Markdown copied”
4. Paste anywhere – VS Code, Slack, your diary.

#### Option B – Keyboard Ninja

Press **Alt + C** (⌥C on macOS). Same magic, 0% mouse.

***

### 🤯 FAQ (Frequently Annoying Questions)

**Q.** It says "Markdown copied", but my clipboard is empty!

**A.** Some browsers block clipboard in insecure contexts. Make sure Notion says `https://` (it does) and you didn't angrily disable clipboard permissions.

***

### 🩹 Troubleshooting

| Symptom              | Fix                                                                                |
| -------------------- | ---------------------------------------------------------------------------------- |
| Button missing       | Refresh page, ensure Tampermonkey icon is ON for this site.                        |
| Markdown looks funny | Blame Notion’s crazy blocks. Then open an issue and we’ll tweak Turndown options.  |
| Hot‑key clashes      | Change `ALT+C` in the script header to something exotic like `CTRL+ALT+SHIFT+F12`. |

***

### 🏗️ Contributing

Pull requests, issues & meme suggestions welcome → `github.com/your‑repo`.

***

### 🪙 License

MIT – do what you wish, credit is sweet but optional.

_(Now go forth and copy things like a boss.)_



Tips:

if you want that file  , you can call me.
