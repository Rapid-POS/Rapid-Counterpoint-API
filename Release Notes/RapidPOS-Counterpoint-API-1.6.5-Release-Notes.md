# Rapid Counterpoint API v1.6.5 Release Notes

_Released July 22, 2025_

## Inventory Adjustment: Serialized Item Fix

The behavior of the `InventoryAdjustments` endpoint has been updated to ensure that the serial number is properly committed to the item during adjustments.

This change resolves a check constraint violation error that previously occurred when deleting an unposted inventory adjustment for a serialized item in Counterpoint.

## Receiver Enhancements – Changes to the API required for new RapidGO Functionality

The `Add Receiver` service has been updated to support new RapidGO functionality for creating multiple receivings from the same purchase order without posting.

This enhancement allows multiple, unposted receiving sessions to be created from a single purchase order, enabling more flexible receiving workflows.
