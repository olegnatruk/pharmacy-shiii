# Karlo — one prompt for your whole part

Build all of `karlo/` with **one prompt**, without waiting for Heather or Cameron. How solo mode works (stand-ins, the `integration` branch): [docs/SOLO-MODE.md](../../docs/SOLO-MODE.md). The team must all use solo mode; don't mix it with the step prompts in [PROMPTS.md](PROMPTS.md).

## How to use
1. Run **Git: start** once.
2. Open your agent at the **repository root** and paste **The prompt**.
3. The agent plans once, waits for your OK, then works through everything, pausing after each group so you can commit.
4. If the session ends before it's done, open a new session and **paste the same prompt again**. It picks up where it stopped.
5. When it says you're done, run **Git: finish**.

## Git: start
```bash
git switch main
```
```bash
git pull
```
```bash
git switch -c karlo/inventory
```

## The prompt
```text
I'm Karlo (Group #8). We use SOLO MODE (docs/SOLO-MODE.md): I build my whole part now,
without waiting for Heather or Cameron.

READ FIRST: AGENTS.md, karlo/AGENTS.md, docs/SOLO-MODE.md, karlo/docs/CONTEXT.md,
docs/GOTCHAS.md, karlo/docs/GOTCHAS.md, and the latest entries of karlo/docs/SESSION_LOG.md.
The spec is docs/BLUEPRINT.md §4, docs/ARCHITECTURE.md (§6 contracts, §7 exceptions,
§8 menu tree, §9.2 stock), docs/FEATURES.md M1-M5, docs/VALIDATION.md (§2, §3.2, §4) and
docs/DATA.md §3.2-3.3, §4, §5.

GOAL: finish everything I own, in this order:
1. Stand-ins + dev launcher (see below), so my code can compile and run.
2. model/ (Medicine with guarded setters and isExpired/isNearExpiry/isLowStock/isSellable,
   MedicineCategory with getLabel, MedicineStatus, StockAudit), MedicineRepository and
   StockAuditRepository (stateless; a wrong column count throws DataAccessException with
   the line number - never skip a line), exception/InsufficientStockException,
   ui/MedicineTable.
3. Catalog CRUD M4 (admin): InventoryService list/view/add/update (blank keeps current)/
   discontinue/reactivate/delete + CatalogMenu, with rules M1-M5, M11, M12. Delete is only
   for medicines with no order history - check Cameron's OrderService.hasOrdersForMedicine,
   otherwise tell the admin to discontinue.
4. The stock contract Cameron depends on, with the EXACT signatures in ARCHITECTURE §6.3:
   findSellable (M7), reserveStock (M8, message like "Only 4 units of Biogesic 500mg
   left."), restoreStock (works for any existing medicine, even discontinued or expired).
   No role check on these three. Javadoc on each.
5. Browse M1, View Details M2, Search & Filter M3: MedicineFilter, InventoryService
   browseSellable/search, MedicineBrowserMenu (pharmacist), "Search & filter" in
   CatalogMenu. Pharmacists only ever see sellable medicines.
6. Stock & Inventory Audit M5: low-stock and expiry reports, record physical count (read
   the system quantity BEFORE overwriting and write a StockAudit row), update
   lot/expiry/reorder level, audit history + StockAuditMenu.

HEATHER'S AND CAMERON'S CODE IS NOT HERE. Create stand-ins in stubs/ for everything I use
from common/ (ARCHITECTURE §6.1, and only the InputValidator methods I call) and for
Cameron's OrderService.hasOrdersForMedicine (scan data/order_items.txt), plus
stubs/pharmacy/dev/DevLauncher.java - all following docs/SOLO-MODE.md §3 exactly. Never edit
common/, heather/ or cameron/. If any of their real classes already exist, use those and
don't stand them in.

HOW TO WORK:
- First check what already exists (files + Progress in karlo/docs/CONTEXT.md). If we did
  part of this in an earlier session, continue from there.
- Give me ONE plan for everything that is left, then wait for my OK once. After that, work
  through the groups above without stopping. After each group: compile with the solo-mode
  command in docs/SOLO-MODE.md §4, run java -cp out pharmacy.dev.DevLauncher (you may script
  input with printf | java ...), tick karlo/docs/CONTEXT.md, and give me a 3-line summary
  plus a commit message "type(area): summary" so I can commit. Then continue.
- After group 3, add 5-8 realistic medicines through my CatalogMenu (OTC and Rx, one near
  expiry, one low stock) and show me data/medicines.txt.
- Stop and ask me only if a spec is unclear or seems wrong, or a §6 contract would have to
  change.
- Simple Java 17, no libraries. Explain each class in 2-3 plain sentences: what it does and
  which OOP pillar or pattern it shows. I must defend every line.
- No git commands that change anything. Read-only git is fine. I commit and push myself.

DONE WHEN: every option in CatalogMenu, MedicineBrowserMenu and StockAuditMenu works with
the right role; expired and discontinued medicines never reach a pharmacist; reserveStock
and restoreStock change data/medicines.txt correctly (show me a throwaway check, not
committed); a physical count below system stock writes a correct stock_audits.txt line;
every hostile input in docs/VALIDATION.md §4 is handled on my screens; deleting data/ and
running again works; karlo/docs/CONTEXT.md is fully ticked; karlo/docs/SESSION_LOG.md has
an entry; any doc my work made wrong is fixed. Then give me: the list of files, a PR
description from .github/pull_request_template.md, and the manual tests I should run myself.
```

## Git: save progress (after each group the agent finishes)
```bash
git add karlo/ docs/
```
```bash
git commit -m "<the message the agent gave you>"
```
```bash
git push -u origin karlo/inventory
```
`stubs/` is ignored by git, so it never gets committed. Check with `git status` if unsure.

## Git: finish
```bash
gh pr create --base integration --title "feat(inventory): Karlo - medicines and inventory"
```
The base is **`integration`**, not `main`. After all three PRs are merged, follow [SOLO-MODE §5 Integration](../../docs/SOLO-MODE.md#5-git-in-solo-mode).
