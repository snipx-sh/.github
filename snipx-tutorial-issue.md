## snipx interactive tutorial playground

**Labels:** `enhancement` `snipx.dev` `vue` `tutorial`
**Repo:** `snipx-sh/snipx.dev`

---

### Background

snipx ([snipx.sh](https://snipx.sh) / [snipx.dev](https://snipx.dev)) is a local-first developer knowledge manager for snippets, docs, and bookmarks. The longer arc of the project is a system that moves knowledge from collection into muscle memory — via interactive tutorials generated from real documentation, and progressive speed-run challenges that strip away scaffolding until syntax becomes automatic.

This issue tracks the implementation of the interactive tutorial playground that sits at the core of that learning system.

A React prototype of the snipx UI already exists (see `src/snipx.tsx` in this repo) but **this feature is not a port of that component**. The tutorial playground is a separate, purpose-built system with its own stack.

---

### What we're building

An interactive, multi-language tutorial playground for snipx.dev — modeled architecturally and visually on [Elysia's tutorial](https://elysiajs.com/tutorial/) and its [source implementation](https://github.com/elysiajs/documentation/tree/main/docs/components/xiao/playground). It is adapted for snipx knowledge scenarios rather than Elysia-specific TypeScript content.

The playground is the primary UI for the snipx learning system: users work through auto-generated lessons drawn from real documentation, producing each code example themselves rather than reading it. Ghost text provides scaffolding. Tests verify their work. Difficulty increases with each pass until the content is second nature.

---

### Tech stack

Match Elysia's actual tutorial stack exactly. Do not introduce alternatives without a clear reason.

| Concern | Technology |
|---------|-----------|
| Components | Vue 3 SFCs, `<script setup lang="ts">` throughout |
| Docs shell | VitePress (or Vite SPA if standalone) |
| State | Pinia |
| Editor | Monaco Editor |
| Browser types | `monaco-editor-auto-typings` via Unpkg |
| In-browser transpilation | Sucrase |
| Module resolution | Rollup |
| Sandboxed execution | Web Worker |
| Panel layout + resize | Reka UI |
| Styling | Tailwind CSS |
| Theme | Catppuccin (default) + Tokyo Night (user-selectable) |

**Do not use React, ReactDOM, styled-components, or inline styles.** Author entirely in the Vue/VitePress/Reka ecosystem.

Tokyo Night is the snipx design system color palette — it must be available as a first-class theme option alongside Catppuccin. The token values are documented in `src/lib/theme.ts` in the snipx.sh repo.

---

### Language support

The playground must handle two distinct execution modes:

**Executable (TypeScript / Hono / Zod)**
- Monaco editor with full IntelliSense
- In-browser transpilation via Sucrase + Rollup
- Execution sandboxed in Web Worker
- Test validation via HTTP test cases (as Elysia does)

**Structural validation (Nushell / Rust / Bash / YAML / VyOS)**
- Monaco editor with syntax highlighting
- No execution — validation via regex or AST structure checks
- Tests verify shape, keywords, patterns — not runtime behavior
- Each language needs its own validator module

Both modes share the same lesson format, UI shell, and test runner component. The distinction is in how tests are evaluated, not in how lessons are structured.

---

### Lesson format

Every lesson is a self-contained metadata object. The format must support all languages and both execution modes.

```ts
interface Lesson {
  id:        string
  title:     string
  lang:      'typescript' | 'nushell' | 'rust' | 'bash' | 'yaml' | 'vyos'
  mode:      'execute' | 'validate'
  code:      string          // starting code shown in the editor
  solution:  string          // used for hint reveal and auto-solve
  hints:     string[]        // progressive hints, revealed one at a time
  tests:     LessonTest[]    // runnable or structural
  nav: {
    prev:    string | null   // lesson id
    next:    string | null   // lesson id
  }
}

interface LessonTest {
  description: string
  type:        'exec' | 'regex' | 'ast'
  value:       string        // regex pattern, HTTP assertion, or AST selector
  expected?:   unknown
}
```

Lessons are loaded from `lessons/[id].ts` files — one lesson per file, importable individually so the bundle doesn't load all lessons upfront.

---

### UI components

All components must be modular and reusable. Follow Elysia's component decomposition exactly where it applies.

- **`<EditorPane>`** — Monaco instance, language-aware, ghost text support, theme sync
- **`<TestRunner>`** — runs tests, shows pass/fail per case, handles both exec and validate modes
- **`<LessonNav>`** — previous/next navigation, progress indicator
- **`<HintPanel>`** — reveals hints one at a time, shows solution on demand
- **`<SplitLayout>`** — Reka UI resizable panels (editor left, output/tests right)
- **`<ThemeToggle>`** — switches between Catppuccin and Tokyo Night, persists to localStorage

---

### State and persistence

- Use Pinia for all lesson state — current lesson id, editor content, test results, hint index, theme
- Persist to `localStorage`: current lesson id, per-lesson editor content, theme preference
- On reload, resume the last active lesson with the last editor state
- Per-lesson progress (tests passed) is tracked but not required to persist across sessions in v1

---

### Reference implementation

Deliver working implementations of four sample lessons:

1. **Welcome** — Monaco TypeScript playground, edit code, see output. Entry point, no prior knowledge assumed.
2. **Nushell pipelines** — structural validation only. User writes a `ps | where cpu > 5 | sort-by cpu --reverse` style pipeline. Regex tests verify pipeline operators and command names.
3. **Rust axum handler** — syntax highlighting + structural validation. Tests check for `async fn`, `State(`, `Path(`, `Ok(`, `StatusCode`.
4. **TypeScript Hono + Zod API** — fully executable. User builds a Hono route with Zod validation. HTTP test cases validate the response shape and status codes.

---

### Relationship to the snipx learning system

This playground is the frontend for the snipx tutorial layer described in the project vision. When a user works through a lesson:

- Completed exercises are saved as tagged snippets in their local snipx library via the snipx API (`localhost:7878`)
- Lesson content is generated from the snipx knowledge base (`~/.snipx/packages/[tool]/tutorials/`)
- The same lesson content is used for the progressive speed-run system (future work) — each difficulty pass strips scaffolding from the same base exercise

The playground does not need to implement the speed-run system now, but the lesson format and component architecture must not preclude it. Design with that extension in mind.

---

### References

- Elysia tutorial playground source: https://github.com/elysiajs/documentation/tree/main/docs/components/xiao/playground
- Monaco config: https://github.com/elysiajs/documentation/blob/main/docs/components/xiao/playground/monaco.ts
- Code execution utils: https://github.com/elysiajs/documentation/blob/main/docs/components/xiao/playground/utils.ts
- snipx UI patterns (React prototype — for visual reference only, not architectural): `src/snipx.tsx` in this repo
- snipx knowledge base structure: https://github.com/snipx-sh/snipx
- snipx.dev docs — tutorial guide: https://snipx.dev/docs/tutorial-guide
- Tokyo Night color tokens: `src/lib/theme.ts` in snipx.sh

---

### Acceptance criteria

- [ ] Playground is visually and architecturally a superset of the Elysia tutorial, adapted for snipx knowledge scenarios
- [ ] All four reference lessons are implemented and working
- [ ] TypeScript lessons execute in-browser via Web Worker; structural lessons validate via regex/AST
- [ ] Both Catppuccin and Tokyo Night themes are available and persist to localStorage
- [ ] Lesson state (current lesson, editor content) resumes correctly on page reload
- [ ] All UI components (editor, test runner, nav, hints, layout) are modular and reusable
- [ ] Lesson format supports all six target languages
- [ ] No React, ReactDOM, styled-components, or inline styles anywhere in this feature
- [ ] README documents the architecture, lesson format, component API, and tech stack rationale

---

### Out of scope for this issue

- Speed-run / progressive difficulty mode (future issue)
- Auto-generation of lessons from doc corpus (future issue)
- Saving completed exercises to the local snipx library (future issue — requires snipx API integration)
- VyOS and advanced Nushell lesson content beyond the reference implementation
