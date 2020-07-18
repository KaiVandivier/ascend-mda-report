# ASCEND MDA Treatment Summary Report

This is an HTML template for a Standard Report in the DHIS2 software, designed to generate a custom report from numerous indicators and program indicators.

Some instructions for editing and maintenance are in comments in code.

TODO: Add more instructions for editing and maintenance here.

## Setting up rows, columns, and cells

### Rows

### Cells

When defining the contents of a cell, you have several options:

- You may use the name of the dimension to query from the database (like an indicator or program indicator, e.g. "TCMDA - Epidemiological Coverage").
  - To do so, set the value `dn` (short for "dimension name") property of the cell to the name of the dimension
  - Ex: `{ dn: "TCMDA - Epidemiological Coverage" }`
  - The script will look up the dimension ID based on the name and populate the `dId` property on the cell
  - The name must exactly match the name of the dimension in the database (it is case sensitive)
  - If the name is not found during the lookup, it will set a "Dimension not found" message as the value
  - Note that if you _also_ manually set the `dId` property of the cell, the script will not perform the name->ID lookup; i.e., the dimension ID will take precedence.
- You may use the ID of the dimensions you wish to query
  - Set the `dId` property (short for "dimension ID") of the cell
  - Ex: `{ dId: "kTtjN2SNEDH" }`
  - This has the benefits of avoiding string-matching sensitivity and being resilient to dimension name changes in the database, but may be tedious to look up individually for many cells.
  - This will override the dimension name (`dn`) property if it is provided.
- You may use custom logic to compute a cell's value based on other cells' values
  - Provide a function on the `customLogic` property that will be executed while the table is being populated with values that have been queried from the database.  The function should return the value that will ultimately be set as the cell's `value`.
  - The function will be called with two arguments: `cells`, an array of the cells in that row, and `idx`, the current cell's index in that array.
  - Helper functions `sumOf()` and `greatestOf()` may be useful for your custom logic.  They take any number of arguments and will operate on the arguments that are numbers (i.e. `!isNaN`), so it's safe to pass them cell values that may be strings or numbers.
  - Ex: This custom logic will sum up the values of the two previous cells in the row: `{ customLogic: (cells, idx) => sumOf(cells[idx - 1].value, cells[idx - 2].value) }`
  - Note that at execution time, the custom logic function will only have access to up-to-date values on the previous cells in the row.
- You may hard-code an explicit value for the cell
  - Set the `value` property of the cell.
  - Ex: `{ value: "This is the value that will end up in the table" }`
  - Note that this value will be overridden if any of the other properties above are set.
