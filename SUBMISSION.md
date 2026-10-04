# ShardLab: Mobile Group Ordering & Payment Prototype

## 1. Prototype and access

**Links**
- **Live prototype (GitHub Pages):** https://esmondteoh.github.io/shardlab-prototype/
- **Source (GitHub repo):** https://github.com/EsmondTeoh/shardlab-prototype

- **File:** `index.html`. It is a single, self-contained file with no build step and no dependencies.
- **To run:** double-click `index.html` to open it in any modern browser, or run `python3 -m http.server 8765` in the same folder and visit `http://localhost:8765/index.html`.
- **Layout:** the app always runs inside a 375px-wide Google Chrome mobile window (status bar, address bar, tab counter, menu), on desktop and on phones. The Chrome controls are decorative; only the app inside is interactive. Every screen was checked at exactly 375px width for overflow.
- **Screen recording:** `[ATTACH: recording not yet made]`

### Suggested walkthrough (about 2 minutes)
1. Landing → choose **Group order** → **Continue**.
2. Either **Create a group** (enter your name) or **Join a group** (enter a code like `KK-4821` and your name) → **Continue**.
3. Wait about 2 seconds: the other members join by themselves, and **Continue** unlocks once the group has 3 people. Try the **Share** button for WhatsApp, Slack and Copy.
4. **Start ordering** → add an item → open the cart → **Lock order & choose payer**.
5. On **Who's paying?** nothing is pre-selected and nothing happens until you choose **I'll pay** or **Not me**. Most friends decline, and occasionally one volunteers. Run the vote a few times to see each outcome:
   - **I'll pay** → you are the payer (if a friend also volunteered, a short draw runs and lands on you).
   - **Not me** while a friend volunteered → that friend pays and you get a "paid by" confirmation.
   - **Not me** while nobody volunteered → the "Nobody volunteered" fallback.
6. **Continue to payment** → the mobile number is prefilled (`12 3456 789`) → pay with any method → use the "Simulate" buttons.
7. See the **cashback earned** card: the 3% is **locked until your next visit or order**, so it cannot be used on the order that earned it. The Profile shows it as locked. Then tap **Order again**.
8. Start a new order (**Order again**) or reload the page (a later visit): the locked cashback is now available. Place another order as **Dine-in**, **Takeaway** or **Group order**. Checkout recognises your last mobile number and shows a **ShardLab cashback** card with your balance. Choose **Use my cashback** (takes it off the bill; any remainder stays saved, and a fully covered bill skips payment) or **Keep saving** (the default; nothing is deducted). The 👤 Profile button on the landing page shows the balance and history (edit the number to look up another).

## 2. What works, what is simulated, what is incomplete

**Works (exercised by scripted runs through the UI)**
- Full solo journey: menu → item options → cart → checkout → payment method → mock gateway → processing → success with e-receipt.
- Malaysian bill maths:
  - 10% service charge on dine-in.
  - 6% SST.
  - Rounding to the nearest 5 sen.
  - `SHARD10` promo (10% off, capped at RM8).
- Group flow:
  - Lobby with a minimum of 3 and a maximum of 12 people, with both Create a group and Join a group.
  - Name required to join, with duplicate-name rejection.
  - Shared cart grouped by person.
  - Volunteer vote with live status visible to everyone.
  - Random pick when several volunteer, and a fallback when nobody does.
  - Payer and non-payer paths.
- Invite sharing from the group lobby: **WhatsApp** opens a real `wa.me` link with the invite prefilled. **Slack** uses the phone's native share sheet when available. Otherwise it copies the invite and opens Slack, because a plain web page cannot post into Slack. **Copy link** also works.
- Checkout is prefilled with a mock Malaysian number (`12 3456 789`), so a second order on the same number finds the cashback.
- Cashback: 3% of the bill total credited to the payer's profile, **locked until their next visit or order** (it unlocks when they tap Order again or reopen the app, and never applies to the order that earned it). From then on, for dine-in, takeaway or group orders, the customer chooses at checkout to use it (partly or fully covering the bill) or keep saving it. A group payer who uses cashback still earns 3% on the full bill.

