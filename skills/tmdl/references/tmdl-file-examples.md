# TMDL File Type Examples

Examples showing the structure of each TMDL file type. Use these as reference when reading or editing TMDL files.

> Always verify against the current file in the project -- these examples are representative, not exhaustive.


## database.tmdl

Minimal file. Contains only the compatibility level:

```tmdl
database
	compatibilityLevel: 1604
```


## model.tmdl

Model configuration, `ref table` entries, query groups, extended properties, and annotations:

```tmdl
model SpaceParts
	culture: en-US
	defaultPowerBIDataSourceVersion: powerBI_V3
	discourageImplicitMeasures

queryGroup Tables

	annotation PBI_QueryGroupOrder = 0

queryGroup Parameters

	annotation PBI_QueryGroupOrder = 1

ref table Brands
ref table Customers
ref table Products
ref table Date
ref table Orders
ref table Invoices
ref table _Measures

ref cultureInfo en-US
```


## expressions.tmdl

Shared M expressions, parameters, and functions. Parameters use the `meta` syntax with `IsParameterQuery = true`.

### Simple Text Parameter

```tmdl
expression SqlEndpoint =
		"server.database.windows.net" meta
		[
			IsParameterQuery = true,
			IsParameterQueryRequired = true,
			Type = type text
		]
	lineageTag: abc-123
	queryGroup: Parameters

	annotation PBI_ResultType = Text
```

### DateTime Parameter (Incremental Refresh)

Uses backtick-enclosed expression (triple backticks) for multi-line M with special characters:

```tmdl
expression RangeStart = ```
		#datetime(2023, 01, 01, 0, 0, 0) meta
		[
			IsParameterQuery = true,
			IsParameterQueryRequired = true,
			Type = type datetime
		]
	```
	lineageTag: abc-123
	queryGroup: Parameters
```

### M Function (Reusable Transformation)

```tmdl
expression fnConvertCurrency =
		(amount as number, rate as number) as number =>
		    amount * rate
	lineageTag: abc-123
	queryGroup: Functions
```

### Backtick-Enclosed Expressions

