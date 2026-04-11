# State Machine Theory — Formal Definitions & Standards

## 1. Formal Definition (Mathematical)

A **Finite State Machine (FSM)** — also called a *Finite Automaton* — is a 5-tuple:

```
M = (Q, Σ, δ, q₀, F)
```

| Symbol | Name | Meaning |
|--------|------|---------|
| `Q` | States | A finite, non-empty set of states |
| `Σ` | Alphabet (Inputs) | A finite set of input symbols / events |
| `δ` | Transition Function | `δ: Q × Σ → Q` (or `→ 2^Q` for NFAs) |
| `q₀` | Initial State | `q₀ ∈ Q`; the state the machine starts in |
| `F` | Accept / Final States | `F ⊆ Q`; states that represent accepted / terminal conditions |

In software engineering, `F` is often replaced by a broader notion of **terminal states** (states with no outgoing transitions) or **goal states**, and `Σ` becomes the domain of **events** rather than characters.

---

## 2. Key Variants

### 2a. Deterministic FSM (DFA / DFSM)
- For every `(state, event)` pair, **exactly one** next state exists.
- `δ: Q × Σ → Q` is a total function.
- Guarantees predictability; the industry default for software design.

### 2b. Non-Deterministic FSM (NFA / NDFSM)
- A `(state, event)` pair may yield **zero or more** possible next states.
- `δ: Q × Σ → 2^Q`
- Used in parsing, regex engines, and theoretical modelling. Rarely used directly in application design.

### 2c. Mealy Machine
- **Output depends on both the current state AND the triggering event.**
- Output is produced *on the transition*.
- Formula: `λ: Q × Σ → Ω` (where `Ω` is the output alphabet)
- Common in protocol design, hardware, and communication systems.

### 2d. Moore Machine
- **Output depends only on the current state.**
- Output is associated with the state, not the transition.
- Formula: `λ: Q → Ω`
- Simpler to reason about; preferred in UI state modelling and XState.

### 2e. Extended State Machine (ESM)
- Augments FSM with **context variables** (extended state / "context").
- Conditions on transitions become **guards**: boolean expressions over context.
- Actions can **update context** on entry, exit, or transition.
- This is the model used by XState, SCXML, and most modern state management libraries.

### 2f. Hierarchical State Machine (HSM) / Statecharts
- Introduced by David Harel (1987) in *"Statecharts: A Visual Formalism for Complex Systems"*.
- States can **contain other states** (nested / compound states).
- Key concepts:
  - **Compound state**: a state that has sub-states
  - **Atomic state**: a state with no sub-states
  - **History state**: a pseudo-state that remembers the last active sub-state
  - **Parallel (orthogonal) regions**: multiple sub-machines running concurrently

### 2g. Pushdown Automaton (PDA)
- FSM + an infinite **stack** memory.
- Can recognise context-free languages (e.g., nested structures).
- Relevant when the domain has recursive or nested event sequences.

---

## 3. Core Concepts & Terminology

### States
| Term | Definition |
|------|-----------|
| **State** | A distinct mode or condition in which a system can exist at a given time. A snapshot of the system's "memory" that determines how it responds to future events. |
| **Initial state** | The state the machine occupies at the moment of creation/reset. Every machine has exactly one. |
| **Final / terminal state** | A state from which no further transitions are taken, or which represents completion. |
| **Transient state** | A state that immediately transitions to another without waiting for an external event (often used for intermediate computation). |
| **Idle / resting state** | A stable state where the machine waits for external events. |

### Transitions
| Term | Definition |
|------|-----------|
| **Transition** | A directed edge from a source state to a target state, labelled with event + optional guard + optional action. |
| **Event** | A named signal that can trigger a transition. Events may carry a payload. |
| **Guard** | A boolean condition that must be true for a transition to fire. Multiple transitions from the same state on the same event can have different guards (conditional branching). |
| **Self-transition** | A transition whose source and target are the same state. Useful for triggering actions without changing state. |
| **Internal transition** | Like a self-transition but does NOT trigger exit/entry actions. |
| **Forbidden transition** | A `(state, event)` pair with no defined transition; the event is ignored or causes an error. |
| **Default transition** | A transition taken when no other guard is satisfied (catch-all). |

### Actions
| Term | Definition |
|------|-----------|
| **Entry action** | Executes when the machine enters a state. |
| **Exit action** | Executes when the machine leaves a state. |
| **Transition action** | Executes during a specific transition (after exit, before entry). |
| **Activity** | A long-running action that executes while in a state (as opposed to instantaneous actions). |

### Extended State
| Term | Definition |
|------|-----------|
| **Context** | Arbitrary data associated with the machine instance that can be read by guards and modified by actions. |
| **Assign action** | An action that updates context variables. |
| **Condition** | A named boolean function over context, used in guards. |

