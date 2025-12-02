# Multi-Zone Presence and Arrival Logic for Home Assistant

## Overview

This Home Assistant setup uses **multi-zone presence tracking** combined with **per-person arriving flags** to reliably detect when someone is approaching or arriving home, while preventing false triggers caused by GPS drift or driving past the house.

It is designed for households with multiple occupants and supports automations like **lights, night mode, media, and last-person-home logic**.

---

## Zones

1. **Home Zone**
   - Radius: 50 m (adjustable)
   - Purpose: Detect *true arrival* at home
   - Triggers home arrival automations

2. **Near Home Zone**
   - Radius: 300 m (adjustable)
   - Purpose: Detect when someone is near home, but not yet arrived
   - Used to determine “approaching” status
   - Prevents false arrival triggers when someone briefly passes through the Home zone

---

## Per-Person Arriving Flags

Each household member has their own `input_boolean` to track arrival intent:

- `input_boolean.rick_arriving`
- `input_boolean.wife_arriving`
- *(Add more for additional household members)*

### Behavior:

| Transition | Action on `<person>_arriving` |
|------------|-------------------------------|
| `not_home → Near Home` | True |
| `home → Near Home` | False |
| `Near Home → not_home` | False |
| `Near Home → home` | False |
| `Near Home → home` AND `<person>_arriving == True` | Someone arrived home, trigger automations |

---

## Shared Arrival Detection

A central boolean (`input_boolean.a_person_arriving`) is set to **True** if any individual arrival flag is True:


This shared flag is used to determine if someone is truly arriving home for shared automations.

---

## State Logic Overview

| Large Zone (Near Home) | Small Zone (Home) | Resulting State | Notes |
|------------------------|------------------|----------------|-------|
| ❌ Outside             | ❌ Outside       | AWAY           | Person is fully away |
| ✔ Inside               | ❌ Outside       | NEAR HOME      | Approaching home, arrival boolean may be True |
| ✔ Inside               | ✔ Inside        | HOME           | Arrival confirmed if arrival boolean is True |
| ❌ Outside             | ✔ Inside        | IGNORED        | Drive-through scenario; prevents false triggers |

---

## Workflow

### Leaving Home
1. Exit Home zone → may still be considered near home (arrival flag remains True if approaching).  
2. Exit Near Home zone → confirmed AWAY; reset individual arrival flag.

### Returning Home
1. Enter Near Home zone → set arrival flag = True.  
2. Enter Home zone → if arrival flag = True, confirm arrival and trigger automations.  
3. Reset arrival flag to False after arrival.

---

## Benefits

- Prevents false arrivals caused by GPS drift or brief pass-by events  
- Supports multiple household members independently  
- Centralized shared arrival detection for automations  
- Scalable for additional people or zones  

---

## Optional Enhancements

- Add a short delay (30–60s) before marking “arrived” to smooth GPS fluctuations  
- Integrate presence with media devices, night mode, or “last person home” logic  
- Visualize zones with a simple map or diagram for easier understanding  

---

## Notes

This system has been beta-tested to reliably track presence and arrivals. Future testing will refine thresholds, delays, and edge case handling.
