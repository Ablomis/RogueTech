<pre>
Unpack to Mods folder

Settings 
CustomAmmoCategoriesSettings.json

{
"debugLog":true, - enable debug log 
"modHTTPServer":false, - enable debug http server
"modHTTPListen":"http://localhost:65080/" - debug http server url, if enabled
"forbiddenRangeEnable:true, - enable or disable forbidden range mechanic, if false ForbiddenRange always counts as 0 
"ClusterAIMult":0.2, - AI use this value to calculate cluster weapon mode/ammo (more projectiles, less damage per projectile) prefer coefficient 
                            in case of low target armor and/or exposed locations
"PenetrateAIMult":0.4 - AI use this value to calculate penetrate weapon mode/ammo (less projectiles, more damage per projectile) prefer coefficient 
                            in case of full target armor and no exposed locations
"JamAIAvoid":1.0, - if higher than 1.0, AI will more avoid jamming modes/ammo. 0.0 will result division by zero exception. 
                      0 < X < 1.0f AI will prefer jamming modes/ammo (i don't know why it needs, but you can)
"DamageJamAIAvoid":2.0, - if higher than 1.0, AI will more avoid modes/ammo that damage weapon on jamming. 0.0 will result division by zero exception. 
                      0 < X < 1.0f AI will prefer damage jamming modes/ammo (i don't know why it needs, but you can)
					  NOTE: if AI has exposed locations, it will lose all fear and will not avoid dangerous modes and ammo.
}

CustomAmmoCategories.json
[
{
	"Id":"LGAUSS", - new ammo category name, precessed for WeaponDef.AmmoCategory and AmmunitionDef.Category fields, using it in other AmmoCategory field will lead load error
	"BaseCategory":"GAUSS" - base category name. Must bt in (AC2/AC5/AC10/AC20/GAUSS/Flamer/AMS/MG/SRM/LRM), 
	                         needed for backward compatibility. 
							 All other game mechanic (for example status effect targeting), except ammo count in battle and mech validator in mech lab will use this value.
							 !Flamer - is base category for energy ammo (plasma, chemical lasers etc)
},
]

Weapon definition
new fields
  "Streak": true/false - if true only success hits will be shown, ammo decremental and heat generation will be based on success hits. 
							with "HitGenerator" : "Streak" - will be true streak effect all-hit-or-no-fire
  "HitGenerator" : "Streak", Set to hit generator. Supported values ("Individual"/"Cluster"/"Streak"). 
                                  Streak hit generator is sort of cluster, 
								  if first projectile hit, rest hit too (location distribution as cluster hit generator),
								  if first projectile misses, rest misses too
								  if not set weapon hit generator will be used.
								  if not set hit generator will be choosed by weapon type.
								  if weapon define has tag "wr-clustered_shots", "Cluster" hit generator will be forced. 
  "DirectFireModifier" : -10.0, Accuracy modifier if weapon can strike directly
  "DamageOnJamming": true/false, - if true on jamm weapon will be damaged
  "FlatJammingChance": 1.0, - Chance of jamming weapon after fire. 1.0 is jamm always. Unjamming logic implemented as in WeaponRealizer
  "GunneryJammingBase": 5, - 
  "GunneryJammingMult": 0.05, - this values uses to alter flat jamming chance by pilot skills 
                                  formula effective jamming chance = FlatJammingChance + (GunneryJammingBase - Pilot Gunnery)* GunneryJammingMult
								  if FlatJammingChance = 1.0, GunneryJammingBase = 6, GunneryJammingMult = 0.1, GunnerySkill = 10
								  result = 1.0 + (6-10)*0.1 = 0.6
								  GunneryJammingBase if ommited in weapon def., ammo def. and mode def. assumed as 5. 
  "DisableClustering": true/false - if true ProjectilesPerShot > 1 will affect only visual nor damage. If omitted consider as true.
  "NotUseInMelee": true, - if true even AntiPersonel weapon type will not fire on melee attack, AI aware. 
  "AlternateDamageCalc": false, - if true alternate damage calc formula will be implemented 
                              DamagePerShot = (damage from weaponDef + (damage from ammo) + (damage from mode)*(damage multiplayer from ammo)*(damage multiplayer from mode)*(damage with effects)/(damage from weaponDef)
  "AlternateHeatDamageCalc": false, - same as  AlternateDamageCalc but for heat 
  "AMSHitChance": 0.0, - if this weapon is AMS, this value is AMS efficiency, 
                         if this weapon is missile launcher this value shows how difficult to intercept missile with AMS. Negative value - is harder, 
						 positive is easer.
  "IsAMS": false, - if true this weapon acts as AMS. It will not fire during normal attack. But tries to intercept incomming missiles.
                    rude model: every 10 meters of missile fly path there is check, if it in range of any AMS. 
					If so, AMS have AMS.AMSHitChance + missile.AMSHitChance chance to shoot missile down. Avaible shoots count of AMS is decrementing.
					If AMS have no shoots avaible it will not shoot. At end of turn AMS shoots count set to AMS.ShootsWhenFired.
					If missile intercepted, no further checks will be made. 
					Commentary: as you can see, if missile fly path is short chance to be shooted down is less. 
					If missiles is few, and fly path is long chance to be shooted down grows rapidly. 
					AMS visual effect commentary: only two visual effects avaible for AMS: ballistic(autocannons) and laser(lasers). 
					I had experimented with "WeaponEffect-Weapon_Laser_Small" and "WeaponEffect-Weapon_AC2" for LAMS ans AMS accordingly.
					AMS can become jammed, have damage-on-jam option and so on. AMSHitChance and ShootsWhenFired can be updated in AMS ammo or mode.
					AMS uses ammunition and heated while firing. Damage and on hit status effects will on be applied. 
  "IsAAMS": false - if true, weapon acts as advanced AMS, it will fire all missiles from enemies in range, not only attacking.
  "AMSImmune": false - if true, weapon missiles is immune to AMS and none AMS will try to intercept them.
	"Modes": array of modes for weapon
	[{
		"Id": "x4",  - Must be unique per weapon
		"UIName": "x4", - This string will be displayed near weapon name
		"isBaseMode":true, - Weapon must have one base mode. Mode with this setting will used by default
		"WeaponEffectID" : "WeaponEffect-Weapon_PPC", Played fire effect can be set in mode definition
		"EvasivePipsIgnored" : 0, This value will be added to EvasivePipsIgnored (current weapon status effects will be used too)

		"AccuracyModifier" : -10.0, This value will be added to AccuracyModifier (current weapon status effects will be used too)
		"CriticalChanceMultiplier" : 0.0, This value will be added to CriticalChanceMultiplier (current weapon status effects will be used too)
		"DamagePerShot": -50.0, This value will be added to DamagePerShot (current weapon status effects will be used too)
		"ShotsWhenFired" : 0, This value will be added to ShotsWhenFired (current weapon status effects will be used too)
		"ProjectilesPerShot" : 0, This value will be added to ProjectilesPerShot (current weapon status effects will be used too)
		"HeatDamagePerShot": 0.0, This value will be added to HeatDamagePerShot (current weapon status effects will be used too)
		   
		"MinRange": 0.0, This value will be added to MinRange (current weapon status effects will be used too)
		"MaxRange": 0.0, This value will be added to MaxRange (current weapon status effects will be used too)
		"ShortRange": 0.0, This value will be added to ShortRange (current weapon status effects will be used too)
		"MiddleRange": 0.0, This value will be added to MiddleRange (current weapon status effects will be used too)
		"LongRange": 0.0, This value will be added to LongRange (current weapon status effects will be used too)
			 NOTE: Range modifications not always displays correctly while viewing shooting arc, but hit chance and possibility calculated normally. 
		"ForbiddenRange": 90, - weapon can't fire at all if range to target is less than this value
		"HeatGenerated" : 0, This value will be added to HeatGenerated (current weapon status effects will be used too)
		"RefireModifier" : 0, This value will be added to RefireModifier (current weapon status effects will be used too)
		"Instability" : 0, This value will be added to Instability (current weapon status effects will be used too)
		"AttackRecoil" : 0, This value will be added to AttackRecoil
		"IndirectFireCapable" : false, Effective IndirectFireCapable will be taken from ammo. If not set in ammo define, weapon value will be used
		"EvasivePipsIgnored" : 0, This value will be added to EvasivePipsIgnored (current weapon status effects will be used too)
		"HitGenerator" : "Individual", Set to hit generator. Supported values ("Individual"/"Cluster"/"Streak"). 
									  Streak hit generator is sort of cluster, 
									  if first projectile hit, rest hit too (location distribution as cluster hit generator),
									  if first projectile misses, rest misses too
									  if not set weapon hit generator will be used.
									  if weapon define has tag "wr-clustered_shots", "Cluster" hit generator will be forced. 
		"DirectFireModifier" : -10.0, Accuracy modifier if weapon can strike directly
		"FlatJammingChance": 1.0, - Chance of jamming weapon after fire. 1.0 is jamm always. Unjamming logic implemented as in WeaponRealizer
	    "GunneryJammingBase": 5, - 
	    "GunneryJammingMult": 0.05, - this values uses to alter flat jamming chance by pilot skills 
									  formula effective jamming chance = FlatJammingChance + (GunneryJammingBase - Pilot Gunnery)* GunneryJammingMult
									  if FlatJammingChance = 1.0, GunneryJammingBase = 6, GunneryJammingMult = 0.1, GunnerySkill = 10
									  result = 1.0 + (6-10)*0.1 = 0.6
								      GunneryJammingBase if omitted in weapon def., ammo def. and mode def. assumed as 5. 
		"DamageVariance": 20, - Simple damage variance as implemented in WeaponRealizer
		"DistantVariance": 0.3, - Distance damage variance as implemented in WeaponRealizer
		"DistantVarianceReversed": false, - Set is distance damage variance is reversed
		"Cooldown": 2, - number of rounds weapon will be unacceptable after fire this mode
		"AIHitChanceCap": 0.3, - not used any more
		"DamageOnJamming": true/false, - if true on jamming weapon will be damaged
		"DamageMultiplier":2.0, - damage multiplier for this mode effective value will be Weapon.DamagePerShot*Ammo.DamageMultiplier*Mode.DamageMultiplier rounded
									to nearest integer. If omitted assumed to be 1.0. HeatDamagePerShot affected too. 
		"AlwaysIndirectVisuals": false, if true missiles will always plays indirect visuals, even if direct line of sight exists
		"AmmoCategory": "LRM", AmmoCategory can now be overridden by weapon mode. If setted as "NotSet" weapon wouldn't use any ammo. 
		                       If weapon has mode with "NotSet" ammo category you will not see warnings in mechlab for this weapon, 
							   even if you are not supply this weapon with ammo.
		  "AMSHitChance": 0.0, - if this weapon is AMS, this value is AMS efficiency, 
								 if this weapon is missile launcher this value shows how difficult to intercept missile with AMS. Negative value - is harder, 
								 positive is easer.
	}]
  
  
Ammo definition
{
   "Description" : {
      "Id" : "Ammunition_LBX10ECM",
      "Name" : "LBX/10 ECM Ammo",
      "UIName" : "ECM",
      "Details" : "Large caliber rounds capable of dealing heavy damage, and designed to be used in an LBX/10.",
      "Icon" : null,
      "Cost" : 0,
      "Rarity" : 0,
      "Purchasable" : false
   },
   "Type" : "Normal",
   "Category" : "LBX10", 
      
   
   "WeaponEffectID" : "WeaponEffect-Weapon_PPC", Played fire effect can be set in ammo definition, for example this LBX AC10 will fire as PPC if ECM ammo is choosed
   "EvasivePipsIgnored" : 0, This value will be added to EvasivePipsIgnored (current weapon status effects will be used too)
   
   "AccuracyModifier" : -10.0, This value will be added to AccuracyModifier (current weapon status effects will be used too)
   "CriticalChanceMultiplier" : 0.0, This value will be added to CriticalChanceMultiplier (current weapon status effects will be used too)
   "DamagePerShot": -50.0, This value will be added to DamagePerShot (current weapon status effects will be used too)
   "AIBattleValue":90, Not used any more
   "ShotsWhenFired" : 0, This value will be added to ShotsWhenFired (current weapon status effects will be used too)
   "ProjectilesPerShot" : 0, This value will be added to ProjectilesPerShot (current weapon status effects will be used too)
   "HeatDamagePerShot": 0.0, This value will be added to HeatDamagePerShot (current weapon status effects will be used too)
       
   "MinRange": 0.0, This value will be added to MinRange (current weapon status effects will be used too)
   "MaxRange": 0.0, This value will be added to MaxRange (current weapon status effects will be used too)
   "ShortRange": 0.0, This value will be added to ShortRange (current weapon status effects will be used too)
   "MiddleRange": 0.0, This value will be added to MiddleRange (current weapon status effects will be used too)
   "LongRange": 0.0, This value will be added to LongRange (current weapon status effects will be used too)
         NOTE: Range modifications not always displays correctly while viewing shooting arc, but hit chance and possibility calculated normally. 
		 
   "HeatGenerated" : 0, This value will be added to HeatGenerated (current weapon status effects will be used too)
   "RefireModifier" : 0, This value will be added to RefireModifier (current weapon status effects will be used too)
   "Instability" : 0, This value will be added to Instability (current weapon status effects will be used too)
   "AttackRecoil" : 0, This value will be added to AttackRecoil
   "IndirectFireCapable" : false, Effective IndirectFireCapable will be taken from ammo. If not set in ammo define, weapon value will be used
   "EvasivePipsIgnored" : 0, This value will be added to EvasivePipsIgnored (current weapon status effects will be used too)
   "HitGenerator" : "Individual", Set to hit generator. Supported values ("Individual"/"Cluster"/"Streak"). 
                                  Streak hit generator is sort of cluster, 
								  if first projectile hit, rest hit too (location distribution as cluster hit generator),
								  if first projectile misses, rest misses too
								  if not set weapon hit generator will be used.
								  if weapon define has tag "wr-clustered_shots", "Cluster" hit generator will be forced. 
   "DirectFireModifier" : -10.0, Accuracy modifier if weapon can strike directly
   "FlatJammingChance": 1.0, - Chance of jamming weapon after fire. 1.0 is jamm always. Unjamming logic implemented as in WeaponRealizer
   "DamageVariance": 20, - Simple damage variance as implemented in WeaponRealizer
   "DistantVariance": 0.3, - Distance damage variance as implemented in WeaponRealizer
   "DistantVarianceReversed": false - Set is distance damage variance is reversed
   "DamageMultiplier":0.1667, - damage multiplier for this mode effective value will be Weapon.DamagePerShot*Ammo.DamageMultiplier*Mode.DamageMultiplier rounded
								to nearest integer. If omitted assumed to be 1.0.
								I can't use existing "ArmorDamageModifier" and "ISDamageModifier" cause 
								  1) it is unknown what damage must be displayed in HUD
								  2) it is difficult separate damage at place of impact 
   "AMSHitChance": 0.0, - if this weapon is AMS, this value is AMS efficiency, 
                         if this weapon is missile launcher this value shows how difficult to intercept missile with AMS. Negative value - is harder, 
						 positive is easer.
   "AlwaysIndirectVisuals": false, if true missiles will always plays indirect visuals, even if direct line of sight exists
   "HeatGeneratedModifier" : 1,
   "ArmorDamageModifier" : 1,
   "ISDamageModifier" : 1,
   "CriticalDamageModifier" : 1,
   "statusEffects" : [   - will be applied on weapon hit (only "OnHit" effectTriggerType)
        {
            "durationData" : {
                "duration" : 5,
                "ticksOnActivations" : true,
                "useActivationsOfTarget" : true,
                "ticksOnEndOfRound" : false,
                "ticksOnMovements" : false,
                "stackLimit" : 0,
                "clearedWhenAttacked" : false
            },
            "targetingData" : {
                "effectTriggerType" : "OnHit",
                "triggerLimit" : 0,
                "extendDurationOnTrigger" : 0,
                "specialRules" : "NotSet",
                "effectTargetType" : "NotSet",
                "range" : 0,
                "forcePathRebuild" : false,
                "forceVisRebuild" : false,
                "showInTargetPreview" : true,
                "showInStatusPanel" : true
            },
            "effectType" : "StatisticEffect",
            "Description" : {
                "Id" : "AbilityDefPPC",
                "Name" : "SENSORS IMPAIRED",
                "Details" : "[AMT] Difficulty to all of this unit's attacks until its next activation.",
                "Icon" : "uixSvgIcon_status_sensorsImpaired"
            },
            "nature" : "Debuff",
            "statisticData" : {
                "appliesEachTick" : false,
                "effectsPersistAfterDestruction" : false,
                "statName" : "AccuracyModifier",
                "operation" : "Float_Add",
                "modValue" : "5.0",
                "modType" : "System.Single",
                "additionalRules" : "NotSet",
                "targetCollection" : "NotSet",
                "targetWeaponCategory" : "NotSet",
                "targetWeaponType" : "NotSet",
                "targetAmmoCategory" : "NotSet",
                "targetWeaponSubType" : "NotSet"
            },
            "tagData" : null,
            "floatieData" : null,
            "actorBurningData" : null,
            "vfxData" : null,
            "instantModData" : null,
            "poorlyMaintainedEffectData" : null
        }
    ]
}


overriden methods

!!!BattleTech.AttackDirector.AttackSequence.GenerateHitInfo
	Prefix
Implement HitGenerator choosing. Original method completely rewritten and never invoking. 

!!!BattleTech.AttackDirector.AttackSequence.OnAttackSequenceResolveDamage:
	Prefix
add per ammo modification to applying status effects. Original method completely rewritten and never invoking

!!!BattleTech.Weapon.DecrementAmmo:
	Prefix
method completely rewritten to make weapon use only selected ammo and implement streak ammo decremental (decrement only success hits)
	
!!!BattleTech.AbstractActor.CalcAndSetAlphaStrikesRemaining:
	Prefix:
Method completely rewritten to make AI calc remaining alpha strikes correctly base on real weapon ammo category. Original method never invoking
	
!!!BattleTech.Weapon.SetAmmoBoxes
	Prefix
Method completely rewritten to make weapon use right ammo. Original method never invoking.

!!!BattleTech.Weapon.CurrentAmmo
	Prefix
Method completely rewritten to make weapon use right ammo. Original method never invoking.

!!!BattleTech.MechValidationRules.ValidateMechHasAppropriateAmmo:
	Prefix
Method completely rewritten to make mechlab validator functioning correctly.

!!!BattleTech.WeaponRepresentation.PlayWeaponEffect:
	Prefix
Method completely rewritten to play correct effect for each ammo. Original method never invoking.
	
!!!WeaponEffect.PlayProjectile:
	Prefix
Method completely rewritten to make correct AttackRecoil. Original method never invoking.

!BattleTech.ToHit.GetEvasivePipsModifier
	Prefix
add per ammo modification. If modification is done original method not invoked.

!BattleTech.WeaponDef.FromJSON
	Prefix
method make some modification on json. Original method always invoking.

!BattleTech.AmmunitionDef.FromJSON
	Prefix
method make some modification on json. Original method always invoking.

!BattleTech.UI.CombatHUDWeaponSlot.OnPointerDown
	Prefix
add trigger to ammo cycling. If click detected on right side of weapon slot original method not invoking.

!BattleTech.UI.CombatHUDWeaponSlot.OnPointerUp
	Prefix
add trigger to ammo cycling. If click detected on right side of weapon slot original method not invoking.

!BattleTech.UI.CombatHUDWeaponSlot.RefreshHighlighted
	Prefix
add check on DisplayWeapon == null if so original method not invoking.

BattleTech.Weapon.DamagePerShot getter
	Postfix
add per ammo/mode modification

BattleTech.Weapon.HeatDamagePerShot getter
	Postfix
add per ammo/mode modification

BattleTech.Weapon.get_ShotsWhenFired
	Postfix
add per ammo/mode modification

BattleTech.Weapon.get_ProjectilesPerShot:
	Postfix
add per ammo/mode modification

BattleTech.Weapon.get_CriticalChanceMultiplier:
	Postfix
add per ammo/mode modification

BattleTech.Weapon.get_AccuracyModifier:
	Postfix
add per ammo/mode modification

BattleTech.Weapon.get_MinRange:
	Postfix
add per ammo/mode modification

BattleTech.Weapon.get_MaxRange:
	Postfix
add per ammo/mode modification

BattleTech.Weapon.get_ShortRange:
	Postfix
add per ammo/mode modification

BattleTech.Weapon.get_MediumRange:
	Postfix
add per ammo/mode modification

BattleTech.Weapon.get_LongRange:
	Postfix
add per ammo/mode modification

BattleTech.Weapon.get_HeatGenerated:
	Postfix
add per ammo/mode modification

BattleTech.Weapon.get_IndirectFireCapable:
	Postfix
add per ammo/mode modification

BattleTech.Weapon.get_AOECapable:
	Postfix
add per ammo/mode modification

BattleTech.Weapon.Instability:
	Postfix
add per ammo/mode modification

BattleTech.Weapon.get_WillFire:
	Postfix
add per ammo/mode modification

BattleTech.Weapon.RefireModifier getter
	Postfix
add per ammo/mode modification
	
BattleTech.MechComponent.UIName getter:
	Postfix
add per ammo/mode modification

BattleTech.WeaponRepresentation.Init:
	Postfix
Registering additional weapon visual effects.

BattleTech.WeaponRepresentation.ResetWeaponEffect:
	Postfix
resetting additional per ammo visual effects

BattleTech.CombatGameState.ShutdownCombatState:
	Postfix
make some cleaning

BattleTech.AttackDirector.AttackSequence.Cleanup:
	Postfix
helps AI to cycle ammo on depletion 

BattleTech.UI.CombatHUDWeaponSlot.RefreshDisplayedWeapon:
	Transpiler
needed show real projectiles count when ProjectilesPerShot > 1

BattleTech.CombatGameState.RebuildAllLists
	Postfix
registering all weapon and ammo on battlefield.

BattleTech.CombatGameState.OnCombatGameDestroyed:
	Postfix
make some cleaning

BattleTech.UI.CombatHUD.Init:
	Prefix
registering all weapon and ammo on battlefield. Original method always invoking

AttackEvaluator.MakeAttackOrderForTarget
	Prefix
AI make decision what ammo he must use to hit target. Original method invoking always
	
AttackEvaluator.MakeAttackOrder
	Postfix
AI make decision what ammo he must use to hit target.

AIUtil.UnitHasLOFToTargetFromPosition:
	Transpiler
add per ammo modification of IndirectFireCapable. weapon.IndirectFireCapable changed to CustomAmmoCategories.getIndirectFireCapable(weapon)

AIUtil.UnitHasLOFToUnit:
	Transpiler
add per ammo modification of IndirectFireCapable. weapon.IndirectFireCapable changed to CustomAmmoCategories.getIndirectFireCapable(weapon)

BattleTech.AIRoleAssignment.EvaluateSniper:
	Transpiler
add per ammo modification of IndirectFireCapable. weapon.IndirectFireCapable changed to CustomAmmoCategories.getIndirectFireCapable(weapon)

BattleTech.AbstractActor.GetLongestRangeWeapon:
	Transpiler
add per ammo modification of IndirectFireCapable. weapon.IndirectFireCapable changed to CustomAmmoCategories.getIndirectFireCapable(weapon)

BattleTech.AbstractActor.HasIndirectLOFToTargetUnit:
	Transpiler
add per ammo modification of IndirectFireCapable. weapon.IndirectFireCapable changed to CustomAmmoCategories.getIndirectFireCapable(weapon)

BattleTech.AbstractActor.HasLOFToTargetUnit:
	Transpiler
add per ammo modification of IndirectFireCapable. weapon.IndirectFireCapable changed to CustomAmmoCategories.getIndirectFireCapable(weapon)

BattleTech.HostileDamageFactor.expectedDamageForShooting:
	Transpiler
add per ammo modification of IndirectFireCapable. weapon.IndirectFireCapable changed to CustomAmmoCategories.getIndirectFireCapable(weapon)

BattleTech.MultiAttack.FindWeaponToHitTarget:
	Transpiler
add per ammo modification of IndirectFireCapable. weapon.IndirectFireCapable changed to CustomAmmoCategories.getIndirectFireCapable(weapon)

BattleTech.MultiAttack.GetExpectedDamageForMultiTargetWeapon:
	Transpiler
add per ammo modification of IndirectFireCapable. weapon.IndirectFireCapable changed to CustomAmmoCategories.getIndirectFireCapable(weapon)

BattleTech.MultiAttack.PartitionWeaponListToKillTarget:
	Transpiler
add per ammo modification of IndirectFireCapable. weapon.IndirectFireCapable changed to CustomAmmoCategories.getIndirectFireCapable(weapon)

BattleTech.MultiAttack.ValidateMultiAttackOrder:
	Transpiler
add per ammo modification of IndirectFireCapable. weapon.IndirectFireCapable changed to CustomAmmoCategories.getIndirectFireCapable(weapon)

BattleTech.PreferExposedAlonePositionalFactor.InitEvaluationForPhaseForUnit:
	Transpiler
add per ammo modification of IndirectFireCapable. weapon.IndirectFireCapable changed to CustomAmmoCategories.getIndirectFireCapable(weapon)

BattleTech.PreferFiringSolutionWhenExposedAllyPositionalFactor.EvaluateInfluenceMapFactorAtPosition:
	Transpiler
add per ammo modification of IndirectFireCapable. weapon.IndirectFireCapable changed to CustomAmmoCategories.getIndirectFireCapable(weapon)

BattleTech.PreferLethalDamageToRearArcFromHostileFactor.expectedDamageForShooting:
	Transpiler
add per ammo modification of IndirectFireCapable. weapon.IndirectFireCapable changed to CustomAmmoCategories.getIndirectFireCapable(weapon)

BattleTech.PreferNotLethalPositionFactor.expectedDamageForShooting:
	Transpiler
add per ammo modification of IndirectFireCapable. weapon.IndirectFireCapable changed to CustomAmmoCategories.getIndirectFireCapable(weapon)

BattleTech.ToHit.GetAllModifiers:
	Transpiler
add per ammo modification of IndirectFireCapable. weapon.IndirectFireCapable changed to CustomAmmoCategories.getIndirectFireCapable(weapon)
	Postfix
add per ammo or weapon modificator if direct fire detected.

BattleTech.ToHit.GetAllModifiersDescription:
	Transpiler
add per ammo modification of IndirectFireCapable. weapon.IndirectFireCapable changed to CustomAmmoCategories.getIndirectFireCapable(weapon)
	Postfix
add per ammo or weapon modificator if direct fire detected.

BattleTech.UI.CombatHUDWeaponSlot.UpdateToolTipsFiring:
	Transpiler
add per ammo modification of IndirectFireCapable. weapon.IndirectFireCapable changed to CustomAmmoCategories.getIndirectFireCapable(weapon)

BattleTech.UI.CombatHUDWeaponTickMarks.GetValidSlots:
	Transpiler
add per ammo modification of IndirectFireCapable. weapon.IndirectFireCapable changed to CustomAmmoCategories.getIndirectFireCapable(weapon)

BattleTech.Weapon.WillFireAtTargetFromPosition:
	Transpiler
add per ammo modification of IndirectFireCapable. weapon.IndirectFireCapable changed to CustomAmmoCategories.getIndirectFireCapable(weapon)

LOFCache.UnitHasLOFToTarget:
	Transpiler
add per ammo modification of IndirectFireCapable. weapon.IndirectFireCapable changed to CustomAmmoCategories.getIndirectFireCapable(weapon)

</pre>
