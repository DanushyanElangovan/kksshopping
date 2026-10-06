# KKS Shopping Store — v3 (shop + accounts + orders + back office)

## How an e-commerce site is organised (and how this project follows it)

| Part | Who uses it | Where | Protected by |
|---|---|---|---|
| **Storefront** – browse, search, add to list | Anyone | `/` (`index.html`) | public |
| **Customer account** – register, log in, place & track orders | Registered customers | dialogs on the storefront + `api/customer.php` | customer login (table `customers`) |
| **Back office (admin)** – orders, inventory, products, customers, staff | **Your staff only** | `/admin/` | staff login (table `admins`) – **a different table, cookie and API** |

**Why customers can never reach the admin panel**
1. The storefront has **no link** to `/admin/`. You open it by typing the address.
2. Hiding the link is not security. The real protection is on the **server**: every file in `admin/api/` refuses any request that is not from a logged-in *staff* account (customer logins don't count – they are in another table and use another session cookie, scoped to `/admin`).
3. Staff accounts **cannot be created from the website**. Only `php tools/create_admin.php` (command line) or the owner inside the admin can create them.
4. Extra layers built in: password hashing (bcrypt), CSRF tokens on every write, login lock-out after 5 wrong tries, 2-hour idle timeout, `noindex` so Google never lists it, two roles (**owner** / **staff**).
5. Optional extra layers (see `admin/.htaccess`): allow only your own IP, or add a second HTTP password in front.

## Order life-cycle
`Received → Confirmed → Packed → On the way → Delivered` (or `Cancelled`)
* The customer places an order (no payment online). You phone/WhatsApp, then enter **unit prices** in the order and mark payment.
* **Confirming** an order *reserves stock* (inventory goes down). **Cancelling** gives it back. Every change is written to a stock history.
* Inventory is **opt-in per product**: a product with no stock number is “not tracked” and always orderable. Type a number in *Inventory → Set* to start tracking. Customers see “Low stock” / “Out of stock” (never the exact count).

## Install (XAMPP / Laragon on your PC)
1. Copy this folder to `C:\xampp\htdocs\kksshopping`.
2. Start **Apache** and **MySQL** in XAMPP. In phpMyAdmin → *Import* → choose `database.sql`.
3. `includes/config.php`: XAMPP defaults (user `root`, empty password) already work. Change them if yours differ.
4. Create your owner login – open XAMPP **Shell** and run:
   `php tools/create_admin.php`   (it asks for username, name, password ≥ 10 chars)
5. Storefront: `http://localhost/kksshopping/`  ·  Back office: `http://localhost/kksshopping/admin/`

## Install (real hosting / cPanel)
Upload everything, create a MySQL database + user in cPanel, import `database.sql` (remove the first two lines `CREATE DATABASE…`/`USE…` if your host pre-creates the DB), put the credentials in `includes/config.php`, create the admin with the host's Terminal (or ask the host to run the command once). **Use HTTPS** (free via Let's Encrypt/AutoSSL) – cookies become `Secure` automatically.
Tip: make the admin address less guessable by renaming the `admin` folder (e.g. `kks-office-7f3`) – it works from any folder name.

## Product pictures
All 247 products have a generated **illustration** (`uploads/products/*.svg`, one per model, varied by type/colour/size). They are *illustrations, not photographs*. Replace any of them with a real photo: Admin → Products → Edit → choose image (JPG/PNG/WEBP, up to 5 MB). SVG uploads are refused on purpose (security).

## Files
`index.html, css/store.css, js/app.js` storefront · `api/` public + customer API · `admin/` back office (+ `admin/api/`) · `includes/` config & shared code · `tools/create_admin.php` · `database.sql` · `uploads/`

## Before going live – checklist
* HTTPS on · strong owner password · `includes/config.php` not world-readable · database backups (cPanel → Backup) · test an order end-to-end.
* Not included (can be added): online card payment (e.g. PayHere), e-mail/SMS notifications, “forgot password” e-mails (today the admin sets a temporary password under *Customers*), Sinhala/Tamil text for the new account/checkout dialogs (they show in English).
