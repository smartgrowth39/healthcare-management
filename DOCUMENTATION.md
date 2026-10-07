# 📋 CARE Group — Health Care Appointment Portal
### Complete Project Documentation

> **Stack:** PHP 8.x (MySQLi) · MySQL · Bootstrap 5.3 · Font Awesome 6 · PHPMailer
> **Type:** Academic / Final Year Project — Medical Appointment System
> **DB Name:** `health_care` (localhost, user `root`, no password)

---

## 1. Project Overview

**CARE Group** is a web-based **medical appointment booking portal** jisme 3 user roles hain:

| Role | Panel Location | Login Source |
|------|----------------|--------------|
| 👤 **Patient** | `patient_panel/` | `login.php` (role selector) |
| 🩺 **Doctor** | `doctor_final/doctor_output/` | `login.php` (role selector) |
| 🛡️ **Admin** | `admin/` | `admin/login.php` (separate) |

Patient doctor search karta hai (city/specialization filter se), available time slot choose karke appointment book karta hai. Doctor appointment ko **confirm / cancel / complete** karta hai, apni weekly availability aur leaves manage karta hai. Admin poora system control karta hai (doctors, patients, cities, specializations, diseases, news, feedback).

**Extra features:**
- 📧 Registration par **PHPMailer** se welcome email (Gmail SMTP)
- 📰 **Medical News** section (`disease_news.php`)
- 🦠 **Diseases Guide** (`diseases.php`) — symptoms, causes, prevention, treatment
- ⭐ **Feedback/Testimonials** system (admin approve karta hai — `feedback_status=1` wale homepage par dikhte hain)

---

## 2. Directory Structure

```
health-care/
├── index.php                  # Homepage (How It Works, Featured Doctors, Specs, Testimonials, CTA)
├── login.php                  # Unified login (Patient + Doctor, role dropdown)
├── register.php               # Patient registration + validation + PHPMailer email
├── connect.php                # DB connection (localhost/root/health_care, Asia/Karachi TZ)
├── search-doctor.php          # Doctor search (City + Specialization filters)
├── appointment.php            # Appointment booking (date → slots → confirm, 3-step)
├── diseases.php               # Diseases information cards
├── disease_news.php           # Medical news page
├── feedback.php               # Public feedback form
├── about.php / contact.php    # Static pages
├── home_crousel.php / crousel.php  # Homepage carousel
│
├── includes/                  # Shared layout partials
│   ├── head.php               # <head>, Bootstrap, fonts, css/style.css
│   ├── navbar.php             # Main navbar (active-link logic, session-aware)
│   ├── footer.php
│   └── scripts.php
│
├── css/style.css              # Main public site styling
├── js/script.js               # Public site scripts
│
├── patient_panel/             # 👤 PATIENT PANEL
│   ├── _auth.php              # Auth guard + CSRF helpers (patient role check)
│   ├── dashboard.php          # Stats, quick actions, recent appointments
│   ├── search_doctor.php      # Doctor search inside panel
│   ├── book_appointment.php   # Booking flow
│   ├── my_appointments.php    # Appointment history + cancel
│   ├── profile.php            # Profile view/edit
│   ├── sidebar_navbar.php     # Sidebar layout
│   └── logout.php
│
├── doctor_final/
│   └── doctor_output/         # 🩺 DOCTOR PANEL
│       ├── _auth.php          # Auth guard + CSRF (doctor role check, HTTPS-aware cookies)
│       ├── dashboard.php      # Today's schedule, stats, recent appointments
│       ├── appointments.php   # Tabs: Today / Upcoming / Past + status update (CSRF protected)
│       ├── availability.php   # Weekly schedule (Mon–Sun) + Leaves management
│       ├── profile.php
│       ├── sidebar_navbar.php
│       └── logout.php
│
├── admin/                     # 🛡️ ADMIN PANEL
│   ├── login.php              # Admin login (prepared statements ✓)
│   ├── admin_register.php     # Admin registration
│   ├── dashboard.php          # Stats + recent doctors/patients + feedback management
│   ├── doctors.php            # Doctor CRUD (add via register, status, delete, image → admin/src/)
│   ├── patients.php           # Patients list
│   ├── cities.php             # Cities CRUD
│   ├── specializations.php    # Specializations CRUD
│   ├── diseases.php           # Diseases CRUD
│   ├── includes/              # header.php, sidebar.php, footer.php (admin layout)
│   ├── assets/                # Admin CSS/JS
│   └── src/                   # 📁 Doctor images upload folder
│
├── PHPMailer-master/          # SMTP email library
└── DOCUMENTATION.md           # ← ye file
```

---

