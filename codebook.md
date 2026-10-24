# Codebook

The rules the data follow, each with its reason. This release holds five tables of the contract
database (`data/core/`), the list of the filings they cite (`data/filings.csv`), and the numbers
the paper's Figure 2 plots. Section 1 describes the tables column by column, Section 2 the
conventions they are coded under, Section 3 the rulings on individual rows, and Section 4 the
rules that settle conflicting figures and the tables' structure. A rule or ruling keeps the id it
was taken under (U1, C2, R7-D2, AUDIT6-D4, CU-7, ...), and the tables' notes cite those ids, so a
reader can follow a cell to its rule here.

## 1. The tables

The tables are UTF-8 CSV files with CRLF line ends, in the form Python's csv module writes (a field
is quoted only where it must be). An empty cell means not applicable or not disclosed, never zero.
A row keeps its id when its content changes.

| Table | One row per | Key |
|---|---|---|
| `entities.csv` | legal entity, anonymized counterparty or role hub | `entity_id` |
| `contracts.csv` | arrangement, as the filings describe it | `contract_id` |
| `instruments.csv` | economic leg of an arrangement (an obligation, guarantee, backstop, stake, prepayment or loan), or a ladder component that is not a leg | `instrument_id` |
| `firm_periods.csv` | sample firm at its ladder balance-sheet date | `firm` |
| `evidence.csv` | quotation behind a coded field of the four tables above | `evidence_id` |
| `filings.csv` (in `data/`) | document of a cited filing | `accession`, `file` |

The database has three further tables that this release does not hold: the amount of each dated
guarantee in every periodic filing (2.6), the filing figures that are not an instrument's amount
("facts": payment schedules by year, the parts of a sum, counts), and the support packages. Where a
cell of the five tables names a fact (`ai_stake_formula`) or says that an amount is "in facts", the
figure is printed in the filing the row cites.

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

One row per arrangement, without amount columns: the amounts are on the arrangement's instruments
(CU-9). The text cells describe the arrangement in the filing's terms and may quote figures; a
figure is data only where an instrument carries it.

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
- `confidence_1to5`: the coder's confidence in the row, 5 the highest; the row's legs carry it as
  `source_quality_1to5`.
- `notes`: what the row rests on, and how it was checked: "verified against the filing" for a row
  the author checked line by line, or the ruling under which it was checked (R8).

### instruments.csv

A row is a network edge when `edge_id` is set, a ladder component when `ladder_firm` is set, and
often both. It fills the coverage, clause and dating columns when it is among the contingent edges
(2.4), the clause rows (2.5) or the dated guarantees (2.6). One instrument is one row, with its
amount in each bound on the row (CU-10). A ladder component that is not an edge has an id
`LD-<TICKER>-nn`; every other id is its contract's id with a letter.

The row and its edge:

- `instrument_id`; `edge_id`: the same id when the row is an edge, else empty.
- `contract_id`: the contract row. `HL-0034;HL-0047` marks the one arrangement both parties'
  filings describe, coded once: Meta's contingent purchase of Nebius's unsold capacity (CU-5).
- `from_entity`, `to_entity`: the party that owes, and the claimholder (2.3).
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

The contingent edges (2.4): `protection_holder` and `reference_entity` (entity ids; several
reference entities `;`-separated), `underlying_edges` (the claim edges that cover the position),
`coverage_class`, `coverage_note`.

The clause rows (2.5): `clause_id` (`SC-nn`), `writer_role`, `clause_contract_id`,
`clause_instrument`, `clause_trigger`, `trigger_type`, the four rights `assumption`, `purchase`,
`sublease` and `sale`, `other_remedy`, `exclusivity`, `carrying`, `clause_note`,
`clause_verified_round`.

The dated guarantees (2.6): `date_is_upper_bound`, `date_confidence`, `robustness_flag`,
`disclosure_filing`, `date_note`, `amount_path_note`, `dating_verified_round`, `firmed_round`.

`ai_class`: for an L2 or L3 component, `ai` where the filing's words or the counterparty tie it to
AI infrastructure, `other` where they tie it to another business, `unallocated` where the filing
gives a company-wide total (R10 (e)); empty on a row that is not a ladder component.

### firm_periods.csv

- `firm` (ticker); `period_end`: the balance-sheet date of the filing the ladder reads
  (`ladder_accession`); `total_assets_bn`.
