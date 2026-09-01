# Turfr Canonical Match Format

## 1. Purpose

The canonical match format is the stable intermediate representation between a
human-written WhatsApp match list and the Turfr domain/database.

The format preserves what the human reported without guessing facts that were
not reported.

Pipeline:

WhatsApp representation → Parse → Validate → Identity resolution → Ingest

Identity resolution happens after parsing and before ingestion.

---

## 2. Core Principles

1. A canonical payload represents the final resolved state of a match,
   produced after the match and after the contribution has been collected.

2. One payload represents exactly one match.

3. A match is a unique event whose participants and associated records
   represent the truth of what occurred.

4. Canonical data represents reported facts, not derived state.
   The canonical record preserves the reported truth; derived state exposes
   what follows from that truth.

5. Names are human-readable identity references, not Player IDs.

6. Identity resolution happens after parsing and before ingestion.

7. The parser must not guess when the representation is ambiguous or incomplete.

8. Don't encode a fact twice when one symbol already establishes it.

---

## 3. Match

Required match facts:

- Date
- Time
- Venue
- Team A
- Team B
- Per-person expected contribution
- Turf payable
- Collector

Optional:

- UPI ID
- Note

Team membership is retained as reported for match insights; accounting is
independent of team membership.

---

## 4. Participants

A participant entry describes facts about a person and may also describe a
relationship between two people.

### 4.1 Basic participant facts

Normal forms:

```text
NAME 🌳
NAME 💸
NAME ⁉️
NAME 🌳 ⁉️
NAME 💸 ⁉️
```

Meaning:

- `🌳` — paid via UPI.
- `💸` — paid via cash.
- `⁉️` — absent.
- No attendance marker — present by default.
- `NAME ⁉️` means absent and no payment is recorded.
- `NAME 🌳 ⁉️` means absent but paid via UPI.
- `NAME 💸 ⁉️` means absent but paid via cash.

Attendance and payment are independent facts.

Absence does not clear financial liability.

### 4.2 Payment amounts

Explicit payment amounts are used only when they add information beyond the
payment marker.

Normal full payment:

```text
NAME 🌳
NAME 💸
```

Split payment:

```text
NAME ₹50 💸 + ₹50 🌳
```

Split components must represent the player's full expected contribution.

A partial amount by itself does not represent a valid resolved payment.

For an expected contribution of ₹100:

```text
NAME ₹50 💸
```

is not a resolved full payment.

Do not use an explicit full amount merely to repeat the expected contribution
when the payment marker already expresses the same fact.

### 4.3 Attribution

Financial facts are attributed to the player whose name they are attached to.

Attendance, financial liability, and payment are independent facts.

A payment clears the liability of the player to whom that payment is attributed.

If a payment is made on behalf of another player, it remains attributable to
the liable player; the arrangement may be recorded in Note.

Examples:

```text
Abbas ⁉️
```

Abbas is absent and unpaid; Abbas remains financially liable.

```text
Abbas 🌳 ⁉️
```

Abbas is absent but has paid; Abbas has no outstanding contribution.

---

## 5. Relationships

Relationship operators connect two player expressions.

The relationship itself does not move payment or attendance facts from one
person to another.

### 5.1 Replacement

`🔄` means replacement.

Form:

```text
PLAYER_EXPRESSION 🔄 PLAYER_EXPRESSION
```

Each side owns its own facts.

```text
Naitik ₹50 💸 + ₹50 🌳 🔄 Abbas
```

Naitik replaced Abbas.

Naitik paid ₹100: ₹50 cash + ₹50 UPI.

```text
Naitik 🔄 Abbas ₹50 💸 + ₹50 🌳
```

Naitik replaced Abbas.

Abbas paid ₹100: ₹50 cash + ₹50 UPI.

```text
Naitik ₹50 🌳 🔄 Abbas ₹50 💸
```

Naitik replaced Abbas.

Naitik paid ₹50 via UPI and Abbas paid ₹50 via cash.

`🔄` does not by itself determine who paid, who was absent, who was present,
or who is financially liable. Those facts belong to the relevant expression.

### 5.2 SOS

`🆘` means the first player came as an SOS player for the second player.

Form:

```text
PLAYER 🆘 PLAYER_EXPRESSION
```

The SOS player is not an accounting participant.

The player for whom the SOS was provided remains the accounting participant
and remains financially responsible unless the representation explicitly
records otherwise.

```text
Rohit 🆘 Abbas ⁉️
```

Rohit played as SOS for Abbas.

Abbas is absent and unpaid; Abbas remains financially liable.

Rohit does not become financially liable merely by playing.

```text
Rohit 🆘 Abbas 🌳
```

Rohit played as SOS for Abbas.

Abbas's contribution was paid via UPI.

Rohit does not become the payer merely by playing as SOS.

An SOS player may be unknown at the time of recording.

---

## 6. Collector

`🤲` identifies the collector.

The collector:

- may be a participant or an external person;
- does not need a payment marker;
- does not become a payer merely by collecting;
- does not need to be physically present at the venue.

If the collector is a participant, `🤲` alone establishes the collector role.
Adding `🌳` or `💸` is appropriate only when that person also has an
independent payment fact.

If the collector is external, the canonical representation must explicitly
identify that external collector.

---

## 7. Extras

Extras are people recorded outside the accounting participant list.

They are retained for future insights and do not:

- count toward accounting participants;
- create contribution liability;
- contribute to collection totals.

An Extra may later appear as a player, SOS player, or replacement participant
in a final representation, but being listed as an Extra alone creates no
accounting relationship.

---

## 8. Identity Resolution

Names in the canonical payload are human-readable identity references, not
database identities.

Identity resolution happens after parsing and before ingestion.

Resolution uses available evidence in this order:

1. First name
2. Last name
3. Phone number
4. Human confirmation

The system must not guess between multiple possible Players.

Rules:

- No matching Player exists → create a Player entry.
- Exactly one unambiguous Player exists → resolve to that Player.
- Multiple possible Players exist → report a conflict and require human
  resolution.
- The resolved Player UUID is the database identity stored by the system.

Last name and phone number are resolution evidence. They do not need to appear
in every canonical payload.

The canonical format does not require a Player ID.

---

## 9. Validation

Syntax errors and semantic validation errors are different.

### Syntax validation

Reject malformed participant or relationship expressions, including malformed
payment splits, malformed relationships, invalid symbol placement, or
incomplete required structure.

### Semantic validation

Reject or flag representations whose facts cannot form a valid final state.

Examples:

- missing required match facts;
- missing payment state for an accounting participant;
- partial payment represented as complete;
- invalid participant combinations;
- payment split that does not reconcile to the expected contribution.

The parser must not silently guess or repair ambiguous input.

---

## 10. Derived Data

Derived state is not part of the canonical representation.

Examples:

- total collection;
- individual payment ledger;
- outstanding amounts;
- collection surplus/deficit;
- active participant count;
- payment totals by method.

These are calculated from canonical facts.

The canonical representation preserves reported facts rather than storing
derived conclusions as if they were reported facts.

---

## 11. Ingestion

One payload represents one final resolved match.

Ingestion must be idempotent.

- Exact re-submission must not create a second match or silently overwrite the
  existing record.
- A same-slot match with different participants must not automatically merge
  with an existing match.
- Existing recorded truth must not be silently overwritten by a conflicting
  incoming payload.

Identity resolution must be complete for every accounting participant before
ingestion is accepted.

---

## 12. Canonical Example

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

This records reported match facts. Collection totals, outstanding amounts,
and other dashboard/accounting state are derived later.
