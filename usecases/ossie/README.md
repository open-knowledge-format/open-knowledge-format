# Semantic model linkage

**A proposed OKF feature, in two stages.**

An OKF concept can already say what a metric means, who approved it, and what to watch out for. It cannot say where that metric is computed, or name anything inside the artifact that computes it.

**Stage one** lets a concept point at a semantic model at a reviewed version, and lets prose address individual metrics and fields inside it. Two optional frontmatter keys and one addressing rule. An agent given a stage one concept answers by naming a metric and the fields to group and filter by, working out what is available from the model and the concept's prose.

**Stage two** adds a declared surface: the exact set of requests one metric accepts, so the agent picks from a written list instead of working it out. It applies per metric, and a metric without one stays open.

**In both stages the agent asks for a metric and fields, not a query**, and the model supplies the names either way. What stage two changes is where the set of legal requests comes from: derived from the model, or enumerated in the concept.

Both are proposed. They are separate stages because stage two is strictly additive on stage one and a bundle can stop after the first, which most will: a model holds far more metrics than anyone will approve surfaces for.

---

## The spectrum

Business knowledge is not expressed at one level of rigor. Two questions place any piece of it: **is there a specific number in question**, and **is it defined precisely enough to compute?**

| Position | Number? | Precise enough? | Example |
| :---- | :---- | :---- | :---- |
| **Business context** | No | n/a | Why we reorganised the sales regions; how we judge partner quality |
| **Described metric** | Yes | No | Health score blends product usage, ticket volume, and exec engagement |
| **Modeled metric** | Yes | Yes | NRR, computed over datasets and relationships a model states |

The middle row is where agents fail worst. Business context is safe: there is no number, so an agent relays it. A modeled metric is tractable: the definition sits in a machine-readable artifact, so an agent asks for it in the model's own names rather than guessing, and the requests worth approving can be written down. A described metric is neither. It reads as computable, so an agent tries, invents the thresholds and joins the prose left out, and returns a number that looks right, cannot be checked, and disagrees with finance without anyone noticing.

**The two stages are how a concept reaches the bottom row.** Stage one names the model that computes the metric, so the prose stops being the only specification of it. Stage two writes down which requests against that model are approved. Neither touches the top row, which is what OKF already does.

Two properties of the spectrum shape the design. **It is not a quality ranking.** A good policy document is worth more than a hasty model, and position reflects only how precisely something can be stated. And **one bundle has to hold all three**, because the person asking does not know which row their answer falls in; that is usually what they came to find out.

---

## At a glance

**Stage one. A concept points at a semantic model, at a reviewed version.**

```yaml
semantic_model: ../models/revenue.ossie.yaml
semantic_model_version: sha256:9f3c1a…
```

**Prose then addresses what is inside it**, by putting a `#` fragment on the path:

```markdown
Prefer [`net_revenue_retention`](../models/revenue.ossie.yaml#metric:net_revenue_retention)
over raw expansion arithmetic; it already nets downgrades.

[`employee.full_name`](../models/revenue.ossie.yaml#field:employee.full_name)
exists in the model and is closed to reporting under privacy policy PRV-4.
```

That is the whole of stage one: two optional frontmatter keys, and one rule that a `#` fragment names an entity inside the file it points at. A concept written this way costs one file for a whole model, however many metrics the model holds.

**Stage two adds a surface**: a concept naming one metric and listing every request that metric accepts.

```yaml
metric: net_revenue_retention
dimensions: [account.segment, account.region, account.industry,
             date_dim.fiscal_quarter, date_dim.fiscal_year]
default_time_dimension: date_dim.fiscal_quarter
```

Nothing outside that list is available for that metric, so an agent chooses from it rather than working out what the model would allow, and a request naming anything else is refused. Three more optional keys, added to a stage one concept without changing a line of it.

## What it gives you

**Stage one:**

