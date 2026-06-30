# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

Gameplan is a single-file habit/routine tracker. The **entire app lives in `index.html`** — markup, CSS (in `<style>`), and vanilla JS (in `<script>`). There is no build step, no dependencies, no package manager, and no test suite. The UI language is German.

## Running

Open `index.html` directly in a browser, or serve it (e.g. `python3 -m http.server`) and visit it. There is nothing to build, lint, or test.

## Architecture

Everything is driven by a single `state` object: `{ updatedAt, routines:[{id,name,target,color,created}], logs:{ [routineId]: { [YYYY-MM-DD]: true } } }`.

- **Persistence:** `state` is loaded from / saved to `localStorage` under `STORE_KEY` (`gameplan.v1`). `save()` stamps `updatedAt`, persists, then calls `schedulePush()` for cloud sync. Use `save()` (not `persist()`) for any user-facing mutation so sync stays consistent.
- **Render flow:** `render()` is the single entry point that rebuilds the whole UI from `state` (`renderSummary`, `renderWeek`, `renderStats`, `renderHistory`, `renderHeatmap`). After mutating `state`, call `save()` then `render()`. There is no virtual DOM — render functions build HTML strings and assign `innerHTML`, then re-attach event listeners.
- **Weeks are Monday-based.** All week math goes through `getMonday()`. Dates are stored/compared as `YYYY-MM-DD` strings via `fmt()`/`parse()`; string comparison (`ds > tStr`) is used for past/future checks. A routine only counts from its `created` start date.
- **Cloud sync (optional)** lives in the `CLOUD SYNC (GitHub Gist)` section. It mirrors `state` to a private GitHub Gist (file `gameplan-data.json`) using a user-supplied PAT with `gist` scope, stored separately in `localStorage` under `SYNC_KEY` (`gameplan.sync`). Conflict resolution is last-write-wins by `updatedAt`. Pushes are debounced (`schedulePush` → 1.2s); pulls happen on load and on window `focus`.

## Conventions

- Always `escapeHtml()` user-supplied strings (routine names) when interpolating into `innerHTML`.
- When adding fields to a routine or to `logs`, keep the shape backward-compatible — old `localStorage` data and imported JSON backups (Export/Import buttons) must still load.
- Bumping the `state` schema means changing `STORE_KEY` and/or handling migration in `load()`.
