## 分离主义机制 - 技术实现详情

本文档包含分离主义干预权购买系统的技术实现详情，包括基于已验证 EU5 先例的代码结构、触发器和机制。

### 参考先例

- 德里陷落：`game/in_game/common/situations/fall_of_delhi.txt`（基于优先级的可见性）
- 意大利战争：`game/in_game/common/situations/italian_wars.txt`（使用自定义评分的 `ordered_country`）
- 奥斯曼崛起：`game/in_game/common/generic_actions/rise_of_the_ottomans.txt`（`press_claims` 行动）
- 干预属国内战：`game/in_game/common/country_interactions/intervene_in_subject_civil_war.txt`（使用 `reason = intervene` 的 `join_war_as_attacker`）
- 红巾军叛乱：`game/in_game/common/generic_actions/red_turban_rebellions.txt`（`rtr_join_chinese_defense` 使用 `join_war_as_attacker`）
- create_instant_rebellion：`game/in_game/common/scripted_effects/global_effects.txt`（展示叛军部队的 `raise_levies` 机制）
- start_civil_war：创建叛军国家（具有 `country_type = location` 的普通国家），受内置游戏限制约束
- **注意：** 叛军在爆发前是叛军实体（不是国家）。当调用 `start_civil_war` 时，它们自动成为普通国家 - 无需将它们视为无地国家
- 红巾军叛乱 `rtr_join_chinese_defense`：使用 `can_join_offensive_war_with` 和 `can_join_defensive_war_with` 进行资格检查，无强制参与，可自由加入（无杜卡特成本）
- 保障机制：`game/in_game/common/scripted_relations/guarantee.txt`（`called_in_defensively = giving`）- 允许在被保障国家被攻击时防御性调用保障国
- 请求加入战争换取恩惠：`game/in_game/common/country_interactions/ask_join_war_for_favors.txt` - 允许国家请求其他国家加入战争的现有国家交互
- 包围网 机制：`game/in_game/common/international_organizations/coalition.txt` - 包围网 具有 `has_leader_country = yes` 和 `leader_type = country`。包围网 领袖由 包围网 成员中 `great_power_score` 最高的国家自动选择（未定义显式 `leader_change_method`，因此游戏默认选择最强成员）。当 包围网 目标参与时，包围网 成员可以通过 `join_defensive_wars_always` 和 `join_offensive_wars_always` 加入战争。这允许 包围网 领袖协调 包围网 干预，同时防止级联升级（只有 包围网 领袖可以触发 包围网 参与）。

### 临时解决方案：使用现有加入战争机制

**临时方法：** 此解决方案可以在完整干预权购买系统之前快速实施，使用现有的 `can_join_offensive_war_with` 和 `can_join_defensive_war_with` 触发器（类似于红巾军叛乱）。这提供了即时修复，同时开发综合系统。

文件：`game/in_game/events/secessionist_rebellion_interim.txt`（或添加到现有事件文件）


```

# 当分离主义内战开始时触发的事件
namespace = secessionist_rebellion_interim

secessionist_rebellion_interim.1 = {
    type = country_event
    title = secessionist_rebellion_interim.1.t
    desc = secessionist_rebellion_interim.1.d
    
    trigger = {
        # 检查此国家是否有资格加入
        OR = {
            # 具有相同文化的原所有者（层级 1 - 最高优先级）
            AND = {
                any_current_war = {
                    is_civil_war_for = scope:defender
                    any_attacker = {
                        is_rebel_country = yes
                        rebel_type = secessionist
                        any_rebelling_location = {
                            previous_owner = this
                        }
                    }
                }
                culture = scope:rebel_culture
                can_join_offensive_war_with = scope:rebel_country
            }
            # 具有相同文化的邻国（层级 2）
            AND = {
                any_current_war = {
                    is_civil_war_for = scope:defender
                    any_attacker = {
                        is_rebel_country = yes
                        rebel_type = secessionist
                    }
                }
                culture = scope:rebel_culture
                is_neighbor_of = scope:defender
                can_join_offensive_war_with = scope:rebel_country
            }
            # 文化主导（层级 3 - 仅当不是原所有者或邻国时）
            AND = {
                any_current_war = {
                    is_civil_war_for = scope:defender
                    any_attacker = {
                        is_rebel_country = yes
                        rebel_type = secessionist
                    }
                }
                culture = scope:rebel_culture
                NOT = {
                    any_current_war = {
                        any_attacker = {
                            any_rebelling_location = {
                                previous_owner = this
                            }
                        }
                    }
                }
                NOT = { is_neighbor_of = scope:defender }
                can_join_offensive_war_with = scope:rebel_country
            }
            # 防御方的宿敌（层级 4）
            AND = {
                any_current_war = {
                    is_civil_war_for = scope:defender
                    any_attacker = {
                        is_rebel_country = yes
                        rebel_type = secessionist
                    }
                }
                is_rival_of = scope:defender
                NOT = { culture = scope:rebel_culture }
                can_join_offensive_war_with = scope:rebel_country
            }
        }
        # 标准资格检查（如红巾军叛乱）
        NOT = { is_at_war_with = scope:defender }
        NOT = { has_truce_with = scope:defender }
        NOT = { in_civil_war = yes }
        NOT = { is_during_bankruptcy = yes }
        distance_to_capital <= 5000
    }
    
    immediate = {
        # 存储战争和叛军国家以供参考
        save_scope_as = secessionist_civil_war
        scope:secessionist_civil_war = {
            any_attacker = {
                is_rebel_country = yes
                rebel_type = secessionist
                save_scope_as = rebel_country
            }
        }
    }
    
    option = {
        name = secessionist_rebellion_interim.1.a  # "加入战争"
        
        # 在进攻方一侧加入内战（如干预属国内战）
        scope:secessionist_civil_war = {
            scope:actor = {
                join_war_as_attacker = {
                    war = prev
                    reason = intervene
                }
            }
        }
    }
    
    option = {
        name = secessionist_rebellion_interim.1.b  # "拒绝"
        
        # 不执行任何操作 - 国家拒绝加入，无惩罚
    }
}

# 宗主的特殊事件
secessionist_rebellion_interim.overlord = {
    type = country_event
    title = secessionist_rebellion_interim.overlord.t
    desc = secessionist_rebellion_interim.overlord.d
    
    trigger = {
        # 防御方的宗主
        any_current_war = {
            is_civil_war_for = scope:defender
            any_attacker = {
                is_rebel_country = yes
                rebel_type = secessionist
            }
        }
        scope:defender = {
            is_subject_of = this
        }
    }
    
    immediate = {
        # 计算怀疑分数（与完整系统相同）
        scope:defender = {
            set_variable = {
                name = vassal_suspicion_score
                value = 0
            }
            # ...（与完整系统相同的怀疑分数计算）...
        }
        save_scope_as = secessionist_civil_war
    }
    
    option = {
        name = secessionist_rebellion_interim.overlord.a  # "加入防御战争"
        
        # 加入防御战争
        scope:secessionist_civil_war = {
            scope:actor = {
                join_war_as_defender = {
                    war = prev
                    reason = intervene
                }
            }
        }
    }
    
    option = {
        name = secessionist_rebellion_interim.overlord.b  # "拒绝 - 让$附庸$独自战斗"
        
        # 不执行任何操作 - 宗主拒绝，附庸独自战斗
    }
    
    option = {
        name = secessionist_rebellion_interim.overlord.c  # "取消$附庸$地位"
        
        trigger = {
            scope:defender = {
                var:vassal_suspicion_score >= 50
            }
            # 检查宗主是否可以取消（重用现有 vassal.txt 逻辑）
            scope:defender = {
                subject_type_is_not_locked = no
            }
        }
        
        # 立即取消附庸地位（重用现有 cancel_subject 机制）
        scope:actor = {
            cancel_subject = {
                target = scope:defender
                type = vassal
            }
        }
    }
}

```


文件：`game/in_game/common/on_actions/secessionist_rebellion_interim.txt`（或添加到现有 on_action 文件）


