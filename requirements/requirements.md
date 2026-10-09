# Requirements — Simple Stock Flow

> **Step 2 of 6.** User stories and non-functional requirements that **the data model makes necessary**.
> Source: [`spec/data-model.md`](../spec/data-model.md) (in Spanish; quotes are translated). Starting architecture: [`05-architecture/architecture.md`](../05-architecture/architecture.md).
> **Citations:** `§n`, `Qn` (access pattern of §6.1), `FK-n`, `DP-nn`, `T-nn`. **`[S-Rn]`** = assumption (see [§5](#5-assumptions)).
> **Inclusion criterion:** a requirement is included only if a table, column, rule, index or access pattern demands it. Nothing else (DP-03: "no invented scope").
> Rule marks used by the model: **engine**, **domain only**, **pending (T-xx)**.

## 1. Actors

| Actor | Who | Citation |
|---|---|---|
| **Administrator** (`admin`) | Internal operator with the `admin` role. Creates sellers | §2.5, §13 (DP-04) |
| **Seller** (`seller`) | Internal operator with the `seller` role. Registers sales | §1 (User), §2.5 |
| **System** | Performs start-up and creates the initial admin | §9.2 |

There is no customer or buyer, neither as an actor nor as data (§1 "there is no customer or buyer entity", §7).
The set of roles is **closed** at two values (§2.5).

## 2. User stories

### Identity

**US-01 · Log in**
*As an operator I want to authenticate with username and password so that I can use the system.*
- AC-01.1 User lookup is by **exact** name (Q10, high frequency: "at every login").
- AC-01.2 The username is normalized (lowercase, trimmed) before lookup (§2.5: without this, `"Ana "` would register an account that could never log in).
- AC-01.3 The password is verified only against `password_hash` through the hash port; the clear-text password never reaches the domain (§2.5, D-09).
- AC-01.4 Neither the password nor its hash ever appears in logs, responses or error messages (§7).

**US-02 · Create sellers**
*As an administrator I want to create seller accounts so that they can operate the system.*
- AC-02.1 The username is mandatory, unique and stored normalized (§2.5).
- AC-02.2 The assigned role belongs to `{admin, seller}` (§2.5).
- AC-02.3 **Nobody grants the `admin` role at runtime**: the administrator only creates sellers; the `admin` role is provisioned by the deployment (§11 H-3, §13 D-3, DP-04).
- AC-02.4 No token → 401; with the `seller` role → 403 (§13 D-3).

**US-03 · Have an initial administrator**
*As the person responsible for deployment I want the system to create the first administrator at start-up, with environment credentials.*
- AC-03.1 The SQL seed does **not** create users; the application start-up does (§9.2).
- AC-03.2 The credential is not versioned in the repository (§9.2, article IX).

### Catalog

**US-04 · View categories**
*As an operator I want to see the categories to classify and filter products.*
- AC-04.1 There are exactly **five**, seeded: General, Herramientas (Tools), Electricidad (Electricity), Fontanería (Plumbing), Pinturas (Paints) (§9.1).
- AC-04.2 They are listed sorted by name (Q4).
- AC-04.3 There is **no create, rename or delete** of categories (§2.1, §4.1).

**US-05 · Create a product**
*As an administrator I want to add a product so that it can be sold.* **[S-R1]**
- AC-05.1 Attributes: **name, price, stock, category and optional image, and nothing else** (§1, DP-03).
- AC-05.2 Name mandatory, non-empty and trimmed (§2.2).
- AC-05.3 Price **strictly positive**, with 2 decimals (§2.2, §3).
- AC-05.4 Stock **non-negative** (§2.2).
- AC-05.5 Category **mandatory and existing** (§2.2, FK-1).

**US-06 · Search products**
*As an operator I want to search products so that I can find them quickly.*
- AC-06.1 Filters by **partial text** of the name and by **category** (Q1).
- AC-06.2 Returns only **active** products (Q1, §2.2 soft delete).
- AC-06.3 Sorted by name, **paged**, with total item count (Q1: "an additional count query").

**US-07 · Edit a product**
*As an administrator I want to change name, price and category.* **[S-R1]**
- AC-07.1 The rules of US-05 still hold after every change (§2.2).
- AC-07.2 Changing name or price **does not alter** reports of closed periods (§1 "Frozen name").

**US-08 · Adjust stock**
*As an administrator I want to restock.* **[S-R1]**
- AC-08.1 `Restock` adds units; stock never goes negative (§2.2).
- AC-08.2 Withdrawing more stock than available **fails** (§2.2, `Product.Withdraw`).
- AC-08.3 Two concurrent operations on the same product cannot leave negative stock (ADR-002, D-04, `ck_product_stock_non_negative`).

**US-09 · Discontinue a product**
*As an administrator I want to remove a product from the catalog without losing its history.* **[S-R1]**
- AC-09.1 **Soft delete** (`deleted_at`); never a physical delete (§2.2, §7.1, ADR-003).
- AC-09.2 A discontinued product no longer appears in searches (Q1) and cannot be sold (§5).
- AC-09.3 Past sales stay intact (§7.1).

**US-10 · Manage a product image**
*As an administrator I want to attach or remove an image.* **[S-R1]**
- AC-10.1 The product stores an **opaque key**, never a path or a binary (D-08).
- AC-10.2 No image = `NULL`, never an empty string (§1, §2.2).
- AC-10.3 On replace or discontinue: first the key is nulled and committed; **then** the binary is deleted (§7.1).

### Sales

**US-11 · Register a sale**
*As a seller I want to register a sale with several lines to withdraw stock and keep a record.*
- AC-11.1 The sale records **who** made it and **when** (`sold_by`, `sold_at`) (§1, §2.3).
- AC-11.2 It must have **at least one line** (§2.3).
- AC-11.3 A product **cannot repeat** within the same sale (§2.3, unique index `(sale_id, product_id)`).
- AC-11.4 Each line: **active** product, **strictly positive** quantity (§2.4, §5).
- AC-11.5 Adding the line and withdrawing stock are **a single operation**; if stock is insufficient, it fails (§2.3, §2.2).
- AC-11.6 The line **freezes** the product name, unit price and category name at that moment (§1, §2.4, D-06).
- AC-11.7 The sale **total** and line **subtotal** are **computed**, not stored (§1, article VII).
- AC-11.8 Once registered, the sale is **neither edited nor deleted** (§1, §2.3, §7.1).

**US-12 · View sales**
*As an operator I want to see a sale with its lines and to list sales by date range.*
- AC-12.1 Detail of one sale with all its lines (Q6).
- AC-12.2 Listing by range, most recent first, **paged** with total (Q7).
- AC-12.3 The range is invalid if the end is before the start (§1, Date range).
- AC-12.4 The **unpaged** listing (Q8) is not offered: it "is redundant and should be removed from the port" (§6.1).

**US-13 · Sales report by product**
*As an administrator I want a report aggregated by product over a date range.* **[S-R1]**
- AC-13.1 Computed **in the engine**, not persisted (§1, D-06, Q9).
- AC-13.2 Groups by **frozen** `product_id, product_name, category_name`; if there was a recategorization within the range, **several rows** of the same product appear (§11.1).
- AC-13.3 Sorted by amount descending; no paging (Q9).
- AC-13.4 **Not broken down by seller** (DP-02, §7.1).
- AC-13.5 A closed report **does not change** when new sales are registered (§11.1).
- ⚠ AC-13.2 contradicts the "one row per product" criterion of `spec.md` CA-06.1; **decision pending with the owner** (§11.1).

## 3. Non-functional requirements

| ID | Requirement | Citation |
|---|---|---|
| **NFR-01** Stock integrity | `stock >= 0` guaranteed by the **engine** as last barrier, with optimistic concurrency | §2.2, ADR-002, D-04 |
| **NFR-02** Referential integrity | FK-1 `RESTRICT`, FK-2 `CASCADE`, FK-3 `RESTRICT`; FK-4 `RESTRICT` **pending** (T-12). `ON UPDATE NO ACTION` on all | §5 |
| **NFR-03** Immutability and history | Sales and lines are never deleted or edited; products only soft-deleted | §7.1 |
| **NFR-04** Monetary accuracy | `numeric(18,2)`; rounding to 2 decimals with `AwayFromZero` in `Money`; both must change together | §2.2, §3 |
| **NFR-05** Single currency | No currency column; not reintroduced | §3, D-05 |
| **NFR-06** Time | All timestamps are `timestamptz`; server in UTC | §3 |
| **NFR-07** Credential security | `password_hash` never in logs, responses, projections or errors; **never indexed** | §7, §6.3 |
| **NFR-08** Minimal privacy | `username` and `sold_by` are personal data (restricted access); no end-customer, payment or health data | §7 |
| **NFR-09** Retention | Sales: indefinite; hash: no history; image binary: the only data that is physically deleted | §7.1 |
| **NFR-10** Performance | Q1, Q3, Q7, Q9 and Q10 are **high** frequency; Q9 is "the most expensive". Partial search with trigrams, unique index with `INCLUDE` for the report | §6.1, §6.2 (T-13) |
| **NFR-11** Schema under migrations | All DDL goes through EF migrations; no other piece defines it | §3.2, §6.2 (ADR-001) |
| **NFR-12** Naming convention | Tables and attributes in English, singular, ASCII; `CHECK` named `ck_{table}_{rule}` | §0, §3.1 |
| **NFR-13** No engine defaults | Values are set by the domain | §3 |
| **NFR-14** No column-based audit | No `created_at`/`updated_at`; if the need appears, it goes back to the owner and is solved with a change log | §8 |
| **NFR-15** Verifiability | The model must be checkable against the engine with the queries of §10; if it contradicts the engine, **the engine wins** | §10, article X |
| **NFR-16** Closed-report stability | A report for an already closed range does not change when new sales are registered (see AC-13.5) | §11.1 |

## 4. Requirements pending implementation (declared by the model)

| Task | What it requires | Requirement it supports |
|---|---|---|
| T-11 | Frozen `category_name` on the line (+ report) | AC-11.6, AC-13.2 |
| T-12 | `sold_by_user_id` + FK-4 and rename of `sold_by` | NFR-02, AC-11.1 |
| T-13 | Three indexes (`product` by category/name, trigrams, unique with `INCLUDE`) and dropping `IX_sale_item_sale_id` | NFR-10 |
| T-20 | CHECKs for `price`, `quantity`, `category.name`, `role` and lowercase | AC-05.3, AC-11.4, AC-02.2, AC-01.2 |

## 5. Assumptions

| ID | Assumption | Reason |
|---|---|---|
| **S-R1** | Administrator: manages the catalog (US-05 to US-10) and views the report (US-13). Seller: views the catalog, registers and views sales. | The model only fixes that the admin creates sellers (§13) and that the user "registers sales" (§1). The other permissions do **not** come from the model |
| **S-R2** | Whoever lists sales (US-12) sees all sales, not only their own. | DP-02 forbids the seller breakdown in the report, but nothing is said about the listing |
| **S-R3** | Restocking is an explicit administration operation (US-08). | `Product.Restock` exists (§2.2); who uses it is not stated |

## 6. Outside these requirements (decided by the model)

API contract, the business acceptance criteria of `spec.md`, decisions D-01…D-10 and the testing strategy (§12); additional product attributes (DP-03); currency column (D-05); breakdown by seller (DP-02).