- `debt_carrying_bn`: L0, debt at carrying value (R4-L0), with `debt_basis`, `debt_note` (the lines,
  and the face or principal amount beside the carrying value) and `debt_status`.
- `op_lease_liab_bn`, `fin_lease_liab_bn`, `other_l1_bn`, each with its `_basis`: the rest of L1
  (CU-7).
- `revenue_bn`, `revenue_basis`, `revenue_source`: annualized revenue (R3-B); `rpo_bn`, `rpo_basis`,
  `rpo_source`: remaining performance obligations, the company total (AUDIT6-D6);
  `denominators_verification`: how the two were checked. A source is accession | file | lines.
- `ai_stake_formula`: the instrument and fact ids whose amounts sum to the firm's AI-sector stakes
  (CU-8); `ai_stake_components`, `ai_stake_cost_bn`, `ai_stake_source`, `ai_stake_note`: the
  holdings' names, their cost where disclosed, the citation, and the judgment behind the selection
  (2.7).
- `disclosure_gap`: where the quarterly report leaves L1 or L2 to the prior annual report; Figure 2
  marks these firms with an asterisk (CU-7).

### evidence.csv

- `evidence_id` (`EV-nnnn`).
- `table`, `row_id`, `field`: the row (`;`-separated where one quote supports several) and field
  the quote supports; `table` is empty for a passage that supports the database as a whole, such as
  a notice of redactions.
- `accession`, `file`, `lines`, `quote` (2.1); `note`.

### filings.csv

