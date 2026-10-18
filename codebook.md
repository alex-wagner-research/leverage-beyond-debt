# Codebook

The rules the data follow and every decision behind them, each with its reason. The data are the
eight tables in `data/core/`, of which this release holds five; everything else is generated
from them by the project's build and checked by its checks. Section 1 describes the tables column by column, Section 2
the conventions they are coded under, Section 3 the rulings of the verification rounds and audits,
condensed, and Section 4 the decisions of the clean-up. A decision keeps the id it was taken under
(R3-A2, R4-L0, AUDIT6-D1, R7-D2, ...; CU-1, CU-2, ...), so a commit message, a changelog line or a
note that cites one can be followed here.

## 1. The tables

The tables are UTF-8 CSV files with CRLF line ends, in the form Python's csv module writes (a field
is quoted only where it must be); CHK-SCHEMA asserts the form. They are edited with the csv module,
not a text editor, so that a comma typed into a note cannot split a row. An empty cell means not
applicable or not disclosed, never zero. A row keeps its id when its content changes; a ladder
component that becomes an edge takes its contract's id (CU-13).

| Table | One row per | Key |
|---|---|---|
| `entities.csv` | legal entity, anonymized counterparty or role hub | `entity_id` |
| `contracts.csv` | arrangement, as the filings describe it | `contract_id` |
| `instruments.csv` | economic leg of an arrangement (an obligation, guarantee, backstop, stake, prepayment or loan), or a ladder component that is not an edge | `instrument_id` |
| `amounts_by_filing.csv` | amount of a dated instrument in one periodic filing | `instrument_id`, `bound`, `filing_date` |
| `firm_periods.csv` | sample firm at its ladder balance-sheet date | `firm` |
| `facts.csv` | filing fact that is not an instrument's amount, a sum of such facts, or a record of the work | `fact_id` |
| `evidence.csv` | quotation behind a coded field | `evidence_id` |
| `packages.csv` | support package: every support leg between one supporter and one supported party (2.8) | `package_id` |

### entities.csv

