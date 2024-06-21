INTRODUCTION TO DAX
Based on SQLBI's Introducing DAX Video Course.

Report Schema
alt text

Report
alt text

Theory
The DAX Language
DAX is a functional language, the execution flows with function calls.

Language of:

Power Pivot
Power BI
SSAS Tabular
Important differences:

No concept of «row» and «column»
Different type system
Many new functions

Designed for data models and business calculations

! Code formatting is important in DAX, as it makes code debugging easy. For automatically formatting DAX code one can use daxformatter.

Calculated Columns
Columns computed using DAX.
Always computed for the current row.
Measures
Written using DAX
Do not work row by row
Instead, use tables and aggregators
Do not have the «current row» concept
Naming Conventions
Measures do not belong to a table => Avoid table name in referencing measures. This way it is easier to move to another table and identify as a measure.
So:
Calculated columns -> Table[Column]
Measures -> [Measure]
Measures vs Calculated Columns
Use a column when:
Needing to slice or filter on the value
Use a measure when:
Calculating percentages or ratios
Needing complex aggregations
Space and CPU usage:
Columns consume memory
Measures consume CPU
Aggregation Functions
Work only on numeric columns.
Aggregate only one column.
 SUM
 AVERAGE
 MIN
 MAX
The «X» Aggregation Functions
Iterators: useful to aggregate formulas
 SUMX
 AVERAGEX
 MINX
 MAXX
Iterate over the table and evaluate the expression for each row
Always receive two parameters:
Table to iterate
Formula to evaluate for each row
Example:
SUMX (
	Sales,
	Sales[Price] * Sales[Quantity]
)
Using Variables
Very useful to avoid repeating subexpressions in your code.
Example:
Quantity = 
VAR TotalQuantity = SUM ( Sales[Quantity] )
RETURN
	IF (
		TotalQuantity > 1000,
		TotalQuantity * 0.95,
		TotalQuantity * 1.25
	)
Date Functions
DATE, DATEVALUE, DAY, EDATE,
EOMONTH, HOUR, MINUTE,
MONTH, NOW, SECOND, TIME,
TIMEVALUE, TODAY, WEEKDAY,
WEEKNUM, YEAR, YEARFRAC
Table Functions
Basic functions that work on full tables and return a table as a result
FILTER
ALL
VALUES
DISTINCT
RELATEDTABLE
Their result is often used in other functions
They can be combined together to form complex expressions
FILTER
Adds a new condition by restricts the number of rows of a table
Returns a table that can be iterated by an «X» function
ALL
Returns all the rows of a table while ignoring the filter context
Returns a table that can be iterated by an «X» function
Can be also used with a single column ALL ( Customers[CustomerName] )the result being a table with one column
DISTINCT
Returns the distinct values of a column, only the ones visible in the current context
NumOfProducts =
COUNTROWS (
DISTINCT ( Product[ProductCode] )
)
RELATEDTABLE
Returns a table with all the rows related with the current one.
NumOfProducts = COUNTROWS ( RELATEDTABLE ( Product ) )
Evaluation Contexts
1. Filter Context
Defined by:
Row Selection
Column Selection
Report Filters
Slicers Selection
Rows outside of the filter context are not considered for the computation
Defined automatically by PivotTable, can be created with specific functions too
2. Row Context
Defined by:
Calculated column definition
Defined automatically for each row
Row Iteration functions
SUMX, AVERAGEX …
All «X» functions and iterators
Defined by the user formulas
Needed to evaluate column values, it is the concept of "current row"
! The Filter Context filters tables. The Row Context Iterates rows !

CALCULATE
Partially replaces the filter context
Conditions
Can replace a whole table
Can replace a single column
CALCULATE works on the filter context
Filters are evaluated in the outer filter context, then combined together in AND and finally used to build a new filter context into which DAX evaluates the expression.
Synthax:
CALCULATE (
	Expression,
	Filter1,
	…
	Filtern
)
Examples:
1. Filter and SUM are on the same table. You can obtain the same result using FILTER.

NumOfBigSales =
	CALCULATE (
			SUM ( Sales[SalesAmount] ),
			Sales[SalesAmount] > 100
	)
2.Clear filter on one column only. ALL used with a single column table.

CALCULATE (
	SUMX (
		Orders,
		Orders[Amount]
	),
	ALL ( Orders[Channel] )
)
Filters and Relationships
-Relationships affect filter context

RELATED
RELATED ( table[column] )

Opens a new row context on the target table
Following relationships
Enables Many side to One Side filtering
RELATEDTABLE
RELATEDTABLE ( table )

Filters the parameter table
Returns only rows related with the current one
It is the companion of RELATED
Context Transition
CALCULATE performs another task:
If executed inside a row context
It takes the row context
Transforms it into an equivalent filter context
Applies it to the data model Before computing its expression
Example: SUM() vs CALCULATE(SUM())
Time Intelligence
Time intelligence needs a date table.
Date table properties:
All dates should be present
From 1° of January, to 31° of December
No holes
Otherwise time intelligence will not work
Time Intelligence covers:
Year To Date
Quarter To Date
Running Total
Same period previous year
Working days computation
Fiscal Year
etc.
Aggregations:
YTD: Year To Date
QTD: Quarter To Date
MTD: Month To Date
CALENDAR
Returns a table with a single column named "Date", containing a contiguous set of dates in the given range, inclusive.
CALENDAR (
    DATE ( YEAR ( MIN ( Sales[Order Date] ) ), 1, 1 ),
    DATE ( YEAR ( MIN ( Sales[Order Date] ) ), 12, 31 )
)
CALENDARAUTO
Automatically creates a calendar table based on the database content. Optionally you can specify the last month (useful for fiscal years)
CALENDARAUTO uses all the dates in the model, excluding only calculated columns and tables
Year To Date
DATESYTD and TOTALYTD

SalesAmountYTD =
CALCULATE (
	SUM ( Sales[SalesAmount] ),
	DATESYTD ( 'Date'[Date] )
)
SalesAmountYTD :=
	TOTALYTD (
	SUM ( Sales[SalesAmount] ),
	'Date'[Date],
	"06-30"
)
Same Period Last Year
Sales_SPLY =
	CALCULATE (
		SUM ( Sales[SalesAmount] ),
		SAMEPERIODLASTYEAR ( 'Date'[Date] )
)
Running Total
Running total requires an explicit filter.
SalesAmountRT =
CALCULATE (
    SUM ( Sales[SalesAmount] ),
    FILTER ( ALL ( 'Date' ), 'Date'[Date] <= MAX ( 'Date'[Date] ) )
)
