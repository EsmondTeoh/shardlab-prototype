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
3. Tap the floating 🧪 button on the right and choose **+ 3 friends join** (or **Preview guest join screen** to see the name-entry step). Try the **Share** button for WhatsApp, Slack and Copy.
4. **Start ordering** → add an item → open the cart → **Lock order & choose payer**.
5. The vote opens with **I'll pay** already selected and your friends declining, so you are the payer by default. Use the floating 🧪 controls to try other scenarios: **Friends respond randomly**, **Everyone volunteers** (draw), **A friend pays instead** (you decline and a friend pays) and **Nobody volunteers**.
6. **Continue to payment** → enter a mobile number such as `12 345 6789` → pay with any method → use the "Simulate" buttons.
7. See the **cashback earned** card, then tap **Order again**.
8. Repeat with the same mobile number. Checkout now offers **Use ShardLab cashback**. The 👤 Profile button on the landing page shows the balance and history.

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
- Prototype controls live in a floating, collapsible button on the right edge of the group and vote screens.
- Checkout is prefilled with a mock Malaysian number (`12 345 6789`), so a second order on the same number finds the cashback.
- Cashback: 3% of the bill total credited to the payer's profile and redeemable at the next checkout on the same mobile number. Redemption can partly or fully cover a bill.

**Simulated**
- **Other group members:** there is no backend or real multi-device sync. Friends are generated in the page and "vote" on timers. The floating Prototype controls steer their behaviour (I'm the payer (default), friends respond randomly, everyone volunteers, a friend pays instead, nobody volunteers) and can add friends. The same list is available on the Who's paying page, where changes restart the vote.
- **Payments:** FPX, TNG, GrabPay, Boost, card and DuitNow are mock screens with "simulate approve/fail" buttons. The card form accepts any value except the decline test card `4000 0000 0000 0002`. The DuitNow QR is decorative and not scannable.
- **Account and cashback:** stored in `localStorage` per browser, keyed by mobile number. There is no login or OTP. A different browser or device shows no balance.
- **Kitchen status:** the Paid → Preparing → Ready tracker advances on timers.
- **Invite link:** `shardlab.my/g/...` is a placeholder domain. Sharing the message works, but the link does not join a real group.
- **Messaging and e-Invoice:** the "sent via WhatsApp" line and the MyInvois e-Invoice request form are UI only.
- **Chrome frame:** the status bar, address bar (branded `shardlab.my/kopi-kaki-cafe`), tab counter and menu are a static mock-up, not a real browser. The address is not the real hosting URL.
- **Draws:** the prototype user is the default payer, including in the "random" draw, which always lands on you when you are in the pool. A friend is picked only in the "A friend pays instead" scenario. A real product would pick truly at random.
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
  - Cashback never expires and has no cap.
  - A payer who volunteers can still change their vote until everyone has voted.
- The group payer is not notified separately. Everyone just sees the result on screen.
- No accessibility audit, and no handling for a person leaving the group mid-order.
- The visual design is inspired by StoreHub's layout and tone, then rebranded as ShardLab. It is not a pixel-faithful clone.

## 3. AI-assisted decisions

| # | Tool | What it suggested or generated | What I accepted or changed | How I checked it |
|---|------|-------------------------------|----------------------------|------------------|
| 1 | Claude Code | **Scope.** I asked for a replica of storehub.com/my focused on the consumer payment journey. Claude opened the site in the built-in browser at mobile width and read its text. It found a merchant-facing marketing site (POS features, a demo form) with no consumer checkout. It proposed building the consumer journey that a StoreHub-style POS would power: scan QR → menu → cart → checkout → pay → order status and receipt, with e-Invoice and Malaysian tax handling. | I accepted this reinterpretation. I did not ask for a literal copy of the marketing pages. Malaysian-specific details were Claude's suggestions and I kept them: FPX and e-wallets, SST, 5-sen rounding, MyInvois. | The page text and a mobile screenshot confirmed there was no consumer flow to copy. The flow itself is based on general knowledge of QR ordering, not on StoreHub's actual consumer app, which I did not inspect. |
| 2 | Claude Code | **Test-driven bug catch.** After generating the first version, Claude ran the full flow by script in the browser instead of assuming it worked. The cart came back empty and the totals were RM 0.00. The cause was a stepper button given the id `q+`, which is not a valid CSS selector, so the add-to-cart handler threw an error. | I accepted the diagnosis and fix: rename the ids to `qMinus` and `qPlus`. | I re-ran the same scripted flow. Two lattes with `SHARD10` gave RM 19.80 − 1.98 + 1.78 service + 1.18 SST + 0.02 rounding = **RM 20.80**. The arithmetic matches by hand, and the flow reaches the success screen. |
| 3 | Claude Code | **Group-payer rules.** My brief was ambiguous: "if everyone chooses themselves, a person will be randomly selected." Claude interpreted it as a random pick among everyone who volunteers (which covers the case where all do). It added a fallback for "nobody volunteers" (pick randomly or ask again), a 3-person minimum, and cashback on the pre-redemption bill total. It also built a Prototype controls panel so each branch can be reproduced on one device. | I accepted the volunteer-pool interpretation. The fallback, the minimum and the cashback base are assumptions I should confirm with the product owner. They are listed above as unverified. | Scripted runs of three scenarios. (a) All volunteer → random draw, random payer shown to everyone. (b) Only I volunteer → I pay, and a RM 53.30 bill credited **RM 1.60 (3%)**. (c) A second order on the same mobile showed a balance of RM 1.60, and redeeming it turned a RM 6.40 bill into **RM 4.80 to pay**. |

> Optional extras for the submission: screenshots of the vote and result screens, plus a prompt excerpt for each row.
