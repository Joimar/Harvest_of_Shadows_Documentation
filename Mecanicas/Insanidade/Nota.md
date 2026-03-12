
## Problema

A insanidade é um debuff que influencia diretamente a percepção do mundo pelo jogador.

Quanto maior a insanidade, maior a probabilidade de encontrar entidades ocultas ao entrar em uma sala.

Além disso, o uso de magia e o contato com o oculto aumentam a insanidade, criando um ciclo de risco crescente.

Ao atingir níveis críticos de insanidade, a mente do jogador entra em **ruptura cognitiva**, causando distorções nas ações executadas.

---

## Ideia

Criar um **StatusEffect permanente** associado ao estado mental do jogador.

Esse StatusEffect observa o nível de insanidade e define estados mentais progressivos.

Quando a insanidade atinge o limiar máximo, o jogador recebe um **StatusEffect de Ruptura**, que ativa um dos seguintes efeitos:

Tempo real

- controles invertidos
- input mismatch
- random cast

Turn based

- target randomizer
- action randomizer
- spell randomizer

Apenas um efeito de ruptura pode estar ativo por vez.

---

## Abordagem

1. Criar um **StatusEffect permanente** responsável por observar o nível atual de insanidade do jogador e definir seu estado mental.
2. Estruturar a insanidade em **faixas progressivas**, permitindo que seus efeitos escalem de forma previsível até atingir o limiar máximo.
3. Quando a insanidade atingir o limiar crítico, aplicar um **StatusEffect de Ruptura Cognitiva**, responsável por ativar um único efeito de distorção por instância.
4. Tratar os efeitos gerados pela insanidade como **StatusEffects especiais**, com atributos próprios voltados para distorções de gameplay, como alteração de input, targeting, resolução de ações e percepção de ameaças.
5. Conectar a insanidade ao sistema de exploração e combate, fazendo com que níveis mais altos aumentem a percepção do véu e, consequentemente, a ocorrência de entidades, eventos ou ameaças associadas ao oculto.
6. Integrar a insanidade ao uso de magia, de forma que magias aumentem esse recurso e, em níveis críticos, passem a gerar consequências imediatas e imprevisíveis, sem necessariamente bloquear totalmente sua conjuração.
7. Desenvolver **branches específicas para cada variação da mecânica**, separando implementações por tipo de efeito, intensidade e contexto de uso, para evitar acoplamento excessivo e facilitar comparação entre versões.
8. Criar **versões experimentais da feature** voltadas para testes com usuários, em que cada branch represente uma hipótese clara de experiência, como:
    - ruptura mais leve e controlada
    - ruptura mais agressiva e caótica
    - foco em distorção visual
    - foco em randomização de ações
    - foco em perda de precisão tática
9. Realizar **testes com usuários reais** para avaliar recepção da mecânica, medindo aspectos como:
    - clareza do efeito
    - sensação de tensão
    - nível de frustração
    - percepção de justiça
    - preservação de agência do jogador
10. Comparar os resultados entre as diferentes versões testadas e consolidar, na branch principal, apenas os comportamentos que gerarem melhor equilíbrio entre identidade temática, impacto dramático e satisfação do jogador.