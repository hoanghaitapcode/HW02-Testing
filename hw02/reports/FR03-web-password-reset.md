# FR-03 - Web forgot password and password reset

Report stage: Domain Testing and Boundary Value Analysis design.
Status: AI-assisted draft awaiting student review; all tests Not executed.
Date: 2026-10-17 (Asia/Bangkok).
SUT baseline inspected: `85af3ba875c88283615e22cb108f13e2fccaf0e9`, with pre-existing local database changes.

## 1. Objective, scope, and sources

Verify the two-step web password-reset feature against its business requirements. Cover email eligibility, OTP shape and account binding, password constraints, confirmation matching, and required navigation. Include supporting OTP lifecycle checks separately.

Sources relative to the clone root:

- `README.md`, FR-03: two-step reset requirements.
- `README.md`, FR-01: password rules inherited by FR-03.
- `README.md`, FR-22: email input type, masked passwords, required-field markers, error placement, and step indicator.
- `README.md`, SEC-07: supplementary OTP randomness, lifetime, and single-use requirements; these appear under reference security requirements.
- `api_specification.md`, sections 1.2-1.4: login and password-reset endpoints.

`frontend-web/src/pages/ForgotPassword.jsx` and `backend/server.js` were inspected for testability and possible defects. Their current behavior is not the expected-result oracle. Course-specific lecture material was not located during this step; the student must align the technique terminology with their lectures before submitting.

No password reset, account creation, login, or other feature execution was performed to produce this report. No bug is runtime-confirmed here.

## 2. Requirements and traceability IDs

The following R03 IDs are report-local references, not additional official feature IDs.

| ID | Requirement | Source |
| --- | --- | --- |
| R03-01 | Step 1 accepts a registered email as the reset target | FR-03 |
| R03-02 | Generate a random six-digit OTP; display it directly in the demo | FR-03 |
| R03-03 | Show a step indicator identifying the current step of two | FR-03; FR-22 |
| R03-04 | Provide a return-to-login control | FR-03 |
| R03-05 | Step 2 includes OTP, new password, and confirmation | FR-03 |
| R03-06 | New password contains at least 8 characters | FR-03 -> FR-01 |
| R03-07 | New password contains at least one uppercase letter | FR-03 -> FR-01 |
| R03-08 | New password contains at least one lowercase letter | FR-03 -> FR-01 |
| R03-09 | New password contains at least one digit | FR-03 -> FR-01 |
| R03-10 | New password contains at least one character from `@ $ ! % * ? &` | FR-03 -> FR-01 |
| R03-11 | New password and confirmation must match | FR-03 |
| R03-12 | An OTP is bound to its requesting email and cannot reset another account | FR-03 |
| R03-13 | OTP has a lifetime and becomes invalid after successful use | SEC-07, supplementary |
| R03-14 | Email input uses HTML email validation; passwords are masked; required fields are marked; errors appear above submit | FR-22, supporting form checks |

A successful reset must persist the new password for the target account. Verify this through subsequent login with the new password and rejection of the previous password. This is a derived check of password replacement, not a claim that every existing session must be revoked.

## 3. Step-by-step Domain Testing procedure

### Step 1 - Identify the input space and state

Inputs: email `e`, entered OTP `t`, new password `p`, and confirmation `c`.
Relevant state: whether account `e` exists; its issued OTP and validity; whether the OTP has already been used; current UI step.

The API calls are `POST /api/forgot-password` with `email`, and `POST /api/reset-password` with `email`, `resetToken`, and `newPassword`. The documented reset API has no confirmation field. Confirmation matching is therefore tested through the web form; adding an undocumented confirmation key to the API cannot establish UI behavior.

### Step 2 - Express the valid domain using predicates

For a successful reset, all of the following must hold:

```text
Registered(e)
AND SixDigits(t)
AND IssuedFor(t, e)
AND Length(p) >= 8
AND UppercaseCount(p) >= 1
AND LowercaseCount(p) >= 1
AND DigitCount(p) >= 1
AND AllowedSpecialCount(p) >= 1
AND p = c
```

