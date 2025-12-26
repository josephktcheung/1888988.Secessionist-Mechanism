# Bug Report: $secessionists$ Mechanism

## Metadata

**Title:** $secessionists$ Mechanism - Forced Participation, Original Owner Ignored, World War Escalation, and Exploits

**Issue Type:** Gameplay

**Bug Type:** AI (non-player characters), Other

**Game Version:** 1.0.10

---

## Summary

The $secessionists$ mechanism has critical design flaws that create game-breaking exploits. Same-culture $game_concept_countries$ are forcibly called to $game_concept_war$ without player control, **completely ignoring original ownership** in favor of cultural dominance. The mechanism breaks $game_concept_personal_union$, creates inconsistent $game_concept_war$ leadership logic where $vassal$s lead $secessionists$ $game_concept_civil_war$s (unlike normal $game_concept_war$ declarations where $game_concept_overlords$ lead), prevents player control when $vassal$s lead $game_concept_wars$, incorrectly applies $game_concept_truce$ penalties, and allows the $WAR_LATERALVIEW_ANNEX_REVOLTER$ button to annex entire war participant $game_concept_countries$ (e.g., $FRA$) with no $game_concept_antagonism$. Asymmetric $game_concept_alliance$ calling creates dangerous world $game_concept_war$ escalation where $game_concept_defender$s can coordinate while forcibly called $game_concept_attacker$s cannot. Landless $secessionists$ $game_concept_rebellions$ ($game_concept_society_of_pops$) produce inconsistent outcomes where culturally dominant $game_concept_countries$ gain nothing for fighting, unlike land-based $game_concept_rebellions$ which become $vassal$s. These flaws enable systematic exploits: (1) $vassal$ players can use $game_concept_overlords$ as weapons to attack same-culture $game_concept_countries$ (VERIFIED); (2) Players can attack opponents during $game_concept_truce$ periods by triggering $secessionists$ $game_concept_rebellions$ that force victims to join and receive -50 $game_concept_stability$ penalties.

---

## Description

The $secessionists$ mechanism has twelve critical issues:

### Issue 1: Same-Culture Countries Forced to Join (Most Severe)

When $secessionists$ $game_concept_rebel$s, the system automatically searches for nearby same-culture $game_concept_countries$ and forcibly calls them to $game_concept_war$ on the $game_concept_rebel$ side. Called $game_concept_countries$ cannot refuse. If they have a $game_concept_truce$ with the enemy, they are incorrectly judged as actively breaking the $game_concept_truce$, triggering severe penalties (-50 $game_concept_stability$, +1 $game_concept_war_exhaustion$, +25 $game_concept_antagonism$). The selection mechanism is hardcoded in the engine and cannot be found in game files, making it impossible for players to predict which country will be called.

**CRITICAL FLAW: Original Ownership Completely Ignored**

The system **completely ignores original ownership** when selecting which same-culture $game_concept_country$ to call, prioritizing cultural dominance over historical claims.

*Example from $BYZ$ save:* Turkish $secessionists$ $game_concept_rebellion$ occurs in territories originally owned by $TUR$. $TUR$ still exist and are the original owner. However, the system selects $ERE$ (culturally dominant Turkish $game_concept_country$) instead, completely ignoring $TUR$. $TUR$ have no opportunity to reclaim their former territories, even though they have the strongest historical claim.

### Issue 2: $game_concept_personal_union$ Breaking (Severe - VERIFIED)

When a $game_concept_personal_union$ $game_concept_junior_partner$ is the culturally dominant $game_concept_country$ for a $secessionists$ $game_concept_rebellion$, it is forcibly called to support the $game_concept_rebellion$ against its $game_concept_senior_partner$. This immediately breaks the $game_concept_personal_union$ because `union_of_crowns_pact` has `break_on_war = yes`.

**VERIFICATION STATUS:**
- $game_concept_personal_union$: **VERIFIED AFFECTED** (tested with $FRA$-$SIC$)
- $vassal$: **VERIFIED EXCLUDED** (tested with $FRA$-$SIC$)
- $fiefdom$: **VERIFIED EXCLUDED** (tested with $FRA$-$SIC$)
- Other subject types ($tributary$, $dominion$, $march$, $appanage$, $samanta$, $tusi$, $uc_bey$, etc.): NOT TESTED

This creates an inconsistency where $game_concept_personal_union$ are vulnerable while some other subject types are protected.

**VERIFICATION:** Tested with $FRA$ ($game_concept_senior_partner$) and $SIC$ ($game_concept_junior_partner$). When Sicilian $secessionists$ $game_concept_rebel$s in $FRA$-controlled territory, $SIC$ was forcibly called, immediately breaking the $game_concept_personal_union$. When $SIC$ was a $vassal$ or $fiefdom$ instead, it was correctly excluded.

### Issue 3: $vassal$ $game_concept_war$ Leadership Problem (Severe)

If a player creates a $vassal$ that becomes the $game_concept_defender$ in a $game_concept_civil_war$, the player is forcibly dragged into the $game_concept_war$ but does not become the $game_concept_war_leader$. The AI $vassal$ controls all peace negotiations. The player cannot call $game_concept_allies$, cannot obtain $game_concept_military_access$, and must bear all $game_concept_war$ costs without control.

**CRITICAL INCONSISTENCY: Normal $game_concept_war$ Declaration vs. $secessionists$ $game_concept_civil_war$**

There is a fundamental inconsistency in game logic regarding $vassal$ $game_concept_war$ leadership between normal $game_concept_war$ declarations and $secessionists$ $game_concept_civil_war$s:

**Normal $game_concept_war$ Declaration (using $game_concept_casus_belli$ and $game_concept_diplomats$):**
- When a $game_concept_country$ declares $game_concept_war$ against a $vassal$ (e.g., $BYZ$ declares $game_concept_war$ against $ALB$ where $ALB$ is a $vassal$ of $NAP$), the $game_concept_overlord$ ($NAP$) becomes the $game_concept_war_leader$, NOT the $vassal$ ($ALB$)
- The $game_concept_overlord$ controls all peace negotiations and $game_concept_war$ decisions
- This is the expected behavior for normal $game_concept_war$ declarations

**$secessionists$ $game_concept_civil_war$ (Issue 3):**
- When a $vassal$ becomes the $game_concept_defender$ in a $secessionists$ $game_concept_civil_war$, the $vassal$ becomes the $game_concept_war_leader$, NOT the $game_concept_overlord$
- The $game_concept_overlord$ is forcibly dragged into the $game_concept_war$ but does not control peace negotiations
- The AI $vassal$ controls all peace negotiations, while the $game_concept_overlord$ has no control

**The Inconsistency:**
- **Normal $game_concept_war$ declarations:** $game_concept_overlord$ is $game_concept_war_leader$ when $vassal$ is attacked
- **$secessionists$ $game_concept_civil_war$s:** $vassal$ is $game_concept_war_leader$ when $vassal$ is $game_concept_defender$
- This creates inconsistent game logic where the same relationship ($game_concept_overlord$-$vassal$) produces different $game_concept_war$ leadership outcomes depending on how the $game_concept_war$ starts
- Players cannot predict or understand which party will control the $game_concept_war$ based on the relationship structure alone

**Impact:**
- Creates confusion for players who expect consistent behavior
- Makes it impossible to predict $game_concept_war$ leadership based on subject relationships
- Breaks game logic consistency between different $game_concept_war$ types
- In $secessionists$ $game_concept_civil_war$s, $game_concept_overlords$ lose control they would normally have in regular $game_concept_wars$

### Issue 4: Inconsistency in $game_concept_alliance$ Auto-Call

When the player is the defensive $game_concept_war_leader$, $game_concept_allies$ can be asked to join. But when a $vassal$ is the defensive $game_concept_war_leader$, the player cannot call $game_concept_allies$ to join.

**CRITICAL LIMITATION: Forcibly Called Culturally Dominant $game_concept_country$ ($game_concept_attacker$) Cannot Call $game_concept_allies$ Using $game_concept_favors$**

When a same-culture $game_concept_country$ is forcibly called to support a $secessionists$ $game_concept_rebellion$ as the $game_concept_attacker$, it **cannot call its $game_concept_allies$ using $game_concept_favors$** because it is not the $game_concept_war_leader$ (the $game_concept_rebellion$ country is the $game_concept_war_leader$). 

**VERIFIED: Cannot Integrate $secessionists$ $vassal$ During $game_concept_war$**

