---
tags:
  - SQL
---
Of course. You can do this by looping through your IDs and inserting the results from each stored procedure call into a temporary table first. After the loop finishes, you can move the combined results into a final, permanent table.

The most common way to loop through records is by using a **CURSOR**.

# Using a CURSOR to Save Results

Here's how you can call a stored procedure for each ID and save the combined records.

**Scenario:**

- **Source Table:** `dbo.SourceIDs` (contains the `ID` column).
    
- **Stored Procedure:** `dbo.GetDetailsForID` (takes an `@ID` parameter and returns a list of records).
    
- **Destination Table:** `dbo.CombinedResults` (will store all records from all procedure calls).
    

**Example Code:**
```SQL
-- 1. Define the table where you'll store the final, combined results.
--    Its structure must match the output of your stored procedure.
CREATE TABLE dbo.CombinedResults (
    ResultID INT,
    DetailValue NVARCHAR(255),
    RecordDate DATE
);
GO

-- 2. Create a temporary table to hold results during the loop.
--    This is the best practice for accumulating results from INSERT...EXEC.
CREATE TABLE #TempResults (
    ResultID INT,
    DetailValue NVARCHAR(255),
    RecordDate DATE
);

-- 3. Declare the variable for the ID and the cursor to loop through your source table.
DECLARE @CurrentID INT;

DECLARE id_cursor CURSOR FOR
SELECT ID FROM dbo.SourceIDs;

-- 4. Open the cursor and start the loop.
OPEN id_cursor;
FETCH NEXT FROM id_cursor INTO @CurrentID;

WHILE @@FETCH_STATUS = 0
BEGIN
    -- For each ID, execute the procedure and insert its results into the TEMP table.
    INSERT INTO #TempResults (ResultID, DetailValue, RecordDate)
    EXEC dbo.GetDetailsForID @ID = @CurrentID;

    -- Get the next ID from the source table.
    FETCH NEXT FROM id_cursor INTO @CurrentID;
END;

-- 5. Close and deallocate the cursor.
CLOSE id_cursor;
DEALLOCATE id_cursor;

-- 6. Now, move all the collected records from the temporary table to your final destination table.
INSERT INTO dbo.CombinedResults (ResultID, DetailValue, RecordDate)
SELECT ResultID, DetailValue, RecordDate FROM #TempResults;

-- 7. Clean up the temporary table.
DROP TABLE #TempResults;

-- 8. Verify the final results.
SELECT * FROM dbo.CombinedResults;

```

---

# Alternative: Using a `WHILE` Loop

A `WHILE` loop can be slightly more performant than a cursor but involves a bit more setup.

SQL

```
-- Create a temporary table with an identity column to manage the loop.
SELECT 
    ROW_NUMBER() OVER (ORDER BY ID) AS RowNum, 
    ID 
INTO #SourceIDsTemp
FROM dbo.SourceIDs;

-- Create the temp table for the results.
CREATE TABLE #TempResults (
    ResultID INT,
    DetailValue NVARCHAR(255)
);

DECLARE @i INT = 1;
DECLARE @max INT = (SELECT COUNT(*) FROM #SourceIDsTemp);
DECLARE @CurrentID INT;

-- Loop through each ID.
WHILE @i <= @max
BEGIN
    -- Get the ID for the current row.
    SELECT @CurrentID = ID FROM #SourceIDsTemp WHERE RowNum = @i;

    -- Execute the procedure and save the results.
    INSERT INTO #TempResults (ResultID, DetailValue)
    EXEC dbo.GetDetailsForID @ID = @CurrentID;

    SET @i = @i + 1;
END;

-- Move results to a final table.
INSERT INTO dbo.CombinedResults (ResultID, DetailValue)
SELECT ResultID, DetailValue FROM #TempResults;

-- Clean up.
DROP TABLE #SourceIDsTemp;
DROP TABLE #TempResults;
```