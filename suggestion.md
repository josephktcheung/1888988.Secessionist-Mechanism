# Secessionist Mechanism Redesign Proposal

## Summary

This document proposes a complete redesign of the secessionist mechanism in Europa Universalis V. The current system has fundamental flaws that create game-breaking exploits and break strategic decision-making. This proposal outlines design principles for a redesigned system that balances secessionist wars with traditional diplomacy, restores player control, and maintains game balance.

**Core Principle:** Secessionist participation should be **optional** and **strategic**, not forced. Countries should be able to evaluate costs and benefits before committing to wars, allowing players to make informed strategic decisions based on expected benefits versus potential costs.

**Note:** This is a design proposal for Paradox to consider. It focuses on design principles and concepts rather than technical implementation details, as a complete redesign requires fundamental changes to the current system.

---

## Problem Statement

The current secessionist mechanism has twelve critical issues that fundamentally break gameplay:

1. **Same-Culture Countries Forced to Join (Most Severe)** - Forced war participation with no refusal option. **CRITICAL FLAW: Original owners completely ignored** - the system only looks at cultural dominance, ignoring historical ownership. Example: $TUR$ (original owner of rebelling territories) are completely ignored in favor of $ERE$ (culturally dominant but not original owner) in $BYZ$ case. Original owners have no opportunity to reclaim their former territories.

2. **$game_concept_personal_union$ Breaking (Severe - VERIFIED)** - When a $game_concept_personal_union$ $game_concept_junior_partner$ is the culturally dominant $game_concept_country$ for a $secessionists$ $game_concept_rebellion$, it is forcibly called to support the $game_concept_rebellion$ against its $game_concept_senior_partner$, immediately breaking the $game_concept_personal_union$. Verified: $FRA$-$SIC$ $game_concept_personal_union$ breaks when Sicilian rebels appear.

3. **$vassal$ $game_concept_war$ Leadership Problem (Severe)** - If a player creates a $vassal$ that becomes the $game_concept_defender$ in a $game_concept_civil_war$, the player is forcibly dragged into the $game_concept_war$ but does not become the $game_concept_war_leader$. The AI $vassal$ controls all peace negotiations. The player cannot call $game_concept_allies$, cannot obtain $game_concept_military_access$, and must bear all $game_concept_war$ costs without control.

4. **Inconsistency in $game_concept_alliance$ Auto-Call** - When the player is the defensive $game_concept_war_leader$, $game_concept_allies$ can be asked to join. But when a $vassal$ is the defensive $game_concept_war_leader$, the player cannot call $game_concept_allies$ to join.

5. **$game_concept_coalition$ Bypass Mechanism (Severe Exploit)** - When a forcibly called $game_concept_country$ is in a $game_concept_coalition$, it cannot call $game_concept_coalition$ members to join the $game_concept_war$ because it is the $game_concept_attacker$. This allows the $game_concept_defender$ to bypass $game_concept_coalition$ restrictions and attack $game_concept_coalition$ members individually.

6. **AI $vassal$ Suboptimal Peace Decisions** - AI $vassal$ may choose white peace instead of annexing the $game_concept_revolter$ even when annexation cost is very low, causing players to lose territories they should have gained.

7. **Overlord Forced to Join** - $secessionists$ automatically and forcibly call $game_concept_overlords$ to $game_concept_war$ through subject type properties.

8. **$game_concept_truce$ Breaking Penalties** - Forcibly called $game_concept_countries$ with $game_concept_truce$ are incorrectly judged as actively breaking $game_concept_truce$, triggering severe penalties (-50 $game_concept_stability$, +1 $game_concept_war_exhaustion$, +25 $game_concept_antagonism$) even though they had no choice.

9. **World War Escalation - Asymmetric $game_concept_alliance$ Calling (Severe)** - A single local $secessionists$ $game_concept_rebellion$ can escalate into a world $game_concept_war$ due to asymmetric $game_concept_alliance$ calling mechanics. The $game_concept_defender$ CAN call all its $game_concept_allies$, while the forcibly called $game_concept_attacker$ CANNOT call its own $game_concept_allies$, creating dangerous escalation mechanism where defenders can systematically use Secessionist rebellions to attack their enemies' $game_concept_alliance$ blocs asymmetrically. This creates an unbalanced game mechanic where one side can coordinate while the other cannot.

