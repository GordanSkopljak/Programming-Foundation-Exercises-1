# Lab Proof — HomeGuard Security System

**Program path:** `homeguard_system_f.ipynb`

**Run command:** `jupyter nbconvert --to notebook --execute --inplace homeguard_system_f.ipynb`
(equivalently: open the notebook and use "Run All" — every cell's output shown below is already baked into the committed file, run in order top to bottom.)

## What changed since the last submission

- **Step 2 added:** `create_sensor_dict()` and `create_alert_dict()` are dictionary-based
  structures used to build the end-of-run alert log and summary (previously skipped entirely).
- **Bug fixed:** temperature checks were collapsed into a single (60–75) range mislabeled
  SAFETY. They are now two independent checks: `isAbnormal()` tests the real safety thresholds
  (**< 35°F** frozen-pipe risk, **> 95°F** equipment-failure/fire risk) in every mode, and
  `isUncomfortable()` tests the comfort band (**65–75°F**) only when `system_mode == "HOME"`.
- **Test scenarios run:** all three required scenarios (Security/AWAY, Safety, Comfort/HOME)
  are executed with `forced_readings` so the output is deterministic and reproducible, not
  left to random chance.

## Fixed-case output

### Test 1 — Security (AWAY mode, door + motion trigger simultaneously)
```
=== HomeGuard Security System ===
Time: 23:35:13
Mode: AWAY

[READING] Living Room Motion: No activity
[READING] Front Door Door: CLOSED
[READING] Kitchen Temperature: 70
[READING] Bedroom Smoke: CLEAR

Time: 23:35:13
[READING] Living Room Motion: DETECTED
[ALERT!] ⚠️ MEDIUM: SECURITY: Motion detected in Living Room!
[LOG] [23:35:13] Sending notification to homeowner...
[READING] Front Door Door: OPENED
[ALERT!] 🚨 HIGH: SECURITY: Front Door opened while in AWAY mode!
[LOG] [23:35:13] Sending notification to homeowner...
[READING] Kitchen Temperature: 70
[READING] Bedroom Smoke: CLEAR
[ALERT!] 🚨 HIGH: SECURITY: Door AND motion triggered in the same cycle - possible break-in!
[LOG] [23:35:13] Escalating to emergency contact...

--- Alert Summary ---
MEDIUM: 2
HIGH: 4
```

### Test 2 — Safety (frozen pipe, equipment failure, smoke)
```
=== HomeGuard Security System ===
Mode: HOME

[READING] Kitchen Temperature: 20
[ALERT!] 🚨 HIGH: SAFETY: Kitchen temperature 20F - frozen pipe risk!
[LOG] [23:35:13] Sending notification to homeowner...

[READING] Kitchen Temperature: 105
[ALERT!] 🚨 HIGH: SAFETY: Kitchen temperature 105F - equipment failure/fire risk!
[LOG] [23:35:13] Sending notification to homeowner...

[READING] Bedroom Smoke: DETECTED
[ALERT!] 🚨 HIGH: SAFETY: Smoke detected in Bedroom!
[LOG] [23:35:13] Sending notification to homeowner...

--- Alert Summary ---
HIGH: 3
```

### Test 3 — Comfort (HOME mode, temperature outside 65–75°F but inside the safety range)
```
=== HomeGuard Security System ===
Mode: HOME

[READING] Kitchen Temperature: 80
[NOTICE] 📋 LOW: COMFORT: Kitchen temperature 80F is outside the comfort range.

[READING] Kitchen Temperature: 60
[NOTICE] 📋 LOW: COMFORT: Kitchen temperature 60F is outside the comfort range.

[READING] Kitchen Temperature: 70
--- Alert Summary ---
No alerts were triggered this run.
```
Note: 80°F and 60°F trip the **comfort** notice only — neither is a SAFETY alert, which
proves the two thresholds are now independent (this is exactly the bug the previous
grade flagged).

## Edge case handled explicitly

**Simultaneous door + motion trigger while AWAY.** A single opened door or a single motion
reading during AWAY mode is treated as a normal (if urgent) security alert. But if a motion
sensor *and* the door sensor both trip in the **same reading cycle**, the system raises an
additional escalated alert ("possible break-in") and logs an "Escalating to emergency
contact..." event, on top of the two individual alerts. This matters because the lab's
requirements explicitly call out "multiple sensors triggered simultaneously (possible
break-in)" as a distinct case from a single sensor tripping — treating them identically
would under-communicate the severity of a likely intrusion to the homeowner. This is
visible in Test 1, iteration 2 above.
