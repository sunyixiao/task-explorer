# AND/OR Semantics

Use this file when there is any risk of mixing alternative routes with mandatory substeps.

## Core Rule

There are two different relationships between child nodes:

- `OR`: competing routes
- `AND`: mandatory decomposition

Do not blur them.
Keep the final goal at the top and expand downward.
Do not introduce extra `Route` wrapper nodes when `AND/OR` already expresses the structure.

## OR Node

Use `OR` when the children are alternative ways to satisfy the parent.

Rules:

- each child must be a self-sufficient candidate route
- one child succeeding is enough for the parent
- sibling progress must not be merged into one composite success
- siblings may still be explored in parallel if worker capacity allows

Example:

```text
[R][OR] Determine the best data source
|- [R1][OR] Use vendor API
|- [R2][OR] Use public CSV data
\- [R3][OR] Use local database snapshot
```

If `R1` succeeds, `R` is satisfied.
You do not need `R2` or `R3`.

## AND Node

Use `AND` when the parent truly requires multiple child steps to be completed together.

Rules:

- every required child contributes to the same chosen route
- all required children must complete for the parent to complete
- failure of a mandatory child threatens that route unless replaced within the same decomposition
- children may run in parallel if they are independent and workers are available

Example:

```text
[E][AND] Execute vendor-API route
|- [E1] Authenticate
|- [E2] Query price history
\- [E3] Validate returned schema
```

`E` is not complete until `E1`, `E2`, and `E3` are all complete.

## The Common Failure Mode

Bad pattern:

```text
[OR] -> A
 /     \
[AND]   D
/   \
B    C
```

This is wrong as a default task-tree layout because the goal is on the right and the logic reads like a sideways flowchart.

The logical structure should be shown top-down:

- `A` at the top
- `OR` beneath `A`
- `AND` beneath the relevant branch when required

Correct pattern:

```text
[A] Final objective
\- OR
   |- AND
   |  |- [B]
   |  \- [C]
   \- [D]
```

This means:

- complete both `B` and `C`, or
- complete `D`

## Shared Prerequisites

If several `OR` routes need the same setup step, do not fake that by merging the routes.

Use one of these patterns:

1. Lift the shared prerequisite higher in the tree.
2. Create a separate `AND` node for the chosen route after selecting it.
3. Duplicate the prerequisite logically inside each route if that keeps the route self-contained.

Choose the clearest representation, but never make sibling `OR` routes depend on each other.

## Sanity Check

Before showing the tree, ask:

- If one child succeeds, does the parent succeed?
  - yes -> likely `OR`
  - no -> likely `AND`

- Would combining two sibling children be necessary to satisfy the parent?
  - yes -> they should not be siblings under `OR`

- Is this child a route choice or a mandatory step?
  - route choice -> `OR`
  - mandatory step -> `AND`
