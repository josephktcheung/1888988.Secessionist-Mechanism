# Secessionist Mechanism Hotfix Mod - White Paper Summary

**Purpose:** This document summarizes the fundamental limitations of modding-based solutions for the secessionist mechanism issues and why an engine fix is ultimately required.

**Target Audience:** Modders, game designers, and developers considering mod-based solutions.

---

## ⚠️ Critical Limitation: Reactive, Not Preventive (Standard Approach)

**Fundamental Problem:** Standard modding approaches cannot achieve the ideal goal of preventing wars that should never happen.

### What Mods Cannot Do (Standard Reactive Approach)

Once a secessionist war is declared, the hardcoded engine logic fires immediately and many effects are **irreversible**:

1. **Hardcoded Engine Logic Fires First:** The engine's `on_civil_war_start` logic calls culturally dominant countries **BEFORE** any mod can intervene
2. **Immediate Penalties:** Truce breaking penalties (-50 stability, +1 war exhaustion, +25 antagonism) are applied instantly when countries with truces are forced into war
3. **PU Breaks:** Personal Unions with `break_on_war = yes` may break immediately, before mods can prevent it
4. **Countries Already in War:** By the time mods can respond (even with 1-day delay), countries are already participants
5. **Damage Already Done:** The war has started, countries are called, penalties are applied, and relationships may have broken

### What Mods Can Do (Limited Mitigation - Standard Approach)

Standard modding approaches can only mitigate damage **after the fact**:

- Allow countries to exit immediately (but cannot undo penalties already applied)
- Remove unwanted truces created by white peace
- Prevent the "Annex Revolter" exploit via war goal validation
- Allow players to control participation via game rules (but only after entry has occurred)

**The Reality (Standard Approach):** By the time `on_civil_war_start` fires and mods can respond, the war has already started, countries are already called, and penalties are already applied. Standard modding is fundamentally a **damage mitigation tool**, not a prevention mechanism.

**Conclusion:** The ideal solution - preventing inappropriate wars from starting - requires either an engine fix OR the alternative preventive approach described below.

---

## Alternative Approach: Simulate Independence Then Declare War (Theoretical)

**Concept:** Completely bypass the current secessionist mechanism by simulating the rebel winning independence first, then creating a new war with correct leadership.

### Key Design Principles

1. **Only Apply to Secessionist Civil Wars (Not Nationalist Independence Wars)**
   - Secessionist rebels are culture-based and trigger `on_civil_war_start`
   - Nationalist rebels (from cores) create regular wars and don't trigger secessionist mechanism
   - **Why:** Nationalist independence wars don't need this fix - they're already regular wars with correct behavior

2. **Assume Protector Exists**
   - If no culturally dominant country exists, the secessionist mechanism wouldn't trigger anyway
   - The rebel would fight alone as a regular nationalist independence war (which is fine)
   - **Focus:** Only handle cases where a protector exists and needs correct war leadership

3. **Priority System for Protector Selection**
   - **Order:** Original Owner > Neighboring Same-Culture Countries > Culturally Dominant Country
   - **Why:** Respects historical claims and prevents distant countries from being called

### Implementation Approach

**Why We Cannot Reuse Original Secessionist Country Creation Code:**
- The country creation logic is **hardcoded in the engine** (C++ code, not in any moddable game file)
- `on_civil_war_start` fires **after** the country is already created
- We **cannot prevent `start_civil_war` from firing** when rebel progress reaches 100% - there's no `on_rebel_progress` or `on_rebel_breakout` on_action to intercept
- The engine logic is **not moddable** - we can only react after the fact

**Recommended Approach: Create Country Ourselves (Before `start_civil_war` Fires)**

**Intercept Timing Options:**
- **Option A: Intercept at 99% progress** (via monthly event checking `rebel_progress >= 0.99` AND `rebel_last_months_progress >= 0.01`)
  - **Safest approach** - guarantees we catch it before 100%
  - Requires validation before creating country (verify rebel still exists and conditions still met)
- **Option B: Intercept at 100% progress** (via monthly event checking `rebel_progress >= 1.0`)
  - **Uncertain timing** - depends on execution order (requires testing)
  - If monthly events fire BEFORE `start_civil_war`: ✅ Can intercept
  - If `start_civil_war` fires immediately: ❌ Cannot intercept (broken mechanism already triggered)