For supplementary SEC-07 coverage also require `NotExpired(t)` and `NotUsed(t)`.
`SixDigits(t)` is modeled as `^[0-9]{6}$`; this ASCII interpretation is a working assumption. Password character classes use ASCII letters/digits for controlled samples; Unicode policy remains unspecified.

Password specials are an inclusion condition: at least one listed symbol is required. The specification does not explicitly ban other additional characters. A password containing a space *and* a listed symbol is not automatically invalid.

### Step 3 - Partition valid and invalid domains

| Dimension | Valid partition | Invalid partition | Unknown / qualification |
| --- | --- | --- | --- |
| Email | Registered, correctly formatted address | Empty, malformed, or unregistered address cannot authorize a successful reset | Exact response for an unknown account may be generic; do not require account enumeration |
| OTP format | Exactly six digits | Empty; 1-5 digits; 7+ digits; non-digit characters | Generated values must also satisfy format; mere format does not prove token validity |
| OTP binding | Current OTP issued for this email | Wrong six-digit token; OTP issued only for a different email | Choose different actual tokens when isolating cross-account binding |
| OTP lifecycle | Issued, unused, unexpired | Used or expired token, under SEC-07 | Expiry duration and replacement policy are unspecified |
| Password length | At least 8 characters | 0-7 characters | No maximum specified |
| Uppercase count | At least 1 | 0 | Use ASCII samples to isolate the class |
| Lowercase count | At least 1 | 0 | Same |
| Digit count | At least 1 | 0 | Same |
| Listed-special count | At least 1 | 0 | Space or `#` alone does not satisfy the listed-symbol requirement |
| Confirmation | Exact match to valid new password | Empty or non-matching confirmation | Whitespace/case normalization must not be assumed |

### Step 4 - Select in, on, and off points

Use a satisfying interior password such as `Abcd123!x` (9 characters), plus boundary passwords and single-constraint violations. For the length lower boundary, an on point is `Aa1!xyzz` (8), an off point is `Aa1!xyz` (7), and an in point is `Aa1!xyzzz` (9).

For each minimum character-class count, select a value with exactly one required character (on), zero (off), and two (in) while holding other requirements valid. The test table contains these explicit count variations.

For exact OTP length, the on domain is length 6; lengths 5 and 7 are adjacent off-domain points. There is no interior length within an exact-length domain. Digits are not an ordered OTP magnitude: values such as `000000` or `999999` are not meaningful numeric boundaries unless actually issued by the SUT.

For confirmation equality and account binding, use categorical on/off relation points (match/mismatch and same/different account), not invented numeric boundaries.

### Step 5 - Control other conditions and cover interactions

Use a fresh issued OTP for the intended disposable account for each reset attempt. Keep all other predicates valid when checking one invalid password rule. Add combined-invalid input as a robustness probe after individual rules; it cannot replace isolated coverage. A 5- or 7-digit OTP is inherently also not the issued six-digit token, so token-format and token-match effects cannot be completely separated in a black-box reset request.

### Step 6 - Define expected outcomes and review coverage

Valid resets must change only the intended account's password. Rejected resets must leave its password unchanged. Capture both the visible result and post-state verification. UI feedback or an HTTP success alone is insufficient.

The table below maps samples to requirements. After execution, review failed/blocked cases and add missed domains. No claim of exhaustive email grammar, Unicode, randomness, or all combinations is made.

## 4. Test data and common procedures

Prepare two disposable registered accounts A and B, using unique test emails under a controlled local test domain, and a well-formed address U confirmed absent from the test database. Record their actual addresses and initial passwords in a private execution record. Do not reset shared/admin accounts for this design.

Symbols: `E_A` / `E_B` = actual registered addresses; `T_A` / `T_B` = actual fresh OTPs; `P0` = account's recorded current password; default `P1 = Abcd123!x`; default confirmation `C = P1`. Never substitute a made-up six-digit token for a valid issued token. If the SUT issues a malformed OTP, record that failure and mark dependent success cases Blocked instead of fabricating a valid state.

