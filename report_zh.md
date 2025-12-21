# Bug Report: 归并国 Mechanism

## Metadata

**Title:** 归并国 Mechanism - Forced Participation, Original Owner Ignored, World War Escalation, and Exploits

**Issue Type:** Gameplay

**Bug Type:** AI (non-player characters), Other

**Game Version:** 1.0.10

---

## Summary

归并国机制存在严重的设计缺陷，导致游戏破坏性利用。同文化国家在没有玩家控制的情况下被强制召唤参战，**完全忽略原始所有权**，优先考虑文化主导地位。该机制破坏联合统治，创造不一致的战争领导权逻辑（附庸在归并国内战中领导，与正常宣战中宗主国领导不同），在附庸领导战争时阻止玩家控制，错误地应用停战协议惩罚，并允许通过吞并叛军按钮吞并整个参战国（例如，法兰西）而不产生敌意。不对称的同盟召唤创造了危险的世界战争升级，防御方可以协调，而被强制召唤的进攻方无法协调。无地归并国叛乱（群众集会）产生不一致的结果，文化主导国家战斗后几乎一无所获（即使达到100%战争分数也仅获得314-350杜卡特），不像有地叛乱会成为附庸。这些缺陷使系统性利用成为可能：（1）附庸玩家可以使用宗主国作为武器攻击同文化国家（已验证）；（2）玩家可以在停战协议期间通过触发归并国叛乱来攻击对手，迫使受害者加入并受到-50稳定度惩罚。

---

## Description

The 归并国 mechanism has twelve critical issues:

### Issue 1: Same-Culture Countries Forced to Join (Most Severe)

When 归并国叛军s, the system automatically searches for nearby same-culture 国家 and forcibly calls them to 战争 on the 叛军 side. Called 国家 cannot refuse. If they have a 停战协议 with the enemy, they are incorrectly judged as actively breaking the 停战协议, triggering severe penalties (-50 稳定度, +1 厌战度, +25 敌意). The selection mechanism is hardcoded in the engine and cannot be found in game files, making it impossible for players to predict which country will be called.

**CRITICAL FLAW: Original Ownership Completely Ignored**

The system **completely ignores original ownership** when selecting which same-culture 国家 to call, prioritizing cultural dominance over historical claims.

*Example from 拜占庭 save:* Turkish 归并国叛乱 occurs in territories originally owned by 奥斯曼. 奥斯曼 still exist and are the original owner. However, the system selects 埃雷特纳 (culturally dominant Turkish 国家) instead, completely ignoring 奥斯曼. 奥斯曼 have no opportunity to reclaim their former territories, even though they have the strongest historical claim.

### Issue 2: 联合统治 Breaking (Severe - VERIFIED)

When a 联合统治邦联从属国 is the culturally dominant 国家 for a 归并国叛乱, it is forcibly called to support the 叛乱 against its 邦联主导国. This immediately breaks the 联合统治 because `union_of_crowns_pact` has `break_on_war = yes`.

**VERIFICATION STATUS:**
- 联合统治: **VERIFIED AFFECTED** (tested with 法兰西-西西里)
- 附庸: **VERIFIED EXCLUDED** (tested with 法兰西-西西里)
- 采邑: **VERIFIED EXCLUDED** (tested with 法兰西-西西里)
- Other subject types (朝贡国, 自治领, 卫戍国, 封邑, 三曼多, 土司, 乌奇贝伊, etc.): NOT TESTED

This creates an inconsistency where 联合统治 are vulnerable while some other subject types are protected.

**VERIFICATION:** Tested with 法兰西 (邦联主导国) and 西西里 (邦联从属国). When Sicilian 归并国叛军led in 法兰西-controlled territory, 西西里 was forcibly called, immediately breaking the 联合统治. When 西西里 was a 附庸 or 采邑 instead, it was correctly excluded.

### Issue 3: 附庸战争 Leadership Problem (Severe)

如果玩家创建的附庸在内战中成为防御方，玩家会被强制拖入战争但不会成为战争领袖。AI附庸控制所有和平谈判。玩家无法召唤同盟，无法获得军事通行权，并且必须在没有控制权的情况下承担所有战争成本。

**关键不一致性：正常宣战 vs. 归并国内战**

在正常宣战和归并国内战之间，关于附庸战争领导权的游戏逻辑存在根本性不一致：

**正常宣战（使用宣战理由和外交官）：**
- 当国家向附庸宣战时（例如，拜占庭向阿尔巴尼亚宣战，而阿尔巴尼亚是那不勒斯的附庸），宗主国（那不勒斯）成为战争领袖，而不是附庸（阿尔巴尼亚）
- 宗主国控制所有和平谈判和战争决策
- 这是正常宣战的预期行为

**归并国内战（问题3）：**
- 当附庸在归并国内战中成为防御方时，附庸成为战争领袖，而不是宗主国
- 宗主国被强制拖入战争但不控制和平谈判
- AI附庸控制所有和平谈判，而宗主国没有控制权

**不一致性：**
- **正常宣战：** 当附庸被攻击时，宗主国是战争领袖
- **归并国内战：** 当附庸是防御方时，附庸是战争领袖
- 这创造了不一致的游戏逻辑，相同的（宗主国-附庸）关系根据战争开始方式产生不同的战争领导权结果
- 玩家无法仅根据关系结构预测或理解哪一方将控制战争

**影响：**
- 对期望一致行为的玩家造成困惑
- 无法根据属国关系预测战争领导权
- 破坏了不同战争类型之间的游戏逻辑一致性
- 在归并国内战中，宗主国失去了在常规战争中通常拥有的控制权

### Issue 4: Inconsistency in 同盟 Auto-Call

When the player is the defensive 战争领袖, 同盟 can be asked to join. But when a 附庸 is the defensive 战争领袖, the player cannot call 同盟 to join.

### Issue 5: 包围网 Bypass Mechanism (Severe Exploit)

When a forcibly called 国家 is in a 包围网, it cannot call 包围网 members to join the 战争 because it is the 进攻方. This allows the 防御方 to bypass 包围网 restrictions and attack 包围网 members individually.

### Issue 6: AI 附庸 Suboptimal Peace Decisions

