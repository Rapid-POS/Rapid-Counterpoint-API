# Rapid CP API v1.08.00 Release Notes
**Release Date:** May 12th, 2026

## New Functionality

### Freight Allocation now respected
RapidGO now distributes Miscellaneous charges (e.g., freight) proportionally across receiver line items when posting to Counterpoint.

Allocation is based on the setting configured on the misc charge.

#### Example

- **Purchase Request:** 10 items totaling $1,000 + $100 freight
- Each item receives $10 freight allocation (1/10th)

If only 5 items are received:

- Total freight applied = $50
- Distributed across the 5 received items

#### Required Settings

When the settings below are enabled, Misc charges cost will be proportionally distributed across all receiver lines:

- **Use Misc Charge 1** = Checked
- **Include in Cost** = Checked
- **Allocate By** = Cost

Settings are sourced from Purchasing Control.

See:  
http://ncrcounterpointhelp.com/#3_back_office/purchasing/allocate_miscellaneous_charges.htm?Highlight=allocate%20misc

<br>

### Transfer To / Transfer From Workgroup Selection
In previous versions of RapidGO, users were unable to select their currently signed-in workgroup as a **Transfer To** location.

With this version of RapidGO, users can now select any location in either the **Transfer To** or **Transfer From** locations.

<br>

### Changing Receiving Header Information
RapidGO now allows users to edit receiver header information directly within the application.

Users can now modify:
- Reference numbers
- Receiver comments
- Additional header-level information associated with the receiver

This enhancement provides greater flexibility when updating receiving information without needing to return to CounterPoint.

<br>

### Modifying Receiving Line Comments
Users can now edit receiver line comments directly in RapidGO.

This allows warehouse and receiving staff to:
- Update line-specific notes during the receiving process
- Correct or add additional receiving details
- Maintain more accurate documentation on individual receiver lines

These updates are now synchronized as part of the receiving workflow.

<br>

### Linking Barcodes
RapidGO now supports linking new barcodes directly to item units within the application.

This enhancement improves efficiency when onboarding new inventory and managing alternate unit barcodes.
