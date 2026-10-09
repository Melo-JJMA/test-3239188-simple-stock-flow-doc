# Scope — Simple Stock Flow

> **Step 5 of 6 (part 2).** What is built and what is not.
> Source: [`spec/data-model.md`](../spec/data-model.md) (§12 "What is explicitly out", §11, §13; quotes translated) · Overview: [`overview.md`](overview.md).
> **Citations:** `§n`, `US-nn`, `DP-nn`, `T-nn`. **`[S-Cn]`** = assumption.

## 1. In scope

| Capability | Stories | Citation |
|---|---|---|
| Authentication by username and password, with two roles | US-01, US-02, US-03 | §2.5, §9.2 |
| Viewing the five seeded categories | US-04 | §2.1, §9.1 |
| Product management: create, edit, stock, soft delete, image | US-05 to US-10 | §2.2, §7.1 |
| Product search (partial text, category, active, paged) | US-06 | Q1 |
| Sale registration with lines and frozen values | US-11 | §2.3, §2.4 |
| Sales queries (detail and paged range) | US-12 | Q6, Q7 |
| Sales report by product over a range | US-13 | Q9, §11.1 |

## 2. Out of scope (decided by the model)

| Out | Why | Citation |
|---|---|---|
| **Customer or buyer** entity | The model records only the internal operator | §1, §7 |
| **Payments and payment methods** | There are no payments or cards | §7 |
| **Multi-currency** | Single-currency by construction | D-05 |
| **Extra product attributes** (description, SKU, code) | Closed: name, price, stock, category, image | DP-03 |
| **Category maintenance** (create, rename, delete) | Read-only seed data; if opened, §4.1 is revisited first | §2.1, §4.1 |
| **Audit columns** `created_at`/`updated_at` | No requirement; if one appears, it is a change log and goes back to the owner | §8 |
| **Report by seller** | It would cross personal data | DP-02 |
| **External analytics, export and anonymization** | There is no consumer | §7.1 |
| **Editing or deleting sales** | Immutable accounting record | §2.3, §7.1 |
| **Physical deletion of products** | Soft delete only | §2.2, ADR-003 |
| **Granting the `admin` role at runtime** | Provisioned by the deployment | DP-04, §13 D-3 |
| **API contract, signed business requirements, technical decisions and testing strategy** | They live in documents that are not delivered | §12 |

## 3. Decisions closed by the owner (not reopened)

| ID | Decision | Citation |
|---|---|---|
| DP-02 | The report is **not** broken down by seller | §7.1, §11 |
| DP-03 | The product has **only** name, price, stock, category and image | §1, §12 |
| DP-04 | Nobody grants `admin` at runtime; an admin creates sellers | §13 |
| H-1 | The report groups **by** the frozen value (several rows if there was a recategorization) | §11.1 |
| D-05 | Single currency; not reintroduced | §3 |

## 4. Pending items and gaps with an owner

| Item | Status | Citation |
|---|---|---|
| T-11 frozen `category_name` and report | Pending (status is inconsistent in the model, see closure H-05) | §3, §11.1 |
| T-12 `sold_by_user_id` + FK-4 and rename | Pending | §3, §5 |
| T-13 Three indexes, `pg_trgm`, drop redundant index | Pending | §6.2 |
| T-20 CHECKs for `price`, `quantity`, `category.name`, `role`, lowercase | Pending / not confirmed | §4, §13 |
| H-2 Cleanup of orphan binaries | No operational owner; no impact on the deliverable | §11 |
| Rewrite `spec.md` CA-06.1 ("one row per product") | **Decision pending with the owner** | §11.1 |
| DP-01 / A-7 Tie-break of the product name in the report | Measured defect; **not fixed in this batch** | §11.1 |

## 5. "Done" criterion **[S-C3]**

The model defines no delivery criterion. Proposed: the system is complete when **every story US-01…US-13 has its criteria verified** and **the queries of §10 return what §10 says** (NFR-15).

## 6. Assumptions

| ID | Assumption |
|---|---|
| **S-C3** | The "done" criterion in §5 is proposed, not from the model. |
