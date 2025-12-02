Multi-Zone Presence and Arrival Logic README
Overview

This Home Assistant setup uses multi-zone presence tracking combined with per-person arriving flags to reliably detect when someone is approaching or arriving home, while preventing false triggers caused by GPS drift or driving past the house.

It is designed for households with multiple occupants and supports automations like lights, night mode, media, and last-person-home logic.

Zones

Home Zone

Radius: 50 m (adjustable)

Purpose: Detect true arrival at home

Triggers home arrival automations

Near Home Zone

Radius: 300 m (adjustable)

Purpose: Detect when someone is near home, but not yet arrived

Used to determine “approaching” status

Prevents false arrival triggers when someone briefly passes through the Home zone

Per-Person Arriving Flags

Each household member has their own input_boolean to track arrival intent:

input_boolean.rick_arriving

input_boolean.wife_arriving

(Add more for additional household members)

Behavior:
Transition	Action
not_home → Near Home	Set <person>_arriving = True
home → Near Home	Set <person>_arriving = False
Near Home → not_home	Set <person>_arriving = False
Near Home → home	Set <person>_arriving = False
Near Home → home AND <person>_arriving = True	Confirm arrival, trigger home automations
Shared Arrival Detection

A central boolean (input_boolean.a_person_arriving) is set to True if any individual arrival flag is True:

a_person_arriving = rick_arriving OR wife_arriving OR ...


This shared flag is used to determine if someone is truly arriving home for shared automations.

State Logic Overview
Large Zone (Near Home)	Small Zone (Home)	Resulting State	Notes
❌ Outside	❌ Outside	AWAY	Person is fully away
✔ Inside	❌ Outside	NEAR HOME	Approaching home, arrival boolean may be True
✔ Inside	✔ Inside	HOME	Arrival confirmed if arrival boolean is True
❌ Outside	✔ Inside	IGNORED	Drive-through scenario; prevents false triggers
Workflow
Leaving Home

Exit Home zone → may still be considered near home (arrival flag remains True if approaching).

Exit Near Home zone → confirmed AWAY; reset individual arrival flag.

Returning Home

Enter Near Home zone → set arrival flag = True.

Enter Home zone → if arrival flag = True, confirm arrival and trigger automations.

Reset arrival flag to False after arrival.

Benefits

Prevents false arrivals caused by GPS drift or brief pass-by events

Supports multiple household members independently

Centralized shared arrival detection for automations

Scalable for additional people or zones
