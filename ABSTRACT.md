# Submission Abstract — Semantic Link: Time Machine

## Short version (for the Notebook Gallery "short abstract" field)

**Semantic Link – Time Machine** gives Power BI semantic models a `git log`. Capture structured snapshots of a model's definition *and* its measure values over time, diff any two points in history to see exactly what changed, use automated binary-search bisect to find the precise snapshot where a measure's value flipped — along with a ranked root-cause explanation of which model change caused it — and visualize any tracked measure's full value history with automatic drift markers. Built on `semantic-link` and `semantic-link-labs`, with snapshot storage in Fabric-native Delta tables. Adopt in any workspace by editing one configuration cell.

---

## Long version (for the notebook header and challenge thread)

### The problem

Semantic models evolve constantly — measures get tweaked, relationships change, columns are renamed, calculated columns get refactored. When a KPI silently starts returning the wrong number, developers have almost no tools to answer the most basic questions:

- *When* did this measure's value change?
- *What* change caused it?
- *Can I see the model exactly as it was last Tuesday?*

Today, these questions are answered with memory, manual Git blame (if you're lucky enough to have Fabric Git integration configured), and rerunning reports against old backups. For teams shipping changes daily, this is the single biggest source of lost debugging time.

### The solution

Time Machine treats semantic models like source code with a proper history. It introduces three operations:

**Capture.** A single `capture_snapshot()` call pulls the full TMSL (BIM JSON) via `sempy-labs`, parses the authoritative measure list directly out of TMSL (bypassing the `list_measures` cache that otherwise serves stale DAX), counts columns / tables / relationships via `sempy.fabric`, evaluates a configurable set of tracked measures via `fabric.evaluate_measure()`, and writes all of it to three Delta tables in the lakehouse.

**Diff.** `diff_snapshots(a, b)` compares any two snapshots, classifies every measure/structural change as breaking / non-breaking / additive, detects value drift on tracked measures beyond a configurable tolerance, and renders a readable markdown report.

**Bisect.** `bisect_measure_change(measure, good_id, bad_id)` binary-searches through the intermediate snapshots in O(log N) probes to find the exact snapshot where a measure's value flipped — and `explain_culprit()` ranks the structural changes in that snapshot by their relevance to the target measure (DIRECT / DEPENDENCY / peripheral), surfacing the root cause.

**Visualize.** `plot_measure_history()` renders any tracked measure's evaluated value across every snapshot on a timeline, automatically annotating points where drift crossed the tolerance.

### Why this advances developer experience

Time Machine turns "this KPI seems wrong" from an hours-long debugging slog into a two-line notebook query. It's reusable with zero setup beyond editing one config cell, and it composes naturally with existing Fabric artifacts — Data Pipelines can schedule captures, Git integration can trigger pre-deploy snapshots, and the Delta tables are queryable by any downstream analytics surface.

### Stack

- `semantic-link` (`sempy`) — live model introspection and measure evaluation
- `se