- **An agent can answer from the model, without writing a query.** A semantic model is machine-readable and self-describing, so an agent handed one and the prose around it names a metric and fields the model defines, with nothing in OKF enumerating the legal combinations.
- **A concept can say where its metric is computed.** Not a table and not a script, but the model that defines it, at the version somebody reviewed.
- **Prose can cite precisely.** A metric or field inside a model becomes addressable without being given its own file, which is what makes a model holding four hundred of them describable at all.
- **Definition drift becomes detectable.** A `semantic_model_version` that no longer matches says the definition moved after the prose was approved.

**Stage two, on top of it:**

- **The set is written down rather than worked out.** Choosing from an enumerated list is cheaper than deriving what the model allows, and much more likely to be correct.
- **Nothing is exposed by accident.** A bounded metric offers what its concept lists and nothing else, so adding fields to the model never widens what that metric can reach.
- **Wrong requests are refused rather than answered.** An unlisted dimension or an invented column fails validation instead of returning a plausible number.
- **The reasoning is recorded.** A consumer serves the concept's prose along with the list, so why the list is what it is, and which available groupings still mislead, reach the agent instead of remaining in one analyst's memory.

## Why two stages

**Stage one stands on its own.** A model-backed concept with no surface puts a reviewed, versioned description of the model where an agent will find it, and the agent can answer from it. A bundle that stops there is finished, not half-migrated.

**Stage two is strictly additive, and applies one metric at a time.** A stage one concept becomes a bounded one by gaining three keys, with no migration and no line rewritten, which §12 counts as a backward-compatible addition and a minor bump. So the two can be adopted on different schedules, and a model can carry surfaces for six metrics and none for the other four hundred.

**They cost different amounts to review.** Stage one is one file for a whole model. Stage two is one file per metric, and the list inside it is a claim somebody has to check by hand against the model. Describing a metric is cheap and approving one is not, and keeping those two claims apart is the point.

---

# Stage one: linking a model

**The first half of the proposed change.** Two optional frontmatter keys and one addressing rule. Examples use Apache Ossie (formerly OSI), a semantic model interchange format; the fragment vocabulary below is drawn from its object model, and nothing else here is specific to it.

## The model link

```yaml
type: Semantic Model
title: Revenue model
semantic_model: ../models/revenue.ossie.yaml
semantic_model_version: sha256:9f3c1a…
```

| Field | Required | Meaning |
| :---- | :---- | :---- |
| `semantic_model` | To be model-backed | Path to a semantic model artifact. It sits in the bundle like any other referenced file, and takes any of the three path shapes in §6.2. |
| `semantic_model_version` | With `semantic_model` | Content hash of the model as reviewed, so a consumer can tell whether the definition has changed since anyone approved it. |

A concept with neither is an ordinary OKF concept and behaves exactly as it does today.

**A known limitation: one model per file.** Ossie's `semantic_model` is a **list**, so one file may hold several. `semantic_model` names a file and offers no way to say which of them the concept describes, and neither does the fragment rule below, which addresses entities inside a model rather than choosing among models. So a model-backed concept assumes its target holds exactly one, and a file holding several is outside what this proposal addresses. Splitting it into one file per model is the workaround.

**Why not reuse `resource`.** §4.1 defines `resource` as identifying "the underlying asset the concept describes," and for a concept whose subject is the model, that URI would serve. Keeping `semantic_model` separate says the target is a semantic model rather than a table or a script, and leaves `resource` free to name the queryable asset, which is what a stage two concept needs it for. The later addition stays an addition rather than a rewrite.

**On `type`.** Nothing here keys off it. §4.1 says type values "are **not** registered centrally," §1 makes a fixed taxonomy a non-goal, and §11 forbids rejecting a bundle over an unknown one, so a consumer identifies a model-backed concept by the presence of `semantic_model`. As a convention only, `Semantic Model` suits a stage one concept and `Metric` suits a stage two one.

**When the hash no longer matches**, the concept still means something and the model link no longer does. A reviewer approved a definition, and a field it named may have been renamed or a relationship rewired, none of which is detectable from the concept alone. A stale link returns the concept to something you read rather than something you compute against, which needs no new field to express.

OKF does not define the model schema and does not generate queries. It assumes a system exists that can calculate a metric definition.