Triple backticks (` ``` `) create a verbatim block for M or DAX expressions that contain characters that would otherwise conflict with TMDL syntax (colons, equals, etc.). The expression is taken literally without TMDL parsing.


## relationships.tmdl

All relationships in one file. Each relationship has a GUID name with `fromColumn` and `toColumn` in `Table.Column` format:

```tmdl
relationship abc-123
	fromColumn: Invoices.'Billing Date'
	toColumn: Date.Date

relationship def-456
	fromColumn: Orders.'Product Key'
	toColumn: Products.'Product Key'

relationship ghi-789
	isActive: false
	fromColumn: Orders.'Request Goods Receipt Date'
	toColumn: Date.Date

relationship jkl-012
	crossFilteringBehavior: bothDirections
	fromColumn: Customers.Station
	toColumn: Regions.Station
```

Optional properties: `isActive`, `crossFilteringBehavior` (`oneDirection`, `bothDirections`), `securityFilteringBehavior`, `relyOnReferentialIntegrity`, `joinOnDateBehavior`.

Inactive relationships include `isActive: false` explicitly.


## roles.tmdl

Security roles with RLS filter expressions:

```tmdl
role 'Regional Sales'
	modelPermission: read

	tablePermission Sales = [Region] = USERPRINCIPALNAME()

role Administrator
	modelPermission: administrator
```


## tables/<Name>.tmdl

Each table gets its own file. Contains the table declaration, columns, measures, hierarchies, partitions, and annotations.

### Table with Columns, Hierarchy, and Partition

```tmdl
table Brands
	lineageTag: abc-123

	column 'Brand Tier'
		dataType: string
		displayFolder: 1. Brand Hierarchy
		lineageTag: def-456
		sourceColumn: Flagship

	column Brand
		dataType: string
		displayFolder: 1. Brand Hierarchy
		lineageTag: ghi-789
		sourceColumn: Brand

	hierarchy 'Brand Hierarchy'
		displayFolder: 1. Brand Hierarchy
		lineageTag: jkl-012

		level Class
			lineageTag: mno-345
			column: 'Brand Class'

		level Brand
			lineageTag: pqr-678
			column: Brand

	partition Brands = m
		mode: import
		queryGroup: Tables
		source =
				let
				    Source = Sql.Database(#"SqlEndpoint",#"Database"),
				    Data = Source{[Schema="Dimview",Item="Brands"]}[Data]
				in
				    Data

	annotation TabularEditor_TableGroup = 1. Dimension Tables
```

### Table with Measures (including multi-line DAX)

```tmdl
table _Measures
	lineageTag: abc-123

	/// Total revenue across all invoices in the current filter context.
	measure 'Gross Sales' =
				SUMX (
				    Invoices,
				    Invoices[Quantity] * Invoices[Price]
				)
		formatString: #,##0
		displayFolder: 1. Sales\Actuals
		lineageTag: def-456

	/// Gross Sales month-to-date, considering only dates in scope.
	measure 'Gross Sales MTD' =
				CALCULATE (
				    [Gross Sales],
				    DATESMTD ( 'Date'[Date] )
				)
		formatString: #,##0
		displayFolder: 2. MTD\Sales
		lineageTag: ghi-789

	column Value
		lineageTag: jkl-012
		isNameInferred
		sourceColumn: [Value]

	partition _Measures = calculated
		mode: import
		source = {1}
```

### Table with SVG Measure (Backtick-Enclosed DAX)

Complex DAX expressions (like SVG measures) use backtick-enclosed blocks to avoid TMDL parsing issues:

```tmdl
table '_SVG Measures'
	lineageTag: abc-123

	/// SVG Bullet Chart of Sales MTD vs Budget MTD
	measure 'Sales vs Budget' = ```

			VAR _Actual = [Gross Sales MTD]
			VAR _Target = [Budget MTD]

			-- SVG rendering logic
			VAR _BarWidth = DIVIDE(_Actual, _Target) * 100
			RETURN
			    "data:image/svg+xml;utf8," &
			    "<svg xmlns='http://www.w3.org/2000/svg'>" &
			    "  <rect width='" & _BarWidth & "' height='20' fill='#4990E2'/>" &
			    "</svg>"

		```
		lineageTag: def-456
```


## cultures/<locale>.tmdl

Linguistic metadata and translations:

```tmdl
cultureInfo en-US

	linguisticMetadata =
			{
			  "Version": "1.0.0",
			  "Language": "en-US"
			}

	linguisticRelationship 'Product.Product Name' = Products.'Product Name'

	translation 'Product Name'
		table Products
		translatedCaption: Produkt
```


## Common Patterns

### Calculated Table (Measure Table)

A measure-only table using a calculated partition:

```tmdl
table _Measures
	lineageTag: abc-123

	column Value
		lineageTag: def-456
		isNameInferred
		sourceColumn: [Value]

	partition _Measures = calculated
		mode: import
		source = {1}
```

### Field Parameter

```tmdl
table '1) Selected Metric'
	lineageTag: abc-123

	column '1) Selected Metric'
		dataType: string
		isHidden
		lineageTag: def-456
		summarizeBy: none
		isNameInferred
		sortByColumn: '1) Selected Metric Order'
		sourceColumn: [1) Selected Metric]

	column '1) Selected Metric Fields'
		dataType: string
		isHidden
		lineageTag: ghi-789
		summarizeBy: none
		isNameInferred
		sourceColumn: [1) Selected Metric Fields]

	column '1) Selected Metric Order'
		dataType: int64
		isHidden
		lineageTag: jkl-012
		summarizeBy: none
		isNameInferred
		sourceColumn: [1) Selected Metric Order]

	partition '1) Selected Metric' = calculated
		mode: import
		source = {("Metric A", NAMEOF([Metric A]), 0), ("Metric B", NAMEOF([Metric B]), 1)}
```

### Calculation Group

```tmdl
table 'Time Intelligence'
	lineageTag: abc-123

	calculationGroup
		precedence: 10

		calculationItem YTD =
				CALCULATE (
				    SELECTEDMEASURE (),
				    DATESYTD ( 'Date'[Date] )
				)
			ordinal: 0

		calculationItem 'Prior Year' =
				CALCULATE (
				    SELECTEDMEASURE (),
				    DATEADD ( 'Date'[Date], -1, YEAR )
				)
			ordinal: 1

	column Name
		dataType: string
		isHidden
		lineageTag: def-456
		summarizeBy: none
		sourceColumn: Name

	partition 'Time Intelligence' = calculationGroup
		mode: import
```

### Date Table with dataCategory

```tmdl
table Date
	dataCategory: Time
	lineageTag: abc-123

	column Date
		isKey
		dataType: dateTime
		formatString: Long Date
		lineageTag: def-456
		summarizeBy: none
		sourceColumn: Date

	column Year
		dataType: int64
		displayFolder: 1. Year
		lineageTag: ghi-789
		summarizeBy: none
		isNameInferred
		sourceColumn: [Year]
		sortByColumn: 'Calendar Year Number'
```
