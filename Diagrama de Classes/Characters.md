```mermaid
classDiagram

class DefensiveSnapshot{
<<struct>>
+float dodge01
+Dictionary~DamageType,float~ SpecificRes01
+Dictionary~DamageType,float~ GeneralRes01
+GetRes(DamageType t) float
+GetRes(DamageCategory c) float
+CreateDefaults()$
}

DefensiveProfile --|> MonoBehaviour

class DefensiveProfile{
+uint version
+event Action OnChanged
+float dodgeChance
+List~SpecDamageResEntry~ specificDamageResistances
+List~GenDamageResEntry~ generalDamageResistances
+GetResistance(DamageType type) float
+GetResistence(DamageCategory category) float
+GetResistence(String categoryName) float
+SetResistance(DamageType type, int amountPct, bool immune)
+SetResistance(DamageCategory category, int amountPct, bool immune)
+Merge(DefenseStats)
+Unmerge(DefenseStats)
+Merge(DefensiveProfile)
+Unmerge(DefensiveProfile)
+GetSnapshot() DefensiveSnapshot
+InitFromDTO(DefensiveProfileDTO)
+ToDTO() DefensiveProfileDTO

-Touch()
}

DefensiveProfile <-- DefensiveSnapshot : is built based on
DefensiveProfileDTO ..> DefensiveProfile

class SpecPenEntry{
<<struct>>
+DamageType Type
+float amountPct
}

class GenPenEntry{
<<struct>>
+DamageCategory category
+float amountPct
}

class PenetrationSet{
<<struct>>
+float GeneralPct
+float SpecificPct
	+PenetrationSet(float t, float c)
}

class OffensiveSnapshot{
<<struct>>
 +float accuracy01
 +float critChance01
 +CritDamageMultiplier01 
}

class OffensiveProfile{
+uint version
+event Action Onchanged
+float baseAccuracyPct
+float baseCritChancePct
+float baseCritDamageBonusPct
+float maxAccuracyPct
+float maxCritChancePct
%% penetration

+List~GenPenEntry~ generalPenetrations
+List~SpecPenEntry~ specificPenetrations

%%Methods
%%Leitura
+GetSnapshot():OffensiveSnapshot
+GetPenetrationFor(DamageType damageType, DamageTypeMap map, out float penGeneralPct, out float penSpecPct)
+GetPenetrationSet(DamageType t, DamageTypeMap map) PenetrationSet
%%Escrita
+SetSpecificPenetration(DamageType type, int ammountPct)
+SetGeneralPenetration(DamageCategory category, int ammountPct)
+ApplyState(OffensiveState state)
+InitiFromDTO(OffensiveProfileDTO dto)
+ToDTO() OffensiveProfileDTO
}

class OffensiveState{
+float Accuracy01
+float CritChance01
+float CrtiDamage01
-Dictionary~DamageCategory, float~ generalPenetrations
-Dictionary~DamageType, float~ specificPenetrations
+OffensiveState()
+OffensiveState(OffensiveProfile profile)
+Defaults() OffensiveState
%%Leitura
+GetGeneralPenetration(DamageCategory category) float
+GetSpecificPenetration(DamageType type) float
+ToOffensiveSnapshot() OffensiveSnapshot
}

OffensiveProfile o-- SpecPenEntry
OffensiveProfile o-- GenPenEntry
OffensiveProfile --> PenetrationSet:Generates
OffensiveProfile --> OffensiveSnapshot: Generates
OffensiveProfile <.. OffensiveProfileDTO
OffensiveProfile --> OffensiveState:Generates
OffensiveState --> OffensiveSnapshot:Generates

class IProfileProvider{
<<Interface>>
}
IProfileProvider *-- DefensiveProfile
IProfileProvider *-- OffensiveProfile

```