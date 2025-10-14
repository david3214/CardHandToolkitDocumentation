# Card Hand Toolkit — API Hub

> This is the entry point for all public APIs. It outlines core concepts, links to deep‑dive pages, and provides an A–Z reference for every public type.

---

## How to Use This Hub

1. **Start with Concepts** for a mental model.
2. Jump to a **Module** page for systems you’ll extend (Interaction, Zones, Views, Data, Visuals, Layout).
3. Use the **A–Z Index** for quick lookups on specific components and interfaces.

> **Docs style per entry** (used across this repo):
>
> * **Description:** What this type does and when to use it.
> * **Customizable Attributes:** Serialized fields/inspector knobs.
> * **Group Details:** Collaborators & relationships.
> * **Cell Details:** Key methods, events, extension points.
> * **Example:** Minimal code or scene steps.

---

## Table of Contents

* [Concepts](#concepts)
* [Modules Map](#modules-map)

  * [Interaction](API/Interaction.md)
  * [Zones & Play](API/ZonesAndPlay.md)
  * [Views & Presentation](API/Views.md)
  * [Data & Deck](API/DataAndDeck.md)
  * [Visuals & Factories](API/Visuals.md)
  * [Layout](API/Layout.md)
* [A–Z Type Index](#a–z-type-index)
* [FAQ](#faq)

> All module links are relative so they work in both **GitHub** and **Unity**.

---

## Concepts

> **Goal:** Give you the exact mental model so you can extend safely. This section is concept‑heavy (with diagrams); the A–Z entries stay terse.

### A. Canonical Card Lifecycle (Local Player)

1. **Draw (data‑only):** `Deck` → GUID pulled. `HandStateService.Add(CardRef)` creates/returns a **CardRef** `{instanceId, definitionGuid}`; if the card already exists, `AddExisting` reuses its `instanceId`.
2. **Hand event:** `PlayerHand` listens → **OnCardAdded(CardRef)**.
3. **Scene reuse check:** `SceneCardLookup.TryGet(instanceId)` →

   * **Exists:** parent under `PlayerHand`, set pose → **Normal**.
   * **Missing:** build a new visual via **VisualFactoryLoader.Build(CardRef, drawOrigin)**.
4. **Build pipeline:**

   * **Lookup data:** `CardLibraryLoader.Get(definitionGuid)` → `ScriptableCard`.
   * **Instantiate prefab** for the card.
   * **Initialize view root:** `CardBehaviour.Initialize(ScriptableCard, instanceId)`

     1. registers into `SceneCardLookup`
     2. pushes textures to all `ICardView` (e.g., `Card3DView.SetFrameSpriteByCardRarity`, `SetArtSprite`).
   * **Create ViewModel** via the VM factory (inside the loader/build path).
   * **Presenter init:** `CardPresenter.Initialize(global CardSkin, viewModel)` →

     1. instantiates **World View Prefab** onto the card’s world canvas
     2. binds the prefab’s **ICardViewBinder** to the **ViewModel**
     3. toggles fields using **ViewModel + CardSkin**
     4. calls **Refresh** to fan out updates.
5. **Add to hand:** parented under `PlayerHand`.
6. **Runtime visuals:** `CardPoseView` (an `ICardView` on the view root) applies **face up/down**, **tapped**, and **hover scale** on `CardBehaviour.Refresh`.
7. **Hover/drag ready:** interaction components enabled based on zone (see policy matrix below).
8. **Play action:** crossing **play height** & drop → iterate scene `IPlayResolver`s by **priority** → first `TryResolvePlay` that succeeds removes from `HandStateService` and adds the card to the accepting `ICardZone`.
9. **Zone visuals & overlay:** In zones, hover does **not** enlarge in place; it emits events handled by `HoverPresenter` to show an **overlay canvas** (uses the same World View Prefab unless the global skin provides an overlay override).

> **Diagram:** `Images/card-lifecycle.png` (Sequence: Deck → HandStateService → PlayerHand → VisualFactoryLoader → CardBehaviour/CardPresenter → PlayerHand → Interaction → Resolvers → Zone).

### B. Sources of Truth (Who owns what?)

* **CardBehaviour** → runtime identity & presentation state: `ScriptableCard`, `instanceId`, pose state (face/tap/hover), references to view root.
* **CardPresenter** → **not** a data owner; it orchestrates **binding & cascade refresh** using `CardSkin + ViewModel`.
* **ViewModel (ICardViewModel)** → structured data for binding; created by the build path.
* **HandStateService** → list of card refs per seat; emits hand events; writes are mirrored by zones via resolvers.
* **SceneCardLookup** → scene‑level map `instanceId → GameObject`.
* **CardVisibilityService** → face‑up/down decisions (global/local knowledge helpers).

### C. Interaction Model (Allowances vs Input)

* **PointerService** (static) abstracts **input backends**: `TryGet(out Pointer)` for both legacy & new Input Systems. Used by **Hands**, **Drag**, **Hover**.
* **InteractionService** evaluates **what’s allowed** for a card given its zone & policy.
* **DefaultInteractionPolicy** encodes:

  * **Hand (local):** hover ✓ / drag ✓ / select (planned)
  * **Opponent hand:** none
  * **Board:** hover ✓ / select ✓ / target ✓ / drag ✗
* Zone changes toggle components (`DragCard`, `CardHoverable`, etc.) accordingly.

### D. Hover Pipeline (Single‑card arbitration)

1. **CardHoverable** detects pointer hit.
2. **HoverPointerArbiter** ensures **only one** hovered card.
3. **CardBehaviour** applies **hover pose**; fires `OnHoverEnter/Exit(HoverEventPayload)`.
4. **HoverEventBinder** forwards to **HoverPolicyRuntime** → **HandleHoverEnter/Exit**.
5. In **hand**: short‑circuits to simple enlarge. In **zones**: `HoverPresenter` shows overlay (uses `HoverAnchor`, `HoverPolicyRuntime` for sizing/placement).

> **Diagram:** `Images/hover-pipeline.png` (Hit → Arbiter → Behaviour → EventBinder → Policy → Presenter).

### E. Play Resolution Pipeline (Priority‑ordered)

1. **Drag drop past threshold** (not a zone collider).
2. Iterate **IPlayResolver** instances by **priority**; call `TryResolvePlay(cardRef, pointerState)` until one accepts.
3. Resolver updates **HandStateService** (remove from hand) and adds to its **ICardZone**.
4. Zone calls `AlignCard` and updates visuals; interaction components updated per policy.

> **Diagram:** `Images/play-resolution.png` (Drop → Resolvers(prio) → Winner → HandStateService → Zone).

### F. Hands are Zones

* `PlayerHand` and `OpponentHand` **implement `ICardZone`** (have zone types, roots, `AddCard/RemoveCard/AlignCard`).
* They primarily act as **layout containers**; **HandStateService** remains the data authority for hand membership.

### G. Operational Loaders (Runtime Utilities)

* **VisualFactoryLoader** builds card visuals + creates ViewModel + kicks Presenter.
* **CardLibraryLoader** resolves `ScriptableCard` by GUID.
* **CardFrameLibraryLoader**, **LayersLoader** provide assets/layers; they’re singletons used at runtime but typically auto‑wired by the Setup Wizard.

### H. Services Map (at a glance)

* **PointerService** — input abstraction (backend‑agnostic).
* **HandStateService** — seat hands & events.
* **SceneCardLookup** — id→GO registry.
* **CardVisibilityService** — face/knowledge helpers.

---

## Modules Map

### Interaction — *(extend here first for custom UX)*

* **Deep doc:** [API/Interaction.md](API/Interaction.md)
* Includes: `InteractionService`, `InteractionController`, `IInteractionPolicy`, `DefaultInteractionPolicy`, `IInteractionOverride`/`CardInteractionOverride`, `CardHoverable`, `DragCard`, `HoverPolicyRuntime`, `HoverRoutingPolicy`, `IHoverPresenter`/`HoverPresenter`, `HoverAnchor`, `PointerService`/`IPointerSource`.

### Zones & Play — *(define how/where cards can be played)*

* **Deep doc:** [API/ZonesAndPlay.md](API/ZonesAndPlay.md)
* Includes: `ICardZone`, `IPlayResolver`, `CardZoneTracker`, examples `SingleSlotZone`, `LineBoardZone`.

### Views & Presentation — *(what a card looks like & how it binds data)*

* **Deep doc:** [API/Views.md](API/Views.md)
* Includes: `ICardView`, `CardBehaviour`, `Card3DView`, `CardPoseView`, `CardPresenter` (sealed).

### Data & Deck — *(what a card IS & how it’s stored/drawn)*

* **Deep doc:** [API/DataAndDeck.md](API/DataAndDeck.md)
* Includes: `ScriptableCard`, `CardDatabase`, `Deck`, `DeckList`, `EmptyDeckPolicies`, `HandStateService`.

### Visuals & Factories — *(skins, frames, factories, and loaders)*

* **Deep doc:** [API/Visuals.md](API/Visuals.md)
* Includes: `CardSkin`, `CardColorPalette`, `CardAddons`, `CardVisualFactorySO`, `VisualFactoryLoader`, `CardFrameLibrary`, `CardFrameLibraryLoader`, `CardLibraryLoader`, `LayersLoader`, `CardToolkitLayers`, `ICardViewModel`, `ICardViewBinder`, `IContextualSwitcher`.

### Layout — *(arrangement & fan poses)*

* **Deep doc:** [API/Layout.md](API/Layout.md)
* Includes: `FanLayout`, `FanPose`, `FanLayoutProfile`.

---

## A–Z Type Index

> Legend: **[E]** Extend/Implement • **[C]** Configure (Inspector) • **[L]** Learn-only

### Card3DView **[C]**

**Description:** *TODO*
**Customizable Attributes:** *TODO*
**Group Details:** *TODO*
**Cell Details:** *TODO*
**Example:** *TODO*

### CardAddons **[C]**

**Description:** *TODO*
**Customizable Attributes:** *TODO*
**Group Details:** *TODO*
**Cell Details:** *TODO*
**Example:** *TODO*

### CardBehaviour **[E]**

**Description:** *TODO*
**Customizable Attributes:** *TODO*
**Group Details:** *TODO*
**Cell Details:** *TODO*
**Example:** *TODO*

### CardColorPalette **[C]**

**Description:** *TODO*
**Customizable Attributes:** *TODO*
**Group Details:** *TODO*
**Cell Details:** *TODO*
**Example:** *TODO*

### CardDatabase **[C]**

**Description:** *TODO*
**Customizable Attributes:** *TODO*
**Group Details:** *TODO*
**Cell Details:** *TODO*
**Example:** *TODO*

### CardFrameLibrary **[C]**

**Description:** *TODO*
**Customizable Attributes:** *TODO*
**Group Details:** *TODO*
**Cell Details:** *TODO*
**Example:** *TODO*

### CardFrameLibraryLoader **[C]** *(Operational)*

**Description:** *TODO*
**Customizable Attributes:** *TODO*
**Group Details:** *TODO*
**Cell Details:** *TODO*
**Example:** *Setup Wizard typically adds this.*

### CardHoverable **[C]**

**Description:** *TODO*
**Customizable Attributes:** *TODO*
**Group Details:** *TODO*
**Cell Details:** *TODO*
**Example:** *TODO*

### CardLibraryLoader **[C]** *(Operational)*

**Description:** *TODO*
**Customizable Attributes:** *TODO*
**Group Details:** *TODO*
**Cell Details:** *TODO*
**Example:** *Setup Wizard typically adds this.*

### CardPoseView **[C]**

**Description:** *TODO*
**Customizable Attributes:** *TODO*
**Group Details:** *TODO*
**Cell Details:** *TODO*
**Example:** *TODO*

### CardPresenter (sealed) **[L]**

**Description:** *TODO*
**Group Details:** *TODO*
**Cell Details:** *Key events/callbacks only*
**Example:** *N/A (sealed)*

### CardSkin **[C]**

**Description:** *TODO*
**Customizable Attributes:** *TODO*
**Group Details:** *TODO*
**Cell Details:** *TODO*
**Example:** *Inspector-focused*

### CardToolkitLayers **[L]** *(Constants)*

**Description:** *Do not modify IDs; used by Setup Wizard and gizmos.*
**Customizable Attributes:** *None*
**Group Details:** *TODO*
**Cell Details:** *TODO*
**Example:** *N/A*

### CardVisualFactorySO **[C]**

**Description:** *TODO*
**Customizable Attributes:** *TODO*
**Group Details:** `VisualFactoryLoader`
**Cell Details:** *TODO*
**Example:** *TODO*

### CardZoneTracker **[E]**

**Description:** *TODO*
**Customizable Attributes:** *TODO*
**Group Details:** `ICardZone`, `IPlayResolver`
**Cell Details:** *TODO*
**Example:** *TODO*

### Deck **[E]**

**Description:** *TODO*
**Customizable Attributes:** *N/A*
**Group Details:** `DeckList`, `EmptyDeckPolicies`
**Cell Details:** *Key methods: draw, peek, shuffle…*
**Example:** *TODO*

### DeckList **[C]**

**Description:** *TODO*
**Customizable Attributes:** *TODO*
**Group Details:** `Deck`
**Cell Details:** *TODO*
**Example:** *TODO*

### DefaultInteractionPolicy **[C]**

**Description:** *TODO*
**Customizable Attributes:** *TODO*
**Group Details:** `InteractionService`, `InteractionController`
**Cell Details:** *TODO*
**Example:** *TODO*

### DragCard **[C]**

**Description:** *TODO*
**Customizable Attributes:** *TODO*
**Group Details:** `InteractionController`, `PointerService`
**Cell Details:** *TODO*
**Example:** *TODO*

### EmptyDeckPolicies **[C]**

**Description:** *TODO*
**Customizable Attributes:** *TODO*
**Group Details:** `Deck`
**Cell Details:** *TODO*
**Example:** *TODO*

### FanLayout / FanPose **[C]**

**Description:** *TODO*
**Customizable Attributes:** *TODO*
**Group Details:** `FanLayoutProfile`, `PlayerHand`, `OpponentHand`
**Cell Details:** *TODO*
**Example:** *TODO*

### FanLayoutProfile **[C]**

**Description:** *Profile asset that defines curvature, spacing, etc.*
**Customizable Attributes:** *TODO*
**Group Details:** `FanLayout`
**Cell Details:** *TODO*
**Example:** *TODO*

### HandStateService **[L]**

**Description:** *TODO*
**Customizable Attributes:** *N/A*
**Group Details:** `PlayerHand`, `OpponentHand`
**Cell Details:** *Key enums/values*
**Example:** *N/A*

### HandViewBase **[L]**

**Description:** *Shared properties for `PlayerHand` and `OpponentHand`.*
**Customizable Attributes:** *List shared props only*
**Group Details:** `FanLayout`
**Cell Details:** *TODO*
**Example:** *N/A*

### HoverAnchor **[C]**

**Description:** *TODO*
**Customizable Attributes:** *TODO*
**Group Details:** `HoverPresenter`
**Cell Details:** *TODO*
**Example:** *TODO*

### HoverPolicyRuntime **[C]**

**Description:** *TODO*
**Customizable Attributes:** *TODO*
**Group Details:** `HoverPresenter`, `CardHoverable`
**Cell Details:** *TODO*
**Example:** *TODO*

### HoverPresenter / IHoverPresenter **[C]/[E]**

**Description:** *TODO*
**Customizable Attributes:** *TODO*
**Group Details:** `HoverAnchor`, `HoverPolicyRuntime`
**Cell Details:** *Interface points & default implementation*
**Example:** *TODO*

### HoverRoutingPolicy **[E]**

**Description:** *TODO*
**Customizable Attributes:** *N/A*
**Group Details:** `HoverPresenter`
**Cell Details:** *TODO*
**Example:** *TODO*

### ICardView **[E]**

**Description:** *TODO*
**Customizable Attributes:** *N/A*
**Group Details:** `CardBehaviour`, `CardPresenter`
**Cell Details:** *Interface members*
**Example:** *Minimal implementation snippet*

### ICardViewBinder **[E]**

**Description:** *TODO*
**Customizable Attributes:** *N/A*
**Group Details:** `ICardViewModel`
**Cell Details:** *Binder hooks*
**Example:** *TODO*

### ICardViewModel **[E]**

**Description:** *TODO*
**Customizable Attributes:** *N/A*
**Group Details:** `ICardViewBinder`, `CardVisualFactorySO`
**Cell Details:** *Model fields*
**Example:** *TODO*

### ICardZone **[E]**

**Description:** *Implement to create board zones.*
**Customizable Attributes:** *Varies per concrete zone*
**Group Details:** `IPlayResolver`, `CardZoneTracker`
**Cell Details:** *Required members*
**Example:** *Minimal custom zone snippet*

### IContextualSwitcher **[E]**

**Description:** *TODO*
**Customizable Attributes:** *N/A*
**Group Details:** `ICardViewModel`
**Cell Details:** *Switch rules*
**Example:** *TODO*

### IInteractionOverride / CardInteractionOverride **[E]**

**Description:** *TODO*
**Customizable Attributes:** *N/A*
**Group Details:** `IInteractionPolicy`
**Cell Details:** *Override surface*
**Example:** *TODO*

### IInteractionPolicy **[E]**

**Description:** *TODO*
**Customizable Attributes:** *N/A*
**Group Details:** `InteractionService`
**Cell Details:** *Policy callbacks*
**Example:** *TODO*

### IPlayResolver **[E]**

**Description:** *Resolve whether/where a card can enter a zone.*
**Customizable Attributes:** *N/A*
**Group Details:** `ICardZone`
**Cell Details:** *Required members*
**Example:** *Quick example with `SingleSlotZone`*

### InteractionController **[E]**

**Description:** *TODO*
**Customizable Attributes:** *TODO*
**Group Details:** `InteractionService`, `DragCard`
**Cell Details:** *TODO*
**Example:** *TODO*

### InteractionService **[E]**

**Description:** *TODO*
**Customizable Attributes:** *TODO*
**Group Details:** `IInteractionPolicy`, `IInteractionOverride`
**Cell Details:** *TODO*
**Example:** *TODO*

### LayersLoader **[C]** *(Operational)*

**Description:** *Creates/validates required layers.*
**Customizable Attributes:** *Usually none*
**Group Details:** `CardToolkitLayers`
**Cell Details:** *TODO*
**Example:** *Setup Wizard adds this.*

### LineBoardZone *(example)* **[L]**

**Description:** *Starter zone; not production-ready.*
**Customizable Attributes:** *Few*
**Group Details:** `ICardZone`, `IPlayResolver`
**Cell Details:** *Minimal*
**Example:** *See Zones deep doc*

### OpponentHand **[C]**

**Description:** *TODO*
**Customizable Attributes:** *TODO*
**Group Details:** `HandViewBase`, `FanLayout`
**Cell Details:** *Events*
**Example:** *TODO*

### PlayerHand **[C]**

**Description:** *TODO*
**Customizable Attributes:** *TODO*
**Group Details:** `HandViewBase`, `FanLayout`
**Cell Details:** *Events*
**Example:** *TODO*

### PointerService / IPointerSource **[L]/[E]**

**Description:** *Abstraction over legacy vs new input systems; implement custom `IPointerSource` if needed.*
**Customizable Attributes:** *N/A*
**Group Details:** `DragCard`, `InteractionController`
**Cell Details:** *`TryGet(out Pointer)`*
**Example:** *Custom input source snippet*

### ScriptableCard **[C]**

**Description:** *TODO*
**Customizable Attributes:** *Fields for name, art, cost, etc.*
**Group Details:** `CardDatabase`
**Cell Details:** *TODO*
**Example:** *Inspector-focused*

### SingleSlotZone *(example)* **[L]**

**Description:** *Starter zone; not production-ready.*
**Customizable Attributes:** *Few*
**Group Details:** `ICardZone`, `IPlayResolver`
**Cell Details:** *Minimal*
**Example:** *See Zones deep doc*

### VisualFactoryLoader **[C]**

**Description:** *Scene singleton providing `ICardVisualFactory`.*
**Customizable Attributes:** *`visualFactory` reference*
**Group Details:** `CardVisualFactorySO`
**Cell Details:** *`IsReady`, boot checks*
**Example:** *Scene setup steps*

---

## FAQ

**Q: Which pages should I read first?**
A: Start with **Interaction** (how players touch cards) and **Zones & Play** (where cards can go). Then skim **Views** to understand binding.

**Q: What’s safe to modify?**
A: Anything tagged **[C]** is intended for configuration. Types tagged **[E]** are designed for extension via interfaces/policies. Types tagged **[L]** are primarily informational.

**Q: Do I have to use the example zones?**
A: No. They’re teaching tools. Implement your own `ICardZone` + `IPlayResolver` for production.

---

> **Next:** Dive into a module: [Interaction](API/Interaction.md) • [Zones & Play](API/ZonesAndPlay.md) • [Views](API/Views.md) • [Data & Deck](API/DataAndDeck.md) • [Visuals](API/Visuals.md) • [Layout](API/Layout.md)