AI 附庸 may choose white peace instead of annexing the 叛乱者 even when annexation cost is very low, causing players to lose territories they should have gained.

### Issue 7: Overlord Forced to Join

归并国 automatically and forcibly call 宗主国 to 战争 through `join_offensive_wars_always` and `join_defensive_wars_always` in `secessionists.txt`.

### Issue 8: 停战协议 Breaking Penalties

Forcibly called 国家 with 停战协议 are incorrectly judged as actively breaking 停战协议, triggering severe penalties even though they had no choice.

### Issue 9: World War Escalation - Asymmetric 同盟 Calling (Severe)

A single local 归并国叛乱 can escalate into a world 战争 due to asymmetric 同盟 calling mechanics. This is **worse than historical world 战争** (WW1/WW2) where both sides could coordinate with 同盟.

**Current EU5 System - Worse Than Historical World Wars:**

When a 归并国叛乱 breaks out:
- The 防御方 CAN call all its 同盟 to join the defensive 战争
- The forcibly called same-culture 国家 (进攻方) CANNOT call its own 同盟 because it is the 进攻方
- Result: A local 叛乱 escalates into a major 战争 where one 同盟 bloc fights alone against another bloc's coordinated response

*Example Scenario (Late 18th Century - Partition Era):*
- Two major 同盟 blocs: Bloc A (俄罗斯, 奥地利, 普鲁士) vs Bloc B (法兰西, 奥斯曼, 瑞典)
- Polish 归并国叛乱 breaks out in 俄罗斯-controlled territory
- 波兰 (culturally dominant) is FORCED to join 叛乱's side (进攻方) - NO CHOICE
- 俄罗斯 (防御方) calls 奥地利 and 普鲁士 → 3 major powers vs 波兰
- 波兰 (进攻方) cannot call 法兰西, 奥斯曼, or 瑞典 → fights alone against 3 major powers

This creates a dangerous escalation mechanism where the 防御方 can systematically use 归并国叛乱 to attack their enemies' 同盟 blocs asymmetrically.

### Issue 10: 吞并叛军 Button Misapplied to War Participant 国家 (Severe - GAME-BREAKING)

