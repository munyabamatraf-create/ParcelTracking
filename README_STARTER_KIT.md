# ParcelTrack Lite Starter Kit

STTCK3033 Secure Software Development · Lecturer: Munya Saleh Saeed Ba Matraf · Deadline: Week 14

This folder is an **optional starting point** for groups that would rather adapt existing code than build ParcelTrack Lite from scratch. It is a working PHP + MySQL survey system ("EazySurvey") built on the AdminLTE dashboard template. It already has a login, three user roles, add/edit/delete screens and a dashboard.

> **Warning: this code is deliberately left insecure.** It contains SQL injection, weak password storage, missing access checks, unsafe file handling and more. Finding and fixing these flaws is the point of the project. Do **not** put it on a public cloud server until you have fixed at least the critical flaws marked **(critical)** below: anyone on the internet could take over the server.

All personal data has been removed. The database contains only fake sample records.

## 1. Rules for using the starter kit

1. **Declare it.** Groups that use this kit must say so when they submit their module allocation (Week 5) and again at the demonstration.
2. **Save the original first.** Before changing anything, commit the untouched kit to your Git repository (or keep a dated, unmodified copy if you do not use Git). The examiner compares your final code with this baseline.
3. **You are marked on what you change.** The kit gives you a skeleton, not marks. Each member must convert their module into ParcelTrack Lite, add the security controls listed for it in the rubric, test it and explain their own code in the live demonstration.
4. **Keep only what you need.** Remove survey features that ParcelTrack Lite does not use (categories, templates, contact inbox and so on), or leave them switched off.

## 2. Running it locally

You need XAMPP or Laragon (PHP 8 and MySQL/MariaDB).

1. Copy this folder into `htdocs` (XAMPP) or `www` (Laragon), for example as `parceltrack`.
2. Open phpMyAdmin, create a database named `survey_db`, then import `database/parceltrack_starter.sql`.
3. Open `http://localhost/parceltrack/login.php`.

The database settings are in `db_connect.php` and `includes/dbconnection.php`. They use `root` with no password, which is acceptable only on your own laptop. Change them before you deploy.

### Demo accounts (fake)

| Email | Password | Role in the survey app | Becomes in ParcelTrack Lite |
|---|---|---|---|
| admin@example.com | Admin@2026 | 1 = Administrator | Admin |
| seller@example.com | Seller@2026 | 2 = Researcher | Seller |
| rider@example.com | Rider@2026 | 3 = Respondent | Rider |

The email features (registration, password reset, contact form) use PHPMailer with placeholder settings (`YOUR_SMTP_USERNAME`). They will not send mail until you configure them. Never type real credentials into the source code (policy P4); read them from a configuration file outside the web root or from environment variables.

## 3. What to adapt for each module

| Module | Start from these files | What to change | Flaws already in this code that you must fix |
|---|---|---|---|
| **M1. Login and User Management** | `login.php`, `registration.php`, `user_list.php`, `new_user.php`, `manage_user.php`, functions `login()`, `save_user()`, `update_user()` in `admin_class.php` | Rename the roles to Admin, Seller and Rider. Sellers self-register; only Admin creates Rider accounts. Add change password. | SQL injection in login and user queries; passwords stored with MD5; no lockout or rate limit; session ID not regenerated at login; registration lets a user choose their own role, including Administrator; the registration email includes the password hash |
| **M2. Parcel Registration and Public Tracking** | `new_template.php` / `new_templates.php` (create), `survey_templates.php` / `templates_list.php` (list), `homepage.php` (public page), function `save_survey()` | Replace the `survey_set` table with a `parcels` table (recipient name, phone, address, weight, tracking number, seller). Turn the public homepage into a tracking-number search that shows status, date and time only. | SQL injection in every query; records loaded and deleted by ID with no ownership check (IDOR); output printed without escaping (XSS); no server-side validation; no tracking-number generation yet (must be cryptographically random) |
| **M3. Delivery Update and Admin Dashboard** | `answer_survey.php` and `save_answer()` (status updates), `home.php` (dashboard counts), `save_page_img()` in `admin_class.php` (file upload) | Replace answers with a `status_history` table (parcel, status, user, time, photo). Add rider assignment. Show parcel counts by status on the Admin dashboard. Build the proof-of-delivery photo form. | `save_page_img()` accepts any file under its original name inside the web folder **(critical)**; status saved without checking the parcel belongs to the rider; admin pages such as `user_list.php` have no server-side role check (the menu only hides them) |

## 4. Flaws that affect the whole application

These belong to the group. Agree who fixes each one, and fix them before cloud deployment.

| Flaw | Where | Why it matters |
|---|---|---|
| **(critical)** No login or role check on any action | `ajax.php` | Anyone can call `delete_user`, `save_user` and the other actions without logging in |
| **(critical)** Page name taken straight from the URL | `index.php` (`include $page.'.php'`) | Lets an attacker load files they should not reach |
| **(critical)** Unrestricted upload | `save_page_img()` | A `.php` file could be uploaded and run on the server |
| Detailed errors shown to users | `ini_set('display_errors', 1)` in `admin_class.php`; `includes/dbconnection.php` prints the database error | Reveals file paths and database details (P4: users see generic messages, details go to logs) |
| Password reset without a token | `forgot-password.php`, `reset_password.php` | Anyone who knows an email address could change that account's password |
| Database login is `root` with no password | `db_connect.php`, `includes/dbconnection.php` | Use a dedicated database user with a strong password, kept out of the source code |
| No logging | whole application | P5 requires logins, failed logins and status changes to be logged |

## 5. A suggested order of work

1. Week 5: decide module owners, commit the untouched kit, and get it running locally.
2. Together: design the ParcelTrack Lite tables and fix the three **(critical)** whole-application flaws.
3. Each member: convert their module, then add the security controls listed for it in the rubric.
4. Each member: write security test cases from the rubric's suggested inputs. Record each test's input, expected result and actual result, before and after your fix.
5. Deploy to a cloud server for full marks in criterion 1b (local-only loses 5 marks), then rehearse the live demonstration.

A good test to start with: before you change anything, type `' OR '1'='1' -- ` as the email on the login page with any password. It logs you in without a valid account. After M1 is fixed, the same input must fail.
