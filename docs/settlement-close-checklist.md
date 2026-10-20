# Settlement Close Checklist

Period-close checks for a multi-store closed-loop gift card program run as a central pool. Replace every figure with your own policy values.

## Program rules (set once)

- [ ] Liability model agreed and written into store agreements (central pool or seller-keeps-liability)
- [ ] Period length and cut-off time, in each store's local time (**illustrative**: daily close, weekly payment)
- [ ] Fee structure: redemption fee, selling commission, fixed fees (**illustrative**: 2% of redemptions)
- [ ] Refund rule for receipted and non-receipted returns at a different store
- [ ] Payment terms for net payers and net receivers, ideally the same
- [ ] Dispute window length (**illustrative**: 5 business days)
- [ ] Breakage treatment confirmed by finance for each jurisdiction

## Each period close

1. Freeze the period at cut-off. Transactions after it go to the next period.
2. Exclude uncaptured holds and voids made inside the void window.
3. Sum sales, reloads, redemptions, refunds-to-card and adjustments per store.
4. Apply fees and commission.
5. Calculate net position per store.
6. Tie-out: total collected minus total paid equals change in outstanding balances plus fees.
7. Issue statements showing gross lines and the net figure.

## Exceptions

- [ ] Late voids from earlier periods carried as adjustments, not dropped
- [ ] Refunds without `original_txn_ref` reviewed before charging any store
- [ ] Cards redeemed whose activation was never funded flagged for chargeback to the activating store
- [ ] Stores exiting the program: outstanding obligations agreed before final payment

## After payment

- [ ] `settlement_ref` recorded against every store statement
- [ ] Open disputes listed with owner and due date
- [ ] Program float account balance matches outstanding card liability
