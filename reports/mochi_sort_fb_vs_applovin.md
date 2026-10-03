# Facebook vs AppLovin: does one predict the other? (Mochi Sort)

**Scope:** installs Sep 3–30, 2026 · Day 0 and D1 metrics only

- **AppLovin:** `MOCH_WW_IOS_ALL (BLD D7 CPM)` (iOS, Sep 22–28). The Aug 26 – Sep 3 AppLovin campaigns are excluded.
- **Facebook:** `MOCH_BR_ID_ADR_VID`, `MOCH_BR_ID_IOS_VID`, `MOCH_US_IOS_VID (MAI AEM D1)`.
- **Sources:** Snowflake `DIM_USERS` (attribution, country, install date) joined to `DAILY_ACTIVE_USERS` (Day 0 activity, D1 return) on `UNITY_USER_ID` + `GAME_ID`.

## 1. Answers

| Question | Answer |
|---|---|
| **Do Facebook results indicate AppLovin results?** | **Not for retention.** In US iOS, Day 0 engagement is similar on both networks, but AppLovin's D1 is less than half of Facebook's (10.8% vs 25.3%). Using Facebook to forecast AppLovin would overstate its D1 about 2.3×. |
| **In which countries?** | **Only the US (iOS) can be tested.** The two networks share no other country, so for any other country there's no evidence either way. |
| **Device or game behaviour: which explains retention?** | **Game behaviour.** Day 0 levels won is the best predictor of D1. Device is the weakest and explains none of the network gap. Behaviour explains about 1/3 of the gap; the remaining 2/3 is the network's audience itself. |

## 2. Where the networks overlap

| Platform · country | Facebook users | AppLovin users | Comparable? |
|---|---|---|---|
| iOS · US | 146 | 686 | ✅ the only overlap |
| iOS · BR / ID | 473 / 222 | 0 | ❌ |
| Android · ID / BR | 1,600 / 830 | 0 / 1 | ❌ |
| iOS · GB / CA / JP / DE / AU | 0–1 | 51–113 each | ❌ |

## 3. US iOS: same country, same platform

### 3a. By device group

Sorted by AppLovin users.

| Device | FB users | AL users | FB D1 | AL D1 | FB median levels won | AL median levels won | FB avg sessions | AL avg sessions | FB D0 ARPU | AL D0 ARPU |
|---|---|---|---|---|---|---|---|---|---|---|
| iPhone new (15–17) | 61 | 232 | **23.0%** | 12.9% | 6 | 6 | 1.67 | 1.23 | $0.04 | $1.08 |
| iPad | 8 | 224 | (50%, n=8) | 11.2% | 8 | 6 | 1.50 | 1.21 | $0.45 | $1.47 |
| iPhone mid (12–14) | 66 | 203 | **22.7%** | 8.4% | 8 | 6 | 1.44 | 1.23 | $0.16 | $1.15 |
| iPhone old (XS–11) | 10 | 27 | 30% | 7.4% | 13 | 6 | 1.60 | 1.52 | $0.07 | $0.69 |
| **Total** | **146** | **686** | **25.3%** | **10.8%** | **7** | **6** | **1.58** | **1.24** | **$0.12** | **$1.21** |

### 3b. By Day 0 levels won

Users who played the same amount, compared directly.

| Day 0 levels won | FB users | AL users | FB D1 | AL D1 |
|---|---|---|---|---|
| 0–2 | 26 | 155 | 11.5% | 4.5% |
| 3–5 | 26 | 142 | 23.1% | 7.7% |
| 6–10 | 43 | 185 | 25.6% | 11.9% |
| 11–20 | 34 | 158 | 23.5% | 12.7% |
| 21+ | 17 | 46 | 52.9% | 30.4% |

### 3c. By Day 0 sessions

| Day 0 sessions | FB users | AL users | FB D1 | AL D1 |
|---|---|---|---|---|
| 1 | 99 | 569 | 18.2% | 9.5% |
| 2 | 32 | 86 | 31.3% | 15.1% |
| 3+ | 15 | 31 | 60.0% | 22.6% |

**Takeaway:** Facebook retains about 2× better at every device group and every engagement level.

## 4. What explains retention

### 4a. Predictive power for D1, all users in scope

AUC: 0.5 = coin flip, 1.0 = perfect. Each factor's D1 rates were learned on one half of users and tested on the other half (2,122 test users).

