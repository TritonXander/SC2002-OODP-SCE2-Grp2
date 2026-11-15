# SC2002 Restaurant Rush - Stage 2: Customer Behaviour

Stage 2 adds **VIPCustomer** and **CriticCustomer** alongside regular customers.
Win/lose criteria, turn sequence and base setting are unchanged from Stage 1.

## Build and run

Run these commands from the repository root with Java 17 or later. Compile one stage at a time; committed .class files are not needed.

```sh
javac --release 17 -d /tmp/restaurant-rush-stage2 "Stage 2 Files/JavaFiles/"*.java
java -cp /tmp/restaurant-rush-stage2 RestaurantGameApp
java -cp /tmp/restaurant-rush-stage2 RestaurantGameApp --cycle
```

Requires Java 17 or later (the UI uses switch expressions).

## Customer types

| Type    | Satisfaction loss per waiting turn | Bill                | Special behaviour |
|---------|------------------------------------|---------------------|-------------------|
| Regular | 8                                  | Full price          | none |
| VIP     | 5                                  | 10% off (rounded half-up to the cent) | none |
| Critic  | 12                                 | Full price          | Reacts to the food when it is served (see below) |

All types follow the same flow: queue -> seated -> order -> cooked -> served -> eating -> ready to pay -> paid.

### Critic reaction (documented special behaviour)

At the moment the Waiter serves the Critic's order, the Critic judges the food with one random roll:

| Chance | Reaction (printed in colour)            | Effect on satisfaction |
|--------|-----------------------------------------|------------------------|
| 50%    | "Compliments to the Chef!" (green)      | Restored to 100, permanently |
| 25%    | "The food is just okay..." (yellow)     | Unchanged: keeps its current value |
| 25%    | "I want to speak to the Chef!" (red)    | Dropped to 0, permanently |

- "Permanently" holds because a served customer no longer loses satisfaction (Step 5 only affects waiting or seated, unserved customers). The new value is the one recorded when the Critic pays.
- A furious Critic (satisfaction 0) does **not** abandon: abandonment applies only to customers still waiting for service. The Critic still pays full price, but a recorded satisfaction of 0 pulls down the average.
- The Critic's roll uses its **own unseeded `Random`** (a field inside `CriticCustomer`), so the reaction differs from run to run. It is independent of the game's seed-42 generator (`CustomerFoodChooser`): menu choices, bills and every other customer type remain fully reproducible. The Critic's verdict is the only random element of the game.

## Arrival schedule (base setting, six arrivals)

| Turn | 1      | 4       | 7   | 10      | 13  | 16      |
|------|--------|---------|-----|---------|-----|---------|
| Type | Critic | Regular | VIP | Regular | VIP | Regular |

The schedule contains exactly one Critic and two VIPs; it is registered in one place (`ARRIVAL_SCHEDULE` in `RestaurantController`).

## How behaviour is dispatched (no type checks in the controller)

`RestaurantController` never asks what type a customer is. It calls methods on `Customer` and each subclass supplies its own behaviour:

| Customer method | Regular | VIP | Critic |
|---|---|---|---|
| `getSatisfactionLossPerTurn()` | 8 | 5 | 12 |
| `calculateBillCents(basePrice)` | full price (default) | 90% of price | full price (default) |
| `reactToServedFood()` | no reaction (default) | no reaction (default) | rolls like / okay / hate |
| `getBehaviourNote()` | text | text | text |

The only places the subclasses are named in the controller are the six constructor references in the arrival schedule.

## Stage 1 -> Stage 2 transition

### New files

| File | Purpose |
|---|---|
| `VIPCustomer.java` | VIP type: loses 5 per turn, overrides `calculateBillCents` for 10% off. |
| `CriticCustomer.java` | Critic type: loses 12 per turn, overrides `reactToServedFood` with the 50/25/25 reaction. |
| `CustomerReaction.java` | Enum for a customer's reaction (`NONE`, `DELIGHTED`, `UNIMPRESSED`, `FURIOUS`) with its spoken line, so the UI can print it without knowing any rule. |

### Existing files changed

