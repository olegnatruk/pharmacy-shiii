# Heather — one prompt for your whole part

Build all of `common/` and `heather/` with **one prompt**, without waiting for Karlo or Cameron. How solo mode works (stand-ins, the `integration` branch): [docs/SOLO-MODE.md](../../docs/SOLO-MODE.md). The team must all use solo mode; don't mix it with the step prompts in [PROMPTS.md](PROMPTS.md).

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
git switch -c heather/accounts
```

## The prompt
```text
I'm Heather (Group #8). We use SOLO MODE (docs/SOLO-MODE.md): I build my whole part now,
without waiting for Karlo or Cameron.

READ FIRST: AGENTS.md, heather/AGENTS.md, docs/SOLO-MODE.md, heather/docs/CONTEXT.md,
common/docs/CONTEXT.md, docs/GOTCHAS.md, heather/docs/GOTCHAS.md, and the latest entries
of heather/docs/SESSION_LOG.md. The spec is docs/BLUEPRINT.md §2-3, docs/ARCHITECTURE.md
(§6 contracts, §7 exceptions, §8 menu tree), docs/FEATURES.md A1-A4, docs/VALIDATION.md
(§2, §3.1, §4) and docs/DATA.md §3.1.

GOAL: finish everything I own, in this order:
1. common/ foundation, exactly as common/docs/CONTEXT.md: the exceptions; Storable, Role,
   AccountStatus, User, Admin, Pharmacist; DataFiles and DataStore (Singleton,
   seedDefaults creates the default admin); Session (Singleton); InputRule, ConsoleInput,
   InputValidator (EVERY require* method in docs/VALIDATION.md §2 - Karlo and Cameron call
   them too), Format; Menu and BaseMenu.
2. Main, AdminMenu and PharmacistMenu with the FULL menu tree from ARCHITECTURE §8, opening
   Karlo's and Cameron's menus by their §6 class names.
3. heather/: UserRepository, AuthService (login A2 with the 3-attempt limit, register A1),
   AuthMenu.
4. Manage Profile A3: UserService profile methods + ProfileMenu.
5. Manage Users A4: UserService admin methods + UserManagementMenu. Delete uses Cameron's
   OrderService.hasOrdersForPharmacist (only pharmacists with no orders; admins can only be
   deactivated).

KARLO'S AND CAMERON'S CODE IS NOT HERE. Create stand-ins in stubs/ for their 6 menu classes
(title only, no options) and for OrderService.hasOrdersForPharmacist, following
docs/SOLO-MODE.md §3 exactly. Never edit karlo/ or cameron/. My common/ is the real one that
everyone integrates with, so it must match ARCHITECTURE §6 exactly - no extra required
parameters, no renamed methods.

HOW TO WORK:
- First check what already exists (files + Progress in heather/docs/CONTEXT.md). If we did
  part of this in an earlier session, continue from there.
- Give me ONE plan for everything that is left, then wait for my OK once. After that, work
  through the groups above without stopping. After each group: compile with the solo-mode
  command in docs/SOLO-MODE.md §4, run java -cp out pharmacy.Main (you may script input
  with printf | java ...), tick heather/docs/CONTEXT.md, and give me a 3-line summary plus a
  commit message "type(area): summary" so I can commit. Then continue.
- Stop and ask me only if a spec is unclear or seems wrong, or a §6 contract would have to
  change.
- Simple Java 17, no libraries. Explain each class in 2-3 plain sentences: what it does and
  which OOP pillar or pattern it shows. I must defend every line.
- No git commands that change anything. Read-only git is fine. I commit and push myself.

DONE WHEN: deleting data/ and running creates data/users.txt with the default admin; I can
register, log in, log out, manage my profile, and (as admin) manage users; 3 wrong passwords
return me to the welcome screen; every admin and pharmacist menu option opens (the stand-in
menus are empty); every hostile input in docs/VALIDATION.md §4 is handled on my screens;
heather/docs/CONTEXT.md is fully ticked; heather/docs/SESSION_LOG.md has an entry; any doc
my work made wrong is fixed. Then give me: the list of files, a PR description from
.github/pull_request_template.md, and the manual tests I should run myself.
```

## Git: save progress (after each group the agent finishes)
```bash
git add common/ heather/ docs/
```
```bash
git commit -m "<the message the agent gave you>"
```
```bash
git push -u origin heather/accounts
```
`stubs/` is ignored by git, so it never gets committed. Check with `git status` if unsure.

## Git: finish
```bash
gh pr create --base integration --title "feat: Heather - common foundation and accounts"
```
The base is **`integration`**, not `main`. After all three PRs are merged, follow [SOLO-MODE §5 Integration](../../docs/SOLO-MODE.md#5-git-in-solo-mode).
