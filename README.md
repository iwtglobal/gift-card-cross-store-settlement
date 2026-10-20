# Gift Card Cross-Store Settlement

How a **gift card system** settles money between stores when a card is sold in one place and spent in another. Common in franchise networks, shopping-mall gift cards and retail groups with several legal entities. Covers who holds the liability, how net positions are worked out each period, refunds that land in a different store, and fees and breakage. Written for finance and program teams evaluating [EVD System](https://evdsystem.com/) by [MoboGage](https://evdsystem.com/about-mobogage/).

---

## What cross-store settlement is

**Cross-store settlement** is the periodic calculation and payment that evens out a closed-loop gift card program when the store that sold a card is not the store that redeemed it. The selling store took the customer's money. The redeeming store handed over goods. Until settlement runs, one is holding cash that belongs to the other.

In a single company with one bank account this is an internal ledger entry. It becomes real work when the stores are separate businesses: franchisees, mall tenants, or group companies that file their own accounts.

## Two ways to hold the liability

**Central pool.** The program manager (the franchisor, the mall operator or a group treasury entity) owns the gift card liability. Every store passes sales money to the program and claims redemption value back from it. Outstanding card balances sit with the program. This is the more common design because no store needs to know where a card was sold.

**Seller keeps the liability.** Each store remains liable for the cards it sold. When a card is redeemed elsewhere, the selling store owes the redeeming store. This needs the system to track which store's value is being spent on every redemption, which gets messy once a card has been reloaded at three different stores. Use it only when there is a legal reason the program entity cannot hold customer money.

The rest of this repo assumes a central pool.

## Settlement flow

```
Period close (for example daily at 23:59 store local time)
   |
   per store: sum gift card sales (value issued or reloaded)
   per store: sum redemptions (value spent)
   per store: sum refunds back to cards, voids and adjustments
   |
   apply program fees and any selling commission
   |
   net position per store:
       sales owed to program
     - redemptions owed by program
     +/- fees, refunds, adjustments
   |
   positive -> store pays program
   negative -> program pays store
   |
   statements issued, disputes window opens, payments made
```

**Settle net, report gross.** Paying one net figure per store keeps bank transfers down. The statement still has to show sales, redemptions and fees separately, or no store accountant will sign it off.

**Same cut-off for everyone.** If stores span time zones, define the period in each store's local time and say so in the program rules. A sale at 23:58 and a redemption at 00:03 belong to different periods, and both sides should agree on that.

**Settle only completed transactions.** Authorisation holds that were never captured, and voided sales inside the void window, stay out. Late voids after settlement become adjustments in the next period.

## Refunds in a different store

This is where most disputes start. A customer spends a gift card at store B, then returns the item at store C, which puts the value back on the card.

If C's refund is treated as a negative redemption at C, then C is charged for value it never received. B keeps the money for goods that came back.

A fairer rule: when the return is linked to the original receipt, reverse the redemption at the store that made it (B). When the return has no receipt and the value goes onto a card as store credit, treat it as new issuance at the refunding store, so C is charged and also holds the returned goods. Write whichever rule you choose into the franchise or tenancy agreement before the first holiday season.

## Fees, commission and breakage

Programs usually charge something to cover running costs. Typical structures (all to be set in your own agreements):

- a percentage fee on redemptions, charged to the redeeming store
- a selling commission paid to the store that sold the card, to encourage sales
- a fixed monthly fee per store for the platform and card stock

Breakage is the value on cards that is never spent. Under a central pool it accrues to the program by default. Some programs share it back to stores in proportion to sales. Unclaimed property and expiry rules differ by country and state, so the finance team needs to confirm what can be recognised and when.

## Worked example (figures are illustrative)

All figures in this section are **illustrative**.

Three stores in a mall program, one week. Program fee 2% of redemptions, charged to the redeeming store. No selling commission.

| Store | Sales | Redemptions | Fee (2%) | Net position |
|-------|-------|-------------|----------|--------------|
| A | 10,000.00 | 4,000.00 | 80.00 | A pays 6,080.00 |
| B | 3,000.00 | 7,500.00 | 150.00 | Program pays B 4,350.00 |
| C | 5,000.00 | 4,200.00 | 84.00 | C pays 884.00 |

How A's figure works: A owes the program 10,000.00 of sales money. The program owes A 4,000.00 for redemptions less the 80.00 fee, which is 3,920.00. Net, A pays 6,080.00.

Check from the program's side. It receives 6,080.00 + 884.00 = 6,964.00 and pays out 4,350.00, so it ends the week 2,614.00 better off. That is the 2,300.00 increase in outstanding card balances (18,000.00 sold less 15,700.00 redeemed) plus 314.00 of fees. If the two numbers do not tie, something is missing from the statements.

Store B is the anchor tenant that everyone spends their cards in, so it is a net receiver almost every week. Expect B's finance team to care most about how quickly the program pays.

## Fields per settlement line

| Field | Why |
|-------|-----|
| `period_id`, `cutoff_at` | Which period and the exact close time |
| `store_id`, `legal_entity` | Who the statement is addressed to |
| `txn_type` | Sale, reload, redemption, refund-to-card, void, adjustment |
| `card_masked`, `txn_ref` | Last 4 only, plus the platform reference |
| `original_txn_ref` | Links a refund to the redemption it reverses |
| `gross_amount`, `fee_amount` | Reported separately even when settled net |
| `settlement_ref`, `paid_at` | Proof of payment |
| `dispute_status` | Open, accepted, rejected, carried to next period |

## Things that go wrong

**Refunds charged to the wrong store.** No `original_txn_ref`, so the system cannot tell a receipted return from store credit.

**Late voids.** A sale voided after the period closed is silently dropped instead of carried as an adjustment, and the totals no longer tie.

**Slow payment to net receivers.** The program collects from payers weekly but pays receivers monthly. The receiving stores are financing the program, and they notice.

**Store exits the program.** A franchisee leaves while customers still hold cards bought there. Under a central pool the program keeps honouring them. Under seller-keeps-liability, someone has to collect the exiting store's outstanding liability before it walks away.

**Fraud at one store.** Cards activated without payment at one till get spent at others. The program should be able to charge back redemptions on cards whose activation was never funded, and the agreement should say so.

A period-close checklist is in [docs/settlement-close-checklist.md](./docs/settlement-close-checklist.md).

---

## FAQ

**What is gift card cross-store settlement?**
It is the periodic process that moves money between stores and the program manager when a closed-loop gift card is sold in one store and redeemed in another.

**Who holds the gift card liability in a franchise or mall program?**
Usually the program manager, through a central pool. Stores pass sales money to the program and claim redemption value back from it.

**How is a store's net settlement position calculated?**
Sales owed to the program, minus redemptions owed by the program, adjusted for fees, refunds to cards and late voids. A positive result means the store pays; a negative result means it receives.

**How should a refund made at a different store be settled?**
If the return is linked to the original receipt, reverse the redemption at the store that originally redeemed it. If not, treat it as new issuance at the store that gave the refund.

**Who keeps gift card breakage in a multi-store program?**
By default the program manager, though some programs share it back to selling stores. Local unclaimed property rules may limit this.

**Does EVD System handle gift cards?**
EVD System by MoboGage covers voucher generation, inventory and lifecycle for telecom airtime, gift cards and loyalty vouchers.

---

## Glossary

| Term | Meaning |
|------|---------|
| Closed-loop gift card | A card that can only be spent within one brand, group or mall |
| Program manager | The entity that runs the card program and, in a central pool, holds the liability |
| Net position | What a store owes or is owed after sales, redemptions and fees are offset |
| Cut-off | The time a settlement period closes |
| Breakage | Card value that is never redeemed |
| Adjustment | A correction carried into a later period |

## Further reading

- [Voucher management system by MoboGage](https://evdsystem.com/voucher-management-system-mobogage/) on evdsystem.com
- [What is an electronic voucher?](https://evdsystem.com/evd-software/what-is-an-electronic-voucher/) on evdsystem.com
- Related repo: [closed-loop-gift-card-redemption](https://github.com/iwtglobal/closed-loop-gift-card-redemption)

## License

MIT. See [LICENSE](./LICENSE). Educational material only, not legal, accounting or tax advice.
