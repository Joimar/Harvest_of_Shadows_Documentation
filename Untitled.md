# Fase 0 — Pré-sanity (antes de tocar em lógica)

**Objetivo:** evitar NREs enquanto mexemos.

- **Guards universais** (já aplicados no `Player`, repita o padrão onde for parecido):
    
    - Sempre que iniciar uma ação:  
        `_pendingReservation?.Rollback(); _pendingReservation = null;`
        
    - `if (_costs == null) _costs = new CostService();` no `Awake()` (fallback).
        
    - Em handlers de input: `if (inventory == null) return;`
        
    - Em chamadas a managers: `if (targetManager == null) return;`
        
- **Feature flags (simbólicos)**:  
    Defina no topo de arquivos relevantes (opcional):  
    `#define COMBAT_V0` e `#define LEGACY_EFFECTS`  
    (permite ligar/desligar caminho novo sem arrancar o antigo)
    

✅ **Checkpoint 0:** projeto compila como está.

---

# Fase 1 — Contratos mínimos (interfaces sem mexer em lógica)

**Objetivo:** criar contratos que não quebram nada e podem ficar “não usados” até ligar.

1. **IDamageable** (apenas vida)
    
    `public interface IDamageable {   void ApplyDamage(float amount, DamageType type);   void Heal(float amount);   bool IsAlive { get; }   string DisplayName { get; }   Transform Transform { get; } }`
    
2. **IProfileProvider** (leitura dos perfis)
    
    `public interface IProfileProvider {   OffensiveProfile OffensiveProfile { get; }   DefensiveProfile DefensiveProfile { get; } }`
    
3. **IStatusAffectable** passa a **herdar**:
    
    `public interface IStatusAffectable : IDamageable, IProfileProvider {   // mantém seus métodos atuais de status }`
    
4. **Extensions** (não obrigatórios, mas úteis):
    
    `public static class ProfileExtensions {   public static OffensiveSnapshot GetOffense(this IProfileProvider p) => p.OffensiveProfile.GetSnapshot();   public static DefensiveSnapshot GetDefense(this IProfileProvider p) => p.DefensiveProfile.GetSnapshot(); }`
    

✅ **Checkpoint 1:** compila; nada mudou em execução.

---

# Fase 2 — Contextos POCO (somente dados)

**Objetivo:** criar estruturas para o futuro, sem ligar nada ainda.

- `CombatMode { Realtime, TurnBased }`
    
- `EncounterContext { public CombatMode Mode; public int GlobalSeed; }` (pode ficar no GameManager depois)
    
- `ActionIntent(Component attacker, ITargetable source, OffensiveSnapshot offense, CombatMode mode, Guid parent=default)`
    
- `TargetSolution { bool IsValid; List<IDamageable> Targets; Vector3? Point; }`
    
- `ExecutionContext(Component attacker, IDamageable defender, OffensiveSnapshot off, DefensiveSnapshot def, DamageTags tags, PenetrationSet pen, int seed)`
    
- `ResolvedAction` (v0) com um **`Action LegacyExecutor`** interno (para chamar seu fluxo atual) e um `Guid Id`.
    

> Todos **selados** ou não — tanto faz — mas são só **dados**, sem dependências fortes.

✅ **Checkpoint 2:** compila; nada em runtime usa ainda.

---

# Fase 3 — CombatManager v0 (router + log)

**Objetivo:** centralizar execução **sem** tocar em Effects.

- `CombatManager : MonoBehaviour` com:
    
    - `Guid Enqueue(ResolvedAction action)` → **executa imediatamente** `action.LegacyExecutor?.Invoke()`
        
    - `_log : List<ActionLogEntry>` com (Id, SourceName, AttackerName, TargetNames, Timestamp, Mode)
        
    - `event Action<ResolvedAction> OnActionExecuted;`
        
- **GameManager (Opção B):**  
    Adicione um campo `[SerializeField] private CombatManager combatManager;` e uma propriedade de acesso.
    

✅ **Checkpoint 3:** compila; liga na cena e o jogo roda igual (só ainda não usa o CM).

---

# Fase 4 — Integrar Player ao CombatManager v0

**Objetivo:** passar pelo CM sem mudar Effects.

- **Self** (já com commit):  
    Em vez de `targetManager.UseTargetable(...)` direto, crie um `ResolvedAction.Legacy(...)` e `GameManager.Combat.Enqueue(resolved)`.
    
- **Não-Self**: no callback de targeting **sucesso** (onde você já faz `Commit()`), crie o `ResolvedAction` e `Enqueue(...)`.  
    Se targeting falhar/cancelar → `Rollback()` (já está implementado).
    

> O `LegacyExecutor` deve apenas chamar o que você **já chamaria** hoje (p. ex. `targetManager.UseTargetable(targetable)`).

✅ **Checkpoint 4:** jogo roda como antes, mas agora todas as execuções passam pelo CombatManager e ficam logadas.

---

# Fase 5 — IA mínima (depois do CM v0)

**Objetivo:** validar o funil comum sem UI.

- `IAiTargetResolver { TargetSolution ResolveFor(Component caster, ITargetable source); }`
    
