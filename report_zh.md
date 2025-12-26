# Bug Report: 归并国 Mechanism

## Metadata

**Title:** 归并国 Mechanism - Forced Participation, Original Owner Ignored, World War Escalation, and Exploits

**Issue Type:** Gameplay

**Bug Type:** AI (non-player characters), Other

**Game Version:** 1.0.10

---

## 摘要

归并国机制存在严重的设计缺陷，导致游戏破坏性利用。同文化国家在没有玩家控制的情况下被强制召唤参战，**完全忽略原始所有权**，优先考虑文化主导地位。该机制破坏联合统治，创造不一致的战争领导权逻辑（附庸在归并国内战中领导，与正常宣战中宗主国领导不同），在附庸领导战争时阻止玩家控制，错误地应用停战协议惩罚，并允许通过吞并叛军按钮吞并整个参战国（例如，法兰西）而不产生敌意。不对称的同盟召唤创造了危险的世界战争升级，防御方可以协调，而被强制召唤的进攻方无法协调。无地归并国叛乱（群众集会）产生不一致的结果，文化主导国家战斗后几乎一无所获，不像有地叛乱会成为附庸。这些缺陷使系统性利用成为可能：（1）附庸玩家可以使用宗主国作为武器攻击同文化国家（已验证）；（2）玩家可以在停战协议期间通过触发归并国叛乱来攻击对手，迫使受害者加入并受到-50稳定度惩罚。

---

## 描述

归并国机制存在十二个关键问题：

### Issue 1: Same-Culture Countries Forced to Join (Most Severe)

When 归并国叛军s, the system automatically searches for nearby same-culture 国家 and forcibly calls them to 战争 on the 叛军 side. Called 国家 cannot refuse. If they have a 停战协议 with the enemy, they are incorrectly judged as actively breaking the 停战协议, triggering severe penalties (-50 稳定度, +1 厌战度, +25 敌意). The selection mechanism is hardcoded in the engine and cannot be found in game files, making it impossible for players to predict which country will be called.

**关键缺陷：完全忽略原始所有权**

系统在选择要召唤的同文化国家时**完全忽略原始所有权**，优先考虑文化主导地位而非历史主张。

*来自拜占庭存档的示例：* 土耳其归并国叛乱发生在原本由奥斯曼拥有的领土上。奥斯曼仍然存在，是原始所有者。然而，系统选择了埃雷特纳（文化主导的土耳其国家）而不是奥斯曼，完全忽略了奥斯曼。奥斯曼没有机会收回其前领土，尽管它们拥有最强的历史主张。

### 问题 2：联合统治破裂（严重 - 已验证）

When a 联合统治邦联从属国 is the culturally dominant 国家 for a 归并国叛乱, it is forcibly called to support the 叛乱 against its 邦联主导国. This immediately breaks the 联合统治 because `union_of_crowns_pact` has `break_on_war = yes`.

**验证状态：**
- 联合统治：**已验证受影响**（已用法兰西-西西里测试）
- 附庸：**已验证被排除**（已用法兰西-西西里测试）
- 采邑：**已验证被排除**（已用法兰西-西西里测试）
- 其他属国类型（朝贡国、自治领、卫戍国、封邑、三曼多、土司、乌奇贝伊等）：未测试

这造成了不一致性，联合统治容易受到影响，而其他一些属国类型受到保护。

**验证：** 已用法兰西（邦联主导国）和西西里（邦联从属国）测试。当西西里归并国叛军在法兰西控制的领土上爆发时，西西里被强制召唤，立即破坏了联合统治。当西西里是附庸或采邑时，它被正确排除。

### 问题 3：附庸战争领导权问题（严重）

如果玩家创建的附庸在内战中成为防御方，玩家会被强制拖入战争但不会成为战争领袖。AI附庸控制所有和平谈判。玩家无法召唤同盟，无法获得军事通行权，并且必须在没有控制权的情况下承担所有战争成本。

