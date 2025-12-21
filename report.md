# Bug Report: $secessionists$ Mechanism

## Metadata

**Title:** $secessionists$ Mechanism - Forced Participation, Original Owner Ignored, World War Escalation, and Exploits

**Issue Type:** Gameplay

**Bug Type:** AI (non-player characters), Other

**Game Version:** 1.0.10

---

## Summary

The $secessionists$ mechanism has critical design flaws that create game-breaking exploits. Same-culture $game_concept_countries$ are forcibly called to $game_concept_war$ without player control, **completely ignoring original ownership** in favor of cultural dominance. The mechanism breaks $game_concept_personal_union$, prevents player control when $vassal$s lead $game_concept_wars$, incorrectly applies $game_concept_truce$ penalties, and allows the $WAR_LATERALVIEW_ANNEX_REVOLTER$ button to annex entire war participant $game_concept_countries$ (e.g., $FRA$) with no $game_concept_antagonism$. Asymmetric $game_concept_alliance$ calling creates dangerous world $game_concept_war$ escalation where $game_concept_defender$s can coordinate while forcibly called $game_concept_attacker$s cannot. Landless $secessionists$ $game_concept_rebellions$ ($game_concept_society_of_pops$) produce inconsistent outcomes where culturally dominant $game_concept_countries$ gain nothing for fighting, unlike land-based $game_concept_rebellions$ which become $vassal$s. These flaws enable systematic exploits: (1) $vassal$ players can use $game_concept_overlords$ as weapons to attack same-culture $game_concept_countries$ (VERIFIED); (2) Players can attack opponents during $game_concept_truce$ periods by triggering $secessionists$ $game_concept_rebellions$ that force victims to join and receive -50 $game_concept_stability$ penalties.

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

**VERIFICATION:** Tested with $FRA$ ($game_concept_senior_partner$) and $SIC$ ($game_concept_junior_partner$). When Sicilian $secessionists$ $game_concept_rebel$led in $FRA$-controlled territory, $SIC$ was forcibly called, immediately breaking the $game_concept_personal_union$. When $SIC$ was a $vassal$ or $fiefdom$ instead, it was correctly excluded.

### Issue 3: $vassal$ $game_concept_war$ Leadership Problem (Severe)

If a player creates a $vassal$ that becomes the $game_concept_defender$ in a $game_concept_civil_war$, the player is forcibly dragged into the $game_concept_war$ but does not become the $game_concept_war_leader$. The AI $vassal$ controls all peace negotiations. The player cannot call $game_concept_allies$, cannot obtain $game_concept_military_access$, and must bear all $game_concept_war$ costs without control.

### Issue 4: Inconsistency in $game_concept_alliance$ Auto-Call

When the player is the defensive $game_concept_war_leader$, $game_concept_allies$ can be asked to join. But when a $vassal$ is the defensive $game_concept_war_leader$, the player cannot call $game_concept_allies$ to join.

### Issue 5: $game_concept_coalition$ Bypass Mechanism (Severe Exploit)

When a forcibly called $game_concept_country$ is in a $game_concept_coalition$, it cannot call $game_concept_coalition$ members to join the $game_concept_war$ because it is the $game_concept_attacker$. This allows the $game_concept_defender$ to bypass $game_concept_coalition$ restrictions and attack $game_concept_coalition$ members individually.

### Issue 6: AI $vassal$ Suboptimal Peace Decisions

AI $vassal$ may choose white peace instead of annexing the $game_concept_revolter$ even when annexation cost is very low, causing players to lose territories they should have gained.

### Issue 7: Overlord Forced to Join

$secessionists$ automatically and forcibly call $game_concept_overlords$ to $game_concept_war$ through `join_offensive_wars_always` and `join_defensive_wars_always` in `secessionists.txt`.

### Issue 8: $game_concept_truce$ Breaking Penalties

Forcibly called $game_concept_countries$ with $game_concept_truce$ are incorrectly judged as actively breaking $game_concept_truce$, triggering severe penalties even though they had no choice.

### Issue 9: World War Escalation - Asymmetric $game_concept_alliance$ Calling (Severe)

A single local $secessionists$ $game_concept_rebellion$ can escalate into a world $game_concept_war$ due to asymmetric $game_concept_alliance$ calling mechanics. This is **worse than historical world $game_concept_wars$** (WW1/WW2) where both sides could coordinate with $game_concept_allies$.

