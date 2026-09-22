# Northwind OSGi Smart Build — GitHub Actions Only

This implementation solves the workshop assignment entirely inside
`.github/workflows/smart-build.yml`. There is no Python orchestration script.

## What the workflow does

1. Detects changed product components from `git diff`.
2. Discovers OSGi bundle symbolic names and exported packages from `MANIFEST.MF`.
3. Discovers feature IDs and feature dependencies from `feature.xml`.
4. Builds a product dependency graph using:
   - feature-to-feature dependencies,
   - `Require-Bundle`, and
   - `Import-Package` → `Export-Package` relationships.
5. Walks the graph in reverse from changed components to select all transitive dependents.
6. Performs a topological sort so dependencies are built before dependents.
7. Displays the selected modules and build order in the GitHub Actions log and job summary.
8. Publishes a Mermaid dependency graph and build-order report as workflow artifacts.
9. Invokes Maven/Tycho sequentially for the calculated product order.

## Build modes

### Build all products

Run the workflow manually and select:

```text
mode = all
```

All nine product components are selected and built in dependency-safe order.

### Build changed products only

For pull requests and pushes, the workflow compares the current commit with the appropriate base commit automatically.

For a manual run, select:

```text
mode = changed
```

Optionally provide `base_sha` to choose the exact commit used for `git diff`.

## Dependency direction

The workflow logs dependencies as:

```text
dependency --> dependent
```

For example, the repository's feature and OSGi metadata establishes relationships such as:

```text
catalog   --> orders
catalog   --> payment
catalog   --> shipping
catalog   --> reporting
customer  --> shipping
customer  --> notification
security  --> payment
orders    --> reporting
payment   --> reporting
shipping  --> notification
thirdparty --> payment
thirdparty --> reporting
thirdparty --> notification
```

The workflow derives these relationships from repository metadata rather than hard-coding the product graph.

## Why this is GitHub Actions only

All orchestration is implemented as Bash steps inside the workflow. GitHub Actions handles checkout, Java setup, Maven caching, change detection, dependency analysis, graph generation, reporting, and Maven/Tycho execution.

Maven/Tycho remains responsible for the actual Java/OSGi build and OSGi resolution.

## CI output

The workflow writes a GitHub Actions Job Summary containing:

- changed components,
- selected rebuild components,
- dependency-safe build order,
- Mermaid dependency graph.

It also uploads `dependency-graph.md` and `build-order.txt` as the `northwind-osgi-build-report` artifact.