```

# 当分离主义内战开始时触发
on_civil_war_start = {
    # 检查这是否是分离主义叛乱
    if = {
        limit = {
            any_attacker = {
                is_rebel_country = yes
                rebel_type = secessionist
            }
        }
        # 按优先级顺序通知符合条件的国家
        
        # 层级 1：具有相同文化的原所有者（最高优先级）
        every_country = {
            limit = {
                culture = scope:rebel_culture
                any_current_war = {
                    any_attacker = {
                        any_rebelling_location = {
                            previous_owner = this
                        }
                    }
                }
                can_join_offensive_war_with = scope:rebel_country
                NOT = { is_at_war_with = scope:defender }
                NOT = { has_truce_with = scope:defender }
                distance_to_capital <= 5000
            }
            trigger_event_non_silently = secessionist_rebellion_interim.1
        }
        
        # 层级 2：具有相同文化的邻国（如果不是原所有者）
        every_country = {
            limit = {
                culture = scope:rebel_culture
                is_neighbor_of = scope:defender
                NOT = {
                    any_current_war = {
                        any_attacker = {
                            any_rebelling_location = {
                                previous_owner = this
                            }
                        }
                    }
                }
                can_join_offensive_war_with = scope:rebel_country
                NOT = { is_at_war_with = scope:defender }
                NOT = { has_truce_with = scope:defender }
                distance_to_capital <= 5000
            }
            trigger_event_non_silently = {
                id = secessionist_rebellion_interim.1
                days = 2
            }
        }
        
        # 层级 3：文化主导（如果不是原所有者或邻国）
        every_country = {
            limit = {
                culture = scope:rebel_culture
                NOT = {
                    any_current_war = {
                        any_attacker = {
                            any_rebelling_location = {
                                previous_owner = this
                            }
                        }
                    }
                }
                NOT = { is_neighbor_of = scope:defender }
                can_join_offensive_war_with = scope:rebel_country
                NOT = { is_at_war_with = scope:defender }
                NOT = { has_truce_with = scope:defender }
                distance_to_capital <= 5000
            }
            trigger_event_non_silently = {
                id = secessionist_rebellion_interim.1
                days = 5
            }
        }
        
        # 层级 4：防御方的宿敌
        every_country = {
            limit = {
                is_rival_of = scope:defender
                NOT = { culture = scope:rebel_culture }
                can_join_offensive_war_with = scope:rebel_country
                NOT = { is_at_war_with = scope:defender }
                NOT = { has_truce_with = scope:defender }
                distance_to_capital <= 5000
            }
            trigger_event_non_silently = {
                id = secessionist_rebellion_interim.1
                days = 10
            }
        }
        
        # 宗主（特殊通知）
        every_country = {
            limit = {
                scope:defender = {
                    is_subject_of = this
                }
            }
            trigger_event_non_silently = secessionist_rebellion_interim.overlord
        }
    }
}

```


**关键实现要点：**
- 使用现有的 `can_join_offensive_war_with` 和 `can_join_defensive_war_with` 触发器（已平衡和测试）
- 使用 `previous_owner` 检查优先考虑原所有者（修复拜占庭案例）
- 可选参与 - 国家可以拒绝而不受惩罚
- 无资源成本 - 比完整系统更简单
- 宗主可以拒绝或取消附庸地位（解决利用 1）
- 资格检查自然防止世界大战（距离、外交关系等）

### 局势文件结构

文件：`game/in_game/common/situations/secessionist_rebellion.txt`