**Implementation Steps:**
1. Create monthly event triggered by `monthly_country_pulse` on_action
2. Check for secessionist rebels meeting intercept conditions (see timing options above)
3. **Critical Validation:** Before creating country, verify:
   - Rebel still exists (not destroyed by player actions)
   - `rebel_progress >= 0.99` (still at high progress)
   - `rebel_last_months_progress > 0` (not suppressed)
   - If validation fails, abort (player intervened)
4. Identify all locations with rebelling culture that rebel controls (using `dominant_culture` or pop allegiance)
5. Create country tag (dynamically generated or culture-based)
6. Add cores for new country tag to those culture-based locations
7. Use `create_country_from_cores_in_our_locations` to create the country
8. Make new country secessionist vassal of protector (using `make_subject_of` with `subject_type:secessionists`)
9. Have protector declare war on original owner (using `cb_conquer_province` or appropriate CB)
10. **Destroy the original rebel** to prevent `start_civil_war` from firing

**Why This Works:** By destroying the rebel before (or immediately when) it reaches 100%, we prevent `start_civil_war` from firing, avoiding the broken mechanism entirely.

### Implementation Flow

```
1. Detect secessionist rebel at 99-100% progress (via monthly event)
2. Validate rebel still exists and conditions still met
3. Identify rebelling culture
4. Find protector using priority system (Original Owner > Neighbors > Culturally Dominant)
5. If no protector found → Skip (becomes regular nationalist war)
6. Identify all locations with rebelling culture that rebel controls
7. Add cores for new country tag to those locations
8. Create country from cores: `create_country_from_cores_in_our_locations`
9. Make new country secessionist vassal of protector
10. Have protector declare war on original owner
11. Destroy original rebel
```

### Advantages

- **Preventive, Not Reactive:** Intercepts before `start_civil_war` fires, preventing the broken mechanism entirely
- **Correct War Leadership:** Both attacker and defender leaders are set correctly from the start
- **Allows Alliance Calling:** Both sides can call allies (symmetric warfare)
- **Respects Original Ownership:** Priority system gives original owners first chance
- **Prevents Exploits:** No forced participation, no truce breaking penalties, no PU breaks
- **Matches Intended Design:** New country becomes secessionist vassal as intended

### Limitations and Complexity Assessment

**Summary:** This approach has **significant complexity** that may make it impractical for modding.

**Key Limitations:**
- **Requires Protector:** Only works if a protector exists (but this is fine - if none exists, it's a regular war)
- **Timing Uncertainty:** Unknown execution order - do monthly events fire before `start_civil_war`? Requires extensive testing
- **Player Intervention Race Conditions:** Player can improve stability or use cabinet crackdown between check and execution, requiring validation steps and potential cleanup mechanisms
- **Multiple Edge Cases:** Negative monthly progress (suppression), very small monthly progress, rebel destroyed by other means, multiple rebels, nested subjects

**Complexity Factors:**
1. **Timing Uncertainty (High Risk):** Unknown execution order requires testing; if timing is wrong, broken mechanism fires anyway
2. **Player Intervention Edge Cases (High Complexity):** Race conditions between check and execution are difficult to handle reliably
3. **Multiple Validation Points:** Must verify rebel exists, progress is high, monthly growth is positive, and handle validation failures
4. **Implementation Complexity:** Requires careful handling of location identification, core creation, country creation, vassal setup, war declaration, and rebel destruction

**Recommendation:**

- **For Modders:** This approach may be **too complex** for reliable modding. The combination of timing uncertainty, player intervention edge cases, and multiple validation points creates a high risk of bugs and edge case failures.

- **Simpler Alternative:** Consider sticking with the **standard reactive approach** (damage mitigation) which is:
  - More predictable (reacts after fact, no timing issues)
  - Fewer edge cases (no player intervention race conditions)
  - More reliable (proven modding patterns)
  - Less complex (fewer moving parts)

- **For Engine Fix:** This approach demonstrates what an **ideal solution** would look like, but the complexity suggests it's better suited for an engine fix where:
  - Timing is guaranteed (engine controls execution order)
  - No race conditions (engine handles all checks atomically)
  - No validation needed (engine knows state is valid)
  - Edge cases handled by design (engine logic is comprehensive)

**Conclusion:** While theoretically possible, this approach's complexity makes it **questionable for modding**. An engine fix would be more reliable and comprehensive.
