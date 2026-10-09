# Product — Simple Stock Flow

> **Step 3 of 6.** The problem it solves and the product vision, rebuilt from the requirements and the model.
> Source: [`spec/data-model.md`](../spec/data-model.md) (in Spanish; quotes are translated) · Requirements: [`04-requirements/requirements.md`](../04-requirements/requirements.md).
> **Citations:** `§n`, `US-nn`, `NFR-nn`, `DP-nn`. **`[S-Pn]`** = assumption (see [§7](#7-assumptions)).
> The model **contains no product text**; everything below is what its decisions allow us to infer.

## 1. The problem

Anyone who sells a small catalog of products needs to answer three questions without spreadsheets or memory:

1. **What do I have and how much?** A catalog with reliable price and stock (§1 Product, Stock).
2. **What did I sell, when, and who recorded it?** A sales record that cannot be altered (§1 Sale; §7.1).
3. **How did I do?** A report by product over a date range (§1 Sales report; Q9).

**What breaks without the system** *(inferred from the rules the model works hard to protect)*:
- Selling more units than exist → that is why `stock >= 0` is in the engine (§2.2, ADR-002).
- A price or name change rewriting the past → that is why the sale **freezes** name, price and category (§1 "Frozen name"; D-06).
- A report that was already read changing when new sales arrive → an explicit owner decision (§11.1).

**Business context [S-P1].** The five seeded categories — General, Tools, Electricity, Plumbing, Paints (§9.1) — suggest a **hardware or supplies shop**. This is an assumption: the model does not name the trade.

## 2. Users

| User | What they need | Citation |
|---|---|---|
| **Seller** | Register sales quickly and look up the catalog | §1 User, US-11, US-06 |
| **Administrator** | Maintain catalog and stock, create sellers, read the report | US-02, US-05 to US-10, US-13 *(permissions: S-R1)* |

The buyer is **neither a user nor data** in the system (§1, §7): the product is an **internal** tool.

## 3. Value proposition

> **A simple stock flow: catalog → sale → report, where stock never lies and the past never changes.**

| Promise | How the model supports it |
|---|---|
| **Stock never lies** | `stock >= 0` in the engine + optimistic concurrency (NFR-01) |
| **The past never changes** | Immutable sales, lines with frozen values, soft delete of products (NFR-03) |
| **Simple** | Five entities, no currency, no customer, no audit, no category CRUD (DP-03, D-05, §8, §4.1) |
| **Stable report** | Groups by the frozen value and is computed in the engine (§11.1, D-06) |

## 4. Vision

A small, verifiable system: **five tables, three aggregates, one report**. Its quality is measured by every rule having a declared place — *engine*, *domain only* or *pending* — and not by the number of features (§How to read this document).

**Product principles** *(derived from model decisions)*:
1. **Minimal, closed scope.** What was not requested is not built: no extra product attributes (DP-03), no currency (D-05), no audit (§8).
2. **The engine has the last word.** If the document contradicts the engine, the engine wins (§Governed by; §10).
3. **Declare the debt, do not hide it.** Anything not implemented is marked pending with its task (§How to read this document; §13).
4. **Privacy by omission.** No customer data; the report does not identify people (DP-02, §7).

## 5. What this product is not

| It is not | Citation |
|---|---|
| A sales system with customers or buyers | §1 |
| A payments system (no cards or payment methods) | §7 |
| A multi-currency system | D-05 |
| A category manager | §2.1, §4.1 |
| An analytics tool with a breakdown by seller or exports | DP-02, §7.1 |
| A change-audit system | §8 |

## 6. Success indicators **[S-P2]**

The model defines no metrics. The following are proposed, tied to verifiable requirements:
- **0** sales that leave negative stock (NFR-01).
- **0** sale lines whose name/price differ from those in force at the moment of the sale (AC-11.6).
- The report of a closed range returns the **same result** before and after new sales (AC-13.5).
- Product search and report respond within a threshold to be agreed with the owner (NFR-10).

## 7. Assumptions

| ID | Assumption |
|---|---|
| **S-P1** | The target business is a hardware/supplies shop (from the names of the seeded categories). |
| **S-P2** | Success indicators are proposed; the model neither sets them nor gives performance thresholds. |
| **S-P3** | The product is operated by a single shop (not multi-tenant): the model has no organization or store entity. |
