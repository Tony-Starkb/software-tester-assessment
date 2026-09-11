# software-tester-assessment

- name: Keshav Agarwal
- e-mail: agarwalkeshav647@gmail.com
- phone: 9045403508

# Assumptions: 
- It's a web application
- Email + password is the primary registration/login method
- There's a backend + database storing tasks


## 1. Registration

Main thing to check here is whether the user's entered details (mainly email +
password) are valid, and the rest of the profile info is secondary.

For email verification, there are basically two common approaches apps use:
1. Google Auth
2. OTP sent to the entered email, and the user types it back in

For password, the main thing is checking it against whatever format/policy the app
has set (length, special char, etc.)

### Test scenarios
- Register with a valid, real email and correct OTP
- Register with wrong/expired OTP
- Register using Google Auth
- Register with an email that already exists in the system
- Password and Confirm Password mismatch
- Spamming OTP requests

### Test cases

| TC ID | Scenario | Steps | Test Data | Expected Result | Type |
|---|---|---|---|---|---|
| TC_REG_01 | Valid email + correct OTP | Enter email → request OTP → enter received OTP | test@gmail.com | OTP matches, registration goes through | Positive |
| TC_REG_02 | Wrong OTP entered | Enter email → request OTP → type incorrect OTP | 000000 | Error - "Invalid OTP", registration blocked | Negative |
| TC_REG_03 | OTP expired (waited too long) | Request OTP → wait past expiry (~5 min) → enter it | expired OTP | Error - OTP expired, resend option shown | Negative |
| TC_REG_04 | Register with Google account | Click "Sign up with Google" → select account | valid gmail account | Account created directly, no OTP step | Positive |
| TC_REG_05 | Fake/non-existent email used | Enter fake email → request OTP | fake123@notreal.com | OTP not delivered, registration doesn't proceed | Negative |
| TC_REG_06 | Password matches policy | Enter password with required format | Pass@1234 | Accepted | Positive |
| TC_REG_07 | Weak password | Enter simple/short password | 1234, abcd | Validation error shown | Negative |
| TC_REG_08 | Password too short | Enter under min length | Ab1@ | Error - min length not met | Edge |
| TC_REG_09 | Extremely long password | Enter 100+ char password | long string | Should not crash - either accepted or proper max length error | Edge |
| TC_REG_10 | Confirm password doesn't match | Enter password, then different value in confirm field | Pass@123 / Pass@124 | Error - passwords don't match | Negative |
| TC_REG_11 | Duplicate email registration | Try registering with already-used email | existing@gmail.com | Error - email already registered | Negative |
| TC_REG_12 | Repeated OTP requests in short time | Click resend OTP multiple times fast | - | Should have some cooldown/rate limit, otherwise spam risk | Edge |

(honestly not sure if this app even has OTP/Google auth built in since the JD doesn't
say - but wanted to show I considered the auth verification angle since that's usually
the trickiest part of registration to get right)


## 2. Login

### Test scenarios
- Correct email + correct password
- Correct email + wrong password
- Login attempt with unregistered email
- Empty fields on submit
- Case sensitivity of email field

### Test cases

| TC ID | Scenario | Test Data | Expected Result | Type |
|---|---|---|---|---|
| TC_LOG_01 | Valid email + valid password | correct creds | Logged in, redirected to dashboard | Positive |
| TC_LOG_02 | Valid email + wrong password | correct email, wrong pw | Generic error shown (shouldn't say "wrong password" specifically - security reasons) | Negative |
| TC_LOG_03 | Email not registered | random@xyz.com | Same generic error as above (so attacker can't tell which part was wrong) | Negative |
| TC_LOG_04 | Empty email/password fields | blank | Validation triggers before hitting server | Negative |
| TC_LOG_05 | Email entered in different case | TEST@gmail.com vs test@gmail.com | Should still log in fine | Edge |
| TC_LOG_06 | Repeated wrong password attempts | 5+ tries | Account lock / captcha triggered (if this is implemented) | Edge |
| TC_LOG_07 | Login, then check session timeout | leave idle for long period | Session should expire and require re-login | Edge |
| TC_LOG_08 | SQL injection attempt in login field | ' OR '1'='1 | Should be sanitized, login fails safely | Negative |

---

## 3. Task CRUD

### Test scenarios
- Create a task with valid data
- Create task with missing required fields
- View task list (empty state + long list)
- Edit an existing task
- Delete a task

### Test cases

| TC ID | Scenario | Test Data | Expected Result | Type |
|---|---|---|---|---|
| TC_TASK_01 | Create task with all fields filled | title, description, due date | Task appears in list immediately | Positive |
| TC_TASK_02 | Create task with empty title | title left blank | Validation error, task not created | Negative |
| TC_TASK_03 | View task list when no tasks exist | new user, 0 tasks | Empty state message shown, not blank/broken screen | Edge |
| TC_TASK_04 | View task list with a large number of tasks | 100+ tasks | List loads properly, pagination or scroll works without lag | Edge |
| TC_TASK_05 | Edit an existing task's title | change title | Updated title reflects in list right away | Positive |
| TC_TASK_06 | Edit task and leave title blank | clear title, save | Should block save with validation error | Negative |
| TC_TASK_07 | Delete a task | select delete on a task | Task removed from list and from DB (shouldn't reappear on refresh) | Positive |
| TC_TASK_08 | Try editing a task that's already been deleted (e.g. in another tab) | edit deleted task id | Should show proper error, not crash the app | Edge |
| TC_TASK_09 | Create task with special characters/emoji in title | Test Task <>&! | Should save and display correctly without breaking UI | Edge |
| TC_TASK_10 | Delete task - check for confirmation step | click delete | Ideally a confirm popup before actual deletion (avoids accidental loss) | Positive/UX |

---


## Bugs / Risk Areas 

**1. Password possibly not encrypted in transit/storage** — Severity: Critical
If it's stored or sent as plain text, that's a major security hole. Common mistake
in early-stage apps.

**2. Login error messages might reveal too much info** — Severity: Major
If "wrong password" and "email not found" show different messages, it lets an
attacker figure out which emails are registered.

**3. No confirmation before deleting a task** — Severity: Major
Easy to lose data by mis-click if there's no "are you sure" step.

**4. Duplicate email not checked at registration** — Severity: Major
Could lead to broken login later if two accounts share the same email somehow.

**5. No pagination/lazy loading for task list** — Severity: Minor
Might work fine for 10 tasks but will get slow/laggy once list grows.

**6. XSS - task titles/descriptions might not be sanitized** — Severity: Critical
If someone enters a script tag in a task name, it needs to be treated as plain text,
not executed by the browser.

**7. No rate limiting on OTP or login attempts** — Severity: Major
Opens door to spam/brute force if not handled.
