# Pokémon Type Matchup (JavaFX)

A small JavaFX desktop app that lets you explore **Pokémon type effectiveness**.  
Pick a type from a grid, then view what it is **super effective** against (2×), **not very effective** against (0.5×), and what it has **no effect** on (0×).

---

## What this project does

- Shows a **type selection screen** with all 18 Pokémon types.
- Clicking a type opens a **detail screen** that lists:
  - **Super effective (2×)**
  - **Not very effective (0.5×)**
  - **No effect (0×)**
- The detail screen buttons are clickable too, so you can quickly “browse” between types.

---

## Project structure (important parts)

### `lab6.Main`
Entry point for the JavaFX application. It launches JavaFX and hands control to the UI controller.

### `lab6.ui.AppController`
Acts like a simple “navigation manager”:
- Creates the main `Stage` and root layout (`StackPane`)
- Switches between:
  - `TypeSelectorView` (the grid of types)
  - `TypeDetailView` (matchup details for one type)
- Loads the battleground background image (and falls back gracefully if it fails)

### `lab6.ui.TypeSelectorView`
The “home screen”:
- Displays a welcome header (trainer image + text)
- Builds a grid of **type buttons**
- Each button carries its corresponding type object and routes clicks to the controller

### `lab6.ui.TypeDetailView`
The “details screen”:
- Shows the selected type’s **icon**, **name**, and themed styling
- Renders three sections based on the type’s relationship lists:
  - super effective
  - not very effective
  - no effect
- Each listed type is a button that navigates to that type’s detail page

---

## Type system design

### `lab6.type.PokemonType` (core abstraction)
This is the key model class. Each type has:
- A **name**
- An **icon path**
- A **theme color**
- Three relationship lists:
  - `superEffective`
  - `notVeryEffective`
  - `noEffect`

It also provides an `effectivenessAgainst(opponent)` method that returns:
- `2.0`, `0.5`, `0.0`, or `1.0` depending on the relationship.

### `lab6.type.*Type` classes (BugType, FireType, etc.)
Each concrete type subclass defines its matchup rules by implementing `loadRelations()` and filling the lists.

### `lab6.type.TypeRegistry`
A centralized registry that:
- Instantiates the 18 type singletons
- Calls `loadRelations()` once in a static initializer so all matchups are ready to use

This makes the UI code simple: it can directly reference `TypeRegistry.FIRE`, `TypeRegistry.WATER`, etc.

---

## How to run

### In IntelliJ IDEA (recommended)
1. Open the Maven project.
2. Make sure JavaFX is available (the project includes `javafx-controls` as a dependency).
3. Run `lab6.Main`.

### From Maven (may require extra JavaFX runtime configuration)
This project uses JavaFX. Depending on your setup, running JavaFX apps from plain `mvn exec:java` often needs additional configuration (like the JavaFX Maven plugin or module path options). If you want a clean CLI run, see the “Improvements” section below.

---

## What can be improved / extended

### 1) Dual-type matchups (biggest gameplay feature)
Right now the app models **single attacking type vs single defending type**.

A practical next step:
- Let the user select **two defending types** (e.g., Grass/Steel).
- Compute the combined multiplier by multiplying the two single-type results:
  - `multiplier = atk.effectivenessAgainst(def1) * atk.effectivenessAgainst(def2)`
- Then display grouped outcomes:
  - **4×**, **2×**, **1×**, **0.5×**, **0.25×**, **0×**

UI idea:
- Add a second “Defender Type” selector (or a toggle for “dual type mode”).
- Show results as a list of attacking types that hit the defender for each multiplier tier.

### 2) Show defensive weaknesses/resistances for the selected type
The current detail screen shows what the selected type is good at *attacking*.

Many users also want the defensive view:
- “Damage taken from” (what hits this type for 2× / 0.5× / 0×)

That can be computed without rewriting every type:
- For each possible attacking type `A`, compute `A.effectivenessAgainst(thisType)`
- Sort `A` into buckets (2×, 0.5×, 0×, 1×)

### 3) Add stat indicators (HP / Def / Sp. Def)
If you want “HP, Def, Sp. Def indicators”, you’ll need a simple Pokémon model layer:
- Create a `Pokemon` (or `PokemonSpecies`) class with:
  - `name`, `types (1 or 2)`, and stats (`hp`, `def`, `spDef`, etc.)
- In the UI:
  - show stat bars (e.g., `ProgressBar` or custom rectangles)
  - optionally combine stat view with matchup view (e.g., “Bulky vs Special attackers”)

### 4) Reduce repetition in the type classes
The per-type subclasses are clear and readable, but they’re repetitive.

Options:
- Move matchup data into a single data structure (table/JSON/map) and generate relations.
- Or keep the classes, but add helper methods in `PokemonType` like:
  - `se(TypeRegistry.GRASS, TypeRegistry.PSYCHIC, ...)`
  - `nve(...)`
  - `ne(...)`
This keeps the same design while making `loadRelations()` shorter.

### 5) Resource loading and validation
Type icons are loaded in multiple UI places. Improvements:
- Cache loaded images so you don’t reload the same PNG repeatedly.
- Consider a startup validation step that verifies all image paths exist, so missing resources fail early and clearly.

### 6) Packaging / running from Maven reliably
If you want the app to run consistently outside the IDE, consider adding:
- JavaFX Maven plugin configuration (or an app packaging approach)
- A documented `mvn` command in this README

---

## Notes
- Built with **Java 25** and **JavaFX Controls**.
- Icons and background are stored in `src/main/resources/lab6/`.

---
