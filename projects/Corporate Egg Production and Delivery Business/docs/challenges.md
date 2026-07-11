# Corporate Egg Production and Delivery Business — Engineering Challenges

A selection of the harder problems encountered while building this system and how they were solved.

---

## 1. Designing the Twin Reconciliation Checkpoints

**The Problem.** In a high-volume poultry business, discrepancies can happen at multiple points: during farm-to-warehouse delivery (upstream) or warehouse sorting-to-ending counts (downstream). Finding a leak requires isolating these two points so they don't muddy each other's audit math.

**The Solution.** The system splits validation into two separate checkpoints:
- **Checkpoint A (Upstream)**: Compares the total farm delivery count against the categorized warehouse total (including cracked and disposed eggs). If they don't balance, the system instantly flags a delivery count variance.
- **Checkpoint B (Downstream)**: Tracks sorting yields against mortality logs and inventory counts to identify downstream wastage or stock losses.
By separating these checks, the owner can immediately pinpoint whether an discrepancy happened during transportation or during warehouse handling.

---

## 2. Rural-Internet Resilience (Save-State Form Pattern)

**The Problem.** The system is operated in agricultural and warehouse locations where internet connectivity is frequently unstable or slow. Standard AJAX/Inertia requests can hang, fail silently, or cause duplicate submissions if the user clicks "Save" multiple times out of frustration, resulting in corrupted double-entries.

**The Solution.** We established a reusable **Save-State form pattern** at the core of all user input components:
1. Upon submission, the form immediately locks the input fields and enters a `Saving...` state.
2. The submit button is disabled and replaced by a loading skeleton.
3. If the server response returns successfully, the state changes to `Saved` with a green indicator badge that fades out.
4. If the request times out or encounters a connection error, the form changes to `Failed` with a red alert and enables a "Retry" button, preserving all user inputs so they can attempt to resubmit once connection is restored.

---

## 3. Unit Conversion Integrity (Operational Trays vs. Accounting Eggs)

**The Problem.** Farm and warehouse staff handle eggs in physical trays (usually 30 eggs per tray). Logging individual loose eggs would be tedious and error-prone. However, cracked and disposed losses are often counted as individual eggs. Performing all math on trays would lead to rounding errors, while forcing users to input in eggs would ruin the data entry flow.

**The Solution.** The database stores all counts and values at the egg level (pieces) as integers, but the presentation layer handles inputs entirely in physical trays. The system bridges these layers using a configurable `eggs_per_tray` constant (default 30).
To prevent historic records from shifting if the `eggs_per_tray` constant is edited in the future, the system snapshots the active `eggs_per_tray` value onto each `daily_receivings` and `categorizations` record at save time, keeping historical calculation logs completely immutable.

---

## 4. Frozen Flag Snapshot Engine

**The Problem.** Flags are raised dynamically when discrepancies exceed settings thresholds (e.g., a disposal rate above 5% or a count variance greater than 1 tray). If the owner updates these threshold settings in the future, re-rendering old flags would cause their calculated metrics to shift, corrupting the historical audit logs.

**The Solution.** We built a **Frozen Flag engine** where flags are generated server-side. When a flag is triggered, its severity, type, and impact quantity are captured. Crucially, the exact calculations, variables, and settings thresholds in play are frozen inside a JSON `calculation_snapshot` column. When the owner reviews a historical flag, the dashboard renders it using the frozen snapshot rather than live settings, preserving audit integrity.

---

<sub>© 2026 Mark Anthony Resoso · [Back to project](../README.md)</sub>
