# FR-04 - Mobile personal profile: requirements and domain analysis

Status: AI-assisted analysis draft for student review; not an execution report.
Prepared: 2026-10-17 (Asia/Bangkok).

## 1. Scope and sources

Pool D feature: personal profile management on the React Native mobile app. FR-20 includes mobile profile functionality; the student's clarified allocation applies FR-04 rules to mobile.

Primary sources, relative to the EShop clone root:

- `README.md`, FR-04: profile business rules.
- `README.md`, FR-20: mobile feature availability.
- `README.md`, SEC-02 and SEC-06: supporting authentication and role restrictions.
- `api_specification.md`, sections 2.1 and 2.2: profile API contract.

Implementation inspected for orientation: `frontend-mobile/App.js` and `backend/server.js`. Implementation behavior must not replace the specification as the expected-result oracle. No external telephone-number standard is assumed.

In scope: editable profile fields, phone validation, email UI restriction, ownership, role protection, and update persistence. Order history and logout are outside the main FR-04 scope even though they appear on the same screen. API probes may support shared-backend investigation but cannot establish that a mobile interaction passed.

## 2. Requirement inventory

IDs below are local analysis IDs, not additional official FR numbers.

| ID | Source | Requirement / expected behavior |
| --- | --- | --- |
| R04-01 | FR-04 | An authenticated user can update their full name. |
| R04-02 | FR-04 | An authenticated user can update their phone number. |
| R04-03 | FR-04 | An authenticated user can update their default shipping address. |
| R04-04 | FR-04 | A valid supplied phone number starts with `0`. |
| R04-05 | FR-04 | A valid supplied phone number contains only digits and has length 10 or 11. |
| R04-06 | FR-04 | Email cannot be changed through the UI. |
| R04-07 | FR-04 | A user can update only their own profile. |
| R04-08 | FR-04; SEC-06 | A user cannot change their own role through profile updates. |
| R04-09 | FR-04; SEC-02; API section 2 | Protected profile operations require valid authentication. |

Persistence after reading the profile again is a derived check of the term "update": an apparent success in local UI state alone is insufficient. Exact response codes, rejection wording, and whether forbidden payload fields are ignored or rejected are not specified by FR-04. Judge protected-state changes independently of those presentation details.

## 3. Inputs and observable results

| Input / context | Documented meaning | Analysis notes |
| --- | --- | --- |
| `name` | Full name | Editable; requiredness, trimming, allowed characters, and length bounds unspecified |
| `phone` | Telephone number | Preserve as text, including the leading zero; defined format constraints below |
| `shipping_address` | Default shipping address in API contract | Editable; length bounds and empty-value rules unspecified |
| Email UI field | Current account email | Must not permit editing through the UI |
| Authentication | `Authorization: Bearer <token>` | Valid user identity is the owner context |
| Extra identity fields | Attempted `id` / `user_id` / other identity overrides | Exploratory API inputs; cannot authorize a different profile update |
| Extra `role` field | Attempt to change account privilege | Must not change the user's role |

Observe: field editability, validation feedback, update result, saved values from a fresh authenticated read, unchanged email/role, and unchanged second-account data. Capture screenshots and API evidence later without including live tokens or real personal data.

## 4. Domain definition

For a nonempty supplied phone string `p`, define:

`ValidPhone(p) = DigitsOnly(p) AND StartsWithZero(p) AND 10 <= Length(p) <= 11`

Working interpretation: digits are ASCII `0` through `9`. An analysis oracle is `^0[0-9]{9,10}$`; it expresses the written rule, not a proposed fix to the SUT. Unicode digit acceptance requires clarification. Requiredness of the field is a separate question from the validity of a supplied phone number.

| Dimension | Valid domain | Invalid domain | Unspecified / exploratory domain |
| --- | --- | --- | --- |
| Phone length (nonempty, digit-only) | 10 or 11 | 1-9; 12 or more | Empty, missing, null |
| First character (10-11 digits) | `0` | `1`-`9` | Input normalization before validation |
| Character set (10-11 characters, starts `0`) | ASCII digits under working interpretation | Letters, punctuation, internal spaces | Unicode digits, trimming surrounding whitespace |
| Full name | Ordinary editable name as a baseline | No explicit invalid character or length domain specified | Empty, whitespace-only, punctuation, Unicode, unusually long values |
| Address | Ordinary editable address as a baseline | No explicit invalid length/character domain specified | Empty, whitespace-only, multiline, Unicode, unusually long values |
| Authentication | Valid token for current user | Missing or invalid token cannot authorize updates | Exact failure status/message |
| Ownership | Authenticated user's own profile | Attempt to update another user | Whether extra identity fields are rejected or ignored |
| Role | Original role unchanged after permitted edits | Role changed through a profile request | Whether an injected role field is rejected or ignored |
| Email UI | Read-only display / no editing control | Editable email through profile UI | Backend email-mutation policy beyond the UI restriction |

Phone partitions apply to supplied strings; transport types such as numbers, arrays, objects, and booleans are robustness probes with unspecified exact error handling. Do not assign invented maximum lengths to name or address.

## 5. Representative domain values

These are analysis samples for later test-case design. No case has an actual result yet.

Baseline: valid session for disposable user A, name `Nguyen Van A`, address `123 Le Loi, Quan 1`, phone `0123456789`. Preserve user B as an ownership comparison account. Other baseline fields stay constant while one phone dimension changes.