**关键不一致性：正常战争宣战 vs. 归并国内战**

在正常战争宣战和归并国内战之间，关于附庸战争领导权的游戏逻辑存在根本性不一致：

**Normal 战争 Declaration (using 宣战理由 and 外交官):**
- When a 国家 declares 战争 against a 附庸 (e.g., 拜占庭 declares 战争 against 阿尔巴尼亚 where 阿尔巴尼亚 is a 附庸 of 那不勒斯), the 宗主国 (那不勒斯) becomes the 战争领袖, NOT the 附庸 (阿尔巴尼亚)
- The 宗主国 controls all peace negotiations and 战争 decisions
- This is the expected behavior for normal 战争 declarations

**归并国内战 (Issue 3):**
- When a 附庸 becomes the 防御方 in a 归并国内战, the 附庸 becomes the 战争领袖, NOT the 宗主国
- The 宗主国 is forcibly dragged into the 战争 but does not control peace negotiations
- AI附庸控制所有和平谈判，而宗主国没有控制权

**不一致性：**
- **正常战争宣战：** 当附庸被攻击时，宗主国是战争领袖
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

**关键限制：被强制召唤的文化主导国家（进攻方）无法使用人情召唤同盟**

当同文化国家被强制召唤支持归并国叛乱作为进攻方时，它**无法使用人情召唤其同盟**，因为它不是战争领袖（叛乱国家是战争领袖）。

**已验证：无法在战争期间整合归并国附庸**

在归并国附庸是战争领袖的持续战争中，**不可能**整合/吞并归并国附庸。游戏阻止吞并处于内战中的国家（参见 `ANNEX_TARGET_CIVIL_WAR_PENALTY` 本地化："目前正在经历内战，在恢复和平之前无法被吞并"）。

**文化主导国家可以召唤同盟的唯一场景：**

**当无地归并国叛军被防御方消灭时**：如果归并国叛乱作为群众集会（无地且无领土）生成，其部队被防御方消灭，叛乱将从战争中移除。参战的文化主导国家随后自动成为战争领袖，可以使用人情召唤同盟（成本：每个同盟10人情）。

**技术验证：**
- 归并国附庸在作为战争领袖的持续战争中无法被整合/吞并（被内战限制阻止）
- `ask_join_war_for_favors` 交互要求调用者是战争领袖（`attacker_leader = scope:actor` 或 `defender_leader = scope:actor`）
- 成本：每个同盟10人情（来自 `ask_join_war_for_favors_cost`）
- 战争领导权转移：当无地叛军（战争领袖）被消灭时，剩余的参战国自动成为新的战争领袖

**影响：**
- 被强制召唤的文化主导国家在整个战争中处于严重劣势，除非无地叛军被防御方消灭，否则无法召唤同盟
- 这创造了不对称的情况，防御方可以从第一天起与同盟协调，而文化主导的进攻方无法协调（参见问题 9）
- 文化主导进攻方获得召唤同盟能力的唯一方式取决于防御方的行动（消灭无地叛军），使防御方控制攻击者何时可以召唤同盟

### Issue 5: 包围网 Bypass Mechanism (Severe Exploit)

When a forcibly called 国家 is in a 包围网, it cannot call 包围网 members to join the 战争 because it is the 进攻方. This allows the 防御方 to bypass 包围网 restrictions and attack 包围网 members individually.

### Issue 6: AI 附庸 Suboptimal Peace Decisions

AI 附庸 may choose 无条件和平 instead of annexing the 叛乱者 even when annexation cost is very low, causing players to lose territories they should have gained.

### Issue 7: Overlord Forced to Join

归并国 automatically and forcibly call 宗主国 to 战争 through `join_offensive_wars_always` and `join_defensive_wars_always` in `secessionists.txt`.

### 问题 8：停战协议违反惩罚

