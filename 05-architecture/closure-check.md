# Architecture closure — verification against the model and against what was produced

> **Step 6 of 6.** We go back to [`architecture.md`](architecture.md) and check that it fits
> `01-context`, `02-domain`, `03-product`, `04-requirements` **and** [`spec/data-model.md`](../spec/data-model.md) (in Spanish; quotes are translated).
> **Method:** (1) every element of the model must land in domain, requirement and architecture block; (2) every architecture block must be traceable to the model; (3) the contradictions of the model itself are listed.
> There was no access to the engine: **nothing was measured**; everything is a reading of the document (§10 of the model describes how to measure).

## 1. Coverage: from the model to what was produced

| Model element | Domain | Requirement | Architecture block | Fits? |
|---|---|---|---|---|
| `category` (§2.1, §9.1) | `Category`, R-22 | US-04 | Category repository (read-only) | ✅ |
| `product` (§2.2) | `Product`, R-01…R-07 | US-05…US-10 | Product repository, catalog aggregate | ✅ |
| `sale` (§2.3) | `Sale`, R-08…R-11, R-17 | US-11, US-12 | Sale repository, sales aggregate | ✅ |
| `sale_item` (§2.4) | `SaleItem`, R-12…R-14 | US-11 | Internal to the `Sale` aggregate | ✅ |
| `user` (§2.5, §9.2) | `User`, R-18…R-21 | US-01…US-03 | User repository, hash port, *bootstrap* | ✅ |
| FK-1 | R-05 | AC-05.5, NFR-02 | Engine | ✅ |
| FK-2 | R-14 | NFR-02 | Engine | ✅ |
| FK-3 | R-16 | AC-09.2, AC-11.4, NFR-02 | Engine | ⚠ H-02 |
| FK-4 | R-17 | NFR-02 | Pending T-12 | ✅ declared |
| `ck_product_stock_non_negative` | R-02 | AC-08.3, NFR-01 | Engine (last barrier) | ✅ |
| `deleted_at` + global filter | R-06 | AC-09.1 | Adapter (shadow property, D-03) | ⚠ H-03 |
| `xmin` | — | AC-08.3 | Adapter (token, D-04) | ✅ |
| Q1 and Q3 | — | AC-06.x, AC-11.4 | Product repository | ✅ |
| Q6, Q7 | — | AC-12.1, AC-12.2 | Sale repository | ✅ |
| Q8 | — | AC-12.4 | **Removed from the port** | ✅ (R-4) |
| Q9 | R-25 | US-13 | Report read port | ✅ |
| Q10 | — | AC-01.1 | User repository | ✅ |
| Q2, Q4, Q5 | — | US-04, US-07 | Repositories | ✅ |
| `image_key` (D-08) | R-07 | US-10 | Image port | ✅ |
| Privacy and retention (§7) | R-20 | NFR-07…NFR-09 | Hash port; logs | ✅ |
| No audit (§8) | — | NFR-14 | No block (deliberate absence) | ✅ |
| Seed (§9) | R-22 | US-03, US-04 | Initial migration + *bootstrap* | ✅ |
| Verification (§10) | — | NFR-15 | Queries of §10 | ✅ |

