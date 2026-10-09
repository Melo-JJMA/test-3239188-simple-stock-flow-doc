# Overview — Simple Stock Flow

> **Step 5 of 6.** What the system is and the environment it lives in. Scope is in [`scope.md`](scope.md).
> Source: [`spec/data-model.md`](../spec/data-model.md) (in Spanish; quotes are translated) · Domain: [`02-domain/domain.md`](../02-domain/domain.md).
> **Citations:** `§n`, `US-nn`. **`[S-Cn]`** = assumption (see [§6](#6-assumptions)).

## 1. What it is

**Simple Stock Flow** is an internal system to keep a **catalog with stock**, **register immutable sales** and **obtain a sales report by product** (§1, Q9).
It is operated by internal users with two roles, `admin` and `seller`; there is **no customer or buyer** entity (§1, §2.5).

## 2. What makes it distinctive

| Trait | Citation |
|---|---|
| Only **five** entities and five tables | §2 |
| Sales **freeze** the name, price and category of the moment | §1, §2.4 |
| Stock **cannot** go negative: the engine guarantees it | §2.2, ADR-002 |
| Every rule declares where it lives: **engine**, **domain only** or **pending** | §How to read this document |
| One currency, no column-based audit, no category CRUD | D-05, §8, §4.1 |

## 3. Actors and systems

| Element | Role | Citation |
|---|---|---|
| Administrator | Maintains catalog and users, reads the report *(S-R1)* | §2.5, §13 |
| Seller | Registers sales | §1 |
| System API | Exposes the functions to operators (contract outside the model) | §12 |
| PostgreSQL 16 | Stores the model (database `simple_stock_flow`, schema `sales`) | §Verified against |
| External image store | Keeps the binary; the system keeps only the key | D-08, §7.1 |
| Deployment environment | Supplies the initial admin's credentials | §9.2 |

## 4. Known technical environment

- **Language and data access:** C# with EF (domain classes, `DbSet<...>`, migrations) (§0, §3.2).
- **Repositories of the original project:** `simple-stock-flow-api` (code), `simple-stock-flow-infra` (database container), `simple-stock-flow-docs` (documentation) (§10, §12).
- **Database:** PostgreSQL 16.14 in container `simple-stock-flow-db-1`, server in UTC (§Verified against).
- **Implementation state according to the model** (2026-09-19/20): `category` = 5 rows (seed), `user` = 1 (start-up admin), `product`/`sale`/`sale_item` empty (§10.4).

## 5. Documents of the original project that are **not** delivered

`constitution.md`, `spec.md`, `plan.md`, the original `architecture.md`, `tasks.md`, `api-contract.md`, `adr/` and `traspaso/HANDOFF-TECNICO.md` (§Governed by, §12, §11.1).
The model cites their identifiers (D-01…D-10, ADR-001…004, T-xx, A-n, articles), but **their content is not a source** for this documentation.

## 6. Assumptions

| ID | Assumption |
|---|---|
| **S-C1** | The system is for **internal** use by a single shop; there is no organization entity. (Consistent with S-P3.) |
| **S-C2** | The documentation produced here describes the system **as the model declares it**, not the real code, which was not delivered. |