## Referring to a metric from prose

Prose has to name things: a metric in the model, a field it groups by, a neighbouring metric it is confused with. Stage one depends entirely on this working, so it is specified rather than left to convention.

### A fragment names an entity inside the target

**This is the one addition to OKF's addressing.** §6.2 gives three shapes for a path and says nothing about fragments, and §2 defines a concept ID as the file path minus `.md`, so today the file is the smallest addressable unit. This feature adds one rule:

> A `#` fragment on a path names an entity inside the target file, resolved in that file's own namespace.

**For a semantic model, the fragment names the kind of thing it points at.**

```text
#<kind>:<name>
```

```markdown
[`net_revenue_retention`](../models/revenue.ossie.yaml#metric:net_revenue_retention)
[`account.segment`](../models/revenue.ossie.yaml#field:account.segment)
[the account table](../models/revenue.ossie.yaml#dataset:account)
```

The five kinds are Ossie's five addressable objects.

| Kind | Name is | Example |
| :---- | :---- | :---- |
| `model` | A model name | `#model:revenue` |
| `dataset` | A dataset name | `#dataset:account` |
| `field` | `<dataset>.<field>` | `#field:account.segment` |
| `relationship` | A relationship name | `#relationship:account_to_owner` |
| `metric` | A metric name | `#metric:arr` |

**`field` covers dimensions, measures, and join keys alike**, because Ossie draws no structural line between them. All three are fields on a dataset, and the optional `dimension` block on a field carries a temporal role marker rather than marking a separate kind of object.

The kind tag is what makes the name unambiguous. A bare `account` could be a dataset, a relationship, or a metric, since each kind has its own namespace and nothing stops a name appearing in more than one. The tag says which to look in, and it fixes how many dot-separated parts to expect, which the parts themselves cannot carry.

The set is closed. A consumer has to recognise the token to resolve it, so adding a kind is a revision to this rule rather than something a producer decides locally. A fragment on a model file with no recognised kind tag does not resolve, which puts it under the tolerate rule below.

**For a concept, a fragment is an ordinary heading anchor.** No kind tag, so existing markdown renderers keep working and a link into a `.md` file behaves the way readers already expect.

```markdown
[NRR's caveats](/metrics/revenue-model.md#net_revenue_retention)
```

**The only hard requirement is to not break.** A consumer MUST tolerate a fragment it does not understand, which §6.1 already requires of broken links generally, and degrading to the whole file is the current behavior. Everything past that is SHOULD: a consumer that resolves the target and understands its type should resolve the fragment, and one building a graph should treat `model.yaml#metric:a` and `model.yaml#metric:b` as distinct nodes rather than collapsing both onto the file. A consumer that ignores fragments entirely stays conformant.

**Why this is safe.** The spec has no rule about `#` to conflict with. An unresolved fragment leaves a working whole-file link. Standard markdown carries fragments through renderers already. And §12 admits backward-compatible additions, which this is: every existing link keeps working unchanged.

**What it buys.** Without fragments, a metric inside a model is unreachable, and the only way to make one addressable is to give it its own concept file. That is a reasonable trade for a metric worth approving and a bad one for the other four hundred in a large model.

### Links to concepts and whole files

**Another metric with its own concept is a link to that concept.**

```markdown
Not to be confused with [gross revenue retention](/metrics/gross-retention.md),
which excludes expansion.
```

Bundle-absolute paths are the recommended form in §6.2 because they survive file moves. Per §6.1 the link is untyped, so the surrounding prose carries the relationship, and consumers "MUST tolerate broken links," so a link to a concept nobody has written yet is legal. Where the metric has no concept, link into the model with a fragment instead.

**The whole model file is still linkable** without a fragment, for when the model itself is the subject. §6.3's `references/` directory exists to hold code and external material, so a non-`.md` target is expected.

## What a stage one concept answers