10. **$WAR_LATERALVIEW_ANNEX_REVOLTER$ Button Misapplied to War Participant $game_concept_countries$ (Severe - GAME-BREAKING)** - When a $secessionists$ $game_concept_rebellion$ spawns as a $game_concept_society_of_pops$ (landless $game_concept_country$ with no territory), the $game_concept_rebel$ $game_concept_country$ has no territory. When the $game_concept_rebel$ $game_concept_country$'s troops are wiped out and the $game_concept_rebellion$ is eliminated, the $game_concept_war$ continues and a war participant $game_concept_country$ becomes the $game_concept_war_leader$. The $WAR_LATERALVIEW_ANNEX_REVOLTER$ button, designed to annex the entire $game_concept_revolter$ $game_concept_country$, can then be used to annex the entire war participant $game_concept_country$ (e.g., $FRA$ - 3rd largest $game_concept_country$ at game start) with no $game_concept_antagonism$ generated. **VERIFIED EXAMPLES:** $FRA$ was annexed by $ENG$; $JAL$ was annexed by $TUR$.

11. **Subjects Forced to Support Distant Rebellions (Severe)** - Any subject (e.g., $vassal$, $secessionists$ subject, or other subject types) that happens to be culturally dominant for the rebelling culture is forcibly called to support distant rebellions of the same culture, even when they are thousands of kilometers away and have no logical connection to the rebellion. **VERIFIED EXAMPLE:** $YUA$ case - small Mongolian subject in Mongolia region forced to support rebellion in Georgia 6000+ km away.

12. **Inconsistent Behavior of Land-Based vs. Landless $secessionists$ $game_concept_rebellions$ (Severe)** - When a $secessionists$ $game_concept_rebellion$ spawns, its behavior differs dramatically based on whether it is land-based (owns territory) or landless ($game_concept_society_of_pops$ with no territory). Land-based $secessionists$ become $vassal$s of the culturally dominant $game_concept_country$ when they win, while landless $secessionists$ become independent $game_concept_countries$ with no reward for the culturally dominant $game_concept_country$. The AI culturally dominant $game_concept_country$ may not take any land even at 100% $game_concept_war_score$, only accepting $game_concept_war_reparations$. **VERIFIED EXAMPLE:** $ENG$ vs $FRA$ case - $FRA$ fought entire $game_concept_war$ and gained only minimal $game_concept_war_reparations$ (314-350 ducats), no territory or $vassal$.

**Root Cause:** The current system uses hardcoded engine logic that forcibly calls culturally dominant countries into secessionist wars without player control, ignores original ownership, and creates asymmetric escalation mechanisms. Many of these behaviors cannot be modified through game scripts alone, requiring a complete redesign.

---

## War Declaration System Imbalance and Strategic Control Issues

### The Two War Declaration Systems

EU5 currently has two parallel war declaration systems with fundamentally different costs, constraints, and player control:

**Traditional War Declaration System:**
- Requires building $game_concept_spy_network$ (costs resources)
- Requires creating $game_concept_casus_belli$ (takes time, costs resources)
- Requires $game_concept_diplomats$ (diplomatic capacity)
- Player has full control over timing and target
- Allies can refuse call to arms
- Respects $game_concept_truce$s, guarantees, and $game_concept_alliance$s
- Has diplomatic action cooldowns

**Secessionist Civil War System:**
- Automatic when secessionist rebels reach 100% progress
- No $game_concept_spy_network$ required
- No $game_concept_casus_belli$ required
- No $game_concept_diplomats$ required
- No player control - happens automatically
- No refusal possible - same-culture countries are forcibly called
- Bypasses $game_concept_truce$s, guarantees, and $game_concept_alliance$s
- No cooldowns - can happen immediately after previous wars

### Fundamental Imbalance

The secessionist system creates an asymmetric bypass of traditional diplomacy:

1. **No Resource Costs:** Traditional wars require $game_concept_spy_network$ investment, $game_concept_diplomats$, and time. Secessionist wars require none of these.

2. **No Diplomatic Constraints:** Traditional wars respect $game_concept_truce$s, guarantees, and $game_concept_alliance$s. Secessionist wars bypass all of these.

