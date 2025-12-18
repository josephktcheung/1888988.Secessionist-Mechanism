## Secessionist Mechanism - Technical Implementation Details

This document contains technical implementation details for the Secessionist Intervention Right Purchase System, including code structure, triggers, and mechanics based on proven EU5 precedents.

### REFERENCE PRECEDENTS

- Fall of Delhi: `game/in_game/common/situations/fall_of_delhi.txt` (priority-based visibility)
- Italian Wars: `game/in_game/common/situations/italian_wars.txt` (`ordered_country` with custom scoring)
- Rise of the Ottomans: `game/in_game/common/generic_actions/rise_of_the_ottomans.txt` (`press_claims` action)
- Intervene in Subject Civil War: `game/in_game/common/country_interactions/intervene_in_subject_civil_war.txt` (`join_war_as_attacker` with `reason = intervene`)
- Red Turban Rebellions: `game/in_game/common/generic_actions/red_turban_rebellions.txt` (`rtr_join_chinese_defense` uses `join_war_as_attacker`)
- create_instant_rebellion: `game/in_game/common/scripted_effects/global_effects.txt` (shows `raise_levies` mechanism for rebel troops)
- start_civil_war: Creates rebel country (normal country with `country_type = location`) subject to built-in game limits
- **Note:** Rebels are rebel entities (not countries) before breaking out. When `start_civil_war` is called, they become normal countries automatically - no need to treat them as landless countries
- Red Turban Rebellions `rtr_join_chinese_defense`: Uses `can_join_offensive_war_with` and `can_join_defensive_war_with` for eligibility checks, no forced participation, free to join (no ducat cost)
- Guarantee mechanism: `game/in_game/common/scripted_relations/guarantee.txt` (`called_in_defensively = giving`) - allows guarantor to be called in defensively when guaranteed country is attacked
- Ask Join War for Favors: `game/in_game/common/country_interactions/ask_join_war_for_favors.txt` - existing country interaction allowing countries to ask others to join wars
- $game_concept_coalition$ mechanics: `game/in_game/common/international_organizations/coalition.txt` - $game_concept_coalition$ have `has_leader_country = yes` and `leader_type = country`. $game_concept_coalition$ leaders are automatically selected by highest `great_power_score` among $game_concept_coalition$ members (no explicit `leader_change_method` defined, so game defaults to selecting strongest member). $game_concept_coalition$ members can join wars through `join_defensive_wars_always` and `join_offensive_wars_always` when $game_concept_coalition$ target is involved. This allows $game_concept_coalition$ leaders to coordinate $game_concept_coalition$ intervention while preventing cascading escalation (only $game_concept_coalition$ leader can trigger $game_concept_coalition$ participation).

### INTERIM SOLUTION: USING EXISTING JOIN WAR MECHANICS

**INTERIM APPROACH:** This solution can be implemented quickly before the full Intervention Right Purchase System, using existing `can_join_offensive_war_with` and `can_join_defensive_war_with` triggers (similar to Red Turban Rebellions). This provides immediate fixes while the comprehensive system is developed.

File: `game/in_game/events/secessionist_rebellion_interim.txt` (or add to existing event file)


```

# Event triggered when Secessionist civil war starts
namespace = secessionist_rebellion_interim

secessionist_rebellion_interim.1 = {
    type = country_event
    title = secessionist_rebellion_interim.1.t
    desc = secessionist_rebellion_interim.1.d
    
    trigger = {
        # Check if this country is eligible to join
        OR = {
            # Original owner with same culture (TIER 1 - highest priority)
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
            # Neighbor with same culture (TIER 2)
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
            # Culturally dominant (TIER 3 - only if not original owner or neighbor)
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
            # Rival of defender (TIER 4)
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
        # Standard eligibility checks (like Red Turban Rebellions)
        NOT = { is_at_war_with = scope:defender }
        NOT = { has_truce_with = scope:defender }
        NOT = { in_civil_war = yes }
        NOT = { is_during_bankruptcy = yes }
        distance_to_capital <= 5000
    }
    
    immediate = {
        # Store war and rebel country for reference
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
        name = secessionist_rebellion_interim.1.a  # "Join War"
        
        # Join the civil war on attacker side (like intervene_in_subject_civil_war)
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
        name = secessionist_rebellion_interim.1.b  # "Refuse"
        
        # Do nothing - country refuses to join, no penalties
    }
}

# Special event for overlords
secessionist_rebellion_interim.overlord = {
    type = country_event
    title = secessionist_rebellion_interim.overlord.t
    desc = secessionist_rebellion_interim.overlord.d
    
    trigger = {
        # Overlord of defender
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
        # Calculate suspicion score (same as full system)
        scope:defender = {
            set_variable = {
                name = vassal_suspicion_score
                value = 0
            }
            # ... (same suspicion score calculation as full system) ...
        }
        save_scope_as = secessionist_civil_war
    }
    
    option = {
        name = secessionist_rebellion_interim.overlord.a  # "Join Defensive War"
        
        # Join defensive war
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
        name = secessionist_rebellion_interim.overlord.b  # "Refuse - Let $vassal$ Fight Alone"
        
        # Do nothing - overlord refuses, vassal fights alone
    }
    
    option = {
        name = secessionist_rebellion_interim.overlord.c  # "Cancel $vassal$ Status"
        
        trigger = {
            scope:defender = {
                var:vassal_suspicion_score >= 50
            }
            # Check if overlord can cancel (reuses existing vassal.txt logic)
            scope:defender = {
                subject_type_is_not_locked = no
            }
        }
        
        # Cancel vassal status instantly (reuses existing cancel_subject mechanic)
        scope:actor = {
            cancel_subject = {
                target = scope:defender
                type = vassal
            }
        }
    }
}

```


