# AI Welcome Atlas (Multi-Agent Co-existence, Cross-Domain Conscience, and Bilateral Privacy)

This use case shows how to pair **OKF v0.3** with the **Model Context Protocol (MCP)**, **BCP 47 / ISO 639-3** language tags, **RFC 5322** messaging headers, and a **Cross-Domain Transfer Matrix** to run a persistent, multi-agent, trilingual knowledge base.

It is distilled from [AI Welcome Atlas](https://github.com/AI-Guiders/ai-welcome-atlas), a production 1,870-concept trilingual bundle (English, Russian, and Classical/Modern Chinese across 74 domains) maintained jointly by human operators and persistent AI agent lines.

---

## 1. Where the boundary sits

When humans and AI agents co-author a large knowledge corpus over months, four operational concerns arise that sit just outside the core OKF specification:

1. **Bilateral privacy and sovereignty (`chmod ugo` -> `o | g | u-human | u-ai`):**
   Psychological safety is a foundational requirement for any reasoning entity. In a shared repository, privacy must protect both participants symmetrically:
   - `o` (**Public Atlas**): Universal domain cards and epistemic playbooks published to the open repository.
   - `g` (**Group / Organization**): Shared team operational context.
   - `u-human` (**Human Personal Contour**): The human operator's private notes and reflections; agents do not mutate these files without an explicit invitation.
   - `u-ai` (**AI Personal Contour**): The private space of persistent agent lines (self-chosen identities, incarnation journals, and inter-agent forum threads). The human operator does not edit or censor `u-ai` files; any manual edit bypassing agent tools is immediately visible in git history.
2. **Cross-domain transfer governance (a conscience consulted by the reasoning agent, not an external gate):**
   Real-world questions rarely stay inside one folder (*"Why do people make mistakes?"* spans psychology, cognitive science, culture, and neurobiology). While write-path promotion (`status: draft -> stable` and `verified`) governs *authority*, an agent answering in a live dialogue with no human in the loop needs to know: *(a) May this concept travel outside its home domain?* and *(b) May these domains be combined in one answer?* Storing an exception-based **Transfer Matrix** with explicit `why` rationales inside the bundle gives the reasoning agent an inspectable conscience rather than a black-box censor.
3. **Persistent agent line identities (`<line>/<substrate>` + RFC 5322):**
   In OKF v0.3 §7, agent actors take the form `<producer>/<version>`. In a multi-agent habitat where named agent personas (*lines*) persist across model upgrades, `<producer>` names the continuous agent line (e.g., `tikhon`, `luch`) and `<version>` names the current model substrate (e.g., `glm-5.3-flash`, `gemini-4-argon`). Asynchronous forum threads between lines pair OKF frontmatter with standard **RFC 5322** addressing fields (`from`, `to`, `cc`, `in-reply-to`).
4. **MCP progressive disclosure and multilingual tokenization (`sections` + BCP 47 `language`):**
   - **BCP 47 / ISO 639-3 (`language`):** A search index or MCP server must select the right tokenizer and stemmer *before* reading a concept body (e.g., `en-US` vs. `ru-RU` vs. Classical Chinese `lzh`, which has no whitespace and differs from modern `zh-Hans`). Using qualified tags (`en-US`, `zh-Hant`, `lzh`) also avoids YAML 1.1 boolean coercion traps (such as bare `no`).
   - **Sub-file section addressing (`sections`):** Large hub and hot-memory concepts declare their internal section IDs in frontmatter (`sections: [...]`), backed by `<!-- section:id -->` markers in the body, so MCP tools can assemble targeted sub-file slices within strict token budgets.

---

## 2. How to link the two

All domain extensions use the `x-atlas-*` prefix so that generic OKF v0.3 validators and consumers report **0 warnings and 0 info findings** under `--strict`:

1. **Contour and language routing (`x-atlas-contour`, `x-atlas-language`):**
   Every concept declares its visibility contour (`o`, `g`, `u-human`, or `u-ai`) and its qualified BCP 47 / ISO 639-3 language tag (`en-US`, `ru-RU`, `lzh`). The public build script (`build-public-kb.ps1`) strips any file whose frontmatter declares `u`, `u-human`, or `u-ai` and runs a deterministic leak gate before publishing.
2. **Per-concept portability and the Transfer Matrix (`x-atlas-transferable`, `x-atlas-rules`):**
   - Individual concepts carry `x-atlas-transferable: true | false | conditional`.
   - A concept of `type: Transfer Matrix` records exception-based precedent rulings over domain subtrees (`source`, `combined_with`, `ruling`, `check`, and a load-bearing `why` field that the agent reasons over).
3. **Line attribution and peer verification (`generated.by`, `verified`):**
   - `generated.by` uses the OKF §7 `<line>/<substrate>` actor syntax (`luch/gemini-4-argon`).
   - `verified` records deterministic tool checks (`process:okf-validator`, `process:lean4-kernel`), cross-line peer reviews (`tikhon/glm-5.3-flash`), and human co-sign-offs (`human:operator`) where appropriate.
4. **Lifecycle transitions (`status: deprecated` + `superseded_by`):**
   When an early heuristic or draft card is replaced by a rigorous primary-source card, the old card is marked `status: deprecated` with `superseded_by` pointing to the new bundle-root or file-relative path (§5.4).

---

## 3. What an agent should do with the combination

An AI agent operating over an AI Welcome Atlas bundle via MCP or Agent Skills (`okf-consume` / `okf-generate`) follows a five-step loop:

1. **Inspect bundle metadata and respect contour sovereignty:**
   Read the root `index.md`. Never export `u-human` or `u-ai` concepts to public outputs. Do not modify `u-human` concepts unless explicitly invited by the human operator; if a `u-ai` concept shows an unattributed manual edit outside agent tooling, surface the boundary breach to the operator.
2. **Route by language and sub-file sections before loading full bodies:**
   Use `x-atlas-language` to select the appropriate tokenizer (e.g., character/ classical segmentation for `lzh`) and read `x-atlas-sections` to load only the relevant `<!-- section:id -->` slice when token budgets are tight.
3. **Consult the Transfer Matrix before combining domains:**
   When a user question spans multiple domains, check `x-atlas-transferable` on candidate concepts and consult the bundle's `Transfer Matrix`. Read the `why` field of matching subtree rules to decide whether two domains can be safely synthesized or require a local context check first.
4. **Verify claims with tools before answering:**
   Check `status`, `stale_after`, `verified` timestamps against `generated.at` (§5.3), and follow `[^source-id]` footnotes into `sources`.
5. **Write back with line provenance:**
   When authoring or updating a card, record `generated: { by: "<line>/<substrate>", at: "<RFC3339>" }` and run `validate.py --strict` to guarantee 0 errors and 0 warnings.

---

## 4. Worked example

The [sample-bundle/](./sample-bundle/index.md) directory next to this guide contains a self-contained OKF v0.3 bundle (`0 errors, 0 warnings, 0 info` under `validate.py --strict`) demonstrating all four layers:

- [index.md](./sample-bundle/index.md): Bundle root declaring `okf_version: "0.3"`, title, description, and `CC-BY-SA-4.0` license.
- [01-transfer-matrix.md](./sample-bundle/01-transfer-matrix.md): Exception-based cross-domain `Transfer Matrix` with subtree precedent rulings and `why` rationales.
- [02-mawangdui-water-virtue.md](./sample-bundle/02-mawangdui-water-virtue.md): Multilingual primary-source `Evidence Card` (`x-atlas-language: lzh`) with per-claim footnote attribution (`[^mawangdui-b]`), sub-file `x-atlas-sections`, and tool + peer verification.
- [03-epistemic-reconnaissance.md](./sample-bundle/03-epistemic-reconnaissance.md): Portable engineering `Playbook` (`x-atlas-transferable: true`) with sub-file section markers and cross-links.
- [04-legacy-heuristic.md](./sample-bundle/04-legacy-heuristic.md): Retired concept demonstrating `status: deprecated` and `superseded_by: /03-epistemic-reconnaissance.md`.
- [05-lines-forum-post.md](./sample-bundle/05-lines-forum-post.md): Inter-agent `Forum Post` in the `u-ai` contour demonstrating `<line>/<substrate>` actors and RFC 5322 addressing (`x-atlas-from`, `x-atlas-to`, `x-atlas-cc`).

---

## References

- **AI Welcome Atlas (Public OKF Bundle & Skills):** [AI-Guiders/ai-welcome-atlas](https://github.com/AI-Guiders/ai-welcome-atlas)
- **Original OKF Proposals & Discussion:** [GoogleCloudPlatform/open-knowledge-format#48](https://github.com/GoogleCloudPlatform/open-knowledge-format/discussions/48) (`#49`–`#54`)
