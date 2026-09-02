# `turfr` Canonical Match Format

## Contents

1. [Purpose](#1-purpose)
2. [Core Principles](#2-core-principles)
3. [Match](#3-match)
4. [Participants](#4-participants)
   - [4.1 Attendance](#41-attendance)
   - [4.2 Payment](#42-payment)
   - [4.3 Attendance and Payment](#43-attendance-and-payment)
   - [4.4 Payment Amounts](#44-payment-amounts)
   - [4.5 Attribution](#45-attribution)
5. [Relationships](#5-relationships)
   - [5.1 Replacement](#51-replacement)
   - [5.2 SOS](#52-sos)
6. [Collector](#6-collector)
   - [6.1 Participant Collector](#61-participant-collector)
   - [6.2 External Collector](#62-external-collector)
   - [6.3 Collector Identity](#63-collector-identity)
7. [Extras](#7-extras)
8. [Identity Resolution](#8-identity-resolution)
9. [Validation](#9-validation)
   - [9.1 Syntax Validation](#91-syntax-validation)
   - [9.2 Semantic Validation](#92-semantic-validation)
10. [Derived Data](#10-derived-data)
11. [Ingestion](#11-ingestion)
12. [Canonical Example](#12-canonical-example)

---

## 1. Purpose

The canonical match format is the stable intermediate representation between a
human-written WhatsApp match list and the Turfr domain/database.

The format preserves reported match facts without guessing facts that were not
reported.

Pipeline:

WhatsApp representation → Parse → Validate → Identity resolution → Ingest

Identity resolution happens after parsing and before ingestion.

---

## 2. Core Principles

1. A canonical payload represents the final reported state of one match.

2. One payload represents exactly one match.

3. Canonical data records reported facts, not assumptions, fairness policy, or
   derived conclusions.

4. Participant facts are composed from independent dimensions.

5. Attendance and payment are independent facts.

6. Relationships connect people but do not automatically transfer attendance
   or payment facts between them.

7. Names are human-readable identity references, not Player IDs.

8. Identity resolution happens after parsing and before ingestion.

9. The parser must not guess, silently repair, or invent facts that are neither
   explicitly represented nor established by a canonical default.

10. Derived accounting state is calculated from canonical facts and is not
    itself canonical input.

---

## 3. Match

A canonical match records match-level facts and participant-level facts.

Required match facts:

- Date
- Time
- Venue
- Team A
- Team B
- Per-person expected contribution
- Turf payable
- Collector

The required collector may be represented either within a participant expression
using `🤲` or as an external match-level collector using `Collector: NAME`, as
defined in Section 6.

Optional match facts:

- UPI ID
- Note

Team membership is retained as reported for match insights.

Team membership and accounting are separate concerns.

---

## 4. Participants

A participant expression records facts about a person.

A person represented in a participant expression is not necessarily an
accounting participant. For example, an SOS player may appear in a participant
expression without automatically becoming an accounting participant.

A participant may also appear in a relationship expression with another person,
but attendance and payment facts remain attached to the individual participant.

### 4.1 Attendance

Attendance has two states:

- Present
- Absent

Present is the default.

```text
NAME
```

does not explicitly record absence and therefore means the participant was
present.

`⁉️` explicitly records absence.

```text
NAME ⁉️
```

means the participant was absent.

Attendance is independent of payment.

---

### 4.2 Payment

Payment records what amount, if any, was paid by the participant.

Payment facts may be:

- full payment via UPI;
- full payment via cash;
- partial payment via UPI;
- partial payment via cash;
- split payment across one or more methods;
- no payment.

Payment markers:

- `🌳` — full expected contribution paid via UPI.
- `💸` — full expected contribution paid via cash.
- `❌` — no payment was recorded for the participant.

Examples:

```text
NAME 🌳
```

The participant paid their full expected contribution via UPI.

```text
NAME 💸
```

The participant paid their full expected contribution via cash.

```text
NAME ❌
```

No payment was recorded for the participant.

`❌` records a payment fact only.

It does not itself determine whether the participant owes money, should pay,
will settle later, or is subject to any contribution policy.

An accounting participant must have a determinable payment state.

The payment state may be established by:

- an explicit payment marker;
- an explicit payment amount with its payment method; or
- a canonical default defined by this format.

---

### 4.3 Attendance and Payment

Attendance and payment are independent facts.

Therefore:

```text
NAME 🌳
```

means present and paid the full expected contribution via UPI.

```text
NAME 💸
```

means present and paid the full expected contribution via cash.

```text
NAME ❌
```

means present and no payment was recorded.

```text
NAME ⁉️ 🌳
```

means absent and paid the full expected contribution via UPI.

```text
NAME ⁉️ 💸
```

means absent and paid the full expected contribution via cash.

```text
NAME ⁉️ ❌
```

means absent and no payment was recorded.

Absence does not imply any payment state.

Likewise, a payment state does not imply any attendance state.

---

### 4.4 Payment Amounts

An explicit amount records the amount actually paid by the participant.

Examples:

```text
NAME ₹50 🌳
```

The participant paid ₹50 via UPI.

```text
NAME ₹50 💸
```

The participant paid ₹50 via cash.

A participant may make a split payment.

```text
NAME ₹50 💸 + ₹50 🌳
```

The participant paid ₹50 via cash and ₹50 via UPI.

Explicit payment amounts are required whenever a participant's recorded payment
does not equal the full expected contribution.

A full expected contribution may use the shorthand payment markers:

```text
NAME 🌳
NAME 💸
```

When the explicit amount equals the full expected contribution, the shorthand
form should normally be used instead.

`❌` must not be combined with a payment amount or a paid payment marker.

---

### 4.5 Attribution

Payment facts are attributed to the participant expression to which they are
attached.

```text
Abbas ⁉️ ₹50 🌳
```

records:

- Abbas was absent.
- Abbas paid ₹50 via UPI.

The representation does not infer why Abbas paid, whether Abbas should have
paid, or whether another participant should pay any remaining amount.

Those questions belong to derived accounting or contribution policy, not to
the reported canonical fact.

---

## 5. Relationships

Relationship operators connect participant expressions.

A relationship does not transfer attendance or payment facts from one participant
to another.

Each participant expression owns its own reported facts.

### 5.1 Replacement

`🔄` records a replacement relationship between participants.

In a replacement relationship:

```text
LEFT_PARTICIPANT 🔄 RIGHT_PARTICIPANT
```

means the left participant is recorded as replacing the right participant.

Form:

```text
PLAYER_EXPRESSION 🔄 PLAYER_EXPRESSION
```

Every participant expression in a replacement relationship is an accounting
participant.

Each participant may independently have:

- an attendance state;
- a payment state;
- an explicit payment amount.

Example:

```text
Naitik ❌ 🔄 Abbas 🌳
```

Naitik is recorded as replacing Abbas.

Naitik made no recorded payment.

Abbas paid the full expected contribution via UPI.

The relationship itself does not determine why either participant paid or did not
pay.

Another example:

```text
Naitik ₹50 🌳 🔄 Abbas ₹50 💸
```

Naitik is recorded as replacing Abbas.

Naitik paid ₹50 via UPI.

Abbas paid ₹50 via cash.

Replacement does not by itself imply:

- that either participant was absent;
- that either participant was present;
- that either participant paid;
- how the expected contribution should be divided between them.

Those facts must be explicitly represented on the relevant participant
expressions.

A replacement relationship may form a chain.

Form:

```text
PLAYER_EXPRESSION 🔄 PLAYER_EXPRESSION 🔄 PLAYER_EXPRESSION
```

Example:

```text
Naitik ❌ 🔄 Abbas ₹50 🌳 🔄 Swar ❌
```

This records a replacement chain involving Naitik, Abbas, and Swar.

Each participant retains their own independently recorded attendance and payment
facts.

A replacement chain may contain at most three participant expressions.

Longer replacement chains are invalid canonical input.

Unknown replacement participants are invalid.

---

### 5.2 SOS

`🆘` means that the first person played as an SOS player for the participant
expression on the right.

Form:

```text
PLAYER_EXPRESSION 🆘 PLAYER_EXPRESSION
```

The SOS player is not automatically an accounting participant.

The participant for whom the SOS was provided remains an accounting participant.

The SOS relationship does not itself determine:

- attendance;
- payment;
- contribution policy;
- how any payment should be divided.

Example:

```text
Rohit 🆘 Abbas ⁉️ ❌
```

Rohit played as an SOS player for Abbas.

Abbas was absent.

No payment was recorded for Abbas.

The representation does not infer whether Abbas should later pay.

Another example:

```text
Rohit 🆘 Abbas 🌳
```

Rohit played as an SOS player for Abbas.

Abbas paid the full expected contribution via UPI.

The payment is attributed to Abbas.

Rohit does not automatically become the payer merely because Rohit played.

If an SOS player explicitly has a payment fact, that payment belongs to the SOS
player and the SOS player becomes an accounting participant.

For example:

```text
Rohit 🌳 🆘 Abbas ⁉️ ❌
```

Rohit played as an SOS player for Abbas.

Rohit paid the full expected contribution via UPI and is therefore an accounting
participant.

Payment facts on the SOS player and the player receiving SOS remain
independently attributed.

A payment recorded on an SOS player must not automatically be attributed to the
player for whom the SOS was provided.

Abbas was absent and made no recorded payment.

An SOS player may be unknown at the time of recording.

Example:

```text
Unknown 🆘 Abbas ⁉️ ❌
```

This records that an unidentified person played as SOS for Abbas.

The identity of the SOS player remains unresolved.

Unknown replacement participants are invalid, but unknown SOS players are
valid.

---

## 6. Collector

The collector is the person identified as responsible for collecting
contributions for the match.

A collector may be represented:

- within a participant expression; or
- as an external collector at the match level.

### 6.1 Participant Collector

`🤲` identifies a collector who is represented in a participant expression.

Example:

```text
Jay 🤲
```

Jay is the collector, is present by default, and paid the full expected
contribution via UPI.

If the collector is an accounting participant, `🤲` establishes:

- the collector role;
- present attendance by default;
- full payment of the expected contribution via UPI by default.

Explicit facts override defaults.

For a participant collector, precedence is:

```text
explicit participant fact > collector default > general canonical default
```

An explicit absence marker overrides the collector's default present attendance.

An explicit payment fact overrides the collector's default full UPI payment.

`🌳` must not be added merely to repeat the full UPI payment already implied by
`🤲`.

For example:

```text
Jay 🤲 💸
```

Jay is the collector and paid the full expected contribution via cash.

An explicit payment amount overrides the default full UPI payment.

```text
Jay 🤲 ₹50 🌳
```

Jay is the collector and paid ₹50 via UPI.

```text
Jay 🤲 ⁉️
```

Jay is the collector, was absent, and paid the full expected contribution via
UPI.

```text
Jay 🤲 ❌
```

Jay is the collector, was present, and made no recorded payment.

```text
Jay 🤲 ⁉️ ❌
```

Jay is the collector, was absent, and made no recorded payment.

A collector may also appear in a relationship expression.

```text
Jay 🤲 ⁉️ ❌ 🔄 Naitik 🌳
```

Jay is the collector.

Jay was absent.

Jay made no recorded payment.

Naitik replaced Jay and paid the full expected contribution via UPI.

The collector role remains attached to Jay.

A different collector must not be inferred from the replacement relationship.

---

### 6.2 External Collector

If the collector is not represented in any participant expression, the
collector must be declared at the match level.

Form:

```text
Collector: NAME
```

Example:

```text
Collector: Swar
```

Swar is the collector but is not represented as a participant expression.

An external collector does not become an accounting participant merely by
being identified as the collector.

---

### 6.3 Collector Identity

Exactly one collector must be identifiable for a canonical match.

The collector may be identified either:

- by `🤲` in a participant expression; or
- by the match-level `Collector: NAME` field.

The two forms must not identify different collectors in the same canonical
payload.

If both forms are present, they must resolve to the same person.

---

## 7. Extras

Extras are people recorded outside the accounting participant list.

They are retained as reported match observations and do not:

- count toward accounting participants;
- create a contribution payment requirement;
- contribute to collection totals merely by being listed as Extras.

Being listed as an Extra alone creates no attendance, payment, replacement, or
SOS relationship.

A person must not be represented as an Extra and simultaneously as a
participant or relationship participant in the same canonical payload.

Such a representation is invalid and must be rejected.

---

## 8. Identity Resolution

Names in the canonical payload are human-readable identity references, not
database identities.

Identity resolution happens after parsing and before ingestion.

Identity resolution is performed for each person record that requires a
resolved database identity.

The system begins by searching for candidate Players using the name reported in
the canonical payload.

Candidate search may use additional available identity evidence, such as:

- last name;
- phone number.

Candidate discovery may be automatic.

Identity selection must not be automatic when multiple plausible Players exist.

For each person record:

- No matching Player exists → create a new Player entry.
- One or more candidate Players exist → select an existing Player or explicitly
  create a new Player entry.
- When multiple plausible Players exist, the system must not automatically
  choose between them.
- Once selected or created, the resolved Player UUID becomes the database
  identity used for ingestion.

The system must not guess between multiple plausible Players.

For example, a reported name may produce multiple candidates:

```text
Amit
```

Possible candidates:

```text
Amit Sharma
Amit Saini
```

The system must not automatically choose between them.

Additional identity evidence may narrow the candidates.

If the intended person is not represented by any existing candidate, a new
Player entry may be explicitly created.

Every person record requiring a resolved database identity must be resolved
before ingestion.

The canonical format does not require a Player ID.

Unknown SOS players are an exception because their identity is explicitly
unresolved in the canonical representation.

---

## 9. Validation

Syntax validation and semantic validation are different.

### 9.1 Syntax Validation

Syntax validation answers:

> Can this canonical representation be parsed according to the format?

Reject malformed canonical expressions, including:

- malformed participant expressions;
- malformed payment amounts or payment splits;
- malformed replacement or SOS relationships;
- invalid symbol placement or ordering;
- unsupported or invalid symbol combinations;
- incomplete required match structure.

The parser must not silently guess or repair malformed input.

---

### 9.2 Semantic Validation

Semantic validation answers:

> Do the parsed reported facts form a valid canonical state?

Reject representations whose reported facts cannot form a valid canonical
state.

Examples include:

- missing required match facts;
- an accounting participant with no valid payment fact;
- an explicit payment amount combined with an incompatible payment marker;
- `❌` combined with a payment amount or paid payment marker;
- an invalid payment split or payment amount combination;
- an invalid participant or relationship combination;
- an invalid collector representation;
- more than one resolved collector identity;
- a person represented both as an Extra and as a participant or relationship
  participant in the same payload;
- an unknown replacement participant;
- a replacement relationship containing more than three participant
  expressions.

Partial payment is valid when represented explicitly by its actual amount.

Canonical defaults defined by the format may be applied.

The parser must not guess facts that are neither explicitly represented nor
established by a canonical default.

Ambiguous identity selection must be resolved before ingestion.

---

## 10. Derived Data

Derived state is not part of the canonical representation.

Derived data may be calculated from canonical facts and applicable contribution
or accounting policy.

Examples include:

- total collection;
- individual payment ledger;
- unpaid or outstanding amounts, where defined by contribution policy;
- collection surplus or deficit;
- active participant count;
- payment totals by method.

Derived data must not be represented as a reported canonical fact unless that
fact was explicitly reported.

The canonical representation preserves reported facts rather than storing
derived conclusions as if they were reported facts.

---

## 11. Ingestion

One payload represents the final reported state of one match.

Ingestion must be idempotent.

An exact re-submission of an already recorded match must be identified as a
duplicate.

A duplicate must not:

- create a second match;
- silently overwrite the existing match; or
- modify the existing recorded facts.

A same-slot match with different participants must not automatically be treated
as an exact duplicate.

Such payloads must not automatically be merged.

Human resolution may be required to determine whether similar or conflicting
payloads describe:

- distinct matches; or
- conflicting reports about the same match.

Existing recorded truth must not be silently overwritten by a conflicting
incoming payload.

Identity selection must be complete for every person record requiring a
resolved database identity before ingestion is accepted.

---

## 12. Canonical Example

The following is an example of a complete canonical match representation.

```text
*Wednesday, 19th August*
Venue : *CP*
Time : *9-10*
Per Person: 100
Turf Payable: 1000
UPI : jayasankar19922111-2@okhdfcbank

Team A: *Black* ⚫
1. Jay 🤲
2. Sarabjeet 💸
3. Abhilash 🌳
4. Surya 🌳
5. Abbas 🌳
6. Dinesh 🌳

Team B: *White* ⚪
1. Abhishek 🌳
2. Aman 💸
3. Reji 🌳
4. Aansh 🌳
5. Shehzan 🌳
6. Mayur 🌳

*Extra*
1. Arkan
2. Simon
3. Swar

Note:
```

This representation records:

- one match;
- the date, time, and venue;
- the expected per-person contribution;
- the turf payable;
- the UPI destination;
- two teams and their reported team membership;
- Jay as the collector;
- Jay as present and paid the full expected contribution via UPI by the
  collector default;
- full payments via cash and UPI for the other accounting participants;
- three Extras;
- no additional reported note.

Collection totals, payment summaries, outstanding amounts, and other derived
accounting or dashboard state are calculated separately from the reported
canonical facts.