3. **Forced Participation:** Traditional wars allow refusal. Secessionist wars force participation with no choice.

4. **Unpredictable Timing:** Traditional wars are player-controlled. Secessionist wars are automatic and unpredictable.

5. **Exploitable Nature:** The system enables systematic exploits where players can force opponents into wars during $game_concept_truce$ periods, bypass $game_concept_coalition$ restrictions, and use $game_concept_overlords$ as weapons.

### Game Design and Balance Problem

The current forced participation system creates fundamental game design and balance issues:

**Game Balance Issues:**
- **Forced Participation Removes Strategic Choice:** The system forces countries into $game_concept_wars$ without allowing players to evaluate costs and benefits. This removes a core strategic decision-making element from the game.
- **No Cost-Benefit Analysis:** Players cannot assess whether participation is worthwhile before being forced into $game_concept_wars$. This breaks strategic gameplay where players should be able to make informed decisions.
- **Asymmetric Game Mechanics:** The forced participation mechanism creates an asymmetric bypass of traditional diplomacy systems, allowing players to exploit the system to force opponents into $game_concept_wars$ during $game_concept_truce$ periods, bypass $game_concept_coalition$ restrictions, and use $game_concept_overlords$ as weapons.
- **Current System Problem:** The forced participation mechanism applies the same forced cultural solidarity logic throughout the entire 500-year game period, from 1337 to 1837. This removes strategic decision-making and creates game-breaking exploits regardless of the time period.

