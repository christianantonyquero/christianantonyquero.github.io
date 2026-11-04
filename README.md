# christianantonyquero.github.io

Portfolio + blog, served by GitHub Pages (Jekyll).

## Writing a post

Add a Markdown file to `_posts/` named `YYYY-MM-DD-slug.md`:

```markdown
---
title: "Post title"
date: 2026-11-10 09:00:00 +0400
description: "One-sentence summary — shown on the blog index, homepage and link previews."
tags: [ai, django]
series: AI-assisted engineering   # optional
---

Body in Markdown. Fenced code blocks get syntax highlighting.
```

Push to `main` and it's live at `/blog/slug/`. The homepage "Writing" section shows the 3 newest posts automatically; the RSS feed is at `/blog/feed.xml`.

To keep a post unpublished while drafting, add `published: false` to its front matter.
