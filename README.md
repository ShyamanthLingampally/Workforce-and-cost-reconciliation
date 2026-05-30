# Workforce Visibility and Labor Cost Reconciliation

An interview case study for a Strategic Operations role. The brief asked for a weekly metrics framework across three disconnected labor tracking systems at a commercial aviation maintenance operation. The analysis reframes the problem from data reconciliation into operational cost control.

*Note: the dataset is synthetic. Names and dates were fabricated for the exercise. Company-specific references have been removed.*

## Table of Contents

* [The Problem](#the-problem)
* [Data Sources](#data-sources)
* [Approach](#approach)
* [Key Findings](#key-findings)
* [Recommendations](#recommendations)
* [Contact](#contact)

## The Problem

Labor is tracked across three systems that do not talk to each other. Connecteam records shift-level activity. ADP runs full-time payroll. Aerotek runs contractor payroll. The team needed a weekly methodology to summarize across the three, flag errors, and define metrics to trend on. The real problem underneath was that 83% of paid hours had no operational record, making every efficiency or cost metric the organization reported unreliable.

## Data Sources

Three source files covering the week ending February 7, during which 162 aircraft were repaired.

* **Connecteam:** 338 shift records, 25 technicians. Job type, start and end times, shift hours.
* **ADP (FTE payroll):** 67 pay entries, 11 full-time employees. Actual hourly rates $34 to $64.50.
* **Aerotek (contractor payroll):** 149 contractor pay entries. Hours only, no rate field.

Contractor cost is modeled at $55 per hour fully loaded (industry mid-range) since Aerotek does not provide rates. At any reasonable rate between $40 and $65 per hour, the order-of-magnitude finding holds.

## Approach

The brief asked for a metrics framework. The first real decision was whether to build one on top of the existing data or stop and ask whether the data itself could support trustworthy metrics. The three-system comparison at the row-count level answered that quickly: Connecteam had 25 technicians on file, ADP plus Aerotek had 160 between them. A metrics framework built on the intersection would describe 17% of the workforce. That reframed the assignment from "build metrics" to "fix the foundation, then build metrics."

I chose technician-level reconciliation as the unit of analysis. Supervisors manage people, payroll pays people, activity tracking records what people do. Aggregating any higher than the individual hides the accountability trail. Aggregating any lower loses the ability to join across systems. Technician-level is the only grain where all three systems can be compared without losing information.

Joining across the three systems meant committing to a name-based match rather than an ID-based one, because no shared ID existed. That introduces a risk of miss-matches on name variants, so I normalized casing and format upfront and documented the approach rather than hiding it. Any reconciliation pipeline built on this pattern will need to handle name collisions explicitly as the dataset scales.

Flag classification was designed around what an ops team can actually act on, not around statistical purity. "Paid but no tracking" maps to a supervisor conversation. "Orphan activity" maps to a payroll or HR ticket. "Hour mismatch over 2 hours" maps to a policy review. "OK" requires no action. The two-hour threshold on mismatches was a deliberate choice to filter out rounding and pick up genuine policy drift.

Contractor cost was the hardest call. Aerotek gives hours but no rates, so any dollar figure here is a modeled estimate. Rather than hide that, I flagged the assumption ($55/hr fully loaded, industry mid-range) explicitly and ran the math at $40/hr as a sensitivity check. The unaccounted spend at the lower bound is still over $12 million per year. The order of magnitude holds, and that is what the recommendations ride on.

## Key Findings

Out of 6,746 paid labor hours across 166 technicians, only 1,145 hours (17%) had matching activity tracking. The remaining 5,600 hours were paid but untracked.

In dollars, total weekly labor spend was roughly $362,500. Only about $44,900 was backed by activity data. The remaining **$317,600 per week was unaccounted, which works out to approximately $16 million per year in labor spend with no operational record behind it**.

The reported efficiency metric of 5.15 hours per aircraft at roughly $380 per aircraft in cost was built on only the tracked 17%. The payroll-based reality is 41.6 hours and $2,238 per aircraft, an 8x gap on both measures.

The root cause is concentrated in one place. 138 of 149 Aerotek contractors had zero Connecteam activity for the week. Since contractors account for roughly $341,000 of the $362,500 weekly spend, the contractor tracking gap drives nearly all of the unaccounted cost.

**Flag breakdown:**

* Paid but no tracking: 138 technicians, ~$313,600 exposure
* Tracked but no payroll (orphan): 4 technicians
* Hour mismatch over 2 hours: 5 technicians, ~$4,000 exposure
* OK: 16 technicians reconcile cleanly

Closing the paid-but-no-tracking flag alone resolves nearly the entire cost control problem.

## Recommendations

**1. Gate contractor payroll approval on tracking submission.** No Connecteam record, no paycheck. This is a single policy change, owned by the contractor supervisor chain, that closes the majority of the visibility gap without any system investment.

**2. Automate the weekly reconciliation pipeline.** A scheduled Python job should pull the three feeds, join on normalized names, and generate the flagged CSV every Monday. Tickets auto-create from the output so the sprint starts with work already queued up.

**3. Run the governance work as a recurring two-week sprint, not a one-time project.** Each flag type maps to a Jira epic with a single owner and a definition of done. Visibility percentage and unaccounted dollars are reported every sprint review. Once you're past the first month, this becomes the operating model rather than a cleanup effort.

**4. Dual-report productivity metrics with visibility attached.** Publish both the tracked-only and payroll-based hours-per-aircraft figures alongside the visibility percentage. As coverage improves, the gap between the two numbers should shrink, and that trend becomes the measure of whether the program is working.

**5. Write down the implicit definitions.** A one-page data dictionary covering what counts as tracked, variance thresholds, break rules, overtime handling, and rounding resolves most of the recurring hour mismatches at the source rather than one ticket at a time.

## Contact

For more information, please contact:

**Email:** [lshyamanth@gmail.com](mailto:lshyamanth@gmail.com)

**LinkedIn:** [linkedin.com/in/shyamanth19a02](https://linkedin.com/in/shyamanth19a02)
