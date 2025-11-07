Perfeito. Vamos fechar um roadmap claro (com pontos de decisão) para restaurar StatusEffects sem quebrar o “silent load” e sem duplicar CombatMods.

# Roadmap de restauração de StatusEffects (alto nível)

## 0) Ponto de partida

- O `StatusEffectController` está desserializando o `StatusEffectControllerDTO`.
    
- Cada `StatusEffectInstanceDTO` tem: `statusEffectId`, `stacks`, `intensity`, `remainingSeconds`, `nextTickSeconds` (+ payload opcional por efeito).
    

## 1) Tabela-verdade no GameManager

- No `GameManager`, mantenha `Dictionary<string, StatusEffect> StatusEffectsById` populado no `Init()`.
    
- Ao restaurar, resolva o SO com `StatusEffectsById[dto.statusEffectId]`. Se faltar, loga e ignora.
    

## 2) Contexto de criação (StatusEffectContext)

Hoje o `StatusEffectContext` não carrega tempo restante/next tick; só `source`, `intensity`, `initialStacks`.
### Decisão A (preferida, mínima): “ajuste pós-construção”

- **Não** mude a assinatura pública do construtor agora.
    
- Crie a instância normalmente (modo restauração silenciosa).
    
- **Em seguida**, sobrescreva `Remaining`, `_nextTick`, `Stacks`, `Intensity` com os valores do DTO (silencioso).
    

### Decisão B (evolução futura): “contexto estendido”

- Adicionar campos opcionais ao `StatusEffectContext` (ex.: `float? remainingOverride`, `float? nextTickOverride`, `bool restoreMode`).
    
- O construtor aplica overrides quando `restoreMode==true`.
    

Ambas funcionam; **A** exige menos refatoração já que o construtor atual fixa `Remaining/_nextTick` e a gente só “corrige” depois.

## 3) Criar a instância em modo restauração (sem hooks)

- No `StatusEffectController.RestoreFromDTO(...)`:
    
    1. `var so = GameManager.StatusEffectsById[id];`
        
    2. `var ctx = new StatusEffectContext { source = ownerGO /*ou self*/, intensity = dto.intensity, initialStacks = Math.Max(1, dto.stacks) };`
    3. **Criar a instância**: `var inst = new StatusEffectInstance(so, target, ctx);` (isso preenche defaults)
    4.  **Sobrescrever**:
        
        - `inst.Remaining = dto.remainingSeconds;`
            
        - `inst.Intensity = dto.intensity;`
            
        - `inst.Stacks = dto.stacks;`
            
        - `inst._nextTick = dto.nextTickSeconds;`
            
    5. **Registrar** no mapa de ativos do controller (sem chamar `Apply`).
        

> Observação: manteremos `LoadGuards.IsRestoring = true` durante toda a restauração para evitar qualquer caminho que acione VFX/telemetria.

## 4) CombatMods do StatusEffect (regra de ouro)

**Nunca salve o CombatModHub do jogador.**  
Recrie o estado dos mods **em runtime** durante a restauração dos StatusEffects/itens. Duas maneiras equivalentes:

- **Caminho 1 (mais simples agora):**  
    Exponha um método silencioso no `StatusEffect`/instância (ex.: `AttachCombatModsSilently(hub)`) que só anexa os mods (sem OnApply/VFX/dano). O controller chama isso **depois** de criar e popular a instância.
    
- **Caminho 2 (mais puro):**  
    Separe a construção do “pacote de mods” em um método puro no SO (ex.: `BuildModBundle(instance)`), e o controller apenas anexa o bundle no hub do dono.
    

> Para que isso funcione, o `StatusEffectController` precisa conhecer o **dono**: guarde uma referência ao `GameObject`/`ICombatModProvider`/`CombatModifierHub` do alvo ao qual pertence o controller. Assim você sabe onde anexar.

## 5) DTOs (sem explosão por tipo)

- **Um** `StatusEffectInstanceDTO` genérico (cabeçalho comum).
    
- **Payload opcional** (string type + string json) apenas para efeitos que **realmente** precisam de estado extra (ex.: seeds, progressos internos, alvos vinculados).
    
- Nada de DTO por tipo de status; mantenha a fábrica/serialização específica no próprio SO quando necessário.
    

## 6) Safe Room / política de salvamento (relembrando)

- **Não** salve DoTs danosos (ou nem é possível salvar com eles ativos).
    
- Ao entrar na safe room, **purifique** DoTs (remove de cara), mantendo o dano sofrido.
    
- Buffs/HoTs/debuffs não danosos: podem ser salvos; seus timers são restaurados via `remaining/nextTick`.
    

## 7) Ordem canônica no load (Player)

1. `GameManager` → inicializa lookups (inclui `StatusEffectsById`).
    
2. Perfis base (Off/Def) → `ApplyFromDTO`.
    
3. Inventário + **equipados** (setup silencioso, sem hooks).
    
4. **StatusEffects** → `StatusEffectController.RestoreFromDTO(...)` (modo restauração):
    
    - resolve SO por ID,
        
    - cria a instância,
        
    - aplica header do DTO,
        
    - anexa CombatMods **silenciosamente** ao hub do dono.
        
5. Sair do modo restauração → (opcional) marcar dirty/recalcular snapshot.
    
6. HP/Insanidade atuais (clamp), posição, direção.
    

## 8) Critérios de aceite / testes rápidos

- **Roundtrip**: salvar → carregar → salvar novamente: o segundo DTO deve equivaler ao primeiro (mesmo conjunto de instâncias, tempos e stacks).
    
- **Idempotência**: chamar `RestoreFromDTO` duas vezes seguidas no mesmo estado não duplica CombatMods (garanta anexação idempotente ou limpe/reatache na restauração).
    
- **Safe Room**: entrar com DoT → purificado → salvar liberado → carregar sem DoT (mas com HP ferido).
    
- **HoT**: salvar com `remaining` e `nextTick` específicos → carregar e o primeiro tick só acontece quando `nextTick` chegar a zero (sem cura “extra”).
    

---

## Resumo das decisões

- **Construtor atual** de `StatusEffectInstance` permanece; usaremos **sobrescrita pós-criação** agora (Decisão A). (Se preferir, evolua depois para contexto estendido).
    
- **Sem DTO por tipo**: cabeçalho comum + payload opcional por efeito.
    
- **CombatMods não são salvos**: são reanexados silenciosamente ao restaurar cada StatusEffect/itens.
    
- **Controller conhece o dono** para anexar no `CombatModifierHub` correto.
    
- **Silent load garantido**: nada de `Apply`/OnApply; nenhum hook de gameplay dispara no restauro.
    

Se quiser, no próximo passo eu te escrevo os **nomes de métodos e campos** (assinaturas) que entram no `StatusEffectControllerDTO`, `StatusEffectInstanceDTO` e nos pontos de restauração (sem implementação), para você só colar nos arquivos certos e ir preenchendo a lógica aos poucos.