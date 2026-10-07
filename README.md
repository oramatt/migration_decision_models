# Database Migration Decision Models

An OraMatt calculator and spreadsheet for comparing **offline database migration** with **online migration using change data capture (CDC)**.

Start with a simple question: can the full data copy fit within the allowed interruption? Then evaluate online migration, recovery time, acceptable data loss, and the evidence needed to support a decision.

The models help identify a suitable migration approach. Selecting a specific tool also requires checking its support for the source database, target database, data types, and application requirements.

## Files

| File | Purpose |
| --- | --- |
| [index.html](index.html) | Self-contained browser calculator with Simple and Advanced models. |
| [OraMatt_Migration_Decision_Models.xlsx](OraMatt_Migration_Decision_Models.xlsx) | Companion spreadsheet for working through the decision models. |
| [LICENSE](LICENSE) | Universal Permissive License (UPL), Version 1.0. |

## Run the calculator

Download `index.html` and open it in a modern browser. No installation, server, build process, or internet connection is needed for the calculations.

1. Choose **Simple model** or **Advanced model**.
2. Enter your business requirements and processing rates.
3. Review the result and the checks supporting it.
4. Use **Load example** to restore the illustrative values, **Clear inputs** to start again, or **Print results** to print or save a PDF through your browser.

The calculator runs entirely in your browser. It does not send or save your inputs, use analytics, or load external JavaScript, stylesheets, or fonts. Its explanatory links open external websites when clicked.

## Shared inputs

| Input | Meaning |
| --- | --- |
| Preferred approach | Offline, online, or no preference. A preference does not override the checks. |
| Data to copy | The amount of data being migrated, in gigabytes (GB). |
| Maximum cutover interruption | Minutes from stopping source writes until the application is available on the target. |
| Maximum rollback interruption | Minutes from starting rollback until the application is available on the source again. |
| Network round-trip delay | Network latency in milliseconds (ms), used as a condition for performance testing. |
| Planning factor | An allowance for uncertainty. A factor of `1.25` adds 25% to the estimated interruption. |

Use consistent units: one GB is one billion bytes, and one terabyte is 1,000 GB. All processing rates are in GB per minute; time budgets are in minutes.

**Data volume and latency alone cannot determine migration duration.** Measure the complete processing rate under the expected network conditions. The calculator does not convert latency into throughput or apply an automatic latency penalty.

## Simple model

The Simple model screens whether offline migration could fit within the cutover budget. It adds two assumptions:

| Input | Meaning |
| --- | --- |
| Assumed bulk speed | The complete rate of reading source data, transferring it, and loading it into the target. |
| Other cutover work | Minutes spent stopping writes, checking readiness, and switching application connections during the interruption. |

```text
Full-copy time = Data to copy / Assumed bulk speed

Offline interruption = (Full-copy time + Other cutover work) Ã— Planning factor

Time available for copying = Cutover budget / Planning factor - Other cutover work

Required bulk speed = Data to copy / Time available for copying
```

The required bulk speed is calculated only when the time available for copying is positive.

| Result | Meaning |
| --- | --- |
| Offline candidate | Estimated offline interruption fits the budget. Demonstrate the assumed speed and verify recovery before selecting a tool. |
| Evaluate online | Estimated offline interruption exceeds the budget. Use the Advanced model to assess online migration, or revise the plan. |
| Revise cutover plan | Other cutover work leaves no time for copying after applying the planning factor. |
| Unknown | Required inputs are missing or invalid. |

The Simple model does not establish online feasibility or assess rollback, even though the rollback budget is entered as a shared requirement.

## Advanced model

The Advanced model evaluates both approaches and their separate return-to-source plans.

### Cutover and CDC inputs

CDC captures changes made on the source and applies them to the target while the source remains in use.

| Input | Meaning |
| --- | --- |
| Bulk speed | Complete copying speed supported by a representative migration test. |
| Source change rate | Change data generated per minute during representative busy periods. |
| CDC processing capacity | Complete capture, transfer, and apply capacity for comparable changes. |
| Remaining source changes | The full unapplied backlog at the final source write freeze, including changes not yet captured. |
| Other offline or online cutover work | Work during the interruption outside copying data or applying the remaining changes. |
| Capabilities confirmed | Whether required copying or CDC coverage, retention, and application readiness have been verified. |
| Performance evidence | Whether the relevant rates are assumed, measured, or unknown. |

