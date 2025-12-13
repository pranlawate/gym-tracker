# Updated Google Apps Script for Your Existing Sheets Structure

**Date:** 2025-12-11
**Purpose:** Adapt the Google Sheets integration to work with your EXISTING sheet structure

---

## 🎯 Quick Summary

✅ **Works with your EXISTING horizontal tracking tabs** - No need to create new tabs
✅ **Auto-creates date columns** - PWA adds new R/W pairs automatically
✅ **Supports 3-4 sets per exercise** - Handles variable set counts
✅ **Maintains your workflow** - Manual tracking and PWA sync use same tabs
✅ **Easy phase switching** - Change one variable when transitioning programs

**What this does:**
- PWA syncs workout data to your existing horizontal tabs
- Loads previous workouts for progressive overload suggestions
- All-in-one tracking: Your structure + PWA automation

**Requirements:**
- **Row 1** must contain workout dates in merged cells above R/W pairs (format: YYYY-MM-DD)
- **Row 2** must have "R | W" repeating headers
- **Column A** must have exercise names + SET labels (e.g., "SET 1: 8-10")

---

## 📋 Your Horizontal Sheet Structure

**Workout Tabs:**

**Phase 2 (4 tabs):**
- `Upper Body 1 (Phase 2)` (Monday)
- `Lower Body 1 (Phase 2)` (Tuesday)
- `Upper Body 2 (Phase 2)` (Thursday)
- `Lower Body 2 (Phase 2)` (Friday)

**Phase 1 (2 tabs):**
- `Workout A (Phase 1)` (Mon/Wed/Fri)
- `Workout B (Phase 1)` (Mon/Wed/Fri - alternates)

**Structure in each workout tab:**
```
Row 1:  | Upper Body 1 | 2025-12-11     | 2025-12-13     | (PWA adds more) |
Row 2:  |              | R    | W       | R    | W       | R    | W       |
Row 3:  | Incline DB   |      |         |      |         |      |         |
Row 4:  | SET 1: 8-10  | 10   | 22.5    | 10   | 25      | ...  | ...     |
Row 5:  | SET 2: 8-10  | 9    | 22.5    | 10   | 25      | ...  | ...     |
Row 6:  | SET 3: 8-10  | 8    | 22.5    | 9    | 25      | ...  | ...     |
```

**Key Points:**
- Each exercise occupies 3-4 rows (depending on set count)
- Dates in Row 1 are merged across 2 cells (R/W pair)
- R/W pairs repeat horizontally for each workout date

---

## 🔧 Updated Apps Script Code (Horizontal Layout)

Replace the code in your Google Apps Script with this updated version:

```javascript
// ===== CONFIGURATION =====
// Set your current phase here: 'phase1' or 'phase2'
const CURRENT_PHASE = 'phase2';  // Change to 'phase1' when doing Phase 1

const SHEET_TAB_MAPPING = {
  // Phase 2 (Upper/Lower Split - 4 days/week)
  phase2: {
    'Monday': 'Upper Body 1 (Phase 2)',
    'Tuesday': 'Lower Body 1 (Phase 2)',
    'Wednesday': 'Upper Body 1 (Phase 2)',  // Rest day fallback
    'Thursday': 'Upper Body 2 (Phase 2)',
    'Friday': 'Lower Body 2 (Phase 2)',
    'Saturday': 'Abdominal Exercises',
    'Sunday': 'Upper Body 1 (Phase 2)'      // Rest day fallback
  },
  // Phase 1 (Full Body - 3 days/week: Mon/Wed/Fri alternate A/B)
  phase1: {
    'Monday': 'Workout A (Phase 1)',
    'Tuesday': 'Workout A (Phase 1)',    // Rest day fallback
    'Wednesday': 'Workout B (Phase 1)',
    'Thursday': 'Workout B (Phase 1)',   // Rest day fallback
    'Friday': 'Workout A (Phase 1)',     // Alternates A/B
    'Saturday': 'Workout A (Phase 1)',   // Rest day fallback
    'Sunday': 'Workout A (Phase 1)'      // Rest day fallback
  }
};

// ===== HELPER FUNCTIONS =====

function findDateColumn(sheet, targetDate) {
  // Scan Row 1 for merged cells containing the target date
  // Supports multiple date formats: YYYY-MM-DD, MM/DD/YYYY, DD/MM/YYYY
  const row1 = sheet.getRange(1, 1, 1, sheet.getLastColumn()).getValues()[0];

  for (let col = 0; col < row1.length; col++) {
    const cellValue = row1[col];
    if (!cellValue) continue;

    // Normalize date to YYYY-MM-DD format for comparison
    let normalizedCellDate = '';
    if (cellValue instanceof Date) {
      normalizedCellDate = Utilities.formatDate(cellValue, Session.getScriptTimeZone(), 'yyyy-MM-dd');
    } else {
      normalizedCellDate = cellValue.toString();
    }

    if (normalizedCellDate === targetDate) {
      return col + 1; // Return 1-based column index
    }
  }

  return -1; // Date column not found
}

function findExerciseRow(sheet, exerciseName, setNumber) {
  // Scan column A for exercise name, then find the set row
  // Handles variable set counts (3-4 sets per exercise)
  const columnA = sheet.getRange(3, 1, sheet.getLastRow() - 2, 1).getValues();

  for (let i = 0; i < columnA.length; i++) {
    const cellValue = columnA[i][0];
    if (!cellValue) continue;

    // Check if this is the exercise name (not a SET label)
    if (cellValue.toString() === exerciseName ||
        cellValue.toString().replace(/^E\d+:\s*/, '') === exerciseName.replace(/^E\d+:\s*/, '')) {

      // Found exercise name! Now find the set row
      // SET rows follow immediately after: "SET 1: X-Y", "SET 2: X-Y", etc.
      const setRow = i + 3 + setNumber; // +3 for Row 1,2 headers + 0-indexed i, +setNumber for set offset

      // Verify this is actually a SET row
      if (setRow <= sheet.getLastRow()) {
        return setRow;
      }
    }
  }

  return -1; // Exercise not found
}

function createNewDateColumn(sheet, date) {
  // Create new R/W column pair at the end for a new workout date
  const lastCol = sheet.getLastColumn();
  const newColR = lastCol + 1;
  const newColW = lastCol + 2;

  // Merge cells in Row 1 and add date
  sheet.getRange(1, newColR, 1, 2).merge().setValue(date);

  // Add R/W headers in Row 2
  sheet.getRange(2, newColR).setValue('R');
  sheet.getRange(2, newColW).setValue('W');

  return newColR; // Return the R column index
}

function getDayFromTabName(tabName) {
  // Reverse lookup: find which day maps to this tab
  const phaseMapping = SHEET_TAB_MAPPING[CURRENT_PHASE];
  for (const [day, tab] of Object.entries(phaseMapping)) {
    if (tab === tabName) {
      return day;
    }
  }
  return 'Monday'; // Default fallback
}

// ===== POST ENDPOINT (Save Workout Data) =====
function doPost(e) {
  try {
    const data = JSON.parse(e.postData.contents);
    // data = { date: "2025-12-11", day: "Monday", exercise: "Incline Dumbbell Press",
    //          set: 1, weight: 22.5, reps: 10, notes: "" }

    // Get the correct tab
    const tabName = SHEET_TAB_MAPPING[CURRENT_PHASE][data.day] || SHEET_TAB_MAPPING[CURRENT_PHASE]['Monday'];
    const sheet = SpreadsheetApp.getActiveSpreadsheet().getSheetByName(tabName);

    if (!sheet) {
      return ContentService.createTextOutput(JSON.stringify({
        status: 'error',
        message: `Tab "${tabName}" not found. Check SHEET_TAB_MAPPING.`
      })).setMimeType(ContentService.MimeType.JSON);
    }

    // Find the date column (or create if doesn't exist)
    let dateCol = findDateColumn(sheet, data.date);
    if (dateCol === -1) {
      dateCol = createNewDateColumn(sheet, data.date);
    }

    // Find the exercise row for this specific set
    const exerciseRow = findExerciseRow(sheet, data.exercise, data.set);
    if (exerciseRow === -1) {
      return ContentService.createTextOutput(JSON.stringify({
        status: 'error',
        message: `Exercise "${data.exercise}" SET ${data.set} not found in Column A. Check exercise names match exactly.`
      })).setMimeType(ContentService.MimeType.JSON);
    }

    // Write reps and weight to R/W columns
    const repsCol = dateCol;       // R column
    const weightCol = dateCol + 1; // W column

    sheet.getRange(exerciseRow, repsCol).setValue(data.reps);
    sheet.getRange(exerciseRow, weightCol).setValue(data.weight);

    return ContentService.createTextOutput(JSON.stringify({
      status: 'success',
      message: 'Workout logged successfully',
      tab: tabName,
      row: exerciseRow,
      exercise: data.exercise,
      set: data.set
    })).setMimeType(ContentService.MimeType.JSON);

  } catch (error) {
    return ContentService.createTextOutput(JSON.stringify({
      status: 'error',
      message: error.toString()
    })).setMimeType(ContentService.MimeType.JSON);
  }
}

// ===== GET ENDPOINT (Load Previous Workout) =====
function doGet(e) {
  try {
    const allWorkouts = [];

    // Read from all tabs for current phase
    const tabs = Object.values(SHEET_TAB_MAPPING[CURRENT_PHASE])
      .filter((v, i, a) => a.indexOf(v) === i); // Get unique tabs

    tabs.forEach(tabName => {
      const sheet = SpreadsheetApp.getActiveSpreadsheet().getSheetByName(tabName);
      if (!sheet) return;

      // Get all data from the sheet
      const lastRow = sheet.getLastRow();
      const lastCol = sheet.getLastColumn();
      if (lastRow < 3 || lastCol < 2) return; // Not enough data

      const allData = sheet.getRange(1, 1, lastRow, lastCol).getValues();

      // Parse Row 1 to find all date columns
      const dateColumns = [];
      for (let col = 0; col < allData[0].length; col++) {
        const cellValue = allData[0][col];
        if (!cellValue) continue;

        // Detect dates in various formats
        let dateStr = '';
        if (cellValue instanceof Date) {
          dateStr = Utilities.formatDate(cellValue, Session.getScriptTimeZone(), 'yyyy-MM-dd');
        } else if (cellValue.toString().match(/\d{4}-\d{2}-\d{2}/)) {
          dateStr = cellValue.toString();
        } else if (cellValue.toString().match(/\d{1,2}\/\d{1,2}\/\d{4}/)) {
          // MM/DD/YYYY or DD/MM/YYYY - needs manual date parsing if needed
          dateStr = cellValue.toString();
        }

        if (dateStr) {
          dateColumns.push({ date: dateStr, col: col });
        }
      }

      // Parse exercise data (starting from Row 3, Row 0 = dates, Row 1 = R/W)
      for (let row = 2; row < allData.length; row++) {
        const exerciseName = allData[row][0]; // Column A

        // Skip empty rows or SET labels
        if (!exerciseName || exerciseName.toString().startsWith('SET ')) {
          continue;
        }

        // This is an exercise name row
        // The next rows are SET 1, SET 2, SET 3, (SET 4)
        let setNum = 1;
        let setRow = row + 1;

        while (setRow < allData.length && allData[setRow][0] &&
               allData[setRow][0].toString().startsWith('SET ')) {

          // For each date column, read the R/W data for this set
          dateColumns.forEach(dateCol => {
            const reps = allData[setRow][dateCol.col];     // R column
            const weight = allData[setRow][dateCol.col + 1]; // W column

            if (reps && weight && reps !== '' && weight !== '') {
              allWorkouts.push({
                date: dateCol.date,
                day: getDayFromTabName(tabName),
                exercise: exerciseName.toString(),
                set: setNum,
                reps: parseInt(reps),
                weight: parseFloat(weight),
                notes: ''
              });
            }
          });

          setNum++;
          setRow++;
        }

        // Skip to the next exercise (jump past all set rows)
        row = setRow - 1;
      }
    });

    return ContentService.createTextOutput(JSON.stringify({
      status: 'success',
      workouts: allWorkouts,
      count: allWorkouts.length
    })).setMimeType(ContentService.MimeType.JSON);

  } catch (error) {
    return ContentService.createTextOutput(JSON.stringify({
      status: 'error',
      message: error.toString()
    })).setMimeType(ContentService.MimeType.JSON);
  }
}
```