**Inverse (architecture → model):** every block in [`architecture.md` §3](architecture.md#3-building-blocks-and-responsibilities) has a citation; the only ones without literal backing are those of assumptions S-A1…S-A5.

## 2. Findings: contradictions and doubtful status **in the model itself**

The model declares itself **signed** and states that its three debts (D-1, D-2, D-3 of §13) are **"settled on 2026-09-20"** with "the text corrected in place". On cross-checking, **leftovers remain**. Resolution rule applied: *the latest and most specific wins* (§13, 2026-09-20, over the measurement of §10, 2026-09-19); *if it contradicts the engine, the engine wins* (§Governed by) — **not verifiable here**.

| # | Where | What it says | What it clashes with | Treatment here |
|---|---|---|---|---|
| **H-01** | §3 heading and §12 vs. §10.1 and §3 (`xmin` note) | "The **22** columns" (§3, §10 intro, §12) versus "**21** rows" (§10.1, `xmin` note, §8) | 21 = columns measured on 2026-09-19; 22 = 21 + `deleted_at` (D-1) | Current count: **22** if `deleted_at` already exists. A number is avoided in the other documents |
| **H-02** | §3 (`sale_item.product_id`: "No foreign key today"), §12 ("FK-3 … pending"), §10.2 (8 constraints) | FK-3 does not exist | §5, §2.4 and §13 D-2: **FK-3 is in the engine** | **FK-3 is taken as engine** (§13) and the other mentions as leftovers |
| **H-03** | §6.2 ("Depends on T-09, which creates the column"), §6.3, §7 and §7.1 ("pending T-09") | Soft delete is pending | §2.2 and §13 D-1: `deleted_at` **exists**, T-09 done | Taken as **engine**; leftovers are in §6.2, §6.3, §7, §7.1 |
| **H-04** | §3 `sale_item.sale_id` ("yes — it is a defect", nullable) and §10.1 (`YES`) | `sale_id` nullable | §2.4, §4 and §13 D-2: it is `NOT NULL` | Taken as **`NOT NULL`**; §10 is an earlier snapshot |
| **H-05** | §3: the `category_name` row appears in the **`product`** table as "engine (T-11)", but its description speaks of "the label frozen at the instant of the sale"; §3 `sale_item` marks it "**pending** (T-11)" and §2.4 marks it "**engine** (T-11)" | Three different marks in two tables | §5 (N:M: `category_name` is `sale_item`'s own data), D-06, ADR-004 | **`category_name` belongs to `sale_item`.** Its status (engine vs. pending) **cannot be resolved** from the document: declared doubtful |
| **H-06** | Unique index `(sale_id, product_id)` with `INCLUDE` | §6.2 calls it **missing (T-13)**; §2.3, §4 and §13 call it **engine (T-20)** | Two tasks for the same object | Taken as **engine** (§13, later). It remains to clarify whether T-13 keeps the other two indexes and the drop of `IX_sale_item_sale_id` |
| **H-07** | §4 and §13 | T-20 pushes down "five CHECKs and one index"; §13 D-2 only settles `sale_id NOT NULL`, the unique index and FK-3 | The five CHECKs do not appear settled anywhere | Treated as **domain only, not confirmed** (S-D3) |
| **H-08** | §10.3 ("None is partial … the three of §6.2 are really missing") versus §13 (unique index with `INCLUDE` already exists) | Index status | Later §13 | §10 is historical; **T-13 partial** |
| **H-09** | §2.2 | `price > 0` is guarded only by `Product.ChangePrice`; `Money(0)` is valid | Nothing says that **creation** uses `ChangePrice` | **Risk for the architecture:** the creation path must go through the same guard. Added to R-2 |
| **H-10** | §11.1 | The report gives **several rows** per product after recategorization | `spec.md` CA-06.1 says "one row per product" (not delivered) | §11.1 is respected (AC-13.2) and flagged **pending with the owner** |
| **H-11** | §11.1 | Tie-break of the product name: measured defect **A-7**, "not fixed in this batch" | DP-01 | Declared in scope §4; no fix is designed |
| **H-12** | §0 ("the singular reaches the table, not the object code") | Naming convention | Could be read as a contradiction with "List attribute: never plural" | Resolved with the boundary §0 itself draws: plural in C# collections, singular in tables |

## 3. What changes in the architecture because of this closure

The five adjustments are applied in [`architecture.md`](architecture.md#11-adjustments-after-closure-v2):

1. Time-reading rule: **§13 rules over §10** (H-01 to H-04, H-08).
2. `category_name` lives in `sale_item` (H-05).
3. The five T-20 CHECKs stay **domain only, not confirmed** (H-07).
4. Risk R-2 widened to the creation path (H-09).
5. Unique index `(sale_id, product_id)` as **engine** (H-06).

## 4. Verdict

**The architecture fits the model and the four previous documents**: the five aggregates/tables, the ten access patterns, the four foreign keys and the three marks of each rule have a place and a requirement. **What does not fit is not in the architecture but in the model** (H-01 to H-08), and that is why it is declared instead of being "fixed" silently: the model is signed, and the project rule is that **if the document contradicts the engine, the engine wins**.

**What must be done before implementing** (none of these actions can be done without access to the engine):
1. Run the three queries of §10 and reconcile **H-01 to H-08**.
2. Have the owner resolve **H-10** (CA-06.1).
3. Decide **S-A3** (what happens on a concurrency conflict).