```

secessionist_rebellion = {
    monthly_spawn_chance = monthly_spawn_chance_unique
    hint_tag = hint_secessionist_rebellion
    
    can_start = {
        any_secessionist_rebel = {
            rebel_progress >= 0.80
            rebel_progress < 1.00
        }
    }
    
    can_end = {
        OR = {
            all_secessionist_rebels = {
                rebel_progress >= 1.00
            }
            all_secessionist_rebels = {
                rebel_progress < 0.80
            }
        }
    }
    
    visible = {
        OR = {
            # 关键限制：排名最高的相同文化国家或防御方的宿敌或包围网领袖（防止世界大战）
            AND = {
                # 排名最高的相同文化国家（仅一个）
                culture = rebel_culture
                # 这将在 on_start 中过滤为仅排名最高的
                game_setting:secessionist_notification_frequency != "disabled"
                game_setting:secessionist_notification_frequency != "rivals_only"
            }
            AND = {
                # 防御方的宿敌（战略利益）
                is_rival_of = defender
                NOT = { culture = rebel_culture }
                game_setting:secessionist_notification_frequency != "disabled"
                game_setting:secessionist_notification_frequency != "same_culture_only"
            }
            AND = {
                # 针对防御方的包围网领袖（通过包围网的战略利益）
                is_leader_of_international_organization = {
                    type = coalition
                    target = defender
                }
                NOT = { culture = rebel_culture }
                game_setting:secessionist_notification_frequency != "disabled"
                game_setting:secessionist_notification_frequency != "same_culture_only"
            }
            # 原所有者（如果是相同文化或宿敌或包围网领袖，则始终符合条件）
            AND = {
                any_rebelling_province = {
                    previous_owner = this
                }
                OR = {
                    culture = rebel_culture
                    is_rival_of = defender
                    is_leader_of_international_organization = {
                        type = coalition
                        target = defender
                    }
                }
            }
            # 防御方的宗主（始终可见以执行特殊行动）
            AND = {
                is_overlord_of = defender
                defender = {
                    is_subject_of = this
                }
            }
        }
    }
    
    on_start = {
        # 层级 1：排名最高的相同文化国家（优先级提供系统 - 一次只有一个可以购买）
        # 步骤 1：对所有相同文化国家进行排名并存储在全局列表中
        ordered_country = {
            limit = {
                culture = rebel_culture
                # 关键：属国取消资格 - 从排名中排除属国（遵循意大利战争先例）
                # 这防止遥远地区的小型分离主义属国被错误地优先考虑用于远距离的叛乱
                # 它还防止属国获取会导致它们加入针对其宗主的战争的干预权
                # 只有具有相同主要文化的独立国家才有资格进行类别 1 排名
                is_subject = no
                # 严格过滤检查
                # 注意：主要文化国家即使有停战协议也可以购买（支持自己文化独立的停战协议例外）
                NOT = { is_at_war_with = defender }
                # 停战协议例外：主要文化国家可以与防御方有停战协议（他们可以选择打破停战协议以支持其文化）
                NOT = { in_civil_war = yes }
                NOT = { is_during_bankruptcy = yes }
                diplomatic_capacity > 0
                distance_to_capital <= 5000
                can_join_as_attacker = yes
                gold >= (monthly_income_trade_and_tax * 2)
            }
            order_by = {
                # 排名公式：(原所有者奖励 * 100) + (邻国奖励 * 50) + (文化影响力 * 10) + (国家等级 * 5) - (到首都距离 / 10)
                # 关键：原所有者奖励（100 分）是最高优先级，修复当前系统缺陷
                # 示例：$TUR$（原所有者）获得 100 分，$ERE$（文化主导但不是原所有者）获得 0 分
                # 这确保 $TUR$ 排名高于 $ERE$，修复 $BYZ$ 案例中 $TUR$ 被完全忽视的问题
                value = {
                    add = {
                        if = {
                            limit = {
                                any_rebelling_province = { previous_owner = this }
                            }
                            value = 100  # 原所有者奖励 - 修复当前系统缺陷
                        }
                    }
                    add = {
                        if = {
                            limit = { is_neighbor_of = defender }
                            value = 50
                        }
                    }
                    add = {
                        value = cultural_influence
                        multiply = 10
                    }
                    add = {
                        value = country_rank_level
                        multiply = 5
                    }
                    subtract = {
                        value = distance_to_capital
                        divide = 10
                    }
                }
            }
            check_range_bounds = no
            # 将所有排名的相同文化国家存储在全局列表中，用于优先级提供系统
            add_to_global_variable_list = {
                name = secessionist_same_culture_queue_$rebellion_id$
                target = this
            }
        }
        
        # 步骤 2：仅向排名最高的国家扩展购买报价（优先级提供系统）
        ordered_country = {
            limit = {
                culture = rebel_culture
                # 关键：属国取消资格 - 从排名中排除属国（遵循意大利战争先例）
                is_subject = no
                # 必须在优先级提供列表中且尚未提供
                in_global_variable_list = secessionist_same_culture_queue_$rebellion_id$
                NOT = { has_variable = secessionist_offered_purchase_$rebellion_id$ }
                # 标准资格检查
                # 注意：主要文化国家即使有停战协议也可以购买（支持自己文化独立的停战协议例外）
                NOT = { is_at_war_with = defender }
                # 停战协议例外：主要文化国家可以与防御方有停战协议（他们可以选择打破停战协议以支持其文化）
                NOT = { in_civil_war = yes }
                NOT = { is_during_bankruptcy = yes }
                diplomatic_capacity > 0
                distance_to_capital <= 5000
                can_join_as_attacker = yes
                gold >= (monthly_income_trade_and_tax * 2)
            }
            order_by = {
                # 与上述相同的排名公式
                value = {
                    add = {
                        if = {
                            limit = {
                                any_rebelling_province = { previous_owner = this }
                            }
                            value = 100
                        }
                    }
                    add = {
                        if = {
                            limit = { is_neighbor_of = defender }
                            value = 50
                        }
                    }
                    add = {
                        value = cultural_influence
                        multiply = 10
                    }
                    add = {
                        value = country_rank_level
                        multiply = 5
                    }
                    subtract = {
                        value = distance_to_capital
                        divide = 10
                    }
                }
            }
            max = 1
            check_range_bounds = no
            add_to_global_variable_list = {
                name = secessionist_rebellion_voters
                target = this
            }
            # 标记为已提供并通知
            set_variable = {
                name = secessionist_offered_purchase_$rebellion_id$
                value = 1
            }
            # 设置过期计时器（30 天响应）
            set_variable = {
                name = secessionist_purchase_offer_expires_$rebellion_id$
                days = 30
            }
            # 立即通知（无延迟）
            trigger_event_non_silently = secessionist_rebellion.1
            set_variable = {
                name = secessionist_notification_cooldown_$rebellion_id$
                months = 6
            }
        }
        
        # 层级 2：防御方的宿敌（交错 5-15 天）
        every_country = {
            limit = {
                is_rival_of = defender
                NOT = { culture = rebel_culture }
                # 不是包围网领袖（包围网领袖在层级 2B 中单独处理）
                NOT = {
                    is_leader_of_international_organization = {
                        type = coalition
                        target = defender
                    }
                }
                # 严格过滤检查
                NOT = { is_at_war_with = defender }
                NOT = { has_truce_with = defender }
                NOT = { in_civil_war = yes }
                NOT = { is_during_bankruptcy = yes }
                diplomatic_capacity > 0
                NOT = { has_variable = secessionist_notification_cooldown_$rebellion_id$ }
                distance_to_capital <= 5000
                can_join_as_attacker = yes
                gold >= (monthly_income_trade_and_tax * 2)
                # 评估分数检查（对于宿敌，基于战略利益）
                evaluation_score >= 0
            }
            add_to_global_variable_list = {
                name = secessionist_rebellion_voters
                target = this
            }
            trigger_event_non_silently = {
                id = secessionist_rebellion.2
                days = { 5 15 }
            }
            set_variable = {
                name = secessionist_notification_cooldown_$rebellion_id$
                months = 6
            }
        }
        
        # 层级 2B：针对防御方的包围网领袖（交错 10-20 天，在常规宿敌之后）
        # 注意：包围网领袖通过包围网成员中最高 great_power_score 自动选择
        # （coalition.txt 具有 has_leader_country = yes 和 leader_type = country，但没有明确的 leader_change_method，
        # 因此游戏默认通过 great_power_score 选择最强成员）
        every_country = {
            limit = {
                # 针对防御方的包围网领袖（通过成员中最高 great_power_score 自动选择）
                is_leader_of_international_organization = {
                    type = coalition
                    target = defender
                }
                NOT = { culture = rebel_culture }
                # 严格过滤检查
                NOT = { is_at_war_with = defender }
                NOT = { has_truce_with = defender }
                NOT = { in_civil_war = yes }
                NOT = { is_during_bankruptcy = yes }
                diplomatic_capacity > 0
                NOT = { has_variable = secessionist_notification_cooldown_$rebellion_id$ }
                distance_to_capital <= 5000
                can_join_as_attacker = yes
                gold >= (monthly_income_trade_and_tax * 2)
                # 评估分数检查（对于包围网领袖，基于战略利益）
                evaluation_score >= 0
            }
            add_to_global_variable_list = {
                name = secessionist_rebellion_voters
                target = this
            }
            trigger_event_non_silently = {
                id = secessionist_rebellion.2_coalition
                days = { 10 20 }
            }
            set_variable = {
                name = secessionist_notification_cooldown_$rebellion_id$
                months = 6
            }
        }
        
        # 层级 3：防御方的宗主（立即通知以执行特殊行动）
        every_country = {
            limit = {
                is_overlord_of = defender
                defender = {
                    is_subject_of = this
                }
            }
            # 计算附庸的怀疑分数
            defender = {
                set_variable = {
                    name = vassal_suspicion_score
                    value = 0
                }
                # 模式 1：尽管有容量但未核心化领土
                add = {
                    if = {
                        limit = {
                            any_owned_location = {
                                NOT = { is_cored = yes }
                                any_secessionist_rebel = {
                                    any_rebelling_location = this
                                }
                            }
                            administrative_capacity > 0
                        }
                        value = 20
                    }
                }
                # 模式 2：尽管有资源但稳定度低
                add = {
                    if = {
                        limit = {
                            stability < 30
                            gold >= (monthly_income_trade_and_tax * 6)
                        }
                        value = 15
                    }
                }
                # 模式 3：加速叛军进度
                add = {
                    if = {
                        limit = {
                            any_secessionist_rebel = {
                                monthly_progress > 0.05
                            }
                        }
                        value = 25
                    }
                }
                # 模式 4：尽管有能力但不使用镇压叛军
                add = {
                    if = {
                        limit = {
                            any_secessionist_rebel = {
                                rebel_progress > 0.5
                            }
                            government_power >= 50
                        }
                        value = 10
                    }
                }
                # 模式 5：高动乱但无对策
                add = {
                    if = {
                        limit = {
                            any_owned_location = {
                                unrest >= 5
                                any_secessionist_rebel = {
                                    any_rebelling_location = this
                                }
                            }
                        }
                        value = 15
                    }
                }
            }
            # 立即通知宗主（带怀疑分数）
            trigger_event_non_silently = secessionist_rebellion.overlord_notification
            set_variable = {
                name = secessionist_notification_cooldown_$rebellion_id$
                months = 6
            }
        }
    }
    
    # 每月检查：如果当前国家拒绝或时间到期，推进优先级提供
    monthly = {
        # 检查是否有收到报价的国家拒绝或报价过期
        if = {
            limit = {
                # 检查是否有收到报价但尚未购买的国家
                any_country = {
                    in_global_variable_list = secessionist_same_culture_queue_$rebellion_id$
                    has_variable = secessionist_offered_purchase_$rebellion_id$
                    NOT = { has_variable = secessionist_intervention_right_$rebellion_id$ }  # 尚未购买
                    OR = {
                        has_variable = secessionist_refused_purchase_$rebellion_id$  # 已拒绝
                        var:secessionist_purchase_offer_expires_$rebellion_id$ <= 0  # 时间到期
                    }
                }
            }
            # 将报价扩展到下一个排名的国家（优先级提供系统）
            ordered_country = {
                limit = {
                    culture = rebel_culture
                    # 关键：属国取消资格 - 从排名中排除属国（遵循意大利战争先例）
                    is_subject = no
                    in_global_variable_list = secessionist_same_culture_queue_$rebellion_id$
                    NOT = { has_variable = secessionist_offered_purchase_$rebellion_id$ }  # 尚未提供
                    NOT = { has_variable = secessionist_intervention_right_$rebellion_id$ }  # 尚未购买
                    NOT = { has_variable = secessionist_refused_purchase_$rebellion_id$ }  # 尚未拒绝
                    # 标准资格检查
                    NOT = { is_at_war_with = defender }
                    NOT = { has_truce_with = defender }
                    NOT = { in_civil_war = yes }
                    NOT = { is_during_bankruptcy = yes }
                    diplomatic_capacity > 0
                    distance_to_capital <= 5000
                    can_join_as_attacker = yes
                    gold >= (monthly_income_trade_and_tax * 2)
                }
                order_by = {
                    # 相同的排名公式
                    value = {
                        add = {
                            if = {
                                limit = {
                                    any_rebelling_province = { previous_owner = this }
                                }
                                value = 100
                            }
                        }
                        add = {
                            if = {
                                limit = { is_neighbor_of = defender }
                                value = 50
                            }
                        }
                        add = {
                            value = cultural_influence
                            multiply = 10
                        }
                        add = {
                            value = country_rank_level
                            multiply = 5
                        }
                        subtract = {
                            value = distance_to_capital
                            divide = 10
                        }
                    }
                }
                max = 1
                check_range_bounds = no
                # 标记为已提供并将购买报价扩展到下一个国家
                set_variable = {
                    name = secessionist_offered_purchase_$rebellion_id$
                    value = 1
                }
                set_variable = {
                    name = secessionist_purchase_offer_expires_$rebellion_id$
                    days = 30
                }
                trigger_event_non_silently = secessionist_rebellion.1
                set_variable = {
                    name = secessionist_notification_cooldown_$rebellion_id$
                    months = 6
                }
            }
        }
    }
}

```