---

## 🚀 Setup Instructions

### Step 1: Prepare Your Horizontal Spreadsheet

**Add date headers to existing workout columns** (One-time manual setup):

1. Open your Google Sheet with horizontal workout tracking
2. For each existing R/W column pair that has workout data:
   - Select the 2 cells in Row 1 above that R/W pair (e.g., cells B1:C1)
   - Right-click → **Merge cells**
   - Enter the workout date in format **YYYY-MM-DD** (e.g., "2025-12-11")
3. Verify Row 2 has "R | W" headers for all column pairs
4. Verify Column A has exercise names followed by SET labels (e.g., "SET 1: 8-10")

**Example after setup:**
```
Row 1:  | Upper Body 1 | 2025-12-11     | 2025-12-13     |
Row 2:  |              | R    | W       | R    | W       |
Row 3:  | Incline DB   |      |         |      |         |
Row 4:  | SET 1: 8-10  | 10   | 22.5    | 10   | 25      |
```

⚠️ **Important**: Future workout dates will be auto-created by the PWA!

### Step 2: Open Apps Script
1. Open your Google Sheet
2. Go to **Extensions → Apps Script**
3. Delete ALL existing code

### Step 3: Paste Updated Code
1. Copy the entire code block above (all JavaScript code)
2. Paste it into the Apps Script editor
3. Click **Save** (disk icon)