**The model is the boundary.** A stage one concept declares no set of approved requests, and the consequence is not that nothing can be asked but that the model's own expressiveness decides. Ossie states the datasets, the relationships between them, the fields on each, and which of those carry a temporal role, so an agent holding the model and the concept's prose names a metric and fields the model supports, with nothing in OKF enumerating the legal combinations. The request it makes has the same shape a bounded concept would accept; what is missing is a list saying which of those requests anyone approved. That is the mode stage one delivers, and for most metrics in most models it is the only mode they will ever have.

**What it cannot do is refuse.** There is no declared set to check a request against, so every model-valid request is answerable, including the ones that are valid and wrong: a grouping that fans out and triples a total, a ratio averaged across periods. Prose describing a field as closed is prose a consumer may serve, not an obligation it must honor. Correctness rests on the model and on the agent's judgment, and the failure mode is a plausible wrong number rather than a refusal.

That is the gap stage two closes, one metric at a time.

---

# Stage two: declaring a surface

**The second half of the proposed change.** Everything here is additive: a stage one concept becomes a stage two concept by gaining three keys, and no existing line changes.

## How the two stages compare

|  | Stage one | Stage two |
| :---- | :---- | :---- |
| Frontmatter | `semantic_model`, `semantic_model_version` | plus `metric`, `dimensions` |
| Claim | This is what the model means | These requests are approved |
| Agent asks for | A metric and fields | A metric and fields |
| Agent gets | The model, and works the set out from it | A written list, with refusal outside it |
| Bounded by | The model's own expressiveness | An authored list |
| A wrong request yields | A plausible wrong number | A refusal |
| Review unit | One model | One metric |
| Cost to author | One file | One file per metric |

A concept that declares a surface is called a **bounded** concept below.

**Mixing is the normal case.** A stage one concept carries the orientation, grain rules, and cross-cutting caveats. Bounded concepts sit beside it for the metrics that matter enough to approve, and link back to it for the shared context rather than repeating it.

**Which one governs is decided per metric.**

- A metric with a bounded concept is answered from that concept's surface, and a request outside it is refused even though the model would permit it.
- A metric without one is answered from the model.

Scope is exactly one metric. Declaring a surface for `net_revenue_retention` does not close `arr` and does not close the model, which is what keeps adoption incremental: authoring the first surface in a bundle cannot make the bundle answer less than it did the day before.

**So a surface binds a cooperating consumer.** The request a bounded concept refuses can be put to the same model directly, and nothing here prevents that. A surface states which requests are meaningful; it is not an access control boundary, and a bundle is the wrong place to look for one. Nothing else in OKF enforces anything either: links break, types are ignorable, and §11 requires unrecognized fields to be tolerated rather than rejected.

The requests themselves are made at runtime. An agent makes one per question and a consumer answers it, and **no file contains one.**

## The surface

A metric's **surface** is the set of requests it accepts, and it is exactly what the concept lists. A request means one concrete ask: this metric, grouped by these dimensions, under these filters, the period among them.

```yaml
metric: net_revenue_retention
dimensions: [account.segment, account.region, account.industry, product.family,
             employee.team, date_dim.fiscal_quarter, date_dim.fiscal_year]
default_time_dimension: date_dim.fiscal_quarter
```

| Key | Required | Meaning |
| :---- | :---- | :---- |
| `metric` | To declare a surface | The name of one metric defined in the linked model |
| `dimensions` | With `metric` | Every dimension this metric may be grouped by, filtered on, or ordered by, time dimensions included |
| `default_time_dimension` | Optional | Which time dimension applies when the asker names none; one of `dimensions` |

The list is **exhaustive**. Anything absent is not available, so a model that gains a field exposes nothing until somebody adds it to a concept. Getting that wrong fails closed: a forgotten dimension is an omission from the list, not an exposure.

### What the list covers

**`dimensions` governs grouping, filtering, and ordering alike.** Reading it as a grouping list only is the one misreading with a real cost: leave `employee.full_name` out of the list but accept it as a filter, and a caller can still obtain per-individual figures one request at a time.