**优先级提供系统说明：**
- 当局势开始时，所有符合条件的相同文化国家被排名并存储在 `secessionist_same_culture_queue_$rebellion_id$` 全局列表中
- 只有排名最高的国家最初收到购买报价（用 `secessionist_offered_purchase_$rebellion_id$` 变量标记）- 这实现了"优先拒绝权"
- 如果排名最高的国家拒绝（设置 `secessionist_refused_purchase_$rebellion_id$`）或 30 天到期（`secessionist_purchase_offer_expires_$rebellion_id$ <= 0`），每月检查将报价扩展到下一个排名的国家
- 下一个排名的国家收到购买报价并可以购买
- 顺序报价继续，直到有人购买（设置 `secessionist_intervention_right_$rebellion_id$`）或所有符合条件的国家拒绝
- 这确保一次只有一个相同文化国家可以购买，防止多个国家同时全部加入
- 购买行动的 `allow` 块检查 `has_variable = secessionist_offered_purchase_$rebellion_id$` 以确保只有收到当前报价的国家可以购买

**拒绝处理：**
- 国家可以通过在通知事件中选择"拒绝"选项明确拒绝（设置 `secessionist_refused_purchase_$rebellion_id$` 变量）
- 如果国家在 30 天内不购买，报价过期（`secessionist_purchase_offer_expires_$rebellion_id$ <= 0`），它们被视为已拒绝
- 每月检查检测拒绝/过期的报价并将报价扩展到下一个排名的国家（优先级提供系统推进）
- 一旦有国家购买，优先级提供系统停止（不再向其他相同文化国家提供报价）

### 行动文件结构

文件：`game/in_game/common/generic_actions/secessionist_rebellion.txt`


```

purchase_secessionist_intervention_right = {
    type = situation
    show_message = no
    
    potential = {
        # 关键：只有排名最高的相同文化国家或防御方的宿敌或包围网领袖才符合条件
        OR = {
            AND = {
                culture = rebel_culture
                # 检查此国家是否是排名最高的相同文化国家
                # （此检查在局势可见性中完成，但我们在这里也验证）
            }
            is_rival_of = defender
            AND = {
                # 针对防御方的包围网领袖（包围网领袖通过成员中最高 great_power_score 自动选择）
                is_leader_of_international_organization = {
                    type = coalition
                    target = defender
                }
                NOT = { culture = rebel_culture }
            }
        }
    }
    
    allow = {
        scope:actor = {
            NOT = { is_at_war_with = defender }
            # 停战协议例外：主要文化国家即使有停战协议也可以购买（支持自己文化独立的停战协议例外）
            # 宿敌和包围网领袖必须没有停战协议，但主要文化国家可以有停战协议
            OR = {
                # 主要文化国家 - 可以有停战协议
                culture = rebel_culture
                # 宿敌或包围网领袖 - 必须没有停战协议
                AND = {
                    OR = {
                        is_rival_of = defender
                        is_leader_of_international_organization = {
                            type = coalition
                            target = defender
                        }
                    }
                    NOT = { has_truce_with = defender }
                }
            }
            NOT = { in_civil_war = yes }
            NOT = { is_during_bankruptcy = yes }
            diplomatic_capacity > 0
            gold >= price:secessionist_intervention_right_price
            can_join_as_attacker = yes
            # 优先级提供系统：只有收到当前报价的国家才允许购买（优先拒绝权）
            has_variable = secessionist_offered_purchase_$rebellion_id$
            NOT = { has_variable = secessionist_refused_purchase_$rebellion_id$ }  # 尚未明确拒绝
        }
    }
    
    automation_tick = never
    automation_tick_frequency = 12
    ai_tick = monthly
    ai_tick_frequency = 3
    
    price = price:secessionist_intervention_right_price
    
    cooldown = {
        type = purchase_secessionist_intervention_right_cooldown_key
        years = 2
    }
    
    select_trigger = {
        looking_for_a = situation
        interaction_source_list = {
            situation:secessionist_rebellion = {
                add_to_list = source
            }
        }
        target_flag = recipient
        name = "choose_situation"
        column = {
            data = name
        }
        visible = {
            situation:secessionist_rebellion = this
            situation_is_active = yes
        }
    }
    
    effect = {
        # 授予干预权（存储为变量）
        scope:actor = {
            set_variable = {
                name = secessionist_intervention_right_$rebellion_id$
                value = 1
            }
            # 移除已提供标志（购买成功，优先级提供系统在此停止）
            remove_variable = secessionist_offered_purchase_$rebellion_id$
        }
        # 为叛军分配杜卡特用于雇佣兵雇佣（以及可能的秘密正规军训练）
        defender = {
            every_secessionist_rebel = {
                set_variable = {
                    name = rebel_ducats
                    add = price:secessionist_intervention_right_price
                }
                # 可选：秘密正规军训练（受内置游戏限制约束）
                # 这将单独实施，尊重叛军国家形成时的限制
            }
        }
    }
    
    ai_will_do = {
        # 基础评估分数计算
        add = {
            desc = "land_desire"
            value = "scope:actor.land_desire(defender)"
            multiply = 100
        }
        subtract = {
            desc = "distance_penalty"
            value = "scope:actor.distance_to_capital(defender.capital)"
            divide = 10
        }
        add = {
            if = {
                limit = {
                    any_rebelling_province = {
                        previous_owner = scope:actor
                    }
                }
                value = 50
                desc = "original_owner_bonus"
            }
        }
        add = {
            if = {
                limit = {
                    scope:actor = { is_neighbor_of = defender }
                }
                value = 25
                desc = "neighbor_bonus"
            }
        }
        add = {
            if = {
                limit = {
                    scope:actor.culture = rebel_culture
                }
                value = 10
                desc = "same_culture_bonus"
            }
        }
        add = {
            if = {
                limit = {
                    scope:actor = { is_rival_of = defender }
                }
                value = 100
                desc = "rival_bonus"
            }
        }
        # Cost considerations
        subtract = {
            if = {
                limit = {
                    scope:actor = { is_during_bankruptcy = yes }
                }
                value = 1000
                desc = "bankruptcy"
            }
        }
        subtract = {
            if = {
                limit = {
                    scope:actor = { num_loans > 0 }
                }
                value = 500
                desc = "loans"
            }
        }
    }
}

# 当叛乱爆发时加入内战的行动
join_secessionist_civil_war = {
    type = situation
    show_message = no
    
    potential = {
        scope:actor = {
            has_variable = secessionist_intervention_right_$rebellion_id$
        }
    }
    
    allow = {
        scope:actor = {
            NOT = { is_at_war_with = defender }
            can_join_as_attacker = yes
        }
        exists = scope:target
        scope:target = {
            is_civil_war_for = defender
            any_attacker = {
                is_rebel_country = yes
                rebel_type = secessionist
            }
        }
    }
    
    select_trigger = {
        looking_for_a = war
        target_flag = target
        name = "choose_civil_war_to_join"
        interaction_source_list = {
            defender = {
                every_current_war = {
                    limit = {
                        is_civil_war_for = defender
                        any_attacker = {
                            is_rebel_country = yes
                            rebel_type = secessionist
                        }
                    }
                    add_to_list = source
                }
            }
        }
        visible = {
            is_civil_war_for = defender
            any_attacker = {
                is_rebel_country = yes
                rebel_type = secessionist
            }
            can_join_as_attacker = scope:actor
        }
    }
    
    effect = {
        scope:target = {
            scope:actor = {
                join_war_as_attacker = {
                    war = prev
                    reason = intervene
                }
            }
        }
        # 破坏停战协议惩罚减少（选项 B 或 C）：
        # 如果主要文化国家与防御方有停战协议，在加入战争时应用减少的惩罚
        # 选项 B：减少的惩罚（50% 严重程度）
        # 选项 C：与更高购买成本结合（已在价格计算中应用）
        if = {
            limit = {
                scope:actor = {
                    culture = rebel_culture
                    has_truce_with = defender
                }
            }
            # 应用减少的破坏停战协议惩罚（选项 B 或 C）
            scope:actor = {
                # 减少的稳定度惩罚：-25 而不是 -50（50% 严重程度）
                add_stability = -25
                # 减少的厌战度：+0.5 而不是 +1（50% 严重程度）
                add_war_exhaustion = 0.5
                # 减少的敌意：+12 而不是 +25（50% 严重程度）
                defender = {
                    add_antagonism = {
                        target = scope:actor
                        value = 12
                    }
                }
            }
        }
        # 移除干预权变量（已使用）
        scope:actor = {
            remove_variable = secessionist_intervention_right_$rebellion_id$
        }
    }
    
    ai_will_do = {
        add = {
            value = 50
            desc = BASE_VALUE
        }
        add = {
            desc = "game_concept_opinion"
            scope:actor = {
                value = "opinion(defender)"
                divide = -5
            }
        }
        add = {
            desc = "game_concept_war_exhaustion"
            scope:actor = {
                value = war_exhaustion
                divide = -2
            }
        }
    }
}

```