It is **NOT possible** to integrate/annex a $secessionists$ $vassal$ during an ongoing $game_concept_war$ where the $secessionists$ $vassal$ is the $game_concept_war_leader$. The game blocks annexation of countries that are in a $game_concept_civil_war$ (see `ANNEX_TARGET_CIVIL_WAR_PENALTY` localization: "is currently going through a [civil_war|e] and can not be annexed until peace has been restored").

**ONLY Scenario Where Culturally Dominant $game_concept_country$ CAN Call $game_concept_allies$:**

**When the landless $secessionists$ $game_concept_rebel$ is eliminated by the $game_concept_defender$**: If the $secessionists$ $game_concept_rebellion$ spawns as a $game_concept_society_of_pops$ (landless with no territory) and its troops are eliminated by the $game_concept_defender$, the $game_concept_rebellion$ is removed from the $game_concept_war$. The war participant culturally dominant $game_concept_country$ then becomes the $game_concept_war_leader$ automatically, and can call $game_concept_allies$ using $game_concept_favors$ (cost: 10 $game_concept_favors$ per $game_concept_ally$).

**Technical Verification:**
- $secessionists$ $vassal$s CANNOT be integrated/annexed during an ongoing $game_concept_war$ where they are the $game_concept_war_leader$ (blocked by civil war restriction)
- The `ask_join_war_for_favors` interaction requires the caller to be the $game_concept_war_leader$ (`attacker_leader = scope:actor` or `defender_leader = scope:actor`)
- Cost: 10 $game_concept_favors$ per $game_concept_ally$ (from `ask_join_war_for_favors_cost`)
- War leadership transfer: When the landless $game_concept_rebel$ ($game_concept_war_leader$) is eliminated, the remaining war participant automatically becomes the new $game_concept_war_leader$

**Impact:**
- The forcibly called culturally dominant $game_concept_country$ is at a severe disadvantage for the entire $game_concept_war$, unable to call $game_concept_allies$ unless the landless $game_concept_rebel$ is eliminated by the $game_concept_defender$
- This creates an asymmetric situation where the $game_concept_defender$ can coordinate with $game_concept_allies$ from day 1, while the culturally dominant $game_concept_attacker$ cannot (see Issue 9)
- The only way for the culturally dominant $game_concept_attacker$ to gain the ability to call $game_concept_allies$ depends on the $game_concept_defender$'s actions (eliminating the landless $game_concept_rebel$), giving the $game_concept_defender$ control over when the attacker can call $game_concept_allies$

### Issue 5: $game_concept_coalition$ Bypass Mechanism (Severe Exploit)

When a forcibly called $game_concept_country$ is in a $game_concept_coalition$, it cannot call $game_concept_coalition$ members to join the $game_concept_war$ because it is the $game_concept_attacker$. This allows the $game_concept_defender$ to bypass $game_concept_coalition$ restrictions and attack $game_concept_coalition$ members individually.

### Issue 6: AI $vassal$ Suboptimal Peace Decisions

AI $vassal$ may choose $game_concept_white_peace$ instead of annexing the $game_concept_revolter$ even when annexation cost is very low, causing players to lose territories they should have gained.

### Issue 7: Overlord Forced to Join

$secessionists$ automatically and forcibly call $game_concept_overlords$ to $game_concept_war$ through `join_offensive_wars_always` and `join_defensive_wars_always` in `secessionists.txt`.

### Issue 8: $game_concept_truce$ Breaking Penalties

Forcibly called $game_concept_countries$ with $game_concept_truce$ are incorrectly judged as actively breaking $game_concept_truce$, triggering severe penalties even though they had no choice.

### Issue 9: World War Escalation - Asymmetric $game_concept_alliance$ Calling (Severe)

A single local $secessionists$ $game_concept_rebellion$ can escalate into a world $game_concept_war$ due to asymmetric $game_concept_alliance$ calling mechanics. This is **worse than historical world $game_concept_wars$** (WW1/WW2) where both sides could coordinate with $game_concept_allies$.

**Current EU5 System - Worse Than Historical World Wars:**

When a $secessionists$ $game_concept_rebellion$ breaks out:
- The $game_concept_defender$ CAN call all its $game_concept_allies$ to join the defensive $game_concept_war$ from day 1
- The forcibly called same-culture $game_concept_country$ ($game_concept_attacker$) CANNOT call its own $game_concept_allies$ using $game_concept_favors$ because it is NOT the $game_concept_war_leader$ (the $game_concept_rebellion$ country is the $game_concept_war_leader$)
- **ONLY exception scenario where the culturally dominant $game_concept_attacker$ CAN call $game_concept_allies$** (see Issue 4 for details):
  - After the landless $secessionists$ $game_concept_rebel$ is eliminated by the $game_concept_defender$ (becomes $game_concept_war_leader$ automatically)
- **VERIFIED: Cannot integrate $secessionists$ $vassal$ during $game_concept_war$** - Annexation is blocked for countries in $game_concept_civil_war$ until peace is restored
- Result: A local $game_concept_rebellion$ escalates into a major $game_concept_war$ where one $game_concept_alliance$ bloc fights alone against another bloc's coordinated response for the entire $game_concept_war$ (unless the $game_concept_defender$ eliminates the landless $game_concept_rebel$, which is rare and gives the $game_concept_defender$ control over when the attacker can call $game_concept_allies$)

*Example Scenario (Late 18th Century - Partition Era):*
- Two major $game_concept_alliance$ blocs: Bloc A ($RUS$, $HAB$, $PRU$) vs Bloc B ($FRA$, $TUR$, $SWE$, $POL$)
- Polish $secessionists$ $game_concept_rebellion$ breaks out in $RUS$-controlled territory
- $POL$ (culturally dominant) is FORCED to join $game_concept_rebellion$'s side ($game_concept_attacker$) - NO CHOICE
- $RUS$ ($game_concept_defender$) calls $HAB$ and $PRU$ → 3 major powers vs $POL$
- $POL$ ($game_concept_attacker$) cannot call $FRA$, $TUR$, or $SWE$ (its own $game_concept_allies$ in Bloc B) → fights alone against 3 major powers

This creates a dangerous escalation mechanism where the $game_concept_defender$ can systematically use $secessionists$ $game_concept_rebellions$ to attack their enemies' $game_concept_alliance$ blocs asymmetrically.

### Issue 10: $WAR_LATERALVIEW_ANNEX_REVOLTER$ Button Misapplied to War Participant $game_concept_countries$ (Severe - GAME-BREAKING)

