
```mermaid
classDiagram

class IAudioController{
		<<interface>>
        +PlaySFX(AudioClip clip)
	    +PlayMusic(AudioClip clip)
	    +StopMusic()
	    +SetMusicVolume(float volume)
	    +SetSFXVolume(float volume)
        }

class LocalAudioController{
        - AudioSource musicSource
        - AudioSource sfxSource
        + Transform player
        - maxSFXdistance
        - maxMusicDistance
        + PlaySFX(AudioClip clip)
        + PlayMusic(AudioClip clip)
        + StopMusic()
        + SetMusicVolume(float volume)
        + SetSFXVolume(float volume)
}
IAudioController <|.. LocalAudioController: implements

class NPCDTO{
	+String UniqueID 
	+Vector3 NpcPosition  
	+Vector3 NpcRotation
	+Vector3 NpcScale
	+NPCDTO(NPCInteractable npc)
}

class PlayerDTO{
	+float healthPoints
	+float insanity
	+Vector2 playerDirection
	+Vector3 playerPosition
	+Inventory inventory
	+PlayerDTO(Player player)
}

class PrefabDatabase{
	+ List~GameObject~ prefabs
	- GetPrefab(Object npcId): GameObject
}
class ScenePrefabDatabase{
	+ PrefabDatabase prefabDatabase
}

ScenePrefabDatabase *-- PrefabDatabase

class StageData{
	+ String stageName
	+ PlayerDTO playerData
	+ List~NPCDTO~ npcList
}

StageData o--NPCDTO
StageData *--PlayerDTO

class UniqueID{
	- String uniqueID
	+ GetId(): String
}
class AudioManager{
	-float sfxVolume
	-float musicVolume
	-float masterVolume
	- AudioSource musicSource
	- List~IAudioController~ audioControllers
	+ Register(IAudioController audioController)
	+ UnRegister(IAudioController audioController)
	+ UpdateSFXVolume()
	+ PlayMusic(AudioClip clip)
	+ StopMusic()
	- TransitionToNewMusic(AudioClip newClip): IEnumerator
	- FadeOutMusic(): IEnumerator
	- FadeInMusic(): IENumerator
	+ UpdateMusicVolume()
	+ UpdateMasterVolume()
}

AudioManager o-- IAudioController: controls

class GameManager{
 <<singleton>>
 + static GameManager Instance
 + StageManager StageManager
 + AudioManager AudioManager
 + PlayerManager PlayerManager
 + Save Manager SaveManager
 + UIManager UIManager
 + GameStateManager GameStateManager
 + DialogueManager DialogueManager
 + PlayerInput PlayerInput
 - PrefabDatabase currentStagePrefabDataBase
 + GameOver()
 + RastartStage()
 + PauseGame()
 + ChangeControlScheme(String controlScheme)
 + ResumeGame()
}

GameManager *-- AudioManager
GameManager *-- StageManager
GameManager *-- PlayerManager
GameManager *-- SaveManager
GameManager *-- UIManager
GameManager *-- GameStateManager
GameManager *-- DialogueManager
GameManager *-- PrefabDatabase
GameManager <-- ScenePrefabDatabase: updates

class GameSaveData{
+ String currentDecisionTreeNode
+ List~PlotVariable~ plotVariables
+ List~StageData~ stageData
}
GameSaveData o-- StageData
GameSaveData o-- PlotVariable

class GameStateManager{
- GlobalDecisionTree globalDecisionTree
- String currentSotyrNode
- List~PlotVariables~ plotVariables
+ MovetoNote(String nodeID)
+ IsPlotVariableActive(String variableName, bool expectedValue):bool
+ GetCurrentNode():String
+ SetPlotVariable(String variableName, PlotVariableType type, object value)
+ GetPlotVariable(String variableName):object
- GetPlotVariableValue(PlotVariable variable): object
+ GetAllPlotVariables(): List~PlotVariable~ 
+ SetAllPlotVariables(List~PlotVariable~ plotVariables)
}

GameStateManager *-- GlobalDecisionTree

class PlayerManager{
- PlayerDTO currentPlayerData
- GameObject PlayerPrefab
+ CreateNewPlayer(PlayerDTO playerDTO) 
+ FillCurrentPlayerData()
}

PlayerManager *-- Player: retrieves data from
PlayerManager *-- PlayerDTO 

class Player{
- Animator animator
- Float moveSpeed
- RigidBody2D playerRB
- Vector2 moveInput
- Float horizontalMovement
- Float healthPoints
- Float insanity 
- Inventory inventory
- bool isMoving
- bool isFacingRight
+ Move(InputAction.CallbackContext context)
- FlipSprite()
+ StartInventory() 
+Player(PlayerDTO)
}

Player --> PlayerDTO: can be instantiated using
Player *-- Inventory

class SaveManager{
- String SaveFolder
- String SaveFileName
- Dictionary~String,StageData~ cachedStageData
+ SaveGame()
+ LoadGame()
+ SaveStageData(String stageName)
+ LoadStageData(String stageName)
- RestorePlayer(StageData stageData)
+ RestoreNPCsFromStageData(StageData stageData)
- CollectStageData(String stageName): StageData
- CollectNPCData():List~NPCDTO~
- RestoreStageData(StageData stageData)
}

SaveManager  --> StageData : uses
SaveManager --> NPCDTO : manipulates
SaveManager --> UniqueID : uses
SaveManager --> GameSaveData: uses

class StageManager{
- Scene currentScene
- String currentSceneName
- int currentSceneIndex
- Init()
+ LoadStage(String stageName)
}

class ScreenCode{
<<enum>>
 Pause
 Options
 Inventory
 Bestiary
 Recipies
 Diary
 Dialoguebox
}

class UIManager{
- GameObject inventoryUI
- GameObject optionsUI
- GameObject optionsUI
- DialogueUI dialogueBox
+ IsDialogueBoxActive
- GameObject EventSystem
+ Show(ScreenCode screenCode)
+ Hide(ScreenCode screenCode)
- HideAll()
+ ShowDialogueBox()
+ HideDialogueBox()
+ IsVisible(ScreeCode screenCode): bool
+ GetActiveScreen(): ScreenCode
}

UIManager o-- DialogueUI
UIManager --> ScreenCode : uses

class DialogueEvents{
+ static event Action~DialogueLine~ OnShowDialogueLine
+ static event Action~List~String~~ OnShowOptions
+ static event OnClearDialogue
+ static ShowDialogueLine(DialogueLine dialogueLine)
+ static ShowOptions(List~String~ Options, Action~int~ optionSelected)
+ static ClearDialogue()
}

class DialogueLine{
 + String characterName
 + String text
 + Texture2D portrait
}

DialogueEvents --> DialogueLine : manipulates

class DialogueState{
<<enum>>
Idle
ShowingText
AwaitingInput
ShowingOptions
Complete
}

class DialogueManager{
+ DialogueState DialogueState
+ List~DialogueLine~ currentLines
+ List~DialogueOption~ currentOptions
- Action~DialogueOption~ onOptionSelected
- bool isOptionProcessed
+ StartDialogue(List~DialogueLine~ lines, List~DialogueOption~ options, Action~DialogueOption~ optionCallback)
+ ShowNextLine()
- HandleTextAdvance(DialogueLine _)
- HandleOptionSelected(DialogueOption selectedOption)
- ShowDialogueLinesForOption(DialogueOption selectedOption)
- ProcessOptionActions(DialogueOption selectedOption)
- EndDialogue() 
}

DialogueManager *-- DialogueState
DialogueManager --> DialogueLine : manipulates
DialogueManager --> DialogueEvents : uses

class DialogueOption{
+ String text
+ List~DialogueLines~ dialogueLines
+ List~PlotVariable~ triggeredFlags
+ String nextStoryNodeId 
}

class GlobalDecisionTree{
+ List~StoryNode~ nodes
+ GetNodeById(String nodeId): StoryNode
}

GlobalDecisionTree o-- StoryNode

class StoryNode{
+ String nodeId
+ List~PlotVariable~ triggeredVariables
+ List~PlotVariable~ requiredVariables
+ List~String~ nextNodeIds
}

class InteractiveResponse{
+ String label
+ String requiredNode
+ List~PlotVariable~ requiredVariables
+ List ~PlotVariable~ triggeredVariables
+ String nextStoryNodeId
+ List ~DialogueOptions~ options
}

InteractiveResponse o-- PlotVariable
InteractiveResponse o-- DialogueOption

class ItemInteractable{
 - Item item
 + int quantity
 + int width
 + int height
 - Player player 
}

ItemInteractable *-- Item
ItemInteractable *-- Player
ItemInteractable ..|> IInteractable : implements

class IInteractable{
<<interface>>
+ Interact()
}

class NPCInteractable{
-List~InteractiveResponse~ responses
- InteractiveResponse lastResponse
- GetValidResponses(GameStateManager gms) 
}

NPCInteractable --> GameStateManager : recieves information from
NPCInteractable ..|> IInteractable : implements
NPCInteractable o-- InteractiveResponse

class PlotVariableType{
<<enum>>
Bool,
Int,
Float,
String
}

class PlotVariable{
+ String variableName
+ PlotVariableType variableType
+ bool boolValue
+ int intValue
+ float floatValue
+ String stringValue
}

PlotVariable --> PlotVariableType : uses

class PlayerInteraction2D{
+ Vector2 boxSizeVertical
+ Vector2 boxSizeHorizontal
+ float boxDistanceVertical
+ float boxDistanceHorizontal
+ LayerMask interactableLayer
- IInteractable interactable
- Vector2 lastMoveDirection
- Vector2 currentBoxSize
- float currentBoxDistance
- UpdateMoveDirection()
- DetectInteractable()
+ Interact(InputAction.CallbackContext context)  
}

PlayerInteraction2D --> IInteractable : manipulates

class ItemType{
<<enum>>
consumable
relic
weapon
armor
none
}

class Item{
<<scriptable object>>
+ ItemType type
+ String displayName
+ sprite Icon
+ String description
+ bool isKeyItem
}

Item --> ItemType : uses
Item <|-- Armor : extends
Item <|-- Weapon : extends
Item <|-- Consumable : extends

class Armor{
+ int deffense
+ float weight
+ DamageType bonusDeffenseType
+ int bonusDeffense
}

Armor --> DamageType : uses

class DamageType{
<<enum>>
	Piercing
    Slashing
    Bludgeoning
    Fire
    Cold
    Light
    Dark
    Acid
    Poison
    Psychic
    Thunder
}

class Weapon{
+ int baseDmg
+ List~AttackType~ attackTypes
}

Weapon o-- AttackType

class AttackType{
<<scriptable object>>
+ String attackName
+ int bonusDamage
+ dmgPenalty
+ DamageType damageType
+ float range
+ float cooldown
}

AttackType --> DamageType : uses

class Consumable{
	+ List ~Effect~ effects
}

Consumable o-- Effect

class Effect{
+ EffectType Type
+ String Name
+ Sprite Icon
+ virtual Init()
+ virtual Use()

}

Effect --> EffectType : uses

class EffectType{
<<enum>>
	Heal
    Damage
    Buff
    Debuff
    None
}

class Heal{
+ int healAmount 
}

Heal --|> Effect : extends

class DialogueUI{
- VisualElement dialogueArea
- VisualElemento optionBox
- VisualElement optionElement
- Label textLabel
- VisualElement portrait
- HangleDialogueLine(DialogueLine dialogueLine)
}

```