### 价格计算

文件：`game/in_game/common/prices/secessionist_rebellion.txt`


```

secessionist_intervention_right_price = {
    base = {
        value = 100
    }
    multiply = {
        # 随经济基础缩放（如 scaled_gold）
        value = scope:actor.monthly_income_trade_and_tax
        multiply = 2
    }
    add = {
        # 距离惩罚
        value = scope:actor.distance_to_capital(defender.capital)
        divide = 100
    }
    add = {
        # 文化传统惩罚（反映间谍网建设难度）
        value = defender.cultural_tradition
        divide = scope:actor.cultural_influence
        multiply = 50
    }
    add = {
        # 反间谍惩罚
        value = defender.counterespionage_modifier
        multiply = 100
    }
    multiply = {
        # 宿敌倍数（宿敌为 1.5x，因为他们缺乏文化联系）
        if = {
            limit = {
                scope:actor = { is_rival_of = defender }
                NOT = { scope:actor.culture = rebel_culture }
            }
            value = 1.5
        }
    }
    multiply = {
        # TRUCE EXCEPTION COST MULTIPLIER (Option A or C): Primary culture countries with truce pay higher cost
        # Option A (Recommended): 2.0x multiplier
        # Option C (Combined): 1.5x multiplier (if combined with reduced penalties)
        if = {
            limit = {
                scope:actor = {
                    culture = rebel_culture
                    has_truce_with = defender
                }
            }
            value = 2.0  # Option A: Higher purchase cost (2.0x). Change to 1.5 for Option C (Combined Approach)
        }
    }
}

```


### 游戏设置

文件：`game/main_menu/common/game_rules/00_game_rules.txt`


```

rule_secessionist_notification_frequency = {
    default = setting_all_notifications
    
    setting_all_notifications = {
        # No filtering beyond strict eligibility checks
    }
    
    setting_same_culture_only = {
        # Only top-ranked same-culture notifications
        # Implemented via visibility trigger: culture = rebel_culture
    }
    
    setting_rivals_only = {
        # Only rival notifications
        # Implemented via visibility trigger: is_rival_of = defender
    }
    
    setting_original_owner_only = {
        # Only original owner notifications (if top-ranked same-culture)
        # Implemented via visibility trigger: any_rebelling_province = { previous_owner = this }
    }
    
    setting_disabled = {
        # No notifications
        # Implemented via visibility trigger: game_setting != "disabled"
    }
}

```


### 评估分数计算

基于意大利战争优先级计算模式：


```

evaluation_score = {
    value = 0
    # 基础因素
    add = {
        value = land_desire(defender)
        multiply = 100
    }
    add = {
        value = conquer_desire(defender)
        multiply = 50
    }
    # 距离惩罚
    subtract = {
        value = distance_to_capital(defender.capital)
        divide = 10
    }
    # 优先级奖励
    add = {
        if = {
            limit = {
                any_rebelling_province = { previous_owner = this }
            }
            value = 50
        }
    }
    add = {
        if = {
            limit = { is_neighbor_of = defender }
            value = 25
        }
    }
    add = {
        if = {
            limit = { culture = rebel_culture }
            value = 10
        }
    }
    # 力量等级奖励
    add = {
        value = country_rank_level
        multiply = 10
    }
    # 战略利益（对于宿敌）
    add = {
        if = {
            limit = {
                is_rival_of = defender
            }
            value = 100
        }
    }
}

```


### 排名最高的相同文化国家的排名公式

用于 `ordered_country` 以选择单个排名最高的相同文化国家。**关键：** 此公式优先考虑原所有权（100 分）而非文化主导地位，修复当前系统的缺陷，其中原所有者（例如，拜占庭案例中的奥斯曼）被完全忽视，而优先选择文化主导国家（例如，埃雷特纳）。


```

ranking_score = {
    value = 0
    # 原所有者奖励（最高优先级）- 修复当前系统缺陷
    # 示例：奥斯曼（原所有者）获得 100 分，埃雷特纳（非原所有者）获得 0 分
    # 这确保奥斯曼排名高于埃雷特纳，修复拜占庭案例中奥斯曼被忽视的问题
    add = {
        if = {
            limit = {
                any_rebelling_province = { previous_owner = this }
            }
            value = 100
        }
    }
    # 邻国奖励（高优先级）
    add = {
        if = {
            limit = { is_neighbor_of = defender }
            value = 50
        }
    }
    # 文化影响力（中等优先级）
    add = {
        value = cultural_influence
        multiply = 10
    }
    # 国家等级（低优先级）
    add = {
        value = country_rank_level
        multiply = 5
    }
    # 距离惩罚（减少远距离国家的分数）
    subtract = {
        value = distance_to_capital(defender.capital)
        divide = 10
    }
}

```


### 雇佣兵雇佣机制

当购买干预权时，杜卡特被分配给叛军：


```

on_intervention_right_purchased = {
    defender = {
        every_secessionist_rebel = {
            # 分配杜卡特
            set_variable = {
                name = rebel_ducats
                add = price:secessionist_intervention_right_price
            }
            # 雇佣雇佣兵（如果有足够的杜卡特）
            if = {
                limit = {
                    var:rebel_ducats >= mercenary_hiring_cost
                }
                hire_mercenaries = {
                    amount = var:rebel_ducats
                    gathering_time = MERCENARY_GATHERING_TIME_BASE  # 14 天
                }
            }
        }
    }
}

```


When rebellion breaks out (`rebel_progress >= 1.00`):


```

on_rebellion_outbreak = {
    defender = {
        every_secessionist_rebel = {
            # Create rebel country via start_civil_war (creates normal country with country_type = location)
            # Note: Rebels are rebel entities (not countries) before breaking out. start_civil_war creates a normal country automatically
            start_civil_war = {
                save_scope_as = rebel_country
                # Rebel country initially gets troops through raise_levies (levies from controlled provinces, based on population and control - NOT random)
                if = {
                    limit = {
                        PREV.owner = {
                            at_war = yes
                        }
                    }
                    every_province = {
                        raise_levies = {
                            instant = yes
                            type = army
                        }
                    }
                }
            }
            # CRITICAL MECHANIC: Check if primary culture country purchased Intervention Right
            # If yes, rebel country joins that country as Secessionist subject, and that country becomes war leader
            # If no, rebel country becomes independent and leads the war
            if = {
                limit = {
                    # Check if any primary culture country purchased Intervention Right
                    any_country = {
                        culture = rebel_culture
                        has_variable = secessionist_intervention_right_$rebellion_id$
                    }
                }
                # Find the primary culture country that purchased Intervention Right (should be only one due to Priority Offer System)
                ordered_country = {
                    limit = {
                        culture = rebel_culture
                        has_variable = secessionist_intervention_right_$rebellion_id$
                    }
                    order_by = {
                        # Same ranking formula as purchase eligibility
                        value = {
                            add = {
                                if = {
                                    limit = {
                                        any_rebelling_province = { previous_owner = this }
                                    }
                                    value = 100
                                }
                            }
                            add = {
                                if = {
                                    limit = { is_neighbor_of = defender }
                                    value = 50
                                }
                            }
                            add = {
                                value = cultural_influence
                                multiply = 10
                            }
                            add = {
                                value = country_rank_level
                                multiply = 5
                            }
                            subtract = {
                                value = distance_to_capital
                                divide = 10
                            }
                        }
                    }
                    max = 1
                    check_range_bounds = no
                    # Rebel country joins this country as Secessionist subject
                    scope:rebel_country = {
                        create_subject = {
                            type = secessionist
                            subject = prev
                        }
                    }
                    # This country becomes attacker war leader (not the rebel country/vassal)
                    # This prevents vassal gaming where vassals take all benefits before their overlord
                    set_attacker_war_leader = this
                }
            }
            else = {
                # No primary culture country purchased Intervention Right
                # Rebel country becomes independent and leads the war as attacker war leader
                # Historical Example: Greek War of Independence (1821) - Byzantine Empire was destroyed in 1453,
                # so no Greek primary culture country existed. Greek rebels became independent and won the war
                # with support from Russia, Britain, and France (rivals of Ottoman Empire).
                # When Greece won, it became a new independent nation, not a subject of any country.
                scope:rebel_country = {
                    set_attacker_war_leader = this
                }
            }
            # If rebels have ducats, they have mercenaries ready
            if = {
                limit = {
                    has_variable = rebel_ducats
                    var:rebel_ducats > 0
                }
                # Rebels have mercenaries ready (gathered before revolution)
                spawn_rebel_army = {
                    has_mercenaries = yes
                    mercenary_amount = var:rebel_ducats
                }
            }
            else = {
                # Rebels only have levies (much weaker)
                spawn_rebel_army = {
                    has_mercenaries = no
                }
            }
        }
        # Notify countries with purchased Intervention Rights
        every_country = {
            limit = {
                has_variable = secessionist_intervention_right_$rebellion_id$
            }
            trigger_event_non_silently = secessionist_rebellion.3
        }
    }
}

```