被强制召唤的有停战协议的国家被错误地判定为主动违反停战协议，触发严重惩罚，尽管它们没有选择。

### Issue 9: World War Escalation - Asymmetric 同盟 Calling (Severe)

单个本地归并国叛乱可能因不对称同盟召唤机制升级为世界大战。这**比历史世界大战**（第一次世界大战/第二次世界大战）更糟糕，因为历史战争中双方都可以与同盟协调。

**当前 EU5 系统 - 比历史世界大战更糟糕：**

当归并国叛乱爆发时：
- 防御方可以从第一天起召唤所有同盟加入防御战争
- 被强制召唤的同文化国家（进攻方）无法使用人情召唤自己的同盟，因为它不是战争领袖（叛乱国家是战争领袖）
- **文化主导进攻方可以召唤同盟的唯一例外场景**（详见问题 4）：
  - 无地归并国叛军被防御方消灭后（自动成为战争领袖）
- **已验证：无法在战争期间整合归并国附庸** - 处于内战中的国家在恢复和平之前无法被吞并
- 结果：本地叛乱升级为重大战争，一个同盟集团在整个战争中独自对抗另一个集团的协调响应（除非防御方消灭无地叛军，这很罕见，并使防御方控制攻击者何时可以召唤同盟）

*示例场景（18 世纪晚期 - 瓜分时代）：*
- 两个主要同盟集团：集团 A（俄罗斯、奥地利、普鲁士）vs 集团 B（法兰西、奥斯曼、瑞典、波兰）
- 波兰归并国叛乱在俄罗斯控制的领土上爆发
- 波兰（文化主导）被迫加入叛乱一方（进攻方）- 没有选择
- 俄罗斯（防御方）召唤奥地利和普鲁士 → 3 个主要强国 vs 波兰
- 波兰（进攻方）无法召唤法兰西、奥斯曼或瑞典（其在集团 B 中的同盟） → 独自对抗 3 个主要强国

这创造了一个危险的升级机制，防御方可以系统地使用归并国叛乱不对称地攻击敌人的同盟集团。

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
11. **英格兰 annexes entire 法兰西** at effectively 0 cost (base 25% cost from `take_country_nationalist` war goal, but reduced by -95% from `PEACE_COST_MODIFIER_FOR_REVOLT_WAR` modifier, making it effectively 0 cost). This is intended only for annexing the 叛乱者, not major powers like 法兰西
12. **No 敌意 is generated** - annexing a major power like 法兰西 should generate massive 敌意, but the peace option generates none

**Technical Details:**
- When 叛乱省份 has <50% population of rebelling culture, 叛乱 spawns as `country_type = pop` (群众集会 - landless with no territory)
- When `country_type = pop` troops are eliminated, the 叛乱 is removed from the 战争
- The war participant 国家 becomes 战争领袖 automatically
- **BUG:** The `take_country_nationalist` war goal has `type = take_country`, which allows annexing the entire 国家 via 吞并叛军 button
- **BUG:** The warscore cost is effectively 0 (base 25% from `conquer_cost = 0.25` in `take_country_nationalist`, but reduced by -95% from `PEACE_COST_MODIFIER_FOR_REVOLT_WAR = -0.95` modifier, making it effectively 0 cost)
- **BUG:** No 敌意 is generated when annexing the war participant 国家
- **Note:** The 防御方 can theoretically conquer land at 25% cost if it is the 战争领袖 (from `take_country_nationalist` war goal), but AI 附庸 will never conquer land for its 宗主国 when the 附庸 is the 战争领袖 in a 归并国叛乱 (see Issue 6)

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
- The AI may only accept 战争赔款 or 无条件和平, leaving the 防御方 with all territory intact
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
4. Player A makes 无条件和平 or gets a small amount of ransom, ending War 2 quickly, creating a new 停战协议 period
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

## 严重性