---

## 4. State Diagram Notation (UML 2.x)

The standard visual language for state machines is defined in **UML 2.x State Machine Diagrams**:

```
● ──> [State A] ──event [guard] / action──> [State B] ──> ◎
```

| Symbol | Meaning |
|--------|---------|
| `●` (filled circle) | Initial pseudo-state |
| `◎` (circled dot) | Final state |
| `[Rectangle]` | State |
| `──event──>` | Transition triggered by event |
| `[guard]` | Guard condition (optional) |
| `/ action` | Action executed on transition (optional) |
| Nested rectangle | Composite / compound state |
| Dashed horizontal line | Parallel (fork/join) regions |
| `H` in circle | Shallow history pseudo-state |
| `H*` in circle | Deep history pseudo-state |

### Transition Label Syntax (UML)
```
event [guard] / action
```
All three parts are optional:
- Just `event` → always fires on that event
- `[guard]` alone → eventless (fires as soon as guard becomes true)
- `/ action` alone → always fires when leaving state (unconditional)

---

## 5. SCXML — The W3C Standard

**SCXML** (State Chart XML) is the W3C standard (REC 2015) for serialising statecharts. It is the most interoperable machine definition format.

Key elements:
```xml
<scxml initial="idle" version="1.0" xmlns="http://www.w3.org/2005/07/scxml">
  <state id="idle">
    <transition event="submit" target="loading"/>
  </state>
  <state id="loading">
    <onentry><send event="fetchData"/></onentry>
    <transition event="done" target="success"/>
    <transition event="error" target="failure"/>
  </state>
  <final id="success"/>
  <final id="failure"/>
</scxml>
```

---

## 6. The Actor Model & State Machines

In modern systems design, state machines are composed via the **Actor Model**:
- Each actor is an independent state machine with its own context.
- Actors communicate exclusively through **message passing** (events).
- Actors can **spawn child actors** (parallel sub-machines).
- This is the model behind XState v5's actor system, Erlang/OTP gen_statem, and Akka.

---

## 7. Fundamental Properties & Theorems

### 7a. Equivalence
Two FSMs are **equivalent** if they accept exactly the same set of input strings. Every NFA has an equivalent DFA (subset construction algorithm).

### 7b. Minimisation
Every DFA has a unique **minimal DFA** (fewest possible states) equivalent to it (Myhill-Nerode theorem). Minimisation is important for understanding when two states are truly distinct vs. collapsible.

### 7c. Reachability
A state `q` is **reachable** if there exists a sequence of transitions from `q₀` to `q`. Unreachable states are dead code and should be eliminated.

### 7d. Liveness vs. Safety
- **Safety property**: "something bad never happens" — no forbidden state is ever entered.
- **Liveness property**: "something good eventually happens" — the machine always makes progress toward a goal.

---

## 8. Common State Machine Patterns

### Traffic Light
```
red ──timer──> green ──timer──> yellow ──timer──> red
```
Classic example: deterministic, cyclical, no context.

### Fetch / Async Operation (Promise model)
```
idle ──fetch──> loading ──success──> success
                        ──error───> failure ──retry──> loading
                                           ──giveUp──> idle
```

### Authentication Flow
```
unauthenticated ──login──> authenticating ──success──> authenticated
                                          ──failure──> unauthenticated
authenticated ──logout──> unauthenticated
              ──expire───> unauthenticated
```

### Form / Wizard
```
step1 ──next [valid]──> step2 ──next [valid]──> step3 ──submit──> submitting
      ──next [!valid]──> step1 (self-transition, show errors)
submitting ──success──> done
           ──error───> step3
```

---

## 9. State Machine vs. Related Concepts

| Concept | How it relates to FSMs |
|---------|----------------------|
| **Behaviour Tree** | Hierarchical but uses ticks (polling) rather than events; preferred in game AI |
| **Workflow engine** | An FSM with persistence, human tasks, timers, and long-running processes (BPMN) |
| **Redux reducer** | A Mealy machine: `(state, action) → state` — pure FSM without guards or hierarchy |
| **Event sourcing** | Treats state as the fold of events over time; compatible with but broader than FSMs |
| **Petri net** | Generalisation allowing concurrent tokens and places; more expressive than FSMs |

---

## 10. Why State Machines for Software Design?

1. **Eliminates impossible states** — by enumerating all valid states, invalid combinations are made structurally impossible.
2. **Makes implicit logic explicit** — conditional spaghetti becomes a visible graph.
3. **Enables exhaustive testing** — each `(state, event)` pair is a testable unit.
4. **Self-documenting** — the diagram IS the specification.
5. **Handles async naturally** — loading, error, and success states are first-class.
6. **Composable** — machines can be nested, parallelised, and communicated as actors.
