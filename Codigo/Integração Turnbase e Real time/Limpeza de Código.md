
## 1 - Alterar os Turn Base Enemy e Turn Base Player

Detalhes: 
- BattleContext tem que gerar turn base enemy e player baseado nos DTOs dos Real Time.
- Adicionar nos DTOs os Profiles DTOs e os Status Effects DTOs.
## 2 - Alterar BattleContextFactory

Detalhes: 
-  Ele vai pegar os DTOs e os prefabs novos e instanciar os prefabs e atribuir os valores dos DTOs nos prefabs
- 2 depende de 1 e de 3
- Primeiro trabalhe no tópico 1, depois 3 e depois 2.

## 3 - Fazer um ScenePrefebDatabase novo para a cena de combate

- Usar elementos de UI Tool Kit, UIGUI para inimigos e jogador (usa game object padrão). Adicionar Game Object chamado Canvas na cena. No Canvas todos os elementos instanciados vão aparecer em screen space.
## 4 - Criar método de carregamento de cena alternativo

Descrição: Carregar para aonde o jogador vai na próxima cena

Motivo: Atualmente se o jogador sair da scene por uma porta, ao voltar para a mesma scene por uma outra porta, ele vai aparecer na primeira porta, pois é o único posicionamento que ele tem salvo no StageData.

## 5 - Modificar o Status Effect para ignorar ticks por segundo quando o combate por turnos ocorrer

