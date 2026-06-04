# RapidGO v1.09.02 Release Notes

## Summary

RapidGO v1.09.02 includes updates to Receiving and Linebusting, with fixes for item entry performance, item description handling, display fields, and validation that helps prevent receiver posting issues in Counterpoint.

## Bug Fixes

### Receiving

Improved performance when adding items in Receiving, including items with many grid dimensions.

Fixed an issue where item descriptions could be blank when receivers were sent to Counterpoint. When a short/long description is not available, RapidGO now uses the default item description.

Fixed an issue where **Label Code** was not available as an option in **Change Display Information**.

### Linebusting

Fixed an object reference error that could occur during item entry when required pricing rule setup was missing or incomplete.

## Additional Improvements

### Receiving

Added validation to prevent inactive items from being added to receivers.

Added validation to prevent items from being added to a receiver when they are not stocked at the receiving location.

## Notes

Receiving now more closely follows Counterpoint behavior by preventing inactive or unstocked items from being added to receivers.
