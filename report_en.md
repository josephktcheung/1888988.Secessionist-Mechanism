# Bug Report: Secessionists Mechanism

## Metadata

**Title:** Secessionists Mechanism - Forced Participation, Original Owner Ignored, World War Escalation, and Exploits

**Issue Type:** Gameplay

**Bug Type:** AI (non-player characters), Other

**Game Version:** 1.0.10

---

## Summary

The Secessionists mechanism has critical design flaws that create game-breaking exploits. Same-culture Countries are forcibly called to War without player control, **completely ignoring original ownership** in favor of cultural dominance. The mechanism breaks Personal Union, creates inconsistent War leadership logic where Vassals lead Secessionists Civil Wars (unlike normal War declarations where Overlords lead), prevents player control when Vassals lead Wars, incorrectly applies Truce penalties, and allows the Annex Revolter button to annex entire war participant Countries (e.g., France) with no Antagonism. Asymmetric Alliance calling creates dangerous world War escalation where Defenders can coordinate while forcibly called Attackers cannot. Landless Secessionists Rebellions (Society of Pops) produce inconsistent outcomes where culturally dominant Countries gain nothing for fighting, unlike land-based Rebellions which become Vassals. These flaws enable systematic exploits: (1) Vassal players can use Overlords as weapons to attack same-culture Countries (VERIFIED); (2) Players can attack opponents during Truce periods by triggering Secessionists Rebellions that force victims to join and receive -50 Stability penalties.

---

## Description

The Secessionists mechanism has twelve critical issues:

### Issue 1: Same-Culture Countries Forced to Join (Most Severe)

When Secessionists Rebels, the system automatically searches for nearby same-culture Countries and forcibly calls them to War on the Rebel side. Called Countries cannot refuse. If they have a Truce with the enemy, they are incorrectly judged as actively breaking the Truce, triggering severe penalties (-50 Stability, +1 War Exhaustion, +25 Antagonism). The selection mechanism is hardcoded in the engine and cannot be found in game files, making it impossible for players to predict which country will be called.

**CRITICAL FLAW: Original Ownership Completely Ignored**

The system **completely ignores original ownership** when selecting which same-culture Country to call, prioritizing cultural dominance over historical claims.

*Example from Byzantium save:* Turkish Secessionists Rebellion occurs in territories originally owned by Ottomans. Ottomans still exist and are the original owner. However, the system selects Eretnids (culturally dominant Turkish Country) instead, completely ignoring Ottomans. Ottomans have no opportunity to reclaim their former territories, even though they have the strongest historical claim.

### Issue 2: Personal Union Breaking (Severe - VERIFIED)

When a Personal Union Junior Partner is the culturally dominant Country for a Secessionists Rebellion, it is forcibly called to support the Rebellion against its Senior Partner. This immediately breaks the Personal Union because `union_of_crowns_pact` has `break_on_war = yes`.

**VERIFICATION STATUS:**
- Personal Union: **VERIFIED AFFECTED** (tested with France-Sicily)
- Vassal: **VERIFIED EXCLUDED** (tested with France-Sicily)
- Fiefdom: **VERIFIED EXCLUDED** (tested with France-Sicily)
- Other subject types (Tributary, Dominion, March, Appanage, Sāmanta, Tǔsī, Uç Bey, etc.): NOT TESTED

This creates an inconsistency where Personal Union are vulnerable while some other subject types are protected.

**VERIFICATION:** Tested with France (Senior Partner) and Sicily (Junior Partner). When Sicilian Secessionists Rebels in France-controlled territory, Sicily was forcibly called, immediately breaking the Personal Union. When Sicily was a Vassal or Fiefdom instead, it was correctly excluded.

### Issue 3: Vassal War Leadership Problem (Severe)

If a player creates a Vassal that becomes the Defender in a Civil War, the player is forcibly dragged into the War but does not become the War Leader. The AI Vassal controls all peace negotiations. The player cannot call Allies, cannot obtain Military Access, and must bear all War costs without control.

**CRITICAL INCONSISTENCY: Normal War Declaration vs. Secessionists Civil War**

There is a fundamental inconsistency in game logic regarding Vassal War leadership between normal War declarations and Secessionists Civil Wars:

**Normal War Declaration (using Casus Belli and Diplomats):**
- When a Country declares War against a Vassal (e.g., Byzantium declares War against Albania where Albania is a Vassal of Naples), the Overlord (Naples) becomes the War Leader, NOT the Vassal (Albania)
- The Overlord controls all peace negotiations and War decisions
- This is the expected behavior for normal War declarations

**Secessionists Civil War (Issue 3):**
- When a Vassal becomes the Defender in a Secessionists Civil War, the Vassal becomes the War Leader, NOT the Overlord
- The Overlord is forcibly dragged into the War but does not control peace negotiations
- The AI Vassal controls all peace negotiations, while the Overlord has no control

**The Inconsistency:**
- **Normal War declarations:** Overlord is War Leader when Vassal is attacked
- **Secessionists Civil Wars:** Vassal is War Leader when Vassal is Defender
- This creates inconsistent game logic where the same relationship (Overlord-Vassal) produces different War leadership outcomes depending on how the War starts
- Players cannot predict or understand which party will control the War based on the relationship structure alone

**Impact:**
- Creates confusion for players who expect consistent behavior
- Makes it impossible to predict War leadership based on subject relationships
- Breaks game logic consistency between different War types
- In Secessionists Civil Wars, Overlords lose control they would normally have in regular Wars

### Issue 4: Inconsistency in Alliance Auto-Call

When the player is the defensive War Leader, Allies can be asked to join. But when a Vassal is the defensive War Leader, the player cannot call Allies to join.

**CRITICAL LIMITATION: Forcibly Called Culturally Dominant Country (Attacker) Cannot Call Allies Using Favors**

When a same-culture Country is forcibly called to support a Secessionists Rebellion as the Attacker, it **cannot call its Allies using Favors** because it is not the War Leader (the Rebellion country is the War Leader). 

**VERIFIED: Cannot Integrate Secessionists Vassal During War**

It is **NOT possible** to integrate/annex a Secessionists Vassal during an ongoing War where the Secessionists Vassal is the War Leader. The game blocks annexation of countries that are in a Civil War (see `ANNEX_TARGET_CIVIL_WAR_PENALTY` localization: "is currently going through a [civil_war|e] and can not be annexed until peace has been restored").

