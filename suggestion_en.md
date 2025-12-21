# Secessionist Mechanism Redesign Proposal

## Summary

This document proposes a complete redesign of the secessionist mechanism in Europa Universalis V. The current system has fundamental flaws that create game-breaking exploits, violate historical realism, and break strategic decision-making. This proposal outlines design principles for a redesigned system that balances secessionist wars with traditional diplomacy, restores player control, and maintains historical authenticity.

**Core Principle:** Secessionist participation should be **optional** and **strategic**, not forced. Countries should be able to evaluate costs and benefits before committing to wars, reflecting the historical reality that 14th-century states prioritized national interests over cultural solidarity.

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

9. **World War Escalation - Asymmetric Alliance Calling (Severe)** - A single local Secessionists Rebellion can escalate into a world War due to asymmetric Alliance calling mechanics. This is **worse than historical world Wars** (WW1/WW2) where both sides could coordinate with Allies. The Defender CAN call all its Allies, while the forcibly called Attacker CANNOT call its own Allies, creating dangerous escalation mechanism where defenders can systematically use Secessionist rebellions to attack their enemies' Alliance blocs asymmetrically.

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
  - **Historical Context:** In 1337, countries prioritized dynastic interests and economic benefits over cultural solidarity. Forced participation contradicts this historical reality.

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

3. **Historical Realism:** The system must reflect 14th-century priorities:
   - Countries prioritize national interests over cultural solidarity
   - Wars are fought for strategic or economic benefit, not cultural unification
   - Cost-benefit analysis determines participation, not automatic cultural ties

4. **Original Owner Priority:** Original owners should have priority over culturally dominant countries when both exist. Historical claims matter more than cultural dominance.

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

### Historical Authenticity

The redesigned system must reflect 14th-century historical reality:
- Countries fought wars for dynastic interests, territorial expansion, and economic benefits
- Cultural solidarity was not a primary motivation for war
- States evaluated costs and benefits before committing to conflicts
- Small cultural minorities in foreign territories were not worth major wars unless there was clear strategic benefit

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

## Summary

The current secessionist mechanism requires a complete redesign, not patches. The fundamental issues stem from forced participation, lack of strategic control, and violation of historical realism. The proposed redesign principles focus on:

1. **Optional participation** with strategic decision-making
2. **Historical authenticity** reflecting 14th-century priorities
3. **Balance with traditional diplomacy** through appropriate costs and constraints
4. **Exclusion of landless rebels** that provide minimal benefit
5. **Player control** over war leadership and peace negotiations
6. **Prevention of exploitation** through proper constraints and optional participation

This redesign would restore strategic gameplay, maintain historical realism, and prevent the systematic exploits that currently plague the secessionist mechanism. The system would become a strategic choice rather than a forced mechanic, allowing players to make informed decisions based on cost-benefit analysis while reflecting the historical reality that 14th-century states prioritized national interests over cultural solidarity.

**Important Considerations:**
- This is a design proposal focusing on principles and concepts
- Implementation would require significant development resources
- Actual effectiveness would need validation through playtesting
- Paradox should evaluate this alongside other potential redesign approaches
- The complexity of a complete redesign must be balanced against development priorities

---

### Note on Contributing to Paradox Development

I am interested in contributing to Paradox Interactive's game development and would welcome opportunities to discuss how my analytical approach to game mechanics design could be valuable to the development team.
