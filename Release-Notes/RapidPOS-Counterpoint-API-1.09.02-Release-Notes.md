
# Rapid CounterPoint API V1.08.04 Release Notes COMING SOON
**Release Date:** TBD
_Bug fix ensuring receiver totals are calculated accurately at upload time._

## Bug Fixes

### `POST /Receivers` — Receiver total excludes miscellaneous charges at creation time
Receivers created via `POST /Receivers` returned a header `Total` equal to the subtotal only; the `Total Misc` (miscellaneous charges) was not included until the receiver was subsequently edited and saved in CounterPoint.

The `Total` field in the receiver response now reflects the correct combined value of subtotal + `Total Misc` immediately upon creation, with no manual refresh required in CounterPoint.
````