**Time dimensions are ordinary dimensions.** `date_dim.fiscal_quarter` is listed alongside `account.segment` and obeys the same rule. Grouping by it returns a quarterly series, filtering on it picks a period, and ordering by it sorts chronologically. There is nothing else to declare, because Ossie has no grain or hierarchy concept: it marks a field's time role with `is_time` and never says that months roll up into quarters.

**Nothing relative appears in a concept**, which lists fields and not a calendar. *Last quarter* becomes `FY26Q2` only once someone knows which quarter it is now.

**`default_time_dimension` exists because time is the one axis where saying nothing is dangerous.** Omit a dimension and you get a coarser answer; omit time and you get every period summed together, which double-counts an additive measure and produces nonsense for a ratio. A consumer identifies which dimensions are time dimensions by reading `is_time` from the model.

### One metric per bounded concept

Each metric with an approved surface gets its own file, and their surfaces differ:

```yaml
# metrics/net-retention.md
dimensions: [account.segment, …, date_dim.fiscal_quarter, date_dim.fiscal_year]

# metrics/arr.md
dimensions: [account.segment, …, date_dim.fiscal_month,
             date_dim.fiscal_quarter, date_dim.fiscal_year]
```

ARR is additive, so a monthly figure is meaningful. NRR is a ratio, so averaging it across months is not. One shared surface would have to be wrong for one of them. Omission does the work, which is why there is no separate mechanism for forbidding things: a grouping that should not be used is a grouping that is not listed.

**A surface is per-metric, so a bounded concept is per-metric**, for two reasons.

- **It is how OKF already works.** §4.1 defines `resource` as identifying "the underlying asset the concept describes," singular, and §2 defines a concept ID as the file path minus `.md`.
- **Review and prose get a scope.** NRR's caveats and ARR's caveats are different, and `verified` applies to a whole file. One file covering both means one sign-off covering both, and a consumer assembling what an agent sees for NRR would serve ARR's warnings alongside it.

**The cost is duplication.** `semantic_model` and `semantic_model_version` repeat in every concept drawn from the same model, so a model edit means updating the hash in several files. This is mechanical, since the hash can be recomputed and rewritten by tooling. Prose shared by several metrics is the harder case, and a stage one concept over the model is its natural home.

### Recording the reasoning

A list says what is offered and cannot say why. A reader sees that NRR has no `date_dim.fiscal_month` and cannot tell whether that was deliberate or an oversight, and nothing in the list warns that a legal request is usually the wrong one. Two things belong in the body every time:

- **Why an entry is missing**, so a refusal can be explained and the next author does not re-add it without realising.
- **Which available groupings still mislead**: the request that is legal, passes every check, and returns a wrong answer that looks authoritative. Nothing mechanical detects that one.

No new field is needed. A consumer serving the list should serve the prose with it.

### What the author is responsible for

Nothing derives this list. It is the author's claim about the model.

**Every entry must name something the model actually has.** A misspelled or deleted field is not caught when the concept is written. It fails when somebody asks for it.

**A dimension must be usable without fanning out.** Grouping by a field on the *many* side of a relationship multiplies rows: one account holding three subscriptions becomes three, and every account-level total triples. That returns a wrong number rather than an error, so it is the one mistake worth checking by hand before publishing.

**A time dimension must be a field the model exposes**, so a model with no date dimension offers none and its metrics may be asked for only as totals.

**A known limitation: `dataset.field` names a destination, not the path that reaches it.** Where a model reaches the same dataset more than one way, a date dimension joined on both sold date and renewal date, `date_dim.fiscal_quarter` is ambiguous and nothing in the entry resolves it. Naming a dimension in a request is the layer Ossie's roadmap claims and has not specified, so there is nothing to defer to yet.

Inside a bounded concept, a bare `account.segment` in prose joins into the concept's own `dimensions` list, the way §5.1's footnote labels join into `sources[].id`. A fragment remains the better choice where correctness matters, since a validator can resolve it.

## What a request looks like

**OKF does not define one.** A concept declares what a metric offers; turning a question into a request, and a request into a query, is the consumer's business. The shape below is illustrative.