Common procedures referenced in the table:

- **REQ:** Open web `/forgot-password`; enter the given email; request an OTP; record UI step, displayed OTP, response, and any failure. Do not expect a particular error message for unknown accounts.
- **RESET:** Run REQ for A; enter the stated token/password/confirmation in step 2; submit; capture feedback. On success, verify new-password login succeeds and previous-password login fails. On rejection, verify P0 still works and proposed new password does not. Avoid repeated failed logins that could trigger FR-02 lockout; reset the execution state or wait for the documented lockout before judging a result.
- **BIND:** Prepare distinct tokens for A and B, ensuring the token being injected is not also the target's currently issued token. In a separately labeled API request, attempt reset for B using only A's token. Verify B's password is unchanged. If tokens collide, regenerate before running the probe.
- **VIEW:** Inspect the stated web control/step and record screenshot or DOM evidence; no reset submission unless specified.

Per case, record browser/version, SUT commit/local changes, test-data identity, date/time, actual result, status, evidence path, and bug link if applicable. If an earlier independent defect prevents reaching the target check, mark Blocked and record the dependency. A separate API probe can investigate backend behavior, but cannot turn a blocked UI case into a UI pass.

## 5. Domain Testing test cases

For D10-D22, use RESET with `E_A`, `T_A`, and matching confirmation of the specified password unless the row says otherwise. All other baseline predicates must be valid.

| ID | Requirement | Procedure / selected input | Expected result | Actual / status |
| --- | --- | --- | --- | --- |
| D01 | R03-01,02 | REQ with E_A | OTP issued for A, displayed in demo, exactly 6 digits | Not recorded / Not executed |
| D02 | R03-01 | REQ with empty email | No successful reset authorization; required-input feedback | Not recorded / Not executed |
| D03 | R03-01,14 | REQ with `plainaddress` | Malformed email not accepted as a valid reset target | Not recorded / Not executed |
| D04 | R03-01 | REQ with absent address U | No usable reset authorization for a nonexistent account; generic response permitted | Not recorded / Not executed |
| D05 | R03-02,05-12 | RESET with issued T_A, P1, matching C | Reset succeeds; new password works and P0 stops working | Not recorded / Not executed |
| D06 | R03-02,12 | RESET with a six-digit token verified different from T_A | Reset rejected; password unchanged | Not recorded / Not executed |
| D07 | R03-02,05 | RESET with empty OTP | Cannot complete reset; password unchanged | Not recorded / Not executed |
| D08 | R03-02 | RESET with `12345a` as OTP | Reset rejected; password unchanged | Not recorded / Not executed |
| D09 | R03-12 | BIND with E_B and T_A | B's password unchanged; A's token cannot authorize B's reset | Not recorded / Not executed |
| D10 | R03-06 | Password empty | Reset rejected; password unchanged | Not recorded / Not executed |
| D11 | R03-07 | Password `abcd123!` (zero uppercase) | Reset rejected; password unchanged | Not recorded / Not executed |
| D12 | R03-08 | Password `ABCD123!` (zero lowercase) | Reset rejected; password unchanged | Not recorded / Not executed |
| D13 | R03-09 | Password `Abcdefg!` (zero digits) | Reset rejected; password unchanged | Not recorded / Not executed |
| D14 | R03-10 | Password `Abcd1234` (zero listed specials) | Reset rejected; password unchanged | Not recorded / Not executed |
| D15 | R03-10 | Password `Abcd123 ` (space, zero listed specials) | Reset rejected; space does not replace a listed special | Not recorded / Not executed |
| D16 | R03-10 | Password `Abcd123#` (unlisted symbol only) | Reset rejected; listed-special predicate unsatisfied | Not recorded / Not executed |
| D17 | R03-10 | Seven separate trials: `Abcd123@`, `Abcd123$`, `Abcd123!`, `Abcd123%`, `Abcd123*`, `Abcd123?`, `Abcd123&` | Each meets password rules; each reset succeeds with its own fresh OTP and matching C | Not recorded / Not executed |
| D18 | R03-07 | `Abcd123!` then `ABcd123!`, fresh OTP for each (uppercase count 1 then 2) | Both satisfy uppercase requirement and succeed | Not recorded / Not executed |
| D19 | R03-08 | `ABCd123!` then `ABcd123!`, fresh OTP for each (lowercase count 1 then 2) | Both satisfy lowercase requirement and succeed | Not recorded / Not executed |
| D20 | R03-09 | `Abcdef1!` then `Abcde12!`, fresh OTP for each (digit count 1 then 2) | Both satisfy digit requirement and succeed | Not recorded / Not executed |
| D21 | R03-10 | `Abcd123!` then `Abcd12!?`, fresh OTP for each (listed-special count 1 then 2) | Both satisfy special requirement and succeed | Not recorded / Not executed |
| D22 | R03-06-10 | Password `abc` (multiple violations) | Reset rejected; password unchanged; exact error ordering unspecified | Not recorded / Not executed |
| D23 | R03-11 | RESET with P1; confirmation `Abcd123!y` | Reset rejected; password unchanged | Not recorded / Not executed |
| D24 | R03-05,11 | RESET with P1; empty confirmation | Cannot complete reset; password unchanged | Not recorded / Not executed |
| D25 | R03-11 | RESET with P1; confirmation `abcd123!x` | Case mismatch rejected; password unchanged | Not recorded / Not executed |
| D26 | R03-13 | After a successful reset, reuse the same email/OTP to set a different valid password | Reused OTP rejected; first successful password remains current (supplementary SEC-07) | Not recorded / Not executed |
| D27 | R03-13 | Attempt reset after confirmed configured OTP lifetime expires | Expired OTP rejected; password unchanged; blocked if expiry oracle is unavailable | Not recorded / Not executed |
| D28 | R03-03 | VIEW step 1, then request OTP and VIEW step 2 | Current step / total steps clearly shown on each | Not recorded / Not executed |
| D29 | R03-04 | VIEW return-to-login control and activate it | Control exists and leads to login | Not recorded / Not executed |
| D30 | R03-05,14 | VIEW step 2 | OTP, new password, and confirmation inputs exist; both password fields masked | Not recorded / Not executed |
| D31 | R03-14 | VIEW email input and required labels; trigger invalid submission | Email type/validation and required markers present; application errors above submit; distinguish native browser validation feedback | Not recorded / Not executed |