**Cost-Benefit Analysis:**
- Even if culturally dominant countries might sympathize with secessionists, they would not automatically join wars unless the benefit justifies the cost
- Countries prioritize the collective benefit of the whole nation over a relatively small size of same-culture citizens living in a foreign country
- States would evaluate: (1) Expected $vassal$ size and value, (2) Potential $game_concept_war$ costs (manpower, $game_concept_gold$, $game_concept_war_exhaustion$), (3) Risk of escalation (defender's $game_concept_allies$, $game_concept_coalition$ members), (4) Opportunity cost of committing resources to a potentially long $game_concept_war$ for minimal gain

**War Duration Risk:**
- Secessionist wars can drag on for years with minimal benefits, making forced participation even less justified
- Late-game complexity: Great powers with presence in multiple continents make secessionist $game_concept_wars$ risky and hard to justify, even for large $vassal$s
- The current system forces countries to fight wars they would rationally refuse based on cost-benefit analysis

**Strategic Rationality:**
- The current system breaks strategic decision-making by forcing participation without allowing cost-benefit evaluation
- Players cannot make informed decisions about whether participation is worthwhile
- This violates core strategic gameplay principles where players should have control over their $game_concept_country$'s foreign policy decisions

### Strategic Decision-Making Problems

**Landless Rebels Provide Almost No Benefit:**
- Landless rebels ($game_concept_society_of_pops$) only provide 314-350 ducats in $game_concept_war_reparations$, no useful $vassal$, no territory
- No reasonable player would want to fight a major $game_concept_war$ for this minimal benefit
- **Recommendation:** Exclude landless rebels from secessionist mechanism entirely - they should use standard rebel mechanics instead

**Unknown Benefits Before War:**
- Players can't evaluate benefit before participation:
  - Unknown $vassal$ size (territories, population) until $game_concept_war$ breaks out
  - Unknown $game_concept_war_reparations$ amount (AI $secessionists$ takes their cut)
  - Unknown $game_concept_war$ costs (especially late-game when great powers have global presence across multiple continents)
  - Hard to justify participation when $game_concept_war$ may involve great powers, even if $vassal$ size is large

**Cost-Benefit Analysis Impossibility:**
- Players cannot perform rational cost-benefit analysis before being forced into $game_concept_war$:
  - Cannot compare expected $vassal$ value vs. potential $game_concept_war$ costs (manpower, $game_concept_gold$, $game_concept_war_exhaustion$)
  - Cannot assess risk of escalation (defender's $game_concept_allies$, $game_concept_coalition$ members, military strength)
  - Cannot evaluate opportunity cost of committing resources to a potentially long $game_concept_war$ for minimal gain
  - Cannot prioritize collective national benefit over a small cultural minority in foreign territory
  - **Game Design Issue:** Forced participation removes strategic decision-making from players. Players should be able to evaluate whether supporting a $secessionists$ $game_concept_rebellion$ is strategically worthwhile based on expected benefits vs. potential costs, regardless of the time period. The current system forces participation without allowing this evaluation, breaking core strategic gameplay.

**No Way to Refuse Participation:**
- Players are forced into $game_concept_wars$ with unknown or minimal benefits
- No strategic control over participation decision

**AI $game_concept_war_leader$ Control:**
- When $vassal$ is $game_concept_war_leader$, player cannot control $game_concept_war$ duration, peace negotiations, or when $game_concept_war$ ends
- Player loses all strategic control over the $game_concept_war$ outcome

---

## Proposed Redesign Principles

### Core Design Principles

1. **Optional Participation:** Secessionist participation must be optional, not forced. Countries should be able to refuse without penalties.

2. **Strategic Decision-Making:** Players must be able to evaluate costs and benefits before committing to participation. This requires:
   - Early warning system (notify when rebels reach 80-90% progress)
   - Benefit preview showing expected $vassal$ size, $game_concept_war_reparations$, and $game_concept_war$ costs
   - Risk assessment showing defender's $game_concept_allies$, $game_concept_coalition$ members, and military strength
   - Cost-benefit analysis to help players decide if participation is worthwhile

3. **Strategic Decision-Making:** The system must support strategic gameplay:
   - Players should prioritize strategic and economic benefits when deciding whether to participate
   - Wars should be fought based on cost-benefit analysis, not forced by automatic cultural ties
   - Players must be able to evaluate expected benefits vs. potential costs before committing to participation

4. **Original Owner Priority:** Original owners should have priority over culturally dominant countries when both exist. Original ownership provides stronger strategic justification for participation than cultural dominance alone, as original owners have direct territorial claims and strategic interests in reclaiming their former territories.

5. **Balance with Traditional Diplomacy:** Secessionist wars should have appropriate costs and constraints that align with traditional war declaration system:
   - Require resource investment (e.g., $game_concept_spy_network$ or ducats)
   - Respect diplomatic constraints (allow refusal, respect $game_concept_truce$s when possible)
   - Provide player control over timing and participation

6. **Exclude Landless Rebels:** Landless rebels ($game_concept_society_of_pops$) provide almost no benefit and should be excluded from secessionist mechanism entirely. They should use standard rebel mechanics instead.

7. **Player War Leader Control:** The culturally dominant country (or original owner) should become $game_concept_war_leader$, not the $vassal$, giving players full control over peace negotiations and $game_concept_war$ duration.

8. **Prevent Exploitation:** The system must prevent systematic exploits:
   - No forced participation during $game_concept_truce$ periods
   - No bypassing $game_concept_coalition$ restrictions
   - No using $game_concept_overlords$ as weapons
   - No forcing subjects into distant wars

### Proposed Solution Framework

**Early Warning and Benefit Preview System:**
- When secessionist rebels reach 80-90% progress, eligible countries are notified
- Notification includes benefit preview:
  - Expected $vassal$ size (territories, population, military strength)
  - Expected $game_concept_war_reparations$ (with AI $secessionists$ taking their cut)
  - Potential $game_concept_war$ costs (manpower, $game_concept_gold$, $game_concept_war_exhaustion$)
  - Risk assessment (defender's $game_concept_allies$, $game_concept_coalition$ members, military strength)
  - Cost-benefit analysis to help players decide if participation is worthwhile

**Optional Participation with Resource Investment:**
- Countries can choose to support secessionists by investing resources (e.g., $game_concept_spy_network$ or ducats)
- This investment grants the right to join the civil war when it breaks out
- Countries can refuse without penalties
- Resource investment aligns secessionist wars with traditional diplomacy costs

**Alternative Solution: Support Rebels Covert Action Requirement (Simpler Implementation)**
- **Key Change:** Secessionist $game_concept_civil_war$ only triggers when the original owner or culturally prominent $game_concept_country$ proactively uses $game_concept_support_rebels$ covert action on the target $game_concept_country$
- **Mechanism:** When a $game_concept_country$ uses $game_concept_support_rebels$ covert action to support secessionist $game_concept_rebels$ in another $game_concept_country$, this creates a "support commitment" that justifies calling that $game_concept_country$ to arms when the $secessionists$ $game_concept_rebellion$ breaks out
- **Strategic Decision:** The supporting $game_concept_country$ makes a strategic decision to support the $game_concept_rebellion$ through the covert action, making their participation justified and voluntary
- **CRITICAL REQUIREMENT: Benefit Preview System:** The game must provide clear benefit preview information when players consider using $game_concept_support_rebels$ covert action for secessionist $game_concept_rebels$:
  - **Potential $secessionists$ population size** - Show the total population of the rebelling culture in the target $game_concept_country$
  - **Number of $game_concept_locations$/$game_concept_provinces$** - Show how many territories the $secessionists$ $game_concept_rebellion$ could potentially control
  - **Expected $vassal$ size and value** - Estimate the size and military strength of the $vassal$ that would be gained if the $game_concept_rebellion$ succeeds
  - **Cost-Benefit Analysis** - Compare expected benefits vs. costs ($game_concept_spy_network$ investment, potential $game_concept_war$ costs, risk of escalation)
  - **Why This Is Essential:** Without clear benefit preview, players cannot justify using $game_concept_support_rebels$ covert action for secessionists when there are other ways to wage $game_concept_wars$ in EU5 (traditional $game_concept_casus_belli$, $game_concept_diplomats$, etc.). The mechanism would become seldomly used if players cannot evaluate whether the potential $vassal$ is worth the investment.
- **Benefits:**
  - Eliminates forced participation - only $game_concept_countries$ that chose to support through covert action are called
  - Requires resource investment ($game_concept_spy_network$ for covert actions)
  - Aligns with traditional diplomacy system (covert actions require $game_concept_spy_network$)
  - Provides player control - players decide whether to support $game_concept_rebels$ before the $game_concept_war$ breaks out
  - Makes participation strategic - players evaluate costs and benefits before using the covert action
  - Enables informed decision-making - benefit preview allows players to justify the investment
- **Implementation:** 
  - Modify secessionist calling mechanism to only call $game_concept_countries$ that have an active $game_concept_support_rebels$ relation with the $game_concept_rebels$ in the target $game_concept_country$
  - Add benefit preview UI/tooltip to $game_concept_support_rebels$ covert action when targeting secessionist $game_concept_rebels$, showing potential $secessionists$ population size, number of territories, and expected $vassal$ value

**Priority System:**
- Original owners receive highest priority (if they still exist)
- Culturally dominant countries receive secondary priority
- Rivals of defender can also participate (at higher cost)
- Subjects are excluded from participation to prevent conflicts with overlords

**War Leadership:**
- If a primary culture country (or original owner) supports the rebellion, they become $game_concept_war_leader$ when the war breaks out
- This gives players full control over peace negotiations and $game_concept_war$ duration
- Prevents AI $vassal$ from controlling the $game_concept_war$

**Exclusion of Landless Rebels:**
- Landless rebels ($game_concept_society_of_pops$) are excluded from secessionist mechanism
- They use standard rebel mechanics instead
- This prevents wars for minimal benefit (314-350 ducats)

**Balanced Escalation:**
- Both sides can coordinate with $game_concept_allies$
- No asymmetric $game_concept_alliance$ calling
- Prevents world $game_concept_war$ escalation through forced participation

---

## Design Considerations

### Game Balance and Strategic Gameplay

The redesigned system must maintain game balance and support strategic gameplay:
- Players should evaluate costs and benefits before committing to $game_concept_wars$
- Participation should be based on strategic considerations (expected $vassal$ value, $game_concept_war$ costs, risk of escalation) rather than forced by automatic cultural ties
- Small cultural minorities in foreign territories should not automatically trigger major $game_concept_wars$ unless players choose to support them based on strategic evaluation

### Strategic Gameplay

The redesigned system must support strategic decision-making:
- Players need information to make informed decisions
- Cost-benefit analysis must be possible before participation
- Players should have control over their country's foreign policy
- Wars should be fought for strategic reasons, not forced by game mechanics

### Balance and Exploitation Prevention

The redesigned system must prevent systematic exploits:
- No forced participation during $game_concept_truce$ periods
- No bypassing $game_concept_coalition$ restrictions
- No using $game_concept_overlords$ as weapons
- No forcing subjects into distant wars
- No exploiting landless rebels to annex major powers

### Integration with Existing Systems

The redesigned system must integrate with existing EU5 mechanics:
- Align costs with traditional diplomacy system
- Respect diplomatic constraints where appropriate
- Maintain unique secessionist flavor while preventing exploitation
- Work with existing $game_concept_alliance$, $game_concept_coalition$, and subject systems

---

## Essential Quick Fixes (Hotfixes)

While a complete redesign is recommended, the following essential quick fixes can address the most critical exploits without requiring a full system redesign:

**Moddability Assessment:**
- **Core secessionist calling mechanism** (automatic same-culture country calling when rebels reach 100%) is **NOT moddable** - it's hardcoded in the engine (C++ code), not in script files
- **`on_civil_war_start` event** (`game/in_game/common/on_action/_hardcoded.txt`) is moddable but fires AFTER the war starts, so it can only remove countries post-facto, not prevent the initial call
- **Subject type properties** (`game/in_game/common/subject_types/secessionists.txt`) are moddable but only affect subject behavior, not the calling mechanism
- **Workarounds possible:** Some fixes can be implemented via `on_civil_war_start` to remove invalid participants, but this is a workaround, not a true prevention
- **Engine changes required:** Most hotfixes require engine code modifications to the secessionist calling mechanism itself

### Hotfix 1: Fix $WAR_LATERALVIEW_ANNEX_REVOLTER$ Button Exploit (Issue 10 - CRITICAL)

**Problem:** The $WAR_LATERALVIEW_ANNEX_REVOLTER$ button can be used to annex war participant $game_concept_countries$ (e.g., $FRA$) instead of only actual $game_concept_revolter$ $game_concept_countries$, allowing major powers to be annexed at 25% cost with no $game_concept_antagonism$.

**Quick Fix:** Add verification check to ensure the target $game_concept_country$ is actually a $game_concept_revolter$ before allowing the Annex Revolter option:
- Verify target is the original $game_concept_revolter$ that started the $game_concept_war$
- OR verify target has revolter flag/variable or is a $game_concept_rebel$ $game_concept_country$ type
- Disable the option for war participant $game_concept_countries$ that are not actual revolters

**Alternative Fix:** If Annex Revolter is used on a non-revolter, apply normal annexation costs (100% instead of 25%) and generate appropriate $game_concept_antagonism$.

**Implementation Location:**
- **File:** Engine code (C++ validation logic) - **NOT moddable via scripts**
- **Workaround (Moddable):** `game/in_game/common/on_action/_hardcoded.txt` (`on_civil_war_start`) can add validation, but cannot prevent button from appearing
- **Code Example (Engine Code):**
```cpp
// In Annex Revolter button validation (engine code)
if (target_country.is_war_participant() && !target_country.is_original_revolter()) {
    // Disable button or apply normal annexation costs
    return false; // or apply_normal_annexation_costs()
}
```

**Impact:** Prevents game-breaking exploit where major powers can be annexed at minimal cost.

### Hotfix 2: Exclude Landless Rebels from Secessionist Mechanism (Issue 12)

**Problem:** Landless $secessionists$ ($game_concept_society_of_pops$) create secessionist $game_concept_wars$ but provide almost no benefit (only 314-350 ducats), and enable the Annex Revolter exploit when eliminated.

**Quick Fix:** Exclude landless $secessionists$ from the secessionist mechanism entirely:
- When secessionist rebels reach 100% progress, check if rebellion would spawn as `country_type = pop` ($game_concept_society_of_pops$)
- If landless, use standard rebel mechanics instead of secessionist $game_concept_civil_war$ mechanism
- Do not create secessionist $game_concept_civil_war$ for landless rebels

**Implementation Location:**
- **File:** Engine code (rebel creation mechanism) - **NOT moddable via scripts**
- **Workaround (Moddable):** `game/in_game/common/on_action/_hardcoded.txt` (`on_civil_war_start`) can detect landless rebels and end war early, but cannot prevent war creation
- **Code Example (Engine Code):**
```cpp
// In secessionist rebel creation logic (engine code)
if (rebel_type == secessionist && would_spawn_as_country_type == pop) {
    // Use standard rebel mechanics instead
    create_standard_rebel_war();
    return; // Skip secessionist civil war creation
}
```

**Impact:** Prevents wars for minimal benefit and eliminates the root cause of the Annex Revolter exploit.

### Hotfix 3: Exclude Subjects from Secessionist Calls (Issue 11)

**Problem:** Subjects (e.g., $vassal$, $secessionists$ subject) that are culturally dominant are forcibly called to support distant rebellions, even thousands of kilometers away.

**Quick Fix:** Exclude all subjects from secessionist calls:
- Add check in secessionist calling mechanism: `is_subject = no`
- This should be consistent with how $vassal$s and $fiefdom$s are already excluded (verified in Issue 2 testing)

**Implementation Location:**
- **File:** Engine code (secessionist calling mechanism) - **NOT moddable via scripts**
- **Workaround (Moddable):** `game/in_game/common/on_action/_hardcoded.txt` (`on_civil_war_start`) can remove subjects from war, but cannot prevent initial call
- **Code Example (Engine Code):**
```cpp
// In secessionist calling logic - filter eligible countries (engine code)
eligible_countries = filter_countries({
    same_culture = yes,
    is_subject = no,  // Add this check
    // ... other conditions
});
```

**Impact:** Prevents subjects from being dragged into distant $game_concept_wars$ and prevents conflicts with $game_concept_overlords$.

### Hotfix 4: Exclude $game_concept_personal_union$ from Secessionist Calls (Issue 2)

**Problem:** $game_concept_personal_union$ $game_concept_junior_partner$s are forcibly called, immediately breaking the $game_concept_personal_union$.

**Quick Fix:** Exclude $game_concept_personal_union$ $game_concept_junior_partner$s from secessionist calls (same as $vassal$s and $fiefdom$s):
- Add check: `NOT = { is_subject_of_type = subject_type:personal_union }` or similar
- Ensure consistency with other subject type exclusions

**Implementation Location:**
- **File:** Engine code (secessionist calling mechanism) - **NOT moddable via scripts**
- **Workaround (Moddable):** `game/in_game/common/on_action/_hardcoded.txt` (`on_civil_war_start`) can remove personal union partners from war, but cannot prevent initial call
- **Code Example (Engine Code):**
```cpp
// In secessionist calling logic - filter eligible countries (engine code)
eligible_countries = filter_countries({
    same_culture = yes,
    is_subject = no,
    NOT = { is_subject_of_type = subject_type:personal_union },  // Add this check
    // ... other conditions
});
```

**Impact:** Prevents $game_concept_personal_union$ from breaking unexpectedly.

### Hotfix 5: Fix $game_concept_truce$ Breaking Penalties (Issue 8)

**Problem:** Forcibly called $game_concept_countries$ with $game_concept_truce$ are incorrectly judged as actively breaking $game_concept_truce$, triggering severe penalties (-50 $game_concept_stability$, +1 $game_concept_war_exhaustion$, +25 $game_concept_antagonism$) even though they had no choice. This can also be exploited to cause harm to opponents by intentionally triggering $secessionists$ $game_concept_rebellions$ during $game_concept_truce$ periods.

**Quick Fix:** Skip same-culture $game_concept_countries$ that have an active $game_concept_truce$ with the $game_concept_defender$ when $secessionists$ choose their supporting $game_concept_country$:
- Add check in secessionist calling mechanism: `NOT = { has_truce_with = $defender$ }`
- If no eligible $game_concept_country$ exists (all have $game_concept_truce$s), the $secessionists$ $game_concept_rebellion$ proceeds without external support
- This prevents the exploit entirely and avoids unfair penalties for countries that had no choice

**Implementation Location:**
- **File:** Engine code (secessionist calling mechanism) - **NOT moddable via scripts**
- **Workaround (Moddable):** `game/in_game/common/on_action/_hardcoded.txt` (`on_civil_war_start`) can remove countries with truces from war, but cannot prevent initial call or truce breaking penalties
- **Code Example (Engine Code):**
```cpp
// In secessionist calling logic - filter eligible countries (engine code)
eligible_countries = filter_countries({
    same_culture = yes,
    is_subject = no,
    NOT = { has_truce_with = defender },  // Add this check
    // ... other conditions
});

// If no eligible country exists, proceed without external support
if (eligible_countries.empty()) {
    create_civil_war_without_external_support();
    return;
}
```

**Impact:** Prevents unfair penalties for countries that had no choice and prevents exploit where players can force opponents into $game_concept_wars$ during $game_concept_truce$ periods.

### Implementation Priority

**Phase 1 (Critical - Immediate):**
- Hotfix 1: Fix Annex Revolter exploit (prevents game-breaking exploit)
- Hotfix 2: Exclude landless rebels (prevents root cause of exploit)

**Phase 2 (High Priority):**
- Hotfix 3: Exclude subjects (prevents distant war participation)
- Hotfix 4: Exclude $game_concept_personal_union$ (prevents union breaking)

**Phase 3 (Medium Priority):**
- Hotfix 5: Fix truce breaking penalties (fairness fix)

**Note:** These hotfixes address the most critical exploits but do not solve the fundamental design issues (forced participation, lack of strategic control, etc.). A complete redesign is still recommended for long-term solution.

### Hotfix 6: Support Rebels Covert Action Requirement (Alternative Solution)

**Problem:** The current system forcibly calls countries into secessionist $game_concept_wars$ without any prior strategic decision or resource investment, removing player control and creating exploits.

**Alternative Solution:** Require $game_concept_support_rebels$ covert action before secessionist $game_concept_civil_war$ can call countries to arms:
- **Key Change:** Secessionist $game_concept_civil_war$ only calls countries to arms if they have proactively used $game_concept_support_rebels$ covert action on the target $game_concept_country$ before the $game_concept_rebellion$ breaks out
- **Mechanism:** When a $game_concept_country$ uses $game_concept_support_rebels$ covert action to support secessionist $game_concept_rebels$ in another $game_concept_country$, this creates a "support commitment" that justifies calling that $game_concept_country$ to arms when the $secessionists$ $game_concept_rebellion$ breaks out
- **Strategic Decision:** The supporting $game_concept_country$ makes a strategic decision to support the $game_concept_rebellion$ through the covert action, making their participation justified and voluntary
- **CRITICAL REQUIREMENT: Benefit Preview System:** The game must provide clear benefit preview information when players consider using $game_concept_support_rebels$ covert action for secessionist $game_concept_rebels$:
  - **Potential $secessionists$ population size** - Show the total population of the rebelling culture in the target $game_concept_country$
  - **Number of $game_concept_locations$/$game_concept_provinces$** - Show how many territories the $secessionists$ $game_concept_rebellion$ could potentially control
  - **Expected $vassal$ size and value** - Estimate the size and military strength of the $vassal$ that would be gained if the $game_concept_rebellion$ succeeds
  - **Cost-Benefit Analysis** - Compare expected benefits vs. costs ($game_concept_spy_network$ investment, potential $game_concept_war$ costs, risk of escalation)
  - **Why This Is Essential:** Without clear benefit preview, players cannot justify using $game_concept_support_rebels$ covert action for secessionists when there are other ways to wage $game_concept_wars$ in EU5 (traditional $game_concept_casus_belli$, $game_concept_diplomats$, etc.). The mechanism would become seldomly used if players cannot evaluate whether the potential $vassal$ is worth the investment compared to alternative war declaration methods.
- **Benefits:**
  - Eliminates forced participation - only $game_concept_countries$ that chose to support through covert action are called
  - Requires resource investment ($game_concept_spy_network$ for covert actions)
  - Aligns with traditional diplomacy system (covert actions require $game_concept_spy_network$)
  - Provides player control - players decide whether to support $game_concept_rebels$ before the $game_concept_war$ breaks out
  - Makes participation strategic - players evaluate costs and benefits before using the covert action
  - Enables informed decision-making - benefit preview allows players to justify the investment
- **Implementation:** 
  - Modify secessionist calling mechanism to only call $game_concept_countries$ that have an active $game_concept_support_rebels$ relation with the $game_concept_rebels$ in the target $game_concept_country$
  - Add benefit preview UI/tooltip to $game_concept_support_rebels$ covert action when targeting secessionist $game_concept_rebels$, showing potential $secessionists$ population size, number of territories, and expected $vassal$ value

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