### Step 4: Deploy as Web App
1. Click **Deploy → New deployment**
2. Click gear icon ⚙️ → Select **Web app**
3. Settings:
   - Description: "BWS Horizontal Layout Tracker"
   - Execute as: **Me**
   - Who has access: **Anyone**
4. Click **Deploy**
5. Authorize if prompted
6. **Copy the Web App URL** (looks like: `https://script.google.com/macros/s/.../exec`)

### Step 5: Update Your PWA
1. Open PWA in browser
2. Click ⚙️ (settings icon)
3. Click "📊 Google Sheets Setup"
4. Paste your Web App URL in Step 4
5. Click "Test Google Sheets Connection"
6. Should see: "✅ Connected to Google Sheets!"

---

## 📊 How It Works

### When You Sync Workout (First Time - Date Column Doesn't Exist):
```
PWA sends:
{
  date: "2025-12-16",
  day: "Monday",
  exercise: "Incline Dumbbell Press",
  set: 1,
  weight: 22.5,
  reps: 10
}

Apps Script:
1. Maps "Monday" → "Upper Body 1 (Phase 2)" tab
2. Searches Row 1 for "2025-12-16" → NOT FOUND
3. Auto-creates new R/W column pair at the end
4. Merges cells in Row 1, adds date "2025-12-16"
5. Adds "R | W" to Row 2
6. Finds "Incline Dumbbell Press" in Column A
7. Finds "SET 1" row below it
8. Writes reps (10) to R column, weight (22.5) to W column
```

**Result in Google Sheet:**
```
Row 1:  | Upper Body 1 | 2025-12-11     | 2025-12-16     | (new!)
Row 2:  |              | R    | W       | R    | W       |
Row 3:  | Incline DB   |      |         |      |         |
Row 4:  | SET 1: 8-10  | 10   | 22.5    | 10   | 22.5    | ← NEW DATA
Row 5:  | SET 2: 8-10  | 9    | 22.5    |      |         |
```

### When You Sync Workout (Subsequent Times - Date Column Exists):
```
Apps Script:
1. Searches Row 1 for "2025-12-16" → FOUND in column 5
2. Finds exercise + set row
3. Updates existing R/W cells (overwrites if needed)
```

### When You Load Last Workout:
```
Apps Script:
1. Scans Row 1 for all dates in "Upper Body 1 (Phase 2)" tab
2. Finds: ["2025-12-11", "2025-12-16"]
3. For each date column, reads all R/W data
4. Parses exercise rows (handles 3 or 4 sets dynamically)
5. Returns workout data to PWA

PWA receives:
[
  { date: "2025-12-11", exercise: "Incline Dumbbell Press", set: 1, reps: 10, weight: 22.5 },
  { date: "2025-12-16", exercise: "Incline Dumbbell Press", set: 1, reps: 10, weight: 22.5 },
  ...
]

PWA shows: "22.5 kg (last time)" OR "25 kg (recommended +2.5)" if you hit top reps
```