## 3. Database Schema (inferred from code)

> ⚠️ **Note:** Repo mein `.sql` file nahi mili. Ye schema code analysis se infer kiya gaya hai.

### `patients`
| Column | Type (approx) | Notes |
|---|---|---|
| patient_id | VARCHAR (PK) | `PAT_` + uniqid, e.g. `PAT_68f2...` |
| name | VARCHAR(30) | only letters + spaces |
| email | VARCHAR | UNIQUE, @gmail/@yahoo/@aptechsite.net only |
| phone | VARCHAR(11) | exactly 11 digits |
| gender | ENUM | male / female |
| city_id | INT (FK → cities) | |
| address | TEXT | |
| password | VARCHAR(255) | `password_hash()` bcrypt |
| status | ENUM | default `active` |

### `doctors`
| Column | Notes |
|---|---|
| doctor_id | INT (PK, AUTO_INCREMENT) |
| first_name, last_name | VARCHAR |
| email | VARCHAR — login email |
| password | bcrypt hash |
| phone, qualification, bio | TEXT |
| experience | INT (years) |
| consultation_fee | DECIMAL (Rs.) |
| doctor_image | VARCHAR — file in `admin/src/` |
| city_id | FK → cities |
| specialize_id | FK → specialization |
| doctor_status | TINYINT — `1` = active/visible |
| created_at | TIMESTAMP |

### `appointments`
| Column | Notes |
|---|---|
| appointment_id | INT PK |
| doctor_id | FK → doctors |
| patient_id | VARCHAR FK → patients |
| appointment_date | DATE |
| appointment_time | TIME (30-min slots) |
| status | ENUM — `pending` / `confirmed` / `cancelled` / `completed` |

### `doctor_availability`
| Column | Notes |
|---|---|
| id | INT PK |
| doctor_id | FK → doctors |
| day_of_week | VARCHAR — `Mon`…`Sun` |
| start_time, end_time | TIME |
| is_available | TINYINT |

### `doctor_leaves`
| Column | Notes |
|---|---|
| id | INT PK |
| doctor_id | FK |
| leave_date | DATE |
| start_time, end_time | NULL = **full-day leave**, set = **partial-day leave** |
| reason | VARCHAR |

### Other tables
- **`cities`** — `city_id`, `city_name`, `city_status` (`active`)
- **`specialization`** — `specialize_id`, `specialize`, `specialization_status` (`active`)
- **`disease`** — `Disease_id`, `disease_name`, `symptoms`, `causes`, `prevention`, `treatment`, `status`, `specialize_id`
- **`feedback`** — `feedback_id`, `full_name`, `email`, `message`, `feedback_status` (1 = homepage par show), `created_at`
- **`admin`** — `admin_id`, `admin_name`, `admin_email`, `admin_password` (bcrypt)
- Disease news ke liye ek table (`disease_news.php` use karta hoga)

### 🔗 Relationships (ER Summary)
```
cities 1───∞ patients
cities 1───∞ doctors
specialization 1───∞ doctors
specialization 1───∞ disease
doctors 1───∞ appointments ∞───1 patients
doctors 1───∞ doctor_availability
doctors 1───∞ doctor_leaves
```

---

## 4. Authentication & Session Flow

### 4.1 Patient/Doctor Login (`login.php`)
```
POST login.php
  ├── role = patient → table `patients` → patient_id
  ├── role = doctor  → table `doctors`  → doctor_id
  └── password_verify() against bcrypt hash
Session set: $_SESSION['user_id'], ['role'], ['user_email']
Redirect: patient → patient_panel/dashboard.php
          doctor  → doctor_final/doctor_output/dashboard.php
```

### 4.2 Admin Login (`admin/login.php`)
Separate session variable: `$_SESSION['admin_email']` → `admin/dashboard.php`.

### 4.3 Auth Guards
- `patient_panel/_auth.php` — checks `user_id` + `role === 'patient'`, else redirect `../login.php`. Also defines `patient_id()`, `csrf_token()`, `check_csrf()`.
- `doctor_final/doctor_output/_auth.php` — same pattern for doctor, plus `secure` cookie flag jab HTTPS ho.
- Admin pages inline check karte hain: `if (!isset($_SESSION["admin_email"])) → login.php`.

### 4.4 Session Cookie Settings
`httponly: true`, `samesite: Lax`, `secure` (HTTPS par) — good practice ✓

### 4.5 CSRF Protection
Patient aur doctor panels mein `csrf_token()` + `check_csrf()` (hash_equals) implemented — doctor `appointments.php` isko use karta hai. ⚠️ Public pages aur admin panel mein CSRF nahi hai.

---

## 5. Core Workflows