这些问题很关键，因为：
- 玩家对被拖入战争完全没有控制
- **完全忽略原始所有者** - 系统忽略原始所有权，只考虑文化主导地位（例如，在拜占庭案例中，奥斯曼被忽略，而选择埃雷特纳）
- **不一致的战争领导权逻辑** - 在正常战争宣战中，当附庸被攻击时，宗主国成为战争领袖，但在归并国内战中，附庸反而成为战争领袖。这创造了不可预测和不一致的游戏逻辑，破坏了玩家期望
- 联合统治因强制参与而意外破裂
- 玩家因未选择的行为受到严重惩罚（-50稳定度）
- **作为群众集会生成的叛乱导致战争分数成本应用于错误目标** - 当群众集会部队被消灭且叛乱被消除时，战争继续，参战国成为战争领袖。吞并叛军按钮（仅用于吞并叛军）随后可用于以有效0成本吞并整个参战国（`take_country_nationalist` 战争目标的基础25%成本，但被 `PEACE_COST_MODIFIER_FOR_REVOLT_WAR` 修正值减少-95%，使其有效成本为0）。这允许像法兰西（游戏开始时第三大国家）这样的主要强国被像英格兰这样较小的国家吞并。**注意：** 如果防御方是战争领袖，理论上可以以25%成本征服土地，但当附庸在归并国叛乱中是战争领袖时，AI附庸永远不会为其宗主国征服土地（参见问题 6）
- 该机制可以被有经验的玩家以多种方式系统地利用（参见利用场景部分）
- 该机制违反历史合理性
- 玩家因AI决策而失去领土和战略控制
- 不一致性：联合统治与其他属国类型受到不同对待（附庸、采邑受到保护，联合统治没有）
- **利用 1：** 附庸玩家可以使用其宗主国作为武器攻击同文化国家，并最终宣布独立，破坏预期的游戏平衡
- **利用 2：** 玩家可以在停战协议期间通过故意通过战争2创建停战协议期间，然后触发归并国叛乱来攻击对手（AI或人类），这迫使受害者加入并受到-50稳定度惩罚，阻止受害者召唤同盟。这完全破坏了单人和多人游戏中的游戏平衡
- **世界大战升级：** 单个本地叛乱可能因不对称同盟召唤升级为世界大战。防御方可以召唤所有同盟，而被强制召唤的进攻方无法召唤自己的同盟，创造了危险的升级机制
- **小型属国被拖入遥远战争：** 任何恰好是文化主导的属国（例如，附庸、归并国属国）都会被强制召唤支持同文化的遥远叛乱，即使相距数千公里。玩家失去对其属国外交政策的控制，必须调查以找出其属国为何处于战争状态（已验证：元案例 - 蒙古地区的小型蒙古属国被迫支持格鲁吉亚6000+公里外的叛乱）
- **有地归并国与无地归并国的不一致行为：** 有地归并国叛乱（>50%文化人口）在获胜时成为文化主导国家的附庸，而无地归并国叛乱（<50%文化人口 - 群众集会）有不一致的结果：要么成为独立国家，要么成为无用的无地附庸（例如，1个骑兵单位，5人）。这创造了不一致的结果，文化主导国家从战斗无地叛乱战争中几乎一无所获（即使在100%战争分数时也只有314-350杜卡特），整个分离主义概念对无地叛军完全崩溃。主要强国为基本上一无所获而进行重大战争（已验证：法兰西 vs 卡斯蒂利亚案例 - 法兰西以100%战争分数战斗了整个战争，只获得314.82到350.72杜卡特，没有领土或有用附庸；无地叛军要么成为独立国家，要么成为5人的无用无地附庸）

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
5. Observe that AI 附庸 chooses 无条件和平 instead of annexing the 归并国, even though annexation only costs 2 和平提议 points
6. Observe that 拜占庭 loses 7 地点 which become new 归并国, becoming a 附庸 of 埃雷特纳

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

**对其他属国类型保护的验证：**

