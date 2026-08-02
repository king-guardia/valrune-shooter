> **STATUS: v0 — pending revision.** Written before the ability/status spreadsheets were reconciled into the plan. Several decisions in this document have since been **reversed**. Where this file conflicts with [`docs/decisions/`](decisions/), **the decision log wins.** Full revision happens in Phase 7.

# 09 — Backend, Monetization, Security & Analytics

## 9.1 The most important thing here

**A single-player game needs almost no backend.** Every component you add can break, cost money, leak data, and block a player from playing a game they paid for.

Minimum viable for v1.0: **verify purchases, serve DLC, store a cloud save, collect crashes, report analytics.**

**Absolute rule: the entire 30-contract campaign is playable offline, permanently, with no account.** Signing in only buys cloud save and purchase restore across devices.

## 9.2 GCP stack

| Need | Service |
|---|---|
| Identity | **Firebase Auth** + Play Games Services sign-in (no password, no email); anonymous fallback |
| Profile + cloud save | **Firestore** (`users/{uid}`) |
| Purchase verification | **Cloud Functions (2nd gen)** + Google Play Developer API |
| Refund/revoke | **Pub/Sub** ← Play Real-Time Developer Notifications |
| DLC delivery | **Cloud Storage** + signed URLs (§9.3) |
| Balance tuning post-launch | **Firebase Remote Config** (cached locally; must work offline) |
| Crashes | **Firebase Crashlytics** |
| Analytics | **GA4 for Firebase** → **BigQuery export** (§9.6) |
| Ads | **AdMob** (§9.7) |
| Secrets | **Secret Manager** |

Estimated cost at 10k MAU: **under $15/month.** Set a billing alert at $50 on day one.

Godot integration: talk to Firestore and Functions over **REST with an ID token** rather than via plugins. Fewer moving parts, no plugin-version pain, more portable.

## 9.3 Purchases

### Landscape (mid-2026, verify before setting prices)

<cite index="24-1">Following the Epic litigation, Google's policies as launched on December 9, 2025 allow developers to link users to external content or offer alternative billing systems, and on March 4, 2026 Google entered a new settlement agreement with Epic with a revised Modified Injunction filed with the court</cite>. <cite index="28-1">Under the emerging global model, transactions completed within 24 hours of an in-app link-out carry a 20% service fee, with a separate 5% billing fee in the US, UK and EEA when using Google Play Billing; rollout began 30 June 2026 in the US, UK and EU</cite>. <cite index="21-1">Google has stated developers will pay 10% on their first $1 million in annual earnings regardless of billing method</cite>.

This is actively litigated. Any specific figure may have moved.

### Recommendation: Play Billing only

At your revenue, the fee delta is a few hundred dollars a year — not worth a second payment path plus webshop, fraud stack, VAT/MOSS and US sales tax nexus, and chargeback handling. Play Billing gives you global tax remittance, refunds, family sharing, restore, and a verifiable purchase token, all one tap.

Revisit above roughly $50k/year. **Abstract the entitlement *source*** so adding a provider later is a small change.

**Implementation:** Godot's first-party `GodotGooglePlayBilling` plugin (Godot 4.2+, GDScript API with signals). **All SKUs are non-consumable one-time purchases** — no consumables, no subscriptions. Acknowledge every purchase immediately (Google auto-refunds unacknowledged purchases after 3 days). Ship a findable **Restore Purchases**.

### DLC delivery — gate the download

Content not on the device can't be unlocked by patching a boolean.

| Mechanism | Notes |
|---|---|
| **Play Asset Delivery** | Google hosts free, but **not entitlement-aware** — any client can request an on-demand pack. Solves size, not access |
| **GCS + signed URLs** | Cloud Function checks entitlement, issues a time-limited URL. Content never reaches an unentitled device. DLC is ~15–40MB, so bandwidth is pennies. **Recommended** |
| Encrypted in the AAB | Bytes on every device, useless without a per-user key. Combine with PAD if size ever matters |

### Verification flow