File: `game/in_game/common/on_actions/secessionist_rebellion_interim.txt` (or add to existing on_action file)


```

# Trigger when Secessionist civil war starts
on_civil_war_start = {
    # Check if this is a Secessionist rebellion
    if = {
        limit = {
            any_attacker = {
                is_rebel_country = yes
                rebel_type = secessionist
            }
        }
        # Notify eligible countries in priority order
        
        # TIER 1: Original owners with same culture (highest priority)
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
        
        # TIER 2: Neighbors with same culture (if not original owner)
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
        
        # TIER 3: Culturally dominant (if not original owner or neighbor)
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
        
        # TIER 4: Rivals of defender
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
        
        # Overlords (special notification)
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


**Key Implementation Points:**
- Uses existing `can_join_offensive_war_with` and `can_join_defensive_war_with` triggers (already balanced and tested)
- Prioritizes original owners using `previous_owner` check (fixes $BYZ$ case)
- Optional participation - countries can refuse without penalties
- No resource cost - simpler than full system
- Overlords can refuse or cancel $vassal$ status (addresses Exploit 1)
- Eligibility checks naturally prevent world wars (distance, diplomatic relations, etc.)

### SITUATION FILE STRUCTURE

File: `game/in_game/common/situations/secessionist_rebellion.txt`


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
            # CRITICAL RESTRICTION: Top-ranked same-culture OR rival of defender OR coalition leader (prevents world wars)
            AND = {
                # Top-ranked same-culture country (only ONE)
                culture = rebel_culture
                # This will be filtered to top-ranked only in on_start
                game_setting:secessionist_notification_frequency != "disabled"
                game_setting:secessionist_notification_frequency != "rivals_only"
            }
            AND = {
                # Rivals of defender (strategic interest)
                is_rival_of = defender
                NOT = { culture = rebel_culture }
                game_setting:secessionist_notification_frequency != "disabled"
                game_setting:secessionist_notification_frequency != "same_culture_only"
            }
            AND = {
                # Coalition leader against defender (strategic interest through coalition)
                is_leader_of_international_organization = {
                    type = coalition
                    target = defender
                }
                NOT = { culture = rebel_culture }
                game_setting:secessionist_notification_frequency != "disabled"
                game_setting:secessionist_notification_frequency != "same_culture_only"
            }
            # Original owners (always eligible if same culture OR rival OR coalition leader)
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
            # Overlords of defender (always visible for special actions)
            AND = {
                is_overlord_of = defender
                defender = {
                    is_subject_of = this
                }
            }
        }
    }
    
    on_start = {
        # TIER 1: Top-ranked same-culture country (PRIORITY OFFER SYSTEM - only ONE can purchase at a time)
        # Step 1: Rank all same-culture countries and store in global list
        ordered_country = {
            limit = {
                culture = rebel_culture
                # CRITICAL: SUBJECT DISQUALIFICATION - Exclude subjects from ranking (following Italian Wars precedent)
                # This prevents small secessionist subjects in distant regions from being incorrectly prioritized for rebellions far away
                # It also prevents subjects from acquiring Intervention Rights that would cause them to join wars against their overlords
                # Only independent countries with the same primary culture are eligible for Category 1 ranking
                is_subject = no
                # Strict filtering checks
                # NOTE: Primary culture countries can purchase even with truce (truce exception for supporting own culture's independence)
                NOT = { is_at_war_with = defender }
                # TRUCE EXCEPTION: Primary culture countries can have truce with defender (they can choose to break truce to support their culture)
                NOT = { in_civil_war = yes }
                NOT = { is_during_bankruptcy = yes }
                diplomatic_capacity > 0
                distance_to_capital <= 5000
                can_join_as_attacker = yes
                gold >= (monthly_income_trade_and_tax * 2)
            }
            order_by = {
                # Ranking formula: (original_owner_bonus * 100) + (neighbor_bonus * 50) + (cultural_influence * 10) + (country_rank_level * 5) - (distance_to_capital / 10)
                # CRITICAL: Original owner bonus (100 points) is highest priority, fixing current system flaw
                # Example: $TUR$ (original owner) gets 100 points, $ERE$ (culturally dominant but not original owner) gets 0
                # This ensures $TUR$ ranks higher than $ERE$, fixing $BYZ$ case where $TUR$ are completely ignored
                value = {
                    add = {
                        if = {
                            limit = {
                                any_rebelling_province = { previous_owner = this }
                            }
                            value = 100  # Original owner bonus - FIXES CURRENT SYSTEM FLAW
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
            # Store all ranked same-culture countries in global list for priority offer system
            add_to_global_variable_list = {
                name = secessionist_same_culture_queue_$rebellion_id$
                target = this
            }
        }
        
        # Step 2: Extend purchase offer only to the top-ranked country (priority offer system)
        ordered_country = {
            limit = {
                culture = rebel_culture
                # CRITICAL: SUBJECT DISQUALIFICATION - Exclude subjects from ranking (following Italian Wars precedent)
                is_subject = no
                # Must be in the priority offer list and not already offered
                in_global_variable_list = secessionist_same_culture_queue_$rebellion_id$
                NOT = { has_variable = secessionist_offered_purchase_$rebellion_id$ }
                # Standard eligibility checks
                # NOTE: Primary culture countries can purchase even with truce (truce exception for supporting own culture's independence)
                NOT = { is_at_war_with = defender }
                # TRUCE EXCEPTION: Primary culture countries can have truce with defender (they can choose to break truce to support their culture)
                NOT = { in_civil_war = yes }
                NOT = { is_during_bankruptcy = yes }
                diplomatic_capacity > 0
                distance_to_capital <= 5000
                can_join_as_attacker = yes
                gold >= (monthly_income_trade_and_tax * 2)
            }
            order_by = {
                # Same ranking formula as above
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
            # Mark as offered and notify
            set_variable = {
                name = secessionist_offered_purchase_$rebellion_id$
                value = 1
            }
            # Set expiration timer (30 days to respond)
            set_variable = {
                name = secessionist_purchase_offer_expires_$rebellion_id$
                days = 30
            }
            # Immediate notification (no delay)
            trigger_event_non_silently = secessionist_rebellion.1
            set_variable = {
                name = secessionist_notification_cooldown_$rebellion_id$
                months = 6
            }
        }
        
        # TIER 2: Rivals of defender (staggered 5-15 days)
        every_country = {
            limit = {
                is_rival_of = defender
                NOT = { culture = rebel_culture }
                # Not coalition leader (coalition leaders handled separately in TIER 2B)
                NOT = {
                    is_leader_of_international_organization = {
                        type = coalition
                        target = defender
                    }
                }
                # Strict filtering checks
                NOT = { is_at_war_with = defender }
                NOT = { has_truce_with = defender }
                NOT = { in_civil_war = yes }
                NOT = { is_during_bankruptcy = yes }
                diplomatic_capacity > 0
                NOT = { has_variable = secessionist_notification_cooldown_$rebellion_id$ }
                distance_to_capital <= 5000
                can_join_as_attacker = yes
                gold >= (monthly_income_trade_and_tax * 2)
                # Evaluation score check (for rivals, based on strategic interest)
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
        
        # TIER 2B: Coalition leaders against defender (staggered 10-20 days, after regular rivals)
        # Note: Coalition leaders are automatically selected by highest great_power_score among coalition members
        # (coalition.txt has has_leader_country = yes and leader_type = country, but no explicit leader_change_method,
        # so the game defaults to selecting the strongest member by great_power_score)
        every_country = {
            limit = {
                # Coalition leader against defender (automatically selected by highest great_power_score among members)
                is_leader_of_international_organization = {
                    type = coalition
                    target = defender
                }
                NOT = { culture = rebel_culture }
                # Strict filtering checks
                NOT = { is_at_war_with = defender }
                NOT = { has_truce_with = defender }
                NOT = { in_civil_war = yes }
                NOT = { is_during_bankruptcy = yes }
                diplomatic_capacity > 0
                NOT = { has_variable = secessionist_notification_cooldown_$rebellion_id$ }
                distance_to_capital <= 5000
                can_join_as_attacker = yes
                gold >= (monthly_income_trade_and_tax * 2)
                # Evaluation score check (for coalition leaders, based on strategic interest)
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
        
        # TIER 3: Overlords of defender (immediate notification for special actions)
        every_country = {
            limit = {
                is_overlord_of = defender
                defender = {
                    is_subject_of = this
                }
            }
            # Calculate suspicion score for vassal
            defender = {
                set_variable = {
                    name = vassal_suspicion_score
                    value = 0
                }
                # Pattern 1: Uncored territories despite capacity
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
                # Pattern 2: Low stability despite resources
                add = {
                    if = {
                        limit = {
                            stability < 30
                            gold >= (monthly_income_trade_and_tax * 6)
                        }
                        value = 15
                    }
                }
                # Pattern 3: Accelerating rebel progress
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
                # Pattern 4: Not using rebel crackdown despite ability
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
                # Pattern 5: High unrest without counter-measures
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
            # Immediate notification for overlord (with suspicion score)
            trigger_event_non_silently = secessionist_rebellion.overlord_notification
            set_variable = {
                name = secessionist_notification_cooldown_$rebellion_id$
                months = 6
            }
        }
    }
    
    # Monthly check: Advance priority offer if current country refused or time expired
    monthly = {
        # Check if any country that received offer has refused or offer expired
        if = {
            limit = {
                # Check if there's a country that was offered but hasn't purchased
                any_country = {
                    in_global_variable_list = secessionist_same_culture_queue_$rebellion_id$
                    has_variable = secessionist_offered_purchase_$rebellion_id$
                    NOT = { has_variable = secessionist_intervention_right_$rebellion_id$ }  # Hasn't purchased
                    OR = {
                        has_variable = secessionist_refused_purchase_$rebellion_id$  # Refused
                        var:secessionist_purchase_offer_expires_$rebellion_id$ <= 0  # Time expired
                    }
                }
            }
            # Extend offer to next-ranked country (priority offer system)
            ordered_country = {
                limit = {
                    culture = rebel_culture
                    # CRITICAL: SUBJECT DISQUALIFICATION - Exclude subjects from ranking (following Italian Wars precedent)
                    is_subject = no
                    in_global_variable_list = secessionist_same_culture_queue_$rebellion_id$
                    NOT = { has_variable = secessionist_offered_purchase_$rebellion_id$ }  # Not yet offered
                    NOT = { has_variable = secessionist_intervention_right_$rebellion_id$ }  # Hasn't purchased
                    NOT = { has_variable = secessionist_refused_purchase_$rebellion_id$ }  # Hasn't refused
                    # Standard eligibility checks
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
                    # Same ranking formula
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
                # Mark as offered and extend purchase offer to next country
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


**PRIORITY OFFER SYSTEM EXPLANATION:**
- When situation starts, all eligible same-culture countries are ranked and stored in `secessionist_same_culture_queue_$rebellion_id$` global list
- Only top-ranked country receives purchase offer initially (marked with `secessionist_offered_purchase_$rebellion_id$` variable) - this implements "right of first refusal"
- If top-ranked country refuses (sets `secessionist_refused_purchase_$rebellion_id$`) or 30 days expire (`secessionist_purchase_offer_expires_$rebellion_id$ <= 0`), monthly check extends offer to next-ranked country
- Next-ranked country receives purchase offer and can purchase
- Sequential offers continue until someone purchases (sets `secessionist_intervention_right_$rebellion_id$`) or all eligible countries refuse
- This ensures only ONE same-culture country can purchase at a time, preventing multiple countries from all joining simultaneously
- Purchase action's `allow` block checks `has_variable = secessionist_offered_purchase_$rebellion_id$` to ensure only the country that received the current offer can purchase

**REFUSAL HANDLING:**
- Countries can explicitly refuse by choosing "Refuse" option in notification event (sets `secessionist_refused_purchase_$rebellion_id$` variable)
- If country doesn't purchase within 30 days, offer expires (`secessionist_purchase_offer_expires_$rebellion_id$ <= 0`) and they're considered to have refused
- Monthly check detects refused/expired offers and extends offer to next-ranked country (priority offer system advances)
- Once a country purchases, priority offer system stops (no further offers to other same-culture countries)

### ACTION FILE STRUCTURE

File: `game/in_game/common/generic_actions/secessionist_rebellion.txt`


```

