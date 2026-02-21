
1) Ao instanciar o CombatScene ele vai receber a lista do EncounterManager diretamente dele como parâmetro
2) Deixa salvo em algum componente do GameManager


Usando Stagemanager

- Toda cena precisa ter uma instância de contexto de scene

Montar o battlecontext antes 



EncounterManager -context> transita para a proxima cena 

TurnBasedManager lê direto do Encounter já na cena de combate.

Basicamente estou trabalhando no Encountermanager. A situação é a seguinte: eu criei uma versão simplificada do EncounterManager para testar a detecção de inimigos e suas respectivas quantidades dentro de um raio ao redor do jogador. Eu consigo detectar, mas a questão é o caminho que eu farei para os dados dos inimigos no EncounterManager chegarem até a tela de combate.

Primeiro terei de fazer o teste com o script específico do Encounter manager sendo acoplado como componente no GameObject do GameManager.

Um novo Battle Context tem que ser gerado dentro do Encounter Manager


## Modificação


O turnbasedCombatManager precisa dos Ibatlers, mas o BattleContextFactory só tem Ids,, 

o TurnbasedCombatManager já consegue converter dos IDs para DTOs, mas isso não é o bastante, precisamos dos prefabs base para recriar os objetos.

 o BattleContextFactory vai pegar o ScenePrefabDatabase da cena de combate para procurar os prefabs necessários, instanciar na cena, aplicar as informações obtidas via DTO e extrair os Iblatlers, para adicionar ao BattleContext novo, e entregar esse novo BattleContext para o TurnbasedCombatManager

**BUGS**

KeyNotFoundException: The given key 'EnemyTest' was not present in the dictionary.
System.Collections.Generic.Dictionary`2[TKey,TValue].get_Item (TKey key) (at <1071a2cb0cb3433aae80a793c277a048>:0)
BattleContextFactory.ExtractEnemyBattlers (BattleBootstrapData bootstrapData) (at Assets/Scripts/GameControl/Combat/BattleContextFactory.cs:35)
EncounterManager.TryStartBattle () (at Assets/Scripts/GameControl/EncounterManager.cs:98)
EnemyProximityTrigger.OnTriggerEnter2D (UnityEngine.Collider2D collision) (at Assets/Scripts/Character/Player/EnemyProximityTrigger.cs:21)