**Simulated**
- **Other group members:** there is no backend or real multi-device sync. Friends are generated in the page, join on their own 2 seconds after the group opens, and "vote" on timers. Most decline; at most half ever volunteer.
- **Payments:** FPX, TNG, GrabPay, Boost, card and DuitNow are mock screens with "simulate approve/fail" buttons. The card form accepts any value except the decline test card `4000 0000 0000 0002`. The DuitNow QR is decorative and not scannable.
- **Account and cashback:** stored in `localStorage` per browser, keyed by mobile number; the last number used is remembered so the next visit recognises you. There is no login or OTP. A different browser or device shows no balance.
- **Kitchen status:** the Paid → Preparing → Ready tracker advances on timers.
- **Invite link:** `shardlab.my/g/...` is a placeholder domain. Sharing the message works, but the link does not join a real group.
- **Messaging and e-Invoice:** the "sent via WhatsApp" line and the MyInvois e-Invoice request form are UI only.
- **Chrome frame:** the status bar, address bar (branded `shardlab.my/kopi-kaki-cafe`), tab counter and menu are a static mock-up, not a real browser. The address is not the real hosting URL.
- **Draws:** when several people volunteer, the draw always lands on the prototype user if they are among the volunteers. A friend pays only when the user declines. A real product would pick truly at random.
- **Menu, store and prices:** invented ("Kopi Kaki Café").

**Incomplete or unverified**
- Not exercised end to end:
  - Takeaway mode.
  - The card, wallet and DuitNow gateways.
  - The failure, cancel and expiry states.
  - The e-Invoice toggle.
- Visual QA was limited. I screenshotted the lobby and the success screen. I did not screenshot the vote or result screens, and there is no cross-browser or real-device testing.
- Business rules I assumed rather than were given:
  - A group needs at least 3 people.
  - The cap is 12.
  - Cashback is 3% of the total bill before any cashback is redeemed.
  - Cashback never expires and has no cap, and "next visit or order" means the next order started after this one finishes, or the next time the app is opened. Unused cashback keeps carrying over until the customer chooses to use it.
  - A payer who volunteers can still change their vote until everyone has voted.
- The group payer is not notified separately. Everyone just sees the result on screen.
- No accessibility audit, and no handling for a person leaving the group mid-order.
- The visual design is inspired by StoreHub's layout and tone, then rebranded as ShardLab. It is not a pixel-faithful clone.

## 3. AI-assisted decisions

| # | Tool | What it suggested or generated | What I accepted or changed | How I checked it |
|---|------|-------------------------------|----------------------------|------------------|
| 1 | Claude Code | **Scope.** I asked for a replica of storehub.com/my focused on the consumer payment journey. Claude opened the site in the built-in browser at mobile width and read its text. It found a merchant-facing marketing site (POS features, a demo form) with no consumer checkout. It proposed building the consumer journey that a StoreHub-style POS would power: scan QR → menu → cart → checkout → pay → order status and receipt, with e-Invoice and Malaysian tax handling. | I accepted this reinterpretation. I did not ask for a literal copy of the marketing pages. Malaysian-specific details were Claude's suggestions and I kept them: FPX and e-wallets, SST, 5-sen rounding, MyInvois. | The page text and a mobile screenshot confirmed there was no consumer flow to copy. The flow itself is based on general knowledge of QR ordering, not on StoreHub's actual consumer app, which I did not inspect. |
| 2 | Claude Code | **Test-driven bug catch.** After generating the first version, Claude ran the full flow by script in the browser instead of assuming it worked. The cart came back empty and the totals were RM 0.00. The cause was a stepper button given the id `q+`, which is not a valid CSS selector, so the add-to-cart handler threw an error. | I accepted the diagnosis and fix: rename the ids to `qMinus` and `qPlus`. | I re-ran the same scripted flow. Two lattes with `SHARD10` gave RM 19.80 − 1.98 + 1.78 service + 1.18 SST + 0.02 rounding = **RM 20.80**. The arithmetic matches by hand, and the flow reaches the success screen. |
| 3 | Claude Code | **Group-payer rules.** My brief was ambiguous: "if everyone chooses themselves, a person will be randomly selected." Claude interpreted it as a random pick among everyone who volunteers (which covers the case where all do). It added a fallback for "nobody volunteers" (pick randomly or ask again), a 3-person minimum, and cashback on the pre-redemption bill total. | I accepted the volunteer-pool interpretation. The fallback, the minimum and the cashback base are assumptions I should confirm with the product owner. They are listed above as unverified. | Scripted runs of the vote paths. (a) A friend volunteers and I decline → that friend is shown as payer to everyone. (b) Only I volunteer → I pay, and a RM 53.30 bill credited **RM 1.60 (3%)**. (c) A second order on the same mobile showed a balance of RM 1.60, and redeeming it turned a RM 6.40 bill into **RM 4.80 to pay**. |

> Optional extras for the submission: screenshots of the vote and result screens, plus a prompt excerpt for each row.
