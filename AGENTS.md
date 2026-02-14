# AGENTS.md

## Scope
Guidance for code agents working in this repository (`Narcissus` WoW addon fork).

## Repository Map
- `Narcissus/`: core addon (UI, settings, modules, API, localization).
- `Narcissus_*/`: load-on-demand companion addons (`Achievements`, `BagFilter`, `Barbershop`, `GamePad`, databases).
- `_Dev/`: helper scripts/data tooling, not runtime addon code.

## Core Runtime Flow
- Entry point is `Narcissus/Narcissus.toc`.
- `Narcissus/Initialization.lua`:
  - Defines `DefaultValues` and initializes `NarcissusDB` / `NarcissusDB_PC`.
  - Runs startup callbacks and `SettingFunctions` hydration.
- `Narcissus/SettingsFrame.lua`:
  - Builds settings UI from declarative category/widget tables.
  - Writes to DB and calls `SettingFunctions.*` or module APIs.

## Config Change Rules (Important)
When adding a new setting, do all of the following:
1. Add default key/value in `DefaultValues` (`Narcissus/Initialization.lua`).
2. If this is a rename, add migration logic in `LoadDatabase()` before defaults are applied.
3. Implement/extend `SettingFunctions.<Name>(value, db)` in the owning module.
4. Ensure `SettingFunctions` handler supports startup hydration:
   - If `value == nil`, read from `db[<key>]`.
5. Add settings widget in `Categories` (`Narcissus/SettingsFrame.lua`) with matching DB key.
6. Add `Narci.L[...]` localization keys (at minimum in `Narcissus/Locales/enUS.lua`).
7. If calling APIs from a load-on-demand module, guard/load module first (`C_AddOns.LoadAddOn`).

## Saved Variable Boundaries
- Account-wide main config: `NarcissusDB`.
- Per-character data: `NarcissusDB_PC`.
- Module-specific DBs exist (example: `NarcissusBagFilterDB`, `NarciBarberShopDB`).
- Preserve existing DB key names for compatibility, including historical typos (example: `AKFScreenDelay`).

## File/Load Order Safety
- WoW addon load order is determined by `.toc` and XML `<Script file=...>` order.
- Do not move files between load phases casually; unresolved globals are common if order changes.
- If you add a new Lua/XML file, wire it into the correct `.toc`/XML sequence.

## Coding Conventions
- Lua style here is legacy WoW addon style: semicolons, local aliases for globals, mixed tabs/spaces depending on file.
- Match the local file’s existing style; avoid broad reformatting.
- Keep edits scoped. Do not touch art/assets unless requested.

## Manual Validation Checklist
- `/reload` with no Lua errors.
- Open addon (`/narci`) and open settings panel.
- Toggle the new option and verify immediate behavior change.
- Reload UI and confirm persistence.
- If per-character behavior is intended, verify on a second character.
- Smoke-test slash commands (`/narci`, `/narci minimap`, `/narci resetposition`).

## Known Practical Constraints
- No automated test suite in this repo; validation is in-game/manual.
- Network/API behavior depends on WoW client state and expansion build.

## Git Task Cycle
- Use the task-oriented git cycle in `docs/git-workflow.md`.
- Required safety mechanism: `.githooks/pre-rebase` creates `backup/<branch>/<timestamp>` before each rebase.