```
Client purchase → purchase token
  → Cloud Function verifyPurchase(token, productId, idToken)
      → Play Developer API purchases.products.get
      → validate purchaseState, productId, packageName, not consumed
      → write users/{uid}/entitlements/{productId}
      → return SIGNED ENTITLEMENT JWT (30-day expiry)
  → client caches token; verifies signature locally each launch (public key embedded)
  → DLC content requires a valid token AT LOAD TIME, not just at download

Pub/Sub RTDN (refund / revoke / chargeback)
  → Cloud Function sets granted = false
  → next online launch, access lapses gracefully
```

**Entitlement is permanent once verified locally.** A player who verifies a purchase and then goes offline for a year keeps everything already unlocked, forever. Re-verification gates only two things: **downloading new DLC content** and **cloud save**. This is the design that survives a six-month offline gap, an expired card, or a dead server — and it costs nothing, because the pirate who patches the client was never going to be stopped by an expiring token anyway.

Refresh opportunistically in the background whenever a connection exists, so the entitlement record stays current without the player ever waiting on it.

## 9.4 Architectural enforcement of the pledge

Three CI tests, each cheap and permanent:

1. **`Wallet.add_credits()` is callable only from `ContractCompleteController`.** Static analysis over the codebase. If any purchase→credits path appears — from a future you at 2am or an over-helpful assistant — CI fails.
2. **Only `EntitlementService.can_access()` reads `requires_entitlement`.** Grep assertion.
3. **The IAP catalogue is a hardcoded constant list** of exactly the SKUs in `01` §1.7, in a file whose header comment points at the pledge. Deliberate friction.

Plus the **pay-to-win test** (`01` §1.7): strongest base build within 15% of strongest DLC build, via the sim. This is the one that protects the model from your own future incentives.

Mirror all four into `.cursor/rules`.

## 9.5 Anti-piracy — a realistic position

**You cannot stop it, and that's fine.** Android apps run on hardware the attacker controls. Goals: make casual cheating require effort, make *entitlement* theft require server compromise, and **never inconvenience a legitimate player.**

### The ladder

| Rung | Measure | Server | Stops | Effort |
|---|---|---|---|---|
| 0 | Local signature check on Play Billing response against the embedded public key | No | Forged/replayed purchase data | Hours |
| 1 | Cloud Function verifies with Play Developer API, issues signed JWT | Yes | Client-side forgery, shared receipts | ~1 day |
| 2 | DLC delivered and loaded only with a valid token | Yes | Unlocking content you don't have | ~1 day |
| 3 | RTDN → revoke on refund | Yes | Buy-play-refund-keep | ~½ day |
| 4 | Play Integrity API | Yes | Modified binaries, sideloads | ~1 day |

**Play Integrity is not a store badge.** It produces no consumer-visible marker, no listing benefit, and no ranking effect. It is a server-side API that reports whether an install is unmodified and Play-licensed. If it is worth adding, it is worth adding for what it actually does, not for a badge that does not exist.

**Build 1 + 2 + 3.** Three days, under $5/month. Moves the attack from "flip a bool in a decompiled APK" to "obtain files from a purchaser and sideload," which is where essentially all realistic loss lives.

**If you use rung 4, restrict it to gating cloud save only.** Play Integrity false positives concentrate among rooted devices, custom ROMs (GrapheneOS, LineageOS), uncertified OEM devices, and developers — **precisely your stated audience.** WC3-custom-map veterans skew toward people who flash ROMs.

**Never build:** root detection that blocks play · hard bans (there is no ban in this game; a cheater in single-player harms only themselves) · server-authoritative gameplay.

### Failure ladder

```
Entitlement invalid or unverifiable
  → 14-day offline grace period
  → after grace: disable CLOUD SAVE and RESTORE only
  → local play of everything already unlocked continues
  → one non-nagging notice explaining how to restore
```

**A false negative locks DLC content and never touches the save.** Progress is sacred.

### False negatives — build for them, they're the common case

| Cause | Mitigation |
|---|---|
| No network at first launch post-purchase | **Optimistic unlock**: rung-0 signature valid → unlock immediately, verify in background. Never make a payer wait on your server |
| New device / account switch | Prominent Restore Purchases in Settings *and* Store |
| Verification errored | **Distinguish "failed to complete" from "returned negative."** Network error → keep entitlement indefinitely. Definitive negative → start the grace clock |
| Family library, regional edge cases | Manual grant |
| Anything unforeseen | In-app "I bought this but it's locked" button emailing order ID, uid, product, last verification result. A `manual_grant` flag in Firestore the function always respects |

