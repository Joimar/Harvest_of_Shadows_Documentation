
Todos os StatusEffects:

| Nome   | Tag    | Duration | Dispellable | Stack Policy   | Power | MaxStacks | Effect                                   |
| ------ | ------ | -------- | ----------- | -------------- | ----- | --------- | ---------------------------------------- |
| Slow   | Debuff | 5s       | Y           | Refresh        | 1     | 5         | Diminui o MoveSpeedMultiplier            |
| Poison | Debuff | 5s       | Y           | Refresh        | 1     | 5         | Aplica um tick de 10HP/s                 |
| Stun   | Debuff | 5s       | Y           | Stack Duration | 1     | 5         | Aplica movespeed =0 e can attack = false |


Tabela de Interações

|        | Slow    | Poison  |     |
| ------ | ------- | ------- | --- |
| Slow   | Refresh |         |     |
| Poison |         | Refresh |     |
|        |         |         |     |
