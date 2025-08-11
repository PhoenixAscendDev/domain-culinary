# 🧠 CulinaryItem and Recipe Core Model

This document defines the foundational business concepts and relationships that form the backbone of the Culinary Domain project. It focuses on the structural definitions and relationships, not on implementation or specific data values.

---

## 🧱 CulinaryItem

A `CulinaryItem` is any discrete food-related object—raw or prepared—that can appear in the system.

### 🔑 Properties

* **IsRawIngredient** (`bool`)
  Indicates the item cannot be broken down into other CulinaryItems within the system.
  Example purpose: Salt, Olive Oil, Basil Leaves.

* **IsIngredient** (`bool`, *read-only*)
  True if the item appears as an input in at least one Recipe.
  Derived dynamically from system data.

* **IsRecipe** (`bool`, *read-only*)
  True if the item is produced as an output of at least one Recipe.
  Derived dynamically from system data.

* **GroceryTaxonomy**
  Links the item to its Grocery Taxonomy classification (Department → Aisle → ShelfSection).

> **Note:** Nutritional data, allergens, and dietary tags are not intrinsic properties of a CulinaryItem. They are determined by how the item is made (via its Recipe) and may be stored separately.

---

## 🍴 Recipe

A `Recipe` defines how to create one CulinaryItem from others.

### 🧾 Structure

* **InputItems**: `List<CulinaryItem>` — Items required to produce the output.
* **OutputItem**: `CulinaryItem` — The resulting item.
* **Instructions**: `List<string>` — Steps for transformation.

### 🔁 Derived Behavior

* Any item in `InputItems` → `IsIngredient = true`
* The `OutputItem` of a recipe → `IsRecipe = true`

---
