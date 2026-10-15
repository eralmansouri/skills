# HTML Report

The review rendered for a human: one self-contained HTML file in the OS temp directory, so nothing lands in the repo. Write it to `<tmpdir>/design-review-<timestamp>.html` (`$TMPDIR`, falling back to `/tmp`; `%TEMP%` on Windows), open it (`open` on macOS, `xdg-open` on Linux, `start` on Windows), and give the user the absolute path.

**Self-contained** means an inline `<style>` and hand-built HTML and SVG diagrams, so the report renders offline and under a strict content policy. The one optional script is Mermaid, for graphs too tangled to place by hand; if it fails to load, its source still reads as a diagram.

## Scaffold

```html
<!doctype html>
<html lang="en">
  <head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1" />
    <title>Design review: {{scope}}</title>
    <style>
      :root {
        --ink: #1c1917; --muted: #78716c; --paper: #fafaf9; --card: #fff; --line: #d6d3d1;
        --accent: #059669; --warn: #d97706; --quiet: #64748b; --leak: #dc2626;
      }
      body { margin: 0; background: var(--paper); color: var(--ink); font: 16px/1.6 system-ui, sans-serif; }
      main { max-width: 64rem; margin: 0 auto; padding: 3rem 1.5rem; }
      h1, h2, h3 { font-family: Georgia, serif; line-height: 1.25; }
      article { background: var(--card); border: 1px solid var(--line); border-radius: 12px; padding: 1.5rem; margin: 2.5rem 0; }
      code, .files { font: 0.85rem ui-monospace, monospace; }
      .files { color: var(--muted); }
      .badge { display: inline-block; padding: 0.25rem 0.6rem; border-radius: 999px; color: #fff;
               font: 600 0.7rem/1 system-ui; text-transform: uppercase; letter-spacing: 0.06em; }
      .strong { background: var(--accent); } .explore { background: var(--warn); } .speculative { background: var(--quiet); }
      .ba { display: grid; grid-template-columns: 1fr 1fr; gap: 1rem; }
      .ba > figure { margin: 0; min-height: 320px; border: 1px solid var(--line); border-radius: 8px; padding: 1rem; }
      .label { font: 600 0.7rem system-ui; text-transform: uppercase; letter-spacing: 0.08em; color: var(--muted); }
      .ctx { border: 1.5px dashed var(--quiet); border-radius: 14px; padding: 0.75rem; }  /* context */
      .mod { border: 1px solid var(--ink); border-radius: 4px; padding: 0.4rem 0.6rem; } /* module */
      .deep { border-width: 3px; background: var(--ink); color: var(--paper); }          /* deep module */
      .lit { background: #fee2e2; border-color: var(--leak); }  /* touched by a change, or a scattered rule */
      .leak { stroke: var(--leak); color: var(--leak); }
      .seam { stroke-dasharray: 4 4; }
      .adr { background: #fef3c7; border-left: 3px solid var(--warn); padding: 0.5rem 0.75rem; }
      @media (max-width: 40rem) { .ba { grid-template-columns: 1fr; } }
    </style>
    <!-- Keep this script only if a diagram uses <pre class="mermaid">. -->
    <script type="module">
      import mermaid from "https://cdn.jsdelivr.net/npm/mermaid@11.4.1/dist/mermaid.esm.min.mjs";
      mermaid.initialize({ startOnLoad: true, theme: "neutral" });
    </script>
  </head>
  <body>
    <main>
      <header>…</header>
      <section id="map">…</section>
      <section id="findings">…</section>
      <section id="holds-up">…</section>
      <section id="top">…</section>
    </main>
  </body>
</html>
```

## Order

1. **Header**: scope, date, short commit SHA, and a one-line legend: solid box = module, dashed region = context, dashed line = seam, red = leak or scatter, thick dark box = deep module. No introduction.
2. **Map**: one diagram of the scope as it is: contexts, how they reach each other, the traced slices. No prose. The reader checks it in seconds, and a wrong map discredits every finding built on it.
3. **Findings**: one card each for Strong and Worth exploring, then Speculative as a one-line list.
4. **Holds up**: a short list, one line each.
5. **Top recommendation**: one larger card: the move, one sentence on why, a link to its card.

## Finding card

The card is the finding format from [SKILL.md](SKILL.md), with a diagram as its centrepiece:

- Title, strength badge, lens tag, files in monospace.
- **Before / after diagram**, side by side.
- Evidence in a line or two, with the change-test count when there is one.
- Problem, Move, and Cost: one sentence each.
- Wins: bullets of six words or fewer.
- ADR callout, if any: one line in the `.adr` box.

If a diagram needs a paragraph to be understood, redraw the diagram.

## Diagram patterns

Pick the pattern that shows this finding's problem, and vary them across the report.

- **Context map**: contexts as dashed regions with their modules inside; arrows between contexts labelled by mechanism (import, table, HTTP, event). Red arrows reach into another context's internals. After: the arrows go through a contract or an event.
- **Layer cake to slice**: before, horizontal bands (controllers, services, repositories, DTOs) with the files of one use case lit in each band; after, one column per use case, the shared domain beside them.
- **Rule scatter**: one invariant, every place it's checked lit in a file list with that copy's variant (30 days, 31 days); after, one mark inside the owning module, callers pointing at it.
- **Aggregate boundary**: a dashed consistency line. Before, a sprawling object graph inside it, loaded for every command; after, only the state the invariant needs, the rest referenced by ID outside the line.
- **Change walk**: the file tree with every file one change touches lit and counted ("7 files, 3 contexts"); after, the same change lands in one module.
- **Repeated switch to seam**: the same `switch` lit in several slices; after, one seam with an adapter per case.
- **Mass diagram**: two rectangles per module, interface over implementation. A shallow module's are nearly equal; a deep one has a short interface over a tall implementation.

## Style

- Editorial, not dashboard: generous whitespace, serif headings, the stone and slate palette above.
- Colour sparingly: one accent, red for leaks and scatter, amber for warnings.
- Diagrams about 320px tall, so before and after sit side by side; they stack on a narrow screen.
- Module labels in `.label` small capitals, so diagrams read as schematics rather than UI.
- Prose uses the vocabulary from [SKILL.md](SKILL.md) and the domain nouns from `GLOSSARY.md`, without hedging. If a sentence could be a bullet, make it one; if a bullet could be cut, cut it.
