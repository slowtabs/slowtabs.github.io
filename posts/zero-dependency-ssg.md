# How I Built a Zero-Dependency Static Site Generator in a Weekend

Sometimes the best way to understand a tool is to build it yourself. I spent a weekend writing a minimal SSG — no frameworks, no build chains — and learned more about templating than two years of using existing tools.

## Why build your own?

Every static site generator I've used — Hugo, Eleventy, Gatsby — comes with a mental model you have to learn. Layouts, partials, data cascades, plugin systems. They're powerful, but they're also a lot.

I wanted to answer a simple question: **what's the smallest useful SSG you can write from scratch?**

## The architecture

The whole thing is about 200 lines of Node.js. Here's the basic idea:

```
src/
  pages/
    index.md
    about.md
    posts/
      hello-world.md
  templates/
    base.html
    post.html
dist/
  index.html
  about.html
  posts/
    hello-world.html
```

### Step 1: Read all markdown files

```javascript
const fs = require('fs');
const path = require('path');

function getPages(dir) {
  const files = fs.readdirSync(dir, { recursive: true });
  return files
    .filter(f => f.endsWith('.md'))
    .map(f => ({
      path: f,
      content: fs.readFileSync(path.join(dir, f), 'utf-8')
    }));
}
```

### Step 2: Parse frontmatter + markdown

I used a simple regex to extract YAML frontmatter, then a basic markdown-to-HTML converter. For a weekend project, I didn't need full CommonMark support — just headings, paragraphs, code blocks, links, and lists.

```javascript
function parseFrontmatter(content) {
  const match = content.match(/^---\n([\s\S]*?)\n---\n([\s\S]*)$/);
  if (!match) return { meta: {}, body: content };
  
  const meta = {};
  match[1].split('\n').forEach(line => {
    const [key, ...rest] = line.split(':');
    meta[key.trim()] = rest.join(':').trim();
  });
  
  return { meta, body: match[2] };
}
```

### Step 3: Template substitution

Instead of a full template engine, I used simple string replacement:

```javascript
function render(template, data) {
  return template.replace(/\{\{(\w+)\}\}/g, (_, key) => data[key] || '');
}
```

## What I learned

1. **Markdown parsing is harder than it looks.** Even "simple" markdown has edge cases — nested lists, inline code inside links, reference-style links. I ended up handling about 80% of cases, which was fine for my content.

2. **File watching is the killer feature.** The first version required re-running the build manually. Adding `fs.watch` with a debounce made the workflow dramatically better.

3. **Most SSG complexity is in features I don't use.** Pagination, taxonomies, asset pipelines — these are genuinely useful for large sites, but for a personal blog with 20 posts, they're overhead.

## The takeaway

Building this didn't make me stop using Eleventy. But it made me understand *why* Eleventy makes the choices it does. Every abstraction in a mature SSG exists because someone ran into a real problem.

If you've never built a tool you use every day, I highly recommend it. You'll learn more in a weekend than in a month of reading docs.

---

*The full source is about 200 lines. Maybe I'll clean it up and publish it someday — but honestly, the point was the learning, not the artifact.*
