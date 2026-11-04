# Bluetooth Transfer Unlock

**Note: the author only knows a little about Android and supplied the ideas — every change in this repository is AI-generated. Review the code before using it, because these hooks touch the system Bluetooth stack and can cause irreversible damage.**

[中文](README.md) | English

Lifts the Bluetooth restrictions on sending and receiving by file type, plus invisibility outside the pairing screen: add Bluetooth to any app's share sheet for every file type if you only want that. Selecting Bluetooth alone as the scope is enough — `system` is not needed.

A LSPosed module on the modern libxposed API (102): no UI, no settings screen. Enable the module, tick the scope, restart the target process.

| Feature | In one line |
| :- | :- |
| 1 | inbound transfers are no longer refused by the MIME accept-list |
| 2 | outbound shares no longer break on a missing or unknown type |
| 3 | permanently discoverable: asserted as soon as Bluetooth is on — no pairing screen, no timeout downgrade |
| 4 | Bluetooth appears in **every** app's share sheet, not only for zip / image / video |

> **Risk:** feature 3 leaves the device discoverable whenever Bluetooth is on (switch it off when you don't want to be found); feature 4 is a global system_server hook that affects every share sheet on the device.

## Enabling

The scope is pre-declared inside the APK and can still be ticked per entry in LSPosed:

- `com.android.bluetooth` — all of features 1-3. **If you only want those three, this one entry is enough.**
- `system` — the only switch for feature 4 (intent resolution runs in system_server). Leave it unticked if you don't need it.

Restart: features 1-3 only need the Bluetooth process restarted; **feature 4 requires a full reboot** — system_server does not reload the module just because the Bluetooth process restarted.

## Diagnostics

```bash
adb logcat -s BTUnlock
adb shell setprop log.tag.BTUnlock VERBOSE   # set this before the target process starts
```

- `[com.android.bluetooth] hooks installed: receive=N, send=N, discoverable=N` and `[system_server] hooks installed: discoverable-fallback=N, share-sheet=M` — these two lines say what got installed.
- No Bluetooth row in the sheet: no `[system_server]` line at all → the scope lacks `system` (tick it, then reboot).
- Not discoverable: `asserted mode … (found 21, watchdog)` every 5 s without end means the assert goes out but the stack ignores it; normally it appears once or twice per boot.

## Known limitations

1. When the sharing app supplies no type, the fill-in happens only inside the Bluetooth process; the row in the sheet comes from feature 4 and runs after you tap it.
2. No Bluetooth row while Bluetooth is off (the activity is disabled) — by design. A custom sheet with its own hard-coded list that never queries the resolver will also miss it.
3. Feature 3 depends on the Bluetooth process being alive: up to a 5 s gap while it restarts.
4. No UI and no notifications.
5. Real-device testing covers one Android 10 and one Android 16 phone; Android 11-15 relies on the same shape matching derived from AOSP source and is not individually verified.

Source and build instructions: <https://github.com/luoyangshovel/Bluetooth_Transfer_Unlock>