| Domain ID | Phone value | Length | Expected classification | Basis |
| --- | --- | --- | --- | --- |
| D-P01 | `0123456789` | 10 | Valid | R04-04, R04-05 |
| D-P02 | `01234567890` | 11 | Valid | R04-04, R04-05 |
| D-P03 | `012345678` | 9 | Invalid: below lower bound | R04-05 |
| D-P04 | `012345678901` | 12 | Invalid: above upper bound | R04-05 |
| D-P05 | `1123456789` | 10 | Invalid: first digit | R04-04 |
| D-P06 | `012345678a` | 10 | Invalid: contains letter | R04-05 |
| D-P07 | `01234 6789` | 10 | Invalid: internal space | R04-05 |
| D-P08 | `012345678-` | 10 | Invalid: punctuation | R04-05 |
| D-P09 | Empty string | 0 | Requiredness unresolved; not a valid formatted phone | Q01 |
| D-P10 | ` 0123456789 ` | 12 raw | Normalization unresolved | Q02 |

For valid values, the permitted update should succeed and persist for user A. For invalid supplied formats, the invalid value must not be persisted as a valid phone. Whether other fields are partially updated on rejection is unresolved (Q07). For ambiguous samples, record observations without claiming a requirements failure until the oracle is clarified.

Do not add real-world carrier-prefix restrictions: the specification permits any digit sequence satisfying the stated rule, including synthetic values such as all zeros. Keyboard type alone does not establish validation; later execution should consider paste as well as typing.

## 6. Boundary Value Analysis plan

The specified ordered variable is phone length, with an inclusive valid interval `[10, 11]`. Each boundary sample below contains only digits and starts with `0`, isolating length.

| Boundary | Below | On | Above |
| --- | --- | --- | --- |
| Lower = 10 | 9 digits: invalid | 10 digits: valid | 11 digits: valid |
| Upper = 11 | 10 digits: valid | 11 digits: valid | 12 digits: invalid |

After deduplication, use D-P03, D-P01, D-P02, and D-P04 (9, 10, 11, 12 digits). Both valid lengths are boundaries because the interval is narrow; there is no interior integer length. The baseline 10-digit sample is already included.

This is a three-point-per-boundary selection, subject to alignment with the course lecture's terminology and required method. Prefix, role, and authentication are categorical partitions; do not describe them as numeric BVA. Name/address bounds and token expiry times cannot receive numeric boundary expectations until specified. Empty phone is a separate requiredness probe, not an adjacent length-boundary sample.

## 7. Assumptions and open questions

| ID | Uncertainty | Treatment until clarified |
| --- | --- | --- |
| Q01 | Are name, phone, and address mandatory on update? May an existing phone be cleared? | Do not invent rejection expectations for empty/missing/null fields |
| Q02 | Are surrounding spaces trimmed before phone validation? | Record raw and persisted values; mark normalization oracle pending |
| Q03 | Are Unicode decimal digits accepted? | ASCII is a working assumption; review before classifying Unicode behavior |
| Q04 | What are name/address length bounds and allowed characters? | No invented 50/255-character limits; exploratory results only |
| Q05 | Are no-op updates and partial updates supported? | Use complete baseline payloads first; explore omissions separately |
| Q06 | What status/message is required for invalid phone or forbidden fields? | Check semantic outcome; no mandatory 400/403 or exact text without a source |
| Q07 | Must a rejected multi-field update be atomic? | Check retained state; clarify whether partial updates violate the contract |
| Q08 | Does the UI-only email restriction also forbid backend email changes? | Enforce UI restriction; API policy remains a question |
| Q09 | Which mobile devices and OS versions must be covered? | Record the actual execution environment; do not claim cross-device coverage |

Assume valid sessions and ordinary nonempty name/address baseline values only to isolate defined phone rules. These are test preconditions, not newly imposed business requirements.

## 8. Static observations for later investigation

These are source-review findings, not runtime-confirmed bugs. They must not alter the requirement-based domains above.

| Observation | Source location | Investigation needed |
| --- | --- | --- |
| Mobile phone regex is `^[1-9][0-9]{8,9}$`, permitting 9-10 digits with a nonzero first digit, contrary to FR-04's stated rule | `frontend-mobile/App.js`, `handleUpdateProfile` | Reproduce rejection of a valid value and acceptance of invalid values on mobile |
| Mobile sends `shippingAddress`; documented API and backend use `shipping_address` | Mobile update handler; API section 2.2; backend profile handler | Verify address persistence after a fresh read/re-login, not just local state |
| Backend profile update accepts `role` from the request body | `backend/server.js`, `PUT /api/users/me` | Verify role protection separately on disposable data; retain request/response and post-state evidence |
| Backend profile update handler has no visible phone-format check | Same backend handler | Check invalid phone behavior with API probes separately from mobile validation |
| Email input is declared `editable={false}` | `frontend-mobile/App.js`, `renderProfile` | Verify actual editability on the selected mobile device |

Do not fix the application during testing preparation. Record implementation changes separately if later authorized so test evidence remains tied to a known SUT version.

## 9. Review checklist and next handoff

- [ ] Student verifies source interpretation and course-specific technique.
- [ ] Student reviews Q01-Q09 and records any lecturer clarification with date/source.
- [ ] Mobile setup/device details are recorded.
- [ ] Detailed cases are created from requirements and domains, including ownership and role protection.
- [ ] Mobile and supporting API execution are reported separately.
- [ ] Actual results, evidence, confirmed bugs, and AI omissions are recorded after execution.

Current execution totals for this preparation: 0 feature tests executed; no runtime-confirmed bugs; no screenshots captured. Counts of detailed designed cases are pending the next step.
