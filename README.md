# FG SimBrief Importer Add-On

## What Is This?

This is an add-on for FlightGear that adds a SimBrief Import dialog.

(See [https://www.simbrief.com/](https://www.simbrief.com/)).

This repository contains a hardened and maintained version of the original
addon, updated for improved stability and compatibility with modern
FlightGear builds.

## What It Does

The SimBrief importer can import various aspects of a SimBrief flight plan into
FlightGear. The import functionality attempts to support as many aircraft types
as possible, but due to the nature of the beast, it does not work equally well
with all of them.

Compared to the original version, this build:

- avoids route parsing hangs on problematic navdata
- safely skips invalid waypoints instead of aborting
- improves compatibility with newer FlightGear versions
- hardens error handling and logging

To use it, you need to create an account on [https://www.simbrief.com/](https://www.simbrief.com/). Then:

1. Create a new flight in the SimBrief "dispatch" system. Generate the OFP for
   it.
2. In FlightGear, select "Equipment" > "SimBrief Import" in the main menu.
3. Enter your SimBrief username
4. Select which parts of the flight plan you want to import (see below)
5. Click "Import" and wait until it says "All Done".

Available options:

- Flight Plan Route: This one imports the departure and destination airports,
  and all enroute waypoints, into the default flightplan. If "Activate
  immediately" is checked, it will also activate the flightplan.
- Departure RWY, SID: This will attempt to select the planned departure runway,
  and the planned SID, from the flightplan. Note that this will only work if
  your FlightGear navdata matches the selections from SimBrief, which may not
  be the case, especially if you're using the default FG navdata and/or an
  outdated AIRAC cycle in SimBrief. SID selection will also fail if SimBrief
  uses a different naming convention than FG's navdata.
- Arrival RWY, STAR: This will attempt to select the planned arrival runway and
  STAR. The same caveats apply as with the departure runway and SID.
- Performance Init: This sets a handful of key performance parameters;
  currently: cruise altitude and callsign.
- Activate immediately: with this checkmark on, the imported flight plan will
  immediately become the active flight plan after a successful import. Without
  it, the flight plan will be staged as the "modified flightplan" first. This
  only really makes sense in aircraft that support this properly, and allow
  you to review the modified flightplan before activating it.
- Payload: Imports passenger and cargo weights, and attempts to distribute them
  sensibly over available payload weight slots.
- Fuel: Imports block fuel as per the flightplan, and attempts to distribute it
  sensibly over available fuel tanks. Two fuel allocation strategies are
  provided:
  - "Balanced": distributes required fuel proportionally over all tanks,
    according to their declared capacities - large tanks get more fuel, small
    tanks get less fuel.
  - "First come, first serve": distributes fuel by filling tanks in declaration
    order. Some crude heuristic is in place that attempts to detect left/right
    pairs of wing tanks from their names, to keep things balanced, but it is
    not foolproof.
    Which of these strategies is more suitable depends on the aircraft type. If
    neither is good enough, but the aircraft comes with a "balance fuel" button,
    the recommended way is to use either fuel strategy, and then simply press
    the "balance fuel" button to let the aircraft sort it out.

- Winds Aloft: Runs a background process that sets winds aloft according to the
  forecast winds in the flightplan. This will only work with Basic Weather,
  since the Advanced Weather engine runs its own wind simulation that will
  overwrite winds aloft regardless of what we set. The "Start" and "Stop"
  buttons allow you to manually start and stop the winds updater.

## Aircraft Developer Information

The add-on should do a decent job on most aircraft types as long as you use the
standard built-in features. Some tips for making your aircraft maximally
compatible:

- If your aircraft supports flight plan staging ("ACTIVE" vs. "MODIFIED" flight
  plans), then you can support this by providing custom implementations of the
  following two Nasal functions, which must be in the global namespace:

  `globals.getFlightplan(index=0)` - `index` is 0 for the active flightplan,
  1 for the modified (staged) flightplan.

  `globals.commitFlightplan()` - activates the modified flightplan, if any,
  and triggers the necessary updates to your instrumentation.

- To make fuel allocation work, it is recommended that tanks that come in
  left/right pairs include the words "Left" and "Right", respectively, in their
  names, and listed consecutively. This way, the fuel import system will keep
  their fuel levels synchronized. It is further recommended to list fuel tanks
  in order of priority: this way, the "first-come-first-serve" strategy will do
  the right thing, topping up the highest-priority tanks first, before adding
  fuel to the additional tanks.

- Passengers and cargo will be distributed over all weight nodes whose names
  look like they're either passenger spaces or cargo holds. Weight nodes that
  contain any of the words "passenger", "cabin", "pax", "class", "baggage" or
  "seat" are considered passenger spaces; weight nodes that contain any of the
  words "cargo" or "payload" are considered cargo holds.

- If your aircraft requires a custom SimBrief import, simply create a global
  Nasal object or namespace named 'simbrief'; the addon will see that and
  disable itself.