- `EnemySpellCaster`:
    
    - Monta `ActionIntent` (OffenseSnapshot do inimigo, `mode = Realtime`).
        
    - Resolve alvo via `IAiTargetResolver` (player mais próximo, p.ex.).
        
    - Cria `ResolvedAction.Legacy(... execute: () => targetManager.UseTargetable(ability))`.
        
    - `GameManager.Combat.Enqueue(...)`.
        

✅ **Checkpoint 5:** inimigos “castam” pelo mesmo funil. Tudo ainda legado por baixo (Effects intactos).

---

# Fase 6 — Primeira migração de Effects (progressiva, com fallback)

**Objetivo:** começar a usar contexto **sem** quebrar o resto.

1. Introduza **`IContextAwareEffect { void Apply(ExecutionContext ctx); }`**.
    
2. No executor legado (no lugar onde percorre `effect.Apply()`), faça:
    
    - `if (effect is IContextAwareEffect e) e.Apply(ctx);`
        
    - `else effect.Apply();` _(caminho antigo)_
        
3. Migre **apenas** o `DealDamageEffect` para usar `ExecutionContext`:
    
    - Use seu `DamageCalculator` com `Offense/Defense` do ctx.
        
    - Aplique no defensor via **`(ctx.Defender).ApplyDamage(final, type)`** (IDamageable).
        
4. Mantenha `LEGACY_EFFECTS` para desligar/ligar behavior antigo se precisar.
    

✅ **Checkpoint 6:** dano usa contexto; cura/buffs/debuffs ainda no legado.

---

# Fase 7 — Ticks e Status (sem mudar controlador ainda)

**Objetivo:** preparar DoT/HoT para o CM sem quebrar o controller atual.

- Ao **aplicar** um Status que cause dano por tick, salve no `StatusEffectInstance`:
    
    - `Component Source`, `IDamageable Target`
        
    - `OffenseSnapshot` do momento da aplicação
        
    - `Seed` do tick
        
    - `DamageType/Tags` do efeito
        
- No **Tick**, crie um pequeno executor:
    
    - **Opção A (curto prazo):** `target.ApplyDamage(amount, type)` diretamente (como hoje).
        
    - **Opção B (pronto pro futuro):** montar `ResolvedAction` de “tick” e `CombatManager.Enqueue()` (permite log e turn-based mais tarde).
        

✅ **Checkpoint 7:** nada quebra; você ganha telemetria de ticks quando quiser.

---

# Fase 8 — Limpesa e endurecimento

**Objetivo:** reduzir risco de exceptions residuais.

- **Null-safety review**:
    
    - Em todas as criações de `ResolvedAction`, valide `Intent`, `Solution.IsValid`, `Targets != null`.
        
    - Em `ExecutionContext` criação, se `defender as IProfileProvider` for nulo → `DefenseSnapshot.Zero()`.
        
- **Validadores suaves** no CombatManager (antes de executar):
    
    - `if (!defender.IsAlive) continue;`
        
    - (Opcional) flags de “ignora LoS” ficam para futuras versões.
        
- **Logs**: adicione “try/catch” no `LegacyExecutor` (já sugerido) e registre exceções com IDs.
    

✅ **Checkpoint 8:** CM robusto a nulos; logs ajudam a detectar outliers.

---

# Fase 9 — (Opcional) Consolidar e desligar legado

**Objetivo:** quando 70–100% dos efeitos estiverem migrados.

- Remova `LEGACY_EFFECTS`.
    
- Padronize aplicadores via `IDamageable`.
    
- Deixe `IStatusAffectable` focada em buffs/debuffs; continue herdando `IDamageable`/`IProfileProvider`.
    

---

## Micro-checklist por arquivo (para você ir marcando)

-  **Interfaces/** `IDamageable.cs`, `IProfileProvider.cs` (+ atualizar `IStatusAffectable : IDamageable, IProfileProvider`)
    
-  **Combat/** `ActionIntent.cs`, `TargetSolution.cs`, `ExecutionContext.cs`, `ResolvedAction.cs`, `ActionLogEntry.cs`
    
-  **Managers/** `CombatManager.cs` (v0) + referência no `GameManager`
    
-  **Player/** integrar CM v0 (Self e Não-Self) — ✅ já adiantado com custos
    
-  **AI/** `IAiTargetResolver.cs`, `EnemySpellCaster.cs` (mínimo)
    
-  **Effects/** introduzir `IContextAwareEffect` e migrar `DealDamageEffect`
    
-  **Status/** (opcional) salvar snapshots/seeds no `StatusEffectInstance` para ticks
    

---

## Dicas anti-exception (durante cada fase)

- **Fail-fast**: `if (obj == null) { Debug.LogWarning(...); return; }` em executores.
    
- **Guards nos arrays/listas**: `if (list == null || list.Count == 0) return;`
    
- **Try/Catch** no `LegacyExecutor`; nunca propague exceção do CM.
    
- **A/B Switch**: mantenha um toggle de `UseCombatManager` no GameManager para isolamento rápido, se precisar.