以下存档证明附庸和采邑被正确排除在归并国召唤之外。然而，其他属国类型尚未测试，也可能受到影响：

**EU5 属国类型（状态）：**
- 联合统治：**已验证受影响**
- 附庸：**已验证被排除**
- 采邑：**已验证被排除**
- 朝贡国、自治领、卫戍国、封邑、三曼多、土司、乌奇贝伊等：未测试

### 案例 3：法兰西存档 - 采邑（无召唤）

存档文件 ID：**#1177c64a**

1. 加载存档 1337.4.1 - 法兰西吞并西西里，创建采邑属国
   - 法兰西使用控制台命令吞并西西里
   - 创建新的西西里采邑属国
2. 加载存档 1337.6.1 - 法兰西征服那不勒斯
   - 法兰西征服 Calabria Ultra 省份（西西里文化）
3. 加载存档 1337.7.1 - 法兰西西西里叛军 100%
   - 使用控制台命令将西西里叛乱进度设置为 100%
4. 加载存档 1337.8.1 - 法兰西西西里归并国不召唤西西里
   - 归并国叛乱爆发
   - 观察西西里（采邑）未被召唤支持叛乱
   - 这证明采邑被正确排除

### 案例 4：法兰西存档 - 附庸（无召唤）

存档文件 ID：**#dae84461**

1. 加载存档 1337.4.1 - 法兰西吞并西西里，创建附庸属国
   - 法兰西使用控制台命令吞并西西里
   - 创建新的西西里附庸属国
2. 加载存档 1337.6.10 - 法兰西征服那不勒斯
   - 法兰西征服 Calabria Ultra 省份（西西里文化）
3. 加载存档 1337.7.1 - 法兰西西西里叛军 100%
   - 使用控制台命令将西西里叛乱进度设置为 100%
4. 加载存档 1337.8.1 - 法兰西归并国不召唤西西里附庸
   - 归并国叛乱爆发
   - 观察西西里（附庸）未被召唤支持叛乱
   - 这证明附庸被正确排除

**结论：** 联合统治已验证会收到归并国召唤，而附庸和采邑已验证被排除。这创造了不一致性，联合统治破裂，而其他一些属国类型受到保护。其他属国类型尚未测试，也可能受到影响。

### 案例 5：法兰西存档 - 包围网绕过和停战协议违反（演示问题 1、5 和 8）

存档文件 ID：**#75e13d1**

加载法兰西存档文件。存档设置如下：
- 法兰西已与阿拉贡和罗马形成同盟
- 法兰西在无宣战理由战争中击败卡斯蒂利亚并占领卡斯蒂利亚领土
- 卡斯蒂利亚归并国叛乱准备生成（或已经生成）
- 卡斯蒂利亚可能处于针对法兰西的包围网中（或在停战协议结束后加入）

**观察问题 1 的步骤**（同文化国家被迫加入）：

1. 加载法兰西存档 1338.4.1 - 法兰西与卡斯蒂利亚停战协议，添加卡斯蒂利亚叛军至 100%
2. 使用控制台命令强制将卡斯蒂利亚叛乱进度设置为 100%
3. 当归并国叛军并宣布内战时，观察卡斯蒂利亚自动且强制被召唤加入叛军一方的战争
4. 观察卡斯蒂利亚无法拒绝此召唤 - 它被迫进入战争

**观察问题 8 的步骤**（停战协议违反）：

1. 加载法兰西存档 1338.4.1 - 法兰西与卡斯蒂利亚停战协议，添加卡斯蒂利亚叛军至 100%
2. 如果卡斯蒂利亚和法兰西在叛乱战争开始时处于停战协议状态，观察卡斯蒂利亚在叛乱爆发的下一个月因"违反停战协议"受到 -50 稳定度惩罚
3. 观察卡斯蒂利亚被判定为主动违反停战协议，尽管它被强制召唤参战且没有选择