**ONLY Scenario Where Culturally Dominant Country CAN Call Allies:**

**When the landless Secessionists Rebel is eliminated by the Defender**: If the Secessionists Rebellion spawns as a Society of Pops (landless with no territory) and its troops are eliminated by the Defender, the Rebellion is removed from the War. The war participant culturally dominant Country then becomes the War Leader automatically, and can call Allies using Favors (cost: 10 Favors per Ally).

**Technical Verification:**
- Secessionists Vassals CANNOT be integrated/annexed during an ongoing War where they are the War Leader (blocked by civil war restriction)
- The `ask_join_war_for_favors` interaction requires the caller to be the War Leader (`attacker_leader = scope:actor` or `defender_leader = scope:actor`)
- Cost: 10 Favors per Ally (from `ask_join_war_for_favors_cost`)
- War leadership transfer: When the landless Rebel (War Leader) is eliminated, the remaining war participant automatically becomes the new War Leader

**Impact:**
- The forcibly called culturally dominant Country is at a severe disadvantage for the entire War, unable to call Allies unless the landless Rebel is eliminated by the Defender
- This creates an asymmetric situation where the Defender can coordinate with Allies from day 1, while the culturally dominant Attacker cannot (see Issue 9)
- The only way for the culturally dominant Attacker to gain the ability to call Allies depends on the Defender's actions (eliminating the landless Rebel), giving the Defender control over when the attacker can call Allies

### Issue 5: Coalition Bypass Mechanism (Severe Exploit)

When a forcibly called Country is in a Coalition, it cannot call Coalition members to join the War because it is the Attacker. This allows the Defender to bypass Coalition restrictions and attack Coalition members individually.

### Issue 6: AI Vassal Suboptimal Peace Decisions

AI Vassal may choose White Peace instead of annexing the Revolter even when annexation cost is very low, causing players to lose territories they should have gained.

### Issue 7: Overlord Forced to Join

Secessionists automatically and forcibly call Overlords to War through `join_offensive_wars_always` and `join_defensive_wars_always` in `secessionists.txt`.

### Issue 8: Truce Breaking Penalties

Forcibly called Countries with Truce are incorrectly judged as actively breaking Truce, triggering severe penalties even though they had no choice.

### Issue 9: World War Escalation - Asymmetric Alliance Calling (Severe)

A single local Secessionists Rebellion can escalate into a world War due to asymmetric Alliance calling mechanics. This is **worse than historical world Wars** (WW1/WW2) where both sides could coordinate with Allies.

**Current EU5 System - Worse Than Historical World Wars:**

When a Secessionists Rebellion breaks out:
- The Defender CAN call all its Allies to join the defensive War from day 1
- The forcibly called same-culture Country (Attacker) CANNOT call its own Allies using Favors because it is NOT the War Leader (the Rebellion country is the War Leader)
- **ONLY exception scenario where the culturally dominant Attacker CAN call Allies** (see Issue 4 for details):
  - After the landless Secessionists Rebel is eliminated by the Defender (becomes War Leader automatically)
- **VERIFIED: Cannot integrate Secessionists Vassal during War** - Annexation is blocked for countries in Civil War until peace is restored
- Result: A local Rebellion escalates into a major War where one Alliance bloc fights alone against another bloc's coordinated response for the entire War (unless the Defender eliminates the landless Rebel, which is rare and gives the Defender control over when the attacker can call Allies)

*Example Scenario (Late 18th Century - Partition Era):*
- Two major Alliance blocs: Bloc A (Russia, Austria, Prussia) vs Bloc B (France, Ottomans, Sweden, Poland)
- Polish Secessionists Rebellion breaks out in Russia-controlled territory
- Poland (culturally dominant) is FORCED to join Rebellion's side (Attacker) - NO CHOICE
- Russia (Defender) calls Austria and Prussia → 3 major powers vs Poland
- Poland (Attacker) cannot call France, Ottomans, or Sweden (its own allies in Bloc B) → fights alone against 3 major powers

This creates a dangerous escalation mechanism where the Defender can systematically use Secessionists Rebellions to attack their enemies' Alliance blocs asymmetrically.

### Issue 10: Annex Revolter Button Misapplied to War Participant Countries (Severe - GAME-BREAKING)