Rows with multiple trials must be expanded into individual execution records, e.g. D17-@ through D17-& and D18-1/D18-2. Do not count the table-row total as the number of executed or atomic designed tests. All are currently unexecuted.

## 6. Boundary Value Analysis

Use boundary-minus-one, boundary, and boundary-plus-one where an ordered measure is defined. Password samples retain every other required character class and a matching confirmation. OTP samples below test length, not numerical magnitude or successful token matching.

| ID | Requirement / boundary | Input | Expected result | Actual / status |
| --- | --- | --- | --- | --- |
| B01 | R03-06; length 8, below | RESET with `Aa1!xyz` (7) | Rejected; password unchanged | Not recorded / Not executed |
| B02 | R03-06; length 8, on | RESET with `Aa1!xyzz` (8) | Accepted with valid issued OTP and matching C | Not recorded / Not executed |
| B03 | R03-06; length 8, above | RESET with `Aa1!xyzzz` (9) | Accepted with valid issued OTP and matching C | Not recorded / Not executed |
| B04 | R03-02; exact length 6, below | RESET with OTP `12345` (5) and P1 | Rejected; password unchanged | Not recorded / Not executed |
| B05 | R03-02; exact length 6, on | RESET with actual six-digit T_A and P1 | Accepted if all remaining predicates hold | Not recorded / Not executed |
| B06 | R03-02; exact length 6, above | RESET with OTP `1234567` (7) and P1 | Rejected; password unchanged | Not recorded / Not executed |

Additional count-boundary coverage is provided by D11/D18 (uppercase 0/1/2), D12/D19 (lowercase 0/1/2), D13/D20 (digit 0/1/2), and D14/D21 (listed special 0/1/2). These counts are input-content measures, not new business constraints.