**观察问题 5 的步骤**（包围网绕过）：

1. 加载法兰西存档 1346.8.2 - 法兰西卡斯蒂利亚联合包围网，添加卡斯蒂利亚叛军至 100%
2. 卡斯蒂利亚已加入针对法兰西的包围网
3. 当归并国叛军时，观察卡斯蒂利亚被强制召唤作为进攻方（叛军一方）参战
4. 观察卡斯蒂利亚无法召唤包围网成员（葡萄牙、纳瓦拉）加入战争，因为它是攻击者
5. 观察法兰西（防御方）可以召唤同盟（阿拉贡、罗马）加入防御战争
6. 结果：法兰西可以绕过包围网单独攻击卡斯蒂利亚，而卡斯蒂利亚无法获得包围网或盟友支持
7. 观察卡斯蒂利亚将在叛乱战争结束后离开包围网

### 案例 6：法兰西 vs 卡斯蒂利亚无地叛军行为（演示问题 12 - 已验证）

存档文件 ID：**#39d23961**

加载法兰西 vs 卡斯蒂利亚无地叛军存档文件。存档演示：
- 卡斯蒂利亚已征服 Loudun（法国文化领土）
- 法国归并国叛乱作为群众集会生成（<50% 法国文化人口）
- 法兰西（文化主导）被强制召唤支持叛乱

**观察问题 12 的步骤**（无地叛军行为）：

1. 加载存档 1337.5.1 - 卡斯蒂利亚征服 Loudun，添加叛乱 100%
   - 作为卡斯蒂利亚游戏
   - 控制台：`conquer loudun`
   - 将叛乱进度添加到 1（100%）
2. 加载存档 1337.6.1 - 法兰西加入归并国战争，占领卡斯蒂利亚，1 - 无土地转移给 AAA00
   - 作为法兰西游戏
   - 控制台：`occupy_country CAS` 以获得 100% 战争分数
   - 观察：没有土地转移给新生成的标签 AAA00（无地叛军）
3. 加载存档 1337.6.1 - 法兰西加入归并国战争，占领卡斯蒂利亚，2 - Loudun 转移给 AAA00
   - 作为法兰西游戏
   - 控制台：`occupy_country CAS` 以获得 100% 战争分数
   - 观察：Loudun 转移给新生成的标签 AAA00（无地叛军）
4. 加载存档 1337.8.1 - 法兰西 1. 设置现金 0
   - 存档 2 后 2 个月
   - 将现金设置为 0 以检查获得的战争赔款
5. 加载存档 1337.8.9 - 法兰西 1. 获得 350.72 战争赔款，AAA00 成为无地归并国
   - 存档 2 后 9 天
   - AAA00 与卡斯蒂利亚议和
   - 观察：法兰西获得 350.72 杜卡特，AAA00 是一个无地归并国附庸，有 1 个骑兵单位（5 人）
6. 加载存档 1337.8.1 - 法兰西 2. 设置现金 0
   - 存档 3 后 2 个月
   - 将现金设置为 0 以检查获得的战争赔款
7. 加载存档 1337.8.11 - 法兰西 2. 获得 314.82 战争赔款，AAA00 成为有地独立国家
   - 存档 2 后 11 天
   - AAA00 与卡斯蒂利亚议和
   - 观察：法兰西获得 314.82 杜卡特，AAA00 现在是一个定居国家，但**不是**法兰西的归并国附庸

**关键观察：**
- 两个主要强国（法兰西和卡斯蒂利亚）为基本上一无所获而战斗
- 100% 战争分数的最佳情况：法兰西只获得 314.82 到 350.72 杜卡特（战争赔款）
- 无地叛军（AAA00）要么：
  1. 成为有 1 个骑兵单位（5 人）的无地归并国附庸 - 实际上无用
  2. 成为定居国家，但**不是**法兰西的归并国附庸 - 完全独立
