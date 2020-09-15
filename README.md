# ASCEND MDA Treatment Summary Report

This is an HTML template for a Standard Report in the DHIS2 software, designed to generate a custom report from numerous indicators and program indicators. [Read more about HTML-based standard reports here.](https://docs.dhis2.org/master/en/user/html/designing-html-based-standard-reports.html)

## Setting up columns, rows, and cells

### Columns

Define column headers in the `columns` array. Each column is an object:

- It should have `name` and `shortName` properties. The short name will be used unless the column has multiple subcategories.
- A column may have an array of `subcategories`, which will be displayed as subheadings.
  - A subcategory should be an object with `name` and `shortName` properties. Only the short name will be displayed on the table.
  - A column with subcategories will result in a number of columns in the table equal to the number of subcategories, and the column title header will span those columns.
  - A few reusable subcategories are included as an enum (`colSubcats`) in the template.

An example column:

```javascript
const columns = [
  // ...
  {
    name: "Onchocerciasis",
    shortName: "Oncho.",
    subcategories: [colSubcats.ROUND_1, colSubcats.ROUND_2, colSubcats.TOTAL],
  },
  // ...
];
```

### Rows

Row definitions live in the `rows` array, an array of objects.

Row objects use these properties:

- `name`: The name of the row, which will be printed as the row header in a `<th>` cell.
- `type`: Defines the type of the row, using values set in an enum:
  - `rowTypes.CATEGORY` indicates that this row is a label for a group of following rows, and a category row will make one big cell that spans the width of the table. A category row should not have a `cells` property.
  - `rowTypes.DATA` indicates this row will be filled with data cells. This row should define the cells in the `cells` property.
- `cells`: An array that will define the contents of each cell in the row. See the following section for cell definitions.

### Cells

Cells are defined by an array for each row. The array should be an appropriate size to fill the table, and the order of the cells should match the column definitions.

Enter `null` to create an empty cell, or define the contents of the cell with an object. When defining the contents of a cell, you have several options: you can specify a dimension name (`dn`), a dimension ID (`dxId`), custom logic (`customLogic`, a function), or a hard-coded value - see this example of a data row and explanations of each option below:

```javascript
const rows = [
  // ...
  {
    name: "Number of IUs which reached the criteria to stop MDA",
    type: rowTypes.DATA,
    cells: [
      { dn: "TCMDA - IUs which reached the criteria to stop MDA", },
      { dxId: "d3AglBM9nF4", },
      { value: "I will end up in the table!", }
      null,
      { dn: "OMDA - IUs which reached the criteria to stop MDA", },
      null,
      null,
      { dn: "SCMDA - IUs which reached the criteria to stop MDA", },
      null,
      null,
      { dn: "STMDA - IUs which reached the criteria to stop MDA", },
      {
        customLogic: (cells, idx) =>
          sumOf(
            cells[0].value,
            cells[1].value,
            cells[4].value,
            cells[7].value,
            cells[10].value
          ),
      },
    ],
  },
  // ...
]
```

#### Ways to define cell contents

- Using a dimension name, like "TCMDA - Epidemiological Coverage" (can be an indicator or program indicator):

  - To do so, set the value `dn` (short for "dimension name") property of the cell to the name of the dimension
  - Ex: `{ dn: "TCMDA - Epidemiological Coverage" }`
  - The script will look up the dimension ID based on the name and populate the `dxId` property on the cell
  - The name must exactly match the name of the dimension in the database (it is case sensitive)
  - If the name is not found during the lookup, it will set a "Dimension not found" message as the value
  - Note that if you _also_ manually set the `dxId` property of the cell, the script will not perform the name->ID lookup; i.e., the dimension ID will take precedence.

- Using a dimension ID:

  - Set the `dxId` property (short for "dimension ID") of the cell
  - Ex: `{ dxId: "kTtjN2SNEDH" }`
  - This has the benefits of avoiding string-matching sensitivity, being resilient to dimension name changes in the database, and the ability to correctly specify an indicator if there are duplicate names, but it may be tedious to look up IDs individually for many cells.
  - This will override the dimension name (`dn`) property if it is provided.

- Using custom logic to compute a cell's value based on other cells' values:

  - Provide a function on the `customLogic` property that will be executed while the table is being populated with values that have been queried from the database. The function should return the value that will ultimately be set as the cell's `value`.
  - Read more in the [custom logic section](#custom-logic-functions-for-cells) below

- Hard-coding an explicit value for the cell:
  - Set the `value` property of the cell.
  - Ex: `{ value: "This is the value that will end up in the table" }`
  - Note that a hard-coded value will be overridden if any of the other properties above are set.

#### Custom logic functions for cells

Here are some details about using custom logic functions:

- The function will be called with two arguments: `cells`, an array of the cells in that row, and `idx`, the current cell's index in that array.
- The function can be asynchronous and return a promise; the return value will be `await`ed upon.
- Note that at execution time, the custom logic function will only have access to up-to-date values on the previous cells in the row.

Here is an example in which the custom logic function returns the sum of the previous two cells in a row:

```javascript
const rows = [
  {
    name: "Row with custom logic example",
    type: rowTypes.DATA,
    cells: [
      // ...
      {
        customLogic: (cells, idx) =>
          sumOf(cells[idx - 1].value, cells[idx - 2].value),
      },
    ],
  },
  // ...
];
```

A few reusable functions are in the template for custom logic that you might want to apply the cells:

##### sumOf(...args)

This function receives any number of arguments and returns the sum of any numbers that were passed in to the arguments. Because it filters out non-number inputs, it's safe to pass it values with an unknown type. Here is an example where the function sums up the values in known columns:

```javascript
const rows = [
  {
    name: "Row with sumOf() example",
    type: rowTypes.DATA,
    cells: [
      // ...
      {
        customLogic: (cells, idx) =>
          sumOf(
            cells[0].value,
            cells[1].value,
            cells[4].value,
            cells[7].value,
            cells[10].value
          ),
      },
    ],
  },
  // ...
];
```

##### greatestOf(...args)

This returns the greatest number in the arguments passed to it. Like `sumOf()`, this function receives any number of arguments, and filters out non-number values. If there are no numbers passed as inputs, it returns 0. Here is an example where the function returns the greatest value of the two previous cells:

```javascript
const rows = [
  {
    name: "Row with greatestOf() example",
    type: rowTypes.DATA,
    cells: [
      // ...
      {
        customLogic: (cells, idx) =>
          greatestOf(cells[idx - 1].value, cells[idx - 2]),
      },
    ],
  },
  // ...
];
```

##### uniqueRespondingIUs(cells)

This function takes as input the list of cells in the row, finds the number of unique orginisation units at a given level (IU level, in this report) which have responded to at least one of the indicators in the row, and returns a promise which resolves to an object with 1) the number of unique responding IUs and 2) the sum of the numerical values reported for each unique IU. When an IU has responded twice or more to indicators in the row, the greater value of the responses is used in the sum of values. Because this returns a promise, retrieve your value of interest from a `.then()` statement (and recall that the custom logic function itself can return a promise). Here is an example, where the total number of unique IUs in the row is reported:

```javascript
const rows = [
  {
    name: "Row with uniqueRespondingIUs() example",
    type: rowTypes.DATA,
    cells: [
      // ...
      {
        customLogic: (cells, idx) =>
          uniqueRespondingIUs(cells).then(({ uniqueIUs, sumOfValues }) => {
            return uniqueIUs;
          }),
      },
    ],
  },
  // ...
];
```