**Current EU5 System - Worse Than Historical World Wars:**

When a $secessionists$ $game_concept_rebellion$ breaks out:
- The $game_concept_defender$ CAN call all its $game_concept_allies$ to join the defensive $game_concept_war$
- The forcibly called same-culture $game_concept_country$ ($game_concept_attacker$) CANNOT call its own $game_concept_allies$ because it is the $game_concept_attacker$
- Result: A local $game_concept_rebellion$ escalates into a major $game_concept_war$ where one $game_concept_alliance$ bloc fights alone against another bloc's coordinated response

*Example Scenario (Late 18th Century - Partition Era):*
- Two major $game_concept_alliance$ blocs: Bloc A ($RUS$, $HAB$, $PRU$) vs Bloc B ($FRA$, $TUR$, $SWE$)
- Polish $secessionists$ $game_concept_rebellion$ breaks out in $RUS$-controlled territory
- $POL$ (culturally dominant) is FORCED to join $game_concept_rebellion$'s side ($game_concept_attacker$) - NO CHOICE
- $RUS$ ($game_concept_defender$) calls $HAB$ and $PRU$ → 3 major powers vs $POL$
- $POL$ ($game_concept_attacker$) cannot call $FRA$, $TUR$, or $SWE$ → fights alone against 3 major powers

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
11. **$ENG$ annexes entire $FRA$** at low warscore cost (intended only for annexing the $game_concept_revolter$, not major powers like $FRA$)
12. **No $game_concept_antagonism$ is generated** - annexing a major power like $FRA$ should generate massive $game_concept_antagonism$, but the peace option generates none

**Technical Details:**
- When $game_concept_rebellion$ $game_concept_province$ has <50% population of rebelling culture, $game_concept_rebellion$ spawns as `country_type = pop` ($game_concept_society_of_pops$ - landless with no territory)
- When `country_type = pop` troops are eliminated, the $game_concept_rebellion$ is removed from the $game_concept_war$
- The war participant $game_concept_country$ becomes $game_concept_war_leader$ automatically
- **BUG:** The `take_country_nationalist` war goal has `type = take_country`, which allows annexing the entire $game_concept_country$ via $WAR_LATERALVIEW_ANNEX_REVOLTER$ button
- **BUG:** No $game_concept_antagonism$ is generated when annexing the war participant $game_concept_country$

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
- When the $game_concept_rebellion$ wins independence, it becomes an **INDEPENDENT** $game_concept_country$, NOT a $vassal$ of the culturally dominant $game_concept_country$
- The culturally dominant $game_concept_country$ gains **NOTHING** for fighting the $game_concept_war$ (or minimal $game_concept_war_reparations$ at 100% $game_concept_war_score$)
- The AI culturally dominant $game_concept_country$ may not take any land even at 100% $game_concept_war_score$, only accepting $game_concept_war_reparations$

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
- The AI may only accept $game_concept_war_reparations$ or white peace, leaving the $game_concept_defender$ with all territory intact
- This is inconsistent with land-based $game_concept_rebellions$ where the culturally dominant $game_concept_country$ gains a $vassal$

**Why This Is Problematic:**
- **Inconsistent outcomes:** Same mechanism produces different results based on population size
- **No reward for culturally dominant $game_concept_country$:** Landless $game_concept_rebellions$ provide no benefit to the culturally dominant $game_concept_country$ that is forced to fight
- **AI suboptimal behavior:** AI may not take land even when it should, wasting $game_concept_war_score$
- **Breaks intended design:** The $secessionists$ mechanism is supposed to reward culturally dominant $game_concept_countries$ with $vassal$s, but landless $game_concept_rebellions$ break this design
- **Unfair to players:** Players who are forcibly called into landless $game_concept_rebellion$ $game_concept_wars$ gain nothing for their effort

**Technical Details:**
- When $game_concept_province$ has <50% population of rebelling culture, $game_concept_rebellion$ spawns as `country_type = pop` ($game_concept_society_of_pops$ - landless with no territory)
- When `country_type = pop` wins independence, it becomes an independent $game_concept_country$, not a $vassal$
- The `secessionists.txt` subject type definition does not have `ai_wants_to_be_overlord`, which may explain why AI doesn't try to make landless rebels into $vassal$s
- The `PEACE_MAKE_SUBJECT_REVOLT_WAR = 0` define in `00_defines.txt` indicates AI has no interest in making subjects in revolt $game_concept_wars$, which explains suboptimal peace decisions