```yaml
metric: net_revenue_retention
dimensions: [account.segment, date_dim.fiscal_quarter]
filters:
  - { dimension: account.region, op: in, values: [EMEA] }
  - { dimension: date_dim.fiscal_quarter, op: in, values: [FY26Q1, FY26Q2] }
order_by: { by: date_dim.fiscal_quarter, direction: asc }
```

One metric per request, because a request is for one number. Every entry under `dimensions` and every `dimension` under `filters` is drawn from the same list, so there is one rule to check rather than two.

**Stage one makes the same request**, in the same shape and the same names. The difference is what it is checked against: a bounded concept's list, or the model itself. In neither case does the agent compose SQL, invent a column, or add a join. Attested Computation draws the same boundary when it lets an agent supply parameter values but not edit the computation.

Leaving the request vocabulary undefined is deliberate. It is the part of this area that changes fastest, and the part Ossie's roadmap already claims.

## Field summary

**Stage one.**

| Field | Required | Kind |
| :---- | :---- | :---- |
| `semantic_model` | To be model-backed | Path |
| `semantic_model_version` | With `semantic_model` | Content hash |

Plus one addressing rule: **a `#` fragment on a path names an entity inside the target file.**

**Stage two.**

| Field | Required | Kind |
| :---- | :---- | :---- |
| `metric` | To declare a surface | Name of one metric in the model |
| `dimensions` | With `metric` | List of dimension references, time dimensions included |
| `default_time_dimension` | Optional | One entry from `dimensions` |

Five optional frontmatter keys and one addressing rule is the whole addition. Every key is optional and none of them is new required behavior, so a concept using none of them is a concept unchanged.

---

# E2E Example

Both stages end to end, so the stage one file can be read on its own and the stage two files show what approving a metric adds to it. The model is core Ossie with no `custom_extensions`.

### 1. The model

A few field objects are shown in full so the real shape is visible; the rest are named in comments. The `dimension` attribute carries only `is_time`, and Ossie's reference models use its presence to flag a field meant for grouping rather than aggregation, which is a convention rather than a schema rule. `is_time` is a role marker, not a granularity, and it defaults from `datatype`, so the `String` fiscal fields below have to set it explicitly while a `Date` column would not.

```yaml
version: "0.2.0.dev0"                           # the schema pins this exact string

semantic_model:
  - name: revenue
    description: Subscription revenue and retention

    datasets:
      - name: subscription                      # the fact table
        source: analytics.finance.subscription
        primary_key: [subscription_id]
        fields:
          - name: arr_amount
            expression: { dialects: [ { dialect: ANSI_SQL, expression: arr_amount } ] }
            datatype: Decimal                   # no dimension block: aggregated
          # arr_amount_prior:                   same shape
          # account_id, product_id, sold_date_sk:   keys, dimension: { is_time: false }

      - name: account
        source: analytics.crm.account
        primary_key: [account_id]
        fields:
          - name: segment
            expression: { dialects: [ { dialect: ANSI_SQL, expression: segment } ] }
            datatype: String
            dimension: {}
          # region, industry:                   same shape
          # owner_employee_id:                  key

      - name: date_dim
        source: analytics.common.date_dim
        primary_key: [date_sk]
        fields:
          - name: fiscal_quarter
            expression: { dialects: [ { dialect: ANSI_SQL, expression: fiscal_quarter } ] }
            datatype: String
            dimension: { is_time: true }        # String would default to false
          # fiscal_month, fiscal_year:          same shape, also is_time: true

      - name: employee
        source: analytics.hr.employee
        primary_key: [employee_id]
        # fields: team, full_name

      - name: product
        source: analytics.catalog.product
        primary_key: [product_id]
        # fields: family

    relationships:                              # from = many side, to = one side
      - { name: subscription_to_account, from: subscription, to: account,  from_columns: [account_id],        to_columns: [account_id] }
      - { name: subscription_to_product, from: subscription, to: product,  from_columns: [product_id],        to_columns: [product_id] }
      - { name: subscription_to_date,    from: subscription, to: date_dim, from_columns: [sold_date_sk],      to_columns: [date_sk] }
      - { name: account_to_owner,        from: account,      to: employee, from_columns: [owner_employee_id], to_columns: [employee_id] }

    metrics:
      - name: net_revenue_retention
        expression:
          dialects:
            - dialect: ANSI_SQL
              expression: SUM(subscription.arr_amount) / NULLIF(SUM(subscription.arr_amount_prior), 0)
        description: Retained and expanded ARR relative to the prior period
        datatype: Decimal
        ai_context:
          synonyms: ["NRR", "net dollar retention"]

      - name: arr
        expression:
          dialects: [ { dialect: ANSI_SQL, expression: SUM(subscription.arr_amount) } ]
        description: Annual recurring revenue
        datatype: Decimal

      # gross_revenue_retention:                same shape, excludes expansion
```

