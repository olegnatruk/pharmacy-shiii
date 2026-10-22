# Solo mode — everyone builds their whole part at once

In solo mode, each member gives their agent **one prompt** (`<member>/docs/SOLO-PROMPT.md`) and builds their entire part **without waiting for anyone**. The three parts meet once at the end, on an `integration` branch. This is the alternative to the step-by-step prompts in `<member>/docs/PROMPTS.md` ([ADR-016](DECISIONS.md)).

| | Step mode (`PROMPTS.md`) | Solo mode (`SOLO-PROMPT.md`) |
|---|---|---|
| Prompts | 5–7 per member, in order | 1 per member, repeated each session |
| Waiting | Yes, at fixed hand-offs | None while building |
| How others' code is reached | It is merged on `main` first | Local **stand-ins** (§3) until the end |
| Merging | Many small PRs into `main` | One PR per member into `integration`, then one PR into `main` (§5) |
| Risk | Slower | Mismatches show up only at integration, so §6 contracts must be followed exactly |

**The whole team picks one mode.** Don't mix them. Both use the same docs, contracts, and rules.

---

## 1. Why it works

Every class one member uses from another is already fixed in [ARCHITECTURE §6](ARCHITECTURE.md#6-cross-member-contracts): package, class name, method names, parameters, return types, exceptions. So each agent can build against those signatures before the real code exists, using a throwaway stand-in. When the real classes arrive, the stand-ins are deleted and the code compiles against the real thing.

That only holds if **nobody changes a §6 signature on their own.** If you need a new cross-member method, post it in the group chat; the owner adds it to §6 and builds it; you add it to your stand-in.

## 2. Who needs which stand-ins

| Member | Stand-ins to create in `stubs/` (only the parts your code calls) | Run with |
|---|---|---|
| **Heather** | Karlo's `MedicineBrowserMenu`, `CatalogMenu`, `StockAuditMenu`; Cameron's `OrderMenu`, `OrderAdminMenu`, `AnalyticsMenu` (each: title only, no options); Cameron's `OrderService.hasOrdersForPharmacist` | the real `pharmacy.Main` |
| **Karlo** | Everything he uses from `common/` (§6.1); Cameron's `OrderService.hasOrdersForMedicine` | `pharmacy.dev.DevLauncher` |
| **Cameron** | Everything he uses from `common/` (§6.1); Heather's `UserRepository` (§6.2); Karlo's `InventoryService`, `MedicineRepository`, `StockAuditRepository`, `Medicine`, `StockAudit`, `InsufficientStockException` (§6.3) | `pharmacy.dev.DevLauncher` |

## 3. Stand-in rules

1. **Location:** `stubs/` at the repository root, in the right package folders, e.g. `stubs/pharmacy/common/data/DataStore.java`. `stubs/` is in `.gitignore` and is **never committed**.
2. **Exact match:** same package, class name, public method signatures, constructors and `throws` as ARCHITECTURE §6. Menus, services and repositories have a public no-argument constructor; Singletons use `getInstance()`.
3. **Small but honest.** Implement only what your code calls, as simply as possible, but behave the way the owner's docs say, so your tests mean something. Examples: a stand-in `DataStore` really reads and writes `data/*.txt` in the [DATA.md](DATA.md) format; a stand-in `reserveStock` really lowers the stock in `data/medicines.txt`; a stand-in `hasOrdersForMedicine` really scans `data/order_items.txt`.
4. **Label every file** on its first line: `// STAND-IN for <owner>'s <Class> (docs/SOLO-MODE.md). Delete when the real class exists.`
5. **Never duplicate a real class.** If a class already exists in `common/`, `heather/`, `karlo/` or `cameron/`, use it and delete your stand-in, or the compiler reports a duplicate class.
6. **Your real code must not know stand-ins exist.** It must never import `pharmacy.dev`, and must not rely on anything a stand-in does that §6 or the owner's docs don't promise.
7. **Dev launcher (Karlo, Cameron):** `stubs/pharmacy/dev/DevLauncher.java` calls `DataStore.getInstance().seedDefaults()`, asks "1 Admin / 2 Pharmacist", logs a test user into `Session`, and offers your menus. It replaces `Main` until integration.
8. **Test data:** create it through your own menus where you can. Otherwise write lines into `data/*.txt` by hand, following [DATA.md](DATA.md) column by column. Cameron writes a few medicine, user and stock-audit lines this way.

## 4. Compile and run in solo mode

From the repository root. Delete `out/` first so old classes don't hide errors.

macOS / Linux:
```bash
rm -rf out && javac -d out $(find common heather karlo cameron stubs -name "*.java")
```
```bash
java -cp out pharmacy.dev.DevLauncher
```
(Heather runs `java -cp out pharmacy.Main` instead.)

Windows (PowerShell):
```powershell
Remove-Item -Recurse -Force out -ErrorAction SilentlyContinue; javac -d out (Get-ChildItem -Recurse -Filter *.java common,heather,karlo,cameron,stubs).FullName
```
```powershell
java -cp out pharmacy.dev.DevLauncher
```

**IDE:** also mark `stubs` as a source folder (see [README → Run it](../README.md#run-it)). Unmark it at integration.

## 5. Git in solo mode

Branches: **`heather/accounts`**, **`karlo/inventory`**, **`cameron/orders`**, plus one shared **`integration`** branch. Exact commands per member are in their `SOLO-PROMPT.md`.

**Once, by one person (before anyone starts):**
```bash
git switch main
```
```bash
git pull
```
```bash
git switch -c integration
```
```bash
git push -u origin integration
```

**While building (each member):** commit after every finished group of classes and push often. Pushing is your backup; nobody merges anything yet.

**When your part is done:** open a PR from your branch **into `integration`** (not `main`). A groupmate reviews it, then it is merged. `integration` will not compile until all three are merged; that is expected.

**Integration (all three, together):**
1. After all three PRs are merged, everyone runs `git switch integration` and `git pull`.
2. Everyone **deletes their `stubs/` folder**.
3. Compile with the normal command in the [README](../README.md#run-it) and run `pharmacy.Main`.
4. Each member gives their agent the [integration prompt](#6-integration-prompt). Each fixes errors **only in their own folder**, on a branch from `integration` (`<member>/integration-fixes`), and opens a PR into `integration`.
5. When it compiles and the M2 checks in the [README build order](../README.md#build-order) pass, open **one PR from `integration` into `main`**. All three review it.
6. Merge it with `gh pr merge --merge` (not `--squash`), so each member's commits keep their own name in the history.

## 6. Integration prompt

Each member pastes this into their agent after step 2 above:
```text
We are integrating (docs/SOLO-MODE.md §5). All three parts are merged into the integration
branch and I deleted my stubs/ folder. I'm <name>.
1. Compile everything with the normal README command (no stubs). List every error, and for
   each one say whose folder it is in and which docs/ARCHITECTURE.md §6 contract it involves.
2. Fix only errors in my own folder. For errors in someone else's folder, write me a short
   message I can send them: file, error, and what §6 says.
3. Run java -cp out pharmacy.Main and walk through every feature I own with both roles,
   now using the real code of the others. Report anything that behaved differently than
   with my stand-ins.
No git commands that change anything.
```
