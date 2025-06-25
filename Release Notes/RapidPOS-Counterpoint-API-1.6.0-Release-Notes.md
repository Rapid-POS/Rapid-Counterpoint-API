# Rapid Counterpoint API v1.6.0 Release Notes

_Released June 24, 2025_

## Store Tax Code Method Adherence

The API will now fully adhere to the **Store Tax Code method** setting configured in Counterpoint. This ensures that tax codes applied through the API align with the store's configuration.

### Where to Configure

In Counterpoint, navigate to:

- **Setup → Point of Sale → Stores → Main Tab → "Use tax code from"**

You can choose from:

- Store
- Customer

** For more details, review the official documentation:
[Counterpoint Store Tax Code Setting](http://ncrcounterpointhelp.com/#cs/frmpsstores/frmpsstores_edtartaxcod.htm) **

## API Behavior

- If you don’t supply a value for the tax code `PS_DOC_HDR.TAX_COD` in the `POST /Document` API call, the store tax code method will now be used.
  _(Previously, the code did not respect "Use Tax Code From" when it was set to "Customer")_
- Any tax code you pass in via the API will be validated. If the tax code does not exist in Counterpoint, the value falls back to the configured default _(See above)_