- `entity_id`: `E-<TICKER>` for a filer, `E-<NAME>` for a named party, `E-ANON-...` for a
 counterparty the filings describe without naming it, or for a role hub that stands for a group
 (a firm's landlords, its customers, its lenders).
- `name_canonical`, `name_in_filings`: the name used here, and the names the filings use.
- `entity_type`: `public_filer`, `private_co`, `financial`, `spv`, `jv`, `group`, `utility`,
 `anonymized`.
- `parent_entity_id`: the parent's `entity_id`, or empty (CU-1).
- `lei`, `lei_confidence`, `gleif_legal_name`: the GLEIF legal entity identifier, how firmly it
 matches (`high`, `high (other-name verified)`, `none`), and GLEIF's legal name.
- `jurisdiction`: ISO 3166 code, with the subdivision where known (`US-DE`).
- `resolution_basis`: how the entity is identified: `gleif`, `ex21` (the filer's list of
 subsidiaries), `filing`, `external` (an attribution from outside the filings, stated in the note
 and never used as a filing fact), `anonymized`.
- `notes`.

### contracts.csv

One row per arrangement, without amount columns: the
amounts are on the arrangement's instruments and in facts (CU-9). The text cells describe the
arrangement in the filing's terms and may quote figures; a figure is data only where an instrument
or a fact carries it.

- `contract_id`: `HL-nnnn`.
- `source_accession`, `source_doc`: the filings (SEC accession numbers) and documents the row is
 coded from, `;`-separated.
- `announcement_date`, `effective_date`: the first public date, and the start as the filing states
 it (a date or a description).
- `obligor`, `beneficiary`, `financier`, `guarantor`, `indemnitor`: the parties in the filing's
 words; the entity ids are on the instruments.
- `asset`: the campus, project or equipment pool the arrangement concerns.
- `instrument_type`: the arrangement's legal form.
- `maturity`, `termination_right`, `recourse_type`, `collateral_type`: as the filing states them;
 `n/a` where the form has none, `not disclosed` where the filing is silent.
- `purchase_link`, `revenue_link`, `capacity_link`: whether the arrangement comes with a purchase
 from the supporter, revenue to the supporter from the supported party, or a capacity reservation
 or take-or-pay (`yes`, `no`, `partially`, `not disclosed`, with the filing's qualifier where it
 gives one).
- `source_quality`: the kind of source (a filed 10-K, 10-Q or 8-K and its exhibits, a furnished
 press release, the press).
- `confidence_1to5`: the coder's confidence in the row, 5 the highest; the row's edges carry it as
 `source_quality_1to5`.
- `notes`.

### instruments.csv

A row is a network edge when `edge_id` is set, a ladder component when `ladder_firm` is set, and
often both. It fills the coverage, clause and dating columns when it is in the naked-position audit
(2.4), the clause table (2.5) or the dated guarantee layer (2.6). One instrument is one row, with its
amount in each bound on the row (CU-10). A ladder component that is not an edge has an id
`LD-<TICKER>-nn`; every other id is its contract's id with a letter.

The row and its edge:

- `instrument_id`; `edge_id`: the same id when the row is an edge, else empty.
- `contract_id`: the contract row. `HL-0034;HL-0047` marks the one arrangement both parties'
 filings describe, coded once: Meta's contingent purchase of Nebius's unsold capacity (CU-5).
- `from_entity`, `to_entity`: the node that owes, and the claimholder (2.3).
- `edge_type`, `layer`, `notional_type`: the kind of claim, its layer and what the amount measures
 (2.3).
- `instrument`: the leg in words.
- `notional_usd_bn`: the disclosed amount (2.1); for a component with no edge, the component's
 disclosed amount.
- `maturity`; `contingent`: 1 when payment is due only on a trigger, which `trigger_or_contingency`
 states.
- `wrong_way_flag`, `circularity_flag`, `source_quality_1to5` (2.3); `notes`.

The ladder (2.2):

- `ladder_firm`, `ladder_rung` (`L2` or `L3`), `ladder_component` (the component's name).
- `rule`, `ladder_class`: the classification rule and the class that follows from it.
- `lower_bn`, `gross_bn`, `upper_bn`: the amount the row adds to its rung in each bound; empty where
 its rule leaves it out of that bound.
- `subsequent_event`: 1 when the component was signed after the balance-sheet date.
- `ladder_basis`: the filing language the rule rests on; `ladder_source`: accession | file | lines.

The naked-position audit (2.4): `protection_holder` and `reference_entity` (entity ids; several
reference entities `;`-separated), `underlying_edges` (the claim edges that cover the position),
`coverage_class`, `coverage_note`.

The clause table (2.5): `clause_id` (`SC-nn`), `writer_role`, `clause_contract_id`,
`clause_instrument`, `clause_trigger`, `trigger_type`, the four rights `assumption`, `purchase`,
`sublease` and `sale`, `other_remedy`, `exclusivity`, `carrying`, `clause_note`,
`clause_verified_round`.

The dated guarantee layer (2.6): `date_is_upper_bound`, `date_confidence`, `robustness_flag`,
`disclosure_filing`, `date_note`, `amount_path_note`, `dating_verified_round`, `firmed_round`.

The stress test's reading of a ladder component: `ai_class`, the share of an L2 or L3
component that the demand shock idles, `ai` where the filing's words or the counterparty tie it
to AI infrastructure, `other` where they tie it to another business, `unallocated` where the
filing gives a company-wide total (R10 (e)); empty on a row that is not a ladder component.

### amounts_by_filing.csv

- `instrument_id`.
- `bound`: empty for the amount the ladder carries (the gross amount, or the upper where the row has
 no gross); `upper` for an upper amount that differs from it (Equinix's cap, CU-10).
- `filing_date`, `accession`: the periodic filing (10-Q, 10-K, 20-F or 6-K) and its SEC filing date.
- `amount_bn`; `status`: `disclosed`, or `absent` when the filing no longer states the instrument
 (the amount is then empty).

### firm_periods.csv

- `firm` (ticker); `period_end`: the balance-sheet date of the filing the ladder reads
 (`ladder_accession`); `total_assets_bn`.
- `debt_carrying_bn`: L0, debt at carrying value (R4-L0), with `debt_basis`, `debt_note` (the lines,
 and the face or principal amount beside the carrying value) and `debt_status`.
- `op_lease_liab_bn`, `fin_lease_liab_bn`, `other_l1_bn`, each with its `_basis`: the rest of L1
 (CU-7).
- `revenue_bn`, `revenue_basis`, `revenue_source`: annualized revenue (R3-B); `rpo_bn`, `rpo_basis`,
 `rpo_source`: remaining performance obligations, the company total (AUDIT6-D6);
 `denominators_verification`: where the two were checked. A source is accession | file | lines.
- `ai_stake_formula`: the instrument and fact ids whose amounts sum to the firm's AI-sector stakes
 (CU-8); `ai_stake_components`, `ai_stake_cost_bn`, `ai_stake_source`, `ai_stake_note`: the
 holdings' names, their cost where disclosed, the citation, and the judgment behind the selection
 (2.7).
- `disclosure_gap`: where the filing leaves L1 or L2 open; Figure 2 stars these firms (CU-7).

### facts.csv

- `fact_id`; `value` in `unit` (`bn` for USD billions, `pct`, `gw`, `years`, `m_shares`, `fy` for a
 fiscal year, and `rows`, `items` or `filings` for records).
- `kind`: `stated`, a figure printed in a filing (`printed`, `scale`, the citation and `quote`; the
 value is the printed figure times the scale, 0.001 for millions); `derived`, this project's
 arithmetic over other facts (`formula`); `link`, a relation between counterparties that is not a
 contract (CL-0001, ruling R6b-J5); `record`, a count of this project's own work (the verification
 rounds, the filings the first reading covered), which the paper reports.
- `accession`, `file`, `lines`, `quote`, `status` (`filed` or `furnished`; `record` on a row that records the work rather than a filing); `note`.
- `restates`: the instrument whose amount the fact states again; CHK-SCHEMA asserts that the two are
 equal (CU-3).
- Payment schedules by year (engine spec amendment A14, 2026-10-12): one stated fact per cell of
 each firm's latest annual report, `<firm>_<key>` with keys `oplease_due_y1..y5|after`,
 `oplease_total`, `oplease_interest` (finance leases `finlease_*`; Amazon's and Rackspace's
 financing obligations `finoblig_*`; Akamai's three lease classes `oplease_realestate_*`,
 `oplease_colocation_*`, `oplease_sublease_*`), `lease_nyc_total` with `lease_nyc_first_fy`,
 `lease_nyc_last_fy`, `lease_nyc_term_min_y`, `lease_nyc_term_max_y` (leases not yet commenced;
 Amazon's by year `lease_nyc_due_*`), `commit_due_y1..|after` and `commit_total` (purchase
 commitments; `othercommit_*`, `construct_*` where the filing separates them; `_y2to3`, `_y4to5`
 where it buckets), `rpo_total` with `rpo_within12m_pct`, `rpo_within24m_pct`, `rpo_duration_y`
 (`deferred_rev_*` where the filing gives deferred revenue instead), and `contracted_rev_due_*`
 (receipts due to the data-center owners and the lessors). Year 1 is the first fiscal year after
 the annual balance-sheet date, as for the debt schedules (A10). A number the filing writes in
 words carries its numeric `value` beside the printed word; a cell printed as a dash is a zero
 printed "-"; a firm whose filing gives no schedule (Arista) has its total and the part due within
 a year; where a filing gives only a total, the engine spreads it straight-line over the parsed
 term (class default). The schedule facts were typed from the filings' tables, each with its line,
 and every quote is located at its line by the checks.

### evidence.csv

- `evidence_id` (`EV-nnnn`).
- `table`, `row_id`, `field`: the row (`;`-separated where one quote supports several) and field
 the quote supports; `table` is empty for a passage that supports the database as a whole, such as
 a notice of redactions.
- `accession`, `file`, `lines`, `quote` (2.1); `note`.

### Generated files

the project's code writes, from these tables alone: `data/derived/test2_L0L3_bounds.csv` (the ladder's
cells), `edges.csv` (the network), `ssb_coverage.csv` (the naked-position audit),
`guarantee_timeline.csv` and `guarantee_timeline_path.csv` (the dated guarantee layer),
`asset_side.csv` (the stakes), `support_clauses.csv` and `tab_clauses.tex` (the clause table),
`fig02_panel_counts.json`, `paper_macros.tex` (the paper's numbers), and the figures in
`output/figures/`; where the paper's own folder is present (it is not part of the public
repository) it also copies the macros, the table bodies and the figures there. `verified_rows.csv`
is a committed input: the contract rows each verification round covered, written from the round
records. `test2_L0L3_bounds.csv` keeps the name of the second test of the first build. No generated
file is edited by hand; CHK-BUILD fails if the committed files differ from a fresh build.

## 2. Conventions

### 2.1 Amounts, dates and citations

- An amount is in USD billions and is the figure the filing prints, with all its digits (CU-3): a
 figure printed in millions is divided by 1,000, so 194,060 is 194.06. An amount the filing does
 not print as one figure says how it is derived (Nebius's energy commitments, ruling R3-A1).
- Dates are ISO 8601. A filing's date is its SEC filing date; `period_end` is a balance-sheet date.
- Accession numbers are dashed (`0001045810-26-000075`). A citation is accession | file | lines: the
 file is the cached text at `data/clean/edgar_text/<ticker>/<accession without dashes>/<file>`, and
 lines are counted from 1 in it.
- A quote is verbatim from the cited lines. Inside it, ` / ` marks a line break or a table-cell
 boundary, ` ... ` text left out, and square brackets a redaction marker or the quoter's completion
 of a cut word. CHK-QUOTES finds every quote at its lines, comparing both in one form: lower case,
 one kind of quotation mark and dash, footnote marks dropped, and the spacing the text conversion
 adds removed (`$ 329.1`, `12 %`, `( 24,590 )`).
- A value is stated once. A note or basis says what a value is and where it comes from, and does not
 restate a value another column holds (CU-4) or record the value's history, which is in git and the
 changelog (CU-11).
- The filings the database reads (`\ledFilingsN`) are the distinct accessions cited in
 `evidence.csv`, `firm_periods.csv` (`ladder_accession`) and `amounts_by_filing.csv`; a periodic
 report, the prior annual report it leaves a disclosure to (CU-7), and every current report or
 exhibit a dated amount or a quote cites each count once (2026-10-01; the record fact
 `extraction_filings` was the hand count for the first fourteen firms and is superseded).

### 2.2 The ladder

Each firm's obligations are stacked in four cumulative rungs, and L2 and L3 carry three bounds.

- **L0**: debt at carrying value, as the balance sheet or the debt note states it (R4-L0).
- **L1**: L0 plus operating and finance lease liabilities and other debt-like obligations, such as
 Amazon's financing obligations (CU-7).
- **L2**: L1 plus the firm's own commitments the balance sheet does not carry: leases signed but not
 commenced, purchase, capacity and construction commitments, funding commitments, and, at the upper
 bound, customer prepayments that are in-substance borrowing (XL).
- **L3**: L2 plus the support the firm wrote on another party's obligation or asset value:
 guarantees, backstops, residual value guarantees, and demand backstops (AUDIT6-D1, R7-D1).

The bounds:

- **Lower**: L1 plus the components whose filing language is firm at the balance-sheet date;
 subsequent events are left out whatever their language.
- **Gross**: everything signed by the filing date at its stated amount: firm commitments, contingent
 commitments (C1), subsequent events (SE1), guarantees not yet effective (C2) and the full
 total of a mixed-language disclosure (M1) (R7-D2).
- **Upper**: gross plus the quantified conditional maxima the filings disclose: pipelines, sized
 options, contractual maxima on variable arrangements, carve-out exposure, top-of-range values, and
 customer prepayments with a financing component.

The lower-to-gross wedge measures what the filer's language leaves unsplit; the gross-to-upper wedge
measures quantified conditional exposure. Each row's `rule` fixes the bounds it enters:

| Rule | Meaning | Lower | Gross | Upper |
|---|---|---|---|---|
| U1 | stated unconditional, non-cancelable, enforceable, take-or-pay, or fixed or guaranteed | yes | yes | yes |
| U1r | U1 disclosed as a range (CU-6) | bottom | midpoint | top |
| U1n | U1 of which customers reimburse a part | net | full | full |
| U2 | executed lease not yet commenced, with no filer termination right disclosed | yes | yes | yes |
| U3 | committed investment or funding obligation in force at period end | yes | yes | yes |
| U4 | demand backstop: the filer cannot cancel, and binds in full exactly when third-party demand is absent (wrong-way); in L3 (R7-D1) | yes | yes | yes |
| E1 | guarantee, backstop, residual value guarantee or standby instrument the firm wrote, in force, at its disclosed maximum exposure or current maximum payment; a contractual cap above that maximum is its upper amount (CU-6) | yes | yes | yes |
| M1 | mixed total with an unquantified cancelable or conditional part: the stated total | no | yes | yes |
| M2 | the unquantified remainder beyond a stated firm total | no | no | yes |
| C1 | contingent commitment (milestones, contingencies); a funding commitment, so in L2 | no | yes | yes |
| C2 | guarantee not yet effective: signed, or agreed as a subsequent event (R16) | no | yes | yes |
| C3 | pipeline subject to finalization, or an option at the firm's sole discretion | no | no | yes, if sized |
| C4 | contractual maximum on a variable arrangement; carve-out-contingent exposure | no | no | yes |
| SE1 | subsequent event, in any language | no | yes | yes |
| D1 | this project's split from the filing's arithmetic, shown on the row | no | no | yes |
| XL | customer prepayments the firm's own filing ties to a significant financing component (R6b-J2b) | no | no | yes |
| X1 to X6 | outside the ladder: nothing quantified disclosed (X1); unquantified (X2); a funding-side liability, such as supply-chain finance or customer prepayments without a financing component (X3); an intra-group guarantee of consolidated debt (X4); credit support a bank writes on the firm's own obligations (X5); a claim the firm holds rather than a promise it wrote (X6) | no | no | no |

`ladder_class` follows from the rule (CU-12): A for U1 to U4 and E1, and for a subsequent event in
firm language; B for M1; C for M2, C1 to C4, D1, and a subsequent event in conditional language; X
for X1 to X6 and XL, and for an option too unsized to enter any bound.

The writer caps (`\ledWriterCapBn<firm>`) sum the upper amounts of the guarantees a firm wrote (rules
E1 and C2), demand backstops excluded (R7-D1); `\ledWriterCurrentMaxBn<firm>` gives the gross amount
where it differs (AUDIT6-D3).

### 2.3 The network

- **Direction.** `from_entity` is the node whose balance sheet carries the obligation, or against
 which the claim runs; `to_entity` holds the claim. A lease runs lessee to lessor, a guarantee
 guarantor to beneficiary, a loan borrower to lender, a funded stake investee to investor, and an
 unfunded commitment investor to investee (`funding_commitment`). Every edge is a liability of its
 from-node.
- **`edge_type`**: `payment_obligation`, `revenue_commitment`, `funding_commitment`, `equity_claim`,
 `convertible_claim`, `warrant_claim`, `receivable_claim`, `prepayment_liability`, `guarantee`,
 `backstop`, `debt`, `concentration_disclosure`, `option_discretionary`, `ppa_commitment`.
- **`layer`**: `L_debt`, `L_lease`, `L_commitment`, `L_guarantee`, `L_equity`, `L_trade`, `L_prepay`.
- **`notional_type`**, what the amount measures: `committed`, `funded`, `carrying`, `carrying_bucket`
 (a portfolio bucket "primarily" one company), `cap`, `max_exposure`, `disclosed`, `estimated`,
 `announced` and `loi_nonbinding` (not commitments), `pipeline`, `unbounded` (no maximum), `nd` (not
 disclosed; the amount is empty), `pct_revenue`, `annual_revenue`, and `unsigned` (the size of an
 instrument whose direction the filing does not give; it enters no sum, AUDIT6-D10). Sums filter
 by notional type.
- **Scope.** Every ladder component in the gross cell is an edge; a component in the upper bound
 only is not (CU-14).
- **Amounts.** A guarantee or backstop is drawn at its maximum, the cap or maximum exposure; an
 obligation the filing gives as a range at its low end (CU-6).
- **`wrong_way_flag`** = 1 when the edge pays exactly in the states where the counterparty or the
 common sector state is impaired: OpenAI reimburses NVIDIA only once OpenAI has defaulted (HL-0003a),
 and Meta's backstop binds when Nebius cannot sell its capacity (HL-0034b). Guarantees of a
 counterparty's data-center debt carry it (R5-J5).
- **`circularity_flag`** = 1 is the coder's label that the parties are linked by a vendor-customer
 relationship the filings describe, whether or not that link is itself an edge. It cannot be
 recomputed from the edges, since many commercial links run to anonymous nodes; no script, figure
 or macro reads it (AUDIT6-D10).
- **`source_quality_1to5`** is the contract row's `confidence_1to5`.
- **The contracted-revenue books.** Five edges (HL-0086a to HL-0090a: Microsoft, Amazon, Alphabet,
 Broadcom, Nebius) carry firm-level backlogs from one anonymous customer node. They are not measured
 on a common basis (Microsoft's includes unearned revenue and states no exclusion of cancelable
 contracts; Broadcom's excludes contracts terminable for convenience; Alphabet's and Nebius's state
 their own exclusions), so a sum across them says so, and none carries a solvent-backed share
 (R6-J1). Figure 2 nets the named contracts out of the CoreWeave and Nebius books (AUDIT6-D4), and
 the Polaris Forge 1 leases out of CoreWeave's leases not yet commenced (CU-13); a netted arrow is a
 floor.

### 2.4 The naked-position audit

Every contingent guarantee, backstop or funding-commitment edge names its protection holder and
reference entity and carries a coverage class (R5-SSB):

- `covered_by_edges`: the protection holder holds coded claims on the reference entity
 (`underlying_edges`) that the protection covers;
- `indemnity_on_payment`: a reimbursement of amounts a guarantor has paid;
- `value_trigger`: pays a value shortfall, such as a residual value guarantee against its threshold;
- `demand_state`: pays when capacity is not sold to third parties;
- `milestone_funding`: a funding commitment that waits on milestones;
- `pending`: not yet finalized;
- `underlying_outside_network`: the protected claim exists but is not a coded edge.

A contingent edge with none of these would be naked; CHK-NAKED fails on one, and there are none.

### 2.5 The clause table

The rows are every guarantee-type instrument a sample firm wrote on another party's obligation or
asset value: the L3 rows under rules E1, C2 and U4, among them NVIDIA's AI cloud agreements and
CoreWeave's New Jersey venture guarantee (R7-A1). Parent guarantees of a firm's own consolidated
vehicles (X4), OpenAI's guaranty of its own tenant subsidiary's lease, options and pipelines,
Digital Realty's carve-out guarantees and Amazon's facility to Anthropic are not rows.

- `writer_role`, the writer's relation to the asset: `supplier`, `internal_use`, `customer`,
 `co_owner`, `tenant` (R7-A2).
- `trigger_type`: `exit` (pays on the user's own exit), `obligor_default`, `demand`, `not disclosed`,
 and, since 2026-10-12, `writer_nonperformance` (the right arises when a contract is terminated
 for the writer's own failure to perform: HPE's SC-15).
- `assumption`, `purchase`, `sublease`, `sale`, the writer's rights over the asset on failure:
 `yes`, `no`, `not disclosed`, or `not applicable` for the two demand backstops (R7-A3).
- Access is derived: `yes` if any right is `yes`, else `not applicable` if every right is, else `no`;
 a right not disclosed counts as no disclosed right (R7-A5, as corrected on 2026-10-12: the earlier
 order returned `not applicable` for Jabil's SC-16, whose purchase option is a right). The paper
 counts two things: writers with any disclosed right (`\ledWriterAnyRightN`) and writers whose
 right arises on the reference party's default (`\ledWriterRemedyN`, `trigger_type`
 `obligor_default`), which excludes a right on the writer's own exit or non-performance.
- `clause_trigger`, `other_remedy`, `exclusivity`, `carrying`: the trigger, the other remedies, any
 exclusivity and the carrying value, in the filing's terms. The table's maximum is the
 instrument's upper amount (CU-2).
- The two demand backstops (SC-09, SC-13) form their own block and do not count in the access macros
 (R7-A3). Prediction 1 compares support written by the asset's user that pays on its own exit
 (SC-07, SC-08) with every other failure-state row (R7-A4).

### 2.6 The dated guarantee layer

- Each L3 component in a bound is dated by the first filing that quantifies it, not by its legal
 effective date (R6-J6). `amounts_by_filing.csv` holds its amount in that filing and in every cached
 periodic report after it, or `absent` from the first report that no longer states it: no longer
 disclosed, not ended (AUDIT6-D8).
- An executed subsequent-event guarantee enters the series at the filing date of the report that
 discloses it (AUDIT6-D9).
- `date_is_upper_bound` = 1 when no earlier filing is cached, so the component may have been disclosed
 before; `robustness_flag` = 1 when the date is an upper bound or its confidence is below high, and a
 test that relies on such a component reports a robustness check (R6-J6).
- `disclosure_filing`, `date_note`, `amount_path_note`: the first filing, the reason for the date,
 and how each filing's amount was read (lines and arithmetic).
- `dating_verified_round`, `firmed_round`, `clause_verified_round`: the verification round that checked
 the dating, firmed an upper-bound date, or checked the clause coding (`ext`: the round of the
 sample extension, R8).

### 2.7 Denominators and stakes

- Revenue is the latest fiscal year's where the target report is an annual report, and otherwise
 annualized from the shorter period the report covers: twice the latest half year, four thirds of
 the latest nine months, or four times the latest quarter. `revenue_basis` opens with the factor
 (`2x`, `4/3`, `4x`), which is what the dagger in Table D.1 and Figure 2 reads; for a fast grower
 the annualization understates run-rate revenue, and Figure 2's caption says so (R3-B).
- RPO is the company total (Microsoft 684, Alphabet 519.5), while the network edges keep the segment
 books (678, 513.9) (AUDIT6-D6, R6-J2). Applied Digital's figure is its lessor minimum lease
 payments (`\ledLessorMinPaymentsBnAPLD`).
- A guarantee writer's equity value (Table 3) is the market value of all its common share classes on
 the last trading day on or before its balance-sheet date: price times shares outstanding of the listed
 class from the Compustat daily files on disk (`data/raw/wrds/compustat/`), plus the classes those files
 do not hold (Meta's class B, Alphabet's classes B and C) at the same price, with the counts from the
 cover page of the same report (`facts.csv`). Only the writers whose books pay in a modelled state are
 valued, since the other dual-class firms' unlisted classes are not coded (2026-09-27).
- A firm's AI-sector stakes are the sum its formula names (CU-8). Which holdings count is a judgment
 the note states (NVIDIA's stake in OpenAI, HL-0026a, is known only from the parties' announcements,
 which the contract row HL-0026 names in full; no NVIDIA filing names OpenAI, and the quarterly
 report's non-marketable securities line corroborates the order of magnitude, EV-0492; the stake
 enters the stress test's equity channel at the press amount by the author's decision of 2026-10-14): NVIDIA's baseline counts its neocloud stakes, and its OpenAI stake as the press
 reports it is a separate marker in Figure 4; Dell carries its non-marketable securities as a
 placeholder; AMD's unnamed investees are left out (R3-C).

### 2.8 The support packages (Table D.2)

`packages.csv` holds one row per package under ruling R7-B: every support leg between one named
supporter and one supported party, across contracts; the supported party may be unnamed where a filing
describes the legs. Columns: `package_id` (the id of the verification candidate it comes from),
`supporter_id` and `supported_id` (entities; `supported_description` where the party has no entity),
`direction` (`forward`: the supporter sells to the supported party; `reversed`: it buys from it;
`co_owner`; `not_stated`), `named` (`yes`, `no`, `inferred`), the legs `sale`, `cash`, `claim`,
`coverage`, `concession` and `counter` (a leg running back to the supporter: a reimbursement, an escrow,
the other partners' guarantees) and `other` (rows recorded but not support legs: a non-binding letter of
intent, an option, a pipeline), each a `;`-separated list of existing instrument or contract ids, whose
quotes are in `evidence.csv`; and `note`. No leg is typed twice: the amounts, quotes and source quality
are those of the rows the ids name.

- A package needs a commercial relation a filing states and at least one leg of cash, claim or coverage
 from the supporter, or a concession on the supporter's own stock granted to the supported party
 (R7-B as amended by R7-C; CHK-SCHEMA asserts the second). A warrant or share on a party's own stock is
 a concession, not a claim, and the table records whether the supporter granted or received it. A
 concession pays when the supported party buys, not if it fails, so it is support without exposure:
 the stress test has no channel for it, and the text says so where the table is discussed.
- Access is not typed: it is derived from the clause coding of the coverage rows (2.5). A leg is marked
 open in the table where any of its rows has source quality below 4 (R7-B).
- The 34 candidate packages gave 29 packages (2026-09-27):
 - P04 and P05 are one package, since both run from NVIDIA to the same unnamed AI cloud partners (the
 AI cloud agreements, and the land, power and shell guarantees).
 - P06 (NVIDIA and xAI's GPU vehicle) is out: the relation is stated only in the press.
 - P11 and P12 (AMD's warrants to OpenAI and Meta) and P29 (CoreWeave's shares to OpenAI with its
 contracts) were out under R7-B, because besides the sale the only leg is a concession. Ruling R7-C admits them: 32 packages, three of them concession-only, counted apart by
 `\ledPackagesConcessionOnlyN`.
 - The sale leg of P01 is the filed statement that OpenAI deploys NVIDIA systems at the PORTS campus
 (HL-0001), not the press-quality letter of intent (HL-0026b, recorded as other).

### 2.9 Which firms the main-text figures show

- A figure with every sample firm as a row is not readable. Figures 2 and 3
 show the firms whose contractual obligations at the gross bound ($L_3$) are at least the
 exhibit floor in `code/python/exhibit_rules.py` (`FIGURE_FLOOR_BN`, 25); the appendix repeats
 each figure with every firm (`fig1c_ladder_panels_all`, `fig4b_other_side_panels_all`). The
 floor is a design constant like the sample rule's size floors; the project's code prints it as
 `\ledFigureFloorBn` and the number of firms it admits as `\ledFigureFirmsN`, and the figure
 scripts import it from the same file. The tables of the appendix carry every firm.

## 3. Rulings of the verification rounds and audits

The data were frozen on 2026-09-27 (`data-freeze-2026-09-27`); every later change to a core table is a
ruling below with a stated freeze exception. Each ruling in one or two sentences, by topic; the id says when it was taken (R3 to R9: verification
rounds 3 to 9; R10 to R14: the rulings of the second revision and of the audit of the public
export; AUDIT and AUDIT6: the data audits of 2026-09-08 and 2026-09-25). The conventions
above apply them.

**The ladder.**
- R3-A1. Nebius's energy commitments (HL-0071) are one row carrying the sum of the six yearly figures
 the filing gives without a total, 5,324.1 million, as this project's arithmetic.
- R3-A2. Dell's supply-chain-finance payables (3.1) are a funding-side liability (X3), outside both
 bounds and in the network only: a bank finances the supplier against Dell's confirmed payable, the
 mirror of a customer prepayment.
- R4-L0. L0 is debt at carrying value for all fourteen firms, the balance-sheet quantity every filer
 discloses; ten cells had held face value, principal or gross maturities.
- R5-J1. Microsoft's construction commitments and its leases not yet commenced are additive: they
 come from different notes, and the filing gives no overlap. The paper states this as a convention.
- R6b-J2, R6b-J2b. Customer prepayments count at the upper bound (XL) when the firm's own filing ties
 them to a significant financing component, the firm booking imputed interest on them, and stay
 outside the ladder otherwise (X3): Nebius's 6.0 and Oracle's 4.6 are in; CoreWeave's 7.0,
 Broadcom's 14.2 and NVIDIA's 2.8 are out. R6b-J2 first applied the NVIDIA half only.
- AUDIT6-D7. CoreWeave's line on financing components is a general definition of interest expense,
 so its deferred revenue stays X3; Nebius's component is sized at 20-F L1169 and 6-K L1716, L1727.
- AUDIT-X4. Parent guarantees of a firm's own consolidated vehicles (CoreWeave's HL-0019b, HL-0037b,
 as AMD's and Applied Digital's) are X4: the underlying obligation is already drawn in full between
 the same parties. The edges stay in the network, flagged intra-group, outside the distinct
 guarantee total.
- AUDIT6-D1. L3 holds support the firm wrote on another party's obligation or asset value. Amazon's
 standby letters of credit, which banks write on Amazon's own obligations, leave L3 (X5); Digital
 Realty's pro-rata share of non-recourse venture debt is upper bound only (C4).
- AUDIT6-D2. Applied Digital's Base Electron loan and stake are a claim it holds, not a promise it
 wrote (X6), and leave L3 and the dated series.
- AUDIT6-D3. Equinix's $48m current maximum payment is in the lower and gross cells;
 `\ledWriterCapBnEQIX` carries the $229m cap and `\ledWriterCurrentMaxBnEQIX` the current maximum.
- AUDIT6-G, R7-D2. The gross cell counts everything signed by the filing date: contingent
 commitments (C1) and subsequent events (SE1) enter it, as signed guarantees not yet effective (C2)
 had. A funding commitment is the firm's own obligation, so Amazon's Anthropic facility sits in L2
 with every other C1 row. AUDIT6-G had kept the old membership until this rule.
- R7-D1. Demand backstops (U4) sit in L3, where NVIDIA's AI cloud agreements join Meta's contingent
 capacity purchase: the payment waits on a state of demand the writer does not control. Writer caps
 count guarantees only.
- AUDIT6-D11, R7-C. CoreWeave's New Jersey venture: its guarantee of up to $95m of milestone
 consideration (L3, E1) and its construction funding of up to $160m (L2, C1) are rows; the 1.7 is the
 two other ventures' commitments.

**The network.**
- AUDIT-M10. Meta's future funding commitment to the Louisiana Venture is 2.80, the filer's
 maximum-exposure arithmetic (46.03 less 2.92, 12.31 and 28.00); its leases not yet commenced are
 drawn net of the 12.31 Venture leaseback, which is its own edge.
- R4-J1. Delta Forge 1, Polaris Forge 3 and Delta Forge 2 share one anonymized tenant, on the
 filer's press release ("third long-term lease with the same ... hyperscaler").
- R4-J2. Where the filing names no lessor subsidiary, the edge runs to Applied Digital itself; no
 placeholder vehicle is created.
- R4-J3. Applied Digital's portfolio date and its Note 17 schedule are inconsistencies inside the
 filing; every row uses the per-campus figure of Item 1.
- R4-J4. `announcement_date` is the 8-K's date, with the execution date beside it in the note.
- R4-J5. The three Applied Digital lease 8-Ks are cached as text under their accessions (copies of
 the earlier design), so their citations resolve.
- R4-J6. The Polaris Forge 2 tenant's right of first refusal, stated only in a press release, is a
 note, not an edge.
- R5-J2. Equinix's AMER 2 loan, on which it lends to its own venture, is coded: the drawn balance as
 debt, the undrawn commitment as funding, and the partners' limited guarantees to Equinix.
- R5-J3. The EMEA 2 venture's facility is a debt edge at its size (notional type `cap`), so Equinix's
 guarantee covers a coded claim; the facility enters no ladder rung.
- R5-J4. The atNorth letter is two rows, the equity support and the capacity offtake, both flagged
 circular.
- R5-J5. Guarantees of a counterparty's data-center debt carry the wrong-way flag; parent guarantees
 of consolidated vehicles do not.
- R5-SSB. Every contingent guarantee, backstop or funding-commitment edge carries its coverage (2.4).
- R6-J1. The contracted-revenue books carry no solvent-backed share; where the engine needs one, it
 is a modelling assumption with its own sensitivity.
- R6-J2. Alphabet's book: headline 519.5 (the total backlog), committed 513.9 (Google Cloud).
- R6-J3. The 30-month expiry after an Anthropic liquidity event is HL-0044's termination right.
- R6-J5. Broadcom's lessee is unnamed in the filing (E-ANON-AVGO-LESSEE); the press reports conflict.
- R6-J7. The FT's "up to 25 per cent" residual value is attributed to the FT and used in no analysis.
- R6b-J1. An option or memorandum without a committed amount has an empty amount, not zero (HL-0091,
 HL-0006).
- R6b-J3. The Groq deferred consideration (HL-0098) is coded, outside the ladder, as no firm's
 deferred acquisition consideration is in it.
- R6b-J4. NVIDIA's extended payment terms (HL-0095) are coded without an amount.
- R6b-J5. NVIDIA's receivable concentration is a link between counterparties (CL-0001), not a
 contract.
- R6b-J6. The Level 2 warrants and preferred (HL-0099) note a possible overlap with the Nebius
 warrant; the stakes do not change.
- R6b-J7. The equity forward (HL-0094) is its own row: one instrument, one row.
- AUDIT6-D4. Figure 2's panel (a) draws the Nebius book net of its named Microsoft and Meta contracts,
 as it draws CoreWeave's; the netted arrows are floors.
- AUDIT6-D5. Figure 2's panel (b), the stress state, draws guarantees and backstops only; contingent
 funding commitments, whose milestones fail in that state, are reported on their own
 (`\ledContingentFundingBn`).
- AUDIT6-D10. The circularity flag is the coder's label (2.3); HL-0094a's notional type is `unsigned`.

**The dated guarantee layer.**
- R6-J6. A component is dated by the first filing that quantifies it (2.6).
- R6b-J8, AUDIT6-D8. Oracle's lessor guarantee is no longer disclosed from the 10-Q filed 2026-09-11,
 which precedes its stated maturity and reports no release: absent, not ended.
- AUDIT6-D9. Alphabet's April 2026 backstop of about $15.3bn enters the executed series on
 2026-04-30, and the pipeline point of that date falls by the same amount.

**Denominators and stakes.**
- R3-B. Revenue is annualized from the latest half year (2.7).
- R3-C. Dell's stakes are a placeholder of 2.4; AMD's unnamed portfolio is left out, since Dell's
 filing ties one investee's mark-up to AI and AMD's names none.
- AUDIT6-D6. The RPO macros carry company totals; the network keeps the segment books (2.7).

**The clause and package tables.**
- R7-A1 to R7-A5. The clause table's rows, the writer's relation to the asset, the demand backstops'
 block, Prediction 1's two groups and the writer counts, as in 2.5.
- R7-A6. The 13 clause rows were verified as coded.
- R7-B. A package is every support leg between one supporter and one supported party, across
 contracts. The supporter is named; the supported party may be unnamed where the filing describes
 the legs. A package needs a commercial relation a filing states and at least one leg of cash, a
 success claim, coverage or access from the supporter. Legs are sale, cash, claim, coverage and
 access; warrants or shares on a party's own stock are concessions, with their direction.
 `\ledPackagesBothSidesN` counts packages in which the supporter both sells to the supported party
 and pays it cash. Legs of source quality below 4, and packages whose direction the filings do not
 state, are in and marked.
- R7-C. A package also qualifies when its only support leg is a concession on the
 supporter's own stock granted to the supported party (warrants or shares issued to a customer with a
 purchase agreement). The supporter's shareholders pay for it, so it is support; it pays when the
 customer buys, not if it fails, so it carries no exposure and enters no loss channel. The table
 counts such packages apart (`\ledPackagesConcessionOnlyN`). Admits P11, P12 and P29.

**The rounds.**
- R6-J4. The ten NVIDIA items the first reading had found missing were verified, and the
 upper-bound disclosure dates firmed. Nebius, a foreign private issuer, files 20-F and 6-K reports (CIK
 1513845), on which its extraction rests.

- R8. The rows of the 23 firms added to the sample were verified by the author under a rule
 fixed before the check: every ladder row of
 the extension with a gross amount of at least \$5bn, every guarantee, backstop, funding
 commitment, stake or warrant, and every other row whose contract number ends in 0 or 5, plus
 the clause rows SC-15 and SC-16. Every row was confirmed as coded; the ladder-only `LD-` rows (class X,
 no counterparty, never summed) were confirmed as intended. The extension's firm-period
 denominators (assets, debt, revenue, RPO) were not part of this round.
- R9. The 23 firms' balance-sheet denominators were verified by the author (for each firm, total assets, reported debt, lease
 liabilities, annual revenue with its basis, and remaining performance obligations, each with the
 filing's lines). Every cell was confirmed; `denominators_verification` and `debt_status` in
 `firm_periods.csv` say so, and the record facts `verification_round_9_*` count 115 items.

- R10 (2026-10-12). (a) IBM's client financing
 commitments (HL-0116a, rule C1) sit in rung L2, where rule C1 puts every funding commitment; they
 had been placed in L3, which put IBM in the dated guarantee layer without a guarantee. (b) HPE's
 performance guarantees (SC-15) carry `trigger_type` `writer_nonperformance`. (c) The access
 derivation reads a disclosed right before the not-applicable marker (above). (d) Payment schedules
 by year for all 37 firms enter `facts.csv` (facts section); the freeze of 2026-09-27 is reopened
 for them, for (a) and (b), and for nothing else. (e) `ai_class` on every L2 and L3 ladder
 component (the project's code): `ai` where the filing's words or the counterparty tie the
 obligation to AI (a laboratory, an AI cloud, AI accelerators or racks, a pure AI cloud's own book);
 `other` where the filing ties it to a business that is not AI infrastructure (a foundry's fabs,
 a content-delivery network's co-location, IT service contracts, client financing lines);
 `unallocated` for a company-wide total or a data-center lease or construction commitment the
 filing does not tie to AI. The classification reads the component's description and
 counterparty, never the firm's name or role. (f) The fiscal-2022 baselines (`fy2022_*` facts,
 typed from the figure firms' annual reports for the last fiscal year that ended before 2023,
 cached by the project's code): the leases not yet commenced and the purchase
 commitments the firm then had, so that the engine's observable AI share of an `unallocated` book
 is the book's growth since then over the book today (amendment A19); a firm that disclosed no
 such leases that year has a baseline of zero for that component.

- R11 (2026-10-14). A contracted-revenue
 book that the filing states includes revenue already collected is carried, in the engine's
 revenue claim, net of the collected part the filing quantifies: Microsoft's commercial
 remaining performance obligations (HL-0086a, "includes unearned revenue") less the unearned
 revenue of its two commercial segments at the same date (facts `msft_unearned_rev_pbp_bn`,
 `msft_unearned_rev_ic_bn`, derived `msft_unearned_rev_commercial_bn`; the project's code,
 `COLLECTED_OF`). The ladder's revenue books (firm_periods.csv, `rpo_bn`) are unchanged: they
 report what the filing reports. Oracle's book excludes usage-based consideration and Amazon's
 is commitments not yet recognized, so nothing is netted there; the other books whose filings
 state a collected part are netted under R14. Freeze exception: three fact rows, no other
 table.
- R12 (2026-10-14; the field-level audit). A check of the 58 contract rows outside the author's
 verification rounds found
 seven filed contracts without an evidence row, ten evidence rows with an empty quote, one
 mistyped amount and one press-only amount in the engine. Applied by
 the project's code under a freeze exception: evidence rows added for
 every quantified instrument of HL-0029, HL-0033, HL-0035, HL-0037, HL-0038 and HL-0041 whose
 passage is in a cached filing; the empty quotes filled verbatim from the cited lines;
 HL-0041a (CoreWeave's Magnetar loan) from 0.23 to 0.189, the remaining obligation at June 30,
 2026 (the 10-K table says 273 at December 31, 2025; 0.23 matched neither); HL-0025a (a
 securitization known only from a press release not on file) from 0.768 to nd under the rule of
 2026-09-30, so the arrangement stays as a row and enters no amount. HL-0040 (Meta's order form
 with CoreWeave, $14.2bn) was cited to a 10-K whose text does not state the amount: the current
 report of September 30, 2025 does (line 54), fetched into the filing cache by
 the project's code with the two Applied Digital current reports on the Polaris Forge 1
 leases (June 2 and August 29, 2025), and the row now cites it. HL-0037b (CoreWeave's springing
 parent guaranties and letter of credit on the Polaris Forge 1 leases, cap 11.0) is in no filed
 document on file, including the filed Building 2 lease (exhibit 10.1 of the June 2 report,
 whose only guaranty is the landlord's): its cap is withdrawn (nd) and the row stays as an
 unquantified guarantee. A second pass by four independent readers, each reporting disagreements only, found: HL-0008a's amount withdrawn (a
 grid-investment pledge, not a PPA notional); HL-0040a at the latest stated commitment,
 $21.0bn (the March 2026 order form; the earlier $14.2bn form is no longer cited and the
 filings do not say whether it stands in addition, so the row is the latest amount, not a
 sum); HL-0037b at the $50m letter of credit the lessor's 10-K names beside the springing
 guaranty (the sentence above that no filed document states the guaranty was wrong); the
 warrants of HL-0037c held by their buyers; HL-0038e added as a subsequent event (the notes
 of June 16, 2026 refinanced the Bridge Facility inside the May 31 reported debt, so the
 engine's L0 reconciliation skips the edge: `POST_BALANCE_SHEET_DEBT`); two wording
 corrections. The record facts `independent_check_*` count the check.
- R13 (2026-10-14). Oracle is read at its latest periodic report, the quarterly report
 for August 31, 2026 (0001193125-26-389274, filed September 11, inside the freeze), as the
 sample rule says; the ladder had read its annual report for May 31. The project's code
 moves the period, the denominators (total assets, notes payable and other borrowings current
 plus non-current, the lease liabilities), the revenue (four times the quarter, R3-B), the
 remaining performance obligations (664, 13 percent within twelve months), the leases not yet
 commenced (288), the unconditional purchase obligations (34.150 under the filing's new
 description, with its schedule; the $19bn signed after May 31 sits inside the total and
 HL-0050a leaves the ladder), the lease maturity tables, and the quarter's prepayments with a
 financing component (11.363, the node's prepayment inflow). The lessor-borrowing guarantee
 (HL-0043b) is absent from the report before its stated maturity (R6b-J8): out of the lower
 bound, kept in gross and upper at its last stated amount. The debt maturity schedule stays
 with the annual report, which the quarterly report leaves it to (`disclosure_gap`, the
 asterisk). Node inputs refetched from XBRL at the new date (the refetch also re-cited
 Applied Digital's May 31 cash to the comparative column of its later 10-Q, same value).

- R14 (2026-10-14; the audit of the public export). Nine independent readings of the export
 against the filings, the code and the paper, reporting disagreements only, settled as follows
 (the project's code; every new amount cites its line and quotes it). (a) Schedules come
 from the latest annual report (A14): Coherent's and Super Micro's lease schedules are retyped
 from their fiscal-2026 reports, where the fiscal-2025 cells had stayed, and purchase-commitment
 schedules are typed where the filing gives one, for Coherent (3.4 in fiscal 2027, 8.4
 thereafter), Rackspace (by year to 2030), Lumentum and Seagate (within one year and after, with
 Seagate's equipment commitments as a second schedule); the fiscal-2025 totals of Coherent,
 Lumentum and Seagate are restated. A commitment whose maturity the filing states in words takes
 the window as its term where the filing says "primarily" or "generally" (Super Micro, Fabrinet)
 and the window over a share of one half where it says "a majority" (Intel, Penguin)
 (the project's code, `TERM_OVERRIDE`). (b) R11 applies to every revenue book whose filing
 says it includes unearned or deferred revenue and states the amount: Alphabet (total deferred
 revenue, which the filing says relates primarily to Google Cloud), Broadcom (contract
 liabilities), Micron (the part recognized as contract liabilities), and the ladder's
 deferred-revenue rows of Arista, Super Micro, HPE and Everpure (`COLLECTED_OF`); Super Micro's
 and HPE's books equal their deferred revenue and carry no claim. (c) Oracle's $19bn of
 cloud-infrastructure commitments signed after May 31 (HL-0050a) sit inside the quarterly
 report's total (HL-0101a, R13) and are netted from it in the engine as a named edge inside an
 aggregate (`LADDER_INSIDE`); they had been a claim twice. (d) CoreWeave's OEM financing
 (HL-0018a) is the outstanding balance the filing states (4.8), not the quarter's
 reclassification line (1.469). (e) Everpure's component-supply commitment, quantified at 1.1
 in the quarterly report for August 2, 2026, becomes the edge HL-0136b (CU-13): a contingent
 purchase obligation of the firm's own, triggered when its contract manufacturers do not buy,
 in the gross and upper cells of L2 (C1); it is not support written for another party, so it is
 not in L3. (f) Marvell's commitment column includes 220.0 of technology licence obligations the
 balance sheet carries: they move to `other_l1_bn` and the row HL-0112b is net of them. (g)
 Oracle's prepayments with a financing component (HL-0051a, XL, the upper bound only): the
 bound is the sum of the two stated inflows, 11.363 and 4.592, since no balance is stated. (h)
 Jabil's $80m of residual value guarantees on fiscal-2026 leases were in force at the
 balance-sheet date, so they are in the lower bound too. (i) Smaller cells: two maturities and
 one note filled from the filings (HL-0065a, HL-0072a, HL-0152a); CoreWeave's deferred revenue
 and Meta's take-or-pay at all their digits (CU-3); the warrant rows HL-0028a and HL-0029a carry
 no amount rather than zero; IBM's client financing commitments in the debt layer; Equinix's
 AMER 2 loan commitment is unconditional on the lender's side and keeps `contingent` = 0 under
 C1. (j) The notes of the record rows in facts.csv name each round and its date. (k) The build sums the balance-sheet layer over every firm (Arista's L1 of
 zero had dropped its 9.7 of commitments from the sum), the macros round half up as the tables
 do, and the engine's nodes are built in a fixed order so that a rerun reproduces every result
 file byte for byte. Read and not changed, open for the author: Meta's El Paso residual value
 guarantees (HL-0046f), C2 by the verification although the exclusivity agreement
 is "subject to the execution of definitive agreements", the language of C3; the schedules of
 the five firms read at a quarterly report (CoreWeave, NVIDIA, Meta, Amazon, Alphabet), which
 the ladder takes from the annual report under the convention of 2026-10-12 while their
 quarterly reports print the same tables at the ladder date (R13 made the exception for
 Oracle); NVIDIA's stake in OpenAI (HL-0026a), stated only by the press, which enters the
 engine's equity channel at quality 3; and the obligor of CoreWeave's Ellendale leases
 (HL-0037a), which the filing splits between CoreWeave and its vehicle.

- R15 (2026-10-15; the third round, after the two outside reports of October 15). The
 stress test's rules A27 to A32 (`code/python/engine/README.md`), and the data they need.
 (a) Capital expenditure joins the node inputs: the investing outflow for property, plant and
 equipment from the SEC's XBRL company facts at each firm's period end, annualized over its
 own window like operating cash flow (the project's code,
 `nodes.csv` columns `capex_*`); where the facts carry none, the cash-flow statement of the
 covered filing is read by hand with its lines (Nebius, six months to 2026-06-30, 8.1303;
 Digital Realty, six months to 2026-06-30, 1.784229). This is an exception to the data freeze
 of 2026-09-27 (one column, every value cited). The pre-shock ledger (A27) charges it at that
 rate; the headline takes the plan as financed, as before, and says so. (b) The idle cost on a
 commitment falls on the share of each year's payment that the firm's contracted receipts of
 the same year do not cover (A28); the lifetime scalar of the earlier rounds is a sensitivity
 row. Purchase commitments are not matched to a business block: the filings do not say which
 contract serves which capacity, and the dated match uses the schedules they give. (c) The
 allowed damages on a rejected lease are the landlord's shortfall after reletting, capped by
 11 U.S.C. 502(b)(6), never the cap alone (A29); the rent on the leases a failed firm keeps
 is paid ahead of its other claims, as an expense of the continuing estate (11 U.S.C. 365(d),
 503(b)); where even that rent cannot be paid in full the uncovered part goes unpaid, with no
 second rejection. (d) The flows after the horizon on a failed firm's claims are paid at its
 last year's rate (A30), so the lifetime loss on every claim follows one rule; the losses
 counted within the horizon are reported beside the lifetime ones. (e) Every variant that
 switches a guarantee off or caps it leaves its covered shortfall with the beneficiary, net of
 the writer's credits (A31), so the operating loss of the perimeter is the same in every
 support setting. (f) The three guarantees whose trigger the filing does not disclose
 (`support_clauses.csv`, trigger type "not disclosed": Alphabet's energy-equipment guarantees
 HL-0048c, Meta's El Paso residual-value guarantees HL-0046f, Oracle's guarantee of a lessor's
 borrowing HL-0043b) keep the baseline rule chosen from the filing's words (R-A25) and are run
 as a pair, every one paying on the shortfall and none paying (A32). Equinix's and CoreWeave's
 venture guarantees are not in the set: their triggers are disclosed (the venture's failure to
 pay), and the venture pays while its tenant pays. (g) Meta's residual-value guarantees
 (HL-0046c, HL-0046f) pay d c of the threshold, the exit on the idled share of the capacity at
 the shocked value, and the rent on the campus leaseback (HL-0046b) carries the idle cost
 besides: the guarantee covers "any shortfall between the fair value at that time and the RVG
 threshold" on termination or non-renewal (EV-0257), the residual after the lease, not the
 rent during it, so the two are different cash flows and both are charged. (h) The share of
 the scheduled dollars whose year profile comes from a typed filing schedule, as against a
 level profile over the parsed term, is a macro and is stated in the paper. (i) In the support
 packages (2.8), a warrant the supporter receives on the supported party's own stock is a
 claim on that party's success, like a stake, and the concession column keeps only the
 warrants or shares the supporter granted on its own stock; the package P30 (CoreWeave and
 Applied Digital) therefore pairs a claim with coverage. (j) The ledger of A27 credits the
 contracted receipts a firm's coded revenue claims bring in at the firm's cash margin
 (operating cash flow over revenue), the plan's own inflows, which the run-rate cash flow does
 not carry.
- R16 (2026-10-18). Meta's El Paso residual value guarantees (HL-0046f, $13bn) stay in the gross
 and upper bounds. Meta's 10-Q for June 30, 2026 reports them in a subsequent-event note: an
 exclusivity agreement of July 2026 "subject to the execution of definitive agreements and
 customary closing conditions" (meta-20260630.txt line 661), on whose closing Meta "will also
 provide" the guarantees (line 662). The gross bound counts every subsequent event in any
 language (SE1, R7-D2), so the guarantees are in it; they keep rule C2, which now reads a
 guarantee signed, or agreed as a subsequent event, and not yet effective, so that they count
 with the PORTS guaranties among the guarantees not yet in force. They are described as agreed,
 subject to definitive agreements, not as signed (the row's note and ladder basis, EV-0493).
 Settles the question R15 left open. Freeze exception: two text cells of one instrument row and
 one evidence row; no amount changes (the project's code).

## 4. Decisions of the clean-up (2026-09-27)

The conflicts the clean-up of 2026-09-26 found between the old files are settled by the rules
below, one commit each; `CHANGELOG.md` lists what each moved. The default for two figures of one
fact: the writer's own filing decides; within one filing, the table the ladder cites.

- **CU-1. Every reference resolves.** An id field names a row of its table or is empty. Ohio Power
 Company's parent field read `aep`, which is not an entity; American Electric Power is not a node,
 so the field is empty and the relation is stated in the entity's note.
- **CU-2. A clause is coded on its instrument's row.** The clause table's edge is the row's edge, so
 it cannot go stale: SC-14, coded before its edge existed, names HL-0021c. A note that states another
 row's state is corrected when that state changes: SC-13's note said NVIDIA's AI cloud agreements
 sit in L2, which ruling R7-D1 changed to L3.
- **CU-3. An amount is the figure the filing prints, all its digits: within one filing, the table
 the ladder cites** (decision 3, second clause). A rounded copy of a printed figure is not a second
 value. The ladder had typed 17 amounts one or two decimals short of the table it cites (Microsoft's
 purchase commitments 194.1 against the table's 194,060 million; Alphabet's credit backstops 43.8
 against 43,785; Nebius's leases 12.1 against $12,054.1 million; and so on) and the network 6
 (Nebius's two, its Microsoft agreement's 17,392.9 and 6,958.1 million, Alphabet's 43,785); all now
 carry the printed figure. NVIDIA's partner guarantees are 3.5 in the guarantee table the ladder
 cites and 3,529 million in the derivatives table: 3.5, in the network and at every point of the dated
 series. An amount the filing does not print as one figure says how it is derived (Nebius's energy
 commitments, the sum of six yearly figures; Equinix's undrawn AMER 2 loan, 392 less 329). Meta's leases
 not yet commenced are one printed figure, 278.99, that the network splits into the landlords' leases
 (266.68) and the Venture leaseback (12.31); the ladder now carries each leg on its own row, 278.99
 together, where it had carried one rounded 279.0. A fact that restates an instrument's amount names it
 (`restates`), and CHK-SCHEMA asserts they are equal. The rule is applied to the amounts the conflict
 list names; other amounts keep their coded precision until an evidence row ties each to its printed
 figure.
- **CU-4. A note does not restate a value another column of its row holds.** The ladder part of an
 edge sits on the edge's row, so the edge notes that repeated the ladder amount ("ladder L2 component
 0.71 (U2)") went stale the moment the amount moved; they now point to the row.
- **CU-5. Two filings, one instrument: the writer's own filing decides** (decision 3, first clause).
 Meta's contingent purchase of unsold capacity is $14.72bn in Meta's filing and "up to $15 billion" in
 Nebius's; the instrument, written by Meta, carries 14.72 in the network as in the ladder and the
 clause table. What Nebius states stays a fact of its own (`nbis_meta_unsold_capacity`), since the
 paper quotes Nebius's description.
- **CU-6. Where the network and the ladder carry one instrument at different amounts, a rule says
 which.** The network draws a guarantee or backstop at its maximum (its cap or maximum exposure, the
 upper bound), and the gross cell carries the current maximum where the filing states one: Equinix's
 EMEA 2 guarantee is 0.229 in the network and 0.048 in the gross cell (ruling AUDIT6-D3). The network
 draws an obligation the filing gives as a range at the low end, so a network sum never overstates it,
 while the gross cell takes the midpoint and the upper bound the top: CoreWeave's equipment obligations
 are 0.5 in the network and 0.5, 0.85 and 1.2 in the three bounds. These are the only two such
 instruments; `\ledGuaranteeLadderVsNetworkBn` itemizes the first.
- **CU-7. L1 is the sum of its parts: debt at carrying value, operating and finance lease liabilities
 and other debt-like obligations, each a cell of `firm_periods.csv`.** The typed L1 had drifted from
 its parts at four firms. Amazon's financing obligations (current 0.415 and long-term 8.9, excluding
 interest) sat only in the prose of its lease cell; they are a part, and its lease parts are the
 printed present values, 96,320 and 13,451 million (CU-3). AMD's 10-Q states only the long-term
 operating lease liability (1,050); the current portion is the latest stated, 159 at 2025-12-27 in the
 10-K, as for Meta's finance leases, and the ladder had used an estimate. Applied Digital's
 and Digital Realty's typed L1 were roundings of their parts. A firm whose L1 or L2 rests on such a gap
 says so in `disclosure_gap`, and Figure 2 stars exactly those firms (they had typed two
 different lists: Figure 1b lacked Meta).
- **CU-8. A firm's AI-sector stake is the sum of the amounts its formula names**
 (`ai_stake_formula` in `firm_periods.csv`: instrument ids and fact ids); no total is typed. The typed
 totals were roundings of their parts: NVIDIA 7.0 of 4.7 + 2.0 + 0.33 (its stakes in CoreWeave and
 Nebius, edges HL-0036c, HL-0035a, HL-0036g), Meta 16.7 of 13.8 + 2.92, CoreWeave 0.2 of 0.117 +
 0.051, Nebius 1.5 of 1,517.7 million, Dell 2.4 of 2,394 million. The parts with no edge are cited
 facts. Which holdings count stays the judgment the components describe (NVIDIA's baseline counts its
 neocloud stakes, not its broader portfolio; AMD's unnamed investees are left out); a firm with none has
 an empty formula.
- **CU-9. The contract table holds no amounts; an amount a contract row stated lives on an instrument
 row or as a cited fact.** Of the amounts in the two dropped columns of the pilot's contracts file (not in this repository), six were
 carried nowhere else. NVIDIA's 13F portfolio, stated as 63.4 on HL-0036, is the exact sum of the eight
 holdings the 13F prints in dollars ($63,439,974,569), while the eight edges carried one-decimal
 roundings that summed to 63.544; the edges now carry the printed values (CU-3). The warrants' fair
 value (HL-0014), CoreWeave's customer shares (HL-0017), the capital providers' $500bn target (HL-0091),
 NVIDIA's $4.7bn maximum loss to the infrastructure financiers (HL-0093) and the Groq consideration
 (HL-0098) are cited facts. The rest were derived sums, figures an instrument or fact already carries,
 or a date the old numeric companion read as an amount.
- **CU-10. One instrument, one row: its disclosed amount in `notional_usd_bn`, and its amount in each
 bound in `lower_bn`, `gross_bn` and `upper_bn`.** Four instruments had been split into a row and
 ladder-only increments that existed to carry a second amount: CoreWeave's equipment obligations (0.5
 plus two increments of 0.35, now 0.5, 0.85 and 1.2 on one row), Equinix's EMEA 2 guarantee (0.048 plus
 the 0.181 increment to the cap, now 0.048, 0.048 and 0.229), Digital Realty's commitments (3.78 plus
 the 0.32 reimbursable, now 3.78, 4.1 and 4.1) and Alphabet's leases not yet commenced (85.2 plus the
 June 2026 short-term lease 5.8, now 91.0 in every bound). The separate ladder amount column goes,
 since the amount and the bound amounts say it all. A dated amount marks the bound it dates (`bound`
 in `amounts_by_filing.csv`): Equinix's cap is stored as the printed figures, 236 to 229 million, not as
 increments derived from them.
- **CU-11. A table says what a value is and where it comes from; what it was is in git.** A note or
 basis gives the value's source, its quote and how it is read, not the record of its changes, which
 is the changelog's and the commits'; the reason for a rule is this codebook's. The bounds notes, one
 per firm, had become a log of every ruling applied to the firm's cells, with before-and-after values
 long superseded, and a summary of its components that restated their amounts, several of them stale
 (Microsoft's purchase commitments 194.1, Meta's leases 279.0, Nebius's leases 12.1). The column goes:
 the component rows describe each bound, and `disclosure_gap` says where a filing leaves one open.
 The debt notes lose the record of the move to carrying value (ruling R4-L0) and their dated check
 stamps; eight bases and notes lose theirs.
- **CU-12. A row's class and bounds follow its rule.** Every ladder row's bounds match the rules
 table of 2.2. Two rows did not follow a rule that had changed. Dell's supply-chain-finance
 payables left the ladder under ruling R3-A2 (rule X3) and kept class A; they are class X. Equinix's
 committed future equity contributions to the xScale ventures (0.118; the partners "are required"
 to fund approved developments) carried the `o` suffix, which kept a firm item the verified gross
 ladder had omitted in the upper bound only, so that the lower bound could not exceed that ladder.
 Since ruling R7-D2 the gross cell is defined by rule, the suffix has no work to do, and the item takes
 rule U3's three bounds. Rule N1, the netting counterpart of a U1n row, has no rows since CU-10.
- **CU-13. Four ladder components become edges**. CoreWeave's leases executed not
 yet commenced, Oracle's unconditional purchase obligations (primarily data-center power), Applied
 Digital's minimum commitment under its Jamestown energy services agreement, and AMD's investment
 commitments of up to 5.0, entered after its balance-sheet date and subject to contingencies, were in
 the ladder's bounds but in no edge. Each is now a contract row (HL-0100 to HL-0103) and an edge to the firm's counterparty hub, at the figure the filing prints
 (CU-3: Oracle's 13,309 million and Applied Digital's 19,178 thousand, which the ladder had typed as
 13.3 and 0.02). A ladder component that becomes an edge takes its contract's id, and its evidence
 follows it. CoreWeave's aggregate contains the Polaris Forge 1 leases that had not commenced by
 2026-06-30, which the network also draws, at the campus's full contract value, as HL-0037a; neither
 filing gives the part inside the aggregate, so Figure 2 draws the aggregate net of the campus, a
 floor, as ruling AUDIT6-D4 nets named contracts out of a revenue book.
- **CU-14. The network draws every component the gross cell counts**. The gross
 cell counts everything signed by the filing date (ruling R7-D2), and the network already drew
 items signed after a balance-sheet date (Oracle's post-year-end commitments, Broadcom's backstop,
 the PORTS guaranties, AMD's investment commitments). Meta's July 2026 leases and AMD's post-quarter
 leases, the two it did not, are edges HL-0104a and HL-0105a with contract rows HL-0104 and HL-0105.
 The components outside the network are now the upper-bound items only (pipelines, contractual
 maxima, Alphabet's open-order gap, prepayments with a financing component), and CHK-SCHEMA fails on
 a ladder row in the gross cell that is not an edge.
