# AD361 – Learning Practice 03: Black-Box Testing, Part 1

## North Vacation Booking

### Objective
Design black-box test cases using equivalence partitioning based on the reservation requirements. No application code is provided, so these are **designed test cases**, not executed tests.

### Requirements Summary
- A booking is 1–31 nights, inclusive, with 1–12 guests.
- Nightly base rate: $200. Cleaning fee: $50 once per valid reservation.
- Each guest above five adds $25 per night.
- Check-in must precede checkout; nights = checkout minus check-in, excluding checkout.
- Every reserved night must be available.
- Invalid inputs raise `ValueError`; unavailable nights raise `UnavailableError`.

**Pricing formula for valid, available bookings:** `total = nights * (200 + 25 * max(guests - 5, 0)) + 50`.

### Test Assumptions
Unless a test states otherwise, check-in is 2026-11-01, all requested nights are available, and there are 3 guests. For night-count tests, checkout is check-in plus the specified number of nights. For guest-count tests, bookings last 2 nights. The requirements do not specify behavior for non-integer guest counts, invalid date formats, or overlapping invalid and unavailable conditions, so those cases are not assigned invented outcomes.

## Step 1–2: Equivalence Partition Tables

### Number of Nights
| Partition | Range / Condition | Representative | Expected Result |
|---|---|---|---|
| N1 – Invalid low | 0 or fewer nights | 0 | `ValueError` |
| N2 – Valid | 1–31 nights | 7 | Accepted when available; price according to formula |
| N3 – Invalid high | More than 31 nights | 32 | `ValueError` |

### Number of Guests
| Partition | Range / Condition | Representative | Expected Result |
|---|---|---|---|
| G1 – Invalid low | Fewer than 1 guest | 0 | `ValueError` |
| G2 – Valid standard rate | 1–5 guests | 3 | No additional guest fee |
| G3 – Valid extra-guest rate | 6–12 guests | 7 | $25 per guest above five, per night |
| G4 – Invalid high | More than 12 guests | 13 | `ValueError` |

### Check-In and Checkout Dates
| Partition | Condition | Representative | Expected Result |
|---|---|---|---|
| D1 – Valid chronological order | Check-in precedes checkout, with 1–31 nights | Nov 1 → Nov 3 | Accepted if available; 2 nights charged |
| D2 – Invalid equal dates | Check-in equals checkout | Nov 1 → Nov 1 | `ValueError` |
| D3 – Invalid reversed dates | Check-in after checkout | Nov 3 → Nov 1 | `ValueError` |

### Availability
| Partition | Condition | Representative | Expected Result |
|---|---|---|---|
| A1 – Fully available | All reserved nights available | Nov 1 and Nov 2 available | Accepted |
| A2 – Partially unavailable | At least one reserved night unavailable | Nov 2 unavailable | `UnavailableError` |
| A3 – Fully unavailable | No reserved nights available | Nov 1 and Nov 2 unavailable | `UnavailableError` |

## Step 3: Test Cases

| ID | Classification | Partition | Input | Expected Result |
|---|---|---|---|---|
| TC01 | Nights | N1 – Invalid low | Nov 1 → Nov 1 (0 nights), 3 guests | `ValueError` |
| TC02 | Nights | N2 – Valid | Nov 1 → Nov 8 (7 nights), 3 guests, all available | Accepted; `7 × $200 + $50 = $1,450` |
| TC03 | Nights | N3 – Invalid high | Nov 1 → Dec 3 (32 nights), 3 guests | `ValueError` |
| TC04 | Guests | G1 – Invalid low | Nov 1 → Nov 3, 0 guests | `ValueError` |
| TC05 | Guests | G2 – Valid standard | Nov 1 → Nov 3, 3 guests, all available | Accepted; `2 × $200 + $50 = $450` |
| TC06 | Guests | G3 – Valid extra-guest | Nov 1 → Nov 3, 7 guests, all available | Accepted; `2 × ($200 + 2 × $25) + $50 = $550` |
| TC07 | Guests | G4 – Invalid high | Nov 1 → Nov 3, 13 guests | `ValueError` |
| TC08 | Dates | D1 – Valid order | Nov 1 → Nov 3, 3 guests, all available | Accepted; checkout excluded; 2 nights, $450 |
| TC09 | Dates | D2 – Equal dates | Nov 1 → Nov 1, 3 guests | `ValueError` |
| TC10 | Dates | D3 – Reversed dates | Nov 3 → Nov 1, 3 guests | `ValueError` |
| TC11 | Availability | A1 – Fully available | Nov 1 → Nov 3, 3 guests; Nov 1 and Nov 2 available | Accepted; $450 |
| TC12 | Availability | A2 – Partially unavailable | Nov 1 → Nov 3, 3 guests; Nov 1 available, Nov 2 unavailable | `UnavailableError` |
| TC13 | Availability | A3 – Fully unavailable | Nov 1 → Nov 3, 3 guests; neither Nov 1 nor Nov 2 available | `UnavailableError` |

### Additional Boundary Tests (Recommended)

| ID | Classification | Input | Expected Result |
|---|---|---|---|
| TC14 | Nights minimum | Nov 1 → Nov 2 (1 night), 3 guests, available | Accepted; $250 |
| TC15 | Nights maximum | Nov 1 → Dec 2 (31 nights), 3 guests, available | Accepted; $6,250 |
| TC16 | Guests minimum | Nov 1 → Nov 3, 1 guest, available | Accepted; $450 |
| TC17 | Guests fee threshold | Nov 1 → Nov 3, 5 guests, available | Accepted; $450 |
| TC18 | Guests first extra | Nov 1 → Nov 3, 6 guests, available | Accepted; $500 |
| TC19 | Guests maximum | Nov 1 → Nov 3, 12 guests, available | Accepted; $800 |
| TC20 | Checkout excluded from availability | Nov 1 → Nov 3, 3 guests; Nov 1 and Nov 2 available, Nov 3 unavailable | Accepted; checkout is not a reserved night; $450 |

## Why Equivalence Partitioning Helps
Instead of testing every possible reservation, I grouped inputs that should behave the same way. I picked representative values from each group and added boundary cases to catch common mistakes. This reduces unnecessary tests while still checking valid bookings, pricing rules, date handling, and error cases.

## Execution Status
These tests are **designed only**. No booking application code or runnable test interface was supplied, so no pass/fail results are claimed.

## Reference
GeeksforGeeks. (2026, July 3). *Equivalence partitioning method*. https://www.geeksforgeeks.org/software-engineering/equivalence-partitioning-method/
