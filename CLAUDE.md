# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**汉兜 Handle** — A Chinese Hanzi (four-character idiom) variation of Wordle. Players guess a four-character Chinese idiom within 10 attempts. Each guess provides feedback on character correctness (exact match, misplaced, none) and pinyin tone accuracy.

Live site: wordle.luomor.com

## Commands

| Command | Description |
|---------|-------------|
| `pnpm install` | Install dependencies |
| `pnpm dev` | Start dev server on port 4444 |
| `pnpm build` | Production build via Vite |
| `pnpm preview` | Preview production build |
| `pnpm test` | Run vitest tests |
| `pnpm test <file>` | Run a specific test file |
| `pnpm lint` | Run eslint |
| `pnpm run update` | Update idiom database by scraping zdic.net (requires `TEST=true` env, run via `vite-node tools/update.ts`) |

Package manager: pnpm@7.8.0 (pnpm workspace with `packages/*`).

## Architecture

### Tech Stack
- Vue 3 (Composition API) + Vite
- UnoCSS for styling, VueUse for composables
- unplugin-auto-import + unplugin-vue-components for auto-imports
- Vitest for testing, jsdom for DOM environment

### Source Structure

```
src/
  App.vue              # Root component, routes between pages
  main.ts              # Entry point
  state.ts             # Reactive game state (parsedAnswer, parsedTries, isPassed, etc.)
  storage.ts           # localStorage persistence (tries, history, settings)
  i18n.ts              # i18n (zh-cn / zh-tw)
  init.ts              # App initialization logic

  logic/               # Core game engine
    types.ts           # ParsedChar, MatchResult, TriesMeta, InputMode types
    constants.ts       # WORD_LENGTH=4, TRIES_LIMIT=10, START_DATE, etc.
    idioms.ts          # Idiom loading and validation
    check.ts           # Answer checking logic
    utils.ts           # Pinyin parsing utilities
    index.ts           # Re-exports all modules

  answers/             # Daily answer system
    list.ts            # Pre-computed answer list (array of [word, hint] pairs)
    index.ts           # getAnswerOfDay(day) — deterministic via seedrandom
    utils.ts           # Answer utilities

  components/          # Vue components (auto-imported via unplugin-vue-components)
    Play.vue           # Main game board / input
    CharBlock.vue      # Individual character block
    Dashboard.vue      # Stats/history dashboard
    ResultFooter.vue   # End-of-game result display
    ShareDialog.vue    # Share functionality
    Settings.vue       # Game settings modal
    ...

  data/                # Game data
    idioms.txt         ~20k+ valid four-character idioms
    polyphones.json    Special pronunciation exceptions
    new.txt            Pending idioms to process
    t2s.json           Traditional to simplified mapping

packages/tools/        @hankit/tools — Reusable Hanzi/pinyin toolkit
  src/pinyin/          Pinyin parsing, phonetics, styles
  src/shuangpin/       Shuangpin (double pinyin) input support
  src/zhuyin/          Zhuyin (Bopomofo) support
  src/hanzi/           Hanzi utilities (filtering, conversion)
  src/map/             Mapping data (phonetics, tone symbols, etc.)

tools/                 Build/automation scripts
  update.ts            Scrapes zdic.net to update idiom database
  zdict.ts             ZDict API client

test/                  Vitest test files
```

### Key Game Logic Flow

1. **Answer selection**: `answers/index.ts` → `getAnswerOfDay(day)` picks the daily idiom deterministically using seedrandom
2. **Input parsing**: `parseWord()` in `state.ts` converts user input into `ParsedChar[]` (char + pinyin parts)
3. **Checking**: `testAnswer()` compares parsed input against parsed answer, returning `MatchResult` per character
4. **State management**: `state.ts` holds reactive refs for game state; `storage.ts` persists tries/history to localStorage
5. **Input modes**: Supports pinyin (`py`), zhuyin (`zy`), and shuangpin (`sp`) input modes

### Important Details

- The answer list (`src/answers/list.ts`) is pre-generated; after day N exceeds the list length, answers are randomly selected via deterministic seed
- `START_DATE` in `logic/constants.ts` controls the epoch for daily puzzles — currently set to 2026-01-01
- `WORD_LENGTH=4` (four-character idioms), `TRIES_LIMIT=10` (10 attempts max)
- The `@hankit/tools` package is aliased in vite/tsconfig and auto-imported; changes there affect the entire app
- UnoCSS is used for all styling — no traditional CSS files except `src/styles/main.css` for global resets