When a Secessionists Rebellion spawns as a Society of Pops (landless Country with no territory) due to low population size (Province has <50% of the rebelling culture's population), the Rebel Country has no territory. When the Rebel Country's troops are wiped out and the Rebellion is eliminated, the War continues and a war participant Country becomes the War Leader. The Annex Revolter button, designed to annex the entire Revolter Country, can then be used to annex the entire war participant Country (e.g., France - 3rd largest Country at game start) with no Antagonism generated.

**Exploitation Flow (England vs France Example):**

1. **England spawns a French Rebellion that becomes a Society of Pops** (Defender - owns rebelling territory with French culture)
2. **France is forcibly called** as Attacker (culturally dominant for French culture)
3. **War structure:** England (Defender/War Leader) vs Rebellion (Attacker/War Leader) + France (Attacker/war participant)
4. **England wipes out Society of Pops troops** (Rebellion eliminated)
5. **War continues** because France is still in the War
6. **France becomes War Leader** (only remaining Attacker)
7. **England gains Warscore** through ticking Warscore (`ticking_war_score = 0.5` per month from `take_country_nationalist` war goal) and by winning battles, sieging enemy Locations, and establishing blockades
8. **England reaches 10 Warscore** - The Annex Revolter button becomes enabled when England reaches at least 10 Warscore (`MIN_WARSCORE_TO_DEMAND = 10` from game defines)
9. **Annex Revolter button becomes available** - Once enabled, this peace option can be used on the war participant Country (France) instead of the eliminated Revolter
10. **England uses Annex Revolter button** to annex the entire France Country
11. **England annexes entire France** at effectively 0 cost (base 25% cost from `take_country_nationalist` war goal, but reduced by -95% from `PEACE_COST_MODIFIER_FOR_REVOLT_WAR` modifier, making it effectively 0 cost). This is intended only for annexing the Revolter, not major powers like France
12. **No Antagonism is generated** - annexing a major power like France should generate massive Antagonism, but the peace option generates none

**Technical Details:**
- When Rebellion Province has <50% population of rebelling culture, Rebellion spawns as `country_type = pop` (Society of Pops - landless with no territory)
- When `country_type = pop` troops are eliminated, the Rebellion is removed from the War
- The war participant Country becomes War Leader automatically
- **BUG:** The `take_country_nationalist` war goal has `type = take_country`, which allows annexing the entire Country via Annex Revolter button
- **BUG:** The warscore cost is effectively 0 (base 25% from `conquer_cost = 0.25` in `take_country_nationalist`, but reduced by -95% from `PEACE_COST_MODIFIER_FOR_REVOLT_WAR = -0.95` modifier, making it effectively 0 cost)
- **BUG:** No Antagonism is generated when annexing the war participant Country
- **Note:** The Defender can theoretically conquer land at 25% cost if it is the War Leader (from `take_country_nationalist` war goal), but AI Vassal will never conquer land for its Overlord when the Vassal is the War Leader in a Secessionists Rebellion (see Issue 6)

**Impact:**
- **GAME-BREAKING:** Major powers like France (3rd largest Country at game start) can be annexed by smaller Countries like England through this exploit
- **VERIFIED EXAMPLE 1:** France was annexed by England through this mechanism - England spawned a Rebellion that became a Society of Pops, France was forcibly called, England eliminated Rebels, gained 10 Warscore (minimum required), then used Annex Revolter button to annex entire France, and no Antagonism was generated
- **VERIFIED EXAMPLE 2:** Jalayirids was annexed by Ottomans through this mechanism - Ottomans had very few Iraqi culture pops in Gumushane location (only 31 Iraqi culture nobles rebelling, <50% of location population), spawning a Rebellion that became a Society of Pops with only 3 soldiers instead of making the location rebel territory. Because it was an Iraqi culture rebellion, Jalayirids (Iraqi culture dominant Country) was forcibly called. Ottomans eliminated the 3 soldiers (Society of Pops disappeared), Jalayirids became War Leader, Ottomans gained 10% Warscore by sieging a few forts, then used Annex Revolter button to annex entire Jalayirids. The annexed land was unintegrated territory requiring manual coring

### Issue 11: Subjects Forced to Support Distant Rebellions (Severe)

Any subject (e.g., Vassal, Secessionists subject, or other subject types) that happens to be culturally dominant for the rebelling culture is forcibly called to support distant rebellions of the same culture, even when they are thousands of kilometers away and have no logical connection to the rebellion.

**The Problem:**
- When a Secessionists Rebellion breaks out, the system searches for culturally dominant Countries to call
- The system does NOT check if the Country is a subject, its size, or distance to the Rebellion
- Any Country with the same culture as the rebelling population can be called, regardless of whether it is a subject
- Result: Small subjects (e.g., Vassals, Secessionists subjects) get forcibly called to support Rebellions thousands of kilometers away

**VERIFIED EXAMPLE:**
- Player (Yuán / Ming) has a small Mongolian culture subject (e.g., Vassal or Secessionists subject) in Mongolia region
- A Mongolian Secessionists Rebellion breaks out in Georgia (6000+ km away)
- The small subject is FORCED to join the Rebellion's side (Attacker) - NO CHOICE
- Player's subject gets dragged into a War far away, player has no control
- Player must search through war participant Countries to find out why their subject is at War
- The subject receives penalties (Truce breaking, War Exhaustion, Antagonism) even though it had no choice
- The Overlord (Yuán) is also dragged into the War (Issue 7), compounding the problem

**Why This Is Problematic:**
- **No logical connection:** A small subject in Mongolia has no reason to support a Rebellion in Georgia
- **No player control:** Player cannot prevent their subject from being called
- **Hidden cause:** Player must investigate to find out why their subject is at War
- **Penalties without choice:** Subject receives severe penalties for actions it did not choose
- **Overlord dragged in:** Overlord is also forced to join (Issue 7), creating a cascade of forced participation
- **Exploitable:** Defenders can use this to force small subjects and their Overlords into Wars

**Technical Details:**
- The hardcoded mechanism in `on_civil_war_start` does not check `is_subject = yes` or subject type
- The system only checks cultural dominance, ignoring subject status, distance, and size
- Subjects are treated the same as independent Countries for the purpose of being called

**Impact:**
- Small subjects get dragged into distant Wars they have no interest in
- Players lose control over their subjects' foreign policy
- Creates confusion and frustration (player must investigate why subject is at War)
- Enables exploits where Defenders can force small subjects and Overlords into Wars

### Issue 12: Inconsistent Behavior of Land-Based vs. Landless Secessionists Rebellions (Severe)

When a Secessionists Rebellion spawns, its behavior differs dramatically based on whether it is land-based (owns territory) or landless (Society of Pops with no territory). This creates inconsistent outcomes that break the intended Secessionists mechanism.

**The Problem:**

**Land-Based Secessionists Rebellions (>50% cultural population):**
- When the Rebellion breaks out, it controls territory
- When the Rebellion wins independence, it becomes a Secessionists Vassal of the culturally dominant Country
- The culturally dominant Country gains a new Vassal as a reward for supporting the Rebellion

**Landless Secessionists Rebellions (<50% cultural population - Society of Pops):**
- When the Rebellion breaks out, it has no territory (only pops)
- When the Rebellion wins independence, it has **TWO INCONSISTENT OUTCOMES:**
  1. **Becomes an INDEPENDENT Country**, NOT a Vassal of the culturally dominant Country
  2. **Becomes a landless Secessionists Vassal** with no territory (only 1 unit of cavalry with 5 people) - effectively useless
- The culturally dominant Country gains **ALMOST NOTHING** for fighting the War - only minimal War Reparations (314.82 to 350.72 ducats) even at 100% Warscore
- The AI culturally dominant Country may not take any land even at 100% Warscore, only accepting War Reparations
- **CRITICAL:** The entire secessionist concept falls apart for landless rebels - culturally dominant Countries fight major Wars between major powers (e.g., France vs Castile) for essentially nothing

**VERIFIED EXAMPLES:**

**Example 1 - Landless Rebel (England vs France):**
- England spawns a French Rebellion that becomes a Society of Pops (<50% French culture population)
- France (culturally dominant) is forcibly called to support the Rebellion
- France fights the War and achieves 100% Warscore
- The landless Rebellion becomes an **INDEPENDENT** Country (not a Vassal of France)
- France receives only a portion of War Reparations (e.g., 238.09 ducats) despite fighting the entire War
- France gains **NO TERRITORY** and **NO Vassal** despite winning the War with 100% Warscore

**Example 2 - AI Not Taking Land:**
- In some cases, the AI culturally dominant Country does not take any land even at 100% Warscore
- The AI may only accept War Reparations or White Peace, leaving the Defender with all territory intact
- This is inconsistent with land-based Rebellions where the culturally dominant Country gains a Vassal

**Example 3 - Landless Rebel Behavior (France vs Castile - VERIFIED):**
- Castile conquers Loudun (French culture territory)
- French Secessionists Rebellion spawns as Society of Pops (<50% French culture population)
- France (culturally dominant) is forcibly called to support the Rebellion
- France fights Castile (two major powers in Europe) and achieves 100% Warscore
- **Outcome 1:** The landless rebel (AAA00) becomes a **landless Secessionists Vassal** of France with no territory, only 1 unit of cavalry (5 people) - effectively useless
- **Outcome 2:** The landless rebel becomes a **settled country** but **NOT a Secessionists Vassal** of France - completely independent
- France gains only **314.82 to 350.72 ducats** (War Reparations) despite fighting an entire major War with 100% Warscore
- France gains **NO TERRITORY** and either **NO USEFUL Vassal** (landless vassal with 5 people) or **NO Vassal AT ALL** (independent country)
- **Summary:** Two major powers (France and Castile) fight a War for either: (1) a landless Secessionists Vassal with 1 unit of cavalry (5 people), (2) an independent settled country, or (3) 314.82 to 350.72 ducats. This demonstrates that the secessionist concept completely falls apart for landless rebels - there is almost no benefit to fighting in a secessionist War for a landless secessionist because the supposedly overlord France gains almost nothing, and the landless rebel does NOT become France's useful Vassal if it becomes settled

**Why This Is Problematic:**
- **Inconsistent outcomes:** Same mechanism produces different results based on population size
- **No meaningful reward for culturally dominant Country:** Landless Rebellions provide almost no benefit to the culturally dominant Country that is forced to fight - only minimal War Reparations (314-350 ducats) even at 100% Warscore
- **Two problematic outcomes:** Landless rebels either become independent (no Vassal reward) or become useless landless Vassals with no territory (only 5 people in 1 unit)
- **Major War for nothing:** Two major powers fight a War for essentially nothing - the culturally dominant Country gains almost no benefit despite winning with 100% Warscore
- **AI suboptimal behavior:** AI may not take land even when it should, wasting Warscore
- **Breaks intended design:** The Secessionists mechanism is supposed to reward culturally dominant Countries with useful Vassals, but landless Rebellions completely break this design
- **Unfair to players:** Players who are forcibly called into landless Rebellion Wars gain almost nothing for their effort
- **Secessionist concept falls apart:** The entire secessionist concept becomes meaningless for landless rebels, as they can be independent or become useless landless Vassals

**Technical Details:**
- When Province has <50% population of rebelling culture, Rebellion spawns as `country_type = pop` (Society of Pops - landless with no territory)
- When `country_type = pop` wins independence, it has inconsistent outcomes:
  1. Becomes an independent Country (not a Vassal)
  2. Becomes a landless Secessionists Vassal with no territory (only pops/units, effectively useless)
- The `secessionists.txt` subject type definition does not have `ai_wants_to_be_overlord`, which may explain why AI doesn't try to make landless rebels into useful Vassals
- The `PEACE_MAKE_SUBJECT_REVOLT_WAR = 0` define in `00_defines.txt` indicates AI has no interest in making subjects in revolt Wars, which explains suboptimal peace decisions
- Even when landless rebels become Vassals, they have no territory and minimal military (e.g., 1 unit of cavalry with 5 people), making them effectively useless

**Impact:**
- Culturally dominant Countries gain almost nothing from supporting landless Rebellions - only minimal War Reparations (314-350 ducats) even at 100% Warscore
- Major powers fight major Wars for essentially nothing (e.g., France vs Castile for a landless Vassal with 5 people or an independent country)
- Players are forced to fight Wars with almost no reward
- AI makes suboptimal peace decisions, not taking land even at 100% Warscore
- Breaks the intended design of the Secessionists mechanism where culturally dominant Countries should gain useful Vassals
- The entire secessionist concept becomes meaningless for landless rebels, as they either become independent or useless landless Vassals

---

## Exploitation Scenarios

### EXPLOIT 1: Vassal PLAYER USING OVERLORD AS WEAPON (VERIFIED)

Players who choose to play as Vassal can systematically use their Overlords as weapons to attack and conquer territories from same-culture Countries, then eventually declare independence.

**Exploitation Strategy** (VERIFIED - Byzantium Case):

1. Player chooses to play as a Vassal (e.g., Byzantium Vassal Hudavendigar)
2. Player intentionally increases Secessionists Rebel monthly progress through various means (not coring territories, low Stability, low legitimacy, maximizing taxes, reducing control, etc.)
3. When Secessionists Rebellion breaks out:
   - The Overlord (e.g., Byzantium) is AUTOMATICALLY dragged into the War on the defensive side (Issue 7)
   - The culturally dominant Country (e.g., Eretnids for Turkish culture) is FORCED to join the Rebellion's side (Attacker) (Issue 1)
   - The player's Vassal becomes the defensive War Leader, giving the player full control over peace negotiations
4. Player advantages as Vassal War leader:
   - Can react to Rebels on day 1, sending regulars to occupy Rebel Provinces immediately
   - Overlord automatically sends troops to help
   - War score to annex Rebels is very low (e.g., 2 points), easily achievable
   - Any extra War score can be used to demand territories/money from the forcibly called war participant Country
5. Player controls War duration based on military situation
6. Player gains all benefits from the War: territories and ducats from the forcibly called war participant Country, uses Overlord's military strength without cost, builds up power for eventual independence
7. Once strong enough, player declares independence and defeats the weakened Overlord

**VERIFICATION:** Tested in Byzantium game. Player played as Byzantium Vassal Hudavendigar. Turkish Secessionists Rebellion broke out, Eretnids (culturally dominant Turkish Country) was forcibly called. Byzantium (Overlord) was automatically dragged into the War. Player (as Vassal War leader) was able to sue for peace and conquer Eretnids's territories.

**Impact:**
- Vassal players can systematically exploit their Overlords to fight Wars for them
- No need to fabricate claims, create casus belli, or manage Truce timers
- Can build up power and resources using Overlord's military strength
- Eventually declare independence when strong enough
- Makes Vassal gameplay exploitable and breaks intended gameplay balance
- In multiplayer, this allows Vassal players to systematically weaken their Overlord players

### EXPLOIT 2: DOUBLE WAR ATTACK

The Secessionists mechanism creates a severe exploit that allows knowledgeable players to attack opponents (AI or human players) during a Truce period by intentionally creating that Truce period, then triggering Secessionists Rebellion which forces the victim to join and receive devastating penalties.

**Exploitation Strategy:**

1. Player A (exploiter) acquires territories with the same primary culture as Player B (victim - AI or human player) through War 1 or from game start/previous conquests
2. Player A intentionally allows or encourages Secessionists Rebellion to develop (Rebel progress approaching 100%)
3. Player A wages War 2 against Player B when the Secessionists Rebellion is about to break out (Rebel progress near 100%)
4. Player A makes White Peace or gets a small amount of ransom, ending War 2 quickly, creating a new Truce period
5. Player A forms Alliance with other Countries/players during this new Truce period
6. When Secessionists Rebellion breaks out:
   - Player B (culturally dominant) is FORCED to join the Rebellion's side (Attacker)
   - Player B receives -50 Stability penalty for "breaking Truce" (even though forced)
   - Player B cannot call Allies or Coalition members because it is the Attacker
   - Player A (Defender) CAN call all Allies to join the defensive War
   - Result: Player A attacks Player B during the Truce period with Player B suffering severe penalties and unable to defend effectively

**Impact:**
- This exploit completely breaks game balance
- Knowledgeable players can systematically destroy opponents (AI or human) by exploiting this mechanism
- Victim Countries have no counterplay - they cannot prevent the Secessionists Rebellion, cannot refuse the call, and cannot call their own Allies
- The -50 Stability penalty can cause Country collapse, making the victim's game unplayable
- This creates an unfair advantage that makes games uncompetitive

**Example Scenario:**
- Player (France) acquires Castilian-culture territories (either through War 1 against Castile if needed, or already owns them)
- Player allows Castilian Secessionists Rebellion to develop (Rebel progress approaching 100%)
- Player declares War 2 against Opponent (Castile - AI or human player) when Castilian Rebellion is at 95% progress
- Player quickly white peaces War 2, creating a new Truce period
- Player forms Alliance with Aragon and Rome during this new Truce period
- Castilian Secessionists Rebellion breaks out
- Opponent (Castile) is forced to join Rebellion side, receives -50 Stability, cannot call Allies
- Player calls Aragon and Rome, attacks Opponent again with overwhelming advantage
- Opponent's Country collapses due to Stability penalty and cannot effectively defend

---

## Severity

These issues are critical because:
- Players have absolutely no control over being dragged into Wars
- **Original owners are completely ignored** - The system ignores original ownership and only looks at cultural dominance (e.g., Ottomans ignored in favor of Eretnids in Byzantium case)
- **Inconsistent War leadership logic** - In normal War declarations, Overlords become War Leader when their Vassals are attacked, but in Secessionists Civil Wars, Vassals become War Leader instead. This creates unpredictable and inconsistent game logic that breaks player expectations
- Personal Union break unexpectedly due to forced participation
- Players receive severe penalties (-50 Stability) for actions they did not choose
- **Rebellions that spawn as Society of Pops cause warscore costs to be applied to wrong target** - When Society of Pops troops are wiped out and Rebellion is eliminated, the War continues with the war participant Country as War Leader. The Annex Revolter button (intended only for annexing Rebels) can then be used to annex the entire war participant Country at effectively 0 cost (base 25% cost from `take_country_nationalist` war goal, but reduced by -95% from `PEACE_COST_MODIFIER_FOR_REVOLT_WAR` modifier, making it effectively 0 cost). This allows major powers like France (3rd largest Country at game start) to be annexed by smaller Countries like England. **Note:** The Defender can theoretically conquer land at 25% cost if it is the War Leader, but AI Vassal will never conquer land for its Overlord when the Vassal is the War Leader in a Secessionists Rebellion (see Issue 6)
- The mechanism can be systematically exploited by knowledgeable players in multiple ways (see EXPLOITATION SCENARIOS section)
- The mechanism violates historical rationality
- Players lose territories and strategic control due to AI decisions
- Inconsistency: Personal Union are treated differently from other subject types (Vassal, Fiefdom are protected, Personal Union are not)
- **EXPLOIT 1:** Vassal players can use their Overlords as weapons to attack same-culture Countries and eventually declare independence, breaking intended gameplay balance
- **EXPLOIT 2:** Players can attack opponents (AI or human) during a Truce period by intentionally creating a Truce period through War 2, then triggering Secessionists Rebellion which forces the victim to join and receive -50 Stability penalties, preventing the victim from calling Allies. This completely breaks game balance in both single-player and multiplayer contexts
- **WORLD War ESCALATION:** A single local Rebellion can escalate into a world War due to asymmetric Alliance calling. The Defender can call all Allies, while forcibly called Attacker cannot call its own Allies, creating a dangerous escalation mechanism
- **SMALL SUBJECTS DRAGGED INTO DISTANT Wars:** Any subject (e.g., Vassal, Secessionists subject) that happens to be culturally dominant is forcibly called to support distant rebellions of the same culture, even when thousands of kilometers away. Players lose control over their subjects' foreign policy and must investigate to find out why their subjects are at War (VERIFIED: Yuán case - small Mongolian subject in Mongolia region forced to support rebellion in Georgia 6000+ km away)
- **INCONSISTENT BEHAVIOR OF LAND-BASED VS. LANDLESS Secessionists:** Land-based Secessionists Rebellions (>50% cultural population) become Vassals of the culturally dominant Country when they win, while landless Secessionists Rebellions (<50% cultural population - Society of Pops) have inconsistent outcomes: either become independent Countries or useless landless Vassals (e.g., 1 unit of cavalry with 5 people). This creates inconsistent outcomes where culturally dominant Countries gain almost nothing from fighting landless Rebellion Wars (only 314-350 ducats even at 100% Warscore), and the entire secessionist concept falls apart for landless rebels. Major powers fight major Wars for essentially nothing (VERIFIED: France vs Castile case - France fought entire War with 100% Warscore and gained only 314.82 to 350.72 ducats, no territory or useful Vassal; landless rebel either became independent or useless landless Vassal with 5 people)

---

## Recommended Priority

Issues 1, 2, and 5 should be prioritized as they have the most severe impact on gameplay experience and players have absolutely no control over them. Issue 2 (Personal Union Breaking) is particularly severe because it creates an inconsistency where one subject type (Personal Union) is vulnerable while some others (Vassal, Fiefdom) are protected. The exploitation scenarios (Exploit 1 and Exploit 2) make this even more critical, as they completely break game balance in both single-player and multiplayer contexts, making games uncompetitive and unplayable for victims. Further testing is needed to determine if other subject types (Tributary, Dominion, March, Appanage, etc.) are also affected.

---

## Steps to Reproduce

Multiple game saves have been prepared to demonstrate these issues. Save file IDs are provided for each set.

### CASE 1: Byzantium SAVES (Demonstrates Issues 1, 3, 4, 6, and 7)

Save File ID: **#f40b1288**

Load the Byzantium save files. The saves are set up with:
- Byzantium has conquered most Ottomans territories
- Byzantium released Vassal Hudavendigar and ceded all Ottomans territories to it
- Byzantium has an Alliance with Trebizond
- Turkish Secessionists Rebellion is ready to spawn (or has already spawned)
- **CRITICAL:** The rebelling territories were originally owned by Ottomans. Ottomans still exist and are the **original owner** of these territories.
- Eretnids is the culturally dominant Country for Turkish culture (this is why it gets selected for support)

**Steps to observe Issue 1** (Same-Culture Countries Forced to Join):

1. Load the Byzantium save 1337.7.2 - Byzantium HUD Revolt 100%
2. Turkish rebellion progress in Hudavendigar is set to 100% using console command
3. When Secessionists Rebels and declare Civil War, observe that Eretnids (the culturally dominant Country for Turkish culture) is automatically and forcibly called to War on the Rebel side (save: 1337.8.1 - Byzantium HUD ERE War)
4. **CRITICAL OBSERVATION:** Observe that Ottomans (the original owner of the rebelling territories) are **COMPLETELY IGNORED** by the system. Ottomans are not selected, even though they have the strongest historical claim to these territories.
5. Observe that Eretnids cannot refuse this call - it is forced into the War on day 1
6. If Eretnids has a Truce with Byzantium, observe that it receives -50 Stability penalty for "breaking Truce" even though it had no choice
7. **Note:** Eretnids is selected because it is the culturally dominant Country for Turkish culture, but players cannot predict this selection as the mechanism is hardcoded in the engine. **The system completely ignores original ownership** - Ottomans (original owner) are not selected because they are not the culturally dominant Turkish Country.

**Steps to observe Issue 3** (Vassal War Leadership):

1. Load the Byzantium save 1337.8.1 - Byzantium HUD ERE War
2. When the Civil War starts, observe that Byzantium is forcibly dragged into the War on day 1
3. Check the War panel - observe that Vassal Hudavendigar is the defensive War Leader, NOT Byzantium
4. Try to call ally Trebizond to War - observe that it's impossible because Byzantium is not the War Leader
5. Try to request military access from Ahis or Candarids - observe that it's impossible because of low opinion or `block_when_at_war = yes`
6. Observe that Byzantium cannot control peace negotiations - only the AI Vassal Hudavendigar can make peace
7. Observe that Byzantium cannot obtain military access to attack Eretnids, creating a stalemate situation

**Steps to observe Issue 4** (Inconsistency in Alliance Auto-Call):

1. Load the Byzantium save 1337.8.1 - Byzantium HUD ERE War
2. When Civil War starts, observe that Trebizond (Byzantium's ally) does NOT automatically join the War
3. Try to manually call Trebizond - observe that it's impossible because Byzantium is not the War Leader
4. This demonstrates the inconsistency: if Byzantium were the War Leader, Byzantium can ask Trebizond to join, but since the Vassal is the leader, Byzantium cannot call its ally

**Steps to observe Issue 6** (AI Vassal Suboptimal Decisions):

1. Load the Byzantium save 1339.2.1 - Byzantium HUD ERE War Stuck
2. Occupy all rebel territories and transfer control to Vassal Hudavendigar
3. Achieve 45%+ warscore
4. Wait for AI Vassal Hudavendigar to make peace
5. Observe that AI Vassal chooses White Peace instead of annexing the Secessionists, even though annexation only costs 2 Peace Offer points
6. Observe that Byzantium loses 7 Locations which become new Secessionists, becoming a Vassal of Eretnids

**Steps to observe Issue 7** (Overlord Forced to Join):

1. Load any Byzantium save during the Civil War
2. Observe that Byzantium (Overlord of Hudavendigar) is automatically dragged into the War
3. Observe that this happens through `join_defensive_wars_always` in `secessionists.txt`

### CASE 2: France SAVES - Personal Union BREAKING (Demonstrates Issue 2)

Save File ID: **#51d1fcfd**

Load the France Personal Union save files. The saves demonstrate:
- France has formed a Personal Union with Sicily (France as Senior Partner)
- France has conquered Sicilian-culture territory from Naples
- Sicilian Secessionists Rebellion is ready to spawn

**Steps to observe Issue 2** (Personal Union Breaking):

1. Load save 1337.5.1.2 - France Personal Union Sicily, No CB Naples
   - France has formed Personal Union with Sicily using console command `create_pu SIC`
   - France has declared No CB War against Naples
2. Load save 1337.6.1 - France Conquer Naples
   - France has conquered Calabria Ultra Province (Sicilian culture)
3. Load save 1337.7.1 - France Sicilian Rebel 100%
   - Sicilian rebellion progress set to 100% using console command
4. Load save 1337.8.1 - France Personal Union Broken
   - Secessionists Rebellion breaks out
   - Observe that Sicily (culturally dominant for Sicilian culture) is forcibly called to support the rebellion
   - Observe that the Personal Union between France and Sicily immediately breaks on day 1
   - Observe that Sicily is now at War with France (its former Senior Partner)

**VERIFICATION OF PROTECTION FOR OTHER SUBJECT TYPES:**

The following saves demonstrate that Vassal and Fiefdom are correctly excluded from Secessionists calls. However, other subject types have not been tested and may also be affected:

**EU5 Subject Types (Status):**
- Personal Union: **VERIFIED AFFECTED**
- Vassal: **VERIFIED EXCLUDED**
- Fiefdom: **VERIFIED EXCLUDED**
- Tributary, Dominion, March, Appanage, Sāmanta, Tǔsī, Uç Bey, etc.: NOT TESTED

### CASE 3: France SAVES - Fiefdom (No Call)

Save File ID: **#1177c64a**

1. Load save 1337.4.1 - France ANNEX Sicily, Create Fiefdom subject
   - France annexed Sicily using console command
   - Created a new Sicily Fiefdom subject
2. Load save 1337.6.1 - France Conquer Naples
   - France conquered Calabria Ultra Province (Sicilian culture)
3. Load save 1337.7.1 - France Sicilian Rebel 100%
   - Sicilian rebellion progress set to 100% using console command
4. Load save 1337.8.1 - France Sicilian Secessionists NOT calling Sicily
   - Secessionists Rebellion breaks out
   - Observe that Sicily (Fiefdom) is NOT called to support the Rebellion
   - This demonstrates that Fiefdom are correctly excluded

### CASE 4: France SAVES - Vassal (No Call)

Save File ID: **#dae84461**

1. Load save 1337.4.1 - France ANNEX Sicily, Create Vassal subject
   - France annexed Sicily using console command
   - Created a new Sicily Vassal subject
2. Load save 1337.6.10 - France Conquer Naples
   - France conquered Calabria Ultra Province (Sicilian culture)
3. Load save 1337.7.1 - France Sicilian Rebel 100%
   - Sicilian rebellion progress set to 100% using console command
4. Load save 1337.8.1 - France Secessionists NOT calling Sicily Vassal
   - Secessionists Rebellion breaks out
   - Observe that Sicily (Vassal) is NOT called to support the Rebellion
   - This demonstrates that Vassal are correctly excluded

**CONCLUSION:** Personal Union are VERIFIED to receive Secessionists calls, while Vassal and Fiefdom are VERIFIED to be excluded. This creates an inconsistency where Personal Union break while some other subject types are protected. Other subject types have not been tested and may also be affected.

### CASE 5: France SAVES - Coalition BYPASS AND Truce BREAKING (Demonstrates Issues 1, 5, and 8)

Save File ID: **#75e13d1**

Load the France save files. The saves are set up with:
- France has formed Alliance with Aragon and Rome
- France has defeated Castile in a No CB War and taken Castilian territories
- Castilian Secessionists Rebellion is ready to spawn (or has already spawned)
- Castile may be in a Coalition against France (or will join after Truce ends)

**Steps to observe Issue 1** (Same-Culture Countries Forced to Join):

1. Load the France save 1338.4.1 - France Truce with Castile, Add Castilian Rebel to 100%
2. Console command is used to force Castilian rebellion progress to 100%
3. When Secessionists Rebels and declare Civil War, observe that Castile is automatically and forcibly called to War on the Rebel side
4. Observe that Castile cannot refuse this call - it is forced into the War

**Steps to observe Issue 8** (Truce Breaking):

1. Load the France save 1338.4.1 - France Truce with Castile, Add Castilian Rebel to 100%
2. If Castile and France have a Truce when the rebellion War begins, observe that Castile receives -50 Stability penalty for "breaking Truce" in the next month when rebellion breaks out
3. Observe that Castile is judged as actively breaking the Truce even though it was forcibly called to War and had no choice

**Steps to observe Issue 5** (Coalition Bypass):

1. Load the France save 1346.8.2 - France Castile Joint Coalition, Add Castilian Rebel to 100%
2. Castile has joined a Coalition against France
3. When Secessionists Rebels, observe that Castile is forcibly called to War as the Attacker (Rebel side)
4. Observe that Castile cannot call Coalition members (Portugal, Navarre) to join the War because it is the attacker
5. Observe that France (Defender) CAN call Allies (Aragon, Rome) to join the defensive War
6. Result: France can bypass the Coalition to attack Castile alone, while Castile cannot obtain Coalition or ally support
7. Observe that Castile will leave the Coalition after the rebellion War ends

### CASE 6: France vs Castile LANDLESS REBEL BEHAVIOR (Demonstrates Issue 12 - VERIFIED)

Save File ID: **#39d23961**

Load the France vs Castile landless rebel save files. The saves demonstrate:
- Castile has conquered Loudun (French culture territory)
- French Secessionists Rebellion spawns as Society of Pops (<50% French culture population)
- France (culturally dominant) is forcibly called to support the Rebellion

**Steps to observe Issue 12** (Landless Rebel Behavior):

1. Load save 1337.5.1 - Castile Conquer Loudun, add revolt 100%
   - Play as Castile
   - Console: `conquer loudun`
   - Add rebel progress to 1 (100%)
2. Load save 1337.6.1 - France Join Secessionist War, Occupy Castile, 1 - No Land Transferred to AAA00
   - Play as France
   - Console: `occupy_country CAS` to gain 100% Warscore
   - Observe: No land is transferred to newly spawned tag AAA00 (a landless rebel)
3. Load save 1337.6.1 - France Join Secessionist War, Occupy Castile, 2 - Loudun Transferred to AAA00
   - Play as France
   - Console: `occupy_country CAS` to gain 100% Warscore
   - Observe: Loudun is transferred to newly spawned tag AAA00 (a landless rebel)
4. Load save 1337.8.1 - France 1. Set Cash 0
   - 2 months after save 2
   - Set cash to 0 to check gained War Reparations
5. Load save 1337.8.9 - France 1. Gain 350.72 War Reparation, AAA00 Becomes Landless Secessionist
   - 9 days after save 2
   - AAA00 makes peace with Castile
   - Observe: France gained 350.72 ducats, and AAA00 is a landless Secessionists Vassal with 1 unit of cavalry (5 people)
6. Load save 1337.8.1 - France 2. Set Cash 0
   - 2 months after save 3
   - Set cash to 0 to check gained War Reparations
7. Load save 1337.8.11 - France 2. Gain 314.82 War Reparation, AAA00 Becomes landed INDEPENDENT
   - 11 days after save 2
   - AAA00 made peace with Castile
   - Observe: France gained 314.82 ducats, and AAA00 is now a settled country but **NOT a Secessionists Vassal** of France

**CRITICAL OBSERVATIONS:**
- Two major powers (France and Castile) fight a War for essentially nothing
- Best case scenario with 100% Warscore: France gains only 314.82 to 350.72 ducats (War Reparations)
- The landless rebel (AAA00) either:
  1. Becomes a landless Secessionists Vassal with 1 unit of cavalry (5 people) - effectively useless
  2. Becomes a settled country but **NOT a Secessionists Vassal** of France - completely independent
- This demonstrates that the secessionist concept completely falls apart for landless rebels - there is almost no benefit to fighting in a secessionist War for a landless secessionist because the supposedly overlord France gains almost nothing, and the landless rebel does NOT become France's useful Vassal if it becomes settled

### CASE 7: Castile SAVES - ATTACKER ALLY CALLING (Demonstrates Issue 4 - VERIFIED)

Save File ID: **#db345ae3**

Load the Castile save files. The saves demonstrate:
- Castile has Alliance with Portugal
- Castile has conquered Loudun (French culture territory)
- French Secessionists Rebellion spawns as Society of Pops (<50% French culture population)
- France (culturally dominant) is forcibly called to support the Rebellion
- France has Alliance with Aragon and Rome
- France has 100 Favors with Aragon and Rome

**Steps to observe Issue 4** (Attacker Ally Calling Using Favors):

1. Load save 1337.5.1 - Castile Ally Portugal, Conquer Loudun, Add Revolt 100%
   - Play as Castile
   - Castile has Alliance with Portugal
   - Console: `conquer loudun`
   - Add rebel progress to 1 (100%)
2. Load save 1337.5.1 - France Ally Rome, Aragon, Favor 100
   - Tag to play as France
   - France allied with Aragon, Rome
   - Use console command `favor ARA 100` and `favor PAP 100` to add Favor
3. Load save 1337.6.1 - France Secessionist War
   - Secessionists Rebel breaks out on 1337.6.1
   - Society of Pops Rebel in Loudun is the War Leader, hence France cannot call Aragon and Rome to arms
4. Load save 1337.6.3 - France Killed Landless Rebel, CAN Call Aragon and Rome
   - Use console command `kill_unit` to kill troops of Society of Pops Secessionists Rebel
   - France becomes War Leader
   - France can call Aragon and Rome to join War using Favors

---

## Game Saves Provided

Multiple game saves have been prepared and attached:

**Byzantium Saves** (Issues 1, 3, 4, 6, 7):
Save File ID: **#f40b1288**
- 1337.7.2 - Byzantium HUD Revolt 100%
- 1337.8.1 - Byzantium HUD ERE War
- 1339.2.1 - Byzantium HUD ERE War Stuck
- Additional saves demonstrating various scenarios

**France Personal Union Saves** (Issue 2 - VERIFIED):
Save File ID: **#51d1fcfd**
- 1337.5.1.2 - France Personal Union Sicily, No CB Naples
- 1337.6.1 - France Conquer Naples
- 1337.7.1 - France Sicilian Rebel 100%
- 1337.8.1 - France Personal Union Broken

**France Fiefdom Saves** (Verification - No Call):
Save File ID: **#1177c64a**
- 1337.4.1 - France ANNEX Sicily, Create Fiefdom subject
- 1337.6.1 - France Conquer Naples
- 1337.7.1 - France Sicilian Rebel 100%
- 1337.8.1 - France Sicilian Secessionists NOT calling Sicily

**France Vassal Saves** (Verification - No Call):
Save File ID: **#dae84461**
- 1337.4.1 - France ANNEX Sicily, Create Vassal subject
- 1337.6.10 - France Conquer Naples
- 1337.7.1 - France Sicilian Rebel 100%
- 1337.8.1 - France Secessionist NOT calling Sicily Vassal

**France Coalition/Truce Saves** (Issues 1, 5, 8):
Save File ID: **#75e13d1**
- 1338.4.1 - France Truce with Castile, Add Castilian Rebel to 100%
- 1346.8.2 - France Castile Joint Coalition, Add Castilian Rebel to 100%
- Additional saves demonstrating various scenarios

**France vs Castile Landless Rebel Saves** (Issue 12 - VERIFIED):
Save File ID: **#39d23961**
- 1337.5.1 - Castile Conquer Loudun, add revolt 100%
- 1337.6.1 - France Join Secessionist War, Occupy Castile, 1 - No Land Transferred to AAA00
- 1337.6.1 - France Join Secessionist War, Occupy Castile, 2 - Loudun Transferred to AAA00
- 1337.8.1 - France 1. Set Cash 0
- 1337.8.9 - France 1. Gain 350.72 War Reparation, AAA00 Becomes Landless Secessionist
- 1337.8.1 - France 2. Set Cash 0
- 1337.8.11 - France 2. Gain 314.82 War Reparation, AAA00 Becomes landed INDEPENDENT

**Castile Saves - Attacker Ally Calling** (Issue 4 - VERIFIED):
Save File ID: **#db345ae3**
- 1337.5.1 - Castile Ally Portugal, Conquer Loudun, Add Revolt 100%
- 1337.5.1 - France Ally Rome, Aragon, Favor 100
- 1337.6.1 - France Secessionist War
- 1337.6.3 - France Killed Landless Rebel, CAN Call Aragon and Rome

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
- `game/in_game/common/wargoals/00_default.txt` (lines 89-99: `take_country_nationalist` - `type = take_country`, `conquer_cost = 0.25`, `subjugate_cost = 0.25` - low warscore costs for Secessionists Wars, intended for annexing Rebels but exploitable when war participant Country becomes War Leader due to Issue 10. The `type = take_country` allows annexing the entire Country via Annex Revolter button)
- `game/main_menu/localization/english/war_overview_l_english.yml` (line 8: `WAR_LATERALVIEW_ANNEX_REVOLTER: "Annex Revolter"` - peace option that allows annexing revolter countries)
- `game/main_menu/localization/english/diplomacy_l_english.yml` (lines 1149-1150: `PEACE_TREATY_ANNEX_REVOLTER` - peace treaty description for annexing revolters)
- `game/loading_screen/common/defines/00_defines.txt` (line 1673: `MIN_WARSCORE_TO_DEMAND = 10` - minimum war score required to enable peace options like Annex Revolter; lines 1832-1833: `PEACE_TREATY_ANNEX_REVOLTER_MAX_COST = 70`, `PEACE_TREATY_REVOLTER_SURVIVES_MAX_COST = 70`)