When a $secessionists$ $game_concept_rebellion$ spawns as a $game_concept_society_of_pops$ (landless $game_concept_country$ with no territory) due to low population size ($game_concept_province$ has <50% of the rebelling culture's population), the $game_concept_rebel$ $game_concept_country$ has no territory. When the $game_concept_rebel$ $game_concept_country$'s troops are wiped out and the $game_concept_rebellion$ is eliminated, the $game_concept_war$ continues and a war participant $game_concept_country$ becomes the $game_concept_war_leader$. The $WAR_LATERALVIEW_ANNEX_REVOLTER$ button, designed to annex the entire $game_concept_revolter$ $game_concept_country$, can then be used to annex the entire war participant $game_concept_country$ (e.g., $FRA$ - 3rd largest $game_concept_country$ at game start) with no $game_concept_antagonism$ generated.

**Exploitation Flow ($ENG$ vs $FRA$ Example):**

1. **$ENG$ spawns a French $game_concept_rebellion$ that becomes a $game_concept_society_of_pops$** ($game_concept_defender$ - owns rebelling territory with French culture)
2. **$FRA$ is forcibly called** as $game_concept_attacker$ (culturally dominant for French culture)
3. **$game_concept_war$ structure:** $ENG$ ($game_concept_defender$/$game_concept_war_leader$) vs $game_concept_rebellion$ ($game_concept_attacker$/$game_concept_war_leader$) + $FRA$ ($game_concept_attacker$/war participant)
4. **$ENG$ wipes out $game_concept_society_of_pops$ troops** ($game_concept_rebellion$ eliminated)
5. **$game_concept_war$ continues** because $FRA$ is still in the $game_concept_war$
6. **$FRA$ becomes $game_concept_war_leader$** (only remaining $game_concept_attacker$)
7. **$ENG$ gains $game_concept_war_score$** through ticking $game_concept_war_score$ (`ticking_war_score = 0.5` per month from `take_country_nationalist` war goal) and by winning battles, sieging enemy $game_concept_locations$, and establishing blockades
8. **$ENG$ reaches 10 $game_concept_war_score$** - The $WAR_LATERALVIEW_ANNEX_REVOLTER$ button becomes enabled when $ENG$ reaches at least 10 $game_concept_war_score$ (`MIN_WARSCORE_TO_DEMAND = 10` from game defines)
9. **$WAR_LATERALVIEW_ANNEX_REVOLTER$ button becomes available** - Once enabled, this peace option can be used on the war participant $game_concept_country$ ($FRA$) instead of the eliminated $game_concept_revolter$
10. **$ENG$ uses $WAR_LATERALVIEW_ANNEX_REVOLTER$ button** to annex the entire $FRA$ $game_concept_country$
11. **$ENG$ annexes entire $FRA$** at effectively 0 cost (base 25% cost from `take_country_nationalist` war goal, but reduced by -95% from `PEACE_COST_MODIFIER_FOR_REVOLT_WAR` modifier, making it effectively 0 cost). This is intended only for annexing the $game_concept_revolter$, not major powers like $FRA$
12. **No $game_concept_antagonism$ is generated** - annexing a major power like $FRA$ should generate massive $game_concept_antagonism$, but the peace option generates none

**Technical Details:**
- When $game_concept_rebellion$ $game_concept_province$ has <50% population of rebelling culture, $game_concept_rebellion$ spawns as `country_type = pop` ($game_concept_society_of_pops$ - landless with no territory)
- When `country_type = pop` troops are eliminated, the $game_concept_rebellion$ is removed from the $game_concept_war$
- The war participant $game_concept_country$ becomes $game_concept_war_leader$ automatically
- **BUG:** The `take_country_nationalist` war goal has `type = take_country`, which allows annexing the entire $game_concept_country$ via $WAR_LATERALVIEW_ANNEX_REVOLTER$ button
- **BUG:** The warscore cost is effectively 0 (base 25% from `conquer_cost = 0.25` in `take_country_nationalist`, but reduced by -95% from `PEACE_COST_MODIFIER_FOR_REVOLT_WAR = -0.95` modifier, making it effectively 0 cost)
- **BUG:** No $game_concept_antagonism$ is generated when annexing the war participant $game_concept_country$
- **Note:** The $game_concept_defender$ can theoretically conquer land at 25% cost if it is the $game_concept_war_leader$ (from `take_country_nationalist` war goal), but AI $vassal$ will never conquer land for its $game_concept_overlord$ when the $vassal$ is the $game_concept_war_leader$ in a $secessionists$ $game_concept_rebellion$ (see Issue 6)

**Impact:**
- **GAME-BREAKING:** Major powers like $FRA$ (3rd largest $game_concept_country$ at game start) can be annexed by smaller $game_concept_countries$ like $ENG$ through this exploit
- **VERIFIED EXAMPLE 1:** $FRA$ was annexed by $ENG$ through this mechanism - $ENG$ spawned a $game_concept_rebellion$ that became a $game_concept_society_of_pops$, $FRA$ was forcibly called, $ENG$ eliminated $game_concept_rebels$, gained 10 $game_concept_war_score$ (minimum required), then used $WAR_LATERALVIEW_ANNEX_REVOLTER$ button to annex entire $FRA$, and no $game_concept_antagonism$ was generated
- **VERIFIED EXAMPLE 2:** $JAL$ was annexed by $TUR$ through this mechanism - $TUR$ had very few Iraqi culture pops in Gumushane location (only 31 Iraqi culture nobles rebelling, <50% of location population), spawning a $game_concept_rebellion$ that became a $game_concept_society_of_pops$ with only 3 soldiers instead of making the location rebel territory. Because it was an Iraqi culture rebellion, $JAL$ (Iraqi culture dominant $game_concept_country$) was forcibly called. $TUR$ eliminated the 3 soldiers ($game_concept_society_of_pops$ disappeared), $JAL$ became $game_concept_war_leader$, $TUR$ gained 10% $game_concept_war_score$ by sieging a few forts, then used $WAR_LATERALVIEW_ANNEX_REVOLTER$ button to annex entire $JAL$. The annexed land was unintegrated territory requiring manual coring

### Issue 11: Subjects Forced to Support Distant Rebellions (Severe)

Any subject (e.g., $vassal$, $secessionists$ subject, or other subject types) that happens to be culturally dominant for the rebelling culture is forcibly called to support distant rebellions of the same culture, even when they are thousands of kilometers away and have no logical connection to the rebellion.

**The Problem:**
- When a $secessionists$ $game_concept_rebellion$ breaks out, the system searches for culturally dominant $game_concept_countries$ to call
- The system does NOT check if the $game_concept_country$ is a subject, its size, or distance to the $game_concept_rebellion$
- Any $game_concept_country$ with the same culture as the rebelling population can be called, regardless of whether it is a subject
- Result: Small subjects (e.g., $vassal$s, $secessionists$ subjects) get forcibly called to support $game_concept_rebellions$ thousands of kilometers away

**VERIFIED EXAMPLE:**
- Player ($YUA$ / Ming) has a small Mongolian culture subject (e.g., $vassal$ or $secessionists$ subject) in Mongolia region
- A Mongolian $secessionists$ $game_concept_rebellion$ breaks out in Georgia (6000+ km away)
- The small subject is FORCED to join the $game_concept_rebellion$'s side ($game_concept_attacker$) - NO CHOICE
- Player's subject gets dragged into a $game_concept_war$ far away, player has no control
- Player must search through war participant $game_concept_countries$ to find out why their subject is at $game_concept_war$
- The subject receives penalties ($game_concept_truce$ breaking, $game_concept_war_exhaustion$, $game_concept_antagonism$) even though it had no choice
- The $game_concept_overlord$ ($YUA$) is also dragged into the $game_concept_war$ (Issue 7), compounding the problem

**Why This Is Problematic:**
- **No logical connection:** A small subject in Mongolia has no reason to support a $game_concept_rebellion$ in Georgia
- **No player control:** Player cannot prevent their subject from being called
- **Hidden cause:** Player must investigate to find out why their subject is at $game_concept_war$
- **Penalties without choice:** Subject receives severe penalties for actions it did not choose
- **$game_concept_overlord$ dragged in:** $game_concept_overlord$ is also forced to join (Issue 7), creating a cascade of forced participation
- **Exploitable:** $game_concept_defender$s can use this to force small subjects and their $game_concept_overlords$ into $game_concept_wars$

**Technical Details:**
- The hardcoded mechanism in `on_civil_war_start` does not check `is_subject = yes` or subject type
- The system only checks cultural dominance, ignoring subject status, distance, and size
- Subjects are treated the same as independent $game_concept_countries$ for the purpose of being called

**Impact:**
- Small subjects get dragged into distant $game_concept_wars$ they have no interest in
- Players lose control over their subjects' foreign policy
- Creates confusion and frustration (player must investigate why subject is at $game_concept_war$)
- Enables exploits where $game_concept_defender$s can force small subjects and $game_concept_overlords$ into $game_concept_wars$

### Issue 12: Inconsistent Behavior of Land-Based vs. Landless $secessionists$ $game_concept_rebellions$ (Severe)

When a $secessionists$ $game_concept_rebellion$ spawns, its behavior differs dramatically based on whether it is land-based (owns territory) or landless ($game_concept_society_of_pops$ with no territory). This creates inconsistent outcomes that break the intended $secessionists$ mechanism.

**The Problem:**

**Land-Based $secessionists$ $game_concept_rebellions$ (>50% cultural population):**
- When the $game_concept_rebellion$ breaks out, it controls territory
- When the $game_concept_rebellion$ wins independence, it becomes a $secessionists$ $vassal$ of the culturally dominant $game_concept_country$
- The culturally dominant $game_concept_country$ gains a new $vassal$ as a reward for supporting the $game_concept_rebellion$

**Landless $secessionists$ $game_concept_rebellions$ (<50% cultural population - $game_concept_society_of_pops$):**
- When the $game_concept_rebellion$ breaks out, it has no territory (only pops)
- When the $game_concept_rebellion$ wins independence, it has **TWO INCONSISTENT OUTCOMES:**
  1. **Becomes an INDEPENDENT $game_concept_country$**, NOT a $vassal$ of the culturally dominant $game_concept_country$
  2. **Becomes a landless $secessionists$ $vassal$** with no territory (only 1 unit of cavalry with 5 people) - effectively useless
- The culturally dominant $game_concept_country$ gains **ALMOST NOTHING** for fighting the $game_concept_war$ - only minimal $game_concept_war_reparations$ (314.82 to 350.72 ducats) even at 100% $game_concept_war_score$
- The AI culturally dominant $game_concept_country$ may not take any land even at 100% $game_concept_war_score$, only accepting $game_concept_war_reparations$
- **CRITICAL:** The entire secessionist concept falls apart for landless rebels - culturally dominant $game_concept_countries$ fight major $game_concept_wars$ between major powers (e.g., $FRA$ vs $CAS$) for essentially nothing

**VERIFIED EXAMPLES:**

**Example 1 - Landless Rebel ($ENG$ vs $FRA$):**
- $ENG$ spawns a French $game_concept_rebellion$ that becomes a $game_concept_society_of_pops$ (<50% French culture population)
- $FRA$ (culturally dominant) is forcibly called to support the $game_concept_rebellion$
- $FRA$ fights the $game_concept_war$ and achieves 100% $game_concept_war_score$
- The landless $game_concept_rebellion$ becomes an **INDEPENDENT** $game_concept_country$ (not a $vassal$ of $FRA$)
- $FRA$ receives only a portion of $game_concept_war_reparations$ (e.g., 238.09 ducats) despite fighting the entire $game_concept_war$
- $FRA$ gains **NO TERRITORY** and **NO $vassal$** despite winning the $game_concept_war$ with 100% $game_concept_war_score$

**Example 2 - AI Not Taking Land:**
- In some cases, the AI culturally dominant $game_concept_country$ does not take any land even at 100% $game_concept_war_score$
- The AI may only accept $game_concept_war_reparations$ or $game_concept_white_peace$, leaving the $game_concept_defender$ with all territory intact
- This is inconsistent with land-based $game_concept_rebellions$ where the culturally dominant $game_concept_country$ gains a $vassal$

**Example 3 - Landless Rebel Behavior ($FRA$ vs $CAS$ - VERIFIED):**
- $CAS$ conquers Loudun (French culture territory)
- French $secessionists$ $game_concept_rebellion$ spawns as $game_concept_society_of_pops$ (<50% French culture population)
- $FRA$ (culturally dominant) is forcibly called to support the $game_concept_rebellion$
- $FRA$ fights $CAS$ (two major powers in Europe) and achieves 100% $game_concept_war_score$
- **Outcome 1:** The landless rebel (AAA00) becomes a **landless $secessionists$ $vassal$** of $FRA$ with no territory, only 1 unit of cavalry (5 people) - effectively useless
- **Outcome 2:** The landless rebel becomes a **settled country** but **NOT a $secessionists$ $vassal$** of $FRA$ - completely independent
- $FRA$ gains only **314.82 to 350.72 ducats** ($game_concept_war_reparations$) despite fighting an entire major $game_concept_war$ with 100% $game_concept_war_score$
- $FRA$ gains **NO TERRITORY** and either **NO USEFUL $vassal$** (landless vassal with 5 people) or **NO $vassal$ AT ALL** (independent country)
- **Summary:** Two major powers ($FRA$ and $CAS$) fight a $game_concept_war$ for either: (1) a landless $secessionists$ $vassal$ with 1 unit of cavalry (5 people), (2) an independent settled country, or (3) 314.82 to 350.72 ducats. This demonstrates that the secessionist concept completely falls apart for landless rebels - there is almost no benefit to fighting in a secessionist $game_concept_war$ for a landless secessionist because the supposedly overlord $FRA$ gains almost nothing, and the landless rebel does NOT become $FRA$'s useful $vassal$ if it becomes settled

**Why This Is Problematic:**
- **Inconsistent outcomes:** Same mechanism produces different results based on population size
- **No meaningful reward for culturally dominant $game_concept_country$:** Landless $game_concept_rebellions$ provide almost no benefit to the culturally dominant $game_concept_country$ that is forced to fight - only minimal $game_concept_war_reparations$ (314-350 ducats) even at 100% $game_concept_war_score$
- **Two problematic outcomes:** Landless rebels either become independent (no $vassal$ reward) or become useless landless $vassal$s with no territory (only 5 people in 1 unit)
- **Major $game_concept_war$ for nothing:** Two major powers fight a $game_concept_war$ for essentially nothing - the culturally dominant $game_concept_country$ gains almost no benefit despite winning with 100% $game_concept_war_score$
- **AI suboptimal behavior:** AI may not take land even when it should, wasting $game_concept_war_score$
- **Breaks intended design:** The $secessionists$ mechanism is supposed to reward culturally dominant $game_concept_countries$ with useful $vassal$s, but landless $game_concept_rebellions$ completely break this design
- **Unfair to players:** Players who are forcibly called into landless $game_concept_rebellion$ $game_concept_wars$ gain almost nothing for their effort
- **Secessionist concept falls apart:** The entire secessionist concept becomes meaningless for landless rebels, as they can be independent or become useless landless $vassal$s

**Technical Details:**
- When $game_concept_province$ has <50% population of rebelling culture, $game_concept_rebellion$ spawns as `country_type = pop` ($game_concept_society_of_pops$ - landless with no territory)
- When `country_type = pop` wins independence, it has inconsistent outcomes:
  1. Becomes an independent $game_concept_country$ (not a $vassal$)
  2. Becomes a landless $secessionists$ $vassal$ with no territory (only pops/units, effectively useless)
- The `secessionists.txt` subject type definition does not have `ai_wants_to_be_overlord`, which may explain why AI doesn't try to make landless rebels into useful $vassal$s
- The `PEACE_MAKE_SUBJECT_REVOLT_WAR = 0` define in `00_defines.txt` indicates AI has no interest in making subjects in revolt $game_concept_wars$, which explains suboptimal peace decisions
- Even when landless rebels become $vassal$s, they have no territory and minimal military (e.g., 1 unit of cavalry with 5 people), making them effectively useless

**Impact:**
- Culturally dominant $game_concept_countries$ gain almost nothing from supporting landless $game_concept_rebellions$ - only minimal $game_concept_war_reparations$ (314-350 ducats) even at 100% $game_concept_war_score$
- Major powers fight major $game_concept_wars$ for essentially nothing (e.g., $FRA$ vs $CAS$ for a landless $vassal$ with 5 people or an independent country)
- Players are forced to fight $game_concept_wars$ with almost no reward
- AI makes suboptimal peace decisions, not taking land even at 100% $game_concept_war_score$
- Breaks the intended design of the $secessionists$ mechanism where culturally dominant $game_concept_countries$ should gain useful $vassal$s
- The entire secessionist concept becomes meaningless for landless rebels, as they either become independent or useless landless $vassal$s

---

## Exploitation Scenarios

### EXPLOIT 1: $vassal$ PLAYER USING OVERLORD AS WEAPON (VERIFIED)

Players who choose to play as $vassal$ can systematically use their $game_concept_overlords$ as weapons to attack and conquer territories from same-culture $game_concept_countries$, then eventually declare independence.

**Exploitation Strategy** (VERIFIED - $BYZ$ Case):

1. Player chooses to play as a $vassal$ (e.g., $BYZ$ $vassal$ Hudavendigar)
2. Player intentionally increases $secessionists$ $game_concept_rebel$ monthly progress through various means (not coring territories, low $game_concept_stability$, low legitimacy, maximizing taxes, reducing control, etc.)
3. When $secessionists$ $game_concept_rebellion$ breaks out:
   - The $game_concept_overlord$ (e.g., $BYZ$) is AUTOMATICALLY dragged into the $game_concept_war$ on the defensive side (Issue 7)
   - The culturally dominant $game_concept_country$ (e.g., $ERE$ for Turkish culture) is FORCED to join the $game_concept_rebellion$'s side ($game_concept_attacker$) (Issue 1)
   - The player's $vassal$ becomes the defensive $game_concept_war_leader$, giving the player full control over peace negotiations
4. Player advantages as $vassal$ $game_concept_war$ leader:
   - Can react to $game_concept_rebels$ on day 1, sending regulars to occupy $game_concept_rebel$ $game_concept_provinces$ immediately
   - $game_concept_overlord$ automatically sends troops to help
   - $game_concept_war$ score to annex $game_concept_rebels$ is very low (e.g., 2 points), easily achievable
   - Any extra $game_concept_war$ score can be used to demand territories/money from the forcibly called war participant $game_concept_country$
5. Player controls $game_concept_war$ duration based on military situation
6. Player gains all benefits from the $game_concept_war$: territories and ducats from the forcibly called war participant $game_concept_country$, uses $game_concept_overlord$'s military strength without cost, builds up power for eventual independence
7. Once strong enough, player declares independence and defeats the weakened $game_concept_overlord$

**VERIFICATION:** Tested in $BYZ$ game. Player played as $BYZ$ $vassal$ Hudavendigar. Turkish $secessionists$ $game_concept_rebellion$ broke out, $ERE$ (culturally dominant Turkish $game_concept_country$) was forcibly called. $BYZ$ ($game_concept_overlord$) was automatically dragged into the $game_concept_war$. Player (as $vassal$ $game_concept_war$ leader) was able to sue for peace and conquer $ERE$'s territories.

**Impact:**
- $vassal$ players can systematically exploit their $game_concept_overlords$ to fight $game_concept_wars$ for them
- No need to fabricate claims, create casus belli, or manage $game_concept_truce$ timers
- Can build up power and resources using $game_concept_overlord$'s military strength
- Eventually declare independence when strong enough
- Makes $vassal$ gameplay exploitable and breaks intended gameplay balance
- In multiplayer, this allows $vassal$ players to systematically weaken their $game_concept_overlord$ players

### EXPLOIT 2: DOUBLE WAR ATTACK

The $secessionists$ mechanism creates a severe exploit that allows knowledgeable players to attack opponents (AI or human players) during a $game_concept_truce$ period by intentionally creating that $game_concept_truce$ period, then triggering $secessionists$ $game_concept_rebellion$ which forces the victim to join and receive devastating penalties.

**Exploitation Strategy:**

1. Player A (exploiter) acquires territories with the same primary culture as Player B (victim - AI or human player) through War 1 or from game start/previous conquests
2. Player A intentionally allows or encourages $secessionists$ $game_concept_rebellion$ to develop ($game_concept_rebel$ progress approaching 100%)
3. Player A wages War 2 against Player B when the $secessionists$ $game_concept_rebellion$ is about to break out ($game_concept_rebel$ progress near 100%)
4. Player A makes $game_concept_white_peace$ or gets a small amount of ransom, ending War 2 quickly, creating a new $game_concept_truce$ period
5. Player A forms $game_concept_alliance$ with other $game_concept_countries$/players during this new $game_concept_truce$ period
6. When $secessionists$ $game_concept_rebellion$ breaks out:
   - Player B (culturally dominant) is FORCED to join the $game_concept_rebellion$'s side ($game_concept_attacker$)
   - Player B receives -50 $game_concept_stability$ penalty for "breaking $game_concept_truce$" (even though forced)
   - Player B cannot call $game_concept_allies$ or $game_concept_coalition$ members because it is the $game_concept_attacker$
   - Player A ($game_concept_defender$) CAN call all $game_concept_allies$ to join the defensive $game_concept_war$
   - Result: Player A attacks Player B during the $game_concept_truce$ period with Player B suffering severe penalties and unable to defend effectively

**Impact:**
- This exploit completely breaks game balance
- Knowledgeable players can systematically destroy opponents (AI or human) by exploiting this mechanism
- Victim $game_concept_countries$ have no counterplay - they cannot prevent the $secessionists$ $game_concept_rebellion$, cannot refuse the call, and cannot call their own $game_concept_allies$
- The -50 $game_concept_stability$ penalty can cause $game_concept_country$ collapse, making the victim's game unplayable
- This creates an unfair advantage that makes games uncompetitive

**Example Scenario:**
- Player ($FRA$) acquires Castilian-culture territories (either through War 1 against $CAS$ if needed, or already owns them)
- Player allows Castilian $secessionists$ $game_concept_rebellion$ to develop ($game_concept_rebel$ progress approaching 100%)
- Player declares War 2 against Opponent ($CAS$ - AI or human player) when Castilian $game_concept_rebellion$ is at 95% progress
- Player quickly white peaces War 2, creating a new $game_concept_truce$ period
- Player forms $game_concept_alliance$ with $ARA$ and $PAP$ during this new $game_concept_truce$ period
- Castilian $secessionists$ $game_concept_rebellion$ breaks out
- Opponent ($CAS$) is forced to join $game_concept_rebellion$ side, receives -50 $game_concept_stability$, cannot call $game_concept_allies$
- Player calls $ARA$ and $PAP$, attacks Opponent again with overwhelming advantage
- Opponent's $game_concept_country$ collapses due to $game_concept_stability$ penalty and cannot effectively defend

---

## Severity

These issues are critical because:
- Players have absolutely no control over being dragged into $game_concept_wars$
- **Original owners are completely ignored** - The system ignores original ownership and only looks at cultural dominance (e.g., $TUR$ ignored in favor of $ERE$ in $BYZ$ case)
- **Inconsistent $game_concept_war$ leadership logic** - In normal $game_concept_war$ declarations, $game_concept_overlords$ become $game_concept_war_leader$ when their $vassal$s are attacked, but in $secessionists$ $game_concept_civil_war$s, $vassal$s become $game_concept_war_leader$ instead. This creates unpredictable and inconsistent game logic that breaks player expectations
- $game_concept_personal_union$ break unexpectedly due to forced participation
- Players receive severe penalties (-50 $game_concept_stability$) for actions they did not choose
- **$game_concept_rebellions$ that spawn as $game_concept_society_of_pops$ cause warscore costs to be applied to wrong target** - When $game_concept_society_of_pops$ troops are wiped out and $game_concept_rebellion$ is eliminated, the $game_concept_war$ continues with the war participant $game_concept_country$ as $game_concept_war_leader$. The $WAR_LATERALVIEW_ANNEX_REVOLTER$ button (intended only for annexing $game_concept_rebels$) can then be used to annex the entire war participant $game_concept_country$ at effectively 0 cost (base 25% cost from `take_country_nationalist` war goal, but reduced by -95% from `PEACE_COST_MODIFIER_FOR_REVOLT_WAR` modifier, making it effectively 0 cost). This allows major powers like $FRA$ (3rd largest $game_concept_country$ at game start) to be annexed by smaller $game_concept_countries$ like $ENG$. **Note:** The $game_concept_defender$ can theoretically conquer land at 25% cost if it is the $game_concept_war_leader$, but AI $vassal$ will never conquer land for its $game_concept_overlord$ when the $vassal$ is the $game_concept_war_leader$ in a $secessionists$ $game_concept_rebellion$ (see Issue 6)
- The mechanism can be systematically exploited by knowledgeable players in multiple ways (see EXPLOITATION SCENARIOS section)
- The mechanism violates historical rationality
- Players lose territories and strategic control due to AI decisions
- Inconsistency: $game_concept_personal_union$ are treated differently from other subject types ($vassal$, $fiefdom$ are protected, $game_concept_personal_union$ are not)
- **EXPLOIT 1:** $vassal$ players can use their $game_concept_overlords$ as weapons to attack same-culture $game_concept_countries$ and eventually declare independence, breaking intended gameplay balance
- **EXPLOIT 2:** Players can attack opponents (AI or human) during a $game_concept_truce$ period by intentionally creating a $game_concept_truce$ period through War 2, then triggering $secessionists$ $game_concept_rebellion$ which forces the victim to join and receive -50 $game_concept_stability$ penalties, preventing the victim from calling $game_concept_allies$. This completely breaks game balance in both single-player and multiplayer contexts
- **WORLD $game_concept_war$ ESCALATION:** A single local $game_concept_rebellion$ can escalate into a world $game_concept_war$ due to asymmetric $game_concept_alliance$ calling. The $game_concept_defender$ can call all $game_concept_allies$, while forcibly called $game_concept_attacker$ cannot call its own $game_concept_allies$, creating a dangerous escalation mechanism
- **SMALL SUBJECTS DRAGGED INTO DISTANT $game_concept_wars$:** Any subject (e.g., $vassal$, $secessionists$ subject) that happens to be culturally dominant is forcibly called to support distant rebellions of the same culture, even when thousands of kilometers away. Players lose control over their subjects' foreign policy and must investigate to find out why their subjects are at $game_concept_war$ (VERIFIED: $YUA$ case - small Mongolian subject in Mongolia region forced to support rebellion in Georgia 6000+ km away)
- **INCONSISTENT BEHAVIOR OF LAND-BASED VS. LANDLESS $secessionists$:** Land-based $secessionists$ $game_concept_rebellions$ (>50% cultural population) become $vassal$s of the culturally dominant $game_concept_country$ when they win, while landless $secessionists$ $game_concept_rebellions$ (<50% cultural population - $game_concept_society_of_pops$) have inconsistent outcomes: either become independent $game_concept_countries$ or useless landless $vassal$s (e.g., 1 unit of cavalry with 5 people). This creates inconsistent outcomes where culturally dominant $game_concept_countries$ gain almost nothing from fighting landless $game_concept_rebellion$ $game_concept_wars$ (only 314-350 ducats even at 100% $game_concept_war_score$), and the entire secessionist concept falls apart for landless rebels. Major powers fight major $game_concept_wars$ for essentially nothing (VERIFIED: $FRA$ vs $CAS$ case - $FRA$ fought entire $game_concept_war$ with 100% $game_concept_war_score$ and gained only 314.82 to 350.72 ducats, no territory or useful $vassal$; landless rebel either became independent or useless landless $vassal$ with 5 people)

---

## Recommended Priority

Issues 1, 2, and 5 should be prioritized as they have the most severe impact on gameplay experience and players have absolutely no control over them. Issue 2 ($game_concept_personal_union$ Breaking) is particularly severe because it creates an inconsistency where one subject type ($game_concept_personal_union$) is vulnerable while some others ($vassal$, $fiefdom$) are protected. The exploitation scenarios (Exploit 1 and Exploit 2) make this even more critical, as they completely break game balance in both single-player and multiplayer contexts, making games uncompetitive and unplayable for victims. Further testing is needed to determine if other subject types ($tributary$, $dominion$, $march$, $appanage$, etc.) are also affected.

---

## Steps to Reproduce

Multiple game saves have been prepared to demonstrate these issues. Save file IDs are provided for each set.

### CASE 1: $BYZ$ SAVES (Demonstrates Issues 1, 3, 4, 6, and 7)

Save File ID: **#f40b1288**

Load the $BYZ$ save files. The saves are set up with:
- $BYZ$ has conquered most $TUR$ territories
- $BYZ$ released $vassal$ Hudavendigar and ceded all $TUR$ territories to it
- $BYZ$ has an $game_concept_alliance$ with $TRE$
- Turkish $secessionists$ $game_concept_rebellion$ is ready to spawn (or has already spawned)
- **CRITICAL:** The rebelling territories were originally owned by $TUR$. $TUR$ still exist and are the **original owner** of these territories.
- $ERE$ is the culturally dominant $game_concept_country$ for Turkish culture (this is why it gets selected for support)

**Steps to observe Issue 1** (Same-Culture Countries Forced to Join):

1. Load the $BYZ$ save 1337.7.2 - Byzantium HUD Revolt 100%
2. Turkish rebellion progress in Hudavendigar is set to 100% using console command
3. When $secessionists$ $game_concept_rebel$s and declare $game_concept_civil_war$, observe that $ERE$ (the culturally dominant $game_concept_country$ for Turkish culture) is automatically and forcibly called to $game_concept_war$ on the $game_concept_rebel$ side (save: 1337.8.1 - Byzantium HUD ERE War)
4. **CRITICAL OBSERVATION:** Observe that $TUR$ (the original owner of the rebelling territories) are **COMPLETELY IGNORED** by the system. $TUR$ are not selected, even though they have the strongest historical claim to these territories.
5. Observe that $ERE$ cannot refuse this call - it is forced into the $game_concept_war$ on day 1
6. If $ERE$ has a $game_concept_truce$ with $BYZ$, observe that it receives -50 $game_concept_stability$ penalty for "breaking $game_concept_truce$" even though it had no choice
7. **Note:** $ERE$ is selected because it is the culturally dominant $game_concept_country$ for Turkish culture, but players cannot predict this selection as the mechanism is hardcoded in the engine. **The system completely ignores original ownership** - $TUR$ (original owner) are not selected because they are not the culturally dominant Turkish $game_concept_country$.

**Steps to observe Issue 3** ($vassal$ $game_concept_war$ Leadership):

1. Load the $BYZ$ save 1337.8.1 - Byzantium HUD ERE War
2. When the $game_concept_civil_war$ starts, observe that $BYZ$ is forcibly dragged into the $game_concept_war$ on day 1
3. Check the $game_concept_war$ panel - observe that $vassal$ Hudavendigar is the defensive $game_concept_war_leader$, NOT $BYZ$
4. Try to call ally $TRE$ to $game_concept_war$ - observe that it's impossible because $BYZ$ is not the $game_concept_war_leader$
5. Try to request military access from $AHI$ or $CND$ - observe that it's impossible because of low opinion or `block_when_at_war = yes`
6. Observe that $BYZ$ cannot control peace negotiations - only the AI $vassal$ Hudavendigar can make peace
7. Observe that $BYZ$ cannot obtain military access to attack $ERE$, creating a stalemate situation

**Steps to observe Issue 4** (Inconsistency in $game_concept_alliance$ Auto-Call):

1. Load the $BYZ$ save 1337.8.1 - Byzantium HUD ERE War
2. When $game_concept_civil_war$ starts, observe that $TRE$ ($BYZ$'s ally) does NOT automatically join the $game_concept_war$
3. Try to manually call $TRE$ - observe that it's impossible because $BYZ$ is not the $game_concept_war_leader$
4. This demonstrates the inconsistency: if $BYZ$ were the $game_concept_war_leader$, $BYZ$ can ask $TRE$ to join, but since the $vassal$ is the leader, $BYZ$ cannot call its ally

**Steps to observe Issue 6** (AI $vassal$ Suboptimal Decisions):

1. Load the $BYZ$ save 1339.2.1 - Byzantium HUD ERE War Stuck
2. Occupy all rebel territories and transfer control to $vassal$ Hudavendigar
3. Achieve 45%+ warscore
4. Wait for AI $vassal$ Hudavendigar to make peace
5. Observe that AI $vassal$ chooses $game_concept_white_peace$ instead of annexing the $secessionists$, even though annexation only costs 2 $game_concept_peace_offer$ points
6. Observe that $BYZ$ loses 7 $game_concept_locations$ which become new $secessionists$, becoming a $vassal$ of $ERE$

**Steps to observe Issue 7** (Overlord Forced to Join):

1. Load any $BYZ$ save during the $game_concept_civil_war$
2. Observe that $BYZ$ ($game_concept_overlord$ of Hudavendigar) is automatically dragged into the $game_concept_war$
3. Observe that this happens through `join_defensive_wars_always` in `secessionists.txt`

### CASE 2: $FRA$ SAVES - $game_concept_personal_union$ BREAKING (Demonstrates Issue 2)

Save File ID: **#51d1fcfd**

Load the $FRA$ $game_concept_personal_union$ save files. The saves demonstrate:
- $FRA$ has formed a $game_concept_personal_union$ with $SIC$ ($FRA$ as $game_concept_senior_partner$)
- $FRA$ has conquered Sicilian-culture territory from $NAP$
- Sicilian $secessionists$ $game_concept_rebellion$ is ready to spawn

**Steps to observe Issue 2** ($game_concept_personal_union$ Breaking):

1. Load save 1337.5.1.2 - $FRA$ $game_concept_personal_union$ $SIC$, No CB $NAP$
   - $FRA$ has formed $game_concept_personal_union$ with $SIC$ using console command `create_pu SIC`
   - $FRA$ has declared No CB $game_concept_war$ against $NAP$
2. Load save 1337.6.1 - $FRA$ Conquer $NAP$
   - $FRA$ has conquered Calabria Ultra Province (Sicilian culture)
3. Load save 1337.7.1 - $FRA$ Sicilian Rebel 100%
   - Sicilian rebellion progress set to 100% using console command
4. Load save 1337.8.1 - $FRA$ $game_concept_personal_union$ Broken
   - $secessionists$ $game_concept_rebellion$ breaks out
   - Observe that $SIC$ (culturally dominant for Sicilian culture) is forcibly called to support the rebellion
   - Observe that the $game_concept_personal_union$ between $FRA$ and $SIC$ immediately breaks on day 1
   - Observe that $SIC$ is now at $game_concept_war$ with $FRA$ (its former $game_concept_senior_partner$)

**VERIFICATION OF PROTECTION FOR OTHER SUBJECT TYPES:**

The following saves demonstrate that $vassal$ and $fiefdom$ are correctly excluded from $secessionists$ calls. However, other subject types have not been tested and may also be affected:

**EU5 Subject Types (Status):**
- $game_concept_personal_union$: **VERIFIED AFFECTED**
- $vassal$: **VERIFIED EXCLUDED**
- $fiefdom$: **VERIFIED EXCLUDED**
- $tributary$, $dominion$, $march$, $appanage$, $samanta$, $tusi$, $uc_bey$, etc.: NOT TESTED

### CASE 3: $FRA$ SAVES - $fiefdom$ (No Call)

Save File ID: **#1177c64a**

1. Load save 1337.4.1 - $FRA$ ANNEX $SIC$, Create $fiefdom$ subject
   - $FRA$ annexed $SIC$ using console command
   - Created a new $SIC$ $fiefdom$ subject
2. Load save 1337.6.1 - $FRA$ Conquer $NAP$
   - $FRA$ conquered Calabria Ultra Province (Sicilian culture)
3. Load save 1337.7.1 - $FRA$ Sicilian Rebel 100%
   - Sicilian rebellion progress set to 100% using console command
4. Load save 1337.8.1 - $FRA$ Sicilian $secessionists$ NOT calling $SIC$
   - $secessionists$ $game_concept_rebellion$ breaks out
   - Observe that $SIC$ ($fiefdom$) is NOT called to support the $game_concept_rebellion$
   - This demonstrates that $fiefdom$ are correctly excluded

### CASE 4: $FRA$ SAVES - $vassal$ (No Call)

Save File ID: **#dae84461**

1. Load save 1337.4.1 - $FRA$ ANNEX $SIC$, Create $vassal$ subject
   - $FRA$ annexed $SIC$ using console command
   - Created a new $SIC$ $vassal$ subject
2. Load save 1337.6.10 - $FRA$ Conquer $NAP$
   - $FRA$ conquered Calabria Ultra Province (Sicilian culture)
3. Load save 1337.7.1 - $FRA$ Sicilian Rebel 100%
   - Sicilian rebellion progress set to 100% using console command
4. Load save 1337.8.1 - $FRA$ $secessionists$ NOT calling $SIC$ $vassal$
   - $secessionists$ $game_concept_rebellion$ breaks out
   - Observe that $SIC$ ($vassal$) is NOT called to support the $game_concept_rebellion$
   - This demonstrates that $vassal$ are correctly excluded

**CONCLUSION:** $game_concept_personal_union$ are VERIFIED to receive $secessionists$ calls, while $vassal$ and $fiefdom$ are VERIFIED to be excluded. This creates an inconsistency where $game_concept_personal_union$ break while some other subject types are protected. Other subject types have not been tested and may also be affected.

### CASE 5: $FRA$ SAVES - $game_concept_coalition$ BYPASS AND $game_concept_truce$ BREAKING (Demonstrates Issues 1, 5, and 8)

Save File ID: **#75e13d1**

Load the $FRA$ save files. The saves are set up with:
- $FRA$ has formed $game_concept_alliance$ with $ARA$ and $PAP$
- $FRA$ has defeated $CAS$ in a No CB $game_concept_war$ and taken Castilian territories
- Castilian $secessionists$ $game_concept_rebellion$ is ready to spawn (or has already spawned)
- $CAS$ may be in a $game_concept_coalition$ against $FRA$ (or will join after $game_concept_truce$ ends)

**Steps to observe Issue 1** (Same-Culture Countries Forced to Join):

1. Load the $FRA$ save 1338.4.1 - $FRA$ $game_concept_truce$ with $CAS$, Add Castilian Rebel to 100%
2. Console command is used to force Castilian rebellion progress to 100%
3. When $secessionists$ $game_concept_rebel$s and declare $game_concept_civil_war$, observe that $CAS$ is automatically and forcibly called to $game_concept_war$ on the $game_concept_rebel$ side
4. Observe that $CAS$ cannot refuse this call - it is forced into the $game_concept_war$

**Steps to observe Issue 8** ($game_concept_truce$ Breaking):

1. Load the $FRA$ save 1338.4.1 - $FRA$ $game_concept_truce$ with $CAS$, Add Castilian Rebel to 100%
2. If $CAS$ and $FRA$ have a $game_concept_truce$ when the rebellion $game_concept_war$ begins, observe that $CAS$ receives -50 $game_concept_stability$ penalty for "breaking $game_concept_truce$" in the next month when rebellion breaks out
3. Observe that $CAS$ is judged as actively breaking the $game_concept_truce$ even though it was forcibly called to $game_concept_war$ and had no choice

**Steps to observe Issue 5** ($game_concept_coalition$ Bypass):

1. Load the $FRA$ save 1346.8.2 - $FRA$ $CAS$ Joint $game_concept_coalition$, Add Castilian Rebel to 100%
2. $CAS$ has joined a $game_concept_coalition$ against $FRA$
3. When $secessionists$ $game_concept_rebel$s, observe that $CAS$ is forcibly called to $game_concept_war$ as the $game_concept_attacker$ ($game_concept_rebel$ side)
4. Observe that $CAS$ cannot call $game_concept_coalition$ members ($POR$, $NAV$) to join the $game_concept_war$ because it is the attacker
5. Observe that $FRA$ ($game_concept_defender$) CAN call $game_concept_allies$ ($ARA$, $PAP$) to join the defensive $game_concept_war$
6. Result: $FRA$ can bypass the $game_concept_coalition$ to attack $CAS$ alone, while $CAS$ cannot obtain $game_concept_coalition$ or ally support
7. Observe that $CAS$ will leave the $game_concept_coalition$ after the rebellion $game_concept_war$ ends

### CASE 6: $FRA$ vs $CAS$ LANDLESS REBEL BEHAVIOR (Demonstrates Issue 12 - VERIFIED)

Save File ID: **#39d23961**

Load the $FRA$ vs $CAS$ landless rebel save files. The saves demonstrate:
- $CAS$ has conquered Loudun (French culture territory)
- French $secessionists$ $game_concept_rebellion$ spawns as $game_concept_society_of_pops$ (<50% French culture population)
- $FRA$ (culturally dominant) is forcibly called to support the $game_concept_rebellion$

**Steps to observe Issue 12** (Landless Rebel Behavior):

1. Load save 1337.5.1 - $CAS$ Conquer Loudun, add revolt 100%
   - Play as $CAS$
   - Console: `conquer loudun`
   - Add rebel progress to 1 (100%)
2. Load save 1337.6.1 - $FRA$ Join Secessionist War, Occupy $CAS$, 1 - No Land Transferred to AAA00
   - Play as $FRA$
   - Console: `occupy_country CAS` to gain 100% $game_concept_war_score$
   - Observe: No land is transferred to newly spawned tag AAA00 (a landless rebel)
3. Load save 1337.6.1 - $FRA$ Join Secessionist War, Occupy $CAS$, 2 - Loudun Transferred to AAA00
   - Play as $FRA$
   - Console: `occupy_country CAS` to gain 100% $game_concept_war_score$
   - Observe: Loudun is transferred to newly spawned tag AAA00 (a landless rebel)
4. Load save 1337.8.1 - $FRA$ 1. Set Cash 0
   - 2 months after save 2
   - Set cash to 0 to check gained $game_concept_war_reparations$
5. Load save 1337.8.9 - $FRA$ 1. Gain 350.72 War Reparation, AAA00 Becomes Landless Secessionist
   - 9 days after save 2
   - AAA00 makes peace with $CAS$
   - Observe: $FRA$ gained 350.72 ducats, and AAA00 is a landless $secessionists$ $vassal$ with 1 unit of cavalry (5 people)
6. Load save 1337.8.1 - $FRA$ 2. Set Cash 0
   - 2 months after save 3
   - Set cash to 0 to check gained $game_concept_war_reparations$
7. Load save 1337.8.11 - $FRA$ 2. Gain 314.82 War Reparation, AAA00 Becomes landed INDEPENDENT
   - 11 days after save 2
   - AAA00 made peace with $CAS$
   - Observe: $FRA$ gained 314.82 ducats, and AAA00 is now a settled country but **NOT a $secessionists$ $vassal$** of $FRA$

**CRITICAL OBSERVATIONS:**
- Two major powers ($FRA$ and $CAS$) fight a $game_concept_war$ for essentially nothing
- Best case scenario with 100% $game_concept_war_score$: $FRA$ gains only 314.82 to 350.72 ducats ($game_concept_war_reparations$)
- The landless rebel (AAA00) either:
  1. Becomes a landless $secessionists$ $vassal$ with 1 unit of cavalry (5 people) - effectively useless
  2. Becomes a settled country but **NOT a $secessionists$ $vassal$** of $FRA$ - completely independent
- This demonstrates that the secessionist concept completely falls apart for landless rebels - there is almost no benefit to fighting in a secessionist $game_concept_war$ for a landless secessionist because the supposedly overlord $FRA$ gains almost nothing, and the landless rebel does NOT become $FRA$'s useful $vassal$ if it becomes settled

### CASE 7: $CAS$ SAVES - ATTACKER ALLY CALLING (Demonstrates Issue 4 - VERIFIED)

Save File ID: **#db345ae3**

Load the $CAS$ save files. The saves demonstrate:
- $CAS$ has $game_concept_alliance$ with $POR$
- $CAS$ has conquered Loudun (French culture territory)
- French $secessionists$ $game_concept_rebellion$ spawns as $game_concept_society_of_pops$ (<50% French culture population)
- $FRA$ (culturally dominant) is forcibly called to support the $game_concept_rebellion$
- $FRA$ has $game_concept_alliance$ with $ARA$ and $PAP$
- $FRA$ has 100 $game_concept_favors$ with $ARA$ and $PAP$

**Steps to observe Issue 4** (Attacker Ally Calling Using $game_concept_favors$):

1. Load save 1337.5.1 - $CAS$ Ally $POR$, Conquer Loudun, Add Revolt 100%
   - Play as $CAS$
   - $CAS$ has $game_concept_alliance$ with $POR$
   - Console: `conquer loudun`
   - Add rebel progress to 1 (100%)
2. Load save 1337.5.1 - $FRA$ Ally $PAP$, $ARA$, $game_concept_favor$ 100
   - Tag to play as $FRA$
   - $FRA$ allied with $ARA$, $PAP$
   - Use console command `favor ARA 100` and `favor PAP 100` to add $game_concept_favor$
3. Load save 1337.6.1 - $FRA$ Secessionist $game_concept_war$
   - $secessionists$ $game_concept_rebel$ breaks out on 1337.6.1
   - $game_concept_society_of_pops$ $game_concept_rebel$ in Loudun is the $game_concept_war_leader$, hence $FRA$ cannot call $ARA$ and $PAP$ to arms
4. Load save 1337.6.3 - $FRA$ Killed Landless $game_concept_rebel$, CAN Call $ARA$ and $PAP$
   - Use console command `kill_unit` to kill troops of $game_concept_society_of_pops$ $secessionists$ $game_concept_rebel$
   - $FRA$ becomes $game_concept_war_leader$
   - $FRA$ can call $ARA$ and $PAP$ to join $game_concept_war$ using $game_concept_favors$

---

## Game Saves Provided

Multiple game saves have been prepared and attached:

**$BYZ$ Saves** (Issues 1, 3, 4, 6, 7):
Save File ID: **#f40b1288**
- 1337.7.2 - Byzantium HUD Revolt 100%
- 1337.8.1 - Byzantium HUD ERE War
- 1339.2.1 - Byzantium HUD ERE War Stuck
- Additional saves demonstrating various scenarios

**$FRA$ $game_concept_personal_union$ Saves** (Issue 2 - VERIFIED):
Save File ID: **#51d1fcfd**
- 1337.5.1.2 - $FRA$ $game_concept_personal_union$ $SIC$, No CB $NAP$
- 1337.6.1 - $FRA$ Conquer $NAP$
- 1337.7.1 - $FRA$ Sicilian Rebel 100%
- 1337.8.1 - $FRA$ $game_concept_personal_union$ Broken

**$FRA$ $fiefdom$ Saves** (Verification - No Call):
Save File ID: **#1177c64a**
- 1337.4.1 - $FRA$ ANNEX $SIC$, Create $fiefdom$ subject
- 1337.6.1 - $FRA$ Conquer $NAP$
- 1337.7.1 - $FRA$ Sicilian Rebel 100%
- 1337.8.1 - $FRA$ Sicilian $secessionists$ NOT calling $SIC$

**$FRA$ $vassal$ Saves** (Verification - No Call):
Save File ID: **#dae84461**
- 1337.4.1 - $FRA$ ANNEX $SIC$, Create $vassal$ subject
- 1337.6.10 - $FRA$ Conquer $NAP$
- 1337.7.1 - $FRA$ Sicilian Rebel 100%
- 1337.8.1 - $FRA$ Secessionist NOT calling $SIC$ $vassal$

**$FRA$ $game_concept_coalition$/$game_concept_truce$ Saves** (Issues 1, 5, 8):
Save File ID: **#75e13d1**
- 1338.4.1 - $FRA$ $game_concept_truce$ with $CAS$, Add Castilian Rebel to 100%
- 1346.8.2 - $FRA$ $CAS$ Joint $game_concept_coalition$, Add Castilian Rebel to 100%
- Additional saves demonstrating various scenarios

**$FRA$ vs $CAS$ Landless Rebel Saves** (Issue 12 - VERIFIED):
Save File ID: **#39d23961**
- 1337.5.1 - $CAS$ Conquer Loudun, add revolt 100%
- 1337.6.1 - $FRA$ Join Secessionist War, Occupy $CAS$, 1 - No Land Transferred to AAA00
- 1337.6.1 - $FRA$ Join Secessionist War, Occupy $CAS$, 2 - Loudun Transferred to AAA00
- 1337.8.1 - $FRA$ 1. Set Cash 0
- 1337.8.9 - $FRA$ 1. Gain 350.72 War Reparation, AAA00 Becomes Landless Secessionist
- 1337.8.1 - $FRA$ 2. Set Cash 0
- 1337.8.11 - $FRA$ 2. Gain 314.82 War Reparation, AAA00 Becomes landed INDEPENDENT

**$CAS$ Saves - Attacker Ally Calling** (Issue 4 - VERIFIED):
Save File ID: **#db345ae3**
- 1337.5.1 - $CAS$ Ally $POR$, Conquer Loudun, Add Revolt 100%
- 1337.5.1 - $FRA$ Ally $PAP$, $ARA$, $game_concept_favor$ 100
- 1337.6.1 - $FRA$ Secessionist $game_concept_war$
- 1337.6.3 - $FRA$ Killed Landless $game_concept_rebel$, CAN Call $ARA$ and $PAP$

---

## Relevant Files

- `game/in_game/common/subject_types/secessionists.txt` (lines 12-17: `join_offensive_wars_always`, `join_defensive_wars_always`)
- `game/in_game/common/subject_types/vassal.txt`
- `game/in_game/common/scripted_relations/alliance.txt` (lines 13-14: `called_in_defensively`, `called_in_offensively`)
- `game/in_game/common/scripted_relations/union_of_crowns_pact.txt` (line 10: `break_on_war = yes`)
- `game/in_game/common/on_action/_hardcoded.txt` (line 717-777: `on_civil_war_start` - hardcoded mechanism for calling same-culture countries)
- `game/in_game/common/scripted_effects/global_effects.txt` (line 218: `start_civil_war`)
- `game/in_game/common/prices/00_hardcoded.txt` (lines 100-103: `war_breaking_truce` - stability = 50, war_exhaustion = 1)
- `game/in_game/common/biases/05_antagonism_hardcoded.txt` (lines 35-38: `antagonism_breaking_truce` - value = 25)
- `game/in_game/common/wargoals/00_default.txt` (lines 89-99: `take_country_nationalist` - `type = take_country`, `conquer_cost = 0.25`, `subjugate_cost = 0.25` - low warscore costs for $secessionists$ $game_concept_wars$, intended for annexing $game_concept_rebel$s but exploitable when war participant $game_concept_country$ becomes $game_concept_war_leader$ due to Issue 10. The `type = take_country` allows annexing the entire $game_concept_country$ via $WAR_LATERALVIEW_ANNEX_REVOLTER$ button)
- `game/main_menu/localization/english/war_overview_l_english.yml` (line 8: `WAR_LATERALVIEW_ANNEX_REVOLTER: "Annex Revolter"` - peace option that allows annexing revolter countries)
- `game/main_menu/localization/english/diplomacy_l_english.yml` (lines 1149-1150: `PEACE_TREATY_ANNEX_REVOLTER` - peace treaty description for annexing revolters)
- `game/loading_screen/common/defines/00_defines.txt` (line 1673: `MIN_WARSCORE_TO_DEMAND = 10` - minimum war score required to enable peace options like $WAR_LATERALVIEW_ANNEX_REVOLTER$; lines 1832-1833: `PEACE_TREATY_ANNEX_REVOLTER_MAX_COST = 70`, `PEACE_TREATY_REVOLTER_SURVIVES_MAX_COST = 70`)