| Rank | Factor | AUC |
|---|---|---|
| 1 | **Day 0 levels won** | **0.77** |
| 2 | Day 0 playtime | 0.76 |
| 3 | Day 0 levels × sessions | 0.76 |
| 4 | Day 0 sessions | 0.69 |
| 5 | Country + device | 0.65 |
| 6 | Country | 0.64 |
| 7 | Device group | 0.60 |
| 8 | Network (FB vs AL) | 0.60 |

Adding country to Day 0 behaviour doesn't improve the score (0.76 → 0.76).

### 4b. Why is AppLovin's D1 lower in US iOS? (gap: 14.5 pp)

Facebook users were reweighted to match AppLovin's mix on each factor.

| If Facebook users matched AppLovin's… | FB D1 becomes | Remaining gap | Share of gap explained |
|---|---|---|---|
| nothing (raw) | 25.3% | 14.5 pp | – |
| device mix | 32.4% | 21.6 pp | none |
| Day 0 levels won | 23.3% | 12.5 pp | 14% |
| Day 0 sessions | 22.0% | 11.2 pp | 23% |
| **Day 0 levels + sessions** | **20.5%** | **9.7 pp** | **33%** |

#### How to read this table

In US iOS:

- Facebook D1 = 25.3%
- AppLovin D1 = 10.8%
- Gap = 14.5 points

There are two possible explanations:

1. **AppLovin brought different kinds of users**, for example users who play less on Day 0 or use different devices. Then the gap is about user mix.
2. **AppLovin's users retain worse even when they look the same**, which points to the network's audience itself.

**How the table separates them (reweighting)**

Take Day 0 levels won as the example. Each network's users split into groups by how many levels they won:

| Levels won | Facebook D1 for this group | Share of AppLovin users in this group |
|---|---|---|
| 0–2 | 11.5% | 23% (155 of 686) |
| 3–5 | 23.1% | 21% |
| 6–10 | 25.6% | 27% |
| 11–20 | 23.5% | 23% |
| 21+ | 52.9% | 7% |

AppLovin has many more low-play users. 23% of its users won 0–2 levels, compared with 18% of Facebook's.

The question: what would Facebook's D1 be if its users were split across these groups the way AppLovin's are? Each Facebook group's D1 is multiplied by AppLovin's share for that group, and the results are added up:

> 11.5%×23% + 23.1%×21% + 25.6%×27% + 23.5%×23% + 52.9%×7% ≈ **23.3%**

So if Facebook had AppLovin's mix of light and heavy players, its D1 would drop from 25.3% to 23.3%. **Only 2 of the 14.5 points (14%) come from AppLovin users playing less.** The other 12.5 points remain even when you compare users who played the same amount.

**Bottom line**

- **About 1/3 of the gap** comes from AppLovin bringing users who play less on Day 0.
- **About 2/3 of the gap** remains even for users with the same Day 0 behaviour and devices. AppLovin's users are simply less likely to come back. Possible causes are the audience itself, the bidding goal (D7 ROAS vs D1 events), the different weeks, or the suspicious zero-play users.

## 5. Caveats

1. **Suspicious AppLovin users:** 155 of 686 (23%) won 0–2 levels with a median Day 0 playtime of about 2 seconds, but average **$1.65 Day 0 revenue**. That could be ad fraud, misattributed revenue, or ads shown before gameplay, and it needs checking. Without them, AppLovin's D1 is only about 12.6%.
2. **Not a clean test:** the campaigns ran in different weeks (AL Sep 22–28, FB Sep 28–30) and optimized for different goals (AL for D7 ROAS, FB for D1 app events).
3. **Small samples:** Facebook US has only 146 users, and the iPad and older-iPhone Facebook groups (8–10 users) are too small to read.
4. **Device groups** are a hand-built mapping from `DEVICE_MODEL`, not an official tiering. Users with no recorded device model are excluded from the factor ranking (4a).

## 6. Next steps

1. **Run a matched test:** Facebook and AppLovin in one or two shared countries (for example GB or DE), in the same week, with the same creatives.
2. **Use Day 0 levels won (11+) as the early quality read** for both networks.
3. **Investigate the zero-play, high-revenue AppLovin users** before trusting AppLovin's revenue numbers.