### 秘密正规军训练机制（可选增强）

**平衡解决方案 - 内置游戏机制：** 当分离主义叛军通过 `start_civil_war` 爆发时，它们形成一个叛军国家（具有 `country_type = location` 的普通国家）。此叛军国家自动受内置游戏限制约束。**注意：** 叛军在爆发前是叛军实体（不是国家），因此不能将它们视为无地国家。当它们爆发时，它们自动成为普通国家 - 无需国家类型转换。


```

# 当购买干预权时，杜卡特可以资助秘密正规军训练
on_intervention_right_purchased_secret_training = {
    defender = {
        every_secessionist_rebel = {
            # 为秘密正规军训练分配杜卡特
            set_variable = {
                name = rebel_secret_training_ducats
                add = price:secessionist_intervention_right_price
            }
            # 秘密训练发生在叛乱爆发之前（地图上不可见）
            # 训练受叛军国家形成时的内置限制约束
        }
    }
}

# 当叛乱爆发时，秘密训练的正规军生成（受内置限制约束）
on_rebellion_outbreak_secret_training = {
    defender = {
        every_secessionist_rebel = {
            # 通过 start_civil_war 创建叛军国家（创建具有 country_type = location 的普通国家）
            # 注意：叛军在爆发前是叛军实体（不是国家）。start_civil_war 自动创建普通国家
            start_civil_war = {
                save_scope_as = rebel_country
            }
            # 叛军国家受内置游戏限制约束：
            # 1. 人力池：受叛军控制地区的人口和建筑限制
            #    （兵营、团营提供 local_manpower）
            # 2. 招募能力：受具有 can_recruit_regiment_in_this_location 的地区限制
            #    （需要军事建筑如兵营、团营、军士院、战士神庙）
            # 3. 训练速度：受每个地区的 max_regiments_trained_at_same_time 限制
            #    （通常每个有招募建筑的地区 1 个团）
            # 4. 经济约束：正规军需要维护成本，随叛军国家的收入缩放
            
            # 生成秘密训练的正规军（受叛军国家的内置限制约束）
            if = {
                limit = {
                    has_variable = rebel_secret_training_ducats
                    var:rebel_secret_training_ducats > 0
                }
                # 根据叛军国家的限制计算可以生成多少正规军
                # 这确保自然平衡 - 无需手动百分比上限
                spawn_secret_regulars = {
                    ducats = var:rebel_secret_training_ducats
                    # 自动应用的限制：
                    # - 不能超过叛军国家的人力池
                    # - 不能超过招募能力（有招募建筑的地区）
                    # - 不能超过训练速度（每个地区的团）
                    # - 必须负担得起（维护成本随叛军国家的收入缩放）
                }
            }
        }
    }
}

```


**关键实现要点：**
- 秘密正规军训练尊重叛军国家的内置限制（人力、招募能力、训练速度、经济约束）
- 无需手动百分比上限 - 游戏的现有国家系统提供自然平衡
- 叛军国家的军事能力与人口、经济基础和招募建筑成比例
- 这利用现有游戏系统实现自然平衡，无需复杂的手动限制

### 加入内战

当拥有已购买干预权的国家想要加入内战时：


```

# 这使用现有的干预属国内战模式
# 国家使用 join_secessionist_civil_war 行动，它调用：
join_war_as_attacker = {
    war = scope:target  # 内战
    reason = intervene
}

# 破坏停战协议惩罚减少（选项 B 或 C）：
# 如果主要文化国家与防御方有停战协议，在加入战争时应用减少的惩罚
# 选项 B：减少的惩罚（50% 严重程度）
# 选项 C：与更高购买成本结合（已在价格计算中应用）
if = {
    limit = {
        scope:actor = {
            culture = rebel_culture
            has_truce_with = defender
            has_variable = secessionist_intervention_right_$rebellion_id$
        }
    }
    # 应用减少的破坏停战协议惩罚（选项 B 或 C）
    scope:actor = {
        # 减少的稳定度惩罚：-25 而不是 -50（50% 严重程度）
        add_stability = -25
        # Reduced war exhaustion: +0.5 instead of +1 (50% severity)
        add_war_exhaustion = 0.5
        # Reduced antagonism: +12 instead of +25 (50% severity)
        defender = {
            add_antagonism = {
                target = scope:actor
                value = 12
            }
        }
    }
}

```


This is identical to how `intervene_in_subject_civil_war` works - countries join existing civil wars using `join_war_as_attacker` with `reason = intervene`. However, if Option B or C is implemented, primary culture countries with a truce receive reduced truce-breaking penalties (50% severity) when joining the war, acknowledging that supporting one's own culture's independence is a form of asymmetric warfare.

### 宗主特殊行动（受战国"下克上"机制启发）

当分离主义叛乱局势在附庸领土上开始时，宗主收到特殊通知并可以采取局势行动（类似于战国幕府行动）。这些行动集成到分离主义局势框架中。

文件：`game/in_game/common/generic_actions/secessionist_rebellion.txt`（添加到现有文件）


