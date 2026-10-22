# Heather — Accounts: context and checklist

Features: **A1 Register · A2 Login/Logout · A3 Manage Profile · A4 Manage Users**. The full behaviour is in [FEATURES §Accounts](../../docs/FEATURES.md#accounts--heather) and the rules in [VALIDATION §3.1](../../docs/VALIDATION.md#31-accounts--heather). Data format: [DATA §3.1](../../docs/DATA.md#31-userstxt--owner-heather).

You also own `common/`: see [common/docs/CONTEXT.md](../../common/docs/CONTEXT.md). Build it first.

## Folder layout
Every file you create, with its purpose and milestone: [BLUEPRINT §3](../../docs/BLUEPRINT.md#3-heather--accounts-heather).

## Classes to build

### `UserRepository` (stateless, reads `users.txt` on every call)
- [ ] `List<User> findAll()`: column 1 (`role`) decides `new Admin(...)` or `new Pharmacist(...)`. A wrong column count throws `DataAccessException` with the line number.
- [ ] `User findById(String userId)` → `NotFoundException("User U-0009 not found.")`
- [ ] `User findByLogin(String usernameOrEmail)`: case-insensitive match on username **or** email.
- [ ] `boolean usernameTaken(String username, String excludeUserId)`, `emailTaken(...)`, `licenseTaken(...)`: case-insensitive. `excludeUserId` lets an update keep its own value; pass `null` on create.
- [ ] `void save(User user)`: replace the record with the same id, or add it; then `DataStore.writeAll`.
- [ ] `void delete(String userId)`
- [ ] `String nextId()` → `DataStore.nextId(DataFiles.USERS, "U-", 4)`

### `AuthService`
- [ ] `public static final int MAX_LOGIN_ATTEMPTS = 3;`
- [ ] `Pharmacist register(fullName, email, contact, license, username, password, confirmPassword)`: re-validate every field, check uniqueness (A5) and the password match (A7), then save (A6).
- [ ] `User login(String usernameOrEmail, String password)`: A1–A3. Uses the same generic message for an unknown user and a wrong password. Does **not** call `Session.login`; `AuthMenu` does that.

### `UserService`
- [ ] Own profile (uses `Session.requireUser()`, never an ID typed by the user):
  - `updateMyProfile(fullName, email, contact)`: a blank value means keep the current one.
  - `changeMyPassword(current, newPassword, confirm)` (A8).
- [ ] Admin only. Every method starts with `Session.getInstance().requireRole(Role.ADMIN)`.
  - `listUsers()`
  - `searchUsers(keyword, Role roleOrNull, AccountStatus statusOrNull)`
  - `createUser(Role role, ...)`
  - `updateUser(userId, ...)`
  - `setStatus(userId, AccountStatus)` (A11)
  - `deleteUser(userId)` (A11, A12). Calls Cameron's `OrderService.hasOrdersForPharmacist`.

### UI
- [ ] `AuthMenu` (not a `BaseMenu`; `Main` calls it directly):
  - `login()`: up to 3 attempts. On success, `Session.getInstance().login(user)` and print the welcome message.
  - `register()`: reads each field with `ConsoleInput.readValidated(..., InputValidator::requireX)`.
- [ ] `ProfileMenu extends BaseMenu`: View · Update details · Change password.
- [ ] `UserManagementMenu extends BaseMenu`: List · Search/filter · View · Create · Update · Deactivate/Reactivate · Delete.

## Contracts

**You provide** ([ARCHITECTURE §6.1–6.2](../../docs/ARCHITECTURE.md#62-provided-by-heather--used-by-cameron-and-common)):
- all of `common/`
- `UserRepository.findAll()` and `findById()`, used by Cameron's prescription audit and analytics
- `AuthMenu`, `ProfileMenu`, `UserManagementMenu`

**You use:**
- `OrderService.hasOrdersForPharmacist(userId)` from Cameron ([§6.4](../../docs/ARCHITECTURE.md#64-provided-by-cameron--used-by-heather-karlo-and-common))
- `CatalogMenu`, `StockAuditMenu`, `MedicineBrowserMenu` (Karlo) and `OrderMenu`, `OrderAdminMenu`, `AnalyticsMenu` (Cameron), opened from `AdminMenu`/`PharmacistMenu`

## Your OOP and pattern evidence (for the defense)
- **Encapsulation:** `User` private fields; no setter for `userId`/`role`; password never printed.
- **Abstraction:** abstract `User` and `BaseMenu`; the `Storable`, `Menu`, and `InputRule` interfaces.
- **Inheritance:** `Admin`/`Pharmacist extends User`; your menus `extend BaseMenu`; the exception hierarchy.
- **Polymorphism:** `user.createMenu()` opens a different menu per role; `toFileLine()` per subclass.
- **Singleton:** `DataStore` and `Session`.
- **Exception handling:** the whole hierarchy, plus `BaseMenu`'s catch.

## Definition of done (per feature)
- [ ] Compiles together with all four folders.
- [ ] Works for both roles where relevant.
- [ ] Every rule A1–A13 that applies is enforced in the **service**.
- [ ] Passes the [hostile-input checklist](../../docs/VALIDATION.md#4-hostile-input-checklist).
- [ ] `data/users.txt` lines match [DATA §3.1](../../docs/DATA.md#31-userstxt--owner-heather).
- [ ] PR reviewed by Karlo or Cameron.

## Progress
| Milestone | Status |
|---|---|
| M0 Skeleton: empty `AuthMenu`, `ProfileMenu`, `UserManagementMenu` | ☐ |
| M1 `common/` complete | ☐ |
| A2 Login/Logout | ☐ |
| A1 Register | ☐ |
| A3 Manage Profile | ☐ |
| A4 Manage Users | ☐ |
