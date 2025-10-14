## Table of Contents

- [Getting Started & Install](#quick-start-install)
- [Run the Setup Wizard](#quick-start-wizard)
- [Create Your CardPlayable Variant](#quick-start-create-cardplayable)
- [Make It Play (Drawing a Card)](#quick-start-make-it-play)
- [Visual Customization & Card Views](#quick-start-visual-customization)
- [Next Steps](#quick-start-next-steps)

<a id="quick-start-install"></a>

# Quick Start — Section 1: Getting Started & Install

> **Goal:** Import the package, confirm folder layout, and open the Setup Wizard so your project is ready for a first scene.

## Requirements

- **Unity:** 2023.1 or newer
- **Input:** Works with both Legacy and New Input System (no extra setup required)
- **Platforms:** Desktop & Mobile

> **Install type:** This asset is distributed as a classic **`.unitypackage`** (not UPM). Paths below assume import into `Assets/CardHandToolkit/…`. If you import to a different folder, adjust paths accordingly.

## 1) Import the package

1. In Unity, go to **Assets ▸ Import Package ▸ Custom Package…**.
2. Select `CardHandToolkit.unitypackage`.
3. In the Import dialog, keep everything **checked** and click **Import**.

## 2) Verify folder layout

After import, you should see:

```
Assets/
└─ CardHandToolkit/
├─ Config/ # Wizard-created ScriptableObjects will live here
├─ Documentation/ # This guide + API docs
├─ Editor/ # Setup Wizard + editor-only utilities
├─ Runtime/
│ ├─ Prefabs/
│ │ ├─ CardToolkitLoader.prefab
│ │ └─ CardPlayable.prefab
│ └─ ... # Runtime scripts, materials, etc.
├─ Samples/ # Optional sample content (scenes, helpers)
└─ Tests/ # (Empty for now)
```

## 3) Open the Setup Wizard

1. Go to **Tools ▸ Card Toolkit ▸ Setup Wizard**.
2. The Wizard will guide you through:
   - **Layers & Libraries:** create or assign
     - `Config/CardToolkitLayers.asset`
     - `Config/CardDatabase.asset`
     - `Config/CardFrameLibrary.asset`
   - **Bootstrap Scene:** create/open your initial scene (default name: **`Bootstrap`**)
     - Create Visual Prefabs and ScriptableObjects:
       - `Config/CardPlayable_User.prefab` (variant of `CardPlayable`)
       - `Config/UserWorldViewPrefab.prefab` (variant of `WorldViewPrefab`)
       - `Config/CardSkin_User.asset` (variant of `CardSkin_Default`)
       - `Config/CardVisualFactory_User.asset` (variant of `CardVisualFactory_Default`)
   - **(Optional) Game Scene:** create/prepare a playable sample scene
   - **Links:** samples, docs, video playlist, support

> **Tip:** The Wizard can **create or use** an existing scene. When creating a new scene, it uses Unity’s default new-scene flow and sets the scene name to the string you enter (default shown in the Wizard as **`Bootstrap`**).

## 4) Common early checks

If a required service is missing from your initial scene, you’ll see a friendly one-shot error like:

CardToolkit: LayersLoader not found in scene. Please add one to your initial scene.  
(It's part of the CardToolkitLoader Prefab, which can be added via the Setup Wizard found under Tools > Card Toolkit > Setup Wizard)

Similar messages appear for:

- `Card Frame Library` (via `CardFrameLibraryLoader`)
- `Card Library` (via `CardLibraryLoader`)
- `HoverPolicyRuntime`
- `InteractionService`
- `VisualFactoryLoader`

## 5) Mouse vs Touch (hover behavior)

> **Mouse:** Hover is literal cursor-over-card.  
> **Touch:** “Hover” is detected when pressing and holding without dragging.

> **Hover Gizmo (Desktop):** There’s a visual gizmo for the **hover anchor** line (where cards rest while hovered). Place this below or at the bottom edge of the screen to avoid flicker.  
> **Play Height:** A separate numeric threshold (default **`1.5f`** on `PlayerHand`) used by both mouse and touch to detect a “play” drag.

<a id="quick-start-wizard"></a>

# Quick Start — Section 2: Run the Setup Wizard

> **Goal:** Use the Wizard to create the required assets, place the Bootstrap prefab, and get your project ready for a first playable scene.

## Overview

Open via **Tools ▸ Card Toolkit ▸ Setup Wizard**. The Wizard guides you through four pages:

1. **Layers & Libraries**
2. **Bootstrap Scene**
3. **Game Scene (Optional)**
4. **Documentation Links**

Each page validates before **Next** is enabled.

## Page 1 — Layers & Libraries

This page ensures your project has the core ScriptableObjects and layer configuration.

### What you’ll assign or create

- `Assets/CardHandToolkit/Config/CardToolkitLayers.asset`
- `Assets/CardHandToolkit/Config/CardDatabase.asset`
- `Assets/CardHandToolkit/Config/CardFrameLibrary.asset`

### Steps

1. **Layers**
   - Select or create two layers in the UI: **Cards** and **PlayAreas** (names are up to you).
   - If you choose “Create new…”, the Wizard writes them into **Project Settings ▸ Tags and Layers** (first free slot ≥ 8).
2. **Libraries**
   - If any ObjectField is empty, click **Create** next to it. The asset is created under `Assets/CardHandToolkit/Config/` or in a location of your choosing.
3. Click **Next** once all three ObjectFields are assigned and both layer pickers are set to concrete layers (not “Create new…”).

### Validation rules

- Both layer fields must be non-empty (not “Create new…”).
- All three assets must be assigned (or newly created).
- If any is missing, a small inline warning appears and **Next** is disabled.

> **Success state:** Three assets exist and referenced in the setup wizard, both layers assigned.

## Page 2 — Bootstrap Scene

This sets up the initial scene that loads first and persists your core services.

### What it does

- Creates or opens a scene (default name: **`Bootstrap`**) using Unity’s editor APIs.
  - **Create new:** a fresh scene with Unity defaults (additive load), then renamed to your entry.
  - **Open existing:** additively opens the selected scene.
- Adds **`CardToolkitLoader.prefab`** from `Assets/CardHandToolkit/Runtime/Prefabs/`.

### Steps

1. Choose **Create New** (default name: `Bootstrap`), **Use Existing** (select a scene asset), or **Skip** (You will need to manually add the `CardToolkitLoader` prefab later).
2. If creating new, optionally select set as first in Build Settings, to make sure it loads first.
3. Create Visual Objects and Prefab Variants:
   - **Base Card Prefab Variant** (required) — prefab variant of `CardPlayable`, allows for customization on interaction (default: `CardPlayable`).
   - **World View Prefab Variant** (required) — prefab for a panel displaying runtime data overlayed on the card (default: `WorldViewPrefab`).
   - **Default Card Skin** (required) — global skin ScriptableObject (default: `CardSkin_Default`).
   - **Default Visual Factory** (required) — prefab factory ScriptableObject, controls initialization of new cards (default: `CardVisualFactory_Default`).
   - These three are required. If any is missing, click **Create** next to it to make a default asset.
4. Click **Next**.

### What’s inside the prefab

- `LayersLoader`
- `CardFrameLibraryLoader`
- `CardLibraryLoader`
- `HoverPolicyRuntime`
- `InteractionService`
- `VisualFactoryLoader`
- Marks itself `DontDestroyOnLoad` in `Awake()`

### Build Settings

After finishing the Wizard, ensure **File ▸ Build Settings…** lists **Bootstrap** **first** in _Scenes In Build_.

> **Missing service checks (shown once):**
>
> ```
> CardToolkit: LayersLoader not found in scene. Please add one to your initial scene.
> (It's part of the CardToolkitLoader Prefab, which can be added via the Setup Wizard found under Tools > Card Toolkit > Setup Wizard)
> ```
>
> Similar messages appear for: `CardFrameLibraryLoader` (Card Frame Library), `CardLibraryLoader` (Card Library), `HoverPolicyRuntime`, `InteractionService`.

> **Success state:** A scene named `Bootstrap` contains `CardToolkitLoader` and saves without errors. It’s first in Build Settings.

## Page 3 — Game Scene (Optional)

Use this page to **create or select your own playable scene** (separate from Bootstrap). You can also place demo objects into it.

### Mode

- **Create New** — makes a new scene additively (you provide a name).
- **Use Existing** — pick any scene asset in your project.
- **Skip** — do nothing on this step.

> ✅ **Validation:**
>
> - _Create New_ requires a non-empty scene name.
> - _Use Existing_ requires a scene asset selected.
> - _Skip_ has no requirements.

### Content To Place (checkboxes)

- **Player Hand** (Checkbox Disabled) — fan layout with hover/drag interactions.
- **Opponent Hand** — same layout with spread multiplier; cards face-down by default.
- **Line Play Zone** — Arena-style line zone (implements `ICardZone`).
- **Single Slot Play Zone** — One-slot zone example (implements `ICardZone`).
- **HoverPresenter** — overlay presenter with its own Canvas.
- **Game Camera Setup** — optional camera configuration as POV right now requires a 2D Camera Angle.

> These are **example placements** to get you moving. You can delete or replace them later, or create your own zones that implement `ICardZone`.

### What Happens

1. The selected/new scene is opened **additively**.
2. Checked items are instantiated into the scene (default transforms centered; adjust as needed).
3. Hands listen to the `HandStateService` for add/remove events; Opponent hand renders face-down by default.

> **Success state:** Your scene exists (created or selected), any chosen content is placed, and the scene is saved.

> **Note on Samples:** The Wizard does **not** open a sample scene here. Sample links (if any) are on the **Links** page; this step configures **your** scene.

## Page 4 — Samples & Docs

This page centralizes learning and support links.

### Documentation

- **Watch Quick-Start Playlist** _(YouTube; link TBD)_ — Short videos covering install → first play.
- **Ask the CHT Assistant (GPT)** _(link TBD)_ — Chat assistant for common setup and API questions.
- **Quick Start** — Opens `Assets/CardHandToolkit/Documentation/QuickStart.md`.
- **API Reference** — Opens `Assets/CardHandToolkit/Documentation/API.md`.
- **Changelog** — Opens `Assets/CardHandToolkit/Documentation/CHANGELOG.md`.
- **License** — Opens `Assets/CardHandToolkit/Documentation/LICENSE.md`.

### Support

- **Report An Issue** — Opens your issue tracker _(GitHub Issues link TBD)_.
- **Contact Support** — Opens your support email _(mailto:… TBD)_.
- **FAQ** — Opens `Assets/CardHandToolkit/Documentation/FAQ.md`.

> **Success state:** You’ve reviewed links and are ready to finish. Click **Finish** to close the Wizard.

## Post-Wizard Checklist

1. **Bootstrap** scene present and first in Build Settings.
2. `CardToolkitLoader` exists in Bootstrap.
3. Three config assets exist in `Assets/CardHandToolkit/Config/` (Or wherever you chose to create them):
   - `CardToolkitLayers.asset`
   - `CardDatabase.asset`
   - `CardFrameLibrary.asset`
   - `CardPlayable_User.prefab`
   - `CardSkin_User.asset`
   - `CardVisualFactory_User.asset`
   - `UserWorldViewPrefab.prefab`
4. (Optional) Your **Game** scene is created/prepped and saved.

> 🎉 You’re ready to add content. Continue to **Section 3: Create a CardPlayable Variant** to make your first card and see it in-hand.

<a id="quick-start-create-cardplayable"></a>

# Quick Start — Section 3: Create Your CardPlayable Variant

> **Goal:** Understand and lightly customize the card prefab, card data, and visual setup that the Setup Wizard just created for you.

## 1) Review what the Wizard created

After finishing **Page 2 – Bootstrap Scene**, you now have these assets:

| Asset                         | Purpose                                                                                                                    | Default Location                 |
| ----------------------------- | -------------------------------------------------------------------------------------------------------------------------- | -------------------------------- |
| **GameCard.prefab**           | Variant of `CardPlayable`, used for all cards in-game. Customize its interaction feel or swap its 3D model.                | `Assets/CardHandToolkit/Config/` |
| **WorldViewPrefab (Variant)** | Runtime display overlay shown on cards (art, text, stats).                                                                 | `Assets/CardHandToolkit/Config/` |
| **CardSkin_Game.asset**       | Clone of the default global Card Skin. Defines colors, frames, and which prefabs display card visuals.                     | `Assets/CardHandToolkit/Config/` |
| **CardVisualFactory.asset**   | Clone of the default Visual Factory. Initializes cards at runtime and holds the global Card Skin + Card Prefab references. | `Assets/CardHandToolkit/Config/` |

The **CardToolkitLoader** in your Bootstrap scene now references this Visual Factory, so every card spawned at runtime will use your customized skin and prefab.

## 2) Create ScriptableCards (your card data)

1. **Create ▸ Tools ▸ Card Toolkit ▸ Card**.
2. Save each new `ScriptableCard` in `Assets/CardHandToolkit/Config/` (e.g., `Fireball.asset`, `Goblin.asset`).
3. Fill in:
   - **Slug** (unique id)
   - **Card Name**
   - **Rarity**
   - **Card Art** (sprite)
   - Any custom data fields your ViewModel reads create as an Addon that will be a part of the addon list on the ScriptableCard. (We'll cover this in a later section)

> Each `ScriptableCard` represents gameplay data only; visuals are handled by the worldViewPrefab + global skin at runtime.

## 3) (Optional) Adjust your GameCard prefab

Open **`GameCard.prefab`** in the Inspector to tune your game’s “feel.”

- **DragCard** – speed, plane, or touch-drag threshold
- **CardHoverable** – hover timeout or long-press delay
- **CardBehaviour** – leave **Definition** empty (runtime assigns it)

> These tweaks affect **every** card instance since all cards use this single prefab.

For deeper visual customization (changing frame style, world-view prefab, etc.), see **Concepts ▸ Visual Pipeline & Tuning**.

## 4) Add your ScriptableCards to the Card Database

1. Open `Assets/CardHandToolkit/Config/CardDatabase.asset`.
2. Add your newly created `ScriptableCard` assets (Fireball, Goblin, etc.).

This makes them discoverable to factories and draw systems.

## 5) Optional smoke test (no code)

1. Drop an instance of your GameCard Prefab Variant under the **Player Hand** in your Game scene and **Play**.
2. Verify hover/drag feel with your tweaked values.

---

**Next:** Section 4 will show how to add cards to the local player’s hand at runtime (auto-draw or via a button), so they bind a `ScriptableCard` and can be played.

<a id="quick-start-make-it-play"></a>

# Quick Start — Section 4: Make It Play (Drawing a Card)

> **Goal:** Automatically draw cards from a deck into the player’s hand and see them come alive with data and visuals.

## 1) How drawing works

At runtime, cards are added to hands through the **`HandStateService`**.

```csharp
HandStateService.Instance.AddCard(int seatId, string definitionGuid);
```

When you call this:

1. An event is fired.
2. The Hand checks if that card already exists in the scene.
3. If not, it asks the **Visual Factory** (wired in the Bootstrap prefab) to create it.
4. The **Visual Factory**
   - Instantiates your `GameCard.prefab`
   - Initializes its `CardBehaviour` with the `ScriptableCard`
   - Creates a ViewModel
   - Binds it to the `CardPresenter` and `GlobalCardSkin`
5. The card is positioned and animated by the **Player Hand** zone.

You never need to manually instantiate cards — just call **AddCard()**.

## 2) Create or use a Deck

If you don’t already have one, create a deck:

1. **Create ▸ Tools ▸ Card Toolkit ▸ Deck List**
2. In the Inspector, add your cards to the list (ScriptableCards).
3. Save it under `Assets/CardHandToolkit/Config/`.

The **Deck List** is a simple list of scriptable cards, you can have duplicates etc. It does not manage state.  
To use the DeckList at runtime, you need to create a **Deck** instance. The **Deck** class supports draw, discard, shuffle, and reshuffle policies.  
By default, it reshuffles when empty, but you can change this (see API docs).

## 3) Auto-draw on Start

Add this helper script anywhere in your scene (for example, an empty GameObject named **QuickStartManager**):

```csharp
using UnityEngine;
using CardHandToolkit;

public class QuickStart_DrawOnStart : MonoBehaviour
{
[SerializeField] DeckList deck; // Assign your DeckList ScriptableObject in the inspector
[SerializeField] int drawCount = 3;

    void Start()
    {
        int localSeat = 0; // Local player seat id
        for (int i = 0; i < drawCount; i++)
        {
            var guid = deck.Draw();
            if (guid != null)
                HandStateService.Instance.AddCard(localSeat, guid);
        }
    }
}
```

On Play, you’ll see cards spawn smoothly into the Player Hand.
Their art is pulled automatically from the `ScriptableCard` definition and once we setup the world view prefab, they’ll be fully detailed.

> **Note:** The `DeckList` asset is a simple list of cards. The `Deck` class manages state (draw, discard, shuffle). You can create a `Deck` instance in code using your `DeckList` and use it to draw cards.  
> Example: `Deck<string> deck = new Deck<string>(deckList.cards.Select(card => card.Guid));`

## 4) Play a card

When you drag a card above the **Play Height** line, it triggers the card’s “play” action.

- Cards check all active **IPlayResolver** components (for example, LineBoardZone, SingleSlotZone).
- The first resolver in priority order whose `TryPlayResolve` method succeeds handles the card.
- Cards then move to that zone and complete their play animation.

Each IPlayResolver has a priority number, if both LineBoardZone and SingleSlotZone exist, priority is handled automatically by resolver order.

> **Note:** Classic Priority order example: A Play Zone: 0, Secret Zone: 5, Discard Pile: 10. - Lower numbers are higher priority.  
> In this example, if a card is played, and can't go to a play zone, it will try to go to the secret zone next, and if that fails, it will go to the discard pile if allowed. Otherwise, it will stay in the player's hand.

## 5) What to expect

| Interaction                | Behavior                                                             |
| -------------------------- | -------------------------------------------------------------------- |
| **Hover (mouse)**          | Card lifts and scales per hover settings                             |
| **Long-press (touch)**     | Behaves like hover                                                   |
| **Drag**                   | Card follows pointer with smoothing                                  |
| **Drag Above Play Height** | Card border highlights                                               |
| **Play above height**      | Card leaves hand and moves to the play zone (if one exists in scene) |
| **No play zone**           | Card returns to hand smoothly                                        |

## 6) What’s happening under the hood

1. **HandStateService** tracks card guids in players hands.
2. **SceneCardLookup** tracks existing cards in the scene.
3. The **PlayerHand** listens for add/remove events.
   1. On add, it checks if the card instance already exists in the scene. (IE returning to hand from play)
   2. If not, it calls `VisualFactory.CreateCard(...)`.
4. The **Visual Factory** builds the prefab and initializes `CardBehaviour` + `CardPresenter`.
5. The **CardPresenter** binds to the global **Card Skin** and **ViewModel**.
6. **PlayerHand** manages layout and animation curves.
7. **Play Resolvers** handle card movement when played.

You can see a full example in the sample scene’s **CardManager.cs**, which handles phase-based draws and discards.

---

**Next:** In **Section 5: Visual Customization & Card Views**, you’ll learn how to customize how your cards _display data_.  
This includes:

- Setting up your **World View Prefab** (the UI or mesh that renders card visuals)
- Implementing your own **ICardViewModelFactory** and **ICardViewModel** to define what data a card exposes
- Using **ICardViewBinder** on the World View Prefab to connect your visual elements to the ViewModel’s data

<a id="quick-start-visual-customization"></a>

# Quick Start — Section 5: Visual Customization & Card Views

> **Goal:** Connect your card data to visuals by setting up your World View Prefab, ViewModel, and Binder.

---

## 1) The World View Prefab

Your card visuals come from the **World View Prefab** referenced by your Card Skin.  
This prefab defines how text, icons, and art\* appear directly on the 3D card and hover overlay.  
This prefab variant was created by the Setup Wizard, and it's default location is `Assets/CardHandToolkit/Config/UserWorldViewPrefab.prefab`.

> \*Note: Card art for in scene cards is rendered using the sprite assigned to the `ScriptableCard` definition. And this is applied to the cards material, same with the frame coming from the rarity and FrameLibrary.  
> Card Art on the WorldViewPrefab is only used for the hover overlay, where there is no 3D model.

### Layout rules

1. **Design Root:**  
   Place all visual elements (texts, images, icons) under the **Design Root** child object.  
   This ensures consistent fitting and scaling across all card sizes.

2. **Anchors:**  
    Use **anchored positioning and scaling**, not fixed offsets.  
    Avoid anchoring just to a corner — this prevents stretching or clipping when the card resizes.  
   ![Example World View layout](Images/UnityCardAnchoring.png)

3. **Field Toggles:**  
   Add a **FieldToggle** component to each element you want to show or hide dynamically.
   - Built-in types: `Title`, `Name`, `CardNumber`, etc.
   - Custom: define your own identifiers (for example, `"Ability Title"`).  
     These toggles respond automatically based on your Card Skin and ViewModel data (see below).

> The default World View created by the Setup Wizard is already linked to everything, so once you have your ViewModel and ViewBinder set up, it should just work.

## 2) The Card ViewModel

Implement **ICardViewModel** to define what data a card exposes to the UI.

- Constructed automatically when a card is initialized.
- Pulls data from the **ScriptableCard** (and optionally from runtime state, such as modifiers or buffs).
- Provides easy-to-read properties like `Title`, `Description`, `Cost`, etc.
- Can maintain a lightweight runtime struct for transient values (for example, “power modifiers”).
- Fires an **OnChanged** event when data updates, triggering the view to refresh.

### Example summary

Your own ViewModel might:

- Store base data from `ScriptableCard`.
- Track runtime changes (damage, counters, etc.).
- Expose these through properties.
- Include a `Capabilities` list to determine which custom fields the card supports.

The included **SampleCardVM** shows a minimal example of this pattern.

## 3) The ViewModel Factory

Cards use an **ICardViewModelFactory** to create the appropriate ViewModel for each card type.  
You can register your own factory to replace the default dummy VM.

- Create a new ScriptableObject that implements `ICardViewModelFactory`.
- Implement `ICardViewModelFactory.Create(ScriptableCard definition)`
- Return your custom `ICardViewModel` implementation.
- Assign your factory asset in the Visual Factory (created by the Wizard, default directory `Assets/CardHandToolkit/Config/CardVisualFactory_User.asset`).

Example of whole factory used in the sample:

```csharp
    [CreateAssetMenu(menuName = "Tools/Card Toolkit/Sample/ViewModelFactory")]
    public class SampleViewModelFactory : ScriptableObject, ICardViewModelFactory
    {
        public ICardViewModel Create(ScriptableCard def)
        {
            var vm = new SampleCardVM();
            vm.Initialize(def);
            return vm;
        }
    }
```

> You only need one global factory; it decides which ViewModel to return based on your game’s card types. For now, you can return the same ViewModel type for all cards.

## 4) The Card View Binder

Each World View prefab needs a component that implements **ICardViewBinder**.

- Exposes a single `Bind(ICardViewModel vm)` method.
- Subscribes to `vm.OnChanged` to update visuals automatically.
- In its refresh/update logic, it assigns text, numbers, sprites, and other elements from the ViewModel.
- Once you create your ICardViewBinder, add it to your World View Prefab Variant.

Typical serialized fields on the binder:

- Title text
- Description text
- Card number
- Icon images or cost symbols

This keeps the prefab visually driven — no hard-coded updates elsewhere.

Example of a simple binder:

```csharp
public sealed class SampleCardViewBinder : MonoBehaviour, ICardViewBinder
  {
    [SerializeField] TMP_Text title;
    [SerializeField] Image art; // Used for Hover overlay only
    [SerializeField] Image frameArt; // Used for Hover overlay only

    SampleCardVM vm;

    public void Bind(ICardViewModel viewModel)
    {
      if (vm != null) vm.OnChanged -= Refresh;

      vm = viewModel as SampleCardVM;

      if (vm == null)
      {
        Debug.LogError("SampleCardViewBinder: Bind requires a SampleCardVM.", this);
        return;
      }

      vm.OnChanged += Refresh;
      Refresh();
    }

    void OnDestroy() { if (vm != null) vm.OnChanged -= Refresh; }

    void Refresh()
    {
      // Just binds data, the CardPresenter handles what should show/hide.
      if (vm == null) return;

      title.text = vm.Title ?? "";

      if (art)
        art.sprite = vm.ArtSprite;

      if (frameArt)
         frameArt.sprite = vm.FrameSprite;
    }
  }
```

## 5) Field Toggles in action

Both **CardPresenter** and **HoverPresenter** control which fields are visible.  
Here’s the order of logic they use:

1. The presenter collects all **FieldToggle** components from the instantiated World View.
2. It builds a list of which fields _should_ be visible by checking:
   - Is the field enabled in the **Global Card Skin**?
   - If it’s a custom field, is it also present in the ViewModel’s **Capabilities** list?
   - For built-in fields, does the ViewModel have non-empty data for that field?
3. If all conditions are met, the field is shown; otherwise, it’s hidden.

> This system means you rarely need manual “show/hide” code — the Presenter automatically shows relevant data based on your skin and ViewModel.

## 6) Testing your visual bindings

1. Open your **World View Prefab Variant** created by the Wizard.
2. Verify the base prefab has a component that implements **ICardViewBinder**.
3. Verify each UI element has a **FieldToggle** and an appropriate field type.
4. In Play Mode, draw cards into your hand using the card drawing mechanic.
5. Verify that card data (title, art, etc.) appears correctly on the cards in your hand and when hovered while in play zones.

**Next:** You’re ready to dive deeper into **Concepts & API Reference**, where we’ll cover:

- Custom ViewModels and capability systems in depth
- Extending play resolvers
- Advanced presenter behavior for overlays and hover views

<a id="quick-start-next-steps"></a>

# Quick Start — Section 6: Next Steps & Resources

> You now have a working setup: cards that draw, display data, and can be played into zones!

With your toolkit configured, you can start shaping your own game’s rules, visuals, and card logic.  
The sections below link to detailed references and customization guides.

---

## 1) Learn the Core APIs

| Topic                            | Description                                                                                                                   |
| -------------------------------- | ----------------------------------------------------------------------------------------------------------------------------- |
| **API Reference**                | Full list of public classes, interfaces, and services (e.g., `PlayerHand`, `HandStateService`, `ICardZone`, `IPlayResolver`). |
| **Visual Pipeline & Tuning**     | Explains how the Visual Factory, Card Skin, ViewModel, and Binder interact.                                                   |
| **Play Resolvers**               | Guide to writing custom `IPlayResolver` components for new board/zone logic.                                                  |
| **Hover & Interaction Policies** | Deep dive into `HoverPolicyRuntime` and `InteractionService`.                                                                 |
| **Decks & Game Phases**          | Shows how to control draw, discard, and reshuffle through your own game logic.                                                |

## 2) Where to go next

- **[API Reference](API.md)** — in-depth look at every class, interface, and event.
- **[Concepts Overview](API.md#concepts)** — high-level explanations of the toolkit’s systems.
- **[FAQ](FAQ.md)** — answers to common setup or integration questions.
- **[Changelog](CHANGELOG.md)** — see what’s new in each version.
- **[License](LICENSE.md)** — terms for use in your commercial or personal projects.

## 3) Community & Support

If you have questions, feedback, or want to show what you’ve built:

- **Discord** _(optional)_ — join the discussion or get quick setup help.
- **Ask the CHT Assistant (GPT)** — interactive help inside the editor.
- **Watch the Quick Start Playlist** _(YouTube link TBD)_.
- **Report an Issue** or **Contact Support** through the links in the Wizard’s _Samples & Docs_ page.

---

> You’ve completed the Quick Start!  
> Your cards are now interactive, data-driven, and visually bound — the rest is pure creativity.
