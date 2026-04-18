# Semantic Link – Time Machine

Git-style version control, diff, and debugging for Power BI semantic models in Microsoft Fabric.

> "When a stakeholder says 'this KPI was right last Tuesday,' you currently have no way to prove what changed. Time Machine gives your semantic model a git log."

## What it does

Time Machine treats semantic models like source code with a proper history. Five core operations:

- **Capture** — Take a structured snapshot of the model (TMSL metadata + tracked measure values) with a single function call. Stored in Fabric-native Delta tables.
- **Diff** — Compare any two snapshots to see exactly what changed in the model definition and in the measure outputs. Changes classified as breaking / non-breaking / additive.
- **Bisect** — Given a "good" snapshot and a "bad" snapshot, binary-search in O(log N) probes to find the exact snapshot where a measure's value flipped.
- **Explain** — Rank the structural changes in the culprit snapshot by relevance to the broken measure (DIRECT / DEPENDENCY / peripheral).
- **Visualize** — Plot any tracked measure's value across all snapshots with automatic drift markers.

## How it works

```
┌─────────────────────────┐
│  Live Semantic Model    │
│  (Power BI / Fabric)    │
└───────────┬─────────────┘
            │ sempy.fabric + sempy_labs
            ▼
     capture_snapshot()
            │
  ┌─────────┼──────────────┐
  ▼         ▼              ▼
tm_snapshots  tm_measure   tm_measure
(Delta)       _definitions  _values
              (Delta)       (Delta)
                    │
         ┌──────────┼───────────┐
         ▼          ▼           ▼
      diff       bisect     plot_measure
    _snapshots  _measure     _history
                _change
```

Measures are parsed directly from TMSL (bypassing the `list_measures` cache that can serve stale DAX). Values use defensive numeric coercion to handle numpy/Int64 types correctly.

## Demo

The notebook includes a scripted demo against the Wide World Importers sample dataset:

1. Capture a baseline snapshot
2. Edit a measure in the model (e.g., multiply Total Sales by 0.9)
3. Capture more snapshots with additional changes
4. Run `diff_snapshots()` to see exactly what changed
5. Run `bisect_measure_change()` to find the exact snapshot that broke the measure
6. Run `explain_culprit()` to see a ranked list of suspect changes

Bisect converges in ~3 probes across 7 snapshots and correctly identifies the culprit with a `[DIRECT]` tag.

## Quick start

1. Import the notebook into any Fabric workspace with a lakehouse.
2. Edit the config cell — set `WORKSPACE_NAME`, `DATASET_NAME`, and `TRACKED_MEASURES`.
3. Run Step 3 to create the three Delta tables.
4. Call `capture_snapshot()` to start building history.
5. When something looks wrong, call `diff_snapshots()` or `bisect_measure_change()`.

## Suggested automation

- **Pre-refresh:** trigger `capture_snapshot("pre-refresh")` as the first step of every dataset refresh pipeline.
- **Pre-deploy:** snapshot before any Fabric Git integration push.
- **Scheduled baseline:** daily capture via Data Pipeline.
- **Alerting:** after each capture, diff against previous snapshot; post to Teams/Slack if drift exceeds threshold.

## Stack

- `semantic-link` (`sempy`) — live model introspection and measure evaluation
- `semantic-link-labs` (`sempy_labs`) — TMSL/BIM retrieval
- `deepdiff` — structural object diffing
- `matplotlib` — measure history timeline visualization
- PySpark + Delta Lake — snapshot storage
- Fabric notebook — execution environment

## Submission

Submitted to the [Fabric Semantic Link Developer Experience Challenge](https://community.fabric.microsoft.com), April 2026.

## License

MIT