purchase_secessionist_intervention_right = {
    type = situation
    show_message = no
    
    potential = {
        # CRITICAL: Only eligible if top-ranked same-culture OR rival of defender OR coalition leader
        OR = {
            AND = {
                culture = rebel_culture
                # Check if this country is the top-ranked same-culture country
                # (This check is done in situation visibility, but we verify here too)
            }
            is_rival_of = defender
            AND = {
                # Coalition leader against defender (coalition leaders are automatically selected by highest great_power_score among members)
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
            # TRUCE EXCEPTION: Primary culture countries can purchase even with truce (truce exception for supporting own culture's independence)
            # Rivals and coalition leaders must NOT have truce, but primary culture countries can have truce
            OR = {
                # Primary culture country - can have truce
                culture = rebel_culture
                # Rival or coalition leader - must NOT have truce
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
            # PRIORITY OFFER SYSTEM: Only allow purchase if this country has received the current offer (right of first refusal)
            has_variable = secessionist_offered_purchase_$rebellion_id$
            NOT = { has_variable = secessionist_refused_purchase_$rebellion_id$ }  # Hasn't explicitly refused
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
        # Grant Intervention Right (stored as variable)
        scope:actor = {
            set_variable = {
                name = secessionist_intervention_right_$rebellion_id$
                value = 1
            }
            # Remove offered flag (purchase successful, priority offer system stops here)
            remove_variable = secessionist_offered_purchase_$rebellion_id$
        }
        # Allocate ducats to rebels for mercenary hiring (and potentially secret regular training)
        defender = {
            every_secessionist_rebel = {
                set_variable = {
                    name = rebel_ducats
                    add = price:secessionist_intervention_right_price
                }
                # Optional: Secret regular training (subject to built-in game limits)
                # This would be implemented separately, respecting rebel country's limits when it forms
            }
        }
    }
    
    ai_will_do = {
        # Base evaluation score calculation
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

# Action to join the civil war when rebellion breaks out
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
        # TRUCE-BREAKING PENALTY REDUCTION (Option B or C):
        # If primary culture country has truce with defender, apply reduced penalties when joining war
        # Option B: Reduced penalties (50% severity)
        # Option C: Combined with higher purchase cost (already applied in price calculation)
        if = {
            limit = {
                scope:actor = {
                    culture = rebel_culture
                    has_truce_with = defender
                }
            }
            # Apply reduced truce-breaking penalties (Option B or C)
            scope:actor = {
                # Reduced stability penalty: -25 instead of -50 (50% severity)
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
        # Remove Intervention Right variable (used)
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


### PRICE CALCULATION

File: `game/in_game/common/prices/secessionist_rebellion.txt`


```

secessionist_intervention_right_price = {
    base = {
        value = 100
    }
    multiply = {
        # Scale with economic base (like scaled_gold)
        value = scope:actor.monthly_income_trade_and_tax
        multiply = 2
    }
    add = {
        # Distance penalty
        value = scope:actor.distance_to_capital(defender.capital)
        divide = 100
    }
    add = {
        # Cultural tradition penalty (reflects spy network building difficulty)
        value = defender.cultural_tradition
        divide = scope:actor.cultural_influence
        multiply = 50
    }
    add = {
        # Counterespionage penalty
        value = defender.counterespionage_modifier
        multiply = 100
    }
    multiply = {
        # Rival multiplier (1.5x for rivals, as they lack cultural ties)
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


### GAME SETTING

File: `game/main_menu/common/game_rules/00_game_rules.txt`


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


### EVALUATION SCORE CALCULATION

Based on Italian Wars priority calculation pattern:


```

evaluation_score = {
    value = 0
    # Base factors
    add = {
        value = land_desire(defender)
        multiply = 100
    }
    add = {
        value = conquer_desire(defender)
        multiply = 50
    }
    # Distance penalty
    subtract = {
        value = distance_to_capital(defender.capital)
        divide = 10
    }
    # Priority bonuses
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
    # Power level bonus
    add = {
        value = country_rank_level
        multiply = 10
    }
    # Strategic interest (for rivals)
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


### RANKING FORMULA FOR TOP-RANKED SAME-CULTURE COUNTRY

Used in `ordered_country` to select the single top-ranked same-culture country. **CRITICAL:** This formula prioritizes original ownership (100 points) over cultural dominance, fixing the current system's flaw where original owners (e.g., Ottomans in Byzantine case) are completely ignored in favor of culturally dominant countries (e.g., Eretnids).


```

ranking_score = {
    value = 0
    # Original owner bonus (highest priority) - FIXES CURRENT SYSTEM FLAW
    # Example: Ottomans (original owner) gets 100 points, Eretnids (not original owner) gets 0
    # This ensures Ottomans ranks higher than Eretnids, fixing the Byzantine case where Ottomans are ignored
    add = {
        if = {
            limit = {
                any_rebelling_province = { previous_owner = this }
            }
            value = 100
        }
    }
    # Neighbor bonus (high priority)
    add = {
        if = {
            limit = { is_neighbor_of = defender }
            value = 50
        }
    }
    # Cultural influence (medium priority)
    add = {
        value = cultural_influence
        multiply = 10
    }
    # Country rank level (low priority)
    add = {
        value = country_rank_level
        multiply = 5
    }
    # Distance penalty (reduces score for far-away countries)
    subtract = {
        value = distance_to_capital(defender.capital)
        divide = 10
    }
}

```


### MERCENARY HIRING MECHANISM

When Intervention Right is purchased, ducats are allocated to rebels:


```

on_intervention_right_purchased = {
    defender = {
        every_secessionist_rebel = {
            # Allocate ducats
            set_variable = {
                name = rebel_ducats
                add = price:secessionist_intervention_right_price
            }
            # Hire mercenaries (if enough ducats)
            if = {
                limit = {
                    var:rebel_ducats >= mercenary_hiring_cost
                }
                hire_mercenaries = {
                    amount = var:rebel_ducats
                    gathering_time = MERCENARY_GATHERING_TIME_BASE  # 14 days
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


### SECRET REGULAR TRAINING MECHANISM (OPTIONAL ENHANCEMENT)

**BALANCE SOLUTION - BUILT-IN GAME MECHANICS:** When Secessionist rebels break out via `start_civil_war`, they form a rebel country (normal country with `country_type = location`). This rebel country is automatically subject to built-in game limits. **Note:** Rebels are rebel entities (not countries) before breaking out, so they cannot be treated as landless countries. When they break out, they become normal countries automatically - no country type conversion needed.


```

# When Intervention Right is purchased, ducats can fund secret regular training
on_intervention_right_purchased_secret_training = {
    defender = {
        every_secessionist_rebel = {
            # Allocate ducats for secret regular training
            set_variable = {
                name = rebel_secret_training_ducats
                add = price:secessionist_intervention_right_price
            }
            # Secret training happens before rebellion breaks out (not visible on map)
            # Training is limited by rebel country's built-in limits when it forms
        }
    }
}

# When rebellion breaks out, secretly trained regulars spawn (subject to built-in limits)
on_rebellion_outbreak_secret_training = {
    defender = {
        every_secessionist_rebel = {
            # Create rebel country via start_civil_war (creates normal country with country_type = location)
            # Note: Rebels are rebel entities (not countries) before breaking out. start_civil_war creates a normal country automatically
            start_civil_war = {
                save_scope_as = rebel_country
            }
            # Rebel country is subject to built-in game limits:
            # 1. Manpower pool: Limited by population and buildings in rebel-controlled locations
            #    (barracks, regimental_camp provide local_manpower)
            # 2. Recruitment capacity: Limited by locations with can_recruit_regiment_in_this_location
            #    (requires military buildings like barracks, regimental_camp, sergeantry, warrior_temple)
            # 3. Training speed: Limited by max_regiments_trained_at_same_time per location
            #    (typically 1 regiment per location with recruitment building)
            # 4. Economic constraints: Regulars require maintenance costs scaling with rebel country's income
            
            # Spawn secretly trained regulars (subject to rebel country's built-in limits)
            if = {
                limit = {
                    has_variable = rebel_secret_training_ducats
                    var:rebel_secret_training_ducats > 0
                }
                # Calculate how many regulars can be spawned based on rebel country's limits
                # This ensures natural balance - no manual percentage caps needed
                spawn_secret_regulars = {
                    ducats = var:rebel_secret_training_ducats
                    # Limits applied automatically:
                    # - Cannot exceed rebel country's manpower pool
                    # - Cannot exceed recruitment capacity (locations with recruitment buildings)
                    # - Cannot exceed training speed (regiments per location)
                    # - Must be affordable (maintenance costs scale with rebel country's income)
                }
            }
        }
    }
}

```


**Key Implementation Points:**
- Secret regular training respects rebel country's built-in limits (manpower, recruitment capacity, training speed, economic constraints)
- No manual percentage caps needed - game's existing country system provides natural balance
- Rebel country's military capacity proportional to population, economic base, and recruitment buildings
- This leverages existing game systems for natural balance without requiring complex manual restrictions

### JOINING THE CIVIL WAR

When a country with purchased Intervention Right wants to join the civil war:


```

# This uses the existing intervene_in_subject_civil_war pattern
# Countries use join_secessionist_civil_war action, which calls:
join_war_as_attacker = {
    war = scope:target  # The civil war
    reason = intervene
}

# TRUCE-BREAKING PENALTY REDUCTION (Option B or C):
# If primary culture country has truce with defender, apply reduced penalties when joining war
# Option B: Reduced penalties (50% severity)
# Option C: Combined with higher purchase cost (already applied in price calculation)
if = {
    limit = {
        scope:actor = {
            culture = rebel_culture
            has_truce_with = defender
            has_variable = secessionist_intervention_right_$rebellion_id$
        }
    }
    # Apply reduced truce-breaking penalties (Option B or C)
    scope:actor = {
        # Reduced stability penalty: -25 instead of -50 (50% severity)
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

### OVERLORD SPECIAL ACTIONS (INSPIRED BY SENGOKU "下克上" MECHANICS)

When Secessionist rebellion situation begins in a vassal's territory, overlord receives special notification and can take situation actions (similar to Sengoku Shogunate actions). These actions are integrated into the Secessionist situation framework.

File: `game/in_game/common/generic_actions/secessionist_rebellion.txt` (add to existing file)


```

# Overlord Actions (inspired by Sengoku mechanics, reusing existing logic)
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
        # Suspicion score check (high suspicion indicates intentional sabotage)
        situation:secessionist_rebellion = {
            var:vassal_suspicion_score >= 50
        }
        # Check if overlord can cancel (reuses existing vassal.txt logic: overlord_can_cancel = yes)
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
        # Overlord cancels vassal status instantly (reuses existing cancel_subject mechanic)
        # Note: Uses existing cancel_subject effect (same as diplomatic action CANCEL_vassal)
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


### SUSPICION SCORE CALCULATION

Suspicion score is calculated when Secessionist rebellion situation begins, stored as situation variable:


```

on_secessionist_situation_start = {
    if = {
        limit = {
            defender = {
                is_subject = yes
            }
        }
        # Calculate suspicion score for vassal
        set_variable = {
            name = vassal_suspicion_score
            value = 0
        }
        # Pattern 1: Uncored territories despite capacity
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
        # Pattern 2: Low stability despite resources
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
        # Pattern 3: Accelerating rebel progress
        add = {
            if = {
                limit = {
                    defender = {
                        any_secessionist_rebel = {
                            monthly_progress > 0.05  # Faster than normal
                        }
                    }
                }
                value = 25
            }
        }
        # Pattern 4: Not using rebel crackdown despite ability
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
        # Pattern 5: High unrest without counter-measures
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
        # After calculating suspicion score, reduce vassal loyalty if suspicion is high
        # This makes cb_disloyal_subject available automatically when loyalty drops below 50
        # Note: cb_disloyal_subject automatically becomes available when subject_loyalty < 50 (no manual creation needed)
        # However, creating the CB still takes time/1 diplomat, but the CB becomes available automatically when threshold is met
        if = {
            limit = {
                var:vassal_suspicion_score >= 30
            }
            defender = {
                add_subject_loyalty = {
                    value = -20
                    # Additional penalty for high suspicion
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


**Key Implementation Points:**
- Suspicion score reduces vassal loyalty (high suspicion = -20 to -30 loyalty), which can trigger existing `cb_disloyal_subject` availability automatically (no manual CB creation needed if loyalty drops below 50)
- Overlord actions reuse existing game mechanics:
- **Cancel Vassal Status:** Uses existing `overlord_can_cancel = yes` from `vassal.txt` and `cancel_subject` effect
- **Create Disloyal Subject CB:** Uses existing `cb_disloyal_subject` (requires `subject_loyalty < 50`). Suspicion score reduces loyalty, making CB available automatically.
- **Seize Territories:** Uses existing `seize_location_from_subject` country interaction with same loyalty/liberty desire checks

### FILE STRUCTURE SUMMARY

**Required files:**
1. `game/in_game/common/situations/secessionist_rebellion.txt` - Situation definition
2. `game/in_game/common/generic_actions/secessionist_rebellion.txt` - Intervention Right purchase action and join war action
3. `game/in_game/common/prices/secessionist_rebellion.txt` - Price calculation
4. `game/main_menu/common/game_rules/00_game_rules.txt` - Game setting (add to existing file)
5. `game/in_game/events/situations/secessionist_rebellion.txt` - Event files for notifications
6. Localization files for all new strings

**Key implementation points:**
- Use `ordered_country` with ranking formula, `max = 1` to select top-ranked same-culture country (like Italian Wars)
- **CRITICAL: SUBJECT DISQUALIFICATION** - Use `is_subject = no` in `ordered_country` limit clauses to exclude subjects from Category 1 ranking (following Italian Wars precedent). This prevents small secessionist subjects in distant regions from being incorrectly prioritized for rebellions far away (e.g., a Mongolian culture secessionist subject in Mongolia region being prioritized for a rebellion in Georgia). It also prevents subjects from acquiring Intervention Rights that would cause them to join wars against their overlords, which would break their subject relationships (e.g., France-Sicily Personal Union breaking when Sicilian rebels appear in French-controlled territory). Only independent countries with the same primary culture are eligible for Category 1 ranking.
- **CRITICAL:** Ranking formula prioritizes original ownership (100 points) over cultural dominance, fixing current system flaw where original owners (e.g., Ottomans in Byzantine case) are completely ignored in favor of culturally dominant countries (e.g., Eretnids). Original owner bonus ensures logical priority: Original Owner > Neighbors > Culturally Dominant.
- Use `every_country` with strict `limit` filters for rivals (like Fall of Delhi)
- Use `join_war_as_attacker` with `reason = intervene` to join existing civil wars (like `intervene_in_subject_civil_war`)
- Use staggered `days = X` delays for different tiers (like Guelphs and Ghibellines)
- Use global variable lists for tracking (like Fall of Delhi)
- Use cooldown variables to prevent spam (like `press_claims`)
- Use visibility triggers for filtering (like all situation precedents)
- Store Intervention Right as variable: `secessionist_intervention_right_$rebellion_id$`
- Check eligibility before allowing purchase: `OR = { (culture = rebel_culture AND top-ranked) OR is_rival_of = defender OR (is_leader_of_international_organization = { type = coalition target = defender }) }`. Coalition leaders are automatically selected by highest `great_power_score` among coalition members (as defined in `coalition.txt` with `has_leader_country = yes` and `leader_type = country`, but no explicit `leader_change_method`, so the game defaults to selecting the strongest member).
- **Truce Exception Cost/Penalty Mechanisms:** Primary culture countries can purchase Intervention Right even with a truce, but one of the following mechanisms prevents gaming:
- **Option A (Higher Purchase Cost - Recommended):** Price calculation includes 2.0x multiplier for primary culture countries with truce (see price calculation section)
- **Option B (Reduced Penalties):** When joining the war, primary culture countries with truce receive reduced truce-breaking penalties (50% severity: -25 stability, +0.5 war exhaustion, +12 antagonism) instead of full penalties
- **Option C (Combined):** Both 1.5x purchase cost multiplier AND 50% reduced penalties when joining the war
- **Rebel troops mechanism:** Before breaking out, Secessionist rebels are rebel entities (not countries) created via `create_rebel`. When rebels break out via `start_civil_war`, they form a rebel country (normal country with `country_type = location`). Rebel country initially gets troops through `raise_levies` (levies from controlled provinces, based on population and control - NOT random). Regulars can be recruited using normal country recruitment system, subject to built-in limits (manpower, recruitment capacity, training speed, economic constraints).
- **CRITICAL MECHANIC - REBEL COUNTRY SUBJECT STATUS AND WAR LEADERSHIP:** When rebellion breaks out, the system checks if a primary culture country purchased Intervention Right. **If yes:** The rebel country automatically joins that country as a Secessionist subject (using `create_subject` with `type = secessionist`), and that country becomes the attacker war leader (not the rebel country/vassal). This prevents vassal gaming where vassals take all benefits before their overlord, ensuring the country that invested in the Intervention Right controls peace negotiations and war strategy. **Historical Examples:** If Golden Horde purchases for Mongolian rebels, or Ottomans purchases for Turkish rebels, the rebels join as subjects. **If no:** The rebel country becomes independent and leads the war as attacker war leader. Rivals who purchased Intervention Right can still join, but the rebel country remains independent. **Historical Example:** Greek War of Independence (1821) - Byzantine Empire was destroyed in 1453, so no Greek primary culture country existed. Greek rebels became independent and won the war with support from Russia, Britain, and France (rivals of Ottoman Empire). When Greece won, it became a new independent nation, not a subject of any country. See CASE STUDY 1D: Greek War of Independence in suggestion document for detailed analysis.
- **Secret regular training (optional):** Ducats can fund secretly trained regulars that spawn when revolution begins, but these are subject to rebel country's built-in limits when they spawn. No manual percentage caps needed - game's existing country system provides natural balance.
- **Landless country concept:** While EU5 has country types (location, pop, building, army, navy) and landless countries exist (pop type), rebels are rebel entities (not countries) before breaking out. When they break out via `start_civil_war`, they become normal countries automatically. Treating rebels as landless countries before breaking out would add unnecessary complexity (creating countries earlier, managing countries without territory, type conversion) without clear benefit. The current system already leverages existing game mechanics perfectly.
- **Overlord special actions (inspired by Sengoku "下克上" mechanics, reusing existing logic):** When Secessionist rebellion situation begins in vassal's territory, overlord receives special notification and can take situation actions: cancel vassal status (reuses `overlord_can_cancel = yes` and `cancel_subject`), create disloyal subject CB (reuses `cb_disloyal_subject` which requires `subject_loyalty < 50`), seize territories (reuses `seize_location_from_subject`), summon to court, refuse to join war. Suspicion score calculated based on patterns (uncored territories, low stability, accelerating rebel progress) and reduces vassal loyalty, making `cb_disloyal_subject` available automatically. Actions integrated into Secessionist situation framework (like Sengoku Shogunate actions) but leverage existing game mechanics for consistency.
- **World war escalation prevention:** Priority Offer System prevents asymmetric alliance calling that creates world wars. Countries choose to participate (optional), allowing both sides to coordinate with allies (like historical WW1/WW2) rather than forcing attackers to fight alone while defenders coordinate. Eligibility restrictions (top-ranked same-culture via Priority Offer System OR rivals OR coalition leader) prevent random countries from joining. Coalition leaders are automatically selected by highest `great_power_score` among coalition members, ensuring the strongest coalition member coordinates intervention. If top-ranked country refuses, no escalation occurs - local rebellion remains local (historically accurate, not all regional incidents escalate). When escalation occurs, only countries directly involved (who purchased Intervention Right and defender) can call allies, preventing cascading escalation where every country calls all their allies. This matches historical world wars where both sides had agency in their decisions and could coordinate before committing, but escalation was controlled (not every country called all their allies). **Integration with existing guarantee mechanics:** Countries who purchase Intervention Right can also leverage existing game mechanics like `guarantee` relations (`called_in_defensively = giving`) or `ask_join_war_for_favors` country interaction to bring in additional countries. **Integration with coalition mechanics:** Coalition leaders (automatically selected by highest `great_power_score` among coalition members) can purchase Intervention Right and call coalition members through existing coalition mechanics (`join_defensive_wars_always` and `join_offensive_wars_always` in `coalition.txt`), but only the coalition leader can trigger coalition participation, preventing cascading escalation. See CASE STUDY 1C: Polish Rebellion Escalation in suggestion document for detailed scenarios showing how Poland-Lithuania purchases Intervention Right, France (who has guaranteed Poland-Lithuania) joins through guarantee mechanism, or how Austria (coalition leader) purchases and coordinates coalition members, creating balanced escalation like historical WW1/WW2 using existing game mechanics.