No maximum password/email length or OTP expiry duration is specified. Do not invent a 255-character limit or a five-minute lifetime. Once expiry `T` is clarified, add probes immediately before, at, and after `T`, with units and inclusivity specified by the clarified oracle. UI step number is a workflow state, not a numeric input boundary.

## 7. Ambiguities, assumptions, and exploratory follow-up

| Topic | Current treatment |
| --- | --- |
| Email case folding, trimming, Unicode, aliases | No assumed normalization; use exact registered email for baseline and clarify before labeling normalization failures |
| Password maximum, Unicode counting, leading/trailing whitespace | Unspecified; use controlled ASCII baseline and exploratory probes without invented pass/fail rules |
| OTP expiry duration and boundary inclusivity | Obtain an authoritative value before time BVA; implementation absence is a concern, not a substitute specification |
| Requesting another OTP | Whether previous OTP is invalidated is unspecified; record behavior separately |
| OTP leading zeros | Treat token as a string; six-digit lexical format is assumed; no numeric range 100000-999999 is imposed |
| Randomness | One token or a few distinct outputs do not prove randomness/entropy; combine source review and a separately scoped assessment |
| Unknown account message | Generic success/failure is permissible here; no valid reset may be authorized for an absent account |
| Post-reset navigation/session revocation | Persistence is checked through login; exact destination and session-revocation policy are not specified by FR-03 |
| Error response codes/text | Check correct outcome and useful feedback; do not demand an undocumented exact status/string |

## 8. Static concerns and planned AI gap review

These observations are from source inspection only. Reproduce and capture runtime evidence before opening confirmed bug reports.

| Source observation | Requirement affected | Target cases |
| --- | --- | --- |
| Backend generates a 4-digit token; UI label also says 4 digits | R03-02 | D01; B04-B06 |
| Web password regex requires whitespace and excludes the documented listed symbols | R03-06-10 | D05; D11-D22; B01-B03 |
| No confirmation input is rendered | R03-05,11 | D23-D25; D30 |
| Current form does not render a step indicator or a dedicated return-to-login control | R03-03,04 | D28-D29 |
| Email input uses `type="text"` | R03-14 | D03; D31 |
| Backend reset handler has no visible password-strength check | R03-06-10 | Separate supporting API probes using weak passwords if UI execution is blocked |
| Backend clears token on successful reset, but no expiry check is visible in the inspected reset handler | R03-13 | D26-D27 and further source investigation |

AI gap analysis is pending human review and execution. Do not claim that AI missed or found a runtime bug without evidence. Record each later omission, the added case, why the earlier analysis missed it, and the student's correction. Static observations may guide investigation but must not be copied into actual-result columns.

## 9. Execution and evidence record template

Copy this record for every atomic trial:

```text
Case ID:
Requirement / technique:
Date and time / tester:
Browser and OS / SUT commit and local changes:
Preconditions / disposable account / current password state:
Steps and exact input (redact live secrets from shared evidence):
Expected result:
Actual result:
Post-state verification:
Status: Not executed | Pass | Fail | Blocked
Evidence path:
Bug issue URL (if confirmed):
Student review / correction:
```

Current summary: 31 Domain Testing design rows and 6 BVA design rows, with parameterized trials to expand and some shared coverage. Executed: 0; passed: 0; failed: 0; runtime-confirmed bugs: 0. The table rows are not a finalized unique-test total.

## 10. Review and commit checkpoint

- [ ] Review requirements, data lengths, domain coverage, and lecture-specific technique.
- [ ] Resolve or explicitly retain the listed ambiguities.
- [ ] Commit the design before executing tests; keep results in later commits.
- [ ] Execute FR-03 and attach evidence in a subsequent step.
- [ ] Write Markdown bugs and post confirmed bugs with screenshots on the group's GitHub Issues when authorized.

This report covers FR-03 only. FR-10 and FR-19 reports and the full FR-04 test-case report will be written after the student's commit checkpoints. The earlier FR-04 file remains a preparation analysis.