One row per document of every filing the tables cite: `accession`, `filer`, `cik`, `document` (the
document's name on EDGAR), `file` (the name of its plain-text conversion, which the `lines` columns
count in), `url` (the document on EDGAR) and `filing_index` (the filing's index page).

## 2. Conventions

### 2.1 Amounts, dates and citations

- An amount is in USD billions and is the figure the filing prints, with all its digits (CU-3): a
  figure printed in millions is divided by 1,000, so 194,060 is 194.06. An amount the filing does
  not print as one figure says how it is derived (Nebius's energy commitments, ruling R3-A1).
- Dates are ISO 8601. A filing's date is its SEC filing date; `period_end` is a balance-sheet date.
- Accession numbers are dashed (`0001045810-26-000075`). A citation is accession | file | lines:
  `file` is the plain-text conversion of the document that was coded (`filings.csv` gives the
  document on EDGAR), and lines are counted from 1 in it.
- A quote is verbatim from the cited lines. Inside it, ` / ` marks a line break or a table-cell
  boundary, ` ... ` text left out, and square brackets a redaction marker or the quoter's completion
  of a cut word. Every quote has been located at its lines, comparing both in one form: lower case,
  one kind of quotation mark and dash, footnote marks dropped, and the spacing the text conversion
  adds removed (`$ 329.1`, `12 %`, `( 24,590 )`).
- A value is stated once. A note or basis says what a value is and where it comes from, and does not
  restate a value another column holds (CU-4).

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
| D1 | a split computed from the filing's arithmetic, shown on the row | no | no | yes |
| XL | customer prepayments the firm's own filing ties to a significant financing component (R6b-J2b) | no | no | yes |
| X1 to X6 | outside the ladder: nothing quantified disclosed (X1); unquantified (X2); a funding-side liability, such as supply-chain finance or customer prepayments without a financing component (X3); an intra-group guarantee of consolidated debt (X4); credit support a bank writes on the firm's own obligations (X5); a claim the firm holds rather than a promise it wrote (X6) | no | no | no |

`ladder_class` follows from the rule (CU-12): A for U1 to U4 and E1, and for a subsequent event in
firm language; B for M1; C for M2, C1 to C4, D1, and a subsequent event in conditional language; X
for X1 to X6 and XL, and for an option too unsized to enter any bound.

A writer's cap sums the upper amounts of the guarantees it wrote (rules E1 and C2), demand
backstops excluded (R7-D1); where the gross amount differs, it is the writer's current maximum
(AUDIT6-D3).

### 2.3 The network

- **Direction.** `from_entity` is the party whose balance sheet carries the obligation, or against
  which the claim runs; `to_entity` holds the claim. A lease runs lessee to lessor, a guarantee
  guarantor to beneficiary, a loan borrower to lender, a funded stake investee to investor, and an
  unfunded commitment investor to investee (`funding_commitment`). Every edge is a liability of its
  from-party.
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
  recomputed from the edges, since many commercial links run to anonymous parties, and no result
  of the paper uses it (AUDIT6-D10).
- **`source_quality_1to5`** is the contract row's `confidence_1to5`.
- **The contracted-revenue books.** Five edges (HL-0086a to HL-0090a: Microsoft, Amazon, Alphabet,
  Broadcom, Nebius) carry firm-level backlogs from one anonymous customer party. They are not
  measured on a common basis (Microsoft's includes unearned revenue and states no exclusion of
  cancelable contracts; Broadcom's excludes contracts terminable for convenience; Alphabet's and
  Nebius's state their own exclusions), so a sum across them says so, and none carries a
  solvent-backed share (R6-J1). Where a book contains named contracts that are also edges (Nebius's
  Microsoft and Meta contracts, CoreWeave's), or an aggregate contains a campus coded on its own
  (the Polaris Forge 1 leases inside CoreWeave's leases not yet commenced, CU-13), a drawing of the
  network nets the named part out, so the netted arrow is a floor (AUDIT6-D4).

### 2.4 The contingent edges

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

Every contingent edge carries one of these classes; none is a naked position.

### 2.5 The clause rows

The rows are every guarantee-type instrument a sample firm wrote on another party's obligation or
asset value: the L3 rows under rules E1, C2 and U4, among them NVIDIA's AI cloud agreements and
CoreWeave's New Jersey venture guarantee (R7-A1). Parent guarantees of a firm's own consolidated
vehicles (X4), OpenAI's guaranty of its own tenant subsidiary's lease, options and pipelines,
Digital Realty's carve-out guarantees and Amazon's facility to Anthropic are not rows.

- `writer_role`, the writer's relation to the asset: `supplier`, `internal_use`, `customer`,
  `co_owner`, `tenant` (R7-A2).
- `trigger_type`: `exit` (pays on the user's own exit), `obligor_default`, `demand`, `not disclosed`,
  and `writer_nonperformance` (the right arises when a contract is terminated for the writer's own
  failure to perform: HPE's SC-15).
- `assumption`, `purchase`, `sublease`, `sale`, the writer's rights over the asset on failure:
  `yes`, `no`, `not disclosed`, or `not applicable` for the two demand backstops (R7-A3).
- Access is derived: `yes` if any right is `yes`, else `not applicable` if every right is, else `no`;
  a right not disclosed counts as no disclosed right (R7-A5). The paper counts two things: writers
  with any disclosed right, and writers whose right arises on the reference party's default
  (`trigger_type` `obligor_default`), which excludes a right on the writer's own exit or
  non-performance.
- `clause_trigger`, `other_remedy`, `exclusivity`, `carrying`: the trigger, the other remedies, any
  exclusivity and the carrying value, in the filing's terms. The row's maximum is the instrument's
  upper amount (CU-2).
- The two demand backstops (SC-09, SC-13) form their own block and do not count among the writers
  with access rights (R7-A3).
- `clause_verified_round`: the verification pass that checked the clause coding.

### 2.6 The dated guarantees

- Each L3 component in a bound is dated by the first filing that quantifies it, not by its legal
  effective date (R6-J6). The database holds its amount in that filing and in every later periodic
  report, or marks it absent from the first report that no longer states it: no longer disclosed,
  not ended (AUDIT6-D8).
- An executed subsequent-event guarantee enters the series at the filing date of the report that
  discloses it (AUDIT6-D9).
- `date_is_upper_bound` = 1 when no earlier filing quantifies the component, so it may have been
  disclosed before; `robustness_flag` = 1 when the date is an upper bound or its confidence is
  below high (R6-J6).
- `disclosure_filing`, `date_note`, `amount_path_note`: the first filing, the reason for the date,
  and how each filing's amount was read (lines and arithmetic).
- `dating_verified_round`, `firmed_round`: the verification pass that checked the dating, or firmed
  an upper-bound date (`ext`: the pass for the firms added to the sample, R8).

### 2.7 Denominators and stakes

- Revenue is the latest fiscal year's where the target report is an annual report, and otherwise
  annualized from the shorter period the report covers: twice the latest half year, four thirds of
  the latest nine months, or four times the latest quarter. `revenue_basis` opens with the factor
  (`2x`, `4/3`, `4x`); for a fast grower the annualization understates run-rate revenue (R3-B).
- RPO is the company total (Microsoft 684, Alphabet 519.5), while the network edges keep the segment
  books (678, 513.9) (AUDIT6-D6, R6-J2). Applied Digital's figure is the minimum lease payments
  owed to it as a lessor.
- A firm's AI-sector stakes are the sum its formula names (CU-8). Which holdings count is a judgment
  the note states: NVIDIA's baseline counts its neocloud stakes, and its stake in OpenAI (HL-0026a),
  known only from the parties' announcements and named in full on the contract row HL-0026, is a
  separate figure (no NVIDIA filing names OpenAI; the quarterly report's non-marketable securities
  line corroborates the order of magnitude, EV-0492); Dell carries its non-marketable securities as
  a placeholder; AMD's unnamed investees are left out (R3-C).

### 2.8 Which firms Figure 2 shows

A figure with every sample firm as a row is not readable. Figure 2 shows the firms whose
contractual obligations at the gross bound (L3) are at least $25bn, a design constant like the
sample rule's size floors (`in_figure` in the figure's table); the table of the figure's numbers in
this release, like the paper's appendix tables, carries all 37 firms.

## 3. Rulings

Each ruling in one or two sentences, by topic. The id says when it was taken: R3 to R9 in the
passes that verified the coded rows against the filings, R10 to R16 dated, AUDIT and AUDIT6 in the
data audits of 2026-09-08 and 2026-09-25. The conventions above apply them. Rulings R15 and R17 to
R21 concern the inputs of the paper's stress test, which this release does not include.

**The ladder.**
- R3-A1. Nebius's energy commitments (HL-0071) are one row carrying the sum of the six yearly figures
  the filing gives without a total, 5,324.1 million.
- R3-A2. Dell's supply-chain-finance payables (3.1) are a funding-side liability (X3), outside both
  bounds and in the network only: a bank finances the supplier against Dell's confirmed payable, the
  mirror of a customer prepayment.
- R4-L0. L0 is debt at carrying value for every firm, the balance-sheet quantity every filer
  discloses.
- R5-J1. Microsoft's construction commitments and its leases not yet commenced are additive: they
  come from different notes, and the filing gives no overlap.
- R6b-J2, R6b-J2b. Customer prepayments count at the upper bound (XL) when the firm's own filing ties
  them to a significant financing component, the firm booking imputed interest on them, and stay
  outside the ladder otherwise (X3): Nebius's 6.0 and Oracle's 4.6 are in; CoreWeave's 7.0,
  Broadcom's 14.2 and NVIDIA's 2.8 are out.
- AUDIT6-D7. CoreWeave's line on financing components is a general definition of interest expense,
  so its deferred revenue stays X3; Nebius's component is sized in its 20-F and 6-K.
- AUDIT-X4. Parent guarantees of a firm's own consolidated vehicles (CoreWeave's HL-0019b, HL-0037b,
  and AMD's and Applied Digital's) are X4: the underlying obligation is already drawn in full
  between the same parties. The edges stay in the network, flagged intra-group, outside the
  distinct guarantee total.
- AUDIT6-D1. L3 holds support the firm wrote on another party's obligation or asset value. Amazon's
  standby letters of credit, which banks write on Amazon's own obligations, are outside L3 (X5);
  Digital Realty's pro-rata share of non-recourse venture debt is upper bound only (C4).
- AUDIT6-D2. Applied Digital's Base Electron loan and stake are a claim it holds, not a promise it
  wrote (X6), and are outside L3 and the dated series.
- AUDIT6-D3. Equinix's $48m current maximum payment is in the lower and gross cells; its cap of
  $229m is the upper amount.
- AUDIT6-G, R7-D2. The gross cell counts everything signed by the filing date: contingent
  commitments (C1), subsequent events (SE1) and signed guarantees not yet effective (C2). A funding
  commitment is the firm's own obligation, so Amazon's Anthropic facility sits in L2 with every
  other C1 row.
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
- R4-J5. The three Applied Digital lease 8-Ks are cited by their accessions.
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
- R6-J1. The contracted-revenue books carry no solvent-backed share; where the paper's stress test
  needs one, it is a modelling assumption with its own sensitivity.
- R6-J2. Alphabet's book: headline 519.5 (the total backlog), committed 513.9 (Google Cloud).
- R6-J3. The 30-month expiry after an Anthropic liquidity event is HL-0044's termination right.
- R6-J5. Broadcom's lessee is unnamed in the filing (E-ANON-AVGO-LESSEE); the press reports conflict.
- R6-J7. The FT's "up to 25 per cent" residual value is attributed to the FT and used in no analysis.
- R6b-J1. An option or memorandum without a committed amount has an empty amount, not zero (HL-0091,
  HL-0006).
- R6b-J3. The Groq deferred consideration (HL-0098) is coded, outside the ladder, as no firm's
  deferred acquisition consideration is in it.
- R6b-J4. NVIDIA's extended payment terms (HL-0095) are coded without an amount.
- R6b-J5. NVIDIA's receivable concentration is a link between counterparties, not a contract.
- R6b-J6. The Level 2 warrants and preferred (HL-0099) note a possible overlap with the Nebius
  warrant; the stakes do not change.
- R6b-J7. The equity forward (HL-0094) is its own row: one instrument, one row.
- AUDIT6-D4. A drawing of the network shows the Nebius book net of its named Microsoft and Meta
  contracts, as it shows CoreWeave's; the netted arrows are floors.
- AUDIT6-D10. The circularity flag is the coder's label (2.3); HL-0094a's notional type is `unsigned`.

**The dated guarantees.**
- R6-J6. A component is dated by the first filing that quantifies it (2.6).
- R6b-J8, AUDIT6-D8. Oracle's lessor guarantee is no longer disclosed from the 10-Q filed 2026-09-11,
  which precedes its stated maturity and reports no release: absent, not ended.
- AUDIT6-D9. Alphabet's April 2026 backstop of about $15.3bn enters the executed series on
  2026-04-30, and the pipeline point of that date falls by the same amount.

**Denominators and stakes.**
- R3-B. Revenue is annualized from the shorter period of a quarterly report (2.7).
- R3-C. Dell's stakes are a placeholder of 2.4; AMD's unnamed portfolio is left out, since Dell's
  filing ties one investee's mark-up to AI and AMD's names none.
- AUDIT6-D6. The paper's RPO figures are company totals; the network keeps the segment books (2.7).

**The clause rows and concessions.**
- R7-A1 to R7-A5. The clause rows, the writer's relation to the asset, the demand backstops' block,
  and the writer counts, as in 2.5.
- R7-A6. The 13 clause rows were verified as coded.
- R7-C. A warrant or share on a supporter's own stock granted to a customer with a purchase
  agreement is a concession, not a claim: the supporter's shareholders pay for it, so it is support,
  but it pays when the customer buys, not if it fails, so it carries no exposure.

**The verification of the coded rows.**
- R6-J4. The ten NVIDIA items the first reading had found missing were verified, and the
  upper-bound disclosure dates firmed. Nebius, a foreign private issuer, files 20-F and 6-K reports
  (CIK 1513845), on which its coding rests.
- R8. The rows of the 23 firms added to the sample were verified by the author under a rule fixed
  before the check: every ladder row of the extension with a gross amount of at least $5bn, every
  guarantee, backstop, funding commitment, stake or warrant, and every other row whose contract
  number ends in 0 or 5, plus the clause rows SC-15 and SC-16. Every row was confirmed as coded; the
  ladder-only `LD-` rows (class X, no counterparty, never summed) were confirmed as intended. Rows
  outside that rule rest on their quoted passages, and their notes say so.
- R9. The 23 firms' balance-sheet denominators were verified by the author: for each firm, total
  assets, reported debt, lease liabilities, annual revenue with its basis, and remaining performance
  obligations, each with the filing's lines. Every cell was confirmed; `denominators_verification`
  and `debt_status` in `firm_periods.csv` say so.

**Later rulings.**
- R10 (2026-10-12). (a) IBM's client financing commitments (HL-0116a, rule C1) sit in rung L2, where
  rule C1 puts every funding commitment. (b) HPE's performance guarantees (SC-15) carry `trigger_type`
  `writer_nonperformance`. (c) Access is derived by reading a disclosed right before the
  not-applicable marker (2.5). (d) Payment schedules by year for all 37 firms are typed from their
  filings, each cell with its line. (e) `ai_class` on every L2 and L3 ladder component: `ai` where
  the filing's words or the counterparty tie the obligation to AI (a laboratory, an AI cloud, AI
  accelerators or racks, a pure AI cloud's own book); `other` where the filing ties it to a business
  that is not AI infrastructure (a foundry's fabs, a content-delivery network's co-location, IT
  service contracts, client financing lines); `unallocated` for a company-wide total or a
  data-center lease or construction commitment the filing does not tie to AI. The classification
  reads the component's description and counterparty, never the firm's name or role. (f) For an
  `unallocated` book, the share tied to the build-out is the book's growth since the last fiscal
  year that ended before 2023, from that year's annual report, as a share of the book today; a firm that disclosed no such
  leases that year has a baseline of zero for that component.
- R11 (2026-10-14). The ladder's revenue books (`rpo_bn`) report what the filing reports. Where a
  filing states that a book includes revenue already collected and quantifies it (Microsoft's
  commercial remaining performance obligations, which include unearned revenue), the paper nets the
  collected part when it counts future receipts; Oracle's book excludes usage-based consideration and
  Amazon's is commitments not yet recognized, so nothing is netted there.
- R12 (2026-10-14). A check of the 58 contract rows outside the author's verification passes found
  seven filed contracts without an evidence row, ten evidence rows with an empty quote, one
  mistyped amount and one press-only amount. Evidence rows were added for every quantified
  instrument of HL-0029, HL-0033, HL-0035, HL-0037, HL-0038 and HL-0041 whose passage is in a cited
  filing; the empty quotes were filled verbatim from the cited lines; HL-0041a (CoreWeave's Magnetar
  loan) is 0.189, the remaining obligation at June 30, 2026; HL-0025a (a securitization known only
  from a press release not on file) has no amount, so the arrangement stays as a row and enters no
  sum. HL-0040 (Meta's order form with CoreWeave) cites the current report of September 30, 2025,
  which states the amount (line 54). HL-0037b (CoreWeave's springing parent guaranties on the
  Polaris Forge 1 leases) carries the $50m letter of credit the lessor's 10-K names beside the
  springing guaranty. A second check, by four independent readers each reporting disagreements only,
  found: HL-0008a has no amount (a grid-investment pledge, not a PPA notional); HL-0040a is at the
  latest stated commitment, $21.0bn (the March 2026 order form; the filings do not say whether the
  earlier $14.2bn form stands in addition, so the row is the latest amount, not a sum); the warrants
  of HL-0037c are held by their buyers; HL-0038e is a subsequent event (the notes of June 16, 2026
  refinanced the Bridge Facility inside the May 31 reported debt).
- R13 (2026-10-14). Oracle is read at its latest periodic report, the quarterly report for August 31,
  2026 (0001193125-26-389274, filed September 11), as the sample rule says. The tables take from it
  the period, the denominators (total assets, notes payable and other borrowings current plus
  non-current, the lease liabilities), the revenue (four times the quarter, R3-B), the remaining
  performance obligations (664, 13 percent within twelve months), the leases not yet commenced
  (288), the unconditional purchase obligations (34.150 under the filing's new description, with
  its schedule; the $19bn signed after May 31 sits inside the total, and HL-0050a is outside the
  ladder), the lease maturity tables, and the quarter's prepayments with a financing component
  (11.363). The lessor-borrowing guarantee (HL-0043b) is absent from the report before its stated
  maturity (R6b-J8): outside the lower bound, kept in gross and upper at its last stated amount. The
  debt maturity schedule stays with the annual report, which the quarterly report leaves it to
  (`disclosure_gap`, the asterisk).
- R14 (2026-10-14). A check of the tables against the filings settled the following, every new
  amount citing its line and quoting it. (a) Schedules come from the latest annual report:
  Coherent's and Super Micro's lease schedules are typed from their fiscal-2026 reports, and
  purchase-commitment schedules are typed where the filing gives one, for Coherent (3.4 in fiscal
  2027, 8.4 thereafter), Rackspace (by year to 2030), Lumentum and Seagate (within one year and
  after, with Seagate's equipment commitments as a second schedule). A commitment whose maturity the
  filing states in words takes the window as its term where the filing says "primarily" or
  "generally" (Super Micro, Fabrinet) and the window over a share of one half where it says "a
  majority" (Intel, Penguin). (b) R11 applies to every revenue book whose filing says it includes
  unearned or deferred revenue and states the amount: Alphabet, Broadcom, Micron, and the
  deferred-revenue rows of Arista, Super Micro, HPE and Everpure; Super Micro's and HPE's books equal
  their deferred revenue. (c) Oracle's $19bn of cloud-infrastructure commitments signed after May 31
  (HL-0050a) sit inside the quarterly report's total (HL-0101a, R13) and are counted once. (d)
  CoreWeave's OEM financing (HL-0018a) is the outstanding balance the filing states (4.8), not the
  quarter's reclassification line (1.469). (e) Everpure's component-supply commitment, quantified at
  1.1 in the quarterly report for August 2, 2026, is the edge HL-0136b (CU-13): a contingent purchase
  obligation of the firm's own, triggered when its contract manufacturers do not buy, in the gross
  and upper cells of L2 (C1); it is not support written for another party, so it is not in L3. (f)
  Marvell's commitment column includes 220.0 of technology licence obligations the balance sheet
  carries: they are in `other_l1_bn`, and the row HL-0112b is net of them. (g) Oracle's prepayments
  with a financing component (HL-0051a, XL, the upper bound only): the bound is the sum of the two
  stated inflows, 11.363 and 4.592, since no balance is stated. (h) Jabil's $80m of residual value
  guarantees on fiscal-2026 leases were in force at the balance-sheet date, so they are in the lower
  bound too. (i) Smaller cells: two maturities and one note filled from the filings (HL-0065a,
  HL-0072a, HL-0152a); CoreWeave's deferred revenue and Meta's take-or-pay at all their digits
  (CU-3); the warrant rows HL-0028a and HL-0029a carry no amount rather than zero; Equinix's AMER 2
  loan commitment is unconditional on the lender's side and keeps `contingent` = 0 under C1. Left
  as coded: the schedules of the five firms read at a quarterly report (CoreWeave, NVIDIA, Meta,
  Amazon, Alphabet), which the ladder takes from the annual report while their quarterly reports
  print the same tables at the ladder date (R13 made the exception for Oracle); NVIDIA's stake in
  OpenAI (HL-0026a), stated only by the press (2.7); and the obligor of CoreWeave's Ellendale leases
  (HL-0037a), which the filing splits between CoreWeave and its vehicle.
- R16 (2026-10-18). Meta's El Paso residual value guarantees (HL-0046f, $13bn) are in the gross and
  upper bounds. Meta's 10-Q for June 30, 2026 reports them in a subsequent-event note: an exclusivity
  agreement of July 2026 "subject to the execution of definitive agreements and customary closing
  conditions" (meta-20260630.txt line 661), on whose closing Meta "will also provide" the guarantees
  (line 662). The gross bound counts every subsequent event in any language (SE1, R7-D2), so the
  guarantees are in it; they keep rule C2, a guarantee signed, or agreed as a subsequent event, and
  not yet effective, so they count with the PORTS guaranties among the guarantees not yet in force.
  They are described as agreed, subject to definitive agreements, not as signed (EV-0493).

## 4. Conflicting figures and the tables' structure

Where two figures of one fact differ, the writer's own filing decides; within one filing, the table
the ladder cites.

- **CU-1. Every reference resolves.** An id field names a row of its table or is empty; a parent that
  is not a party is stated in the entity's note (Ohio Power Company's parent, American Electric
  Power).
- **CU-2. A clause is coded on its instrument's row,** so the clause row and its edge cannot
  diverge.
- **CU-3. An amount is the figure the filing prints, all its digits: within one filing, the table
  the ladder cites.** A rounded copy of a printed figure is not a second value (Microsoft's purchase
  commitments are 194.06, the table's 194,060 million). NVIDIA's partner guarantees are 3.5 in the
  guarantee table the ladder cites and 3,529 million in the derivatives table: 3.5, everywhere. An
  amount the filing does not print as one figure says how it is derived (Nebius's energy
  commitments, the sum of six yearly figures; Equinix's undrawn AMER 2 loan, 392 less 329). Meta's
  leases not yet commenced are one printed figure, 278.99, that the network splits into the
  landlords' leases (266.68) and the Venture leaseback (12.31), each on its own row.
- **CU-4. A note does not restate a value another column of its row holds;** it points to the row.
- **CU-5. Two filings, one instrument: the writer's own filing decides.** Meta's contingent purchase
  of unsold capacity is $14.72bn in Meta's filing and "up to $15 billion" in Nebius's; the
  instrument, written by Meta, carries 14.72 in the network as in the ladder and the clause row.
- **CU-6. Where the network and the ladder carry one instrument at different amounts, a rule says
  which.** The network draws a guarantee or backstop at its maximum (its cap or maximum exposure, the
  upper bound), and the gross cell carries the current maximum where the filing states one: Equinix's
  EMEA 2 guarantee is 0.229 in the network and 0.048 in the gross cell (AUDIT6-D3). The network draws
  an obligation the filing gives as a range at the low end, so a network sum never overstates it,
  while the gross cell takes the midpoint and the upper bound the top: CoreWeave's equipment
  obligations are 0.5 in the network and 0.5, 0.85 and 1.2 in the three bounds. These are the only
  two such instruments.
- **CU-7. L1 is the sum of its parts: debt at carrying value, operating and finance lease liabilities
  and other debt-like obligations, each a cell of `firm_periods.csv`.** Amazon's financing
  obligations (current 0.415 and long-term 8.9, excluding interest) are a part, and its lease parts
  are the printed present values, 96,320 and 13,451 million (CU-3). AMD's 10-Q states only the
  long-term operating lease liability (1,050); the current portion is the latest stated, 159 at
  2025-12-27 in the 10-K, as for Meta's finance leases. A firm whose L1 or L2 rests on such a gap
  says so in `disclosure_gap`, and Figure 2 marks exactly those firms.
- **CU-8. A firm's AI-sector stake is the sum of the amounts its formula names** (`ai_stake_formula`
  in `firm_periods.csv`: instrument ids and fact ids); no total is typed (NVIDIA: its stakes in
  CoreWeave and Nebius, edges HL-0036c, HL-0035a, HL-0036g; Meta 13.8 + 2.92; CoreWeave 0.117 +
  0.051; Nebius 1,517.7 million; Dell 2,394 million). Which holdings count is the judgment the
  components describe (2.7); a firm with none has an empty formula.
- **CU-9. The contract table holds no amounts; an amount a contract row states lives on an
  instrument row.** NVIDIA's 13F portfolio (HL-0036) is the sum of the eight holdings the 13F prints
  in dollars ($63,439,974,569), and the eight edges carry the printed values (CU-3).
  The warrants' fair value (HL-0014), CoreWeave's customer shares (HL-0017), the capital providers'
  $500bn target (HL-0091), NVIDIA's $4.7bn maximum loss to the infrastructure financiers (HL-0093)
  and the Groq consideration (HL-0098) are cited figures of the facts table.
- **CU-10. One instrument, one row: its disclosed amount in `notional_usd_bn`, and its amount in each
  bound in `lower_bn`, `gross_bn` and `upper_bn`.** CoreWeave's equipment obligations are 0.5, 0.85
  and 1.2 on one row; Equinix's EMEA 2 guarantee 0.048, 0.048 and 0.229; Digital Realty's
  commitments 3.78, 4.1 and 4.1 (the 0.32 customers reimburse); Alphabet's leases not yet commenced
  91.0 in every bound, the June 2026 short-term lease of 5.8 included.
- **CU-11. A table says what a value is and where it comes from.** A note or basis gives the value's
  source, its quote and how it is read; the reason for a rule is this codebook's; how a value
  changed between versions is in `CHANGELOG.md`.
- **CU-12. A row's class and bounds follow its rule,** as the rules table of 2.2 gives them: Dell's
  supply-chain-finance payables (X3, R3-A2) are class X; Equinix's committed future equity
  contributions to the xScale ventures (0.118; the partners "are required" to fund approved
  developments) take rule U3's three bounds.
- **CU-13. A ladder component with a counterparty is an edge.** CoreWeave's leases executed not yet
  commenced, Oracle's unconditional purchase obligations (primarily data-center power), Applied
  Digital's minimum commitment under its Jamestown energy services agreement, and AMD's investment
  commitments of up to 5.0, entered after its balance-sheet date and subject to contingencies, are
  contract rows (HL-0100 to HL-0103) and edges to the firm's counterparty hub, at the figure the
  filing prints (Oracle's 13,309 million, Applied Digital's 19,178 thousand). A ladder component that
  becomes an edge takes its contract's id, and its evidence follows it. CoreWeave's aggregate
  contains the Polaris Forge 1 leases that had not commenced by 2026-06-30, which the network also
  draws, at the campus's full contract value, as HL-0037a; neither filing gives the part inside the
  aggregate, so a drawing of the network shows the aggregate net of the campus, a floor (2.3).
- **CU-14. The network draws every component the gross cell counts,** among them the items signed
  after a balance-sheet date (Oracle's post-year-end commitments, Broadcom's backstop, the PORTS
  guaranties, AMD's investment commitments, Meta's July 2026 leases, HL-0104a, and AMD's
  post-quarter leases, HL-0105a). The components outside the network are the upper-bound items only
  (pipelines, contractual maxima, Alphabet's open-order gap, prepayments with a financing
  component).
