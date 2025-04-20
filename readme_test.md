**Assumptions:**

1. You have the raw data files (e.g., SQL Server access, Excel files, CSVs) containing the information needed for the tables listed in Block 1.
2. You have downloaded the PDF report mockup and the `PowerBI_Layout_Coordinates.xlsx` file.
3. You are using a Windows PC.

---

**Goal:** Replicate the Loan Overview report PDF exactly in Power BI Desktop.

**Estimated Time for a Beginner:** 4-6 hours (depending on data familiarity and clicking speed).

---

## Block 0: Setting Up Your Power BI Environment (15 min)

* **Purpose:** Install the necessary software and tools. Following these steps ensures compatibility and access to helpful features.

| Step | What to Do | Why Do This? | Detailed How-To |
| :---- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | :---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 0-a | Install **Power BI Desktop**. **Crucially, download it directly from the Microsoft website**, *not* from the Microsoft Store. | The direct download version often updates more predictably and is required for the "External Tools" ribbon we'll use later. | 1. Go to the [Power BI Desktop download page](https://www.microsoft.com/en-us/download/details.aspx?id=58494). <br> 2. Click "Download". <br> 3. Choose the `PBIDesktopSetup_x64.exe` (most modern PCs). <br> 4. Run the downloaded `.exe` file and follow the installation prompts (accept defaults). |
| 0-b | Install three **free helper tools**: Tabular Editor 2, DAX Studio, and Bravo for Power BI. | **Tabular Editor:** Makes creating many DAX measures faster. <br> **DAX Studio:** Helps analyze and optimize DAX measure performance. <br> **Bravo:** Useful for various tasks like analyzing your model (we won't use it heavily here, but good to have). <br> These will appear in Power BI's "External Tools" ribbon. | 1. **Tabular Editor 2:** Go to [github.com/TabularEditor/TabularEditor/releases](https://github.com/TabularEditor/TabularEditor/releases), find the latest release (e.g., 2.x.x), download `TabularEditor.msi`, and run it. <br> 2. **DAX Studio:** Go to [daxstudio.org](https://daxstudio.org/), click "Download", download the `.exe` or `.msi`, and run it. <br> 3. **Bravo:** Go to [bravo.bi](https://bravo.bi/), click "Download", download the `.msi`, and run it. |
| 0-c | Download the **PDF mock-up** of the report and the **Excel layout file** (`PowerBI_Layout_Coordinates.xlsx`) to an easy-to-find folder (e.g., `C:\PowerBI_Project\Design`). | We need the PDF to reference the design and pick colors. We need the Excel file for precise visual placement. | Save the files provided into a dedicated project folder. |
| 0-d | **Turn on file extensions** in Windows File Explorer. | Makes it easier to see file types like `.pbix`, `.pbit`, and `.json` which we'll encounter. | 1. Open File Explorer. <br> 2. Click the "View" tab at the top. <br> 3. Check the box for "File name extensions". |
| 0-e | (Optional but Recommended) Install **PowerToys for Windows**. | Contains a very handy "Color Picker" tool (Win + Shift + C) which we'll use to grab colors from the PDF. | 1. Go to the [Microsoft PowerToys GitHub releases page](https://github.com/microsoft/PowerToys/releases). <br> 2. Download the latest `PowerToysSetup-x.x.x-x64.exe`. <br> 3. Run the installer. |

---

## Block 1: Create the Star-Schema Data Model (45-60 min)

* **Purpose:** Load your raw data into Power BI, clean it up slightly, and define relationships between tables. This structure makes calculations easier and reporting faster. A "star schema" has one central "Fact" table connected to several "Dimension" tables.

1. **Open Power BI Desktop.** You'll see a blank canvas. Close any welcome screens.
2. **Get Data:** On the "Home" ribbon, click "Get Data". Choose the appropriate source for your data (e.g., "SQL Server", "Excel workbook", "Text/CSV").
3. **Load Each Table:** Connect to your source and select the tables listed below. If your source is multiple files (like Excel/CSV), repeat the "Get Data" step for each file/table. **Crucially, click "Transform Data"** (instead of "Load") after selecting the tables in the Navigator window. This opens the Power Query Editor.

 | Table Name (Use these *exact* names in Power Query) | Grain (What one row represents) | Minimum Columns Needed (Rename in Power Query if necessary) | Role |
 | :-------------------------------------------------- | :------------------------------ | :-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | :-------- |
 | **FactLoan** | 1 loan | `LoanID` (Unique ID for each loan), `Status`, `LoanType`, `FinancingType`, `Currency`, `AuthorizedAmount`, `CurrentBalance`, `InterestRate` (store as decimal, e.g., 0.0475), `OriginalTerm` (months), `RemainingTerm` (months), `FirstPaymentDate`, `ExtensionFlag` (Yes/No or True/False), `MaturityDate`, `AmortizationYears`, `RiskRating`, `LastReviewDate` | FACT |
 | **DimLenders** | 1 lender's share in a loan | `LoanID` (Links to FactLoan), `LenderID` (Unique ID for this row), `Entity` (Lender Name), `SharePct` (e.g., 0.65), `ShareAmt` (e.g., 3412500), `Role` (e.g., Lead Lender) | DIMENSION |
 | **DimParticipants** | 1 borrower/guarantor | `LoanID` (Links to FactLoan), `ParticipantID` (Unique ID for this row), `Role` (Borrower/Guarantor), `Entity` (Participant Name), `SharePct` (e.g., 1.00 or 0.75) | DIMENSION |
 | **DimProperties** | 1 property tied to a loan | `LoanID` (Links to FactLoan), `PropertyID` (Unique ID for this row), `Code` (P-001), `Name`, `City`, `State`, `PrimaryFlag` (Yes/No or True/False), `TotalSquareFootage`, `YearBuilt`, `OccupancyRate` (e.g., 0.92), `MajorTenants` (Count) | DIMENSION |
 | **DimAppraisals** | 1 appraisal value | `LoanID` (Links to FactLoan), `AppraisalID` (Unique ID), `AppraisalDate`, `AppraisalType` (Full/Desktop), `Valuation` | DIMENSION |
 | **DimInsurance** | 1 insurance policy | `LoanID` (Links to FactLoan), `InsuranceID` (Unique ID), `InsuranceType` (Property/Liability/Flood), `Provider`, `ExpiryDate` | DIMENSION |
 | **DimTaxes** | 1 tax bill for a property | `LoanID` (Links to FactLoan), `TaxID` (Unique ID), `PropertyID` (Optional, links to DimProperties), `PropertyName` (Or link to DimProperties), `DueDate`, `Amount` | DIMENSION |
 | **DateDim** | 1 row = 1 day | `Date` (Unique date column), `Year`, `MonthName`, `MonthOfYear`, `Day` (etc. - create a standard Calendar table if you don't have one. Search online for "Power BI DAX Calendar Table") | DIMENSION |

4. **Power Query Clean-ups** (Inside the Power Query Editor window):
 * For *each* table loaded (listed on the left pane):
 * **Check Column Names:** Ensure they match the "Minimum Columns Needed" list above. Right-click a column header -> "Rename" if needed.
 * **Check Data Types:** Power Query tries to guess, but verify.
 * Select the `Currency` column -> "Transform" ribbon -> "Data Type" -> Select "Text". (We format currency later in Power BI).
 * Select columns like `AuthorizedAmount`, `CurrentBalance`, `ShareAmt`, `Valuation`, `Amount` -> Ensure Data Type is "Decimal Number" or "Fixed decimal number".
 * Select columns like `InterestRate`, `SharePct`, `OccupancyRate` -> Ensure Data Type is "Decimal Number".
 * Select columns like `OriginalTerm`, `RemainingTerm`, `AmortizationYears`, `MajorTenants`, `YearBuilt` -> Ensure Data Type is "Whole Number".
 * Select date columns (`FirstPaymentDate`, `MaturityDate`, `LastReviewDate`, `AppraisalDate`, `ExpiryDate`, `DueDate`, `Date`) -> Ensure Data Type is "Date" or "Date/Time" as appropriate.
 * Select ID columns (`LoanID`, `LenderID`, etc.) -> Can be "Whole Number" or "Text" depending on your source, ensure consistency for `LoanID` across tables.
 * Select text columns (`Status`, `LoanType`, etc.) -> Ensure Data Type is "Text".
 * Select Yes/No columns (`ExtensionFlag`, `PrimaryFlag`) -> "Transform" ribbon -> "Data Type" -> Select "True/False". If they contain "Yes"/"No", right-click header -> "Replace Values" (Replace "Yes" with `true`, Replace "No" with `false`), *then* change Data Type to True/False.
5. **Close & Apply:** Once all tables are loaded, renamed, and data types are set, click the "Home" ribbon in Power Query Editor -> "Close & Apply". Power BI will load the data into the model.

6. **Model View & Relationships:**
 * Click the "Model" view icon on the left sidebar (looks like three connected boxes). You'll see your tables. Arrange them visually: `FactLoan` in the center, Dimension tables (`Dim...`) surrounding it.
 * **Create LoanID Relationships:** Click and drag the `LoanID` field from `FactLoan` and drop it onto the `LoanID` field in `DimLenders`. A line will appear. Repeat this for `FactLoan` -> `DimParticipants`, `FactLoan` -> `DimProperties`, `FactLoan` -> `DimAppraisals`, `FactLoan` -> `DimInsurance`, `FactLoan` -> `DimTaxes`.
 * **Set Relationship Properties:** Double-click each line (relationship) you just created.
 * Ensure **Cardinality** is "One to Many (1:*)", with the "1" side on `FactLoan`.
 * Ensure **Cross filter direction** is "Single". This means filters flow *from* `FactLoan` *to* the dimension tables.
 * Click "OK".
 * **Create Date Relationships:** Click and drag the `Date` field from `DateDim` and drop it onto the `MaturityDate` field in `FactLoan`. Repeat for other key date fields you want to filter by time (e.g., `Date` from `DateDim` onto `AppraisalDate` in `DimAppraisals`, `DueDate` in `DimTaxes`, etc.). *Important:* Power BI might mark only one date relationship as "active". That's okay for now.
7. **Hide Unnecessary Fields:** We want users to use Measures (Block 2), not raw numeric columns directly.
 * Click the "Data" view icon (looks like a table).
 * In the "Data" pane on the right, expand `FactLoan`.
 * Right-click on `AuthorizedAmount` -> "Hide". Repeat for `CurrentBalance`, `InterestRate`, `OriginalTerm`, `RemainingTerm`, `AmortizationYears`.
 * Hide the `LoanID` foreign key columns in all the `Dim...` tables (e.g., right-click `DimLenders`.[`LoanID`] -> Hide). They are only needed for the relationship.
 * Hide any other technical/intermediate columns not needed for direct display in visuals.

Now your data model is structured correctly!

---

## Block 2: Create Base Measures (30 min)

* **Purpose:** Define the core calculations (KPIs - Key Performance Indicators) using DAX (Data Analysis Expressions). Measures are reusable formulas that respond to filters.

1. **Open Tabular Editor 2:** Click the "External Tools" ribbon in Power BI Desktop -> Click "Tabular Editor". (Allow any security prompts).
2. **Create a Measure Folder (Optional but good practice):** In Tabular Editor's left pane ("TOM Explorer"), right-click on "Tables" -> "Create New" -> "Calculation Group". *Correction:* Let's use a Display Folder instead. Right-click on a measure later -> "Display folder". For now, just create measures directly.
3. **Create Measures:** Right-click on the `FactLoan` table in the TOM Explorer -> "Create New" -> "Measure". A new measure appears. Select it. In the properties pane below (or the main script area), paste the DAX expression. Rename the measure in the properties pane.

 * **Measure Name:** `Authorized Amount`
 ```DAX
 SUM ( FactLoan[AuthorizedAmount] )
 ```
 * **Formatting:** In the Properties pane (bottom left usually), find "Format String Expression". Set it to `"$#,0"` (or leave blank for now, format later in Power BI).

 * **Measure Name:** `ABC Share Authorized Amount`
 ```DAX
 // Placeholder: Assumes ABC Financial always has 65% share. 
 // Replace 0.65 with a dynamic calculation if share % varies or is stored in DimLenders.
 SUM ( FactLoan[AuthorizedAmount] ) * 0.65 
 ```
 * **Formatting:** `"$#,0"`

 * **Measure Name:** `Current Balance`
 ```DAX
 SUM ( FactLoan[CurrentBalance] )
 ```
 * **Formatting:** `"$#,0"`

 * **Measure Name:** `ABC Share Current Balance`
 ```DAX
 // Placeholder: Assumes ABC Financial always has 65% share.
 SUM ( FactLoan[CurrentBalance] ) * 0.65 
 ```
 * **Formatting:** `"$#,0"`

 * **Measure Name:** `Interest Rate`
 ```DAX
 // Shows the rate only if a single loan is selected/filtered.
 SELECTEDVALUE ( FactLoan[InterestRate] ) 
 ```
 * **Formatting:** `"0.00%"`

 * **Measure Name:** `Remaining Term Months`
 ```DAX
 // Shows term only if a single loan is selected/filtered.
 SELECTEDVALUE ( FactLoan[RemainingTerm] )
 ```
 * **Formatting:** `"0"` (General number)

 * **Measure Name:** `Remaining Term Display`
 ```DAX
 VAR _Term = SELECTEDVALUE ( FactLoan[RemainingTerm] )
 RETURN
 IF ( NOT ISBLANK(_Term), _Term & " months", BLANK() )
 ```
 * **Formatting:** General (Text)

 * **Measure Name:** `Original Term Months`
 ```DAX
 SELECTEDVALUE( FactLoan[OriginalTerm] )
 ```
 * **Formatting:** `"0"`

 * **Measure Name:** `ABC Participation Pct`
 ```DAX
 // Placeholder: Assumes fixed 65% participation. Calculate dynamically if needed.
 0.65 
 ```
 * **Formatting:** `"0%"`

 * **Measure Name:** `Lead Lender`
 ```DAX
 // Assumes 'Role' column exists in DimLenders and contains 'Lead Lender'.
 // Adjust filter if needed. Shows only if a single Lead Lender exists for the selection.
 CALCULATE(
 SELECTEDVALUE( DimLenders[Entity] ),
 DimLenders[Role] = "Lead Lender" // Modify text if your role name is different
 )
 ```
 * **Formatting:** General (Text)

 * **Measure Name:** `Risk Rating Display`
 ```DAX
 SELECTEDVALUE ( FactLoan[RiskRating] )
 ```
 * **Formatting:** General (Text)

 * **Measure Name:** `Last Review Date Display`
 ```DAX
 VAR _LastReview = SELECTEDVALUE( FactLoan[LastReviewDate] )
 RETURN
 IF( NOT ISBLANK(_LastReview), "Last Review: " & FORMAT( _LastReview, "MMM yyyy"), BLANK())
 ```
 * **Formatting:** General (Text)

 * **Measure Name:** `Balance at Maturity`
 ```DAX
 // NOTE: This is a placeholder. Calculating true balance at maturity requires
 // complex amortization logic based on payments, rate, term etc. 
 // Using a fixed value from PDF for demo. Replace with real calculation if possible.
 4125000 
 ```
 * **Formatting:** `"$#,0"`

 * **Measure Name:** `ABC Share Balance at Maturity`
 ```DAX
 // Placeholder based on fixed value and 65% share.
 4125000 * 0.65
 ```
 * **Formatting:** `"$#,0"`

 * **Measure Name:** `Amount to Fund`
 ```DAX
 // Placeholder based on PDF. Logic might be Authorized - Current or similar.
 375000
 ```
 * **Formatting:** `"$#,0"`

 * **Measure Name:** `ABC Share Amount to Fund`
 ```DAX
 // Placeholder based on fixed value and 65% share.
 375000 * 0.65
 ```
 * **Formatting:** `"$#,0"`

 * **Measure Name:** `Next Payment Amount`
 ```DAX
 // Placeholder based on PDF. Likely comes from a payment schedule table or calculation.
 32500
 ```
 * **Formatting:** `"$#,0"`

 * **Measure Name:** `ABC Share Next Payment`
 ```DAX
 // Placeholder based on fixed value and 65% share.
 32500 * 0.65
 ```
 * **Formatting:** `"$#,0"`

 * **Measure Name:** `Spread Display`
 ```DAX
 // Placeholder: Assumes a single spread value applies. If multiple tranches exist, 
 // this needs the CONCATENATEX logic from Block 7-d.
 // Let's use a placeholder for now based on the fixed rate loan.
 // If your InterestRate column IS the final rate (4.75%), spread isn't directly shown here
 // based on the data model. If you have separate Spread/Index columns, use SELECTEDVALUE.
 // For demo, assume a fixed spread was part of the calculation.
 "2.25%" // Hardcoding based on PDF example, replace if you have spread data.
 ```
 * **Formatting:** General (Text)

 * **Measure Name:** `Index Display`
 ```DAX
 // Placeholder: Linked to Spread Display above.
 "Over SOFR" // Hardcoding based on PDF, replace if you have index data.
 ```
 * **Formatting:** General (Text)


4. **Save in Tabular Editor:** Click File -> Save (or the floppy disk icon). This pushes the measures back to Power BI Desktop.
5. **Close Tabular Editor.**
6. **Refresh Now:** Back in Power BI Desktop, you might see a yellow bar asking to refresh. Click "Refresh now". Your measures should appear in the "Data" pane on the right, usually under the table you right-clicked (e.g., `FactLoan`).
7. **(Optional) Organize Measures:** In Power BI Desktop's "Model" view, select multiple measures (Ctrl+Click) in the Data pane. In the "Properties" pane (if not visible, go to View ribbon -> check "Properties"), type a name like `_Key Measures` into the "Display folder" box and press Enter. This groups them nicely.

---

## Block 3: Build the Color Theme (15 min)

* **Purpose:** Define the color palette and default fonts to match the PDF, ensuring visual consistency.

1. **Pick Colors from PDF:**
 * Open the PDF report mockup.
 * Use PowerToys Color Picker (Press `Win + Shift + C`). Your mouse cursor will change.
 * Click on the dark blue color used in titles and highlights. It will copy the HEX code (e.g., `#1A2B5C`). Paste this into Notepad.
 * Click on the light grey background color of the main sections. Copy its HEX code (e.g., `#F4F5F7` - though the PDF seems mostly white `#FFFFFF`). Paste into Notepad.
 * Pick any other key colors (e.g., the green used for "Active" status `#2ECC71`, maybe a red/yellow for risk). We primarily need the dark blue. The overall background seems white (`#FFFFFF`). Let's use:
 * Dark Blue: `#1A2B5C`
 * White: `#FFFFFF`
 * Medium Grey (for text/borders): `#707070`
 * Light Grey (potential subtle background): `#F4F5F7`
 * Green (Accent): `#2ECC71`
 * Blue (Accent): `#6C93FF` (from Cookbook 2)
2. **Generate Theme JSON:**
 * Go to a theme generator website like [themes.powerbi.tips](https://themes.powerbi.tips/v3/theme-generator). (Or you can manually create a JSON file).
 * **Colors Tab:**
 * `Name`: `Loan Report Theme`
 * `Data Color 1`: `#1A2B5C` (Dark Blue)
 * `Data Color 2`: `#6C93FF` (Blue Accent)
 * `Data Color 3`: `#F38230` (Orange Accent from Cookbook 2 - optional)
 * `Data Color 4`: `#2ECC71` (Green Accent)
 * `Background`: `#FFFFFF` (White)
 * `Secondary background`: `#F4F5F7` (Light Grey - for subtle alternates if needed)
 * `Foreground`: `#1A2B5C` (Dark Blue for text)
 * `Table accent`: `#1A2B5C`
 * **Text Tab:**
 * Set **Font Family** for Title, Cards & KPIs, Tab Headers, General to `Segoe UI`.
 * Set font sizes (can adjust later): Title=24, Card Value=20, Card Category=11, Header=12, Body=11.
 * **Visuals Tab:**
 * **Card:**
 * `Background Color`: `#FFFFFF`
 * `Border`: On -> `Color`: `#E0E0E0`, `Radius`: 4 px
 * `Shadow`: On -> `Preset`: Custom, `Position`: Bottom Right, `Size`: 2px, `Blur`: 4px
 * **Table and Matrix:**
 * `Style Preset`: None
 * `Grid` -> `Horizontal gridline`: Off, `Vertical gridline`: Off
 * `Values` -> `Alternate background color`: `#FFFFFF` (effectively off)
 * `Values` -> `Row padding`: 4px
 * **Download Theme:** Click the "Download Theme" button. It will save a `.json` file (e.g., `Loan Report Theme.json`).
3. **Import Theme into Power BI:**
 * In Power BI Desktop, go to the "View" ribbon.
 * Click the dropdown arrow under "Themes".
 * Select "Browse for themes".
 * Navigate to and select the `.json` file you just downloaded. Click "Open".

The default colors and fonts for new visuals will now match your theme.

---

## Block 4: Set Page Canvas & Grid (5 min)

* **Purpose:** Define the overall size of the report page and set up visual guides for alignment.

1. **Select the Page:** Click on a blank area of the report canvas (make sure no visual is selected).
2. **Format Canvas:** In the "Visualizations" pane on the right, click the "Format your report page" icon (looks like a paint roller).
3. **Canvas Settings:**
 * Expand the "Canvas settings" section.
 * **Type:** Choose "Custom".
 * **Width:** Enter `1280` pixels.
 * **Height:** Enter `2300` pixels (based on coordinate file, adjust if needed).
4. **Canvas Background:**
 * Expand "Canvas background".
 * **Color:** Should already be White (`#FFFFFF`) from the theme. Set **Transparency** to 0%.
5. **Enable Gridlines:**
 * Go to the "View" ribbon.
 * Check the box for "Gridlines".
 * Check the box for "Snap to grid". This helps align visuals easily.
 * *(Optional: Use Alt+Drag to move visuals slightly off-grid if needed for precise pixel placement not matching the 8px grid).*

---

## Block 5: Position Visuals Using Coordinates (15 min)

* **Purpose:** Place placeholder visuals onto the canvas exactly where they belong according to the layout file. We'll populate them later.

1. **Open the Layout Excel:** Have the `PowerBI_Layout_Coordinates.xlsx` file open for reference.
2. **Insert and Position Each Visual:** For *each row* in the Excel file:
 * **Identify Visual Type:** Look at the "Visual" column in the Excel (e.g., "Title", "Slicer", "Card", "Matrix").
 * **Insert the Visual:** In Power BI Desktop, go to the "Insert" ribbon (or use the "Visualizations" pane).
 * For "Title": Insert -> "Text box".
 * For "Slicer": Visualizations pane -> Click the "Slicer" icon.
 * For "Card": Visualizations pane -> Click the "Card" icon.
 * For "Matrix": Visualizations pane -> Click the "Matrix" icon.
 * For "Table": Visualizations pane -> Click the "Table" icon.
 * *(For panels later, we might insert a "Shape" -> "Rectangle" first, size it, and then place visuals inside, or just group visuals).*
 * **Select the New Visual:** Click on the visual you just added to the canvas.
 * **Set Position & Size:**
 * In the "Visualizations" pane, click the "Format visual" icon (paint roller).
 * Go to the "General" tab.
 * Expand the "Properties" section.
 * Expand "Size". Enter the `Height` and `Width` from the Excel row.
 * Expand "Position". Enter the `Horizontal` (X) and `Vertical` (Y) from the Excel row.
 * **Repeat:** Do this for every item listed in the Excel layout file. You'll have a canvas full of empty, correctly positioned visuals.

---

## Block 6: Configure Top Section Visuals (Slicers & KPIs) (25 min)

* **Purpose:** Populate and format the title, slicers, and KPI cards at the top of the report.

1. **Title Text Box (X:20, Y:0):**
 * Select the text box you placed at the top.
 * Type the title: `Loan Overview - Real Estate Financing`
 * Select the text inside the box.
 * Use the formatting bar that appears (or the Format pane -> Text):
 * Font: `Segoe UI`
 * Size: `24 pt`
 * **Bold:** On
 * Font Color: `#1A2B5C` (Dark Blue)
 * Format Pane -> General -> Effects -> Background: Off, Shadow: Off.

2. **Slicers (Row at Y:60):** Select each slicer placeholder one by one.
 * **Loan ID Slicer (X:20):**
 * Drag `FactLoan[LoanID]` (the original ID column, not the measure) from the "Data" pane onto the "Field" well in the Visualizations pane.
 * Format Pane -> Visual -> Slicer settings -> Options -> Style: `Tile`.
 * Format Pane -> Visual -> Slicer settings -> Selection -> Single select: `On`.
 * Format Pane -> Visual -> Slicer header: `Off`.
 * Format Pane -> Visual -> Values -> Font: `Segoe UI`, Size: `11pt`.
 * **Status Slicer (X:220):**
 * Drag `FactLoan[Status]` onto the "Field" well.
 * Format Pane -> Visual -> Slicer settings -> Options -> Style: `Tile`.
 * Format Pane -> Visual -> Slicer header: `Off`.
 * Format Pane -> Visual -> Values -> Font: `Segoe UI`, Size: `11pt`.
 * **Country Slicer (X:420):** *(Note: Country wasn't in the initial data model - add `Country` to `DimProperties` or `FactLoan` if needed)*
 * Drag the `Country` field onto the "Field" well.
 * Format Pane -> Visual -> Slicer settings -> Options -> Style: `Tile`.
 * Format Pane -> Visual -> Slicer header: `Off`.
 * Format Pane -> Visual -> Values -> Font: `Segoe UI`, Size: `11pt`.
 * **Financing Type Slicer (X:620):**
 * Drag `FactLoan[FinancingType]` onto the "Field" well.
 * Format Pane -> Visual -> Slicer settings -> Options -> Style: `Tile`.
 * Format Pane -> Visual -> Slicer header: `Off`.
 * Format Pane -> Visual -> Values -> Font: `Segoe UI`, Size: `11pt`.
 * **Property Type Slicer (X:820):** *(Note: PropertyType wasn't in the initial data model - add `PropertyType` to `DimProperties` if needed)*
 * Drag the `PropertyType` field onto the "Field" well.
 * Format Pane -> Visual -> Slicer settings -> Options -> Style: `Tile`.
 * Format Pane -> Visual -> Slicer header: `Off`.
 * Format Pane -> Visual -> Values -> Font: `Segoe UI`, Size: `11pt`.
 * **Borrower Slicer (X:1020):**
 * Drag `DimParticipants[Entity]` onto the "Field" well. *(You might need to filter DimParticipants to only show Role="Borrower" via the Filter pane if Guarantors are mixed in)*.
 * Format Pane -> Visual -> Slicer settings -> Options -> Style: `Tile`.
 * Format Pane -> Visual -> Slicer header: `Off`.
 * Format Pane -> Visual -> Values -> Font: `Segoe UI`, Size: `11pt`.

3. **KPI Cards (Rows at Y:120 & Y:220):** Select each card placeholder one by one.
 * **Authorized Amount Card (X:20, Y:120):**
 * Drag measure `[Authorized Amount]` onto the "Fields" well.
 * Drag measure `[ABC Share Authorized Amount]` onto the "Fields" well, *below* the first one.
 * Format Pane -> Visual -> Callout value -> Font: `Segoe UI`, Size: `20pt`, Color: Dark Blue. Apply to: All.
 * Format Pane -> Visual -> Callout value -> Display units: `None`. Decimal places: `0`.
 * Format Pane -> Visual -> Category label -> Font: `Segoe UI`, Size: `11pt`, Color: `#707070` (Medium Grey). Show: `On`.
 * Format Pane -> Visual -> Card -> Padding: Adjust slightly if needed (e.g., 8px).
 * *Rename Category Labels:* In the "Fields" well, double-click `[Authorized Amount]` -> Rename for this visual: `Authorized Amount`. Double-click `[ABC Share Authorized Amount]` -> Rename: `ABC Share: [Value]`. *(Power BI might automatically add the value, check how it looks)*. Or, use a text box below the card for the secondary label if renaming doesn't work well. Let's try the built-in category label first. *Correction:* The PDF shows the secondary value *below* the primary. The standard Card visual puts the category label below. To match the PDF exactly with two values stacked, we might need *two separate cards* stacked closely, or use a *Multi-Row Card* styled carefully. Let's try the **Multi-Row Card**:
 * Change Visual Type to "Multi-row card".
 * Drag `[Authorized Amount]` and `[ABC Share Authorized Amount]` to Fields.
 * Format Pane -> Visual -> Cards -> Title (Category Label) Font/Size/Color.
 * Format Pane -> Visual -> Cards -> Value Font/Size/Color.
 * Format Pane -> Visual -> Style -> Border: Off (use General->Effects->Border instead). Padding: 0.
 * Format Pane -> General -> Effects -> Turn on Border (should inherit theme).
 * Format Pane -> General -> Title: Off.
 * *Self-Correction:* The standard "Card" visual *does* support adding a second measure to the "Tooltips" data role, which *sometimes* can be displayed, but the most reliable way for the PDF layout is likely two stacked cards or careful Multi-Row card formatting. Let's stick to the **Multi-Row Card** approach for consistency with the Risk Rating card later. Adjust padding/spacing inside the multi-row card format options.
 * **Current Balance Card (X:220, Y:120):** Use Multi-Row Card. Fields: `[Current Balance]`, `[ABC Share Current Balance]`. Format similarly to Authorized Amount.
 * **Interest Rate Card (X:420, Y:120):** Use **standard Card**. Field: `[Interest Rate]`. Add a Text Box below it for "Fixed Rate".
 * Card Format Pane -> Visual -> Callout value -> Size: 20pt. Display units: None. Decimal places: 2.
 * Card Format Pane -> Visual -> Category label: Off.
 * Insert Text Box below -> Type "Fixed Rate" -> Font Size 11pt, Color Grey. Position carefully.
 * **Remaining Term Card (X:620, Y:120):** Use **standard Card**. Field: `[Remaining Term Display]`. Add Text Box below for "of 60 months".
 * Card Format Pane -> Visual -> Callout value -> Size: 20pt.
 * Card Format Pane -> Visual -> Category label: Off.
 * Insert Text Box below -> Type "of 60 months" -> Font Size 11pt, Color Grey. Position carefully. *(Make "60 months" dynamic if possible by referencing `[Original Term Months]`)*.
 * **ABC Participation Card (X:820, Y:220):** Use **standard Card**. Field: `[ABC Participation Pct]`. Add Text Box below for "Lead Lender".
 * Card Format Pane -> Visual -> Callout value -> Size: `32pt` (make it prominent). Display units: None. Decimal places: 0.
 * Card Format Pane -> Visual -> Category label: Off.
 * Insert Text Box below -> Type "Lead Lender" -> Font Size 11pt, Color Grey. Position carefully. *(Ideally, link this text box dynamically to the `[Lead Lender]` measure, but simple text is faster for now)*.
 * **Risk Rating Card (X:1020, Y:220):** Use **Multi-row card**.
 * Fields: `[Risk Rating Display]`, `[Last Review Date Display]`.
 * Format -> Visual -> Cards -> Adjust Title (Category) and Value fonts/sizes/colors. Remove padding/borders within the card visual itself.
 * Format -> General -> Effects -> Apply theme border/shadow.
 * Format -> General -> Title: Off.
 * **Ensure Borders/Shadows:** Verify all Cards/Multi-Row Cards have the Border and Shadow applied correctly via Format Pane -> General -> Effects (should be inherited from the theme).

---

## Block 7: Create Vertical Definition Lists (Loan/Property/Financial) (30 min)

* **Purpose:** Display sets of field labels and their corresponding values vertically, mimicking the look of definition lists in the PDF. We'll use the Matrix visual cleverly.

1. **Create Label Tables:** We need small tables that just list the labels we want to show, in the desired order.
 * Go to the "Home" ribbon -> "Enter data".
 * A "Create Table" window appears.
 * **For Loan Summary:**
 * Rename Column1 to `DisplayOrder` (Data Type: Whole Number).
 * Rename Column2 to `Label` (Data Type: Text).
 * Rename Column3 to `DataKey` (Data Type: Text).
 * Enter rows based on the PDF's "Loan Summary" section:
 | DisplayOrder | Label | DataKey |
 |--------------|--------------------|----------------|
 | 1 | ID | LoanID |
 | 2 | Status | Status |
 | 3 | Loan Type: | LoanType |
 | 4 | Financing Type: | FinancingType |
 | 5 | Currency: | Currency |
 | 6 | Original Term: | OriginalTerm |
 | 7 | Remaining Term: | RemainingTerm |
 | 8 | First Payment: | FirstPayment |
 | 9 | Extension: | Extension |
 | 10 | Maturity Date: | MaturityDate |
 | 11 | Amortization: | Amortization |
 * Name the table: `LoanFieldLabels`. Click "Load".
 * **Repeat "Enter data" for Property Summary:**
 * Columns: `DisplayOrder`, `Label`, `DataKey`.
 * Rows based on PDF's "Property Summary":
 | DisplayOrder | Label | DataKey |
 |--------------|------------------------|--------------------|
 | 1 | Total Square Footage: | SqFt |
 | 2 | Year Built: | YearBuilt |
 | 3 | Occupancy Rate: | OccupancyRate |
 | 4 | Major Tenants: | MajorTenants |
 * Name the table: `PropertyFieldLabels`. Click "Load".
 * **Repeat "Enter data" for Financial Details:**
 * Columns: `DisplayOrder`, `Label`, `DataKey`.
 * Rows based on PDF's "Financial Details" metrics:
 | DisplayOrder | Label | DataKey |
 |--------------|---------------------|---------------------|
 | 1 | Authorized Amount | AuthAmt |
 | 2 | Current Balance | CurrBal |
 | 3 | Balance at Maturity | BalMaturity |
 | 4 | Amount to Fund | AmtFund |
 | 5 | Next Payment | NextPay |
 | 6 | Current Rate | CurrRate |
 | 7 | Spread | SpreadIndex |
 | 8 | *(Add others if any)* | *(Add keys)* |
 * Name the table: `FinancialFieldLabels`. Click "Load".

2. **Create SWITCH Measures:** These measures will look at the selected label (from the Matrix row) and return the corresponding *value* from our base measures or data columns.
 * On the "Home" ribbon, click "New measure".
 * Paste the following DAX formula.

 ```DAX
 Loan Detail Value :=
 VAR _SelectedKey = SELECTEDVALUE ( LoanFieldLabels[DataKey] )
 RETURN
 SWITCH ( TRUE(),
 _SelectedKey = "LoanID", SELECTEDVALUE ( FactLoan[LoanID] ), // Assuming LoanID is text
 _SelectedKey = "Status", SELECTEDVALUE ( FactLoan[Status] ),
 _SelectedKey = "LoanType", SELECTEDVALUE ( FactLoan[LoanType] ),
 _SelectedKey = "FinancingType", SELECTEDVALUE ( FactLoan[FinancingType] ),
 _SelectedKey = "Currency", SELECTEDVALUE ( FactLoan[Currency] ),
 _SelectedKey = "OriginalTerm", FORMAT( SELECTEDVALUE ( FactLoan[OriginalTerm] ), "0" ) & " months",
 _SelectedKey = "RemainingTerm", FORMAT( SELECTEDVALUE ( FactLoan[RemainingTerm] ), "0" ) & " months",
 _SelectedKey = "FirstPayment", FORMAT ( SELECTEDVALUE ( FactLoan[FirstPaymentDate] ), "MMM dd, yyyy" ),
 _SelectedKey = "Extension", IF(SELECTEDVALUE(FactLoan[ExtensionFlag]) = TRUE(), "Yes", "No"),
 _SelectedKey = "MaturityDate", FORMAT ( SELECTEDVALUE ( FactLoan[MaturityDate] ), "MMM dd, yyyy" ),
 _SelectedKey = "Amortization", FORMAT( SELECTEDVALUE ( FactLoan[AmortizationYears] ), "0" ) & " years",
 BLANK() // Default case
 )
 ```

 * Create another **New measure**:

 ```DAX
 Property Detail Value :=
 VAR _SelectedKey = SELECTEDVALUE ( PropertyFieldLabels[DataKey] )
 // Assuming Property details are in DimProperties and you have a relationship
 // Or if they are in FactLoan, adjust accordingly. Using DimProperties here.
 VAR _Prop = SELECTEDVALUE( DimProperties[PropertyID] ) // Need context if multiple properties
 RETURN
 SWITCH ( TRUE(),
 // This part assumes ONLY ONE property is relevant/selected for the context.
 // If multiple properties per loan, this logic needs refinement (e.g., filter for PrimaryFlag).
 _SelectedKey = "SqFt", FORMAT( SELECTEDVALUE ( DimProperties[TotalSquareFootage] ), "#,0 sq ft" ),
 _SelectedKey = "YearBuilt", FORMAT( SELECTEDVALUE ( DimProperties[YearBuilt] ), "0" ),
 _SelectedKey = "OccupancyRate", FORMAT( SELECTEDVALUE ( DimProperties[OccupancyRate] ), "0%" ),
 _SelectedKey = "MajorTenants", FORMAT( SELECTEDVALUE ( DimProperties[MajorTenants] ), "0" ),
 BLANK()
 )
 ```
 * *Note on Property Detail:* This measure is tricky if a loan has multiple properties. The `SELECTEDVALUE(DimProperties[...])` will only work if the filter context (e.g., slicers, or filtering on the property list) results in a single property being active. For the summary section, you might need to explicitly filter for the primary property using `CALCULATE(..., DimProperties[PrimaryFlag] = TRUE())`.

 * Create a third **New measure** (for the main Financial Details section):

 ```DAX
 Financial Detail Value @100% :=
 VAR _SelectedKey = SELECTEDVALUE ( FinancialFieldLabels[DataKey] )
 RETURN
 SWITCH ( TRUE(),
 _SelectedKey = "AuthAmt", FORMAT([Authorized Amount], "$#,0"),
 _SelectedKey = "CurrBal", FORMAT([Current Balance], "$#,0"),
 _SelectedKey = "BalMaturity", FORMAT([Balance at Maturity], "$#,0"), // Using placeholder measure
 _SelectedKey = "AmtFund", FORMAT([Amount to Fund], "$#,0"), // Using placeholder measure
 _SelectedKey = "NextPay", FORMAT([Next Payment Amount], "$#,0"), // Using placeholder measure
 _SelectedKey = "CurrRate", FORMAT([Interest Rate], "0.00%"),
 _SelectedKey = "SpreadIndex", [Spread Display] & " " & [Index Display], // Combine spread & index
 BLANK()
 )
 ```

 * Create a fourth **New measure** (for the ABC Share column):

 ```DAX
 Financial Detail Value ABC Share :=
 VAR _SelectedKey = SELECTEDVALUE ( FinancialFieldLabels[DataKey] )
 RETURN
 SWITCH ( TRUE(),
 _SelectedKey = "AuthAmt", FORMAT([ABC Share Authorized Amount], "$#,0"),
 _SelectedKey = "CurrBal", FORMAT([ABC Share Current Balance], "$#,0"),
 _SelectedKey = "BalMaturity", FORMAT([ABC Share Balance at Maturity], "$#,0"), // Using placeholder measure
 _SelectedKey = "AmtFund", FORMAT([ABC Share Amount to Fund], "$#,0"), // Using placeholder measure
 _SelectedKey = "NextPay", FORMAT([ABC Share Next Payment], "$#,0"), // Using placeholder measure
 // Rate, Spread, Index typically aren't shown split by share, return BLANK or replicate if needed
 _SelectedKey = "CurrRate", BLANK(), // Or FORMAT([Interest Rate], "0.00%") if it applies
 _SelectedKey = "SpreadIndex", BLANK(), 
 BLANK()
 )
 ```

3. **Build the Matrix Visuals:**
 * **Loan Summary Matrix (X:20, Y:320):**
 * Select the Matrix placeholder at these coordinates.
 * Drag `LoanFieldLabels[Label]` to "Rows".
 * Drag `LoanFieldLabels[DisplayOrder]` to "Rows" (above Label). Right-click `DisplayOrder` in the Rows well -> uncheck "Subtotals". *Then hide DisplayOrder later.*
 * Drag `[Loan Detail Value]` measure to "Values".
 * **Sort:** Click the ellipsis (...) on the Matrix visual -> Sort by -> `DisplayOrder`. Click again -> Sort ascending.
 * **Format:**
 * Format Pane -> Visual -> Style presets -> Style: `None`.
 * Format Pane -> Visual -> Grid -> Horizontal gridlines: `Off`, Vertical gridlines: `Off`, Row padding: `4px`, Border: `None`.
 * Format Pane -> Visual -> Column headers: `Off`.
 * Format Pane -> Visual -> Row headers: `Stepped layout`: `Off`. +/- icons: `Off`. Values -> Font `Segoe UI`, Size `11pt`.
 * Format Pane -> Visual -> Values -> Font `Segoe UI`, Size `11pt`. Alignment based on PDF (Labels left, Values right perhaps?).
 * **Hide DisplayOrder Column:** Format Pane -> Visual -> Column Headers. Find `DisplayOrder` specific settings if available, or adjust column width to 0. *Alternative:* In the Fields well, right-click `DisplayOrder` -> "Hide". This might be simpler if possible. *If hiding doesn't work directly*: Select the Matrix, go to Format Pane -> Visual -> Specific column -> Select `DisplayOrder` -> Apply to header: Off, Apply to values: Off. Set Width to 0. *Correction:* Easiest is often: Format Pane -> Visual -> Row headers -> Options -> Stepped layout: Off. Then manually resize the `DisplayOrder` column in the visual itself to be very narrow or zero width if possible. Best way: drag `DisplayOrder` to the *Tooltip* field well instead of rows after initially sorting.
 * **Property Summary Matrix (X:440, Y:320):** (Use the placeholder near the Property Info title)
 * Select the Matrix placeholder.
 * Rows: `PropertyFieldLabels[DisplayOrder]`, then `PropertyFieldLabels[Label]`.
 * Values: `[Property Detail Value]` measure.
 * Sort by `DisplayOrder` ascending. Then drag `DisplayOrder` to Tooltips well.
 * Format identically to the Loan Summary Matrix (Style None, Grids Off, Headers Off, Stepped Off, Fonts).
 * **Financial Details Matrix (X:860, Y:320):**
 * Select the Matrix placeholder.
 * Rows: `FinancialFieldLabels[DisplayOrder]`, then `FinancialFieldLabels[Label]`.
 * Values: `[Financial Detail Value @100%]` AND `[Financial Detail Value ABC Share]`.
 * Sort by `DisplayOrder` ascending. Then drag `DisplayOrder` to Tooltips well.
 * **Format:**
 * Format like the others (Style None, Grids Off, Row Headers Stepped Off, Fonts).
 * **Keep Column Headers ON** for this one. Format Pane -> Visual -> Column headers -> Text: Font `Segoe UI`, Size `11pt`, **Bold**. Alignment: Center/Right as per PDF. Rename headers: Click the down-arrow on the measures in the "Values" well -> Rename for this visual -> `@100%` and `ABC Share`.
 * Format Pane -> Visual -> Specific column -> Select `@100%` -> Align Right. Select `ABC Share` -> Align Right. Select `Label` (Row Header) -> Align Left.
 * Format Pane -> Visual -> Values -> Align Right.

---

## Block 8: Property List & (Separate) Metrics Table (15 min)

* **Purpose:** Display the list of properties associated with the loan and a summary table for financial metrics (this duplicates some info from Block 7's financial list but uses a Table visual as per the coordinate file).

1. **Property Matrix (X:20, Y:520 - wide visual):**
 * Select the Matrix placeholder here.
 * Rows: Drag `DimProperties[Code]`, `DimProperties[Name]`, `DimProperties[City]`, `DimProperties[PrimaryFlag]` onto the "Rows" well.
 * Format Pane -> Visual -> Style presets -> Style: `Minimal` (or None and format manually).
 * Format Pane -> Visual -> Row headers -> Stepped layout: `Off`. +/- icons: `Off`.
 * Format Pane -> Visual -> Values: *(No explicit value columns needed based on PDF)*.
 * Format Pane -> Visual -> Grid -> Borders: `None`. Row Padding: `6px`.
 * Format Pane -> Visual -> Cell elements -> Select series: `PrimaryFlag`. Turn `Background color` ON. Click `fx`.
 * Format style: `Rules`.
 * Field: `DimProperties[PrimaryFlag]`.
 * Rule 1: If value is `true` (or "Yes") then color `#1A2B5C` (Dark Blue). Click New Rule.
 * Rule 2: If value is `false` (or "No") then color `#FFFFFF` (White, or a light grey).
 * Format Pane -> Visual -> Cell elements -> Select series: `PrimaryFlag`. Turn `Font color` ON. Click `fx`.
 * Format style: `Rules`.
 * Field: `DimProperties[PrimaryFlag]`.
 * Rule 1: If value is `true` then color `#FFFFFF` (White).
 * Rule 2: If value is `false` then color `#000000` (Black, or default).
 * Adjust column widths manually by dragging borders in the visual header.

2. **Metrics Table (X:860, Y:520 - placeholder near Financial Details):** *(This seems redundant given the Financial Matrix above it. Let's assume the layout meant the Financial Matrix from Block 7 IS this element. If a separate table visual IS required here, follow these steps, but it will show the same data as the Matrix above).*
 * Select the Table placeholder here.
 * Drag `FinancialFieldLabels[Label]` to "Columns".
 * Drag `[Financial Detail Value @100%]` to "Columns".
 * Drag `[Financial Detail Value ABC Share]` to "Columns".
 * Drag `FinancialFieldLabels[DisplayOrder]` to Tooltips (for sorting).
 * Filter Pane: Filter this visual -> `FinancialFieldLabels[Label]` -> Select the metrics you want *only* in this table (e.g., maybe just the $ amounts).
 * Sort: Click header for `Label` -> Sort by `DisplayOrder`.
 * Format Pane -> Visual -> Style presets -> Style: `None`.
 * Format Pane -> Visual -> Grid -> Off.
 * Format Pane -> Visual -> Column headers: Format as needed (Bold, Size 11pt). Rename headers `@100%`, `ABC Share`.
 * Format Pane -> Visual -> Values -> Font 11pt. Align Right.
 * Format Pane -> General -> Title: Off.
 * *Decision:* It's highly likely the "Metrics Table" in the coordinate list refers to the Financial Details *Matrix* created in Block 7 (X:860, Y:320). Let's proceed assuming that, and ignore this separate Table visual unless explicitly needed.

---

## Block 9: Configure Lower Panels (Participants, Metrics, Insurance/Tax) (25 min)

* **Purpose:** Populate the three distinct panels at the bottom of the report.

1. **Panel Backgrounds & Borders (Optional but helps visually group):**
 * Insert -> Shapes -> Rectangle.
 * Size and position it to cover the area for the "Participants" panel (approx X:20, Y:820, W:400, H:400 based on coordinates of visuals within it).
 * Format Pane -> Shape -> Style -> Fill: Off. Border: On, Color `#E0E0E0`, Width 1px, Round corners: 4px.
 * Format Pane -> General -> Effects -> Shadow: On (Theme default).
 * Send to Back: Right-click shape -> Arrange -> Send to back.
 * Copy/Paste this rectangle shape and resize/reposition it for the "Key Metrics & Appraisal" panel (X:420, Y:820) and "Insurance & Tax" panel (X:860, Y:820).

2. **Participants Panel (Inside first rectangle, approx X:20, Y:820):**
 * **Lenders Table:**
 * Select the Table placeholder (coords approx X:20, Y:850).
 * Columns: Drag `DimLenders[Entity]`, `DimLenders[SharePct]`, `DimLenders[ShareAmt]` onto the "Columns" well.
 * Format Pane -> Visual -> Values -> `SharePct` -> Display Units: None, Decimal Places: 0. Set Format to Percentage in the Data/Model view if not done already.
 * Format Pane -> Visual -> Values -> `ShareAmt` -> Display Units: None, Decimal Places: 0. Format as Currency ($#,0).
 * Format Pane -> Visual -> Style: None. Grid: Off.
 * Format Pane -> Visual -> Column Headers -> Font: Segoe UI, 11pt, Bold. Text Color: Dark Blue. Background: White.
 * Format Pane -> Visual -> Values -> Font: Segoe UI, 11pt.
 * Format Pane -> General -> Title: `On`. Text: `Lenders`. Font: Segoe UI, 12pt, Bold. Color: Dark Blue.
 * **Borrowers & Guarantors Table:**
 * Select the Table placeholder below Lenders (coords approx X:20, Y:?)
 * Columns: Drag `DimParticipants[Role]`, `DimParticipants[Entity]`, `DimParticipants[SharePct]` onto "Columns".
 * Format Pane -> Visual -> Values -> `SharePct` -> Display Units: None, Decimal Places: 0. Format as Percentage.
 * Format similarly to Lenders table (Style None, Grid Off, Headers Bold, Values 11pt).
 * Format Pane -> General -> Title: `On`. Text: `Borrowers & Guarantors`. Font: Segoe UI, 12pt, Bold. Color: Dark Blue.

3. **Key Metrics & Appraisal Panel (Inside second rectangle, approx X:420, Y:820):**
 * *(Note: RCD, RPC, RPV, RBV measures were not defined earlier. Assume they exist or create placeholder measures returning numbers/percentages for now).*
 * **Create Placeholder Measures (if needed):**
 ```DAX
 RCD = 0.65 
 RPC = 0.72
 RPV = 0.68
 RBV (Initial) = 0.70 
 ```
 *Format these as Percentage with 0 decimal places.*
 * **KPI Cards (Row approx Y:850):**
 * Place 4 standard Card visuals side-by-side.
 * Card 1: Field `[RCD]`. Category Label: "RCD". Callout Size: `32pt`.
 * Card 2: Field `[RPC]`. Category Label: "RPC". Callout Size: `32pt`.
 * Card 3: Field `[RPV]`. Category Label: "RPV". Callout Size: `32pt`.
 * Card 4: Field `[RBV (Initial)]`. Category Label: "RBV (Initial)". Callout Size: `32pt`.
 * **Format All Cards:** Category Label Size 11pt Grey. Callout Value Dark Blue. General->Effects->Border/Shadow Off (let panel border define edge). General->Title: Off.
 * **(Optional) Conditional Formatting & Arrows:**
 * Select RCD Card -> Format -> Visual -> Callout value -> Color -> `fx` icon.
 * Format style: Rules. Field: `[RCD]`. Rule 1: If value >= 0.7 Then Green (`#2ECC71`). Rule 2: If value >= 0.5 AND < 0.7 Then Yellow. Rule 3: If value < 0.5 Then Red.
 * Repeat for RPC, RPV cards.
 * *(Arrows require more complex DAX using UNICHAR symbols based on comparison to previous values - skip for beginner version unless critical).*
 * **Appraisal Details Table:**
 * Select Table placeholder below cards (approx Y:950).
 * Columns: Drag `DimAppraisals[AppraisalDate]`, `DimAppraisals[AppraisalType]`, `DimAppraisals[Valuation]` onto "Columns".
 * Format Date: Select `AppraisalDate` column in Data/Model view -> Format: Short Date (or custom).
 * Format Valuation: Format as Currency ($#,0).
 * Format Table: Style None, Grid Off, Headers Bold 11pt, Values 11pt.
 * Format Pane -> General -> Title: `On`. Text: `Appraisal Details`. Font: Segoe UI, 12pt, Bold. Color: Dark Blue.
 * **Appraiser Comments Text Box:**
 * Insert -> Text Box. Position below Appraisal table.
 * Type in the comments from the PDF. Format text (11pt Segoe UI).
 * Format -> General -> Title: `On`. Text: `Appraiser Comments`. Font: Segoe UI, 12pt, Bold. Color: Dark Blue.

4. **Insurance & Tax Panel (Inside third rectangle, approx X:860, Y:820):**
 * **Insurance Table:**
 * Select Table placeholder (approx Y:850).
 * Columns: Drag `DimInsurance[InsuranceType]`, `DimInsurance[Provider]`, `DimInsurance[ExpiryDate]` onto "Columns".
 * Format Date: `ExpiryDate` -> Short Date format.
 * Format Table: Style None, Grid Off, Headers Bold 11pt, Values 11pt.
 * Format Pane -> General -> Title: `On`. Text: `Insurance`. Font: Segoe UI, 12pt, Bold. Color: Dark Blue.
 * **Tax Table:**
 * Select Table placeholder below Insurance (approx Y:?).
 * Columns: Drag `DimTaxes[PropertyName]`, `DimTaxes[DueDate]`, `DimTaxes[Amount]` onto "Columns". *(Link `PropertyName` from `DimProperties` if needed via `PropertyID` relationship)*.
 * Format Date: `DueDate` -> Short Date format.
 * Format Amount: Currency ($#,0).
 * Format Table: Style None, Grid Off, Headers Bold 11pt, Values 11pt.
 * Format Pane -> General -> Title: `On`. Text: `Tax`. Font: Segoe UI, 12pt, Bold. Color: Dark Blue.

---

## Block 10: Add Hover "Info" Icons (Tooltips) (20 min)

* **Purpose:** Add small 'i' icons next to certain fields that show helpful descriptions when hovered over.

1. **Create Tooltip Content Table:** We need a table mapping a key (which field the tooltip is for) to the message to display.
 * Home -> Enter Data.
 * Column1: `InfoKey` (Text), Column2: `InfoMessage` (Text).
 * Enter rows like:
 | InfoKey | InfoMessage |
 |-----------------|---------------------------------------------------|
 | Status | Current status of the loan (Active, Paid Off...). |
 | RiskRating | Internal risk assessment score/category. |
 | AuthorizedAmt | The total amount approved for the loan. |
 | *(Add more rows for each field needing a tooltip)* | |
 * Table Name: `InfoDictionary`. Click Load.
2. **Create Tooltip Page:**
 * Create a **New Page** (click the '+' at the bottom).
 * **Rename Page:** Right-click the new page tab -> Rename -> `tt_Info` (lowercase 'tt_' is a convention).
 * **Page Settings:** With the blank `tt_Info` page selected (no visuals selected), go to Format Pane -> Page information -> Allow use as tooltip: `On`.
 * **Canvas Settings:** Format Pane -> Canvas settings -> Type: `Tooltip`. (Size defaults are usually fine, adjust Width/Height if needed later e.g., 300W x 120H).
3. **Create Tooltip Content Visual:**
 * On the `tt_Info` page, insert a **Card** visual.
 * Create a **New Measure** for the tooltip text:
 ```DAX
 Info Tooltip Message :=
 // Looks up the message based on the key passed from the button
 LOOKUPVALUE(
 InfoDictionary[InfoMessage],
 InfoDictionary[InfoKey], SELECTEDVALUE ( InfoDictionary[InfoKey] ) // This seems circular, need the key from the button
 )
 // Correction: The key needs to be passed TO the tooltip page.
 // The measure on the tooltip page should just display the relevant field 
 // passed from the main page's button. Let's try a simpler approach.
 // On tt_Info page, add a CARD visual.
 // We will pass the InfoKey to this page via the button setup.
 // The card's field should be this measure:
 Selected Info Message := SELECTEDVALUE( InfoDictionary[InfoMessage] ) 
 ```
 * Drag the `[Selected Info Message]` measure onto the Card's "Fields" well on the `tt_Info` page.
 * Format the Card: Turn Category Label Off. Adjust Callout Value font size/color as desired (e.g., 10pt Segoe UI). Remove Card background/border if you want it borderless.
4. **Hide Tooltip Page:** Right-click the `tt_Info` page tab -> "Hide Page". Hidden pages can still be used as tooltips.
5. **Add Info Buttons to Main Page:**
 * Go back to your main report page.
 * Find the first place you need an info icon (e.g., next to the "Status" slicer header, or next to a field in the Loan Summary list).
 * Insert -> Buttons -> Navigator -> **Info** (it's a small 'i' icon button).
 * **Resize:** Select the button. Format Pane -> General -> Properties -> Size -> Set Height/Width to `14` x `14` pixels.
 * **Position:** Place it precisely next to the field it relates to.
 * **Format Icon:** Format Pane -> Button -> Icon -> Color: `#1A2B5C`. Transparency: 0%.
 * **Format Hover (Optional):** Format Pane -> Button -> Style -> Apply settings to: `On hover`. Set Icon -> Color: maybe a lighter blue or keep dark blue.
 * **Set Action:** Format Pane -> Button -> Action -> Action: `On`. Type: `Page navigation`. Destination: `tt_Info`. *Correction:* Action Type should be `Tooltip`.
 * Format Pane -> Button -> Action -> Action: `On`. Type: `Tooltip`. Tooltip: `tt_Info` (Report Page).
 * **Crucially: Link the Data:** In the "Visualizations" pane, find the "Tooltip" data field well (it appears when Action->Tooltip is on). Drag the `InfoDictionary[InfoKey]` field into this well.
 * **Filter for This Button:** Go to the "Filters" pane. Add a filter *on this visual* (the info button). Filter type: Basic filtering. Select the specific `InfoKey` value that corresponds to this button (e.g., filter `InfoDictionary[InfoKey]` is "Status").
 * **Accessibility:** Format Pane -> General -> Properties -> Alt Text -> Enter helpful text like "More info about Status".
6. **Copy and Configure Other Buttons:**
 * Copy (Ctrl+C) the configured info button.
 * Paste (Ctrl+V) it next to another field (e.g., Risk Rating).
 * Select the *new* button.
 * Go to the "Filters" pane -> change the filter on *this visual* to the new relevant `InfoKey` (e.g., "RiskRating").
 * Update the Alt Text.
 * Repeat pasting and re-filtering for every info icon needed.

---

## Block 11: Performance & QA (Quality Assurance) (15 min)

* **Purpose:** Check if the report loads quickly and filters correctly, and ensure visual consistency.

1. **Performance Analyzer:**
 * Go to the "View" ribbon -> Check "Performance analyzer". A new pane appears.
 * Click "Start recording".
 * Interact with the report: Click different slicers (LoanID, Status, etc.). Scroll up and down.
 * Click "Stop recording".
 * Examine the durations in the Performance Analyzer pane. Pay attention to "DAX query" and "Visual display". Anything consistently taking > 120 milliseconds (ms) might warrant investigation.
 * **If slow:** Copy the DAX query from the analyzer -> Go to External Tools -> DAX Studio -> Paste query -> Analyze/Run. See if the query plan suggests issues (e.g., slow relationships, inefficient filtering). Optimization is advanced, but this tells you *which* visual/measure is slow.
2. **Check Filtering:**
 * Click the "LoanID" slicer -> Select a specific loan. Do ALL visuals update correctly to show data ONLY for that loan?
 * Click other slicers (Status, Country, etc.). Do the relevant visuals filter as expected? (Ctrl+Click to multi-select if enabled).
 * Clear all slicers (click the eraser icon on each). Does the report show aggregated/default data correctly?
3. **Visual QA:**
 * **Font Consistency:** View -> Selection pane. Ctrl + A to select all visuals on the page. Go to Format Pane -> Text -> Font Family. If it says "Mixed", you have inconsistent fonts somewhere. Check Text Boxes, Card Labels, Axis Labels, etc. Make sure all are `Segoe UI`.
 * **Border/Shadow Consistency:** Visually scan all Cards and Panels. Do they all have the same border radius (4px) and shadow effect?
 * **Alignment:** Zoom in (Ctrl + Mouse Wheel) and check if visuals are perfectly aligned according to the grid/coordinates. Check spacing between elements.
 * **Scrolling:** Press `Ctrl + Home` to go to the top. Use `PageDown` key or scroll bar to go all the way down. Are there any awkward gaps? Do any visuals overlap? Is there any *horizontal* scrollbar at the bottom (there shouldn't be if Width=1280)?
4. **Data Accuracy (Spot Check):** Compare a few key numbers (e.g., Authorized Amount, Current Balance for a specific loan) against your source data or the PDF mockup to ensure the measures and relationships are working correctly.

---

## Block 12: Save & Share (5 min)

* **Purpose:** Save your work and potentially create a template for future reports.

1. **Save as PBIX:**
 * File -> Save As.
 * Choose a location (your project folder).
 * File name: e.g., `Loan Overview Report V1.pbix`.
 * Click Save. **Save frequently during development!**
 * The `.pbix` file contains the report layout, data model, queries, measures, and (optionally) imported data.
2. **Export as PBIT (Template):**
 * File -> Export -> Power BI template (`.pbit`).
 * Enter a template description if desired. Click OK.
 * Save the `.pbit` file (e.g., `Loan Report Template.pbit`).
 * The `.pbit` file contains everything *except* the imported data. When someone opens a PBIT, they are prompted to connect to the data sources, but the entire report structure, theme, measures, and layout are pre-built. This is great for standardizing reports.
3. **Publish to Power BI Service (Optional):**
 * If you have a Power BI Pro/Premium account and want to share online:
 * Home ribbon -> "Publish".
 * Choose a Workspace. Click Select.
 * Once published, you can access it via app.powerbi.com and share links with colleagues (permissions required).
4. **Enable Power BI Project (pbip) + Git (Advanced - Optional):**
 * For better version control, especially if working in a team:
 * File -> Options and settings -> Options -> Preview features -> Check "Power BI project (.pbip) save option". Restart Power BI Desktop.
 * Now, File -> Save As allows saving as `.pbip`. This saves the report definition (layout, model metadata) as separate text files (JSON, TMDL) in a folder structure, which works very well with Git version control systems (like GitHub, Azure DevOps).

---

### You're Done!

Following these 12 blocks meticulously should result in a Power BI report that very closely matches the provided PDF. Remember that getting data connections right (Block 1) and ensuring DAX measures accurately reflect your business logic (Block 2 & 7) are the most critical parts beyond just visual replication.
