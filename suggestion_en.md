# Secessionist Mechanism Redesign Proposal

## Summary

This document proposes a complete redesign of the secessionist mechanism in Europa Universalis V. The current system has fundamental flaws that create game-breaking exploits and break strategic decision-making. This proposal outlines design principles for a redesigned system that balances secessionist wars with traditional diplomacy, restores player control, and maintains game balance.

**Core Principle:** Secessionist participation should be **optional** and **strategic**, not forced. Countries should be able to evaluate costs and benefits before committing to wars, allowing players to make informed strategic decisions based on expected benefits versus potential costs.

**Note:** This is a design proposal for Paradox to consider. It focuses on design principles and concepts rather than technical implementation details, as a complete redesign requires fundamental changes to the current system.

---

## Problem Statement

The current secessionist mechanism has twelve critical issues that fundamentally break gameplay:

1. **Same-Culture Countries Forced to Join (Most Severe)** - Forced war participation with no refusal option. **CRITICAL FLAW: Original owners completely ignored** - the system only looks at cultural dominance, ignoring historical ownership. Example: Ottomans (original owner of rebelling territories) are completely ignored in favor of Eretnids (culturally dominant but not original owner) in Byzantium case. Original owners have no opportunity to reclaim their former territories.

2. **Personal Union Breaking (Severe - VERIFIED)** - When a Personal Union Junior Partner is the culturally dominant Country for a Secessionists Rebellion, it is forcibly called to support the Rebellion against its Senior Partner, immediately breaking the Personal Union. Verified: France-Sicily Personal Union breaks when Sicilian rebels appear.

3. **Vassal War Leadership Problem (Severe)** - If a player creates a Vassal that becomes the Defender in a Civil War, the player is forcibly dragged into the War but does not become the War Leader. The AI Vassal controls all peace negotiations. The player cannot call Allies, cannot obtain Military Access, and must bear all War costs without control.

4. **Inconsistency in Alliance Auto-Call** - When the player is the defensive War Leader, Allies can be asked to join. But when a Vassal is the defensive War Leader, the player cannot call Allies to join.

5. **Coalition Bypass Mechanism (Severe Exploit)** - When a forcibly called Country is in a Coalition, it cannot call Coalition members to join the War because it is the Attacker. This allows the Defender to bypass Coalition restrictions and attack Coalition members individually.

6. **AI Vassal Suboptimal Peace Decisions** - AI Vassal may choose white peace instead of annexing the Revolter even when annexation cost is very low, causing players to lose territories they should have gained.

7. **Overlord Forced to Join** - Secessionists automatically and forcibly call Overlords to War through subject type properties.

8. **Truce Breaking Penalties** - Forcibly called Countries with Truce are incorrectly judged as actively breaking Truce, triggering severe penalties (-50 Stability, +1 War Exhaustion, +25 Antagonism) even though they had no choice.

9. **World War Escalation - Asymmetric Alliance Calling (Severe)** - A single local Secessionists Rebellion can escalate into a world War due to asymmetric Alliance calling mechanics. The Defender CAN call all its Allies, while the forcibly called Attacker CANNOT call its own Allies, creating dangerous escalation mechanism where defenders can systematically use Secessionist rebellions to attack their enemies' Alliance blocs asymmetrically. This creates an unbalanced game mechanic where one side can coordinate while the other cannot.

10. **Annex Revolter Button Misapplied to War Participant Countries (Severe - GAME-BREAKING)** - When a Secessionists Rebellion spawns as a Society of Pops (landless Country with no territory), the Rebel Country has no territory. When the Rebel Country's troops are wiped out and the Rebellion is eliminated, the War continues and a war participant Country becomes the War Leader. The Annex Revolter button, designed to annex the entire Revolter Country, can then be used to annex the entire war participant Country (e.g., France - 3rd largest Country at game start) with no Antagonism generated. **VERIFIED EXAMPLES:** France was annexed by England; Jalayirids was annexed by Ottomans.

11. **Subjects Forced to Support Distant Rebellions (Severe)** - Any subject (e.g., Vassal, Secessionists subject, or other subject types) that happens to be culturally dominant for the rebelling culture is forcibly called to support distant rebellions of the same culture, even when they are thousands of kilometers away and have no logical connection to the rebellion. **VERIFIED EXAMPLE:** Yuán case - small Mongolian subject in Mongolia region forced to support rebellion in Georgia 6000+ km away.