| File | Change and reason |
|---|---|
| `Customer.java` | Added three overridable hooks: `calculateBillCents` (default full price), `reactToServedFood()` (default `NONE`; randomness is owned by the subclass) and abstract `getBehaviourNote`. Reason: lets subclasses carry their own billing and reactions so the controller needs no type checks (OCP). |
| `RegularCustomer.java` | Implements `getBehaviourNote()`. Reason: required by the new abstract method; behaviour is otherwise identical to Stage 1. |
| `Order.java` | Added `billCents`, computed once at order creation via `customer.calculateBillCents(...)`, plus `getBillCents()`. Reason: the discount is applied when the order is taken, so the price is locked and the bill is available to the UI and the Cashier step. |
| `ActionResult.java` | Added an optional `CustomerReaction` (default `NONE`) with an `ok(message, reaction)` overload. Reason: carries the Critic's reaction from the controller to the UI without the UI inspecting customers. Existing `ok`/`fail` calls are unchanged. |
| `RestaurantController.java` | (1) `ARRIVAL_TURNS` set replaced by `ARRIVAL_SCHEDULE`, a map from turn to customer constructor; arrivals no longer hard-code `RegularCustomer`. (2) `waiterServeOrder` calls `customer.reactToServedFood()` and returns the reaction. (3) Payment and the order message use `order.getBillCents()` instead of the menu price. (4) Added `getCustomerTypeLegend()` for the UI banner, built from the customer objects. Turn sequence, abandonment, payment order and victory logic are unchanged. |
| `ConsoleUI.java` | Banner says Stage 2 and lists the customer types; queue, table, order and task lines show each customer's type and real loss rate (previously a hard-coded "loses 8/turn"); orders show the bill with the list price when discounted; the arrival line shows the behaviour note; the Critic's reaction is printed in colour after serving. Display only: no game rules added. |

### Files unchanged

`Cashier.java`, `Chef.java`, `CustomerFoodChooser.java`, `CustomerStatus.java`, `MenuItem.java`, `OrderStatus.java`, `RestaurantGameApp.java`, `Table.java`, `Waiter.java`.

## Test notes (base setting, seed 42)

Menu choices use seed 42, so orders, bills and every customer other than the Critic are identical on every run. Only the Critic's reaction varies (see above), which changes Customer #1's recorded satisfaction and therefore the average.

### Reproducible victory run

Customers: #1 Critic (T1), #2 Regular (T4), #3 VIP (T7), #4 Regular (T10), #5 VIP (T13), #6 Regular (T16).
Menu choices come from the default seeded randomness. Customer #1 (Critic) is served at satisfaction 64. Enter these Waiter / Chef choices:

| Turn | Waiter | Chef |
|---|---|---|
| 1 | Seat Customer #1 at Table 1 | Wait |
| 2 | Take order at Table 1 | Cook Order #1 |
| 3 | Wait | Cook Order #1 |
| 4 | Serve Order #1 | Wait |
| 5 | Seat Customer #2 at Table 2 | Wait |
| 6 | Take order at Table 2 | Cook Order #2 |
| 7 | Serve Order #2 | Wait |
| 8 | Seat Customer #3 at Table 1 | Wait |
| 9 | Take order at Table 1 | Cook Order #3 |
| 10 | Serve Order #3 | Wait |
| 11 | Wait | Wait |
| 12 | Seat Customer #4 at Table 2 | Wait |
| 13 | Seat Customer #5 at Table 1 | Wait |
| 14 | Take order at Table 1 | Cook Order #4 |
| 15 | Take order at Table 2 | Cook Order #4 |
| 16 | Serve Order #4 | Cook Order #5 |
| 17 | Serve Order #5 | Wait |
| 18 | Wait | Wait |

Result: **Victory whichever way the Critic reacts** - 4 paid customers (#1, #2, #3, #5), revenue **$51.30** ($18.00 + $9.00 + $8.10 + $16.20), 0 unhappy departures. Only the average paid satisfaction depends on the Critic's roll:

| Critic reaction | Customer #1 recorded satisfaction | Average paid satisfaction |
|---|---|---|
| "Compliments to the Chef!" | 100 | 86.50 |
| "The food is just okay..." | 64 | 77.50 |
| "I want to speak to the Chef!" | 0 | 61.50 (still >= 60) |

This run includes both VIP discounts (Customer #3 pays $8.10, Customer #5 pays $16.20). Replaying these inputs 150 times produced all three reactions and a victory every time, with identical orders and payments.

### Reproducible defeat run

Choose Wait for both the Waiter (option 4) and the Chef (option 2) on all 18 turns.
Result: **Defeat** - 0 paid customers, $0.00 revenue, average N/A, 2 unhappy departures.