### 5.1 Patient Registration → Email
1. `register.php` — server-side validation:
   - Name: letters/spaces only, max 30 chars
   - Email: valid + only `@gmail.com` / `@yahoo.com` / `@aptechsite.net`
   - Phone: exactly 11 digits
   - Password: min 8 chars, 1 uppercase + 1 lowercase + 1 number, confirm match
2. Duplicate email check
3. `password_hash(PASSWORD_DEFAULT)` → INSERT
4. **PHPMailer** (Gmail SMTP, port 587 TLS) → welcome email
5. Redirect → `login.php`

### 5.2 Book Appointment (3-Step) — `appointment.php`
```
Step 1: Date select (min = today)
Step 2: Slot generation
   ├── doctor_availability check (day_of_week = selected day)
   ├── doctor_leaves check (full-day ya partial overlap)
   ├── 30-min slots generate hote hain (start→end time)
   └── booked slots filter (status != cancelled)
Step 3: Summary + Confirm → INSERT status='pending'
```

### 5.3 Doctor Appointment Management — `doctor_final/doctor_output/appointments.php`
- Tabs: **Today / Upcoming / Past**
- Status update: `confirmed` / `cancelled` / `completed` (CSRF check + prepared statement ✓)
- Notes: `completed`/`cancelled` hamesha "Past" tab mein jaate hain

### 5.4 Doctor Availability — `availability.php`
- Weekly schedule upsert (7 din, start/end time + is_available)
- Leaves: full-day ya partial (start/end time ke saath), duplicate date block

### 5.5 Admin Feedback Moderation — `admin/dashboard.php`
- Edit: `feedback_status` update (0/1 — homepage testimonials control)
- Delete feedback row
- ⚠️ Raw queries use hoti hain (SQL injection risk — dekho Section 7)

---

## 6. Frontend / UI Architecture

| Area | Tech |
|---|---|
| Framework | Bootstrap 5.3 (CDN) |
| Icons | Font Awesome 6.4 (public/patient/doctor), Bootstrap Icons (admin) |
| Fonts | Poppins + Playfair Display (public), Plus Jakarta Sans (panels), DM Serif (feedback page) |
| CSS | `css/style.css` (main), `patient_panel/style.css`, `doctor_final/css/style.css`, admin/assets |
| Animations | `animate-on-scroll` CSS classes, carousel |
| Design | Card-based UI, gradient accents, badge statuses (pending/confirmed/cancelled/completed) |

**Public site pages:** Home → Search Doctor → Appointment → Diseases → Medical News → About → Contact → Feedback.

**Design inconsistency note:** Alag panels alag fonts/colors use karte hain (public = Poppins, patient/doctor = Plus Jakarta Sans, feedback = DM Sans). Unified design system ho sakta hai.

---

## 7. ⚠️ Issues & Security Findings (Code Review)

### 🔴 Critical
1. **SQL Injection — multiple files**
   - `admin/dashboard.php`: feedback delete/update direct `$_POST` values query mein:
     ```php
     $delete_feedback = "delete from feedback where feedback_id='$get_feedback_id'";
     ```
   - `register.php`: `mysqli_real_escape_string()` use hota hai but prepared statement nahi.
   - `index.php`, `search-doctor.php`, `diseases.php`, `admin/doctors.php` etc. — string-concatenated queries.
   - **Fix:** Har jagah `mysqli_prepare()` + `bind_param()` (jaise `appointment.php` aur panels mein already hai ✓).

2. **Gmail App Password hard-coded** — `register.php`:
   ```php
   $mail->Password = 'jjpc paeo hwqu dkzn';
   ```
   Ye public repo mein leak ho chuka hai. **Turant Google account se ye app password revoke karo** aur env variable/config file mein daalo.

3. **Admin registration publicly accessible** — `admin/admin_register.php` koi bhi khol sakta hai aur naya admin bana sakta hai. Deploy hone ke baad is file ko remove/lock karo.

### 🟠 High
4. **No CSRF on admin + public forms** — admin feedback delete/edit, contact/feedback forms token check nahi karte.
5. **`doctor_availability` overwrite risk** — availability.php mein upsert per-day hai, koi transaction nahi.
6. **`echo $today;` debug line** — `doctor_final/doctor_output/appointments.php` line 4 par raw date print ho rahi hai. ⚠️ **Changed file** — remove karna chahiye.