12. **Inconsistent Behavior of Land-Based vs. Landless Secessionists Rebellions (Severe)** - When a Secessionists Rebellion spawns, its behavior differs dramatically based on whether it is land-based (owns territory) or landless (Society of Pops with no territory). Land-based Secessionists become Vassals of the culturally dominant Country when they win, while landless Secessionists become independent Countries with no reward for the culturally dominant Country. The AI culturally dominant Country may not take any land even at 100% Warscore, only accepting War Reparations. **VERIFIED EXAMPLE:** England vs France case - France fought entire War and gained only minimal War Reparations (314-350 ducats), no territory or Vassal.

**Root Cause:** The current system uses hardcoded engine logic that forcibly calls culturally dominant countries into secessionist wars without player control, ignores original ownership, and creates asymmetric escalation mechanisms. Many of these behaviors cannot be modified through game scripts alone, requiring a complete redesign.

---

## War Declaration System Imbalance and Strategic Control Issues

### The Two War Declaration Systems

EU5 currently has two parallel war declaration systems with fundamentally different costs, constraints, and player control:

**Traditional War Declaration System:**
- Requires building Spy Network (costs resources)
- Requires creating Casus Belli (takes time, costs resources)
- Requires Diplomats (diplomatic capacity)
- Player has full control over timing and target
- Allies can refuse call to arms
- Respects Truces, guarantees, and Alliances
- Has diplomatic action cooldowns

**Secessionist Civil War System:**
- Automatic when secessionist rebels reach 100% progress
- No Spy Network required
- No Casus Belli required
- No Diplomats required
- No player control - happens automatically
- No refusal possible - same-culture countries are forcibly called
- Bypasses Truces, guarantees, and Alliances
- No cooldowns - can happen immediately after previous wars

### Fundamental Imbalance

The secessionist system creates an asymmetric bypass of traditional diplomacy:

1. **No Resource Costs:** Traditional wars require Spy Network investment, Diplomats, and time. Secessionist wars require none of these.

2. **No Diplomatic Constraints:** Traditional wars respect Truces, guarantees, and Alliances. Secessionist wars bypass all of these.

3. **Forced Participation:** Traditional wars allow refusal. Secessionist wars force participation with no choice.

4. **Unpredictable Timing:** Traditional wars are player-controlled. Secessionist wars are automatic and unpredictable.

5. **Exploitable Nature:** The system enables systematic exploits where players can force opponents into wars during Truce periods, bypass Coalition restrictions, and use Overlords as weapons.

### Game Design and Balance Problem

The current forced participation system creates fundamental game design and balance issues:

**Game Balance Issues:**
- **Forced Participation Removes Strategic Choice:** The system forces countries into Wars without allowing players to evaluate costs and benefits. This removes a core strategic decision-making element from the game.
- **No Cost-Benefit Analysis:** Players cannot assess whether participation is worthwhile before being forced into Wars. This breaks strategic gameplay where players should be able to make informed decisions.
- **Asymmetric Game Mechanics:** The forced participation mechanism creates an asymmetric bypass of traditional diplomacy systems, allowing players to exploit the system to force opponents into Wars during Truce periods, bypass Coalition restrictions, and use Overlords as weapons.
- **Current System Problem:** The forced participation mechanism applies the same forced cultural solidarity logic throughout the entire 500-year game period, from 1337 to 1837. This removes strategic decision-making and creates game-breaking exploits regardless of the time period.

