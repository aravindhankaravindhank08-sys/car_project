# FEATURES.md — Community Carpooling & Ride-Sharing Platform

Per `../GROUNDRULES.md` §13. Drafted and self-reviewed against the suite's calibration target
(4–6 core modules, 4–6 CRUD entities, comparable screen count to sibling projects such as
`fest-event-organizer` and `campusconnect`) — proceeding straight to build per this project's
brief ("simple and clear scope", no approval pause needed).

## 1. One-line pitch

A ride-matching platform where drivers post point-to-point rides with seats and a per-seat
price, riders search by origin/destination/date and book one or more seats at race-safe
capacity, and both sides rate each other once a ride is completed.

## 2. Core modules

1. **Auth & role-based access** — single login form, three personas (Super Admin, Driver,
   Rider), driver/rider self-registration, server-side role enforcement everywhere.
2. **Vehicle management** — drivers register the vehicle(s) they drive; Admin can deactivate a
   vehicle (e.g. reported as unsafe) which immediately blocks it from being used to post rides.
3. **Ride posting & search matching** — drivers post a ride (origin, destination, date/time,
   seats, price/seat); riders search/filter by origin, destination and date. Matching is plain
   text (case-insensitive substring), deliberately not a maps/geocoding integration.
4. **Seat booking & fare calculation** — riders book 1+ seats on a ride; capacity enforcement is
   race-safe (the project's technical centerpiece, see §3 below); fare is computed server-side
   as `seats_booked × price_per_seat`, shown as a per-seat line plus a running total.
5. **Ride lifecycle & trip management** — scheduled → ongoing → completed/cancelled, driver-
   controlled; a driver's own ride roster shows every rider booked on it; Admin has full
   platform-wide oversight of rides and bookings.
6. **Post-ride ratings & issue reporting** — once a ride is completed, the driver and every
   rider who had a confirmed booking on it can rate each other (1–5 stars + optional comment,
   one rating per direction per ride); either side can also file a report against a ride/user,
   which Admin resolves or dismisses from a moderation queue.

## 3. Technical centerpiece: race-safe seat booking

Same "row lock as mutex" pattern already used elsewhere in this suite (project 3's slot
reservation, project 16's seat holds, project 19's registration capacity check): booking a ride
opens a transaction, runs `SELECT ... FOR UPDATE` on the `rides` row to serialize every
concurrent booking attempt against that ride, re-checks `seats_available >= seats_requested`
while holding the lock, inserts the `bookings` row and decrements `rides.seats_available` in the
same transaction, then commits. No request ever trusts a client-submitted "seats remaining"
number. See `README.md` for the concurrency smoke-test result.

## 4. Feature breakdown by persona

| Feature | Super Admin | Driver | Rider |
|---|---|---|---|
| Register / login / logout | seeded only (no self-register) | self-register | self-register |
| Manage own vehicle(s) | view/deactivate any vehicle | add / edit / deactivate own | — |
| Post / edit / cancel a ride | view/cancel any ride | create, edit (pre-departure), cancel own | — |
| Search & browse rides | view all rides | view own rides list | search by origin/destination/date |
| Book seats on a ride | — | — | book N seats (race-safe), see fare breakdown |
| Cancel own booking | — | — | cancel own upcoming (pre-departure) booking |
| Mark ride ongoing / completed | — | update own ride's status | — |
| View a ride's rider roster | view any ride's roster | view own ride's roster | — |
| Rate the other side post-ride | view all ratings | rate each rider on a completed ride | rate driver on a completed ride |
| File / resolve reports | resolve or dismiss any report | file a report | file a report |
| User account management | activate / deactivate any user | — | — |
| Platform dashboard | platform-wide stats | own rides/bookings/earnings summary | own bookings/spend summary |

## 5. Screen list per persona

**Super Admin** (7): Dashboard · Users · Vehicles (oversight) · Rides (oversight) · Bookings
(oversight) · Reports (moderation queue) · Ratings (oversight)

**Driver** (8): Dashboard · My Vehicles · Add/Edit Vehicle (form) · My Rides · Post/Edit Ride
(form) · Ride Roster & mark status (per ride) · Rate Riders (per completed ride) · Profile
(own ratings received)

**Rider** (7): Dashboard · Search Rides · Ride Details & Book · My Bookings · Rate Driver (per
completed ride) · Profile (own ratings received) · Report an issue

Comparable in count to `fest-event-organizer` (6 admin / 8 organizer / 10 participant screens)
and `campusconnect` (7 admin / 6 coordinator / 5 student).

## 6. Out of scope (and why)

- **Real maps/geocoding/turn-by-turn routing** — no API keys available in this environment;
  origin/destination stay plain text locality names per the project brief.
- **Live GPS tracking of an in-progress ride** — no maps API to plot it against; project 9
  (Vehicle Rental) already covers a simulated-GPS angle for this suite.
- **In-app chat/messaging between driver and rider** — project 17 (Skill Exchange) and project
  25 (RelayChat) already cover real-time messaging in this suite; out of scope here to keep
  this project's own surface focused on the matching/booking/rating loop.
- **Online payment gateway / in-app wallet** — fare is calculated and displayed for reference
  (cash/UPI settled outside the app, as is realistic for informal carpooling); no payment
  processor integration.
- **Fare-splitting across multiple riders' separate payment methods** — kept to straightforward
  per-seat pricing (`seats_booked × price_per_seat`) as the brief specifies, not a multi-payer
  settlement engine.
- **Recurring/repeating ride schedules** (e.g. "every weekday 9am") — each ride is posted as a
  single one-off trip; considered and dropped as unnecessary complexity for the core matching
  demo.
- **Push/SMS notifications** — in-app flash messages only, consistent with the rest of the
  suite's local-demo scope.
- **Dynamic/surge pricing** — price per seat is fixed by the driver at posting time.
- **Driver document/license verification workflow** — Admin can deactivate a vehicle or user
  account, which is the realistic level of "trust & safety" control for a student project;
  a full KYC document-upload/approval pipeline was considered and dropped as out of proportion
  to this project's scope.