- 这证明分离主义概念对无地叛军完全崩溃 - 对于无地分离主义者来说，在分离主义战争中战斗几乎没有好处，因为所谓的宗主国法兰西几乎一无所获，如果无地叛军成为定居国家，它不会成为法兰西的有用附庸

### 案例 7：卡斯蒂利亚存档 - 攻击方召唤盟友（演示问题 4 - 已验证）

存档文件 ID：**#db345ae3**

加载卡斯蒂利亚存档文件。存档演示：
- 卡斯蒂利亚与葡萄牙有同盟
- 卡斯蒂利亚已征服 Loudun（法国文化领土）
- 法国归并国叛乱作为群众集会生成（<50% 法国文化人口）
- 法兰西（文化主导）被强制召唤支持叛乱
- 法兰西与阿拉贡和罗马有同盟
- 法兰西与阿拉贡和罗马有 100 人情

**观察问题 4 的步骤**（使用人情召唤攻击方盟友）：

1. 加载存档 1337.5.1 - 卡斯蒂利亚与葡萄牙结盟，征服 Loudun，添加叛乱 100%
   - 作为卡斯蒂利亚游戏
   - 卡斯蒂利亚与葡萄牙有同盟
   - 控制台：`conquer loudun`
   - 将叛乱进度添加到 1（100%）
2. 加载存档 1337.5.1 - 法兰西与罗马、阿拉贡结盟，人情 100
   - 切换到法兰西游戏
   - 法兰西与阿拉贡、罗马结盟
   - 使用控制台命令 `favor ARA 100` 和 `favor PAP 100` 添加人情
3. 加载存档 1337.6.1 - 法兰西归并国战争
   - 归并国叛军在 1337.6.1 爆发
   - Loudun 中的群众集会叛军是战争领袖，因此法兰西无法召唤阿拉贡和罗马参战
4. 加载存档 1337.6.3 - 法兰西消灭无地叛军，可以召唤阿拉贡和罗马
   - 使用控制台命令 `kill_unit` 消灭群众集会归并国叛军部队
   - 法兰西成为战争领袖
   - 法兰西可以使用人情召唤阿拉贡和罗马加入战争

---

## Game Saves Provided

Multiple game saves have been prepared and attached:

**拜占庭存档**（问题 1、3、4、6、7）：
存档文件 ID：**#f40b1288**
- 1337.7.2 - Byzantium HUD Revolt 100%
- 1337.8.1 - Byzantium HUD ERE War
- 1339.2.1 - Byzantium HUD ERE War Stuck
- 演示各种场景的其他存档

**法兰西联合统治存档**（问题 2 - 已验证）：
存档文件 ID：**#51d1fcfd**
- 1337.5.1.2 - 法兰西联合统治西西里，无宣战理由对那不勒斯
- 1337.6.1 - 法兰西征服那不勒斯
- 1337.7.1 - 法兰西西西里叛军 100%
- 1337.8.1 - 法兰西联合统治破裂

**法兰西采邑存档**（验证 - 无召唤）：
存档文件 ID：**#1177c64a**
- 1337.4.1 - 法兰西吞并西西里，创建采邑属国
- 1337.6.1 - 法兰西征服那不勒斯
- 1337.7.1 - 法兰西西西里叛军 100%
- 1337.8.1 - 法兰西西西里归并国不召唤西西里

**法兰西附庸存档**（验证 - 无召唤）：
存档文件 ID：**#dae84461**
- 1337.4.1 - 法兰西吞并西西里，创建附庸属国
- 1337.6.10 - 法兰西征服那不勒斯
- 1337.7.1 - 法兰西西西里叛军 100%
- 1337.8.1 - 法兰西归并国不召唤西西里附庸