```

# 宗主行动（受战国机制启发，重用现有逻辑）
secessionist_cancel_vassal_status = {
    type = situation
    show_message = no
    
    potential = {
        scope:actor = {
            is_overlord_of = defender
        }
        defender = {
            is_subject_of = scope:actor
            is_subject_type = vassal
        }
    }
    
    allow = {
        scope:actor = {
            NOT = { is_at_war_with = defender }
        }
        # 怀疑分数检查（高怀疑表明故意破坏）
        situation:secessionist_rebellion = {
            var:vassal_suspicion_score >= 50
        }
        # 检查宗主是否可以取消（重用现有 vassal.txt 逻辑：overlord_can_cancel = yes）
        defender = {
            subject_type_is_not_locked = no
        }
    }
    
    automation_tick = never
    automation_tick_frequency = 12
    ai_tick = monthly
    ai_tick_frequency = 6
    
    select_trigger = {
        looking_for_a = situation
        interaction_source_list = {
            situation:secessionist_rebellion = {
                add_to_list = source
            }
        }
        target_flag = recipient
        name = "choose_situation"
        column = { data = name }
        visible = {
            situation:secessionist_rebellion = this
            situation_is_active = yes
            defender = {
                is_subject_of = scope:actor
            }
        }
    }
    
    effect = {
        # 宗主立即取消附庸地位（重用现有 cancel_subject 机制）
        # 注意：使用现有 cancel_subject 效果（与外交行动 CANCEL_vassal 相同）
        scope:actor = {
            cancel_subject = {
                target = defender
                type = vassal
            }
        }
    }
    
    ai_will_do = {
        add = {
            value = situation:secessionist_rebellion.var:vassal_suspicion_score
            multiply = 2
        }
        subtract = {
            value = scope:actor.opinion(defender)
            divide = 5
        }
    }
}

secessionist_create_disloyal_subject_cb = {
    type = situation
    show_message = no
    
    potential = {
        scope:actor = {
            is_overlord_of = defender
        }
        defender = {
            is_subject_of = scope:actor
        }
    }
    
    allow = {
        scope:actor = {
            NOT = { is_at_war_with = defender }
        }
        # Suspicion score reduces loyalty, making cb_disloyal_subject available
        situation:secessionist_rebellion = {
            var:vassal_suspicion_score >= 30
        }
        # Check if vassal loyalty is below 50 (requirement for cb_disloyal_subject)
        defender = {
            subject_loyalty < 50
        }
        # Check if CB doesn't already exist
        scope:actor = {
            NOT = {
                has_casus_belli_of_type_on = {
                    type = casus_belli:cb_disloyal_subject
                    target = defender
                }
            }
        }
    }
    
    automation_tick = never
    automation_tick_frequency = 12
    ai_tick = monthly
    ai_tick_frequency = 6
    
    select_trigger = {
        looking_for_a = situation
        interaction_source_list = {
            situation:secessionist_rebellion = {
                add_to_list = source
            }
        }
        target_flag = recipient
        name = "choose_situation"
        column = { data = name }
        visible = {
            situation:secessionist_rebellion = this
            situation_is_active = yes
            defender = {
                is_subject_of = scope:actor
            }
        }
    }
    
    effect = {
        # Overlord gains CB against vassal using existing cb_disloyal_subject (reuses existing CB creation logic)
        # Note: This action is optional - if suspicion score reduces loyalty below 50, cb_disloyal_subject becomes available automatically
        # This action allows overlord to manually create CB if suspicion is high but loyalty hasn't dropped enough yet
        scope:actor = {
            add_casus_belli = {
                type = casus_belli:cb_disloyal_subject
                target = defender
            }
        }
    }
    
    ai_will_do = {
        add = {
            value = situation:secessionist_rebellion.var:vassal_suspicion_score
        }
        subtract = {
            value = scope:actor.opinion(defender)
            divide = 5
        }
    }
}

secessionist_summon_to_court = {
    type = situation
    show_message = no
    
    potential = {
        scope:actor = {
            is_overlord_of = defender
        }
        defender = {
            is_subject_of = scope:actor
            has_ruler = yes
        }
    }
    
    allow = {
        scope:actor = {
            NOT = { is_at_war_with = defender }
            at_war = no
        }
    }
    
    automation_tick = never
    automation_tick_frequency = 12
    ai_tick = monthly
    ai_tick_frequency = 6
    
    cooldown = {
        type = secessionist_summon_to_court
        months = 6
    }
    
    price = price:secessionist_summon_to_court
    
    select_trigger = {
        looking_for_a = situation
        interaction_source_list = {
            situation:secessionist_rebellion = {
                add_to_list = source
            }
        }
        target_flag = recipient
        name = "choose_situation"
        column = { data = name }
        visible = {
            situation:secessionist_rebellion = this
            situation_is_active = yes
            defender = {
                is_subject_of = scope:actor
            }
        }
    }
    
    effect = {
        # Overlord demands vassal ruler appear at court (like sengoku_summon_to_court)
        defender = {
            trigger_event_non_silently = secessionist_rebellion.overlord_summon_to_court
        }
    }
    
    ai_will_do = {
        add = {
            value = situation:secessionist_rebellion.var:vassal_suspicion_score
            divide = 2
        }
    }
}

secessionist_revoke_territories = {
    type = situation
    show_message = no
    
    potential = {
        scope:actor = {
            is_overlord_of = defender
        }
        defender = {
            is_subject_of = scope:actor
        }
    }
    
    allow = {
        scope:actor = {
            NOT = { is_at_war_with = defender }
            at_war = no
        }
    }
    
    automation_tick = never
    automation_tick_frequency = 12
    ai_tick = monthly
    ai_tick_frequency = 6
    
    cooldown = {
        type = secessionist_revoke_territories
        months = 6
    }
    
    price = price:secessionist_revoke_territories
    
    select_trigger = {
        looking_for_a = situation
        interaction_source_list = {
            situation:secessionist_rebellion = {
                add_to_list = source
            }
        }
        target_flag = recipient
        name = "choose_situation"
        column = { data = name }
        visible = {
            situation:secessionist_rebellion = this
            situation_is_active = yes
            defender = {
                is_subject_of = scope:actor
            }
        }
    }
    
    select_trigger = {
        looking_for_a = location
        target_flag = target_location
        source = defender
        name = "choose_location"
        column = { data = name }
        visible = {
            owner = defender
            is_capital = no
            # Only locations with Secessionist rebels or high unrest
            OR = {
                any_secessionist_rebel = {
                    any_rebelling_location = this
                }
                unrest >= 5
            }
        }
    }
    
    effect = {
        # Overlord revokes specific territories from vassal (like sengoku_revoke_clans_land)
        if = {
            limit = { exists = scope:target_location }
            scope:target_location = {
                change_location_owner = scope:actor
            }
            defender = {
                trigger_event_non_silently = secessionist_rebellion.overlord_revoked_territories
            }
        }
    }
    
    ai_will_do = {
        add = {
            value = situation:secessionist_rebellion.var:vassal_suspicion_score
        }
    }
}

secessionist_refuse_to_join_war = {
    type = situation
    show_message = no
    
    potential = {
        scope:actor = {
            is_overlord_of = defender
        }
        defender = {
            is_subject_of = scope:actor
        }
    }
    
    allow = {
        # Only available when rebellion breaks out (civil war starts)
        defender = {
            any_current_war = {
                is_civil_war_for = defender
                any_attacker = {
                    is_rebel_country = yes
                    rebel_type = secessionist
                }
            }
        }
    }
    
    automation_tick = never
    automation_tick_frequency = 12
    ai_tick = monthly
    ai_tick_frequency = 1
    
    select_trigger = {
        looking_for_a = situation
        interaction_source_list = {
            situation:secessionist_rebellion = {
                add_to_list = source
            }
        }
        target_flag = recipient
        name = "choose_situation"
        column = { data = name }
        visible = {
            situation:secessionist_rebellion = this
            situation_is_active = yes
            defender = {
                is_subject_of = scope:actor
            }
        }
    }
    
    effect = {
        # Overlord explicitly refuses to join vassal's defensive war
        scope:actor = {
            set_variable = {
                name = secessionist_refused_to_join_war_$rebellion_id$
                value = 1
            }
        }
        # Remove overlord from war if already joined
        defender = {
            every_current_war = {
                limit = {
                    is_civil_war_for = defender
                    any_attacker = {
                        is_rebel_country = yes
                        rebel_type = secessionist
                    }
                    any_defender = { this = scope:actor }
                }
                scope:actor = {
                    leave_war = {
                        war = prev
                        actor = this
                    }
                }
            }
        }
    }
    
    ai_will_do = {
        add = {
            value = situation:secessionist_rebellion.var:vassal_suspicion_score
            multiply = 3
        }
        subtract = {
            value = scope:actor.opinion(defender)
            divide = 3
        }
    }
}

```


### 怀疑分数计算

怀疑分数在分离主义叛乱局势开始时计算，存储为局势变量：


```

on_secessionist_situation_start = {
    if = {
        limit = {
            defender = {
                is_subject = yes
            }
        }
        # 计算附庸的怀疑分数
        set_variable = {
            name = vassal_suspicion_score
            value = 0
        }
        # 模式 1：尽管有容量但未核心化领土
        add = {
            if = {
                limit = {
                    defender = {
                        any_owned_location = {
                            NOT = { is_cored = yes }
                            any_secessionist_rebel = {
                                any_rebelling_location = this
                            }
                        }
                        administrative_capacity > 0
                    }
                }
                value = 20
            }
        }
        # 模式 2：尽管有资源但稳定度低
        add = {
            if = {
                limit = {
                    defender = {
                        stability < 30
                        gold >= (monthly_income_trade_and_tax * 6)
                    }
                }
                value = 15
            }
        }
        # 模式 3：加速叛军进度
        add = {
            if = {
                limit = {
                    defender = {
                        any_secessionist_rebel = {
                            monthly_progress > 0.05  # 比正常快
                        }
                    }
                }
                value = 25
            }
        }
        # 模式 4：尽管有能力但不使用镇压叛军
        add = {
            if = {
                limit = {
                    defender = {
                        any_secessionist_rebel = {
                            rebel_progress > 0.5
                        }
                        government_power >= 50
                    }
                }
                value = 10
            }
        }
        # 模式 5：高动乱但无对策
        add = {
            if = {
                limit = {
                    defender = {
                        any_owned_location = {
                            unrest >= 5
                            any_secessionist_rebel = {
                                any_rebelling_location = this
                            }
                        }
                    }
                }
                value = 15
            }
        }
        # 计算怀疑分数后，如果怀疑度高，减少附庸忠诚度
        # 这使 cb_disloyal_subject 在忠诚度降至 50 以下时自动可用
        # 注意：cb_disloyal_subject 在 subject_loyalty < 50 时自动可用（无需手动创建）
        # 但是，创建 CB 仍然需要时间/1 个外交官，但 CB 在达到阈值时自动可用
        if = {
            limit = {
                var:vassal_suspicion_score >= 30
            }
            defender = {
                add_subject_loyalty = {
                    value = -20
                    # 高怀疑的额外惩罚
                    if = {
                        limit = {
                            var:vassal_suspicion_score >= 50
                        }
                        add = -10
                    }
                }
            }
        }
    }
}

```


**关键实现要点：**
- 怀疑分数减少附庸忠诚度（高怀疑 = -20 到 -30 忠诚度），这可以自动触发现有 `cb_disloyal_subject` 可用性（如果忠诚度降至 50 以下，无需手动创建 CB）
- 宗主行动重用现有游戏机制：
- **取消附庸地位：** 使用现有的 `vassal.txt` 中的 `overlord_can_cancel = yes` 和 `cancel_subject` 效果
- **创建不忠属国 CB：** 使用现有的 `cb_disloyal_subject`（需要 `subject_loyalty < 50`）。怀疑分数减少忠诚度，使 CB 自动可用。
- **夺取领土：** 使用现有的 `seize_location_from_subject` 国家交互，具有相同的忠诚度/自由欲望检查