Expect **1–3 support emails per 1,000 purchases.** Manageable solo; the manual-grant hatch makes each a 60-second fix.

### Fraudulent entitlement rate

Through the intended path, effectively zero — entitlement originates from Google's servers. Real vectors are narrow: carded purchases (RTDN revokes), buy-download-refund (mitigated by requiring the token at load time), and APK patching (unstoppable, and only relevant at a popularity level that would itself be success).

## 9.6 Analytics

Firebase → GA4 → BigQuery. Two purposes: design decisions and **conversion optimization for paid acquisition.**

### Design events

`stage_start` / `stage_complete` / `stage_fail` (contract_id, duration, difficulty, clauses) · **`killed_by`** (enemy_id, contract_id) — the single most useful number you have · `build_snapshot` at each boss (element levels, attunement config, loadout, top stat ranks) · `upgrade_purchased` · `element_point_spent` · `loadout_changed` · `respec` · `rune_equipped` · `challenge_earned` · `settings_changed`

### Conversion events

Purchase intent forms over ~20 hours of play while the terminal conversion is sparse, so you need **dense early proxy signals** to optimize against:

| Event | Params | Why |
|---|---|---|
| `world_complete` | world_id | Clean early milestone, high volume |
| `encyclopedia_locked_element_view` | element_id, dwell_ms | **The strongest intent signal in the game.** Someone reading the Chrono page is shopping |
| `side_world_map_tap` | pack_id | Direct expansion interest |
| `store_view` | source | Funnel entry |
| `purchase_complete` | product_id, **value, currency** | The real conversion |

`value` and `currency` must be populated correctly or downstream value-based bidding won't work.

**The Cloud Function also writes a server-side purchase record.** Client purchase events are lossy and spoofable; the server number is the source of truth when reconciling against ad spend.

**Not collected:** location, contacts, cross-app tracking, session-length obsession. A genuine opt-out in Settings that actually stops collection.

## 9.7 Ads

**AdMob interstitial, skippable.** Trigger: **miniboss defeat and boss defeat only** — 10 slots per playthrough, ~1 per 18 minutes.

| Rule | Value |
|---|---|
| Frequency cap | 1 per 10 minutes regardless of triggers |
| Never shown on | death, retry, contract start, menu entry, pause, first session |
| Removed by | any purchase, permanently, immediately |
| Rewarded ads | **None.** Time-for-currency breaks the pledge |

Implemented at exactly one node in the state machine (`06` §6.1).

The consent and advertising-ID infrastructure overlaps substantially with what paid user acquisition requires anyway, which is what makes the marginal compliance cost acceptable. Ad revenue is a partial offset against media spend, not a primary line.

## 9.8 Privacy and compliance

- **Privacy policy** at a real URL, describing every SDK including AdMob.
- **Data Safety form** must match reality — mismatches get apps pulled. AdMob means declaring an advertising identifier.
- **GDPR/CCPA consent flow** for EU users (Google's UMP SDK).
- **Rating:** Everyone 10+ / PEGI 7. **Do not opt into Designed for Families** — it pulls in COPPA obligations you don't want, and ads make it worse.
- **Account deletion** path required by Play. A Cloud Function deleting the Firestore doc and Auth user. One afternoon.

## 9.9 Store listing prep (start at M4)

Feature graphic 1024×500 · icon 512×512 · 4–8 phone screenshots (**lead with the Hangar** — the build screen is your differentiator, not a generic explosion) · 30s capture video · 80-char short description · full description · content rating · Data Safety · privacy policy URL · $25 Play Console fee.

**The gate that surprises people:** <cite index="16-1">newly created personal developer accounts must run a closed test with a minimum of 12 testers opted-in for at least the last 14 days continuously before applying for production access</cite>. <cite index="20-1">This applies to personal accounts created after November 13, 2023; organization accounts and older personal accounts are exempt</cite>.

- Create the account at **M0** — verification and the clock both take real time.
- An **organization account** (legal entity + D-U-N-S) skips the requirement entirely. If you were forming an LLC anyway, doing it early has this bonus.
- If for any reason the organization route falls through, line up 12 real testers on real devices around M4. Avoid paid tester farms — some are fine, some get accounts suspended, and the downside is catastrophic.