**法兰西包围网/停战协议存档**（问题 1、5 和 8）：
存档文件 ID：**#75e13d1**
- 1338.4.1 - 法兰西与卡斯蒂利亚停战协议，添加卡斯蒂利亚叛军至 100%
- 1346.8.2 - 法兰西卡斯蒂利亚联合包围网，添加卡斯蒂利亚叛军至 100%
- 演示各种场景的其他存档

**法兰西 vs 卡斯蒂利亚无地叛军存档**（问题 12 - 已验证）：
存档文件 ID：**#39d23961**
- 1337.5.1 - 卡斯蒂利亚征服 Loudun，添加叛乱 100%
- 1337.6.1 - 法兰西加入归并国战争，占领卡斯蒂利亚，1 - 无土地转移给 AAA00
- 1337.6.1 - 法兰西加入归并国战争，占领卡斯蒂利亚，2 - Loudun 转移给 AAA00
- 1337.8.1 - 法兰西 1. 设置现金 0
- 1337.8.9 - 法兰西 1. 获得 350.72 战争赔款，AAA00 成为无地归并国
- 1337.8.1 - 法兰西 2. 设置现金 0
- 1337.8.11 - 法兰西 2. 获得 314.82 战争赔款，AAA00 成为有地独立国家

**卡斯蒂利亚存档 - 攻击方召唤盟友**（问题 4 - 已验证）：
存档文件 ID：**#db345ae3**
- 1337.5.1 - 卡斯蒂利亚与葡萄牙结盟，征服 Loudun，添加叛乱 100%
- 1337.5.1 - 法兰西与罗马、阿拉贡结盟，人情 100
- 1337.6.1 - 法兰西归并国战争
- 1337.6.3 - 法兰西消灭无地叛军，可以召唤阿拉贡和罗马

---

## 相关文件

- `game/in_game/common/subject_types/secessionists.txt`（第 12-17 行：`join_offensive_wars_always`、`join_defensive_wars_always`）
- `game/in_game/common/subject_types/vassal.txt`
- `game/in_game/common/scripted_relations/alliance.txt`（第 13-14 行：`called_in_defensively`、`called_in_offensively`）
- `game/in_game/common/scripted_relations/union_of_crowns_pact.txt`（第 10 行：`break_on_war = yes`）
- `game/in_game/common/on_action/_hardcoded.txt`（第 717-777 行：`on_civil_war_start` - 用于召唤同文化国家的硬编码机制）
- `game/in_game/common/scripted_effects/global_effects.txt`（第 218 行：`start_civil_war`）
- `game/in_game/common/prices/00_hardcoded.txt`（第 100-103 行：`war_breaking_truce` - stability = 50, war_exhaustion = 1）
- `game/in_game/common/biases/05_antagonism_hardcoded.txt`（第 35-38 行：`antagonism_breaking_truce` - value = 25）
- `game/in_game/common/wargoals/00_default.txt`（第 89-99 行：`take_country_nationalist` - `type = take_country`、`conquer_cost = 0.25`、`subjugate_cost = 0.25` - 归并国战争的低战争分数成本，用于吞并叛军，但当参战国因问题 10 成为战争领袖时可被利用。`type = take_country` 允许通过吞并叛军按钮吞并整个国家）
- `game/main_menu/localization/english/war_overview_l_english.yml`（第 8 行：`WAR_LATERALVIEW_ANNEX_REVOLTER: "Annex Revolter"` - 允许吞并叛军国家的和平选项）
- `game/main_menu/localization/english/diplomacy_l_english.yml`（第 1149-1150 行：`PEACE_TREATY_ANNEX_REVOLTER` - 吞并叛军的和平条约描述）
- `game/loading_screen/common/defines/00_defines.txt`（第 1673 行：`MIN_WARSCORE_TO_DEMAND = 10` - 启用吞并叛军等和平选项所需的最低战争分数；第 1832-1833 行：`PEACE_TREATY_ANNEX_REVOLTER_MAX_COST = 70`、`PEACE_TREATY_REVOLTER_SURVIVES_MAX_COST = 70`）
