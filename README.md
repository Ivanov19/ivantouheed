# Your personal site — how to use it

## Files
```
mysite/
├─ index.html        ← Home/About (name, photo, intro, updates)
├─ experience.html   ← education / work / clubs
├─ blog.html         ← list of posts
├─ assets/CV.pdf     ← your CV (replace this file, keep the name)
├─ images/me.jpg     ← your square photo (add this file)
└─ posts/
   └─ first-post.html ← example post (copy this for new posts)
```
Each page is self-contained — the styling lives in a `<style>` block at the
top of every file. So every page looks right even when opened on its own.

## Where to edit
- Name, intro, links, updates → `index.html` (look for `<!-- comments -->`)
- Education/jobs/clubs → `experience.html`
- Photo → save a square photo as `images/me.jpg`
- CV → replace `assets/CV.pdf` (keep the name)
- Colors → the `:root { ... }` line near the top of each file. Note: since the
  style is in every page, change the color in ALL files to keep them matching
  (or just find-and-replace the hex code across files).

## Add a blog post (2 steps)
1. Copy `posts/first-post.html`, rename it e.g. `posts/second.html`,
   change the title / date / paragraphs.
2. Add one line to the `<ul>` in `blog.html`:
   ```html
   <li><a href="posts/second.html">My second post</a> <span class="muted">— Jul 2026</span></li>
   ```
Posts sit in `posts/`, so their links use `../` (the example already does this).

## Host on GitHub Pages (free)
1. Make a github.com account.
2. New repository named exactly: `yourusername.github.io`
3. Add file → Upload files → drag in everything inside `mysite/`
   (index.html must be at the top level). Commit.
4. Settings → Pages → Branch: main / root → Save.
5. After ~1 min visit https://yourusername.github.io
