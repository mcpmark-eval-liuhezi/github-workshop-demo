# Contributor FAQ — Getting Started with Vite

> Distilled from the Vite community's GitHub **Discussions** Q&A, primarily
> [vitejs/vite#23854 — "Looking for a sensible starting path (pnpm + motion)"](https://github.com/vitejs/vite/discussions/23854)
> (with corroboration from [#22287](https://github.com/vitejs/vite/discussions/22287) and
> [#23224](https://github.com/vitejs/vite/discussions/23224)).
> Each item is written so it can be walked through live, one step at a time.

## 1. I ran the starter command and read the docs — what do I learn next?

**Q:** I've run `pnpm create vite` and skimmed the docs. What's the sensible order for learning config and plugins?

**A:** Start with the [official Vite guide](https://vite.dev/guide), then pull in the
[Using Plugins](https://vite.dev/guide/using-plugins) section only when you actually need a plugin.
You don't need to learn everything at once — the community answer to nearly every "where do I
start?" question is: guide first, plugins on demand.

## 2. Should I learn the whole config surface up front?

**Q:** There's a lot of configuration available. Do I need it all before I begin?

**A:** No — and this came up repeatedly. Analysis paralysis is a bigger risk than knowledge gaps.
Ship a default scaffold first; touch `vite.config` only when something concrete forces you to.

## 3. Is `pnpm add motion` (or any library) all I need?

**Q:** After installing a library, is there extra wiring, tree-shaking, or build config required?

**A:** For a plain ESM library in a vanilla Vite project: no. Install it and import it —

```js
import { animate } from "motion";

animate(".box", { x: 100, opacity: 1 });
```

— and it just works. If a library *doesn't* work out of the box, that's the moment to check the
"Using Plugins" docs, not before.

## 4. Which `.env` files do I commit, and which stay local?

**Q:** The generic advice says "never commit `.env`", but Vite's docs suggest `.env` is fine. Which is it?

**A:** Vite splits env files by *purpose*, not just by name:

| File | Committed? | Purpose |
| --- | --- | --- |
| `.env` | ✅ yes | Shared, non-secret defaults |
| `.env.development` / `.env.production` | ✅ yes | Shared per-mode defaults |
| `.env.local` | ❌ no (git-ignored) | Your personal overrides |
| `.env.*.local` | ❌ no (git-ignored) | Personal per-mode overrides / secrets |

Rules of thumb:

1. Nothing you'd be unhappy to see in the public JS bundle belongs in a committed file.
2. Keep `.env*.local` in `.gitignore` (Vite's templates do this for you).
3. Commit an `.env.example` with placeholder values so newcomers know what to fill in.

## 5. What's the `VITE_` prefix rule?

**Q:** How do environment variables actually reach my client code?

**A:** Only variables prefixed with `VITE_` are exposed to the browser, via `import.meta.env`:

```js
const apiUrl = import.meta.env.VITE_API_URL; // use `import.meta.env`, not `process.env`
```

The prefix is a **gate for exposure, not a marker of safety** — any `VITE_`-prefixed value is
embedded in the public bundle, so real secrets must never use it. Unprefixed variables stay
server-side only (available in `vite.config`, never in client code).

## 6. Why is `process.env` empty inside `vite.config`?

**Q:** My `.env.development` values aren't in `process.env` until *after* `defineConfig` runs. Am I doing it wrong?

**A:** No — this surprises almost everyone. Vite deliberately loads `.env*` files only *after*
resolving the config (because `root`, `envDir`, and `mode` can be set by the config itself). The
documented solution is `loadEnv` inside the function form of `defineConfig`:

```ts
import { defineConfig, loadEnv } from "vite";

export default defineConfig(({ mode }) => {
  const env = loadEnv(mode, process.cwd(), "");
  return {
    base: env.VITE_BASE_PATH || "/",
  };
});
```

If you need the value to exist *before* Vite starts, it must come from the launching process
(IDE launch profile, a `cross-env` npm script, etc.) — not from a `.env` file.

## 7. Where do I ask questions that aren't bugs?

**Q:** My question is about tooling integration, not a Vite defect. Where does it belong?

**A:** In the repository's **Discussions → Q&A** category, not in Issues. That's where this FAQ's
source material lives, and it's the venue the community uses for onboarding, tooling-adjacent, and
"not-quite-a-bug" questions. When you open a discussion there, include your setup, what you
expected, and what happened — the better-asked threads get answered the same day.
