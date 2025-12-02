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