### 文件结构摘要

**必需文件：**
1. `game/in_game/common/situations/secessionist_rebellion.txt` - 局势定义
2. `game/in_game/common/generic_actions/secessionist_rebellion.txt` - 干预权购买行动和加入战争行动
3. `game/in_game/common/prices/secessionist_rebellion.txt` - 价格计算
4. `game/main_menu/common/game_rules/00_game_rules.txt` - 游戏设置（添加到现有文件）
5. `game/in_game/events/situations/secessionist_rebellion.txt` - 通知的事件文件
6. 所有新字符串的本地化文件

**关键实现要点：**
- 使用带排名公式的 `ordered_country`，`max = 1` 选择排名最高的相同文化国家（如意大利战争）
- **关键：属国取消资格** - 在 `ordered_country` 限制子句中使用 `is_subject = no` 从类别 1 排名中排除属国（遵循意大利战争先例）。这防止遥远地区的小型分离主义属国被错误地优先考虑用于远距离的叛乱（例如，蒙古地区的蒙古文化分离主义属国被优先考虑用于格鲁吉亚的叛乱）。它还防止属国获取会导致它们加入针对其宗主的战争的干预权，这会破坏它们的属国关系（例如，当西西里叛军出现在法兰西控制的领土时，法兰西-西西里联合统治破裂）。只有具有相同主要文化的独立国家才有资格进行类别 1 排名。
- **关键：** 排名公式优先考虑原所有权（100 分）而非文化主导地位，修复当前系统缺陷，其中原所有者（例如，拜占庭案例中的奥斯曼）被完全忽视，而优先选择文化主导国家（例如，埃雷特纳）。原所有者奖励确保逻辑优先级：原所有者 > 邻国 > 文化主导。
- 对宿敌使用带严格 `limit` 过滤器的 `every_country`（如德里陷落）
- 使用带 `reason = intervene` 的 `join_war_as_attacker` 加入现有内战（如 `intervene_in_subject_civil_war`）
- 对不同层级使用交错的 `days = X` 延迟（如圭尔夫派和吉伯林派）
- 使用全局变量列表进行跟踪（如德里陷落）
- 使用冷却变量防止垃圾邮件（如 `press_claims`）
- 使用可见性触发器进行过滤（如所有局势先例）
- 将干预权存储为变量：`secessionist_intervention_right_$rebellion_id$`
- 在允许购买前检查资格：`OR = { (culture = rebel_culture AND top-ranked) OR is_rival_of = defender OR (is_leader_of_international_organization = { type = coalition target = defender }) }`。包围网领袖通过包围网成员中最高 `great_power_score` 自动选择（如 `coalition.txt` 中定义的，具有 `has_leader_country = yes` 和 `leader_type = country`，但没有明确的 `leader_change_method`，因此游戏默认选择最强成员）。
- **停战协议例外成本/惩罚机制：** 主要文化国家即使有停战协议也可以购买干预权，但以下机制之一防止利用：
- **选项 A（更高购买成本 - 推荐）：** 价格计算包括对有停战协议的主要文化国家的 2.0x 倍数（见价格计算部分）
- **选项 B（减少的惩罚）：** 加入战争时，有停战协议的主要文化国家收到减少的破坏停战协议惩罚（50% 严重程度：-25 稳定度，+0.5 厌战度，+12 敌意）而不是完整惩罚
- **选项 C（组合）：** 1.5x 购买成本倍数和加入战争时 50% 减少的惩罚
- **叛军部队机制：** 在爆发前，分离主义叛军是通过 `create_rebel` 创建的叛军实体（不是国家）。当叛军通过 `start_civil_war` 爆发时，它们形成一个叛军国家（具有 `country_type = location` 的普通国家）。叛军国家最初通过 `raise_levies` 获得部队（来自控制省份的征召兵，基于人口和控制 - 不是随机的）。正规军可以使用普通国家招募系统招募，受内置限制约束（人力、招募能力、训练速度、经济约束）。
- **关键机制 - 叛军国家属国地位和战争领导权：** 当叛乱爆发时，系统检查是否有主要文化国家购买了干预权。**如果是：** 叛军国家自动加入该国作为分离主义属国（使用 `create_subject` 和 `type = secessionist`），该国成为进攻方战争领袖（不是叛军国家/附庸）。这防止附庸利用，其中附庸在宗主之前获得所有利益，确保投资干预权的国家控制和平谈判和战争策略。**历史示例：** 如果金帐汗国为蒙古叛军购买，或奥斯曼为土耳其叛军购买，叛军作为属国加入。**如果不是：** 叛军国家独立并作为进攻方战争领袖领导战争。购买干预权的宿敌仍可以加入，但叛军国家保持独立。**历史示例：** 希腊独立战争（1821年）- 拜占庭帝国于 1453 年被摧毁，因此不存在希腊主要文化国家。希腊叛军独立并在俄罗斯、不列颠和法兰西（奥斯曼帝国的宿敌）的支持下赢得战争。当希腊获胜时，它成为一个新的独立国家，不是任何国家的属国。详见建议文档中的案例研究 1D：希腊独立战争的详细分析。
- **秘密正规军训练（可选）：** 杜卡特可以资助在革命开始时生成的秘密训练的正规军，但这些在生成时受叛军国家的内置限制约束。无需手动百分比上限 - 游戏的现有国家系统提供自然平衡。
- **无地国家概念：** 虽然 EU5 有国家类型（location、pop、building、army、navy）且无地国家存在（pop 类型），但叛军在爆发前是叛军实体（不是国家）。当它们通过 `start_civil_war` 爆发时，它们自动成为普通国家。在爆发前将叛军视为无地国家会增加不必要的复杂性（更早创建国家、管理无领土的国家、类型转换），而没有明显的好处。当前系统已经完美利用现有游戏机制。
- **宗主特殊行动（受战国"下克上"机制启发，重用现有逻辑）：** 当分离主义叛乱局势在附庸领土上开始时，宗主收到特殊通知并可以采取局势行动：取消附庸地位（重用 `overlord_can_cancel = yes` 和 `cancel_subject`）、创建不忠属国 CB（重用 `cb_disloyal_subject`，需要 `subject_loyalty < 50`）、夺取领土（重用 `seize_location_from_subject`）、召唤到宫廷、拒绝加入战争。怀疑分数基于模式计算（未核心化领土、低稳定度、加速叛军进度）并减少附庸忠诚度，使 `cb_disloyal_subject` 自动可用。行动集成到分离主义局势框架中（如战国幕府行动），但利用现有游戏机制以保持一致性。
- **世界大战升级预防：** 优先级提供系统防止创建世界大战的不对称同盟调用。国家选择参与（可选），允许双方与盟友协调（如历史第一次世界大战/第二次世界大战），而不是强迫进攻方独自战斗而防御方协调。资格限制（通过优先级提供系统排名最高的相同文化国家或宿敌或包围网领袖）防止随机国家加入。包围网领袖通过包围网成员中最高 `great_power_score` 自动选择，确保最强包围网成员协调干预。如果排名最高的国家拒绝，不会发生升级 - 局部叛乱保持局部（历史准确，并非所有区域事件都会升级）。当升级发生时，只有直接涉及的国家（购买干预权的国家和防御方）可以调用盟友，防止级联升级，其中每个国家都调用所有盟友。这匹配历史世界大战，其中双方在其决策中有代理权，可以在承诺之前协调，但升级是受控的（并非每个国家都调用所有盟友）。**与现有保障机制的集成：** 购买干预权的国家还可以利用现有游戏机制，如 `guarantee` 关系（`called_in_defensively = giving`）或 `ask_join_war_for_favors` 国家交互，以引入其他国家。**与包围网机制的集成：** 包围网领袖（通过包围网成员中最高 `great_power_score` 自动选择）可以购买干预权并通过现有包围网机制调用包围网成员（`coalition.txt` 中的 `join_defensive_wars_always` 和 `join_offensive_wars_always`），但只有包围网领袖可以触发包围网参与，防止级联升级。详见建议文档中的案例研究 1C：波兰叛乱升级的详细场景，展示波兰-立陶宛如何购买干预权，法兰西（保障了波兰-立陶宛）如何通过保障机制加入，或奥地利（包围网领袖）如何购买并协调包围网成员，使用现有游戏机制创建如历史第一次世界大战/第二次世界大战的平衡升级。
