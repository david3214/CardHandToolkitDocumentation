# Card Hand Toolkit — API Hub

> This is the entry point for all public APIs. It outlines core concepts, links to deep‑dive pages, and provides an A–Z reference for every public type.

---

## How to Use This Hub

1. **Start with [Concepts](Concepts.md)** for a mental model.
2. ~~Jump to a **Module** page for systems you’ll extend (Interaction, Zones, Views, Data, Visuals, Layout).~~ (Coming Soon)
3. Use the **A–Z Index** for quick lookups on specific components and interfaces.

> **Docs style per entry** (used across this repo):
>
> - **Description:** What this type does and when to use it.
> - **Where it lives:** Under which component/object, or as a static class.
> - **Customizable Attributes:** Serialized fields/inspector knobs.
> - **Hooks:** Key methods, events, extension points.
> - **See Also** Related types or interfaces.
> - **Module:** Which module it belongs to.
> - **Example:** Minimal code or scene steps.

---

## Table of Contents

- [Modules Map](#modules-map)
  <!-- - [Interaction](API/Interaction.md)
  - [Zones & Play](API/ZonesAndPlay.md)
  - [Views & Presentation](API/Views.md)
  - [Data & Deck](API/DataAndDeck.md)
  - [Visuals & Factories](API/Visuals.md)
  - [Layout](API/Layout.md) Coming Soon -->
- [A–Z Type Index](#a–z-type-index)
- [FAQ](#faq)

## Modules Map

### Interaction

_(extend here first for custom User Experience)_

- **Deep doc:** ~~API/Interaction.md~~ (Coming Soon)
- Includes: `InteractionService`, `InteractionController`, `IInteractionPolicy`, `DefaultInteractionPolicy`, `CardInteractionOverride`, `CardHoverable`, `DragCard`, `HoverPolicyRuntime`, `HoverRoutingPolicy`, `IHoverPresenter`/`HoverPresenter`, `HoverAnchor`, `PointerService`/`IPointerSource`.

### Zones & Play

_(define how/where cards can be played)_

- **Deep doc:** ~~API/ZonesAndPlay.md~~ (Coming Soon)
- Includes: `ICardZone`, `IPlayResolver`, `CardZoneTracker`, `PlayerHand`, `OpponentHand`, examples `SingleSlotZone`, `LineBoardZone`.

### Views & Presentation — _(what a card looks like & how it binds data)_

- **Deep doc:** ~~API/Views.md~~ (Coming Soon)
- Includes: `ICardView`, `CardBehaviour`, `Card3DView`, `CardPoseView`, `CardPresenter` (sealed).

### Data & Deck — _(what a card IS & how it’s stored/drawn)_

- **Deep doc:** ~~API/DataAndDeck.md~~ (Coming Soon)
- Includes: `ScriptableCard`, `CardDatabase`, `Deck`, `DeckList`, `IEmptyDeckPolicy`, `HandStateService`.

### Visuals & Factories — _(skins, frames, factories, and loaders)_

- **Deep doc:** ~~API/Visuals.md~~ (Coming Soon)
- Includes: `CardSkin`, `CardColorPalette`, `CardAddons`, `CardVisualFactorySO`, `VisualFactoryLoader`, `CardFrameLibrary`, `CardFrameLibraryLoader`, `CardLibraryLoader`, `LayersLoader`, `CardToolkitLayers`, `ICardViewModel`, `ICardViewBinder`, `IContextualSwitcher`.

### Layout — _(arrangement & fan poses)_

- **Deep doc:** ~~API/Layout.md~~ (Coming Soon)
- Includes: `FanLayout`, `FanPose`, `FanLayoutProfile`.

## A–Z Type Index

> Legend: **[E]** Extend/Implement • **[C]** Configure (Inspector) • **[L]** Learn-only

### Card3DView **[L]**

- **Description:** Applies art/frame textures to the card’s front material and toggles mesh elements (e.g., highlight glow) driven by `CardBehaviour` state.
- **Where it lives:** On the card prefab’s **view root child** (the 3D model/mesh under the card root).
- **Customizable Attributes:** none (driven by `CardPresenter` + `CardBehaviour`).
- **Hooks:**
  - `SetFrameSpriteByCardRarity(Sprite frame)` — set frame/albedo from rarity.
  - `SetArtSprite(Sprite art)` — set the main art.
  - Responds to `CardBehaviour.Refresh()` to enable/disable highlight mesh when card is **Highlighted**.
- **See Also:** [CardBehaviour](#cardbehaviour-l), [CardSkin](#cardskin-c)
- **Module:** Views & Presentation
- **Example:** _N/A (CardBehaviour-driven)_

### CardAddons **[E]**

- **Description:** Extension point on Scriptable Card to attach optional data (e.g., Card Suits, Tokens, etc) that can be accessed by other systems.
- **Where it lives:** As a list on `ScriptableCard`, once you create a class that inherits from `CardAddon` you can add it directly through the inspector on Scriptable Card.
- **Customizable Attributes:** Whatever you define in your subclass.
- **Hooks:** none (should be data-only).
- **See Also:** [ScriptableCard](#scriptablecard-c)
- **Module:** Data & Deck
- **Example:**

```csharp
public class ExampleCardAddon : CardAddon
  {
      public int cardNumber;
      public CardSuit cardSuit;
      public CardType type;

      [Header("Abilities")]
      public string abilityTitle;
      public string abilityDescription;


      [Header("Ability Triggers")]
      public bool inOrder;
      public bool isPassive;
      public bool isQuick;
      public bool isEndOfRound;

      public AudioClip playSound;
      public AnimationClip playAnimation;
  }
```

### CardBehaviour **[L]**

- **Description:** The card’s runtime identity and presentation state holder (scriptable data, instance id, pose: face/tap/hover/highlight). Triggers downstream view updates.
- **Where it lives:** On the card prefab root on every spawned card.
- **Customizable Attributes:** none (state set via methods)
- **Hooks:**
  - `Initialize(ScriptableCard definition, int instanceId)` — sets identity, registers in `SceneCardLookup`.
  - `SetFaceUp(bool)`, `SetTapped(bool)`, `SetHighlighted(bool)`, `SetHoverScale(float)` — update pose state.
  - `Refresh()` — push current state to all `ICardView` on the prefab.
  - Events: `OnHoverEnter(HoverEventPayload)`, `OnHoverExit(HoverEventPayload)`, `OnClicked()`
- **See Also:** [ICardView](#icardview-e), [CardPresenter](#cardpresenter-l), [Card3DView](#card3dview-l)
- **Module:** Views & Presentation
- **Example:** Typical usage

```csharp
card.GetComponent<CardBehaviour>().SetHighlighted(true);
card.GetComponent<CardBehaviour>().Tap(true);
```

### CardColorPalette **[C]**

- **Description:** Color palettes for card specific displays
- **Customizable Attributes:**
  - `ColorsList` (name + color)
- **Hooks:**
  - `TryGet(string key, out Color color)` - retrieve color by name
- **See Also:** [CardSkin](#cardskin-c), [ICardViewBinder](#icardviewbinder-e)
- **Module:** Visuals & Factories
- **Example:** Example in an ICardViewBinder

```csharp
if (cardSkin.colorPalette.TryGet("spaceSuit", out Color myColor))
{
    suitIcon.color = myColor;
}
```

### CardDatabase **[C]**

- **Description:** Central registry of `ScriptableCard` assets used at runtime for lookups by definition guid. Loaded by `CardLibraryLoader`.
- **Where it lives:** ScriptableObject in Assets under `CardHandToolkit/Config` folder; referenced by loaders/factories.
- **Customizable Attributes:**
  - `Cards` — List<ScriptableCard> — assign all card definitions you want available.
- **Hooks:** none (data-only)
- **See Also:** [ScriptableCard](#scriptablecard-c), [CardLibraryLoader](#cardlibraryloader-c)
- **Module:** Data & Deck
- **Example:**

```csharp
ScriptableCard cardDefinition = CardLibraryLoader.ByGuid[cr.DefinitionGuid];
```

### CardFrameLibrary **[C]**

- **Description:** Holds frame sprites/materials per rarity or theme so `Card3DView` can apply the right look.
- **Where it lives:** ScriptableObject in Assets under `CardHandToolkit/Config` folder; referenced by loaders/factories.
- **Customizable Attributes:**
  - Sprites per rarity (common, rare, epic, and default)
- **Hooks**:
  - `Get(CardRarity rarity)` — retrieve sprite by rarity enum.
- **See Also:** [Card3DView](#card3dview-l), [CardFrameLibraryLoader](#cardframelibraryloader-c), [CardSkin](#cardskin-c)
- **Module:** Visuals & Factories
- **Example:** Generally used by `Card3DView` internally.

```csharp
Sprite frame = CardFrameLibraryLoader.Instance.FrameLibrary.Get(cardRarity);
```

### CardFrameLibraryLoader **[C]**

_(Operational)_

- **Description:** _TODO_
- **Where it lives:** _Scene singleton on boostrap prefab; auto-added by Setup Wizard._
- **Customizable Attributes:**
  - `FrameLibrary` reference to `CardFrameLibrary` asset.
- **Hooks:** none (data-only)
- **See Also:** [CardFrameLibrary](#cardframelibrary-c), [Card3DView](#card3dview-l)
- **Module:** Visuals & Factories
- **Example:**

```csharp
Sprite frame = CardFrameLibraryLoader.Instance.FrameLibrary.Get(cardRarity);
```

### CardHoverable **[C]**

- **Description:** Enables hover detection with single-card arbitration; in-hand enlarges the card, in-zones routes to overlay canvas via hover policies.
- **Where it lives:** On the card prefab root on every spawned card.
- **Customizable Attributes:**
  - `hoverTimeout` - float - delay before hover exit triggers. (mouse only)
  - `mouseOutOfHandHoverDelay` - float - For mouse, how long to delay before triggering hover (Only for cards not in hand)
  - `longPressDelay` - float - For touch, how long to hold before triggering hover (Only for cards not in hand)
  - `longPressMoveThreshold` - float - For touch, how far finger can move before cancelling long press (Only for cards not in hand)
- **Hooks:** none (all internal state that communicates with `CardBehaviour`)
- **See Also:** [CardBehaviour](#cardbehaviour-l), [HoverPresenter](#hoverpresenter-ihoverpresenter-c-e)
- **Module:** Interaction
- **Example:** In CardBehaviour Apply hover. (Generally all internal, you wouldn't need to call this yourself)

```csharp
SetFlag(CardStateFlags.HoveredInhand, hoverCtx == HoverContext.InHand && isHovered);
```

### CardLibraryLoader **[C]**

_(Operational)_

- **Description:** Runtime gateway for resolving `ScriptableCard` by definition guid or slug. Used by the build pipeline to fetch card data.
- **Where it lives:** Scene-level singleton component in your bootstrap/initial scene (typically added by the Setup Wizard).
- **Customizable Attributes:**
  - `database` — CardDatabase — asset that lists all ScriptableCard definitions.
- **Hooks:** none (access via `Instance.ByGuid` or `Instance.BySlug`)
- **See Also:** [CardDatabase](#carddatabase-c), [VisualFactoryLoader](#visualfactoryloader-c)
- **Module:** Data & Deck
- **Example:**

```csharp
ScriptableCard cardDefinition = CardLibraryLoader.Instance.ByGuid[definitionGuid];
```

### CardPoseView **[C]**

- **Description:** View-layer Position controller; applies face up/down, tapped rotation, and in-hand hover scaling when `CardBehaviour.Refresh` is called.
- **Where it lives:** On the card prefab’s view root child (the UI/mesh object representing the card’s face).
- **Customizable Attributes:**
  - `hoverScale` — float — default 2.0 — in-hand enlarge factor on hover.
  - `hoverAnimTime` — float — default 2f — smoothing time for scale.
  - `hoverZ` — float — default -1.01f — Z offset when hovering in hand to avoid z-fighting. (towards camera)
- **Hooks:**
  - `Initialize(CardStateFlags initialState)` — set initial pose state. (no animation)
  - Reacts to CardBehaviour.Refresh to apply current pose.
- **See Also:** [CardBehaviour](#cardbehaviour-l), [Card3DView](#card3dview-l), [CardPresenter](#cardpresenter-l)
- **Module:** Views & Presentation
- **Example:**

```csharp
cardPoseView.Initialize(cardBehaviour.visualState);
```

### CardPresenter **[L]**

_(sealed)_

- **Description:** Instantiates the World View Prefab, binds the ICardViewBinder to the ICardViewModel, toggles fields using CardSkin rules, then triggers a refresh cascade. Does not own card state.
- **Where it lives:** On the card root of every spawned card.
- **Customizable Attributes:** most initialized programmatically.
  - `ViewAnchor` - Transform - optional override for where to parent the world view prefab (defaults to the card's world canvas).
- **Hooks:**
  - `Initialize(CardSkin skin, ICardViewModel vm)` — set up world view and binding.
  - `Refresh()` — re-run toggle/bind visibility based on skin and vm.
- **See Also:** [CardBehaviour](#cardbehaviour-l), [ICardViewBinder](#icardviewbinder-e), [ICardViewModel](#icardviewmodel-e)
- **Module:** Views & Presentation
- **Example:** Usually done in CardVisualFactorySO

```csharp
cardPresenter.Initialize(cardSkin, cardViewModel);
```

### CardSkin **[C]**

- **Description:** Scriptable object with fields for visual configuration for how a card’s data is presented: assigns world view prefab, optional overlay view, display mode, and palette and frame resources, plus field toggle rules evaluated during presenter refresh.
- **Where it lives:** ScriptableObject under `CardHandToolkit/Config`; referenced by `CardVisualFactorySO`.
- **Customizable Attributes:**
  - `WorldViewPrefab` — GameObject — Used to display card data in world space.
  - `OverlayViewPrefabOptional` — GameObject (optional) — alternate view for HoverPresenter in zones. (Defaults to WorldViewPrefab if not set)
  - `ColorPalette` — CardColorPalette — shared colors for labels/accents.
  - `EnabledFields` — list — list of default fields to show or hide for your cards. (If it isn't set in this list, it won't be shown.)
  - `EnabledCustomFields` - list<string> - list of custom field keys to show or hide for your cards. (If it isn't set in this list, it won't be shown.)
  - `DisplayMode` — Modular or Single Sprite - Modular uses individual UI elements for title, description, etc. Single Sprite uses one image for the entire card face.
- **Hooks:** none (data-only)
- **See Also:** [CardPresenter](#cardpresenter-l), [CardColorPalette](#cardcolorpalette-c), [CardFrameLibrary](#cardframelibrary-c)
- **Module:** Visuals & Factories
- **Example:**

```csharp
IHoverPresenter.ToggleAllFieldsBasedOnData(currentSkin, currentVM, visibleCustom, togglesCache);
```

### CardToolkitLayers **[L]**

_(Constants)_

- **Description:** Canonical layer and tag names/IDs required by the toolkit (e.g., Cards layer). Used by gizmos and raycasts.
- **Where it lives:** Static constants class. Values created/validated by Setup Wizard and LayersLoader.
- **Customizable Attributes:**
  - `cards Layer` — LayerMask — layer for card objects.
- **Hooks:** none
- **See Also:** [LayersLoader](#layersloader-c), [PointerService](#pointerservice--ipointersource-le)
- **Module:** Operational
- **Example:**

```csharp
bool hitSelf = Physics.Raycast(ray, out var hit, Mathf.Infinity, LayersLoader.Instance.Layers.cardsLayerMask)
```

### CardVisualFactorySO **[C]**

- **Description:** Scriptable factory that builds a card GameObject from a CardRef, creates the ICardViewModel, and initializes CardPresenter using the global CardSkin.
- **Where it lives:** ScriptableObject under `CardHandToolkit/Config`; referenced by VisualFactoryLoader.
- **Customizable Attributes:**
  - `CardPrefab` — GameObject — base prefab to instantiate for each card.
  - `ViewModelFactory` — ScriptableObject or reference providing ICardViewModel creation.
  - `GlobalCardSkin` — CardSkin — default skin at build time.
- **Hooks:**
  - `GameObject Build(CardRef cardRef, Transform drawOrigin)` — instantiate prefab, initialize CardBehaviour, create ViewModel, run presenter.
- **See Also:** [VisualFactoryLoader](#visualfactoryloader-c), [CardPresenter](#cardpresenter-l), [ICardViewModel](#icardviewmodel-e)
- **Module:** Visuals & Factories
- **Example:**

```csharp
CardBehaviour cb = VisualFactoryLoader.Instance.VisualFactory.Build(cardRef, drawOrigin);
```

### CardZoneTracker **[E]**

- **Description:** Tracks which ICardZone currently owns the card and raises change events; used to toggle interaction components and drive layout updates.
- **Where it lives:** Component on the card root.
- **Customizable Attributes:**
  - `HomeAssignmentMode` - Defines how the HomeZone is assigned. Options are:
    - `ExplicitOnly` - Only when manually assigned.
    - `FirstZoneEncountered` - The first zone the card enters is assigned as the home zone.
    - `FirstHandEncountered` - The first hand zone the card enters is assigned as the home zone. (Default)
- **Hooks:**
  - `SetHomeZone(ICardZone zone)` — manually assign home zone.
  - `ICardZone CurrentZone` — get active zone reference.
  - `ICardZone HomeZone` — get the card’s home zone (if assigned).
  - `Action<ICardZone, ICardZone> OnZoneChanged` — raised after ownership changes.
  - `RebindZone()` - Re-evaluates and rebinds the current zone based on the card's transform parent. (private method, so just informational)
- **See Also:** [ICardZone](#icardzone-e), [IPlayResolver](#iplayresolver-e), [InteractionService](#interactionservice-e)
- **Module:** Zones & Play
- **Example:** On CardHoverable

```csharp
ICardZone CurrentZone => zoneTracker.CurrentZone;

if (CurrentZone == null || CurrentZone.IsTransitioning || !CurrentZone.IsFocused)
{
  //Custom Logic
}
```

### Deck **[E]**

- **Description:** Runtime container and API for drawing, shuffling, and peeking card references; used by `HandStateService` to produce CardRef values.
- **Where it lives:** Utility class in runtime (non-MonoBehaviour); typically constructed by your game bootstrap or test harness.
- **Customizable Attributes:** N/A
- **Hooks:**
  - `Deck(IEnumerable<T> seedCards, IEmptyDeckPolicy emptyPolicy)` — constructor with initial card list and empty deck policy. Defaults to `ReshuffleOnEmpty`.
  - `Count`, `DiscardCount` — deck size queries
  - `EmptyEvent` — event raised when deck runs out of cards, and the ReshufflePolicy doesn't allow a reshuffle.
  - `Shuffle()`, `Reshuffle()` — randomize current deck or shuffle discard pile back in.
  - `ShuffleRange(int startIndex, int count)` — partial shuffle (IE top 3 cards, bottom 4).
  - `Discard(T id)` — Add card to discard pile
  - `Draw(int count = 1)`, `Peek(int count = 1)` — retrieve next card(s)
  - `InsertTop(T id)`, `InsertBottom(T id)` — programmatic injections
- **See Also:** [DeckList](#decklist-c), [IEmptyDeckPolicy](#iemptydeckpolicy-e)
- **Module:** Data & Deck
- **Example:**

```csharp
Deck deck = deckList.ToGuidDeck(new ReshuffleOnEmpty<string>());
deck.Shuffle();

string cardGuid = deck.Draw().FirstOrDefault();
```

### DeckList **[C]**

- **Description:** Serialized list that defines a deck’s contents for initialization (Card definitions and counts). Used to seed a `Deck`. Useful for game modes with fixed decks.
- **Where it lives:** ScriptableObject asset under `CardHandToolkit/Config` folder; typically created per deck archetype.
- **Customizable Attributes:**
  - `cards` - List<ScriptableCard> - assign card definitions, if a deck has duplicates add multiple entries.
- **Hooks:**
  - `ToGuidDeck(IEmptyDeckPolicy policy)` — produce a runtime `Deck` with card Guids
  - `ToSlugDeck(IEmptyDeckPolicy policy)` — produce a runtime `Deck` with card slugs
- **See Also:** [CardDatabase](#carddatabase-c), [Deck](#deck-e)
- **Module:** Data & Deck
- **Example:**

```csharp
Deck deck = deckList.ToGuidDeck(new ReshuffleOnEmpty<string>());
```

### DefaultInteractionPolicy **[L]**

- **Description:** Built-in rule set that enables or disables interactions based on the card’s current zone and context.
- **Where it lives:** Component or Scriptable configuration referenced by `InteractionService`. (You can assign an override policy on `InteractionService` or per card using `CardInteractionOverride`.)
- **Customizable Attributes:** none (logic-only)
- **Hooks:**
  - `GetPermissions(CardBehaviour card, InteractionContext ctx)` — returns allowed interactions for the card in its current zone.
- **See Also:** [IInteractionPolicy](#iinteractionpolicy-e), [InteractionService](#interactionservice-e), [InteractionController](#interactioncontroller-e)
- **Module:** Interaction
- **Example:** Interaction Controller on Card Recomputes every zone change

```csharp
// Resolve policy: Per-card > ZoneOverride > Global
var zonePolicy = zoneTracker.CurrentZone.GetComponent<IInteractionPolicyProvider>()?.Policy;
var global = InteractionService.Instance.Policy; // holds default policy instance
InteractionPerms p = (zonePolicy ?? global).GetPermissions(card, ctx);

// Per-card masks
if (TryGetComponent<IInteractionOverride>(out var o))
{
  p = (p | o.AllowMask) & ~o.DenyMask;
}
```

### DragCard **[C]**

- **Description:** Handles pointer-driven dragging while in hand, updates highlight when crossing play height, and emits drop signals for play resolution.
- **Where it lives:** Component on the card root; enabled when the card is in a zone that allows dragging (e.g., local hand).
- **Customizable Attributes:**
  - `dragThreshold` - float - minimum pointer movement to start drag (pixels); defaults to 100.
  - `tiltStrength` - float - how much the card tilts based on pointer movement; defaults to 15.
  - `lerpSpeed` - float - how quickly the card follows the pointer; defaults to 10.
  - `dragPlaneZ` - float - Z position of the drag plane in world space; defaults to -5 (Have it closer to the camera than the hand and board).
- **Hooks:**
  - `static ActiveDragCount` - int - number of cards currently being dragged in the scene.
  - `isDragging` - bool - whether this card is currently being dragged.
  - Events: `OnAnyDragStart`, `OnAnyDragEnd`
- **See Also:** [PointerService](#pointerservice--ipointersource-le), [IPlayResolver](#iplayresolver-e), [PlayerHand](#playerhand-c)
- **Module:** Interaction
- **Example:** Usually internal, but you can check if a card is being dragged

### FanLayout **[C]**

- **Description:** Positions cards in a curved fan for hands; `FanLayout` computes transforms for each index.
- **Where it lives:** `FanLayout` is a logic class that uses data defined in the `FanLayoutProfile` ScriptableObject. `PlayerHand` and `OpponentHand` use `FanLayout` internally to arrange their child cards.
- **Customizable Attributes:** none (static class logic-only)
- **Hooks:**
  - `ComputeFromProfile(int count, float handSize, FanLayoutProfile profile, FanSeatModifiers mods)` — returns a list of `FanPose` for the given count, hand size, profile, and seat modifiers.
  - `FindCurvePositionForX(float xTarget, float effectiveHandSize, FanLayoutProfile profile, int iters = 18)` - helper to find a position on the fan curve by X coordinate. (Used for reordering cards in hand by dragging)
- **See Also:** [PlayerHand](#playerhand-c), [OpponentHand](#opponenthand-c), [FanLayoutProfile](#fanlayoutprofile-c)
- **Module:** Layout
- **Example:** Opponent Hand excerpt

```csharp
var poses = FanLayout.ComputeFromProfile(count, handSize, fanLayout, seatModifier);

for (int i = 0; i < count; i++)
{
    _targetsPos[i] = poses[i].LocalPosition;
    _targetsRot[i] = poses[i].LocalRotation;
}
```

### FanLayoutProfile **[C]**

- **Description:** Scriptable profile that defines the curve and spacing of card hands; consumed by `FanLayout` to compute `FanPose` for each slot.
- **Where it lives:** ScriptableObject under `CardHandToolkit\Runtime\ScriptableObjects`; referenced by `PlayerHand`/`OpponentHand` via `FanLayout`. You can **create and assign** your own to PlayerHand/OpponentHand components.
- **Customizable Attributes:**
  - `xCurve` — AnimationCurve — Produces the X position along the fan based on normalized index (0 to 1).
  - `yCurve` — AnimationCurve — Evaluates 0→1 to lift the cards vertically. Default is a gentle ⌒ arc.
  - `zCurve` — AnimationCurve — Evaluates 0→1 to push cards back along Z. Default is cards on the right are above those on the left.
  - `rotCurve` — AnimationCurve — Evaluates 0→1 to rotate cards about the center (A Fan Angle)
  - `maxHandSpread` — float — Maximum count of cards before clamping spread. (Additional cards will squish closer together)
  - `heightScale` — float — Multiplier for vertical lift curve.
  - `zAmplitude` — float — Multiplier for Z curve (Defaults to 1, just following the curve).
  - `rotDegreesScale` — float — Multiplier for rotation curve (Defaults to 15).
- **Hooks:** none (data-only)
- **See Also:** [FanLayout](#fanlayout-c), [PlayerHand](#playerhand-c), [OpponentHand](#opponenthand-c)
- **Module:** Layout
- **Example:** Data Only, default found in `CardHandToolkit/Runtime/ScriptableObjects/FanLayoutProfile_Default.asset`

### HandStateService **[L]**

- **Description:** Static service that owns per-seat hand membership and emits hand events used by `PlayerHand`/`OpponentHand` and resolvers.
- **Where it lives:** Static class; accessible from runtime systems.
- **Customizable Attributes:** none
- **Hooks:**
  - `AddCard(int seat, string definitionGuid)`— adds to seat hand; raises `OnCardAdded`.
  - `AddExisting(int seat, string instanceId, string definitionGuid)` — reuses instance id if already tracked.
  - `RemoveCard(int seat, string instanceId)` — removes and raises `OnCardRemoved`.
  - `GetHand(int seat)` — returns current CardRefs for a seat.
  - Events: `OnCardAdded`, `OnCardRemoved`.
- **See Also:** [SceneCardLookup](#scenecardlookup-l), [PlayerHand](#playerhand-c), [OpponentHand](#opponenthand-c)
- **Module:** Data & Deck
- **Example:**

```csharp
HandStateService.Instance.AddCard(playerSeat, cardDefinitionGuid);
```

### HandViewBase **[L]**

- **Description:** Shared base for `PlayerHand` and `OpponentHand` providing common seat and draw behavior.
- **Where it lives:** Component on the hand GameObject; `PlayerHand` and `OpponentHand` derive from it.
- **Customizable Attributes:**
  - `seatId` — int — which player seat this hand belongs to.
  - `drawOrigin` - Transform — optional spawn point for new cards.
- **Hooks:**
  - `SeatId` — get seat.
  - `IsLocalPlayer` — true if seat matches local player.
- **See Also:** [PlayerHand](#playerhand-c), [OpponentHand](#opponenthand-c)
- **Module:** Layout
- **Example:**

```csharp
if (GetComponent<PlayerHand>().IsLocalPlayer)
{
    // Custom logic for local player hand
}
```

### HoverAnchor **[C]**

- **Description:** Mouse-hover anchor line for in-hand previews on PC. Position it so the base of the enlarged card rests on this line; keep the gizmo just below the screen bottom to prevent hover flicker. Touch uses a different hover path.
- **Where it lives:** On the **Focus Position** transform of the **PlayerHand** prefab (overlay systems may reference it).
- **Customizable Attributes:**
  - `referenceCard` — GameObject — generic card prefab root used to calculate the card base offset. (If your card prefab variant changes the base collider size, update this reference.)
- **Hooks:**
  - `GetCardYOffset()` — returns half the card view collider height in world units (accounts for scaling) for positioning the anchor line.
- **See Also:** [IZoneHoverAnchorProvider](#izonehoveranchorprovider-e), [HoverPoseSolver](#hoverposesolver-c)
- **Module:** Interaction
- **Example:** Usage in HoverPoseSolver

```csharp
float anchorWorldY = anchorInfo.Anchor.position.y;

// Where should the card's world Y be so its bottom sits on the anchor line?
float desiredWorldY = anchorWorldY + anchorInfo.MarginY + bottomOffsetWorld;
```

### HoverPolicyRuntime **[C]**

- **Description:** Listens to `CardBehaviour` hover events (via a binder) and decides how to present them (only for out of hand hovers). In-hand hovers are ignored; out-of-hand hovers are routed to an `IHoverPresenter` with simple arbitration so only one overlay is shown.
- **Where it lives:** Scene-level controller (singleton at runtime). Lives on the bootstrap prefab created by the Setup Wizard.
- **Customizable Attributes:**
  - `presenterPrefab` — HoverPresenter — overlay presenter to instantiate if one is not found.
  - `routingPolicy` — HoverRoutingPolicy — rules for when and how to route to overlay.
- **Hooks:**
  - Subscribes to CardBehaviour OnHoverEnter/Exit (via HoverEventBinder).
  - `HandleHoverEnter(HoverEventPayload payload)` — process new hover request. (Show/Hide presenter as needed)
  - `HandleHoverExit(HoverEventPayload payload)` — process hover exit. (Hide presenter if needed)
- **See Also:** [HoverPresenter](#hoverpresenter-c), [HoverAnchor](#hoveranchor-c), [CardHoverable](#cardhoverable-c)
- **Module:** Interaction
- **Example:** All Internal, shouldn't need to call directly.

### HoverPoseSolver **[L]**

(static class)

- **Description:** Math helper used with a hover anchor to place the **bottom** of a hovered card exactly on the anchor line (PC mouse). Converts anchor space, card footprint, and parent transform into a target local Y.
- **Where it lives:** Static utility class; called by PlayerHand when positioning in-hand hovers.
- **Customizable Attributes:** none
- **Hooks:**
  - `TryBottomToAnchorY(CardBehaviour card, IZoneHoverAnchorProvider zoneAnchor, ICardFootprint footprint, out float targetLocalY)` — sets targetLocalY so the card’s bottom rests on the anchor; returns false if data is missing (caller should fall back).
- **See Also:** [HoverAnchor](#hoveranchor-c), [PlayerHand](#playerhand-c)
- **Module:** Interaction
- **Example:**

```csharp
if (HoverPoseSolver.TryBottomToAnchorY(child, this, footprint, out float targetLocalY))
    pose.LocalPosition.y = targetLocalY;
else
    pose.LocalPosition.y += touchHoverHeight;

```

### HoverPresenter **[C]**

- **Description:** Renders the out-of-hand hover overlay card. The concrete `HoverPresenter` controls a world-space or screen-space panel, an optional backdrop, and positions the preview near the pointer (mouse) or fixed on the Left (touch).
- **Where it lives:** On a UI Canvas (screen-space overlay recommended for clarity); referenced by `HoverPolicyRuntime`.
- **Customizable Attributes:**
  - `cardPanelRoot` — RectTransform — where the card view prefab is attached.
  - `backdrop` — Image — optional dimming layer behind the card.
  - `mouseOffset` — How far the card is offset from the pointer (mouse).
  - `touchMargin` - How far from the screen edge to place the card (touch).
- **Hooks:**

  - `ShowPreview(CardBehaviour card, bool isMouse)` — instantiate and show the overlay card. Pulls the cards viewModel from the CardPresenter component.
  - `HidePreview()` — remove the overlay card.
  - `PositionNearCardMouse(CardPresenter)` — place near pointer with clamping.
  - `PositionFixedLeftTouch()` — place at a fixed left-side slot.
  - `Refresh()` - reapply cardSkin and view model, will toggle fields based on current data. (private method, called on ViewModel changes)

- **See Also:** [HoverPolicyRuntime](#hoverpolicyruntime-c), [CardSkin](#cardskin-c)
- **Module:** Interaction
- **Example:**

### HoverRoutingPolicy **[C]**

- **Description:** Scriptable rules for hover overlay visibility and arbitration. Used by `HoverPolicyRuntime` to decide if and when to show an out-of-hand preview.
- **Where it lives:** ScriptableObject asset (create via Tools ▸ Card Toolkit ▸ HoverRoutingPolicy). Assign it on `HoverPolicyRuntime`.
- **Customizable Attributes:**
  - `exclusivePreview` — bool — only one overlay visible at a time.
  - `suppressWhenPointerOverUI` — bool — skip overlay if pointer "hover" is over UI.
  - `allowFaceDownPreview` — bool — allow previews for face-down cards. If false, face-down cards never route to overlay.
  - `suppressOutOfHandWhenInHand` — bool — if true, in-hand hovers never route to overlay. (Default true)
  - `lastEnterWins` — bool — on conflicts, newest hovered card takes ownership.
- **Hooks:** none (policy holds data; `HoverPolicyRuntime` enforces it)
- **See Also:** [HoverPolicyRuntime](#hoverpolicyruntime-c), [HoverPresenter](#hoverpresenter-c)
- **Module:** Interaction
- **Example:** Data Only - create via Tools ▸ Card Toolkit ▸ HoverRoutingPolicy

### ICardView **[E]**

- **Description:** View interface implemented by components on the card’s view root to receive art/frame sprites and react to pose updates.
- **Where it lives:** On the card prefab’s view root child (e.g., `Card3DView`, `CardPoseView` implement this).
- **Customizable Attributes:** none
- **Hooks:**
  - `SetFrameSprite(Sprite frame)` — set a specific frame sprite.
  - `SetFrameSpriteByCardRarity(CardRarity rarity)` — convenience for rarity-based frames.
  - `SetArtSprite(Sprite art)` — set the main art sprite.
  - `Refresh(CardStateFlags state)` — update visuals from the current card state.
- **See Also:** [CardBehaviour](#cardbehaviour-l), [Card3DView](#card3dview-l), [CardPoseView](#cardposeview-c)
- **Module:** Views & Presentation
- **Example:** CardPoseView excerpt

```csharp
public void Refresh(CardStateFlags f)
{
    bool faceDn = f.HasFlag(CardStateFlags.FaceDown);
    bool tapped = f.HasFlag(CardStateFlags.Tapped);

    /* ----- rotation flags (flip + tap) ----- */
    lerpT = 0f;

    float yRot = faceDn ? 180 : 0;
    float zRot = tapped ? 45 : 0;
    _targetRot = Quaternion.Euler(0, yRot, zRot);
}
```

### ICardViewBinder **[E]**

- **Description:** Binds a WorldViewPrefab (labels, icons, images) to an `ICardViewModel`. On bind, subscribe to VM changes and update the view.
- **Where it lives:** Component on your **WorldViewPrefabVariant** (user-owned).
- **Customizable Attributes:** none (bind references to text/images via serialized fields in your implementation)
- **Hooks:**
  - `Bind(ICardViewModel viewModel)` — subscribe to `viewModel.OnChanged` and apply initial values
- **See Also:** [ICardViewModel](#icardviewmodel-e), [CardPresenter](#cardpresenter-l), [HoverPresenter](#hoverpresenter-c)
- **Module:** Views & Presentation
- **Example:** CardPresenter excerpt

```csharp
GameObject prefab = skin.GetWorldViewPrefab();
viewInstance = Instantiate(prefab, viewAnchor != null ? viewAnchor : transform);

viewBinder = viewInstance.GetComponentInChildren<ICardViewBinder>();
viewBinder?.Bind(vm);
```

### ICardViewModel **[E]**

- **Description:** View-model for a card. Emits `OnChanged` when display-relevant state changes and exposes basic fields plus capability keys for conditional UI.
- **Where it lives:** Runtime class created by your factory (e.g., via `CardVisualFactorySO`); not a MonoBehaviour.
- **Customizable Attributes:** none
- **Hooks:**
  - event Action `OnChanged`
  - string `Title`, `RulesText`
  - Sprite `ArtSprite`, `CardFrameSprite`, `FullImageSprite`
  - int? `Cost`, `Attack`, `Health`
  - IReadOnlyCollection<string> `Capabilities` — presence-only keys for field toggles
  - void `Initialize`(ScriptableCard card) - populate from definition
  - bool `SetCapability`(string key, bool show) - add/remove capability key; returns true if changed.
- **See Also:** [ICardViewBinder](#icardviewbinder-e), [CardPresenter](#cardpresenter-l), [HoverPresenter](#hoverpresenter-c), [CardVisualFactorySO](#cardvisualfactoryso-c)
- **Module:** Views & Presentation
- **Example:** Exerpt from SampleCardViewModel

```csharp
public void Initialize(ScriptableCard card)
{
  _card = card ?? throw new ArgumentNullException(nameof(card));
  _state = new SampleRuntimeState();
  RecomputeAll();
}

public void RecomputeAll()
{
  ExampleCardAddon addon = _card.GetAddon<ExampleCardAddon>(_card);
  Title = _card.cardName;
  AbilityTitle = addon.abilityTitle;
  //...
  _caps.Clear();
  // If the card has an ability title, show the ability title field. (Ability Title is a custom field)
  SetCapability("abilityTitle", !string.IsNullOrEmpty(AbilityTitle));
  OnChanged?.Invoke();
}
```

### ICardZone **[E]**

- **Description:** Contract for any area that owns cards and can align them (hands, boards, piles).
- **Where it lives:** Implemented by zone components (e.g., PlayerHand, OpponentHand, board zones).
- **Customizable Attributes:** none (zones expose their own inspector fields separately)
- **Hooks:**
  - `Transform ZoneRoot` — parent for card GameObjects (Defaults to the zone’s own transform)
  - `ZoneType Type` — Hand, Board, Deck, Discard, OpponentHand, etc.
  - `int SeatId` — -1 for neutral zones, such as boards; 0+ for player-specific zones.
  - `bool IsFocused` — true when actively interacted with (If False, can't hover cards - Board Zones default to true always)
  - `bool IsTransitioning` — true during animated moves (used to block interaction)
  - `bool AddCard(CardBehaviour card, int insertAt = -1)` - Reparents and adds a card to the zone at the specified index (or end if -1).
  - `bool RemoveCard(CardBehaviour card)` - Does not reparent; just removes from internal tracking.
  - `void AlignCards(Pointer currentPtr = default, bool hasPointer = false)` - align cards in the zone.
  - `bool ScreenPointToLocal(Vector2 screenPoint, Camera cam, out Vector3 local)` - convert screen point to local zone space; returns false if outside zone bounds. (From `CardZoneExtensions`)
  - `TryProjectScreenPoint(Vector2 screenPos, Camera cam, out Vector3 world)` - convert screen point to world space on the zone's plane; returns false if raycast misses. (From `CardZoneExtensions`)
- **See Also:** [PlayerHand](#iplayerhand-e), [IReorderBandZone](#ireorderbandzone-e), [IPlayResolver](#iplayresolver-e)
- **Module:** Zones & Play
- **Example:** Line Board Zone Add and Remove

```csharp
public bool AddCard(CardBehaviour card, int index = -1)
{
    _cards.Insert(index < 0 ? _cards.Count : Mathf.Clamp(index, 0, _cards.Count), card);
    card.transform.SetParent(ZoneRoot, false);

    AlignCards();
    return true;
}

public bool RemoveCard(CardBehaviour card)
{
    var removed = _cards.Remove(card);
    if (removed) AlignCards();
    return removed;
}
```

### IContextualSwitcher **[E]**

- **Description:** Per-field strategy that adjusts UI for **world** vs **overlay** contexts. Lets the same element render differently when shown in-hand vs shown in the hover overlay.
- **Where it lives:** Components on your **WorldViewPrefab** (and its overlay variant if used). `CardPresenter` and `HoverPresenter` discover and call them.
- **Customizable Attributes:** your field references or numeric limits (e.g., min/max font size)
- **Hooks:**
  - `ApplyWorld(ICardViewModel vm)` — apply world-view settings
  - `ApplyOverlay(ICardViewModel vm)` — apply overlay-view settings
- **See Also:** [HoverPresenter](#hoverpresenter-c), [CardPresenter](#cardpresenter-l)
- **Module:** Views & Presentation
- **Example:** DescriptionField excerpt from sample

```csharp
public void ApplyWorld() => Apply(worldMin, worldMax);
public void ApplyOverlay() => Apply(overlayMin, overlayMax);

void Apply(float min, float max)
{
  if (!t) return; t.enableAutoSizing = true;
  t.fontSizeMin = min; t.fontSizeMax = max;
  t.overflowMode = TextOverflowModes.Overflow;
}
```

### IEmptyDeckPolicy<TId> **[E]**

- **Description:** Strategy interface invoked when a `Deck<TId>` is empty during `Draw`. Return true to allow the draw operation to continue (e.g., after a reshuffle), or false to signal emptiness (the deck will fire `EmptyEvent`).
- **Where it lives:** Runtime interface implemented by policies such as `ReshuffleOnEmpty<TId>` and `NoReshufflePolicy<TId>`. Passed into or owned by the deck pipeline.
- **Customizable Attributes:** none
- **Hooks:**
  - `bool TryResolveEmpty(Deck<TId> deck)` — attempt to resolve an empty-deck state; return true if the caller should retry Draw.
- **See Also:** [Deck](#deck-e), [DeckList](#decklist-c)
- **Module:** Data & Deck
- **Example:** TryResolveEmpty excerpt from ReshuffleOnEmpty

```csharp
public bool TryResolveEmpty(Deck<TId> deck)
{
  if (deck.DiscardCount > 0)
  {
    deck.Reshuffle();
    return true;
  }

  // If no cards in discard, reshuffle does nothing
  return false;
}
```

### CardInteractionOverride **[E]**

- **Description:** Per-card override for interaction permissions. Applied after zone policy and global policy, it can explicitly allow or deny specific actions on a single card.
- **Where it lives:** Component on the **card root** (add on prefab or at runtime).
- **Customizable Attributes:**
  - `allow` — InteractionPerms — bitmask to force-allow.
  - `deny` — InteractionPerms — bitmask to force-deny.
- **Hooks:**
  - `InteractionPerms AllowMask` — get allow overrides.
  - `InteractionPerms DenyMask` — get deny overrides.
- **See Also:** [IInteractionPolicy](#iinteractionpolicy-e), [InteractionController](#interactioncontroller-c), [InteractionService](#interactionservice-c)
- **Module:** Interaction
- **Example:** Data Only - add component to card prefab or at runtime

### IInteractionPolicy **[E]**

- **Description:** Stateless evaluator that returns the effective permissions for a card in a given context (zone, ownership, phase). Used by controllers and services to decide which interaction components should be enabled.
- **Where it lives:** Runtime interface; implemented by `DefaultInteractionPolicy` or custom policies. Can also be provided by a zone via an `IInteractionPolicyProvider`.
- **Customizable Attributes:** none (policy instances may be ScriptableObjects with their own serialized fields)
- **Hooks:**
  - `InteractionPerms GetPermissions(CardBehaviour card, InteractionContext ctx)` — compute the allowed actions based on cards zone etc.
- **See Also:** [InteractionController](#interactioncontroller-c), [InteractionService](#interactionservice-c), [DefaultInteractionPolicy](#defaultinteractionpolicy-l)
- **Module:** Interaction
- **Example:** Default Example Policy excerpt

```csharp
public InteractionPerms GetPermissions(CardBehaviour card, InteractionContext c)
{
  switch (c.Zone)
  {
    case ZoneType.Hand: // player hand
      return InteractionPerms.Hover | InteractionPerms.Drag | InteractionPerms.Select;
    case ZoneType.OpponentHand: // No interactions allowed
      return InteractionPerms.None;
    case ZoneType.Board:
      return InteractionPerms.Hover | InteractionPerms.Select | InteractionPerms.Target;
    default:
      return InteractionPerms.None;
  }
}
```

### ILockableZone **[E]**

- **Description:** Optional capability for zones to temporarily disable interactions and membership changes. Useful during animations, AI turns, or atomic updates so cards cannot be added, removed, or reordered.
- **Where it lives:** Implemented alongside your zone component (e.g., PlayerHand, OpponentHand, board zones) when you need lock/unlock control.
- **Customizable Attributes:** none
- **Hooks:**
  - bool IsLocked — current lock state
  - void Lock() — enter locked state
  - void Unlock() — exit locked state
  - event Action<bool> OnLockChanged — raised after lock state changes
- **See Also:** [ICardZone](#icardzone-e), [IReorderBandZone](#ireorderbandzone-e), [PlayerHand](#playerhand-c)
- **Module:** Zones & Play
- **Example:** Sample Scene locks SingleSlotZone during Reveal Phase

### IPlayHeightZone **[E]**

- **Description:** Zone capability that tells whether a point is above the “play height” threshold used by drag-to-play logic.
- **Where it lives:** Implement on zones that contribute to play-height checks (e.g., PlayerHand).
- **Customizable Attributes:** none
- **Hooks:**
  - `bool IsAbovePlayHeight(Vector3 localPosInZone)`
- **See Also:** [DragCard](#dragcard-c), [IPlayResolver](#iplayresolver-e)
- **Module:** Zones & Play
- **Example:** Drag Card excerpt

```csharp
IPlayHeightZone PlayHeightZone => CurrentZone as IPlayHeightZone;

bool stayedInHand = hasLocal && ReorderZone.IsLocalPointInReorderBand(local);

if (!stayedInHand && PlayHeightZone != null)
{
  bool play = PlayHeightZone.IsAbovePlayHeight(localPosInZone);
  if (play) PlayCardFromHand();
}
```

### IPlayResolver **[E]**

- **Description:** Scene-level rule module that decides whether a dropped card should enter a specific `ICardZone`. Resolvers are queried in priority order; the first one to accept performs the move.
- **Where it lives:** Component on a GameObject in your gameplay scene (On an `ICardZone`). Multiple resolvers can coexist in the scene.
- **Customizable Attributes:**
  - `Priority` — int — lower numbers checked first.
- **Hooks:**
  - `PlacementResult TryResolvePlay(PlacementQuery query)` — PlacementResult.Allowed if succeeded; resolver removes from HandStateService (if needed) and adds the card to its zone.
- **See Also:** [ICardZone](#icardzone-e), [PlayerHand](#playerhand-c), [InteractionController](#interactioncontroller-c)
- **Module:** Zones & Play
- **Example:** Sample Board Zone excerpt

```csharp
public PlacementResult TryResolvePlay(PlacementQuery query)
{
    PlacementResult result = new PlacementResult
    { allowed = false, slotIndex = null, worldPos = Vector3.zero, worldRot = Quaternion.identity};

    if (IsLocked || _cards.Count >= Capacity)
        return result;

    if (query.fromZone != null)
    {
        if (query.fromZone.Type == ZoneType.Hand || query.fromZone.Type == ZoneType.OpponentHand)
            HandStateService.Instance.RemoveCard(query.fromZone.SeatId, query.card.InstanceId);
        else
        {
            query.fromZone.RemoveCard(query.card);
        }
    }

    AddCard(query.card);
   // Result Update
   // ...
    return result;
}
```

### IPointerSource **[E]**

### InteractionController **[C]**

- **Description:** Per-card controller that evaluates the active interaction policy chain and enables or disables components like `CardHoverable` and `DragCard`. Resolution order: global policy → zone policy → per-card overrides.
- **Where it lives:** Component on the **card root**; requires `CardBehaviour` and `CardZoneTracker`.
- **Customizable Attributes:** none (discovers `DragCard` and `CardHoverable` automatically)
- **Hooks:**
  - `Recompute()` — resolve policies and toggle components when zone or phase changes.
  - Subscribes to `CardZoneTracker.OnZoneChanged` — recompute on zone transitions.
- **See Also:** [IInteractionPolicy](#iinteractionpolicy-e), [CardInteractionOverride](#cardinteractionoverride-e), [CardHoverable](#cardhoverable-c), [DragCard](#dragcard-c)
- **Module:** Interaction
- **Example:** Handles everything internally; no direct calls needed.

### InteractionService **[C]**

- **Description:** Scene-level singleton that provides the **global** interaction policy used when a zone does not supply one. Falls back to `DefaultInteractionPolicy` if no custom asset is assigned.
- **Where it lives:** On the **bootstrap** prefab in your initial scene (`DontDestroyOnLoad` pattern).
- **Customizable Attributes:**
  - `defaultPolicyAsset` — ScriptableObject — optional asset implementing `IInteractionPolicy`.
- **Hooks:**
  - `static InteractionService Instance` — global accessor.
  - `IInteractionPolicy Policy` — active global policy (lazy-initialized to `DefaultInteractionPolicy`).
- **See Also:** [IInteractionPolicy](#iinteractionpolicy-e), [DefaultInteractionPolicy](#defaultinteractionpolicy-c), [InteractionController](#interactioncontroller-c)
- **Module:** Interaction
- **Example:**

```csharp
// holds default policy instance
IInteractionPolicy global = InteractionService.Instance.Policy;
InteractionPerms p = global.GetPermissions(card, ctx);
```

### IReorderBandZone **[E]**

- **Description:** Zone capability for in-hand reordering. Converts screen to local space, tests if the pointer is within a reorder band, and computes insert indices.
- **Where it lives:** Implement on hand-like zones that allow reordering.
- **Customizable Attributes:** none
- **Hooks:**
  - `bool IsLocalPointInReorderBand(Vector3 local)`
  - `int CalcInsertIndexFromLocal(float localX)`
  - `void MoveCardInHand(CardBehaviour card, int insertIndex)`
- **See Also:** [PlayerHand](#playerhand-c), [DragCard](#dragcard-c)
- **Module:** Zones & Play
- **Example:** Excerpt from DragCard

```csharp
if (ReorderZone != null && CurrentZone.ScreenPointToLocal(currentPtr.screenPos, cam, out Vector3 local))
{
  if (ReorderZone.IsLocalPointInReorderBand(local))
  {
    int idx = ReorderZone.CalcInsertIndexFromLocal(local.x);
    // Shift the hand alignment to show space for the dragged card
    ReorderZone.MoveCardInHand(this.GetComponent<CardBehaviour>(), idx);
  }
}
```

### IZoneHoverAnchorProvider **[E]**

- **Description:** Provides a hover anchor for PC mouse previews so a hovered card’s **bottom edge** can rest on a consistent line (used by `HoverPoseSolver`).
- **Where it lives:** Implemented by zones that support anchored hover placement (e.g., PlayerHand). Consumed by `HoverPoseSolver`.
- **Customizable Attributes:** none (zone exposes its own anchor in inspector)
- **Hooks:**
  - `Transform ZoneRoot` — root transform used to resolve local anchor space
  - `bool TryGetHoverAnchor(out AnchorInfo info)` — returns anchor data if available
    - `AnchorInfo.Anchor` — Transform — the anchor transform
    - `AnchorInfo.Space` — enum — World or ZoneLocal
    - `AnchorInfo.MarginY` — float — extra vertical margin above the line
- **See Also:** [HoverPoseSolver](#hoverposesolver-l), [HoverAnchor](#hoveranchor-c), [PlayerHand](#playerhand-c)
- **Module:** Interaction

### LayersLoader **[C]**

_(Operational)_

- **Description:** Ensures required layers and tags (e.g., Cards) exist in the project. Intended to be added by the Setup Wizard; avoids manual project setup mistakes.
- **Where it lives:** Scene-level utility on your bootstrap scene or invoked by the Setup Wizard.
- **Customizable Attributes:** none (operational)
- **Hooks:** none (performs validation/creation at edit/build time)
- **See Also:** [CardToolkitLayers](#cardtoolkitlayers-l), Setup Wizard docs
- **Module:** Operational

### LineBoardZone **[L]**

_(Starter Example)_

- **Description:** Minimal board zone that arranges played cards along a line. Intended as a learning scaffold; replace with a production zone in real games.
- **Where it lives:** Add to a board GameObject; implements `ICardZone`, `IPlayResolver` and `ILockableZone`.
- **Customizable Attributes:**
  - `spacing` — float — distance between cards on the line
  - `capacity` — int — maximum number of cards in the line
  - `priority` — int — resolver priority when multiple zones are valid for play. Lower numbers are higher priority.
  - `maxWidth` — int — maximum number of cards in zone before squishing.
  - `zLiftPerIndex` — float — how much to lift cards along Z as index increases (for slight overlap)
- **Hooks:**
  - Implements `ICardZone: AddCard, RemoveCard, AlignCards, ZoneRoot, Type, SeatId, IsFocused, IsTransitioning`
  - Implements `IPlayResolver: TryResolvePlay, Priority`
- **See Also:** [ICardZone](#icardzone-e), [IPlayResolver](#iplayresolver-e), [SingleSlotZone](#singleslotzone-l)
- **Module:** Zones & Play
- **Example:** Slimmed TryResolvePlay excerpt

```csharp
public PlacementResult TryResolvePlay(PlacementQuery query)
{
    PlacementResult result = new PlacementResult
    { allowed = false, slotIndex = null, worldPos = Vector3.zero, worldRot = Quaternion.identity};

    if (IsLocked || _cards.Count >= Capacity)
        return result;

    if (query.fromZone != null)
    {
        if (query.fromZone.Type == ZoneType.Hand || query.fromZone.Type == ZoneType.OpponentHand)
            HandStateService.Instance.RemoveCard(query.fromZone.SeatId, query.card.InstanceId);
        else
        {
            query.fromZone.RemoveCard(query.card);
        }
    }

    AddCard(query.card);
   // Result Update
   // ...
    return result;
}
```

### OpponentHand **[C]**

- **Description:** Hand zone for the opponent. Implements `ICardZone` and uses the same fan layout system; interaction is typically suppressed by the default policy.
- **Where it lives:** Scene object representing the opponent’s hand.
- **Customizable Attributes:**
  - `HandViewBase Attributes:` [see HandViewBase](#handviewbase-l)
  - `fanLayout` — FanLayoutProfile — curve and spacing
  - `seatModifier` — FanSeatModifiers — Spread Modifiers for opponent seat (usually smaller spread)
  - `posLerpSpeed` — float — card position animation speed
  - `rotLerpSpeed` — float — card rotation animation speed
  - `stopPosEpsilon` — float — threshold to consider position "close enough"
  - `stopRotEpsilon` — float — threshold to consider rotation "close enough"
- **Hooks:**
  - `Implements ICardZone: AddCard, RemoveCard, AlignCards, ZoneRoot, Type, SeatId`
  - Listens to `HandStateService OnCardAdded/Removed` to build/re-parent
- **See Also:** [PlayerHand](#playerhand-c), [FanLayout / FanPose](#fanlayout--fanpose-c), [HandStateService](#handstateservice-l)
- **Module:** Zones & Play

### PlayerHand **[C]**

- **Description:** Local player’s hand zone. Listens to `HandStateService` events, spawns visuals via `VisualFactoryLoader`, and lays out cards using `FanLayout`.
- **Where it lives:** Scene object representing the player’s hand; implements `ICardZone`, `IPlayHeightZone`, `IReorderBandZone`, `IZoneHoverAnchorProvider`.
- **Customizable Attributes:**
  - `HandViewBase Attributes:` [see HandViewBase](#handviewbase-l)
  - `fanLayout` — FanLayoutProfile — curve and spacing
  - `seatModifier` — FanSeatModifiers — Spread Modifiers for player seat (Defaults to 1)
  - `touchHoverHeight` — float — how high cards lift when hovered (mobile)
  - `focusPosition` — Transform — where the hand moves when focused. (Scale of this transform is applied to the hand)
  - `unfocusPosition` — Transform — where the hand moves when unfocused (Scale of this transform is applied to the hand)
  - `hoverAnchor` — HoverAnchor — optional anchor for PC hover placement
  - `anchorMarginY` — float — extra margin above anchor line
  - `handTransitionSpeed` — float — speed of hand focus transitions
  - `cam` — Camera — optional override for screen-to-world conversions (defaults to Camera.main)
  - `playHeight` — float — local height above hand to count as "played"
  - `bandHeight` — float — vertical size of reorder band
  - `bandDepth` — float — depth size of reorder band
  - `minBandWidth` — float — minimum width of reorder band (expands with more cards)
- **Hooks:**
  - Implements `ICardZone: AddCard, RemoveCard, AlignCards, ZoneRoot, Type, SeatId`
  - Implements `IPlayHeightZone: IsAbovePlayHeight`
  - Implements `IReorderBandZone: IsLocalPointInReorderBand, CalcInsertIndexFromLocal, MoveCardInHand`
  - Implements `IZoneHoverAnchorProvider: TryGetHoverAnchor, ZoneRoot`
  - Listens to `HandStateService OnCardAdded/Removed` to build/re-parent
- **See Also:** [HandStateService](#handstateservice-l), [VisualFactoryLoader](#visualfactoryloader-c), [FanLayout / FanPose](#fanlayout--fanpose-c)
- **Module:** Zones & Play

### PointerService **[L]**

- **Description:** Static facade over input backends. `TryGet(out Pointer p)` returns the current pointer using either the **New Input System** or **Legacy Input** based on project definition.
- **Where it lives:** Static runtime class. Extend by implementing `IPointerSource` if you need a custom backend.
- **Customizable Attributes:** none
- **Hooks:**
  - `static bool TryGet(out Pointer p)` — fetch current pointer state
  - `SetSource(IPointerSource source)` — set a custom source (for testing or specialized input)
- **See Also:** [DragCard](#dragcard-c), [CardHoverable](#cardhoverable-c), [InteractionController](#interactioncontroller-c)
- **Module:** Interaction
- **Example:** DragCard excerpt

```csharp
if (PointerService.TryGet(out Pointer currentPtr))
{
    // use currentPtr.screenPos etc
}
```

### ScriptableCard **[C]**

- **Description:** Authoring-time data for a card definition (name, art, rarity, base stats, and custom addons). Looked up at runtime by guid when building visuals.
- **Where it lives:** ScriptableObject assets wherever you place them. Indexed by `CardDatabase` and retrieved via `CardLibraryLoader`.
- **Customizable Attributes:**
  - `slug` — string — stable identifier for lookups.
  - `cardName` — string — display title.
  - `rarity` — enum/string — maps to frame selection.
  - `art` — Sprite — main artwork (also used for Full Image if desired).
  - `baseStats` — int? Cost, Attack, Health — optional.
  - `addons` — List<CardAddon> — user-defined modular data (e.g., suit, counters). Note, ScriptableCards has a custom inspector to help manage addons, once you create your own addon types.
- **Hooks:** none (data only; behavior via view model or systems)
- **See Also:** [CardAddons](#cardaddons-e), [CardDatabase](#carddatabase-c), [CardLibraryLoader](#cardlibraryloader-c)
- **Module:** Data & Deck
- **Example:** ScriptableCard data used in SampleCardViewModel

```csharp
public void Initialize(ScriptableCard card)
{
  _card = card ?? throw new ArgumentNullException(nameof(card));
  _state = new SampleRuntimeState();
  RecomputeAll();
}

public void RecomputeAll()
{
  // Example getting game specific data from an addon
  ExampleCardAddon addon = _card.GetAddon<ExampleCardAddon>(_card);
  Title = _card.cardName;
  AbilityTitle = addon.abilityTitle;
  //...
  _caps.Clear();
  // If the card has an ability title, show the ability title field. (Ability Title is a custom field)
  SetCapability("abilityTitle", !string.IsNullOrEmpty(AbilityTitle));
  OnChanged?.Invoke();
}
```

### SingleSlotZone **[L]**

_(Starter Example)_

- **Description:** Minimal `ICardZone` that holds a single card (or last-wins replacement). Intended as a lean starting point, not a production zone.
- **Where it lives:** Component on a scene transform representing the slot. Implements `ICardZone`, `ILockableZone`, and `IPlayResolver`.
- **Customizable Attributes:**
  - `slot` — Transform — parent for the card.
  - `allowReplaceWhenOccupied` — bool — if true, new plays replace the current occupant.
  - `priority` — int — resolver priority when multiple zones are valid for play. Lower numbers are higher priority.
- **Hooks:**
  - Implements `ICardZone: AddCard, RemoveCard, AlignCards, ZoneRoot, Type, SeatId, IsFocused, IsTransitioning.`
  - Implements `IPlayResolver: TryResolvePlay, Priority`
- Implements `ILockableZone: Lock, Unlock, IsLocked, OnLockChanged`
- **See Also:** [ICardZone](#icardzone-e), [IPlayResolver](#iplayresolver-e), [LineBoardZone](#lineboardzone-l)
- **Module:** Zones & Play
- **Example:** TryResolvePlay excerpt

```csharp
public PlacementResult TryResolvePlay(PlacementQuery query)
{
  PlacementResult result = new PlacementResult
      { allowed = false, slotIndex = null, worldPos = Vector3.zero, worldRot = Quaternion.identity };

  if (IsLocked || (cards.Count > 0 && !allowReplaceWhenOccupied)) return result;

  if (cards.Count > 0 && !allowReplaceWhenOccupied)
    return result;

  // Remove from source
  if (query.fromZone != null && query.fromZone as Object != this)
  {
    if (query.fromZone.Type == ZoneType.Hand || query.fromZone.Type == ZoneType.OpponentHand)
      HandStateService.Instance.RemoveCard(query.fromZone.SeatId, query.card.InstanceId);
    else
    {
      query.fromZone.RemoveCard(query.card);
    }
  }

  // Auto-replace behavior
  if (cards.Count > 0)
  {
    var replaced = cards[0];
    if (replaced == query.card)
      return result;

    cards.RemoveAt(0);
    HandStateService.Instance.AddCard(replaced.GetComponent<CardZoneTracker>()?.HomeZone?.SeatId ?? -1, replaced.InstanceId);
  }

  // Adopt new card at slot
  AddCard(query.card, 0);
  AlignCards();

  // Result Update
  // ...
  return result;
}
```

### VisualFactoryLoader **[C]**

- **Description:** Scene singleton that builds card **visuals** from a `CardRef`: instantiates the card prefab, initializes `CardBehaviour`, creates the `ICardViewModel`, and runs `CardPresenter` with the active `CardSkin`.
- **Where it lives:** On the bootstrap/initial scene (added by Setup Wizard). Accessible via `VisualFactoryLoader.Instance`.
- **Customizable Attributes:**
  - `visualFactory` — ICardVisualFactory — strategy asset that performs the actual build.
- **Hooks:**
  - `bool IsReady` — true when the loader is present and configured.
  - `ICardVisualFactory VisualFactory` — active factory reference.
  - `void SetVisualFactory(ICardVisualFactory newFactory)` — swap factory at runtime (used also in Setup Wizard to assign it in editor).
- **See Also:** [CardVisualFactorySO](#cardvisualfactoryso-c), [CardPresenter](#cardpresenter-l), [PlayerHand](#playerhand-c)
- **Module:** Visuals & Factories
- **Example:** Excerpt from PlayerHand OnCardAdded

```csharp
protected override void OnCardAdded(int seat, CardRef cardRef)
{
  if (seat != SeatId) return; // only care about local player
  CardBehaviour cb;
  if (!SceneCardLookup.TryGet(cardRef.InstanceId, out cb))
  {
    cb = VisualFactoryLoader.Instance.VisualFactory.Build(cardRef, drawOrigin);
  }

  AddCard(cb); // your existing layout method

  NotifyOrderDirty();
}
```

---

## FAQ

**Q: What’s safe to modify?**  
A: Anything tagged **[C]** is intended for configuration. Types tagged **[E]** are designed for extension via interfaces/policies. Types tagged **[L]** are primarily informational.

**Q: Do I have to use the example zones?**  
A: No. They’re teaching tools. Implement your own `ICardZone` + `IPlayResolver` for production.

**Q: Why does hover flicker near the bottom of the screen on PC?**  
**A:** Use a `HoverAnchor` on the PlayerHand **Focus Position**. Ensure the teal gizmo below the screen bottom

**Q: How do I change what’s allowed (hover/drag/select/target)?**  
**A:** Supply an `IInteractionPolicy` (globally via `InteractionService`, or per-zone via a provider). Per-card overrides: add `CardInteractionOverride`.

**Q: Can I block drag on one specific card?**  
**A:** Yes. Add `CardInteractionOverride` and set **Deny: Drag**.

**Q7: What happens when the deck is empty?**  
**A:** `IEmptyDeckPolicy.TryResolveEmpty` runs. If it returns **false**, the deck fires `EmptyEvent`.

**Q: Can players reorder cards in hand by dragging?**  
**A:** Yes, if the hand implements `IReorderBandZone`. `DragCard` uses it to compute insert indices and shift spacing.

**Q10: Mobile vs PC hover behavior?**  
**A:** PC uses pointer hover and `HoverAnchor`. Mobile uses touch long-press with its own delays (tune in `CardHoverable`).
