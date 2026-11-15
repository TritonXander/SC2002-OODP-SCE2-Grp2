# SC2002-OODP-SCE2-Grp2
SC2002 Restaurant Rush Group Assignment

## Verified stage snapshots and reproducible runs

| Stage | Historical commit | Source folder | Test notes |
|---|---|---|---|
| [stage-1](https://github.com/Jiarot/SC2002-OODP-SCE2-Grp2/tree/stage-1) | `a43030803ae9` | `Stage 1 Files/JavaFiles` | [Stage 1 victory and defeat](Stage%201%20Files/Test%20Notes.md) |
| [stage-2](https://github.com/Jiarot/SC2002-OODP-SCE2-Grp2/tree/stage-2) | `fd439e4764bb` | `Stage 2 Files/JavaFiles` | [Stage 2 README and test notes](Stage%202%20Files/Stage%202%20README.md) |
| [stage-3](https://github.com/Jiarot/SC2002-OODP-SCE2-Grp2/tree/stage-3) | `7aefd8e8fa7e` | `Stage 3 Files/Java Files` | [Stage 3 victory and defeat](Stage%203%20Files/Test%20Notes.md) |

The tags identify existing historical commits and were added retrospectively. Stage 1 contains no Stage 2/3 Java features; Stage 2 contains no Stage 3 Java features. All three source versions compile independently with Java 17 compatibility. The Stage 1 and Stage 3 test notes and replay files were added later on `main`; they are not present inside the historical tags. Download or retain the replay files from `main` when testing a tagged checkout.

Stage comparisons: [Stage 1 → 2](https://github.com/Jiarot/SC2002-OODP-SCE2-Grp2/compare/stage-1...stage-2) · [Stage 2 → 3](https://github.com/Jiarot/SC2002-OODP-SCE2-Grp2/compare/stage-2...stage-3).

Build/run commands and exact replay inputs are in the linked test notes. Compile only one stage at a time because the stages reuse class names.

## Stage 2 → Stage 3 transition

The source comparison adds five classes, changes eight existing classes functionally and retains ten classes without gameplay changes. No Java classes are deleted or renamed. These counts compare corresponding class filenames across the separate stage folders; Git history adds the Stage 3 folder as a saved version.

### New files

| File | Responsibility and reason |
|---|---|
| `StaffMember.java` | Centralises the one-task availability rule used by Host, Waiter and Chef. Cashier remains separate. |
| `Host.java` | Adds one seating task before the Waiter, using shared seating rules. |
| `Dish.java` | Defines food name, price and preparation so orders can hold single items or combos. |
| `ComboMeal.java` | Combines two existing menu items, sums preparation and charges 90% of the combined price rounded to cents. |
| `HappyHour.java` | Holds once-per-game activation, two-turn duration and 20% discount policy. |

### Existing files changed

| File | Change and reason |
|---|---|
| `Waiter.java` | Extends `StaffMember` instead of duplicating availability state. |
| `Chef.java` | Extends `StaffMember` for the same one-task rule. |
| `MenuItem.java` | Implements `Dish`, preserving the existing single-item data. |
| `Order.java` | Stores `Dish` instead of `MenuItem` and receives the locked bill from the controller. This is the one-time food-type refactor needed for combos and policy pricing. |
| `Customer.java` | Adds eligible satisfaction recovery for Happy Hour; it was already abstract in Stage 1. |
| `CustomerFoodChooser.java` | Adds the documented combo rule at satisfaction 95 or above while retaining the original main-dish selection rule. |
| `RestaurantController.java` | Owns Host/Happy Hour, shares seating through `seatCustomer(StaffMember,...)`, applies customer discount before Happy Hour and invokes policy recovery. |
| `ConsoleUI.java` | Adds Host and Happy Hour choices/status while retaining the established turn pipeline. |

### Files without gameplay changes

`ActionResult.java`, `Cashier.java`, `CustomerReaction.java`, `CustomerStatus.java`, `OrderStatus.java`, `Table.java`, `RegularCustomer.java`, `VIPCustomer.java`, `CriticCustomer.java` and `RestaurantGameApp.java` retain their behavior. Some copies differ only in line endings or comments; these are not additional functional modifications.

The waiting/abandonment and victory method bodies remain unchanged across all three stages. Stage 2 → 3 also preserves the eating/payment method and all three concrete customer subclasses. Preparation and payment continue through `Order` and `Table`; Chef/Cashier do not acquire concrete combo type-check chains. Each historical candidate and current stage folder independently compiles with Java 17 compatibility. Reproducible victory/defeat strategies and recorded metrics remain in the linked test notes; compilation is not a substitute for their gameplay checks.

[Historical source comparison](https://github.com/Jiarot/SC2002-OODP-SCE2-Grp2/compare/fd439e4764bb3106bf445d7d6d610fd070dfcece...7aefd8e8fa7e8c296f1ceeddb6d6ab3ec88abc23). Published stage tags identify the historical commits listed above.

## Presentation video and timestamps

**Video:** `Group 2 Video Demonstration SC2002 - Captioned.mp4` (included with the group submission)  
**Duration:** 14 minutes 41 seconds  
**Group members:** Ernest, Triton, Jackson, Calvin and Sze Waye  
**Repository:** [SC2002-OODP-SCE2-Grp2](https://github.com/TritonXander/SC2002-OODP-SCE2-Grp2)

All timestamps use `MM:SS`, measured from the start of the video. They mark the start of the explanation; the corresponding screen may change shortly afterwards. The original and captioned videos use the same timeline.

## Main sections

| Time range | Presenter | Section | Content |
|---|---|---|---|
| 00:00 - 02:49 | Ernest | Game overview and the three stages | Base game, turn sequence, customer types, Stage 3 features and the overall design approach. |
| 02:49 - 08:23 | Triton, Jackson and Calvin | Gameplay demonstration and outcomes | Staff task limits, combo preparation, serving, Critic behaviour, payment, Happy Hour, victory and defeat. |
| 08:23 - 10:48 | Calvin | Class diagrams and stage changes | How the class design develops from Stage 1 to Stage 3 through inheritance, polymorphism, interfaces and composition. |
| 10:48 - 12:53 | Sze Waye | Stage 3 sequence diagram | The Critic's order lifecycle, including arrival, seating, ordering, preparation, serving and payment. |
| 12:53 - 14:41 | Ernest, Triton, Jackson, Calvin and Sze Waye | Individual contributions and learning | Each member explains their contribution and key learning. Spoken content ends at approximately 14:33. |

## Detailed timestamps

### Game overview

| Start | Presenter | Topic |
|---|---|---|
| 00:00 | Ernest | Introduction to Group 2's Restaurant Rush project. |
| 00:16 | Ernest | Stage 1: baseline game, staff roles, customer arrivals and victory conditions. |
| 00:43 | Ernest | The five steps in each turn: arrivals, Waiter action, Chef action, payment and waiting penalties. |
| 01:04 | Ernest | Stage 2: VIP and Critic behaviour, customer subclasses and polymorphism. |
| 01:55 | Ernest | Stage 3: Host, combo meals, Happy Hour and the supporting abstractions. |

### Gameplay demonstration

| Start | Presenter | Topic |
|---|---|---|
| 02:49 | Triton | Launching Stage 3, the manager's role, staff task limits and seating the Critic. |
| 04:10 | Jackson | Following a pasta-and-salad combo through preparation, serving, the Critic's reaction and payment. |
| 06:18 | Calvin | Stage 3 features, combo eligibility, Happy Hour satisfaction recovery and discounts on new orders. |
| 07:21 | Calvin | A separate successful 18-turn run: four paid customers, $54.54 revenue and average satisfaction above 60. |
| 07:57 | Triton | A separate defeat run: staff wait, customers abandon the restaurant, and no payments are collected. |

### Class diagrams and design evolution

| Start | Presenter | Topic |
|---|---|---|
| 08:23 | Calvin | Stage 1: core classes, controller coordination and separation of responsibilities. |
| 08:53 | Calvin | Stage 2: extending `Customer` with VIP and Critic subclasses; overriding billing and food-reaction methods. |
| 09:30 | Calvin | Stage 3: `StaffMember`, the `Dish` interface, `ComboMeal` and `HappyHour`. |
| 10:23 | Calvin | Comparing the stages: code reuse, extensibility and changes that preserve the order lifecycle. |

### Sequence diagram: Stage 3 Critic order lifecycle

| Start | Presenter | Topic |
|---|---|---|
| 10:48 | Sze Waye | Use case and starting conditions: Critic #1, reproducible menu choices using `--cycle`, and the optional Happy Hour event. |
| 11:07 | Sze Waye | `ConsoleUI` and `RestaurantController` interactions, staff reset and customer arrival. |
| 11:27 | Sze Waye | Host seating and Waiter ordering within the same turn. |
| 11:37 | Sze Waye | Food selection, combo eligibility and bill locking. |
| 11:58 | Sze Waye | Chef preparation and why an order cannot be served in the turn it becomes ready. |
| 12:09 | Sze Waye | Serving and dynamic dispatch to `CriticCustomer.reactToServedFood()`. |
| 12:31 | Sze Waye | Eating, readiness to pay and the Cashier's payment timing rule. |
| 12:43 | Sze Waye | Payment completes the use case: revenue and satisfaction are recorded, and the table is freed. |

## Individual contributions and key learning

| Time range | Presenter | Contribution and learning discussed |
|---|---|---|
| 12:53 - 13:13 | Ernest | Stage design, comparing implementation approaches, checking that each stage runs independently and documenting design evolution. |
| 13:13 - 13:30 | Triton | Gameplay mechanics, staff constraints, defeat scenarios, resource management and systematic testing. |
| 13:30 - 13:52 | Jackson | Order-processing lifecycle, customer behaviour, victory conditions and polymorphism. |
| 13:52 - 14:11 | Calvin | Stage 3 features, class-design evolution, abstraction, inheritance and interfaces. |
| 14:11 - 14:33 | Sze Waye | Stage 1 and Stage 2 implementation, class and sequence diagrams, and how early design decisions affect later extensions. |
