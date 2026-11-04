# Authoring dialogue with StateTree

The current authoring format is an Unreal `StateTree` asset whose `Schema` is `Dialogue StateTree Schema`. Dialogue steps are tasks whose names begin with `Dialogue:`.

## Structure rules

- Use one enabled `Dialogue:` task per state. If a state contains multiple tasks, only the first enabled dialogue task is used.
- The first state containing an enabled dialogue task, in StateTree order, is the entry point.
- Use the task's `State` links (`Next State`, `True State`, and so on) to connect steps. Every required link must target a state with an enabled dialogue task.
- Tasks that do not display a prompt (`Branch`, `Trigger`, `Set Variable`, `Random`, `Wait`, `Jump`, `Chance`, `Emotion`, `Check Visited`) advance automatically.
- Save the asset and ensure it is compiled and ready to run before testing.

## Available tasks

| Task name | Purpose and important fields |
|---|---|
| `Dialogue: Line` | Displays `Speaker Id`, `Text`, and optionally `Voice Over`, `Speaker Display Name`, and `Portrait`. `Next State` is required. |
| `Dialogue: Choice` | Displays entries in `Choices`. Each entry has `Text`, `Enabled` (on by default), `Condition`, and `Target State`. `Choice Timeout` defaults to `-1` (no timeout); a non-negative value automatically selects `Choice Default Index`, initially `0`. |
| `Dialogue: Branch` | Routes to `True State` or `False State`. With `Use Condition` disabled (the default), `Default` (on by default) selects the route; with `Use Condition` enabled, the condition selects it. Both links are required. |
| `Dialogue: Trigger` | Emits `Event Name` and can execute an `Event Action Class`. `Next State` is required. |
| `Dialogue: End` | Ends the conversation. |
| `Dialogue: Set Variable` | Writes `Variable Name` and `Value` with `Scope` (`Local Instance`, `Global Dialogue Store`, or `Local + Global`), then follows `Next State`. |
| `Dialogue: Random` | Selects a weighted entry from `Entries`. Each entry has a `Weight` (initially `1.0`), optional `Label`, and `Target State`. Higher weight means higher relative probability. |
| `Dialogue: Wait` | Waits for `Duration` seconds (initially `1.0`), then follows `Next State`. |
| `Dialogue: Jump` | Jumps directly to `Target State`. |
| `Dialogue: Comment` | An editor annotation; do not use it as a playable step or link target. |
| `Dialogue: Chance` | Selects from `Entries` using `Chance` (initially `50`, range `0` to `100`), `Label`, and `Target State`. Values are relative weights; a total of `100` makes them intuitive to read as percentages. |
| `Dialogue: Emotion` | Sets `Speaker Id`, `Emotion`, and `Intensity` (initially `1`, range `0` to `1`), emits `On Emotion Changed`, then follows `Next State`. |
| `Dialogue: Check Visited` | Checks `Check State`, then follows `Visited State` or `Not Visited State`. Line and choice states are marked visited when displayed. |

## Line timing

In `Dialogue: Line`, `Auto Advance After` is measured in seconds:

- Greater than `0`: advances after that delay.
- Equal to `0`: waits for manual confirmation, even when `Voice Over` is assigned.
- Less than `0` (the default): uses the `Voice Over` duration if one is assigned; otherwise, waits for manual confirmation.

## Choices and conditions

A choice entry is shown only when `Enabled` is on and its `Condition` is true. The index passed to `Advance` or received by `On Choice Selected` is based on the **visible** list after filtering.

Conditions can test Boolean, integer, float, name, or string variables:

- Boolean and name: equal or not equal.
- Integer and float: equal, not equal, less, less than or equal, greater, or greater than or equal.
- String: equal, not equal, or contains. String comparisons are case-insensitive.

A missing variable or a variable with a different type makes the condition false. Initialize variables before using them in a condition.

## Validate links

Required links are `Next State` for `Line`, `Trigger`, `Set Variable`, `Wait`, and `Emotion`; `True State` and `False State` for `Branch`; each `Target State` for `Choice`, `Random`, and `Chance`; `Target State` for `Jump`; and all three links for `Check Visited`. Ensure each target has an enabled dialogue task. A missing link or invalid target prevents the conversation from starting correctly.