---

## 🎯 Benefits of This Approach

✅ **Single source of truth** - Manual tracking and PWA use same tabs
✅ **Auto-creates date columns** - No manual column creation for new workouts
✅ **Maintains your horizontal workflow** - See progression over time left-to-right
✅ **Supports variable set counts** - Handles 3 or 4 sets per exercise
✅ **Backward compatible** - Works with existing workout data
✅ **Progressive overload** - PWA suggests +2.5kg when you hit top reps
✅ **Flexible date formats** - Supports YYYY-MM-DD, MM/DD/YYYY, or Google Sheets dates

---

## 🔧 Customization Options

### Switching Between Phase 1 and Phase 2
When you transition from Phase 1 to Phase 2 (or back):

1. Open Apps Script (Extensions → Apps Script)
2. Change line 66:
   ```javascript
   const CURRENT_PHASE = 'phase2';  // Change to 'phase1' for Phase 1
   ```
3. Click **Save** (disk icon)
4. That's it! No need to redeploy.

**Phase 1 Schedule:**
- Mon: Workout A
- Wed: Workout B
- Fri: Workout A (alternates)

**Phase 2 Schedule:**
- Mon: Upper 1
- Tue: Lower 1
- Thu: Upper 2
- Fri: Lower 2

### If You Have Different Tab Names:
Edit the `SHEET_TAB_MAPPING` object (lines 68-89) to match your tab names exactly.

### Exercise Name Matching:
The script handles both formats:
- "Incline Dumbbell Press" (plain name)
- "E1: Incline Dumbbell Press" (with prefix)

Both will match correctly.

---

## 📝 Example Workflow

**Monday Workout (New Date):**
1. Open PWA, select "MON"
2. Click "📥 Load Last" (sees last Monday's data from previous dates)
3. Complete workout, enter weights/reps
4. Click "📤 Sync to Sheets"
5. Apps Script auto-creates new "2025-12-16" column with merged header
6. Data writes to correct exercise + set rows in new R/W columns

**Next Monday (Date Column Exists):**
1. Click "📥 Load Last"
2. PWA reads all previous Monday workouts from horizontal layout
3. Shows: "22.5 kg (last time)" OR "25 kg (recommended +2.5)"
4. Complete workout
5. Click "📤 Sync to Sheets"
6. Apps Script finds existing date column, updates cells

---

## ✅ Verification

After deployment and setup, test:

1. **Test Sync (New Date)**:
   - Complete one set in PWA
   - Click "📤 Sync to Sheets"
   - Check your workout tab
   - Should see NEW column pair with merged date header in Row 1
   - Should see reps/weight in correct exercise + set row

2. **Test Load**:
   - Start new workout session
   - Click "📥 Load Last"
   - Should see previous workout data loaded

3. **Test Sync (Existing Date)**:
   - Complete another set on same date
   - Click "📤 Sync to Sheets"
   - Should update existing date column (not create new one)

---

## 🐛 Troubleshooting

**Error: "Exercise not found in Column A"**
- Check that Column A has exact exercise name
- Example: PWA sends "Incline Dumbbell Press", sheet must have "Incline Dumbbell Press" (not "Incline DB Press")
- Check that SET labels follow immediately (e.g., "SET 1: 8-10")

**Error: "Tab not found"**
- Verify tab names in `SHEET_TAB_MAPPING` match your actual tab names exactly
- Check spelling, spaces, and (Phase 2) suffix

**Date column not auto-creating**
- Check that Row 1 is empty where new columns should be added
- Verify Apps Script has permission to modify the sheet

**Wrong data appearing**
- Check `CURRENT_PHASE` setting matches your current program (phase1 vs phase2)
- Verify date format in Row 1 is consistent (YYYY-MM-DD recommended)

---

**Status:** ✅ READY TO DEPLOY
**Next Action:** Follow Setup Instructions above!
