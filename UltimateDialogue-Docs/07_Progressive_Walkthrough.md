# Progressive walkthrough: a conversation with a choice

This walkthrough creates a merchant conversation with a conditional choice, a global variable, and a Blueprint event.

## Step 1 — Set up the variable

For a simple test, add a `Set Global Bool` node before starting the dialogue:

- `Name`: `Player.HasMap`
- `Value`: `true` or `false`

Conditions can read this value because `Fallback To Global Variables` is enabled by default. To keep it after the game session, call `Save Global Dialogue Variables`.

## Step 2 — Create the StateTree

1. Create a `StateTree` asset and select `Dialogue StateTree Schema` in `Schema`.
2. Add these states in this order: `Greeting`, `Offer`, `Accept`, `Accepted`, `Decline`, `NoMap`, `Finish`.
3. Add one enabled `Dialogue:` task to each state.

The first dialogue state is the entry point.

## Step 3 — Configure the conversation

- `Greeting` — `Dialogue: Line`
  - `Speaker Id`: `Merchant`
  - `Text`: `I have an old map for sale.`
  - `Next State`: `Offer`
- `Offer` — `Dialogue: Choice`
  - Entry 0: `Show me the map.` → `Accept`
  - Entry 1: `Not now.` → `Decline`
  - Leave `Choice Timeout` at `-1` to wait for the player.
- `Accept` — `Dialogue: Branch`
  - Enable `Use Condition`.
  - `Condition`: `Player.HasMap`, Bool, `Equal`, `true`.
  - `True State`: `NoMap`; `False State`: `Accepted`.
- `Accepted` — `Dialogue: Trigger`
  - `Event Name`: `Merchant.AcceptedMapOffer`
  - `Next State`: `Finish`.
- `NoMap` — `Dialogue: Line`
  - `Text`: `You already have a map.`
  - `Next State`: `Finish`.
- `Decline` — `Dialogue: Line`
  - `Text`: `Come back if you change your mind.`
  - `Next State`: `Finish`.
- `Finish` — `Dialogue: End`.

For your game, adjust the condition and routes to match their intended meaning. A local value with the same name takes precedence over the global value; global fallback is used only when the local variable is missing.

## Step 4 — Start and display

For a level test, place `DialogueStarter`, assign the StateTree, and leave `Auto Start` and `Show Default Widget` enabled.

To start it from a Blueprint:

1. Call `Get Game Instance Subsystem` → `Dialogue Subsystem`.
2. Call `Start Dialogue` and keep the returned instance.
3. Call `Show Default Dialogue Widget` with that instance.
4. Bind `On Choice Selected` if you want to display or record the confirmed response.

## Step 5 — Add a Blueprint reaction

1. In `Accepted`, use the `Dialogue: Trigger` task configured above.
2. In the Blueprint that starts the dialogue, bind `On Trigger`.
3. When `Event Name` equals `Merchant.AcceptedMapOffer`, run the desired gameplay logic.

The name is an identifier chosen by your project. The plugin emits the event; your Blueprint decides what it does.

## Variations to try

- Add `Dialogue: Set Variable` before `Finish` to remember a response.
- Use `Dialogue: Emotion` and observe `On Emotion Changed`.
- Add a `Dialogue: Wait` state between lines.
- Duplicate `Offer` and use `Dialogue: Check Visited` to change a line that was already displayed.
- Select `WBP_Dialogue_Cinematic` or `WBP_Dialogue_Minimal` in `Default Widget Class` to try another presentation.
