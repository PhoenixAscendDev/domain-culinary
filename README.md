# Culinary Domain — Architectural Definition (Foundation Layer)

## Purpose

A single, reusable foundation for all culinary-focused software. It contains the **Middle Tier** (domain logic) and **Data Repository** (persistence access) that serve as the backbone for higher layers. All upper layers — API, UI, and feature projects — build on this foundation. **Culinary Domain extends PXA’s Aerie, Phoenix Ascend’s developer toolkit, applying its patterns to the culinary domain.**

---

## Scope (What Lives Here)

* **Domain Model**

  * Entities, Value Objects, Domain Events
  * Aggregates & invariants (e.g., Recipe, Ingredient, Unit, ConversionRule, PantryItem, MenuPlan)
* **Domain Services / Middle Tier**

  * Business rules, policies, calculators (e.g., unit normalization, yield scaling, allergen checks)
  * Orchestrations that span multiple aggregates (no IO here except via repos)
* **Repository Interfaces + Data Mappers**

  * `IRecipeRepository`, `IIngredientRepository`, etc.
  * Mapping between domain objects and storage models (DTOs/data rows)
* **Data Repo Implementations (optional per storage)**

  * Concrete repos that target a storage abstraction (kept cloud/storage-agnostic behind interfaces)
* **Shared Kernel for Culinary Concepts**

  * Enums/lexicon (e.g., MeasurementUnit, DietaryTag), canonical IDs, error codes
* **Validation & Normalization**

  * Ingredient canonicalization, unit conversion, allergy flags, nutrition slotting (algorithms only)
* **Test Fixtures**

  * Deterministic fixtures for recipes, ingredients, conversions

---

## Out of Scope (Lives Above in Other Projects)

* **API endpoints/controllers**
* **UI / web apps / CLIs**
* **Feature- or product-specific workflows**
* **Auth, rate limiting, telemetry wiring**
* **Cloud SDKs or direct vendor clients in domain code**

---

## Module Boundaries

* **Domain** (pure): entities, VOs, services, policies, events
* **Application/Middle Tier**: use-cases/interactors (e.g., “GenerateRecipeProfile”, “ConvertUnits”)
* **Data Contracts**: repository interfaces + mapping contracts
* **Data Repo**: concrete repo implementations using a storage abstraction
* **Infra Adapters (thin)**: optional adapters to storage interfaces (kept behind Data Repo)

**Dependency Rule:**
Domain → **depends on PXA’s Aerie**
Application → Domain
Data Repo → Domain + Data Contracts
Upper layers depend **only** on Application + Data Contracts. No upward dependencies.

---

## Key Abstractions (examples)

* **Entities / VOs**

  * `Recipe{ Id, Name, Steps, Items[], Yield }`
  * `RecipeItem{ IngredientId, Quantity, Unit }`
  * `Ingredient{ Id, CanonicalName, Aliases[], NutritionProfile?, AllergenInfo? }`
  * `Unit` (VO), `Quantity` (VO), `ConversionRule` (VO)
* **Services**

  * `UnitConversionService` (natural/precise transforms; deterministic)
  * `RecipeScalingService` (servings/yield adjustments)
  * `IngredientCanonicalizer` (alias → canonical)
  * `AllergenPolicy` (flags cross-ingredient)
* **Repositories (Interfaces)**

  * `IRecipeRepository`
  * `IIngredientRepository`
  * `IConversionRuleRepository`
  * Each returns domain objects; pagination and search go through query objects/specifications.

---

## Data & Storage Philosophy

* **Storage-Agnostic Domain**: domain never references a cloud/vendor SDK.
* **Mapping Layer**: separate storage DTOs from domain models.
* **Identifiers**: treat IDs as strings; domains own their canonical ID formats.
* **Natural vs. Unnatural Units**: middle tier exposes naturalized views; precision preserved internally.

---

## Versioning & Compatibility

* **Semantic Versioning** for the package (`MAJOR.MINOR.PATCH`).
* **Domain Schema Changes**

  * Additive changes → MINOR
  * Breaking model/repo contract changes → MAJOR
* **Migrations** handled by Data Repo implementers; domain provides migration hooks/events but not vendor code.

---

## Validation & Error Handling

* **Result Pattern** for operations (e.g., `MethodResult<T>` or equivalent): success flag, data, error code, context.
* **Validation**: fail fast at aggregate boundaries; include machine-readable codes for UI/API mapping.

---

## Testing Strategy

* **Unit Tests**: pure domain rules, exhaustive edge cases (units, conversions, scaling, allergens).
* **Contract Tests**: repositories must pass a shared suite validating behavior against the interfaces.
* **Deterministic Fixtures**: fixed seed data for repeatability.

---

## Performance & Determinism

* **Pure Functions** for calculators (stateless, time-independent).
* **Cache-Aside Option** at repo layer (opt-in via interface, not embedded in domain logic).

---

## Observability Hooks

* **No vendor coupling.** Expose domain-level events/notifications; upper layers may subscribe and forward to logs/metrics.

---

## Package & Repo Layout (example)

```
/culinary-domain
  /domain                # Entities, VOs, policies, services (pure)
  /application           # Use-cases/interactors
  /contracts             # Repo interfaces, query/specification types
  /data-repo             # Concrete repos (behind interfaces), mappers
  /shared-kernel         # Lexicon, enums, error codes, ID helpers
  /tests
```

---

## Integration Guide (for Upper Layers)

1. **Reference** the `culinary-domain` package.
2. **Register** services and repositories with your DI container.
3. **Use** application use-cases (interactors) as your API/UI boundary.
4. **Translate** domain errors to transport-layer responses in your API project.
5. **Never** import cloud/vendor SDKs into the domain or application modules.

---

## Initial Roadmap (Foundational Backlog)

* Define canonical **Units & Conversions** (core tables + rules)
* Implement **Ingredient Canonicalization**
* Implement **Recipe Scaling** with precision retention
* Define **Repository Contracts** for Recipe, Ingredient, ConversionRule
* Ship **Contract Test Suite** for repo implementers
* Provide **Reference Data Repo** (in-memory) + one example persistent adapter (as a separate package)

---

## Definition of Done (for this Foundation)

* Domain/API boundaries stable and documented
* All repos pass contract tests
* 95%+ unit coverage for calculators/policies
* Public docs: module boundaries, dependency rules, versioning, integration steps
