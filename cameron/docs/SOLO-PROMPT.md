# Cameron — one prompt for your whole part

Build all of `cameron/` with **one prompt**, without waiting for Heather or Karlo. How solo mode works (stand-ins, the `integration` branch): [docs/SOLO-MODE.md](../../docs/SOLO-MODE.md). The team must all use solo mode; don't mix it with the step prompts in [PROMPTS.md](PROMPTS.md).

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
git switch -c cameron/orders
```

## The prompt
```text
I'm Cameron (Group #8). We use SOLO MODE (docs/SOLO-MODE.md): I build my whole part now,
without waiting for Heather or Karlo.

READ FIRST: AGENTS.md, cameron/AGENTS.md, docs/SOLO-MODE.md, cameron/docs/CONTEXT.md,
docs/GOTCHAS.md, cameron/docs/GOTCHAS.md, and the latest entries of
cameron/docs/SESSION_LOG.md. The spec is docs/BLUEPRINT.md §5, docs/ARCHITECTURE.md
(§5.2 Strategy, §6 contracts, §7 exceptions, §8 menu tree, §9 orders/stock/status),
docs/FEATURES.md O1-O11, docs/VALIDATION.md (§2, §3.3, §4), docs/DATA.md §3.4-3.6, §4, §5
and docs/DECISIONS.md ADR-007, ADR-010, ADR-011.

GOAL: finish everything I own, in this order:
1. Stand-ins + dev launcher + test data (see below), so my code can compile and run.
2. Strategy pattern and status machine: DiscountStrategy, RegularDiscount,
   SeniorPwdDiscount (RATE 0.20), BulkDiscount (RATE 0.10, MIN_UNITS 50), DiscountType with
   getLabel and toStrategy; OrderStatus with canTransitionTo/isEditable/isTerminal (the
   table in ARCHITECTURE §9.1); InvalidStatusTransitionException.
3. Order data: Order (unmodifiable items, DiscountStrategy field, totals through
   Format.round2), OrderItem (unitPriceAtSale snapshot), VoidRecord, OrderRepository (save
   replaces the order line AND all its item lines), VoidRecordRepository, and
   OrderService.hasOrdersForPharmacist / hasOrdersForMedicine with the EXACT signatures in
   ARCHITECTURE §6.4 - Heather and Karlo depend on them.
4. Process Dispensing Order O1 + Apply Discount O2 (rules O1-O7): the cart lives in memory
   and only checks stock; placeOrder reserves stock for every item with rollback on failure
   (ARCHITECTURE §9.2-9.3); OrderMenu "New order" ends with Complete now / Save as pending /
   Discard (Discard touches nothing).
5. Own orders O3-O6 (rules O7-O12): view/filter, update pending (reserve/restore the
   difference, re-check Bulk), hold/resume, complete (re-check expiry), cancel (restore ALL
   stock). Every method checks the order belongs to the logged-in user.
6. Admin O7-O10 (rules O10, O12, O13, O15): all orders, search & filter, update status
   (never to VOIDED), void (COMPLETED only, restore stock, VoidRecord, confirm first),
   prescription audit + OrderAdminMenu. Show pharmacist names via Heather's
   UserRepository.findById.
7. Analytics O11 (rule O14): SalesReport, AnalyticsService.report(from, to) - only COMPLETED
   orders by completedAt count; shrinkage from Karlo's StockAuditRepository.findAll()
   (negative differences) priced with MedicineRepository.findById - and AnalyticsMenu
   (Daily, Monthly, Yearly, Custom range).

HEATHER'S AND KARLO'S CODE IS NOT HERE. Create stand-ins in stubs/ for everything I use from
common/ (ARCHITECTURE §6.1, and only the InputValidator methods I call), Heather's
UserRepository (§6.2), and Karlo's InventoryService, MedicineRepository,
StockAuditRepository, Medicine, StockAudit and InsufficientStockException (§6.3), plus
stubs/pharmacy/dev/DevLauncher.java - all following docs/SOLO-MODE.md §3 exactly. The stock
stand-ins must really read and change data/medicines.txt. Never change stock except through
InventoryService. Never edit common/, heather/ or karlo/. If any of their real classes
already exist, use those and don't stand them in.
TEST DATA: write by hand, exactly in the docs/DATA.md column order, a users.txt with the
default admin and 2 pharmacists, a medicines.txt with 6-8 medicines (OTC and Rx, one
expired, one with low stock, one with 100+ units for Bulk), and 2 stock_audits.txt lines
(one negative difference).

HOW TO WORK:
- First check what already exists (files + Progress in cameron/docs/CONTEXT.md). If we did
  part of this in an earlier session, continue from there.
- Give me ONE plan for everything that is left, then wait for my OK once. After that, work
  through the groups above without stopping. After each group: compile with the solo-mode
  command in docs/SOLO-MODE.md §4, run java -cp out pharmacy.dev.DevLauncher (you may script
  input with printf | java ...), tick cameron/docs/CONTEXT.md, and give me a 3-line summary
  plus a commit message "type(area): summary" so I can commit. Then continue.
- After group 2, show me a throwaway check (not committed) printing each strategy's discount
  on subtotal 1000 with 10 and 60 units, and every allowed/blocked status transition.
- Stop and ask me only if a spec is unclear or seems wrong, or a §6 contract would have to
  change.
- Simple Java 17, no libraries. Explain each class in 2-3 plain sentences: what it does and
  which OOP pillar or pattern it shows. Explain the Strategy pattern using my code, the way
  I would in the defense. I must defend every line.
- No git commands that change anything. Read-only git is fine. I commit and push myself.

DONE WHEN: every option in OrderMenu, OrderAdminMenu and AnalyticsMenu works with the right
role; a pharmacist sees only their own orders; stock in data/medicines.txt is right after
place, update, cancel, discard and void (show me before/after); insufficient stock, an
expired medicine, an Rx item without prescription and Bulk with 49 units are all rejected
with clear messages; analytics numbers match a hand count; every hostile input in
docs/VALIDATION.md §4 is handled on my screens; deleting data/ and re-adding the test data
works; cameron/docs/CONTEXT.md is fully ticked; cameron/docs/SESSION_LOG.md has an entry;
any doc my work made wrong is fixed. Then give me: the list of files, a PR description from
.github/pull_request_template.md, and the manual tests I should run myself.
```

## Git: save progress (after each group the agent finishes)
```bash
git add cameron/ docs/
```
```bash
git commit -m "<the message the agent gave you>"
```
```bash
git push -u origin cameron/orders
```
`stubs/` and `data/` are ignored by git, so they never get committed. Check with `git status` if unsure.

## Git: finish
```bash
gh pr create --base integration --title "feat(orders): Cameron - orders, discounts and analytics"
```
The base is **`integration`**, not `main`. After all three PRs are merged, follow [SOLO-MODE §5 Integration](../../docs/SOLO-MODE.md#5-git-in-solo-mode).
