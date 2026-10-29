# Modbus TCP Register Map

Source: `"DB_Jetson_Comm".Holding_Data` (`Array[0..29] of Int`), served by `MB_SERVER` in OB1.

| Parameter | Value |
|---|---|
| Port | 502 |
| Unit / slave ID | 1 |
| Read | FC03 Read Holding Registers |
| Write | FC06 Write Single Register / FC16 Write Multiple Registers |
| Data type | 16-bit signed Int (a value of -1 reads as 65535 if read unsigned) |
| Clients | One at a time (`CONNECT_ID 1`) |
| Recommended poll interval | 0.2 – 1 s (must stay well below the 10 s watchdog) |

## Holding registers

"Address" is 0-based. The 4xxxx column is the 1-based convention (address + 40001).
If values look shifted by one in a client tool, switch its 0-based / 1-based setting.

| Addr | 4xxxx | Name | Values | Direction | Value at PLC start |
|---|---|---|---|---|---|
| 0 | 40001 | Active state | 0 standby, 1 request, 2 amber, 3 all red 1, 4 crossing, 5 all red 2, 6 cooldown | PLC → client | – |
| 1 | 40002 | Total violations | Reg 14 + Reg 15 | PLC → client | 0 |
| 2 | 40003 | State 1 duration | seconds | client → PLC | 10 if ≤ 0 |
| 3 | 40004 | State 2 duration (amber) | seconds | client → PLC | 3 if ≤ 0 |
| 4 | 40005 | State 3 duration (all red 1) | seconds | client → PLC | 3 if ≤ 0 |
| 5 | 40006 | State 4 duration (crossing) | seconds | client → PLC | 20 if ≤ 0 |
| 6 | 40007 | State 5 duration (all red 2) | seconds | client → PLC | 3 if ≤ 0 |
| 7 | 40008 | State 6 duration (cooldown) | seconds | client → PLC | 60 if ≤ 0 |
| 8 | 40009 | Crossing buzzer | 0 mute, 1 on | client → PLC | 1 |
| 9 | 40010 | Standby mode | 0 green, 1 flashing amber | client → PLC | DB start value |
| 10 | 40011 | Parade lock | 1 hold in state 0, 0 normal | client → PLC | DB start value; set to 0 by fail-safe |
| 11 | 40012 | Run / pause | 1 run, 0 pause | client → PLC | 1; set to 1 by fail-safe |
| 12 | 40013 | Manual state jump | 0..6 command, -1 idle (auto-cleared by PLC) | client → PLC | -1 |
| 13 | 40014 | Floodlight mode | 0 auto (RTC schedule), 1 force on, 2 force off | client → PLC | DB start value |
| 14 | 40015 | Violations, side 1 | accumulated, button `%I0.0` | PLC → client | 0 |
| 15 | 40016 | Violations, side 2 | accumulated, button `%I0.1` | PLC → client | 0 |
| 16 | 40017 | Floodlight on – hour | 0..23 | client → PLC | 17 if regs 16–19 are all 0 |
| 17 | 40018 | Floodlight on – minute | 0..59 | client → PLC | 30 if regs 16–19 are all 0 |
| 18 | 40019 | Floodlight off – hour | 0..23 | client → PLC | 6 if regs 16–19 are all 0 |
| 19 | 40020 | Floodlight off – minute | 0..59 | client → PLC | 0 if regs 16–19 are all 0 |
| 20 | 40021 | Output bitmask | see below | PLC → client | – |
| 21 | 40022 | Seconds in current state | stays 0 in state 0 | PLC → client | 0 |
| 22 | 40023 | Heartbeat | +1 every second, wraps after 32000 | PLC → client | 0 |
| 23 | 40024 | Reset violations | write 1 to clear regs 1, 14, 15 (auto-cleared) | client → PLC | 0 |
| 24 | 40025 | Violations in running cycle, side 1 | added to reg 14 at the end of state 5 | PLC → client | 0 |
| 25 | 40026 | Violations in running cycle, side 2 | added to reg 15 at the end of state 5 | PLC → client | 0 |
| 26–29 | 40027–40030 | Reserved | – | – | – |

Registers 12 and 23 are one-shot commands: the PLC clears them on the next scan.

## Register 20 – output bitmask

| Bit | Value | Output | Meaning |
|---|---|---|---|
| 0 | 1 | `%Q0.0` | Vehicle green |
| 1 | 2 | `%Q0.1` | Vehicle amber |
| 2 | 4 | `%Q0.2` | Vehicle red |
| 3 | 8 | `%Q0.3` | Pedestrian green |
| 4 | 16 | `%Q0.4` | Pedestrian red |
| 5 | 32 | `%Q0.5` | Spare (never set) |
| 6 | 64 | `%Q0.6` | Buzzer |
| 7 | 128 | `%Q0.7` | Floodlight |

Typical values (floodlight off, buzzer ignored):

| State | Reg 20 | Lights |
|---|---|---|
| 0, 1, 6 (green standby) | 17 | Vehicle green + pedestrian red |
| 2 | 18 | Vehicle amber + pedestrian red |
| 3, 5 | 20 | Vehicle red + pedestrian red |
| 4 | 12 | Vehicle red + pedestrian green (+64 while the buzzer sounds) |

Add 128 when the floodlight is on. In flashing-amber standby the value toggles every 0.5 s.
Register 20 reflects the program's outputs, not the physical lamps (a blown lamp is not detected).

## Watchdog and fail-safe

| Condition | PLC action |
|---|---|
| 10 s without a Modbus request (repeats every 10 s) | Drops the current connection (`DISCONNECT`) so a hung socket can reconnect |
| 60 s without a Modbus request | Reg 10 := 0 (release parade lock), Reg 11 := 1 (run), Reg 12 := -1 (cancel manual jump) |

## Button violation rule (v0.8.2+)

- The **first** press in a cycle, from either side, is a valid request and is not counted.
- **Every later press** during the running cycle, from either side, counts as a violation for the side that pressed.
- Both sides pressing in the same scan with no request pending count as a single request.
- Each button is debounced for 50 ms.