### 2. The concepts

```text
revenue-model.md              stage one: model link only
metrics/net-retention.md      stage two: metric: net_revenue_retention
metrics/arr.md                stage two: metric: arr
```

**`revenue-model.md`**, the stage one concept. It stands on its own: the model is answerable from it before either metric file below exists.

```markdown
---
type: Semantic Model
title: Revenue model
description: Orientation for the subscription revenue semantic model.
semantic_model: models/revenue.ossie.yaml
semantic_model_version: sha256:9f3c1a…
status: stable
verified: { by: human:ahormati, at: 2026-07-08T09:00:00Z }
---

One row per subscription. Accounts may hold several, so a grouping on
[`product.family`](models/revenue.ossie.yaml#field:product.family) splits an
account across products and per-product figures do not reconstruct the
account's own.

**Cohorts.** Retention is measured on the account cohort active at the start
of the period. Cohort construction changed in FY26Q1: accounts acquired
through the Meridian acquisition were previously excluded and are now
included from their contract start date. Figures either side of that boundary
are not like-for-like, and this applies to every metric in the model.

**Closed fields.**
[`employee.full_name`](models/revenue.ossie.yaml#field:employee.full_name)
exists in the model and is closed to reporting under privacy policy PRV-4. Use
[`employee.team`](models/revenue.ossie.yaml#field:employee.team).

**Metrics.** The two with approved surfaces are
[`net_revenue_retention`](metrics/net-retention.md) and
[`arr`](metrics/arr.md). Others in the model, including
[`gross_revenue_retention`](models/revenue.ossie.yaml#metric:gross_revenue_retention),
have no surface, so a request for one is checked against the model and the
caveats above are the only thing narrowing it.
```

`gross_revenue_retention` gets a link and a sentence without anyone writing it a concept, the link says exactly where it lives, and it can be asked for. That is stage one in one line: addressable, described, answerable, and unbounded.

**`metrics/net-retention.md`**, stage two. The first two frontmatter keys are the stage one pair unchanged; the surface is added below them.

```markdown
---
type: Metric
title: Net revenue retention
description: Expansion-inclusive retention for the subscription business.
semantic_model: ../models/revenue.ossie.yaml
semantic_model_version: sha256:9f3c1a…
metric: net_revenue_retention
dimensions: [account.segment, account.region, account.industry, product.family,
             employee.team, date_dim.fiscal_quarter, date_dim.fiscal_year]
default_time_dimension: date_dim.fiscal_quarter
status: stable
verified: { by: human:ahormati, at: 2026-07-08T09:00:00Z }
sources:
  - id: retention-standard
    resource: /domains/revenue/policies/retention-standard.md
    title: Retention reporting standard
---

Retention is measured on the account cohort active at the start of the
period, per the retention reporting standard.[^retention-standard] Cohort
construction, including the FY26Q1 Meridian boundary, is described in the
[revenue model](/domains/revenue/revenue-model.md).

**On time.** This metric offers `date_dim.fiscal_quarter` and
`date_dim.fiscal_year`, and deliberately not `date_dim.fiscal_month`: it is a
ratio and cannot be averaged across months, and contracts renew annually, so
monthly movement reflects renewal timing rather than performance.
[ARR](/domains/revenue/metrics/arr.md) is additive and does offer month.

**On owner-level groupings.** `employee.team` is offered and
`employee.full_name` is not; individual attribution is closed under privacy
policy PRV-4.

**On grouping by product family.** It is available, and it usually misleads.
Retention is an account-level measure and accounts commonly hold several
products, so per-product figures do not reconstruct the account's retention.
Prefer `account.segment` unless product-level attribution is what was asked
for. The same grouping on ARR is sound.

[^retention-standard]: Retention reporting standard
```