**Impact:**
- Culturally dominant $game_concept_countries$ gain nothing from supporting landless $game_concept_rebellions$
- Players are forced to fight $game_concept_wars$ with no reward
- AI makes suboptimal peace decisions, not taking land even at 100% $game_concept_war_score$
- Breaks the intended design of the $secessionists$ mechanism where culturally dominant $game_concept_countries$ should gain $vassal$s

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
4. Player A makes white peace or gets a small amount of ransom, ending War 2 quickly, creating a new $game_concept_truce$ period
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
- $game_concept_personal_union$ break unexpectedly due to forced participation
- Players receive severe penalties (-50 $game_concept_stability$) for actions they did not choose
- **$game_concept_rebellions$ that spawn as $game_concept_society_of_pops$ cause warscore costs to be applied to wrong target** - When $game_concept_society_of_pops$ troops are wiped out and $game_concept_rebellion$ is eliminated, the $game_concept_war$ continues with the war participant $game_concept_country$ as $game_concept_war_leader$. The low warscore costs (intended only for annexing $game_concept_rebels$) remain active, allowing the $game_concept_defender$ to annex the war participant $game_concept_country$ at 25% cost instead of just the $game_concept_rebels$. This allows major powers like $FRA$ (3rd largest $game_concept_country$ at game start) to be annexed by smaller $game_concept_countries$ like $ENG$
- The mechanism can be systematically exploited by knowledgeable players in multiple ways (see EXPLOITATION SCENARIOS section)
- The mechanism violates historical rationality
- Players lose territories and strategic control due to AI decisions
- Inconsistency: $game_concept_personal_union$ are treated differently from other subject types ($vassal$, $fiefdom$ are protected, $game_concept_personal_union$ are not)
- **EXPLOIT 1:** $vassal$ players can use their $game_concept_overlords$ as weapons to attack same-culture $game_concept_countries$ and eventually declare independence, breaking intended gameplay balance
- **EXPLOIT 2:** Players can attack opponents (AI or human) during a $game_concept_truce$ period by intentionally creating a $game_concept_truce$ period through War 2, then triggering $secessionists$ $game_concept_rebellion$ which forces the victim to join and receive -50 $game_concept_stability$ penalties, preventing the victim from calling $game_concept_allies$. This completely breaks game balance in both single-player and multiplayer contexts
- **WORLD $game_concept_war$ ESCALATION:** A single local $game_concept_rebellion$ can escalate into a world $game_concept_war$ due to asymmetric $game_concept_alliance$ calling. The $game_concept_defender$ can call all $game_concept_allies$, while forcibly called $game_concept_attacker$ cannot call its own $game_concept_allies$, creating a dangerous escalation mechanism
- **SMALL SUBJECTS DRAGGED INTO DISTANT $game_concept_wars$:** Any subject (e.g., $vassal$, $secessionists$ subject) that happens to be culturally dominant is forcibly called to support distant rebellions of the same culture, even when thousands of kilometers away. Players lose control over their subjects' foreign policy and must investigate to find out why their subjects are at $game_concept_war$ (VERIFIED: $YUA$ case - small Mongolian subject in Mongolia region forced to support rebellion in Georgia 6000+ km away)
- **INCONSISTENT BEHAVIOR OF LAND-BASED VS. LANDLESS $secessionists$:** Land-based $secessionists$ $game_concept_rebellions$ (>50% cultural population) become $vassal$s of the culturally dominant $game_concept_country$ when they win, while landless $secessionists$ $game_concept_rebellions$ (<50% cultural population - $game_concept_society_of_pops$) become independent $game_concept_countries$ with no reward for the culturally dominant $game_concept_country$. This creates inconsistent outcomes where culturally dominant $game_concept_countries$ gain nothing from fighting landless $game_concept_rebellion$ $game_concept_wars$, and AI may not take land even at 100% $game_concept_war_score$ (VERIFIED: $ENG$ vs $FRA$ case - $FRA$ fought entire $game_concept_war$ and gained only minimal $game_concept_war_reparations$, no territory or $vassal$)

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
5. Observe that AI $vassal$ chooses white peace instead of annexing the Secessionists, even though annexation only costs 2 peace offer points
6. Observe that $BYZ$ loses 7 locations which become new Secessionists, becoming a $vassal$ of $ERE$

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
