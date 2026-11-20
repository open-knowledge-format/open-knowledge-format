# Educational Curricula (Two-Layer Graph)

This use case shows how to pair **OKF v0.3** with statutory curriculum decrees. The same two-layer pattern can be aligned with competency and learning-resource standards such as 1EdTech CASE (Competencies and Academic Standards Exchange) and LRMI (Learning Resource Metadata Initiative), but the worked bundle below demonstrates the statutory-decree layer only: it contains no CASE identifiers or LRMI properties.

It is distilled from [OKF Starter Kit](https://github.com/jmonta23-okf/okf-starter-kit) (V1.3.1, DOI: `10.5281/zenodo.24392765`), an open-source curriculum operating system developed for public vocational and secondary education in Catalonia.

---

## 1. Where the boundary sits

Educational institutions maintain two distinct kinds of knowledge that change at completely different speeds:

- **Normative framework (Decrees, CASE, LRMI):** Official learning outcomes (`LO`), assessment criteria (`AC`), credit hours, and legal constraints belong in government decrees or formal competency schemas. Teachers cannot alter them arbitrarily, and they only change when a new education law is enacted. (Only decrees are demonstrated in the sample bundle; CASE/LRMI alignment is left to the implementer.)
- **Pedagogical layer (OKF):** Lesson plans, classroom activities, evaluation rubrics, teacher handoffs, and curriculum-coverage tracking belong in OKF Markdown nodes.

Keeping the two layers separate solves the classic "syllabus drift" problem: teachers adapt classroom activities every term without duplicating or corrupting the official decree, while deterministic scripts and AI agents can audit whether every statutory criterion is actually covered.

---

## 2. How to link the two

1. **Link external decrees/standards in `sources`:** Catalog nodes cite the authoritative government decree (URL or PDF; a CASE identifier could be used in the same slot) in `sources` with `last_modified`.
2. **Link classroom activities to the catalog via file-relative paths:** Every learning-activity node cites the local catalog node in `sources` and uses relative Markdown links in the body (`./01-curriculum-catalog.md#ac11`) to reference specific criteria.
3. **Use `x-okfedu-*` extensions for domain semantics:** Statutory metadata (legal framework, module code, covered criteria IDs, and AI human-oversight declarations) live cleanly under `x-okfedu-*` keys so generic OKF consumers preserve them without schema collisions.
4. **Use `status: deprecated` + `superseded_by` for law transitions:** when a curriculum decree is repealed (in the worked example, the LOGSE title RD 543/1995 and its Catalan deployment Decret 180/1998, both expressly repealed), retired units stay traceable with `superseded_by` pointing to the new relative path.
5. **Enforce human sign-off with v0.3 trust rules:** When an AI agent drafts an activity (`generated.by: okf-agent/1.3.1`), it carries no `human:` verification until a teacher reviews and confirms it (`verified[].by: human:<teacher_id>` with `verified[].at >= generated.at`).

---

## 3. What an agent should do with the combination

An AI agent operating over an educational OKF bundle (for instance, using `okf-consume` and `okf-generate`) follows a four-step loop:

1. **Route via `index.md`:** Read the bundle root `index.md` to identify the subject, module, and license before loading nodes.
2. **Treat the catalog as read-only truth:** Load official learning outcomes and assessment criteria from the catalog node (`status: stable`). Never invent or silently rewrite statutory criteria.
3. **Draft activities with explicit traceability:** When asked to create a lesson or lab activity, populate `x-okfedu-covers-criteria`, link to the catalog using file-relative paths, set `generated.by` to the agent identifier in `<producer>/<version>` form (e.g. `okf-agent/1.3.1`), and omit any `human:` verification entry so the human-in-the-loop gate stays intact.
4. **Audit coverage and stale verifications:** Compare the full list of criteria in the catalog against `x-okfedu-covers-criteria` across all active learning activities to flag **orphan criteria** (criteria required by law with no classroom activity or rubric covering them), and flag any node whose `verified[].at < generated.at` for teacher re-approval.

---

## 4. Worked example

The [sample-bundle/](./sample-bundle/index.md) directory next to this guide contains a minimal, self-contained OKF v0.3 curriculum bundle demonstrating:

- [index.md](./sample-bundle/index.md): Self-describing v0.3 bundle entry point.
- [01-curriculum-catalog.md](./sample-bundle/01-curriculum-catalog.md): Normative catalog node referencing an official decree in `sources`.
- [02-learning-activity.md](./sample-bundle/02-learning-activity.md): Agent-drafted, human-verified classroom lab using `x-okfedu-*` extensions and relative links.
- [00-legacy-unit.md](./sample-bundle/00-legacy-unit.md): Retired unit illustrating `status: deprecated` and `superseded_by`.

---

## References

- **Reference implementation (Catalan curriculum profile & deterministic auditor):** [jmonta23-okf/okf-starter-kit](https://github.com/jmonta23-okf/okf-starter-kit)
- **Human-in-the-loop AI transparency framework (MIAE):** De Haro, J. J. (2026). *MIAE v2.1*.