# Changelog

The full change notes (in Indonesian) are also kept in the header of `src/PelicanCrossing.scl`.

## v0.8.5 – FB version 1.4
- **Fix:** crossing buzzer pattern restored to the v0.8.3 rhythm; only the last 5 s are faster: 1 s on / 1 s off for most of state 4, then 250 ms on / 250 ms off (`Clock_2Hz`, `%M0.3`) in the last 5 s (v0.8.3: 0.5 s / 0.5 s). v0.8.4 wrongly applied 250 ms to the whole state.
- No interface change from v0.8.4.

## v0.8.4 – FB version 1.3
- **Change:** (corrected in v0.8.5) the crossing buzzer beeps 250 ms on / 250 ms off for the whole of state 4. It uses the 2 Hz clock memory bit `%M0.3`; the buzzer output stays on `%Q0.6`.
- **New:** state indicator panel on an SM 1222 RLY module: states 0–6 light `%Q8.0`–`%Q8.6` (`%Q8.7` spare).
- New PLC tags: `Clock_2Hz` (`%M0.3`), `out_StateLamp_0`…`out_StateLamp_6` (`%Q8.0`–`%Q8.6`).
- FB interface changed (new input `Clock_2Hz`): the instance DB is reinitialised on download.

## v0.8.3 – FB version 1.2
- **New:** registers 24 / 25 expose violations of the running cycle per side, so the dashboard can show them before the end of state 5.
- **Fix:** Parade Lock now adds the running cycle's violations to the totals before clearing the counters (v0.8.2 lost them).
- **Fix:** Int overflow guard: when any violation total exceeds 30000, all three are reset to 0 (history is kept on the Jetson database).
- **Change:** `RD_LOC_T` called in formal form `RD_LOC_T(OUT => #dtLocalTime)`. No interface change from v0.8.2.

## v0.8.2 – FB version 1.1
- **Change:** "no excuses" violation rule: the first press in a cycle from either side is the request; every further press from either side, in any state of the running cycle, is a violation for that side. Simultaneous first presses count as one request.
- Per-side request latches from v0.8.1 removed; 50 ms debounce kept.
- FB interface changed: the instance DB is reinitialised on download.

## v0.8.1 – FB version 1.0
- **Fix:** violations counted per side with separate request latches (v0.8 counted side 2's first press as a violation after side 1 had requested).
- **New:** 50 ms debounce per button (`TON`), so contact bounce is not logged as a violation.
- FB interface changed: the instance DB is reinitialised on download.

## v0.8 – FB version 0.9
- **New:** crossing buzzer enabled from PLC start-up (Reg 8 := 1 in the OB1 initialisation). It can still be muted remotely.

## v0.7.1 – FB version 0.8
- **Fix:** Modbus watchdog drops the old socket every 10 s while idle (v0.7 did it only once).
- **Fix:** interlocks completed: jump to state 5 from vehicle green goes through amber and all red; jumps to 3/4/5 during amber finish amber first; jump to state 2 during crossing goes through clearance.
- **Fix:** Modbus idle counter clamped to avoid Int overflow after ~9 h.
- **Fix:** manual jump to state 0 / 6 now tallies pending violations and releases the button latch; jumps into states 1..5 keep the latch.

## v0.7 – Safety Interlock & Autonomous Fail-Safe
- **Safety 1:** manual jump to crossing while vehicles are green is routed through amber → all red → crossing.
- **Safety 2:** manual jump to vehicle green while pedestrians are crossing is routed through the all-red clearance.
- **Safety 3:** if Modbus is silent for ≥ 60 s, the PLC releases pause and parade lock and runs autonomously.
- **New:** register 23 resets the violation counters.

## v0.6
- **Fix:** Pause works in state 0 (Reg 11 default applied once at start-up, not every scan).
- **Fix:** flashing amber is a clean 0.5 s on / 0.5 s off.
- **Fix:** buzzer goes silent while paused.
- **Fix:** manual jump to state 0 clears the button latch.
- **Fix:** Modbus watchdog drops the connection after 10 s without requests (prevents hung socket, status 16#7002).
- **Fix:** default floodlight schedule only written at start-up or when all schedule registers are 0.
- **New:** Reg 20 output bitmask, Reg 21 seconds in current state, Reg 22 heartbeat.

## v0.5 (header: 0.5.1)
- Standby mode selectable: steady green or flashing amber.
- Parade / event lock (force state 0) and run / pause from the Jetson.
- Scheduled floodlight with on/off hour and minute set via registers 16–19.
- Two independent push buttons on opposite kerbs with separate violation counters.
- Remote buzzer control, run/pause at the current position and direct manual jump (0..6).