**`metrics/arr.md`**, frontmatter only. The surface differs by one entry, and that entry is the point.

```yaml
type: Metric
title: ARR
description: Annual recurring revenue for the subscription business.
semantic_model: ../models/revenue.ossie.yaml
semantic_model_version: sha256:9f3c1a…
metric: arr
dimensions: [account.segment, account.region, account.industry, product.family,
             employee.team, date_dim.fiscal_month, date_dim.fiscal_quarter,
             date_dim.fiscal_year]
default_time_dimension: date_dim.fiscal_quarter
```

### 3. What the agent sees

The menu is what stage two adds. With stage one alone, an agent gets `revenue-model.md`'s prose and the model itself, and works out an equivalent list for itself from the model's fields and relationships, with nothing marking which entries anyone approved.

```text
net_revenue_retention
  dimensions  account.segment, account.region, account.industry,
              product.family, employee.team,
              date_dim.fiscal_quarter (time, default), date_dim.fiscal_year (time)

arr
  dimensions  account.segment, account.region, account.industry,
              product.family, employee.team,
              date_dim.fiscal_month (time), date_dim.fiscal_quarter (time, default),
              date_dim.fiscal_year (time)

current     date_dim.fiscal_month   = FY26M09
            date_dim.fiscal_quarter = FY26Q3
            date_dim.fiscal_year    = FY26
```

**The menu spans concepts.** Each metric's list comes from its own file, and the consumer assembles them. The `default` marker comes from the concept too. The **`time` marker does not**: the consumer reads `is_time` from the model, or derives it from `datatype` where a field does not set one.

**`gross_revenue_retention` is absent from the menu**, which does not make it unanswerable. It makes it unbounded, and the model is what an agent asking for it has to work from.

**`current` comes from the consumer's own clock**, which is why it is not a bundle field: it would be out of date within a quarter. It is there so the agent can work out *last quarter* without a calendar of its own.

Each concept's body is served with its own metric, so the reason `fiscal_month` is unavailable for NRR reaches an agent working on NRR and not one working on ARR.

### 4. Requests accepted and rejected

| Request | Result |
| :---- | :---- |
| NRR by `account.segment`, fiscal quarter, EMEA | **Accepted** |
| NRR by `employee.full_name` | **Rejected**: not declared; the body cites privacy policy PRV-4 |
| NRR filtered to one `employee.full_name` | **Rejected**: filters are checked against the surface too |
| NRR by `date_dim.fiscal_month` | **Rejected**: not in NRR's `dimensions` |
| **ARR** by `date_dim.fiscal_month` | **Accepted**: ARR is additive, so month is listed |
| NRR by `product.family` | **Accepted**: declared, though the body warns it misleads |
| "What's our NRR?" | **Accepted** at `default_time_dimension`, stated in the answer |
| **GRR** by `employee.full_name` | **Accepted**: no surface bounds GRR, so the model governs |

The **month rows show why each metric needs its own surface**: the same field on the same fact table is right for one metric and wrong for the other. The **`employee.full_name` rejection** comes from omission alone, and the body is what turns that omission into an explanation. The **filter rejection** is the one most easily missed. And the **`product.family` request is accepted**, because it is a legal request that is usually the wrong one, and no mechanical check can tell the difference.

**The last row is what the open mode costs.** The field NRR refuses is available on any metric nobody has bounded, and the prose closing it is advice. That is the argument for bounding the metrics that matter, and it is why stage one does not claim to bound anything.

Three of those eight rows can only be understood from the body, and the FY26Q1 cohort discontinuity appears in no list and in no model. Yet it is the most likely reason an otherwise correct answer would mislead.
