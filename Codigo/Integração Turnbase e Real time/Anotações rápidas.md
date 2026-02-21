
## TurnBasePlayer como Mono

### Prós

- Sempre vai estar acoplado ao player que ele gerar (vai ser um component)
- Quando pegarmos o get current player do game manager, ele já vai vir com o Run Time Player instanciado
- Player manager sempre vai procurar um player na scene, caso não exista ele usa um DTO para criar um novo player
- Melhor acoplamento aos elementos visuais em cena

## Contras

- Sempre presente no player prefab e isso ocupa espaço na memória, enquanto que se usássemos um script separado para administrar isso apenas na cena de combate, o espaço de memória seria menor.
- Sempre que iniciar uma scene em real time, teremos de desabilitá-lo.

## Segunda proposta

Passar o jogador real time invisível na cena de combate.

Transferir os dados desse jogador real time para o jogador turn base

Deixar os scripts turn based acoplados aos seus elementos visuais tendo neles os dados que as suas contrapartes real time lhes forneceram.

## Terceira Proposta

Não usar real time nenhum na cena de combate. Ao invés disso, usar os respectivos DTOs para pegar os dados brutos do real time player e real time enemies. Com tais dados, criar as versões turn base na cena de combate. Daí, não precisamos instanciar os grandes objetos real time.