When a 归并国叛乱 spawns as a 群众集会 (landless 国家 with no territory) due to low population size (省份 has <50% of the rebelling culture's population), the 叛军国家 has no territory. When the 叛军国家's troops are wiped out and the 叛乱 is eliminated, the 战争 continues and a war participant 国家 becomes the 战争领袖. The 吞并叛军 button, designed to annex the entire 叛乱者国家, can then be used to annex the entire war participant 国家 (e.g., 法兰西 - 3rd largest 国家 at game start) with no 敌意 generated.

**Exploitation Flow (英格兰 vs 法兰西 Example):**

1. **英格兰 spawns a French 叛乱 that becomes a 群众集会** (防御方 - owns rebelling territory with French culture)
2. **法兰西 is forcibly called** as 进攻方 (culturally dominant for French culture)
3. **战争 structure:** 英格兰 (防御方/战争领袖) vs 叛乱 (进攻方/战争领袖) + 法兰西 (进攻方/war participant)
4. **英格兰 wipes out 群众集会 troops** (叛乱 eliminated)
5. **战争 continues** because 法兰西 is still in the 战争
6. **法兰西 becomes 战争领袖** (only remaining 进攻方)
7. **英格兰 gains 战争分数** through ticking 战争分数 (`ticking_war_score = 0.5` per month from `take_country_nationalist` war goal) and by winning battles, sieging enemy 地点, and establishing blockades
8. **英格兰 reaches 10 战争分数** - The 吞并叛军 button becomes enabled when 英格兰 reaches at least 10 战争分数 (`MIN_WARSCORE_TO_DEMAND = 10` from game defines)
9. **吞并叛军 button becomes available** - Once enabled, this peace option can be used on the war participant 国家 (法兰西) instead of the eliminated 叛乱者
10. **英格兰 uses 吞并叛军 button** to annex the entire 法兰西国家
11. **英格兰 annexes entire 法兰西** at low warscore cost (intended only for annexing the 叛乱者, not major powers like 法兰西)
12. **No 敌意 is generated** - annexing a major power like 法兰西 should generate massive 敌意, but the peace option generates none

**Technical Details:**
- When 叛乱省份 has <50% population of rebelling culture, 叛乱 spawns as `country_type = pop` (群众集会 - landless with no territory)
- When `country_type = pop` troops are eliminated, the 叛乱 is removed from the 战争
- The war participant 国家 becomes 战争领袖 automatically
- **BUG:** The `take_country_nationalist` war goal has `type = take_country`, which allows annexing the entire 国家 via 吞并叛军 button
- **BUG:** No 敌意 is generated when annexing the war participant 国家

**Impact:**
- **GAME-BREAKING:** Major powers like 法兰西 (3rd largest 国家 at game start) can be annexed by smaller 国家 like 英格兰 through this exploit
- **VERIFIED EXAMPLE 1:** 法兰西 was annexed by 英格兰 through this mechanism - 英格兰 spawned a 叛乱 that became a 群众集会, 法兰西 was forcibly called, 英格兰 eliminated 叛军, gained 10 战争分数 (minimum required), then used 吞并叛军 button to annex entire 法兰西, and no 敌意 was generated
- **VERIFIED EXAMPLE 2:** 札剌亦儿 was annexed by 奥斯曼 through this mechanism - 奥斯曼 had very few Iraqi culture pops in Gumushane location (only 31 Iraqi culture nobles rebelling, <50% of location population), spawning a 叛乱 that became a 群众集会 with only 3 soldiers instead of making the location rebel territory. Because it was an Iraqi culture rebellion, 札剌亦儿 (Iraqi culture dominant 国家) was forcibly called. 奥斯曼 eliminated the 3 soldiers (群众集会 disappeared), 札剌亦儿 became 战争领袖, 奥斯曼 gained 10% 战争分数 by sieging a few forts, then used 吞并叛军 button to annex entire 札剌亦儿. The annexed land was unintegrated territory requiring manual coring

### Issue 11: Subjects Forced to Support Distant Rebellions (Severe)

Any subject (e.g., 附庸, 归并国 subject, or other subject types) that happens to be culturally dominant for the rebelling culture is forcibly called to support distant rebellions of the same culture, even when they are thousands of kilometers away and have no logical connection to the rebellion.

**The Problem:**
- When a 归并国叛乱 breaks out, the system searches for culturally dominant 国家 to call
- The system does NOT check if the 国家 is a subject, its size, or distance to the 叛乱
- Any 国家 with the same culture as the rebelling population can be called, regardless of whether it is a subject
- Result: Small subjects (e.g., 附庸s, 归并国 subjects) get forcibly called to support 叛乱 thousands of kilometers away

**VERIFIED EXAMPLE:**
- Player (元 / Ming) has a small Mongolian culture subject (e.g., 附庸 or 归并国 subject) in Mongolia region
- A Mongolian 归并国叛乱 breaks out in Georgia (6000+ km away)
- The small subject is FORCED to join the 叛乱's side (进攻方) - NO CHOICE
- Player's subject gets dragged into a 战争 far away, player has no control
- Player must search through war participant 国家 to find out why their subject is at 战争
- The subject receives penalties (停战协议 breaking, 厌战度, 敌意) even though it had no choice
- The 宗主国 (元) is also dragged into the 战争 (Issue 7), compounding the problem

**Why This Is Problematic:**
- **No logical connection:** A small subject in Mongolia has no reason to support a 叛乱 in Georgia
- **No player control:** Player cannot prevent their subject from being called
- **Hidden cause:** Player must investigate to find out why their subject is at 战争
- **Penalties without choice:** Subject receives severe penalties for actions it did not choose
- **宗主国 dragged in:** 宗主国 is also forced to join (Issue 7), creating a cascade of forced participation
- **Exploitable:** 防御方s can use this to force small subjects and their 宗主国 into 战争

**Technical Details:**
- The hardcoded mechanism in `on_civil_war_start` does not check `is_subject = yes` or subject type
- The system only checks cultural dominance, ignoring subject status, distance, and size
- Subjects are treated the same as independent 国家 for the purpose of being called

**Impact:**
- Small subjects get dragged into distant 战争 they have no interest in
- Players lose control over their subjects' foreign policy
- Creates confusion and frustration (player must investigate why subject is at 战争)
- Enables exploits where 防御方s can force small subjects and 宗主国 into 战争

### Issue 12: Inconsistent Behavior of Land-Based vs. Landless 归并国叛乱 (Severe)

When a 归并国叛乱 spawns, its behavior differs dramatically based on whether it is land-based (owns territory) or landless (群众集会 with no territory). This creates inconsistent outcomes that break the intended 归并国 mechanism.

**The Problem:**

**Land-Based 归并国叛乱 (>50% cultural population):**
- When the 叛乱 breaks out, it controls territory
- When the 叛乱 wins independence, it becomes a 归并国附庸 of the culturally dominant 国家
- The culturally dominant 国家 gains a new 附庸 as a reward for supporting the 叛乱

**Landless 归并国叛乱 (<50% cultural population - 群众集会):**
- When the 叛乱 breaks out, it has no territory (only pops)
- When the 叛乱 wins independence, it has **TWO INCONSISTENT OUTCOMES:**
  1. **Becomes an INDEPENDENT 国家**, NOT a 附庸 of the culturally dominant 国家
  2. **Becomes a landless 归并国附庸** with no territory (only 1 unit of cavalry with 5 people) - effectively useless
- The culturally dominant 国家 gains **ALMOST NOTHING** for fighting the 战争 - only minimal 战争赔款 (314.82 to 350.72 ducats) even at 100% 战争分数
- The AI culturally dominant 国家 may not take any land even at 100% 战争分数, only accepting 战争赔款
- **CRITICAL:** The entire secessionist concept falls apart for landless rebels - culturally dominant 国家 fight major 战争 between major powers (e.g., 法兰西 vs 卡斯蒂利亚) for essentially nothing

**VERIFIED EXAMPLES:**

**Example 1 - Landless Rebel (英格兰 vs 法兰西):**
- 英格兰 spawns a French 叛乱 that becomes a 群众集会 (<50% French culture population)
- 法兰西 (culturally dominant) is forcibly called to support the 叛乱
- 法兰西 fights the 战争 and achieves 100% 战争分数
- The landless 叛乱 becomes an **INDEPENDENT** 国家 (not a 附庸 of 法兰西)
- 法兰西 receives only a portion of 战争赔款 (e.g., 238.09 ducats) despite fighting the entire 战争
- 法兰西 gains **NO TERRITORY** and **NO 附庸** despite winning the 战争 with 100% 战争分数

**Example 2 - AI Not Taking Land:**
- In some cases, the AI culturally dominant 国家 does not take any land even at 100% 战争分数
- The AI may only accept 战争赔款 or white peace, leaving the 防御方 with all territory intact
- This is inconsistent with land-based 叛乱 where the culturally dominant 国家 gains a 附庸

**Example 3 - Landless Rebel Behavior (法兰西 vs 卡斯蒂利亚 - VERIFIED):**
- 卡斯蒂利亚 conquers Loudun (French culture territory)
- French 归并国叛乱 spawns as 群众集会 (<50% French culture population)
- 法兰西 (culturally dominant) is forcibly called to support the 叛乱
- 法兰西 fights 卡斯蒂利亚 (two major powers in Europe) and achieves 100% 战争分数
- **Outcome 1:** The landless rebel (AAA00) becomes a **landless 归并国附庸** of 法兰西 with no territory, only 1 unit of cavalry (5 people) - effectively useless
- **Outcome 2:** The landless rebel becomes a **settled country** but **NOT a 归并国附庸** of 法兰西 - completely independent
- 法兰西 gains only **314.82 to 350.72 ducats** (战争赔款) despite fighting an entire major 战争 with 100% 战争分数
- 法兰西 gains **NO TERRITORY** and either **NO USEFUL 附庸** (landless vassal with 5 people) or **NO 附庸 AT ALL** (independent country)
- **Summary:** Two major powers (法兰西 and 卡斯蒂利亚) fight a 战争 for either: (1) a landless 归并国附庸 with 1 unit of cavalry (5 people), (2) an independent settled country, or (3) 314.82 to 350.72 ducats. This demonstrates that the secessionist concept completely falls apart for landless rebels - there is almost no benefit to fighting in a secessionist 战争 for a landless secessionist because the supposedly overlord 法兰西 gains almost nothing, and the landless rebel does NOT become 法兰西's useful 附庸 if it becomes settled

**Why This Is Problematic:**
- **Inconsistent outcomes:** Same mechanism produces different results based on population size
- **No meaningful reward for culturally dominant 国家:** Landless 叛乱 provide almost no benefit to the culturally dominant 国家 that is forced to fight - only minimal 战争赔款 (314-350 ducats) even at 100% 战争分数
- **Two problematic outcomes:** Landless rebels either become independent (no 附庸 reward) or become useless landless 附庸s with no territory (only 5 people in 1 unit)
- **Major 战争 for nothing:** Two major powers fight a 战争 for essentially nothing - the culturally dominant 国家 gains almost no benefit despite winning with 100% 战争分数
- **AI suboptimal behavior:** AI may not take land even when it should, wasting 战争分数
- **Breaks intended design:** The 归并国 mechanism is supposed to reward culturally dominant 国家 with useful 附庸s, but landless 叛乱 completely break this design
- **Unfair to players:** Players who are forcibly called into landless 叛乱战争 gain almost nothing for their effort
- **Secessionist concept falls apart:** The entire secessionist concept becomes meaningless for landless rebels, as they can be independent or become useless landless 附庸s

**Technical Details:**
- When 省份 has <50% population of rebelling culture, 叛乱 spawns as `country_type = pop` (群众集会 - landless with no territory)
- When `country_type = pop` wins independence, it has inconsistent outcomes:
  1. Becomes an independent 国家 (not a 附庸)
  2. Becomes a landless 归并国附庸 with no territory (only pops/units, effectively useless)
- The `secessionists.txt` subject type definition does not have `ai_wants_to_be_overlord`, which may explain why AI doesn't try to make landless rebels into useful 附庸s
- The `PEACE_MAKE_SUBJECT_REVOLT_WAR = 0` define in `00_defines.txt` indicates AI has no interest in making subjects in revolt 战争, which explains suboptimal peace decisions
- Even when landless rebels become 附庸s, they have no territory and minimal military (e.g., 1 unit of cavalry with 5 people), making them effectively useless

**Impact:**
- Culturally dominant 国家 gain almost nothing from supporting landless 叛乱 - only minimal 战争赔款 (314-350 ducats) even at 100% 战争分数
- Major powers fight major 战争 for essentially nothing (e.g., 法兰西 vs 卡斯蒂利亚 for a landless 附庸 with 5 people or an independent country)
- Players are forced to fight 战争 with almost no reward
- AI makes suboptimal peace decisions, not taking land even at 100% 战争分数
- Breaks the intended design of the 归并国 mechanism where culturally dominant 国家 should gain useful 附庸s
- The entire secessionist concept becomes meaningless for landless rebels, as they either become independent or useless landless 附庸s

---

## Exploitation Scenarios

### EXPLOIT 1: 附庸 PLAYER USING OVERLORD AS WEAPON (VERIFIED)

Players who choose to play as 附庸 can systematically use their 宗主国 as weapons to attack and conquer territories from same-culture 国家, then eventually declare independence.

**Exploitation Strategy** (VERIFIED - 拜占庭 Case):

1. Player chooses to play as a 附庸 (e.g., 拜占庭附庸 Hudavendigar)
2. Player intentionally increases 归并国叛军 monthly progress through various means (not coring territories, low 稳定度, low legitimacy, maximizing taxes, reducing control, etc.)
3. When 归并国叛乱 breaks out:
   - The 宗主国 (e.g., 拜占庭) is AUTOMATICALLY dragged into the 战争 on the defensive side (Issue 7)
   - The culturally dominant 国家 (e.g., 埃雷特纳 for Turkish culture) is FORCED to join the 叛乱's side (进攻方) (Issue 1)
   - The player's 附庸 becomes the defensive 战争领袖, giving the player full control over peace negotiations
4. Player advantages as 附庸战争 leader:
   - Can react to 叛军 on day 1, sending regulars to occupy 叛军省份 immediately
   - 宗主国 automatically sends troops to help
   - 战争 score to annex 叛军 is very low (e.g., 2 points), easily achievable
   - Any extra 战争 score can be used to demand territories/money from the forcibly called war participant 国家
5. Player controls 战争 duration based on military situation
6. Player gains all benefits from the 战争: territories and ducats from the forcibly called war participant 国家, uses 宗主国's military strength without cost, builds up power for eventual independence
7. Once strong enough, player declares independence and defeats the weakened 宗主国

**VERIFICATION:** Tested in 拜占庭 game. Player played as 拜占庭附庸 Hudavendigar. Turkish 归并国叛乱 broke out, 埃雷特纳 (culturally dominant Turkish 国家) was forcibly called. 拜占庭 (宗主国) was automatically dragged into the 战争. Player (as 附庸战争 leader) was able to sue for peace and conquer 埃雷特纳's territories.

**Impact:**
- 附庸 players can systematically exploit their 宗主国 to fight 战争 for them
- No need to fabricate claims, create casus belli, or manage 停战协议 timers
- Can build up power and resources using 宗主国's military strength
- Eventually declare independence when strong enough
- Makes 附庸 gameplay exploitable and breaks intended gameplay balance
- In multiplayer, this allows 附庸 players to systematically weaken their 宗主国 players

### EXPLOIT 2: DOUBLE WAR ATTACK

The 归并国 mechanism creates a severe exploit that allows knowledgeable players to attack opponents (AI or human players) during a 停战协议 period by intentionally creating that 停战协议 period, then triggering 归并国叛乱 which forces the victim to join and receive devastating penalties.

**Exploitation Strategy:**

1. Player A (exploiter) acquires territories with the same primary culture as Player B (victim - AI or human player) through War 1 or from game start/previous conquests
2. Player A intentionally allows or encourages 归并国叛乱 to develop (叛军 progress approaching 100%)
3. Player A wages War 2 against Player B when the 归并国叛乱 is about to break out (叛军 progress near 100%)
4. Player A makes white peace or gets a small amount of ransom, ending War 2 quickly, creating a new 停战协议 period
5. Player A forms 同盟 with other 国家/players during this new 停战协议 period
6. When 归并国叛乱 breaks out:
   - Player B (culturally dominant) is FORCED to join the 叛乱's side (进攻方)
   - Player B receives -50 稳定度 penalty for "breaking 停战协议" (even though forced)
   - Player B cannot call 同盟 or 包围网 members because it is the 进攻方
   - Player A (防御方) CAN call all 同盟 to join the defensive 战争
   - Result: Player A attacks Player B during the 停战协议 period with Player B suffering severe penalties and unable to defend effectively

**Impact:**
- This exploit completely breaks game balance
- Knowledgeable players can systematically destroy opponents (AI or human) by exploiting this mechanism
- Victim 国家 have no counterplay - they cannot prevent the 归并国叛乱, cannot refuse the call, and cannot call their own 同盟
- The -50 稳定度 penalty can cause 国家 collapse, making the victim's game unplayable
- This creates an unfair advantage that makes games uncompetitive

**Example Scenario:**
- Player (法兰西) acquires Castilian-culture territories (either through War 1 against 卡斯蒂利亚 if needed, or already owns them)
- Player allows Castilian 归并国叛乱 to develop (叛军 progress approaching 100%)
- Player declares War 2 against Opponent (卡斯蒂利亚 - AI or human player) when Castilian 叛乱 is at 95% progress
- Player quickly white peaces War 2, creating a new 停战协议 period
- Player forms 同盟 with 阿拉贡 and 罗马 during this new 停战协议 period
- Castilian 归并国叛乱 breaks out
- Opponent (卡斯蒂利亚) is forced to join 叛乱 side, receives -50 稳定度, cannot call 同盟
- Player calls 阿拉贡 and 罗马, attacks Opponent again with overwhelming advantage
- Opponent's 国家 collapses due to 稳定度 penalty and cannot effectively defend

---

## Severity

These issues are critical because:
- Players have absolutely no control over being dragged into 战争
- **Original owners are completely ignored** - The system ignores original ownership and only looks at cultural dominance (e.g., 奥斯曼 ignored in favor of 埃雷特纳 in 拜占庭 case)
- **不一致的战争领导权逻辑** - 在正常宣战中，当附庸被攻击时，宗主国成为战争领袖，但在归并国内战中，附庸反而成为战争领袖。这创造了不可预测和不一致的游戏逻辑，破坏了玩家的期望
- 联合统治 break unexpectedly due to forced participation
- Players receive severe penalties (-50 稳定度) for actions they did not choose
- **叛乱 that spawn as 群众集会 cause warscore costs to be applied to wrong target** - When 群众集会 troops are wiped out and 叛乱 is eliminated, the 战争 continues with the war participant 国家 as 战争领袖. The low warscore costs (intended only for annexing 叛军) remain active, allowing the 防御方 to annex the war participant 国家 at 25% cost instead of just the 叛军. This allows major powers like 法兰西 (3rd largest 国家 at game start) to be annexed by smaller 国家 like 英格兰
- The mechanism can be systematically exploited by knowledgeable players in multiple ways (see EXPLOITATION SCENARIOS section)
- The mechanism violates historical rationality
- Players lose territories and strategic control due to AI decisions
- Inconsistency: 联合统治 are treated differently from other subject types (附庸, 采邑 are protected, 联合统治 are not)
- **EXPLOIT 1:** 附庸 players can use their 宗主国 as weapons to attack same-culture 国家 and eventually declare independence, breaking intended gameplay balance
- **EXPLOIT 2:** Players can attack opponents (AI or human) during a 停战协议 period by intentionally creating a 停战协议 period through War 2, then triggering 归并国叛乱 which forces the victim to join and receive -50 稳定度 penalties, preventing the victim from calling 同盟. This completely breaks game balance in both single-player and multiplayer contexts
- **WORLD 战争 ESCALATION:** A single local 叛乱 can escalate into a world 战争 due to asymmetric 同盟 calling. The 防御方 can call all 同盟, while forcibly called 进攻方 cannot call its own 同盟, creating a dangerous escalation mechanism
- **SMALL SUBJECTS DRAGGED INTO DISTANT 战争:** Any subject (e.g., 附庸, 归并国 subject) that happens to be culturally dominant is forcibly called to support distant rebellions of the same culture, even when thousands of kilometers away. Players lose control over their subjects' foreign policy and must investigate to find out why their subjects are at 战争 (VERIFIED: 元 case - small Mongolian subject in Mongolia region forced to support rebellion in Georgia 6000+ km away)
- **INCONSISTENT BEHAVIOR OF LAND-BASED VS. LANDLESS 归并国:** Land-based 归并国叛乱 (>50% cultural population) become 附庸s of the culturally dominant 国家 when they win, while landless 归并国叛乱 (<50% cultural population - 群众集会) have inconsistent outcomes: either become independent 国家 or useless landless 附庸s (e.g., 1 unit of cavalry with 5 people). This creates inconsistent outcomes where culturally dominant 国家 gain almost nothing from fighting landless 叛乱战争 (only 314-350 ducats even at 100% 战争分数), and the entire secessionist concept falls apart for landless rebels. Major powers fight major 战争 for essentially nothing (VERIFIED: 法兰西 vs 卡斯蒂利亚 case - 法兰西 fought entire 战争 with 100% 战争分数 and gained only 314.82 to 350.72 ducats, no territory or useful 附庸; landless rebel either became independent or useless landless 附庸 with 5 people)

---

## Recommended Priority

Issues 1, 2, and 5 should be prioritized as they have the most severe impact on gameplay experience and players have absolutely no control over them. Issue 2 (联合统治 Breaking) is particularly severe because it creates an inconsistency where one subject type (联合统治) is vulnerable while some others (附庸, 采邑) are protected. The exploitation scenarios (Exploit 1 and Exploit 2) make this even more critical, as they completely break game balance in both single-player and multiplayer contexts, making games uncompetitive and unplayable for victims. Further testing is needed to determine if other subject types (朝贡国, 自治领, 卫戍国, 封邑, etc.) are also affected.

---

## Steps to Reproduce

Multiple game saves have been prepared to demonstrate these issues. Save file IDs are provided for each set.

### CASE 1: 拜占庭 SAVES (Demonstrates Issues 1, 3, 4, 6, and 7)

Save File ID: **#f40b1288**

Load the 拜占庭 save files. The saves are set up with:
- 拜占庭 has conquered most 奥斯曼 territories
- 拜占庭 released 附庸 Hudavendigar and ceded all 奥斯曼 territories to it
- 拜占庭 has an 同盟 with 特拉比松
- Turkish 归并国叛乱 is ready to spawn (or has already spawned)
- **CRITICAL:** The rebelling territories were originally owned by 奥斯曼. 奥斯曼 still exist and are the **original owner** of these territories.
- 埃雷特纳 is the culturally dominant 国家 for Turkish culture (this is why it gets selected for support)

**Steps to observe Issue 1** (Same-Culture Countries Forced to Join):

1. Load the 拜占庭 save 1337.7.2 - Byzantium HUD Revolt 100%
2. Turkish rebellion progress in Hudavendigar is set to 100% using console command
3. When 归并国叛军s and declare 内战, observe that 埃雷特纳 (the culturally dominant 国家 for Turkish culture) is automatically and forcibly called to 战争 on the 叛军 side (save: 1337.8.1 - Byzantium HUD ERE War)
4. **CRITICAL OBSERVATION:** Observe that 奥斯曼 (the original owner of the rebelling territories) are **COMPLETELY IGNORED** by the system. 奥斯曼 are not selected, even though they have the strongest historical claim to these territories.
5. Observe that 埃雷特纳 cannot refuse this call - it is forced into the 战争 on day 1
6. If 埃雷特纳 has a 停战协议 with 拜占庭, observe that it receives -50 稳定度 penalty for "breaking 停战协议" even though it had no choice
7. **Note:** 埃雷特纳 is selected because it is the culturally dominant 国家 for Turkish culture, but players cannot predict this selection as the mechanism is hardcoded in the engine. **The system completely ignores original ownership** - 奥斯曼 (original owner) are not selected because they are not the culturally dominant Turkish 国家.

**Steps to observe Issue 3** (附庸战争 Leadership):

1. Load the 拜占庭 save 1337.8.1 - Byzantium HUD ERE War
2. When the 内战 starts, observe that 拜占庭 is forcibly dragged into the 战争 on day 1
3. Check the 战争 panel - observe that 附庸 Hudavendigar is the defensive 战争领袖, NOT 拜占庭
4. Try to call ally 特拉比松 to 战争 - observe that it's impossible because 拜占庭 is not the 战争领袖
5. Try to request military access from 阿赫 or 詹达尔 - observe that it's impossible because of low opinion or `block_when_at_war = yes`
6. Observe that 拜占庭 cannot control peace negotiations - only the AI 附庸 Hudavendigar can make peace
7. Observe that 拜占庭 cannot obtain military access to attack 埃雷特纳, creating a stalemate situation

**Steps to observe Issue 4** (Inconsistency in 同盟 Auto-Call):

1. Load the 拜占庭 save 1337.8.1 - Byzantium HUD ERE War
2. When 内战 starts, observe that 特拉比松 (拜占庭's ally) does NOT automatically join the 战争
3. Try to manually call 特拉比松 - observe that it's impossible because 拜占庭 is not the 战争领袖
4. This demonstrates the inconsistency: if 拜占庭 were the 战争领袖, 拜占庭 can ask 特拉比松 to join, but since the 附庸 is the leader, 拜占庭 cannot call its ally

**Steps to observe Issue 6** (AI 附庸 Suboptimal Decisions):

1. Load the 拜占庭 save 1339.2.1 - Byzantium HUD ERE War Stuck
2. Occupy all rebel territories and transfer control to 附庸 Hudavendigar
3. Achieve 45%+ warscore
4. Wait for AI 附庸 Hudavendigar to make peace
5. Observe that AI 附庸 chooses white peace instead of annexing the Secessionists, even though annexation only costs 2 peace offer points
6. Observe that 拜占庭 loses 7 locations which become new Secessionists, becoming a 附庸 of 埃雷特纳

**Steps to observe Issue 7** (Overlord Forced to Join):

1. Load any 拜占庭 save during the 内战
2. Observe that 拜占庭 (宗主国 of Hudavendigar) is automatically dragged into the 战争
3. Observe that this happens through `join_defensive_wars_always` in `secessionists.txt`

### CASE 2: 法兰西 SAVES - 联合统治 BREAKING (Demonstrates Issue 2)

Save File ID: **#51d1fcfd**

Load the 法兰西联合统治 save files. The saves demonstrate:
- 法兰西 has formed a 联合统治 with 西西里 (法兰西 as 邦联主导国)
- 法兰西 has conquered Sicilian-culture territory from 那不勒斯
- Sicilian 归并国叛乱 is ready to spawn

**Steps to observe Issue 2** (联合统治 Breaking):

1. Load save 1337.5.1.2 - 法兰西联合统治西西里, No CB 那不勒斯
   - 法兰西 has formed 联合统治 with 西西里 using console command `create_pu SIC`
   - 法兰西 has declared No CB 战争 against 那不勒斯
2. Load save 1337.6.1 - 法兰西 Conquer 那不勒斯
   - 法兰西 has conquered Calabria Ultra Province (Sicilian culture)
3. Load save 1337.7.1 - 法兰西 Sicilian Rebel 100%
   - Sicilian rebellion progress set to 100% using console command
4. Load save 1337.8.1 - 法兰西联合统治 Broken
   - 归并国叛乱 breaks out
   - Observe that 西西里 (culturally dominant for Sicilian culture) is forcibly called to support the rebellion
   - Observe that the 联合统治 between 法兰西 and 西西里 immediately breaks on day 1
   - Observe that 西西里 is now at 战争 with 法兰西 (its former 邦联主导国)

**VERIFICATION OF PROTECTION FOR OTHER SUBJECT TYPES:**

The following saves demonstrate that 附庸 and 采邑 are correctly excluded from 归并国 calls. However, other subject types have not been tested and may also be affected:

**EU5 Subject Types (Status):**
- 联合统治: **VERIFIED AFFECTED**
- 附庸: **VERIFIED EXCLUDED**
- 采邑: **VERIFIED EXCLUDED**
- 朝贡国, 自治领, 卫戍国, 封邑, 三曼多, 土司, 乌奇贝伊, etc.: NOT TESTED

### CASE 3: 法兰西 SAVES - 采邑 (No Call)

Save File ID: **#1177c64a**

1. Load save 1337.4.1 - 法兰西 ANNEX 西西里, Create 采邑 subject
   - 法兰西 annexed 西西里 using console command
   - Created a new 西西里采邑 subject
2. Load save 1337.6.1 - 法兰西 Conquer 那不勒斯
   - 法兰西 conquered Calabria Ultra Province (Sicilian culture)
3. Load save 1337.7.1 - 法兰西 Sicilian Rebel 100%
   - Sicilian rebellion progress set to 100% using console command
4. Load save 1337.8.1 - 法兰西 Sicilian 归并国 NOT calling 西西里
   - 归并国叛乱 breaks out
   - Observe that 西西里 (采邑) is NOT called to support the 叛乱
   - This demonstrates that 采邑 are correctly excluded

### CASE 4: 法兰西 SAVES - 附庸 (No Call)

Save File ID: **#dae84461**

1. Load save 1337.4.1 - 法兰西 ANNEX 西西里, Create 附庸 subject
   - 法兰西 annexed 西西里 using console command
   - Created a new 西西里附庸 subject
2. Load save 1337.6.10 - 法兰西 Conquer 那不勒斯
   - 法兰西 conquered Calabria Ultra Province (Sicilian culture)
3. Load save 1337.7.1 - 法兰西 Sicilian Rebel 100%
   - Sicilian rebellion progress set to 100% using console command
4. Load save 1337.8.1 - 法兰西归并国 NOT calling 西西里附庸
   - 归并国叛乱 breaks out
   - Observe that 西西里 (附庸) is NOT called to support the 叛乱
   - This demonstrates that 附庸 are correctly excluded

**CONCLUSION:** 联合统治 are VERIFIED to receive 归并国 calls, while 附庸 and 采邑 are VERIFIED to be excluded. This creates an inconsistency where 联合统治 break while some other subject types are protected. Other subject types have not been tested and may also be affected.

### CASE 5: 法兰西 SAVES - 包围网 BYPASS AND 停战协议 BREAKING (Demonstrates Issues 1, 5, and 8)

Save File ID: **#75e13d1**

Load the 法兰西 save files. The saves are set up with:
- 法兰西 has formed 同盟 with 阿拉贡 and 罗马
- 法兰西 has defeated 卡斯蒂利亚 in a No CB 战争 and taken Castilian territories
- Castilian 归并国叛乱 is ready to spawn (or has already spawned)
- 卡斯蒂利亚 may be in a 包围网 against 法兰西 (or will join after 停战协议 ends)

**Steps to observe Issue 1** (Same-Culture Countries Forced to Join):

1. Load the 法兰西 save 1338.4.1 - 法兰西停战协议 with 卡斯蒂利亚, Add Castilian Rebel to 100%
2. Console command is used to force Castilian rebellion progress to 100%
3. When 归并国叛军s and declare 内战, observe that 卡斯蒂利亚 is automatically and forcibly called to 战争 on the 叛军 side
4. Observe that 卡斯蒂利亚 cannot refuse this call - it is forced into the 战争

**Steps to observe Issue 8** (停战协议 Breaking):

1. Load the 法兰西 save 1338.4.1 - 法兰西停战协议 with 卡斯蒂利亚, Add Castilian Rebel to 100%
2. If 卡斯蒂利亚 and 法兰西 have a 停战协议 when the rebellion 战争 begins, observe that 卡斯蒂利亚 receives -50 稳定度 penalty for "breaking 停战协议" in the next month when rebellion breaks out
3. Observe that 卡斯蒂利亚 is judged as actively breaking the 停战协议 even though it was forcibly called to 战争 and had no choice

**Steps to observe Issue 5** (包围网 Bypass):

1. Load the 法兰西 save 1346.8.2 - 法兰西卡斯蒂利亚 Joint 包围网, Add Castilian Rebel to 100%
2. 卡斯蒂利亚 has joined a 包围网 against 法兰西
3. When 归并国叛军s, observe that 卡斯蒂利亚 is forcibly called to 战争 as the 进攻方 (叛军 side)
4. Observe that 卡斯蒂利亚 cannot call 包围网 members (葡萄牙, 纳瓦拉) to join the 战争 because it is the attacker
5. Observe that 法兰西 (防御方) CAN call 同盟 (阿拉贡, 罗马) to join the defensive 战争
6. Result: 法兰西 can bypass the 包围网 to attack 卡斯蒂利亚 alone, while 卡斯蒂利亚 cannot obtain 包围网 or ally support
7. Observe that 卡斯蒂利亚 will leave the 包围网 after the rebellion 战争 ends

### CASE 6: 法兰西 vs 卡斯蒂利亚 LANDLESS REBEL BEHAVIOR (Demonstrates Issue 12 - VERIFIED)

Save File ID: **#39d23961**

Load the 法兰西 vs 卡斯蒂利亚 landless rebel save files. The saves demonstrate:
- 卡斯蒂利亚 has conquered Loudun (French culture territory)
- French 归并国叛乱 spawns as 群众集会 (<50% French culture population)
- 法兰西 (culturally dominant) is forcibly called to support the 叛乱

**Steps to observe Issue 12** (Landless Rebel Behavior):

1. Load save 1337.5.1 - 卡斯蒂利亚 Conquer Loudun, add revolt 100%
   - Play as 卡斯蒂利亚
   - Console: `conquer loudun`
   - Add rebel progress to 1 (100%)
2. Load save 1337.6.1 - 法兰西 Join Secessionist War, Occupy 卡斯蒂利亚, 1 - No Land Transferred to AAA00
   - Play as 法兰西
   - Console: `occupy_country CAS` to gain 100% 战争分数
   - Observe: No land is transferred to newly spawned tag AAA00 (a landless rebel)
3. Load save 1337.6.1 - 法兰西 Join Secessionist War, Occupy 卡斯蒂利亚, 2 - Loudun Transferred to AAA00
   - Play as 法兰西
   - Console: `occupy_country CAS` to gain 100% 战争分数
   - Observe: Loudun is transferred to newly spawned tag AAA00 (a landless rebel)
4. Load save 1337.8.1 - 法兰西 1. Set Cash 0
   - 2 months after save 2
   - Set cash to 0 to check gained 战争赔款
5. Load save 1337.8.9 - 法兰西 1. Gain 350.72 War Reparation, AAA00 Becomes Landless Secessionist
   - 9 days after save 2
   - AAA00 makes peace with 卡斯蒂利亚
   - Observe: 法兰西 gained 350.72 ducats, and AAA00 is a landless 归并国附庸 with 1 unit of cavalry (5 people)
6. Load save 1337.8.1 - 法兰西 2. Set Cash 0
   - 2 months after save 3
   - Set cash to 0 to check gained 战争赔款
7. Load save 1337.8.11 - 法兰西 2. Gain 350.72 War Reparation, AAA00 Becomes Landless Secessionist
   - 11 days after save 2
   - AAA00 made peace with 卡斯蒂利亚
   - Observe: 法兰西 gained 314.82 ducats, and AAA00 is now a settled country but **NOT a 归并国附庸** of 法兰西

**CRITICAL OBSERVATIONS:**
- Two major powers (法兰西 and 卡斯蒂利亚) fight a 战争 for essentially nothing
- Best case scenario with 100% 战争分数: 法兰西 gains only 314.82 to 350.72 ducats (战争赔款)
- The landless rebel (AAA00) either:
  1. Becomes a landless 归并国附庸 with 1 unit of cavalry (5 people) - effectively useless
  2. Becomes a settled country but **NOT a 归并国附庸** of 法兰西 - completely independent
- This demonstrates that the secessionist concept completely falls apart for landless rebels - there is almost no benefit to fighting in a secessionist 战争 for a landless secessionist because the supposedly overlord 法兰西 gains almost nothing, and the landless rebel does NOT become 法兰西's useful 附庸 if it becomes settled

---

## Game Saves Provided

Multiple game saves have been prepared and attached:

**拜占庭 Saves** (Issues 1, 3, 4, 6, 7):
Save File ID: **#f40b1288**
- 1337.7.2 - Byzantium HUD Revolt 100%
- 1337.8.1 - Byzantium HUD ERE War
- 1339.2.1 - Byzantium HUD ERE War Stuck
- Additional saves demonstrating various scenarios

**法兰西联合统治 Saves** (Issue 2 - VERIFIED):
Save File ID: **#51d1fcfd**
- 1337.5.1.2 - 法兰西联合统治西西里, No CB 那不勒斯
- 1337.6.1 - 法兰西 Conquer 那不勒斯
- 1337.7.1 - 法兰西 Sicilian Rebel 100%
- 1337.8.1 - 法兰西联合统治 Broken

**法兰西采邑 Saves** (Verification - No Call):
Save File ID: **#1177c64a**
- 1337.4.1 - 法兰西 ANNEX 西西里, Create 采邑 subject
- 1337.6.1 - 法兰西 Conquer 那不勒斯
- 1337.7.1 - 法兰西 Sicilian Rebel 100%
- 1337.8.1 - 法兰西 Sicilian 归并国 NOT calling 西西里

**法兰西附庸 Saves** (Verification - No Call):
Save File ID: **#dae84461**
- 1337.4.1 - 法兰西 ANNEX 西西里, Create 附庸 subject
- 1337.6.10 - 法兰西 Conquer 那不勒斯
- 1337.7.1 - 法兰西 Sicilian Rebel 100%
- 1337.8.1 - 法兰西 Secessionist NOT calling 西西里附庸

**法兰西包围网/停战协议 Saves** (Issues 1, 5, 8):
Save File ID: **#75e13d1**
- 1338.4.1 - 法兰西停战协议 with 卡斯蒂利亚, Add Castilian Rebel to 100%
- 1346.8.2 - 法兰西卡斯蒂利亚 Joint 包围网, Add Castilian Rebel to 100%
- Additional saves demonstrating various scenarios

**法兰西 vs 卡斯蒂利亚 Landless Rebel Saves** (Issue 12 - VERIFIED):
Save File ID: **#39d23961**
- 1337.5.1 - 卡斯蒂利亚 Conquer Loudun, add revolt 100%
- 1337.6.1 - 法兰西 Join Secessionist War, Occupy 卡斯蒂利亚, 1 - No Land Transferred to AAA00
- 1337.6.1 - 法兰西 Join Secessionist War, Occupy 卡斯蒂利亚, 2 - Loudun Transferred to AAA00
- 1337.8.1 - 法兰西 1. Set Cash 0
- 1337.8.9 - 法兰西 1. Gain 350.72 War Reparation, AAA00 Becomes Landless Secessionist
- 1337.8.1 - 法兰西 2. Set Cash 0
- 1337.8.11 - 法兰西 2. Gain 350.72 War Reparation, AAA00 Becomes Landless Secessionist

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
- `game/in_game/common/wargoals/00_default.txt` (lines 89-99: `take_country_nationalist` - `type = take_country`, `conquer_cost = 0.25`, `subjugate_cost = 0.25` - low warscore costs for 归并国战争, intended for annexing 叛军s but exploitable when war participant 国家 becomes 战争领袖 due to Issue 10. The `type = take_country` allows annexing the entire 国家 via 吞并叛军 button)
- `game/main_menu/localization/english/war_overview_l_english.yml` (line 8: `WAR_LATERALVIEW_ANNEX_REVOLTER: "Annex Revolter"` - peace option that allows annexing revolter countries)
- `game/main_menu/localization/english/diplomacy_l_english.yml` (lines 1149-1150: `PEACE_TREATY_ANNEX_REVOLTER` - peace treaty description for annexing revolters)
- `game/loading_screen/common/defines/00_defines.txt` (line 1673: `MIN_WARSCORE_TO_DEMAND = 10` - minimum war score required to enable peace options like 吞并叛军; lines 1832-1833: `PEACE_TREATY_ANNEX_REVOLTER_MAX_COST = 70`, `PEACE_TREATY_REVOLTER_SURVIVES_MAX_COST = 70`)