### 🟡 Medium
7. **Output escaping missing** — kai jagah `htmlspecialchars()` nahi lagaya (e.g. `admin/dashboard.php` doctor/patient/feedback tables, `index.php` testimonials `<?= $row['message'] ?>`). Stored XSS possible.
8. **No pagination** — search results LIMIT 9 / full tables bina pagination ke.
9. **Session fixation** — `session_regenerate_id(true)` login ke waqt call nahi hota.
10. **Upload validation** — doctor image upload (`admin/doctors.php`) mein file type/size validation check karna chahiye.
11. **Mixed languages** — UI text English + Roman Urdu mix (e.g. "Koi doctor nahi mila.").

### ✅ Jo Achha Hai
- Prepared statements patient/doctor panels ke core flows mein (appointment, availability, auth)
- `password_hash` / `password_verify` har jagah (koi plain-text password nahi)
- CSRF implementation patient/doctor panels mein
- Role-based routing + auth guards har panel mein
- Session cookie flags (httponly, samesite, secure)
- Clean 3-step booking UX with double-booking prevention
- `window.top !== window.self` frame-busting (clickjacking protection) doctor panel mein

---

## 8. Setup & Run (XAMPP)

1. **XAMPP install** karo → Apache + MySQL start karo
2. Project copy karo: `C:\xampp\htdocs\health-care\`
3. **Database import:**
   - phpMyAdmin kholo → `http://localhost/phpmyadmin`
   - New database banao: `health_care`
   - SQL import karo (agar .sql file hai) ya tables manually banao (Section 3 dekho)
4. **DB credentials** (`connect.php`) default XAMPP se match:
   ```php
   $connect = mysqli_connect("localhost", "root", "", "health_care");
   ```
5. Browser mein kholo: `http://localhost/health-care/`
6. **PHPMailer ke liye:** apna Gmail + App Password daalo (`register.php` mein hard-coded hai)
7. Admin panel: `http://localhost/health-care/admin/login.php`

### Default Test Flow
```
Admin:     admin/login.php → register → login → doctors/cities/specs add karo
Doctor:    admin panel se doctor add hota hai (login credentials ke saath)
Patient:   register.php → email verify → login → search-doctor → book
Doctor:    login → availability set karo → appointments confirm/cancel
```

---

## 9. Page Reference Table

| Page | URL | Access | Purpose |
|---|---|---|---|
| `index.php` | `/` | Public | Homepage, featured doctors, testimonials |
| `register.php` | `/register.php` | Public | Patient signup + email |
| `login.php` | `/login.php` | Public | Patient/Doctor login |
| `search-doctor.php` | `/search-doctor.php` | Public | Doctor directory + filters |
| `appointment.php` | `?doctor_id=X` | Patient | 3-step booking |
| `diseases.php` | Public | Diseases guide |
| `disease_news.php` | Public | Medical news |
| `feedback.php` | Public | Feedback form |
| `patient_panel/dashboard.php` | Patient | Stats + recent appts |
| `patient_panel/my_appointments.php` | Patient | History + cancel |
| `patient_panel/profile.php` | Patient | Profile edit |
| `doctor_final/doctor_output/dashboard.php` | Doctor | Today's schedule |
| `doctor_final/doctor_output/appointments.php` | Doctor | Manage all appts |
| `doctor_final/doctor_output/availability.php` | Doctor | Weekly schedule + leaves |
| `admin/login.php` | Admin | Admin login |
| `admin/dashboard.php` | Admin | Overview + feedback moderation |
| `admin/doctors.php` | Admin | Doctor management |
| `admin/patients.php` | Admin | Patients list |
| `admin/cities.php`, `specializations.php`, `diseases.php` | Admin | Master data CRUD |

---

## 10. Suggested Improvements (Roadmap)

| Priority | Improvement |
|---|---|
| 🔴 P0 | Gmail app password revoke + config se bahar rakho |
| 🔴 P0 | `admin_register.php` ko deploy se pehle remove karo |
| 🔴 P0 | Admin panel mein prepared statements |
| 🟠 P1 | Har output par `htmlspecialchars()` |
| 🟠 P1 | CSRF tokens admin + feedback/contact forms mein |
| 🟠 P1 | Doctor image upload validation (MIME + size + random filename) |
| 🟡 P2 | `session_regenerate_id(true)` after login |
| 🟡 P2 | Pagination (search, admin tables) |
| 🟡 P2 | Email verification flow (token-based) instead of plain welcome mail |
| 🟡 P2 | Password reset / "forgot password" feature |
| 🟢 P3 | Design system unify (ek font/color palette sab panels) |
| 🟢 P3 | Appointment reminders (cron + email) |
| 🟢 P3 | Doctor rating system (feedback se link) |
| 🟢 P3 | `.sql` export repo mein daalo (documentation + setup ke liye) |

---

*Documentation generated: September 29, 2026 — CARE Group Health Care Portal*