```text
Offline interruption = (Data to copy / Bulk speed + Other offline work) Ã— Planning factor

CDC spare capacity = CDC processing capacity - Source change rate

Online interruption = (Remaining source changes / CDC processing capacity + Other online work) Ã— Planning factor

Catch-up time while source writes continue = Remaining source changes / CDC spare capacity
```

CDC capacity must exceed the source change rate. Catch-up time is available only when spare capacity is positive and assumes steady rates. At the final write freeze, source writes stop, so the final backlog uses the full CDC capacity.

The online initial copy takes place before the final interruption. Its duration, change-log retention, and storage requirements must be evaluated separately.

### Rollback inputs

Once the target accepts writes, it may contain new data that the source does not have. Enter a separate recovery plan for offline and online migration.

| Input | Meaning |
| --- | --- |
| Target changes to restore | Data that must be preserved in the source when returning to it. |
| Restore speed | Complete rate of restoring or reconciling that data with the source. |
| Other rollback work | Time to stop target writes, check source readiness, and switch back. |
| Recovery method confirmed | Whether a compatible, tested return-to-source procedure exists. |
| Maximum allowed data loss | Minutes of recent committed activity that may be lost. Zero means no loss. This is the recovery point objective (RPO). |
| Expected data loss | Estimated loss from the recovery procedure, expressed in the same units. |
| Required rollback coverage | How long after cutover the return-to-source procedure must remain available. |
| Demonstrated rollback coverage | How long the tested recovery procedure remains available. |

```text
Rollback interruption = (Target changes to restore / Restore speed + Other rollback work) Ã— Planning factor

Expected data loss â‰¤ Maximum allowed data loss

Demonstrated rollback coverage â‰¥ Required rollback coverage
```

When there are no target changes to restore, restoration time is zero and restore speed is not required. Rollback coverage describes how long recovery remains possible; it is separate from how long recovery takes.

### Results

| Status | Meaning |
| --- | --- |
| Feasible | Required timing, recovery, and capability checks pass, and the relevant processing rates are confirmed as measured. |
| Not feasible | At least one requirement fails under the entered inputs. |
| Unknown | No known failure establishes rejection, but inputs, confirmation, or performance evidence remain incomplete. |

Known failures take priority over unknown checks. Assumed performance alone does not produce an overall **Feasible** result.

| Combined result | Meaning |
| --- | --- |
| Choose offline | Offline is feasible and online is not feasible. |
| Choose online | Online is feasible and offline is not feasible. |
| Compare both approaches | Both are feasible; compare tool support, cost, complexity, and operational effort. |
| Revise migration plan | Both approaches are not feasible under the inputs. |
| Resolve unknown checks | At least one approach still needs evidence or valid inputs before completing the comparison. |

## Hosted on GitHub Pages

The project address is:

**https://oramatt.github.io/migration_decision_models/**

## Assumptions and interpretation

- Example values illustrate the calculations; they are not vendor benchmarks.
- Measure the entire migration pipeline. For sequential export, transfer, and import, use a bulk speed that accounts for their combined duration.
- Use comparable units for source changes, CDC capacity, backlog, and recovery volume.
- Include required checks and application switching in the interruption estimates. Complete schema conversion and other preparation before cutover when the plan assumes this.
- The models address migration cutover and rollback. Ongoing replication also needs a target freshness or lag requirement.
- Decisions use full numerical precision. Displayed values are rounded.
- The JavaScript and formulas are visible to users because calculations run in the browser. Encoding or obfuscation would not keep the algorithm secret.

## Contributing

Open an issue or pull request with the proposed change, representative inputs, and expected result. When changing the algorithm, keep the calculator, spreadsheet, formulas, and plain-language explanations aligned. Check missing inputs, boundary values, CDC capacity, and recovery requirements before proposing a change.

## About and license

Created for the database migration discussions on [OraMatt.com](https://oramatt.com/).

Licensed under the **Universal Permissive License (UPL), Version 1.0**. See [LICENSE](LICENSE) for the full terms and copyright notice.
