# DRY (Do Not Repeat Yourself)

DRY is a software design principle that encourages removing duplicated logic and keeping a single, authoritative source of truth for knowledge in a codebase.

## What DRY Means

If the same business rule or logic appears in multiple places, it should usually be extracted into one reusable unit such as a function, class, module, or shared service.

## Why DRY Matters

1. Lower maintenance cost: a change is made once instead of many times.
2. Better reliability: fewer inconsistent implementations of the same logic.
3. Easier testing: one shared implementation can be tested thoroughly.
4. Better readability: developers can find the source of truth quickly.
5. Reduced defect risk: avoids bugs caused by updating one copy and missing others.

## Common Places Where Duplication Appears

1. Validation rules repeated across handlers/controllers.
2. Repeated data transformation/mapping logic.
3. Copy-pasted SQL queries or API request-building logic.
4. Duplicated error handling and response formatting.
5. The same constants and configuration values scattered in many files.

## How to Apply DRY in Practice

1. Extract repeated logic into reusable functions or modules.
2. Centralize business rules in domain services.
3. Keep constants/enums/configuration in shared definitions.
4. Use templates or abstractions for repetitive workflows.
5. Prefer composition and utility layers over copy-paste.

## Important Caution

Do not over-abstract too early. Premature DRY can create complicated abstractions that are harder to understand than small, temporary duplication.

A practical approach:

1. If duplication is accidental and already recurring, extract it.
2. If two pieces of code only look similar but may evolve differently, keep them separate until patterns stabilize.

## Quick Example

Instead of repeating email validation in multiple endpoints, create one shared validator and reuse it everywhere.

This keeps behavior consistent and makes future rule changes safer.