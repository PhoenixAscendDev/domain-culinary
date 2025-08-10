# 🧠 CulinaryItem - Core Model

This document defines the foundational business concepts and relationships that form the backbone of the recipe system.

---

## 🧱 CulinaryItem

A `CulinaryItem` is any discrete food-related object—raw or prepared—that can appear in a recipe system.

### 🔑 Properties

* **IsRawIngredient** (`bool`)

  * `true` if the item cannot be broken down into other CulinaryItems (in-app).
  * Examples: Salt, Olive Oil, Basil Leaves

* **IsIngredient** (`bool`, *read-only*)

  * `true` if the item appears as an input in at least one Recipe.
  * Derived dynamically based on system data.

* **IsRecipe** (`bool`, *read-only*)

  * `true` if the item is produced as an output of at least one Recipe.
  * Derived dynamically based on system data.

---

## 🍴 Recipe

A `Recipe` is a transformation rule that defines how to create one CulinaryItem from others.

> **Note:** `Recipe` is a separate aggregate root in the domain. It is included here only to provide context for how `CulinaryItem` flags (`IsIngredient`, `IsRecipe`) are derived.

### 🧾 Structure

* **InputItems**: `List<CulinaryItem>`
  Items required to produce the output.

* **OutputItem**: `CulinaryItem`
  The resulting item after following the recipe.

* **Instructions**: `List<string>`
  Steps to combine or transform the inputs.

### 🔁 Derived Behavior

* Any item listed in `InputItems` will have `IsIngredient = true`
* The `OutputItem` of a recipe will have `IsRecipe = true`

---

## 🧪 Example: Pesto Sauce

**Recipe: Make Pesto**

* **Inputs**:

  * Basil (IsRawIngredient)
  * Olive Oil (IsRawIngredient)
  * Parmesan (IsRawIngredient)
  * Garlic (IsRawIngredient)
  * Pine Nuts (IsRawIngredient)
* **Output**:

  * Pesto Sauce

**Resulting Flags:**

* Basil, Olive Oil, etc. → `IsIngredient = true`, `IsRawIngredient = true`
* Pesto Sauce → `IsRecipe = true`, `IsIngredient = false`, `IsRawIngredient = false`

If later used in a new recipe (e.g., "Pesto Noodles"), Pesto Sauce would also have:

* `IsIngredient = true`

---
