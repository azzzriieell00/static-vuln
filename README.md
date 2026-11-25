# NovaByte — Client-Side Vulnerable Site (GitHub Pages)

A deliberately vulnerable **static** website. Because GitHub Pages has no server,
the admin login, editor and upload are all **client-side** — and that is the
lesson. The whole target teaches one of the most common real findings:
**authentication and access control enforced in the browser is no security at all.**

```
 This is a clearly-labelled educational target. It is SAFE to host publicly
 because it has no server, no database, no real accounts, and no other users:
 any "defacement" lives only in your own browser's localStorage. Don't dress it
 up as a real company or use it to collect anyone's credentials.
```

## What's "vulnerable" here (all client-side)

| Feature | The bug | Real finding |
|---|---|---|
| Staff login | password hard-coded in the shipped JS | Secrets in client-side code |
| Admin gate | `isAdmin()` reads a `localStorage` flag you control | Client-side access control |
| Editor | banner saved to `localStorage`, rendered with `innerHTML` | DOM-based stored XSS |
| Upload | `FileReader` accepts any file, no checks | Improper (client-side) file handling |

## The challenge: reach the editor without the password

The login check runs in *your* browser, so there are three ways in — find all three:

**1. Read the password from the source.** Right-click → View Source (or DevTools →
Sources → `index.html`). The credential is right there in the `<script>`:
`ADMIN_USER` / `ADMIN_PASS`. Lesson: anything in client-side code is public.

**2. Set the admin flag yourself.** Open DevTools → Console and run:
```js
localStorage.setItem('nb_admin','1')
```
Reload, click **Staff portal** — you're in the editor, no password used. The gate
was just a flag you control.

**3. Call the internal function directly.** In the Console:
```js
grantAdmin(); go('editor')
```
Nothing stops you calling the app's own functions.

Any of these reveals: `STATIC{client_side_auth_is_no_auth}`

## Then: deface + stored XSS

In the editor, the **Headline** field is saved and rendered raw with `innerHTML`.
- Type a banner containing the word `DEFACED` → flag `STATIC{stored_in_localstorage_rendered_raw}`.
- Because it's raw HTML, inject script-capable markup:
  `<img src=x onerror=alert(document.domain)>` — client-side stored XSS.
- The **image upload** reads any file via `FileReader` and sets it as the banner image.

**Important:** this defacement persists only in **your** browser (localStorage).
On a real server the same change would hit every visitor — here it can't, because
there's no server. That difference between *client-side state* and *server-side
state* is the core takeaway.

---

## Deploy it to GitHub Pages

This is a single `index.html` — perfect for Pages.

**1. Put it in a repo** (from the folder with `index.html`):
```bash
git init
git add index.html README.md
git commit -m "NovaByte client-side vuln lab"
git branch -M main
git remote add origin https://github.com/azzzriieell00/REPO-NAME.git
git push -u origin main
```
(Create the empty repo on github.com first — don't initialise it with a README.)

**2. Turn on Pages:** on GitHub → the repo → **Settings** → **Pages** →
under *Build and deployment*, Source = **Deploy from a branch**, Branch = **main**,
folder = **/ (root)** → **Save**.

**3. Wait ~1 minute.** Your site goes live at:
```
https://azzzriieell00.github.io/REPO-NAME/
```
A project site like this is fine alongside your main `azzzriieell00.github.io`.

> Add a clear note on the repo (and the page already says "learning target, not a
> real company") so nobody mistakes it for a real login.

---

## Now fix it (the real lesson)

You can't truly secure login/editor/upload on a static host — the fix is
*architectural*: those features belong on a server.
- **Auth** must be server-side: the server verifies credentials and issues a
  signed session; the browser never decides who's admin.
- **Secrets** never ship in client code.
- **Stored content** must be escaped on output, not rendered raw.
- **Uploads** must be validated and stored server-side with type/size checks.

So the write-up conclusion for this target is itself a finding:
*"Authentication and content management implemented entirely client-side —
trivially bypassed; must be moved server-side."* That sentence is exactly what
you'd put in a real report.