**Cost-Benefit Analysis:**
- Even if culturally dominant countries might sympathize with secessionists, they would not automatically join wars unless the benefit justifies the cost
- Countries prioritize the collective benefit of the whole nation over a relatively small size of same-culture citizens living in a foreign country
- States would evaluate: (1) Expected Vassal size and value, (2) Potential War costs (manpower, Ducats, War Exhaustion), (3) Risk of escalation (defender's Allies, Coalition members), (4) Opportunity cost of committing resources to a potentially long War for minimal gain

**War Duration Risk:**
- Secessionist wars can drag on for years with minimal benefits, making forced participation even less justified
- Late-game complexity: Great powers with presence in multiple continents make secessionist Wars risky and hard to justify, even for large Vassals
- The current system forces countries to fight wars they would rationally refuse based on cost-benefit analysis

**Strategic Rationality:**
- The current system breaks strategic decision-making by forcing participation without allowing cost-benefit evaluation
- Players cannot make informed decisions about whether participation is worthwhile
- This violates core strategic gameplay principles where players should have control over their Country's foreign policy decisions

### Strategic Decision-Making Problems

**Landless Rebels Provide Almost No Benefit:**
- Landless rebels (Society of Pops) only provide 314-350 ducats in War Reparations, no useful Vassal, no territory
- No reasonable player would want to fight a major War for this minimal benefit
- **Recommendation:** Exclude landless rebels from secessionist mechanism entirely - they should use standard rebel mechanics instead

**Unknown Benefits Before War:**
- Players can't evaluate benefit before participation:
  - Unknown Vassal size (territories, population) until War breaks out
  - Unknown War Reparations amount (AI Secessionists takes their cut)
  - Unknown War costs (especially late-game when great powers have global presence across multiple continents)
  - Hard to justify participation when War may involve great powers, even if Vassal size is large

**Cost-Benefit Analysis Impossibility:**
- Players cannot perform rational cost-benefit analysis before being forced into War:
  - Cannot compare expected Vassal value vs. potential War costs (manpower, Ducats, War Exhaustion)
  - Cannot assess risk of escalation (defender's Allies, Coalition members, military strength)
  - Cannot evaluate opportunity cost of committing resources to a potentially long War for minimal gain
  - Cannot prioritize collective national benefit over a small cultural minority in foreign territory
  - **Game Design Issue:** Forced participation removes strategic decision-making from players. Players should be able to evaluate whether supporting a Secessionists Rebellion is strategically worthwhile based on expected benefits vs. potential costs, regardless of the time period. The current system forces participation without allowing this evaluation, breaking core strategic gameplay.

**No Way to Refuse Participation:**
- Players are forced into Wars with unknown or minimal benefits
- No strategic control over participation decision

**AI War Leader Control:**
- When Vassal is War Leader, player cannot control War duration, peace negotiations, or when War ends
- Player loses all strategic control over the War outcome

---

## Proposed Redesign Principles

### Core Design Principles

1. **Optional Participation:** Secessionist participation must be optional, not forced. Countries should be able to refuse without penalties.

2. **Strategic Decision-Making:** Players must be able to evaluate costs and benefits before committing to participation. This requires:
   - Early warning system (notify when rebels reach 80-90% progress)
   - Benefit preview showing expected Vassal size, War Reparations, and War costs
   - Risk assessment showing defender's Allies, Coalition members, and military strength
   - Cost-benefit analysis to help players decide if participation is worthwhile

3. **Strategic Decision-Making:** The system must support strategic gameplay:
   - Players should prioritize strategic and economic benefits when deciding whether to participate
   - Wars should be fought based on cost-benefit analysis, not forced by automatic cultural ties
   - Players must be able to evaluate expected benefits vs. potential costs before committing to participation

4. **Original Owner Priority:** Original owners should have priority over culturally dominant countries when both exist. Original ownership provides stronger strategic justification for participation than cultural dominance alone, as original owners have direct territorial claims and strategic interests in reclaiming their former territories.

5. **Balance with Traditional Diplomacy:** Secessionist wars should have appropriate costs and constraints that align with traditional war declaration system:
   - Require resource investment (e.g., Spy Network or ducats)
   - Respect diplomatic constraints (allow refusal, respect Truces when possible)
   - Provide player control over timing and participation

6. **Exclude Landless Rebels:** Landless rebels (Society of Pops) provide almost no benefit and should be excluded from secessionist mechanism entirely. They should use standard rebel mechanics instead.

7. **Player War Leader Control:** The culturally dominant country (or original owner) should become War Leader, not the Vassal, giving players full control over peace negotiations and War duration.

8. **Prevent Exploitation:** The system must prevent systematic exploits:
   - No forced participation during Truce periods
   - No bypassing Coalition restrictions
   - No using Overlords as weapons
   - No forcing subjects into distant wars

### Proposed Solution Framework

**Early Warning and Benefit Preview System:**
- When secessionist rebels reach 80-90% progress, eligible countries are notified
- Notification includes benefit preview:
  - Expected Vassal size (territories, population, military strength)
  - Expected War Reparations (with AI Secessionists taking their cut)
  - Potential War costs (manpower, Ducats, War Exhaustion)
  - Risk assessment (defender's Allies, Coalition members, military strength)
  - Cost-benefit analysis to help players decide if participation is worthwhile

**Optional Participation with Resource Investment:**
- Countries can choose to support secessionists by investing resources (e.g., Spy Network or ducats)
- This investment grants the right to join the civil war when it breaks out
- Countries can refuse without penalties
- Resource investment aligns secessionist wars with traditional diplomacy costs

**Alternative Solution: Support Rebels Covert Action Requirement (Simpler Implementation)**
- **Key Change:** Secessionist Civil War only triggers when the original owner or culturally prominent Country proactively uses Support Rebels covert action on the target Country
- **Mechanism:** When a Country uses Support Rebels covert action to support secessionist Rebels in another Country, this creates a "support commitment" that justifies calling that Country to arms when the Secessionists Rebellion breaks out
- **Strategic Decision:** The supporting Country makes a strategic decision to support the Rebellion through the covert action, making their participation justified and voluntary
- **CRITICAL REQUIREMENT: Benefit Preview System:** The game must provide clear benefit preview information when players consider using Support Rebels covert action for secessionist Rebels:
  - **Potential Secessionists population size** - Show the total population of the rebelling culture in the target Country
  - **Number of Locations/Provinces** - Show how many territories the Secessionists Rebellion could potentially control
  - **Expected Vassal size and value** - Estimate the size and military strength of the Vassal that would be gained if the Rebellion succeeds
  - **Cost-Benefit Analysis** - Compare expected benefits vs. costs (Spy Network investment, potential War costs, risk of escalation)
  - **Why This Is Essential:** Without clear benefit preview, players cannot justify using Support Rebels covert action for secessionists when there are other ways to wage Wars in EU5 (traditional Casus Belli, Diplomats, etc.). The mechanism would become seldomly used if players cannot evaluate whether the potential Vassal is worth the investment.
- **Benefits:**
  - Eliminates forced participation - only Countries that chose to support through covert action are called
  - Requires resource investment (Spy Network for covert actions)
  - Aligns with traditional diplomacy system (covert actions require Spy Network)
  - Provides player control - players decide whether to support Rebels before the War breaks out
  - Makes participation strategic - players evaluate costs and benefits before using the covert action
  - Enables informed decision-making - benefit preview allows players to justify the investment
- **Implementation:** 
  - Modify secessionist calling mechanism to only call Countries that have an active Support Rebels relation with the Rebels in the target Country
  - Add benefit preview UI/tooltip to Support Rebels covert action when targeting secessionist Rebels, showing potential Secessionists population size, number of territories, and expected Vassal value

**Priority System:**
- Original owners receive highest priority (if they still exist)
- Culturally dominant countries receive secondary priority
- Rivals of defender can also participate (at higher cost)
- Subjects are excluded from participation to prevent conflicts with overlords

**War Leadership:**
- If a primary culture country (or original owner) supports the rebellion, they become War Leader when the war breaks out
- This gives players full control over peace negotiations and War duration
- Prevents AI Vassal from controlling the War

**Exclusion of Landless Rebels:**
- Landless rebels (Society of Pops) are excluded from secessionist mechanism
- They use standard rebel mechanics instead
- This prevents wars for minimal benefit (314-350 ducats)

**Balanced Escalation:**
- Both sides can coordinate with Allies
- No asymmetric Alliance calling
- Prevents world War escalation through forced participation

---

## Design Considerations

### Game Balance and Strategic Gameplay

The redesigned system must maintain game balance and support strategic gameplay:
- Players should evaluate costs and benefits before committing to Wars
- Participation should be based on strategic considerations (expected Vassal value, War costs, risk of escalation) rather than forced by automatic cultural ties
- Small cultural minorities in foreign territories should not automatically trigger major Wars unless players choose to support them based on strategic evaluation

### Strategic Gameplay

The redesigned system must support strategic decision-making:
- Players need information to make informed decisions
- Cost-benefit analysis must be possible before participation
- Players should have control over their country's foreign policy
- Wars should be fought for strategic reasons, not forced by game mechanics

### Balance and Exploitation Prevention

The redesigned system must prevent systematic exploits:
- No forced participation during Truce periods
- No bypassing Coalition restrictions
- No using Overlords as weapons
- No forcing subjects into distant wars
- No exploiting landless rebels to annex major powers

### Integration with Existing Systems

The redesigned system must integrate with existing EU5 mechanics:
- Align costs with traditional diplomacy system
- Respect diplomatic constraints where appropriate
- Maintain unique secessionist flavor while preventing exploitation
- Work with existing Alliance, Coalition, and subject systems

---

## Essential Quick Fixes (Hotfixes)

While a complete redesign is recommended, the following essential quick fixes can address the most critical exploits without requiring a full system redesign:

### Hotfix 1: Fix Annex Revolter Button Exploit (Issue 10 - CRITICAL)

**Problem:** The Annex Revolter button can be used to annex war participant Countries (e.g., France) instead of only actual Revolter Countries, allowing major powers to be annexed at 25% cost with no Antagonism.

**Quick Fix:** Add verification check to ensure the target Country is actually a Revolter before allowing the Annex Revolter option:
- Verify target is the original Revolter that started the War
- OR verify target has revolter flag/variable or is a Rebel Country type
- Disable the option for war participant Countries that are not actual revolters

**Alternative Fix:** If Annex Revolter is used on a non-revolter, apply normal annexation costs (100% instead of 25%) and generate appropriate Antagonism.

**Impact:** Prevents game-breaking exploit where major powers can be annexed at minimal cost.

### Hotfix 2: Exclude Landless Rebels from Secessionist Mechanism (Issue 12)

**Problem:** Landless Secessionists (Society of Pops) create secessionist Wars but provide almost no benefit (only 314-350 ducats), and enable the Annex Revolter exploit when eliminated.

**Quick Fix:** Exclude landless Secessionists from the secessionist mechanism entirely:
- When secessionist rebels reach 100% progress, check if rebellion would spawn as `country_type = pop` (Society of Pops)
- If landless, use standard rebel mechanics instead of secessionist Civil War mechanism
- Do not create secessionist Civil War for landless rebels

**Impact:** Prevents wars for minimal benefit and eliminates the root cause of the Annex Revolter exploit.

### Hotfix 3: Exclude Subjects from Secessionist Calls (Issue 11)

**Problem:** Subjects (e.g., Vassal, Secessionists subject) that are culturally dominant are forcibly called to support distant rebellions, even thousands of kilometers away.

**Quick Fix:** Exclude all subjects from secessionist calls:
- Add check in secessionist calling mechanism: `is_subject = no`
- This should be consistent with how Vassals and Fiefdoms are already excluded (verified in Issue 2 testing)

**Impact:** Prevents subjects from being dragged into distant Wars and prevents conflicts with Overlords.

### Hotfix 4: Exclude Personal Union from Secessionist Calls (Issue 2)

**Problem:** Personal Union Junior Partners are forcibly called, immediately breaking the Personal Union.

**Quick Fix:** Exclude Personal Union Junior Partners from secessionist calls (same as Vassals and Fiefdoms):
- Add check: `NOT = { is_subject_of_type = subject_type:personal_union }` or similar
- Ensure consistency with other subject type exclusions

**Impact:** Prevents Personal Union from breaking unexpectedly.

### Hotfix 5: Fix Truce Breaking Penalties (Issue 8)

**Problem:** Forcibly called Countries with Truce are incorrectly judged as actively breaking Truce, triggering severe penalties (-50 Stability, +1 War Exhaustion, +25 Antagonism) even though they had no choice. This can also be exploited to cause harm to opponents by intentionally triggering Secessionists Rebellions during Truce periods.

**Quick Fix:** Skip same-culture Countries that have an active Truce with the Defender when Secessionists choose their supporting Country:
- Add check in secessionist calling mechanism: `NOT = { has_truce_with = $defender$ }`
- If no eligible Country exists (all have Truces), the Secessionists Rebellion proceeds without external support
- This prevents the exploit entirely and avoids unfair penalties for countries that had no choice

**Impact:** Prevents unfair penalties for countries that had no choice and prevents exploit where players can force opponents into Wars during Truce periods.

### Implementation Priority

**Phase 1 (Critical - Immediate):**
- Hotfix 1: Fix Annex Revolter exploit (prevents game-breaking exploit)
- Hotfix 2: Exclude landless rebels (prevents root cause of exploit)

**Phase 2 (High Priority):**
- Hotfix 3: Exclude subjects (prevents distant war participation)
- Hotfix 4: Exclude Personal Union (prevents union breaking)

**Phase 3 (Medium Priority):**
- Hotfix 5: Fix truce breaking penalties (fairness fix)

**Note:** These hotfixes address the most critical exploits but do not solve the fundamental design issues (forced participation, lack of strategic control, etc.). A complete redesign is still recommended for long-term solution.

### Hotfix 6: Support Rebels Covert Action Requirement (Alternative Solution)

**Problem:** The current system forcibly calls countries into secessionist Wars without any prior strategic decision or resource investment, removing player control and creating exploits.

**Alternative Solution:** Require Support Rebels covert action before secessionist Civil War can call countries to arms:
- **Key Change:** Secessionist Civil War only calls countries to arms if they have proactively used Support Rebels covert action on the target Country before the Rebellion breaks out
- **Mechanism:** When a Country uses Support Rebels covert action to support secessionist Rebels in another Country, this creates a "support commitment" that justifies calling that Country to arms when the Secessionists Rebellion breaks out
- **Strategic Decision:** The supporting Country makes a strategic decision to support the Rebellion through the covert action, making their participation justified and voluntary
- **CRITICAL REQUIREMENT: Benefit Preview System:** The game must provide clear benefit preview information when players consider using Support Rebels covert action for secessionist Rebels:
  - **Potential Secessionists population size** - Show the total population of the rebelling culture in the target Country
  - **Number of Locations/Provinces** - Show how many territories the Secessionists Rebellion could potentially control
  - **Expected Vassal size and value** - Estimate the size and military strength of the Vassal that would be gained if the Rebellion succeeds
  - **Cost-Benefit Analysis** - Compare expected benefits vs. costs (Spy Network investment, potential War costs, risk of escalation)
  - **Why This Is Essential:** Without clear benefit preview, players cannot justify using Support Rebels covert action for secessionists when there are other ways to wage Wars in EU5 (traditional Casus Belli, Diplomats, etc.). The mechanism would become seldomly used if players cannot evaluate whether the potential Vassal is worth the investment compared to alternative war declaration methods.
- **Benefits:**
  - Eliminates forced participation - only Countries that chose to support through covert action are called
  - Requires resource investment (Spy Network for covert actions)
  - Aligns with traditional diplomacy system (covert actions require Spy Network)
  - Provides player control - players decide whether to support Rebels before the War breaks out
  - Makes participation strategic - players evaluate costs and benefits before using the covert action
  - Enables informed decision-making - benefit preview allows players to justify the investment
- **Implementation:** 
  - Modify secessionist calling mechanism to only call Countries that have an active Support Rebels relation with the Rebels in the target Country
  - Add benefit preview UI/tooltip to Support Rebels covert action when targeting secessionist Rebels, showing potential Secessionists population size, number of territories, and expected Vassal value

**Impact:** Transforms secessionist participation from forced to optional and strategic, addressing the core design issue while using existing game systems (covert actions). However, **benefit preview is essential** - without it, the mechanism may be seldomly used as players cannot justify the investment compared to alternative war declaration methods.

---

## Summary

The current secessionist mechanism requires a complete redesign, not patches. The fundamental issues stem from forced participation, lack of strategic control, and game balance problems. The proposed redesign principles focus on:

1. **Optional participation** with strategic decision-making
2. **Strategic gameplay** where players evaluate costs and benefits before committing
3. **Balance with traditional diplomacy** through appropriate costs and constraints
4. **Exclusion of landless rebels** that provide minimal benefit
5. **Player control** over war leadership and peace negotiations
6. **Prevention of exploitation** through proper constraints and optional participation

This redesign would restore strategic gameplay, maintain game balance, and prevent the systematic exploits that currently plague the secessionist mechanism. The system would become a strategic choice rather than a forced mechanic, allowing players to make informed decisions based on cost-benefit analysis and strategic considerations.

**Important Considerations:**
- This is a design proposal focusing on principles and concepts
- Implementation would require significant development resources
- Actual effectiveness would need validation through playtesting
- Paradox should evaluate this alongside other potential redesign approaches
- The complexity of a complete redesign must be balanced against development priorities

---

### Note on Contributing to Paradox Development

I am interested in contributing to Paradox Interactive's game development and would welcome opportunities to discuss how my analytical approach to game mechanics design could be valuable to the development team.
