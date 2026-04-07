# 🖨 Introduction to Server-Side Templating

An interactive Reveal.js presentation covering server-side templating — from rendering pipelines and engine comparison through to EJS, Pug, Handlebars, security, performance, and Express integration.

## ▶ [Open the Presentation](https://brendanjameslynskey.github.io/Introduction_to_Server_Side_Templating/)

## 📄 [Markdown Version](presentation.md)

---

## Contents

| # | Topic | Description |
|---|-------|-------------|
| 01 | Title | Introduction to Server-Side Templating |
| 02 | Agenda | Overview of all topics covered |
| 03 | What Is Server-Side Templating? | Core concept, SSR vs CSR, request flow |
| 04 | How Templating Engines Work | Parse, compile, render pipeline and caching |
| 05 | The Template Engine Landscape | EJS, Pug, Handlebars, Nunjucks, Mustache, Liquid comparison |
| 06 | EJS — Embedded JavaScript | Tag reference, syntax, Express setup |
| 07 | Pug — Indentation-Based Syntax | Syntax, mixins, template inheritance |
| 08 | Handlebars — Logic-Less Templates | Expressions, custom helpers, partials |
| 09 | Nunjucks — Jinja2 for JavaScript | Filters, macros, template functions |
| 10 | Template Inheritance & Layouts | extends/block patterns, engine comparison |
| 11 | Passing Data to Templates | res.render, app.locals, res.locals, resolution order |
| 12 | Partials & Includes | Code reuse across EJS, Pug, Handlebars; directory structure |
| 13 | Security — XSS & Escaping | Auto-escaping, raw output, sanitisation, CSP headers |
| 14 | Performance | Template caching, precompilation, streaming renders |
| 15 | Server-Side vs Client-Side Rendering | Trade-offs across SEO, TTFB, bundle size, interactivity |
| 16 | Hybrid Approaches | HTMX, Islands Architecture, Partial Hydration, React Server Components |
| 17 | Choosing a Template Engine | Decision matrix across learning curve, logic, inheritance, security |
| 18 | Express Integration | Complete working example with views/, layouts, partials |
| 19 | Summary & Next Steps | Key takeaways, resources, and further learning |

---

## Slide Controls

| Action | Key |
|--------|-----|
| Next / Previous | `→` `←` or swipe |
| Overview | `Esc` |
| Fullscreen | `F` |
| Export to PDF | Append `?print-pdf` to URL, then print |

## Technology

[Reveal.js 4.6](https://revealjs.com) · [highlight.js](https://highlightjs.org) · Playfair Display + DM Sans + JetBrains Mono

Single self-contained `index.html` — no build step, no npm, no dependencies to install.

## References

- [EJS Documentation](https://ejs.co)
- [Pug Documentation](https://pugjs.org)
- [Handlebars Guide](https://handlebarsjs.com)
- [Nunjucks Documentation](https://mozilla.github.io/nunjucks/)
- [Mustache Manual](https://mustache.github.io/mustache.5.html)
- [Liquid Template Language](https://shopify.github.io/liquid/)
- [Express Using Template Engines](https://expressjs.com/en/guide/using-template-engines.html)
- [HTMX Documentation](https://htmx.org)
- [Astro Islands Architecture](https://docs.astro.build/en/concepts/islands/)
- [OWASP XSS Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Cross-Site_Scripting_Prevention_Cheat_Sheet.html)

## License

Educational use. Code examples provided as-is.
