# Sonos Receiver Sync

A Home Assistant blueprint that makes a **Sonos Connect / Port** and the **AV receiver** it feeds behave like one device.

Press play on Sonos and the receiver powers on, switches to the right input, and matches volume. Stop playing and the receiver shuts itself off. No remote, no input hunting, no receiver left running all night.

[![Open your Home Assistant instance and show the blueprint import dialog.](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fraw.githubusercontent.com%2Fpartytimeexcellent%2Fhome-assistant%2Fmain%2Fsonos_receiver_sync.yaml)

## Why you might want this

If your Sonos Connect is wired into a receiver, you've probably lived with this routine: turn on the receiver, pick the Sonos input, then open the Sonos app and play something. And later, forget to turn the receiver off.

This blueprint removes those steps.

| You do this | The blueprint does this |
|---|---|
| Start playing on Sonos (app, voice, automation, anything) | Turns the receiver on, waits for it to boot, selects the Sonos input, and copies the Sonos volume to the receiver |
| Change volume on either device | Mirrors it to the other one |
| Stop or pause Sonos | Turns the receiver off after a delay (default 5 minutes), but only if it's still on the Sonos input |
| Switch the receiver to a different input (TV, turntable, etc.) | Pauses the Sonos so it isn't playing to nobody |

It works with any receiver that Home Assistant exposes as a `media_player` supporting `turn_on`, `turn_off`, `select_source` and `volume_set`.

## What's new?

This is an updated version of the original [Sonos Connect Sync by Qonstrukt](https://gist.github.com/Qonstrukt/ca1e761b2ec0a2d52fdb8c86490fbcbd), rewritten because the original stopped working on current Home Assistant releases.

- **Actually turns the receiver on.** The original only selected a source and assumed the receiver would wake up. Many don't. Sonos Receiver Sync sends `turn_on`, waits for the receiver to report it's on, then waits a configurable settle time before selecting the input.
- **Modern syntax.** Uses `triggers:`, `actions:`, `action:`, `target:` and the `filter:` entity selector. Requires Home Assistant 2024.10.0 or newer.
- **Safer volume sync.** Skips syncing when the receiver is off or on another input, and ignores tiny differences (default 2%) so two devices with slightly different volume rounding don't bounce values back and forth.
- **Guards against unavailable devices.** Checks for `off`, `unavailable` and `unknown` states before sending commands, so it doesn't throw errors while a device is booting or offline.
- **Queued runs.** `mode: queued` means quick successive events are handled in order instead of dropped.

## Requirements

- Home Assistant **2024.10.0** or newer
- The [Sonos integration](https://www.home-assistant.io/integrations/sonos/) with your Sonos Connect / Port set up
- A receiver available as a `media_player` entity in Home Assistant, with a source (input) that the Sonos is plugged into

## Installation

### One click

Click the **Import blueprint** badge at the top of this page, then follow the prompts.

### Manual

1. Download `sonos_receiver_sync.yaml`.
2. Copy it to `config/blueprints/automation/<your_folder>/sonos_receiver_sync.yaml` in your Home Assistant configuration.
3. Go to **Settings → Automations & Scenes → Blueprints** and confirm **Sonos Receiver Sync** is listed. If it isn't, check **Settings → System → Logs** for a blueprint error.

## Setup

1. Go to **Settings → Automations & Scenes → Create Automation → Use Blueprint**.
2. Choose **Sonos Receiver Sync**.
3. Fill in the inputs (below) and save.

**Important:** the receiver source name is **case-sensitive** and must exactly match an entry in the receiver's `source_list` attribute. To find it, open **Developer Tools → States**, select your receiver, and look at `source_list`. For example, it may be `SONOS`, `Sonos`, or `AUDIO2`, depending on the receiver. A mismatch is the most common reason nothing happens.

## Inputs

| Input | Default | Description |
|---|---|---|
| **Sonos** | — | The Sonos Connect / Port `media_player`. |
| **Receiver** | — | The receiver `media_player` the Sonos is connected to. |
| **Receiver source name** | `SONOS` | Exact, case-sensitive name of the receiver input the Sonos is plugged into. |
| **Receiver turn off delay** | 300 s | How long the Sonos must stay stopped before the receiver turns off. |
| **Receiver power-on settle time** | 3 s | Extra wait after the receiver reports it's on, before selecting the source. Raise this if the source selection is ignored right after power-on. |
| **Receiver power-on timeout** | 30 s | Maximum time to wait for the receiver to report that it's on. |
| **Synchronise volume** | On | Mirror volume changes between Sonos and the receiver. |
| **Volume sync buffer time** | 1000 ms | How long a volume change must hold steady before it's copied. |
| **Volume tolerance** | 0.02 | Differences smaller than this are ignored (0.02 = 2%). |

## How it works

The blueprint has five triggers, each handled by its own branch:

1. **Sonos starts playing** → turn the receiver on if it's off, wait for it, select the Sonos input, then copy the Sonos volume.
2. **Sonos leaves `playing` for the turn-off delay** → turn the receiver off if it's still on the Sonos input.
3. **Receiver source changes away from the Sonos input** → pause the Sonos.
4. **Sonos volume changes** → set the receiver to match (only while the receiver is on the Sonos input).
5. **Receiver volume changes** → set the Sonos to match (same condition).

## Troubleshooting

**Nothing happens when Sonos plays.**
Check the source name first (see the note under Setup). Then open the automation and view its **Traces** to see which step failed.

**The receiver turns on but stays on the wrong input.**
Increase **Receiver power-on settle time**. Some receivers ignore input changes for a few seconds after waking.

**The receiver turns off while I'm still listening.**
Increase **Receiver turn off delay**. Pausing longer than the delay counts as stopped.

**Volume keeps jumping around.**
Increase **Volume tolerance** or **Volume sync buffer time**.

**The blueprint doesn't show up after import.**
Home Assistant drops blueprints with schema errors. Look in **Settings → System → Logs** for `blueprint`. Make sure you're on 2024.10.0 or newer.

**I have another automation that also controls the receiver.**
Disable it. Two automations reacting to the same Sonos state change will conflict.

## Credits

Based on the original *Sonos Connect Sync* blueprint by [Qonstrukt](https://gist.github.com/Qonstrukt/ca1e761b2ec0a2d52fdb8c86490fbcbd). Sonos Receiver Sync updates it for current Home Assistant and adds the power-on handling and volume safeguards described above.

## License

Add a license of your choice here (MIT is a common default for blueprints).
