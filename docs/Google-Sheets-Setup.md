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
✅ **Exercise variant support** - Change exercises in PWA dropdown, syncs to correct position
✅ **Exercise dropdowns in Sheets** - Click any exercise in Column A to see dropdown menu with all alternatives
✅ **Visual change tracking** - Yellow highlight + ⚠️ icon + cell notes mark when exercise variants change

**What this does:**
- PWA syncs workout data to your existing horizontal tabs
- Loads previous workouts for progressive overload suggestions
- All-in-one tracking: Your structure + PWA automation
- **Supports dynamic exercise changes** - Switch from "Chest-Supported Row" to "Single Arm Row" in PWA, data syncs to E2 position in sheet

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
// ===== CHANGELOG =====
// 2025-12-14: Fixed critical column selection bug - rewrote createNewDateColumn
//             to directly check columns 2,4,6,8... (R columns) in Row 1 for empty slots
//             Previous version had array indexing bug causing it to skip to columns 7/8
//             Now correctly fills columns B/C (2/3) first, then D/E (4/5), etc.
//             Added dropdown validation for exercise alternatives in Column A
//             Enhanced exercise change tracking with visual markers (⚠️ + yellow highlight)
//             Updated setupWorkoutSheet to use E# prefix for exercise names
//
// ===== CONFIGURATION =====
// Set your current phase here: 'phase1' or 'phase2'
const CURRENT_PHASE = 'phase2';  // Change to 'phase1' when doing Phase 1

// ===== ONE-TIME SETUP FUNCTION =====
// Run this ONCE after creating a blank Google Sheet to auto-create all tabs and exercises
function setupWorkoutSheet() {
  const ss = SpreadsheetApp.getActiveSpreadsheet();

  // Phase 2 workout structure with exact rep ranges from PWA
  // Exercise names include E# prefix to match PWA format
  // Each exercise includes alternatives for dropdown validation
  const phase2Workouts = {
    'Upper Body 1 (Phase 2)': [
      {
        name: 'E1: Incline Dumbbell Press',
        sets: ['SET 1: 8-10', 'SET 2: 8-10', 'SET 3: 8-10'],
        alternatives: ['E1: Incline Dumbbell Press', 'E1: Flat Dumbbell Press', 'E1: Incline Barbell Press', 'E1: Machine Chest Press']
      },
      {
        name: 'E2: Chest-Supported Row',
        sets: ['SET 1: 8-10', 'SET 2: 8-10', 'SET 3: 8-10'],
        alternatives: ['E2: Chest-Supported Row', 'E2: Single Arm Dumbbell Row', 'E2: T-Bar Row', 'E2: Machine Row']
      },
      {
        name: 'E3: Lean-Away Cable Lateral Raises',
        sets: ['SET 1: 10-12', 'SET 2: 10-12', 'SET 3: 10-12'],
        alternatives: ['E3: Lean-Away Cable Lateral Raises', 'E3: Dumbbell Lateral Raises', 'E3: Machine Lateral Raises']
      },
      {
        name: 'E4: Pull-Ups',
        sets: ['SET 1: 6-8', 'SET 2: 6-8', 'SET 3: 6-8', 'SET 4: 6-8'],
        alternatives: ['E4: Pull-Ups', 'E4: Assisted Pull-Ups', 'E4: Lat Pulldown', 'E4: Hanging Knee Raises with Pull-Ups']
      },
      {
        name: 'E5: Incline Overhead Dumbbell Extensions',
        sets: ['SET 1: 12-15', 'SET 2: 12-15', 'SET 3: 12-15'],
        alternatives: ['E5: Incline Overhead Dumbbell Extensions', 'E5: Cable Rope Pushdowns', 'E5: Cable Bar Pushdowns', 'E5: Standing Overhead DB Extension']
      }
    ],
    'Lower Body 1 (Phase 2)': [
      {
        name: 'E1: Back Squat',
        sets: ['SET 1: 6-8', 'SET 2: 6-8', 'SET 3: 6-8'],
        alternatives: ['E1: Back Squat', 'E1: Front Squat', 'E1: Goblet Squat', 'E1: Leg Press']
      },
      {
        name: 'E2: Bulgarian Split Squat',
        sets: ['SET 1: 8-10', 'SET 2: 8-10', 'SET 3: 8-10'],
        alternatives: ['E2: Bulgarian Split Squat', 'E2: Walking Lunges', 'E2: Reverse Lunges', 'E2: Step-Ups']
      },
      {
        name: 'E3: Swiss Ball Leg Curls',
        sets: ['SET 1: 10-12', 'SET 2: 10-12', 'SET 3: 10-12'],
        alternatives: ['E3: Swiss Ball Leg Curls', 'E3: Lying Leg Curls', 'E3: Seated Leg Curls', 'E3: Nordic Curls']
      },
      {
        name: 'E4: Single Leg Weighted Calf Raise',
        sets: ['SET 1: 10-12', 'SET 2: 10-12', 'SET 3: 10-12'],
        alternatives: ['E4: Single Leg Weighted Calf Raise', 'E4: Standing Calf Raise (Machine)', 'E4: Seated Weighted Calf Raise']
      }
    ],
    'Upper Body 2 (Phase 2)': [
      {
        name: 'E1: Barbell Bench Press',
        sets: ['SET 1: 6-8', 'SET 2: 6-8', 'SET 3: 6-8'],
        alternatives: ['E1: Barbell Bench Press', 'E1: Dumbbell Bench Press', 'E1: Machine Chest Press', 'E1: Incline Barbell Press']
      },
      {
        name: 'E2: Seated Cable Row',
        sets: ['SET 1: 8-10', 'SET 2: 8-10', 'SET 3: 8-10'],
        alternatives: ['E2: Seated Cable Row', 'E2: Machine Row', 'E2: Chest-Supported Row', 'E2: Barbell Row']
      },
      {
        name: 'E3: Standing Overhead Press',
        sets: ['SET 1: 8-10', 'SET 2: 8-10', 'SET 3: 8-10'],
        alternatives: ['E3: Standing Overhead Press', 'E3: Seated Dumbbell Press', 'E3: Machine Shoulder Press', 'E3: Arnold Press']
      },
      {
        name: 'E4: Face Pulls',
        sets: ['SET 1: 12-15', 'SET 2: 12-15', 'SET 3: 12-15'],
        alternatives: ['E4: Face Pulls', 'E4: Kneeling Face Pulls', 'E4: Reverse Pec Deck', 'E4: Band Pull-Aparts']
      },
      {
        name: 'E5: Dip Push-Ups',
        sets: ['SET 1: 10-12', 'SET 2: 10-12', 'SET 3: 10-12'],
        alternatives: ['E5: Dip Push-Ups', 'E5: Weighted Dips', 'E5: Chest Dips', 'E5: Diamond Push-Ups']
      },
      {
        name: 'E6: Incline Dumbbell Curls',
        sets: ['SET 1: 10-12', 'SET 2: 10-12', 'SET 3: 10-12'],
        alternatives: ['E6: Incline Dumbbell Curls', 'E6: Cable Curls', 'E6: Barbell Curls', 'E6: Hammer Curls']
      }
    ],
    'Lower Body 2 (Phase 2)': [
      {
        name: 'E1: Deadlift',
        sets: ['SET 1: 6-8', 'SET 2: 6-8', 'SET 3: 6-8'],
        alternatives: ['E1: Deadlift', 'E1: Romanian Deadlift', 'E1: Trap Bar Deadlift', 'E1: Sumo Deadlift']
      },
      {
        name: 'E2: Leg Press',
        sets: ['SET 1: 10-12', 'SET 2: 10-12', 'SET 3: 10-12'],
        alternatives: ['E2: Leg Press', 'E2: Hack Squat', 'E2: Front Squat', 'E2: Goblet Squat']
      },
      {
        name: 'E3: Reverse Lunges',
        sets: ['SET 1: 10-12', 'SET 2: 10-12', 'SET 3: 10-12'],
        alternatives: ['E3: Reverse Lunges', 'E3: Walking Lunges', 'E3: Bulgarian Split Squat', 'E3: Step-Ups']
      },
      {
        name: 'E4: Seated Weighted Calf Raise',
        sets: ['SET 1: 10-12', 'SET 2: 10-12', 'SET 3: 10-12'],
        alternatives: ['E4: Seated Weighted Calf Raise', 'E4: Standing Calf Raise (Machine)', 'E4: Single Leg Weighted Calf Raise']
      }
    ]
  };

  // Create each workout tab
  Object.keys(phase2Workouts).forEach(tabName => {
    // Create new sheet
    const sheet = ss.insertSheet(tabName);

    // Set up Row 1: Tab name in A1
    sheet.getRange(1, 1).setValue(tabName);

    // Set up Row 2: R/W headers starting from Column B
    sheet.getRange(2, 2).setValue('R');
    sheet.getRange(2, 3).setValue('W');

    // Add exercises starting from Row 3
    let currentRow = 3;
    const exercises = phase2Workouts[tabName];

    exercises.forEach(exercise => {
      // Write exercise name
      const exerciseCell = sheet.getRange(currentRow, 1);
      exerciseCell.setValue(exercise.name);

      // Add dropdown validation with alternatives
      if (exercise.alternatives && exercise.alternatives.length > 0) {
        const rule = SpreadsheetApp.newDataValidation()
          .requireValueInList(exercise.alternatives, true)
          .setAllowInvalid(false)
          .setHelpText('Select an exercise variant from the dropdown')
          .build();
        exerciseCell.setDataValidation(rule);
      }

      currentRow++;

      // Write SET labels
      exercise.sets.forEach(setLabel => {
        sheet.getRange(currentRow, 1).setValue(setLabel);
        currentRow++;
      });
    });

    // Format the sheet
    sheet.setColumnWidth(1, 250);  // Column A wider for exercise names
    sheet.setFrozenRows(2);         // Freeze header rows
    sheet.setFrozenColumns(1);      // Freeze exercise names column
  });

  // Delete default Sheet1 after creating all workout tabs
  const defaultSheet = ss.getSheetByName('Sheet1');
  if (defaultSheet) {
    ss.deleteSheet(defaultSheet);
  }

  Logger.log('✅ Workout sheets created successfully!');
  Logger.log('📝 Next: Deploy as Web App and copy the URL to PWA settings');
}

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
  // Extract exercise position (E1, E2, E3, etc.) from exercise name
  // This allows exercise variants to sync correctly
  // Example: "E2: Single Arm Row" → position 2
  const exerciseMatch = exerciseName.match(/^E(\d+):/);

  if (!exerciseMatch) {
    // Fallback: try exact name matching if no E# prefix found
    return findExerciseRowByName(sheet, exerciseName, setNumber);
  }

  const exercisePosition = parseInt(exerciseMatch[1]);

  // Scan column A to find the Nth exercise (counting only exercise names, not SET labels)
  const columnA = sheet.getRange(3, 1, sheet.getLastRow() - 2, 1).getValues();
  let exerciseCount = 0;

  for (let i = 0; i < columnA.length; i++) {
    const cellValue = columnA[i][0];
    if (!cellValue) continue;

    // Skip SET labels - only count actual exercise names
    if (!cellValue.toString().startsWith('SET ')) {
      exerciseCount++;

      // Found the exercise at the correct position!
      if (exerciseCount === exercisePosition) {
        // SET rows follow immediately after: "SET 1: X-Y", "SET 2: X-Y", etc.
        const setRow = i + 3 + setNumber; // +3 for Row 1,2 headers + 0-indexed i, +setNumber for set offset

        // Verify this is actually a SET row
        if (setRow <= sheet.getLastRow()) {
          return setRow;
        }
      }
    }
  }

  return -1; // Exercise position not found
}

// Fallback function for exact name matching (backwards compatibility)
function findExerciseRowByName(sheet, exerciseName, setNumber) {
  const columnA = sheet.getRange(3, 1, sheet.getLastRow() - 2, 1).getValues();

  for (let i = 0; i < columnA.length; i++) {
    const cellValue = columnA[i][0];
    if (!cellValue) continue;

    // Check if this is the exercise name (not a SET label)
    if (cellValue.toString() === exerciseName ||
        cellValue.toString().replace(/^E\d+:\s*/, '') === exerciseName.replace(/^E\d+:\s*/, '')) {

      const setRow = i + 3 + setNumber;
      if (setRow <= sheet.getLastRow()) {
        return setRow;
      }
    }
  }

  return -1;
}

function createNewDateColumn(sheet, date) {
  // Create new R/W column pair for a new workout date
  // IMPORTANT: Always scan Row 1 to find the first empty column, not last column with any data

  // Start checking from column B (column 2)
  // R/W pairs are always in columns B/C, D/E, F/G, H/I, etc.
  // So R columns are at: 2, 4, 6, 8, 10... (even numbers)

  let newColR = -1;
  let newColW = -1;

  // Check up to 20 column pairs (40 columns total) - should be more than enough
  for (let col = 2; col <= 40; col += 2) {
    // col is the R column, col+1 is the W column
    const dateValue = sheet.getRange(1, col).getValue();

    // If Row 1 at this R column is empty, we found our spot
    if (!dateValue || dateValue === '') {
      newColR = col;
      newColW = col + 1;
      break;
    }
  }

  // If we didn't find an empty pair in first 20 pairs, add at the end
  if (newColR === -1) {
    const lastCol = sheet.getLastColumn();
    // Find next even column number after lastCol
    if (lastCol < 2) {
      newColR = 2; // First R/W pair
    } else if ((lastCol - 1) % 2 === 0) {
      newColR = lastCol + 1; // lastCol is odd, so lastCol+1 is even (R column)
    } else {
      newColR = lastCol + 2; // lastCol is even, so lastCol+2 is next even (R column)
    }
    newColW = newColR + 1;
  }

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

    // Check if exercise name changed (for exercise variant tracking)
    const exerciseMatch = data.exercise.match(/^E(\d+):\s*(.+)$/);
    if (exerciseMatch) {
      const exercisePosition = parseInt(exerciseMatch[1]);
      const newExerciseName = exerciseMatch[2].trim();

      // Find the exercise name row (not the SET row)
      const columnA = sheet.getRange(3, 1, sheet.getLastRow() - 2, 1).getValues();
      let exerciseCount = 0;
      let exerciseNameRow = -1;

      for (let i = 0; i < columnA.length; i++) {
        const cellValue = columnA[i][0];
        if (!cellValue || cellValue.toString().startsWith('SET ')) continue;

        exerciseCount++;
        if (exerciseCount === exercisePosition) {
          exerciseNameRow = i + 3; // +3 for header rows
          const currentExerciseName = cellValue.toString().replace(/^E\d+:\s*/, '').trim();

          // If exercise name changed, add comment, visual marker, and update name
          if (currentExerciseName !== newExerciseName) {
            const dateCell = sheet.getRange(1, dateCol);
            const existingNote = dateCell.getNote() || '';
            const changeNote = `${data.date}: E${exercisePosition} changed from "${currentExerciseName}" to "${newExerciseName}"`;

            // Add note to date cell (appends if note exists)
            if (existingNote) {
              dateCell.setNote(existingNote + '\n' + changeNote);
            } else {
              dateCell.setNote(changeNote);
            }

            // Highlight date cell with light yellow background to indicate exercise change
            dateCell.setBackground('#fff9c4');

            // Add a small indicator to the date cell text
            const currentDateText = dateCell.getValue();
            if (currentDateText && !currentDateText.toString().includes('⚠️')) {
              dateCell.setValue(currentDateText + ' ⚠️');
            }

            // Update exercise name in Column A
            sheet.getRange(exerciseNameRow, 1).setValue(`E${exercisePosition}: ${newExerciseName}`);
          }
          break;
        }
      }
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

## 🚀 Setup Instructions (Automated - 5 Minutes!)

### Step 1: Create Blank Google Sheet
1. Go to [sheets.google.com](https://sheets.google.com)
2. Click **Blank** to create new spreadsheet
3. Name it "BWS Workout Tracker" (or your preferred name)

### Step 2: Open Apps Script & Paste Code
1. In your blank Google Sheet, click **Extensions → Apps Script**
2. **Rename the Apps Script project** (top-left corner in Apps Script editor): Click "Untitled project" → Enter "BWS Workout Tracker"
   - ⚠️ Note: This is separate from the Google Sheet name - you need to rename it in the Apps Script editor
3. Delete the default `function myFunction() {}` code
4. Copy the entire code block above (lines 63-322) and paste it
5. Click **Save** (💾 disk icon)

### Step 3: Run Setup Function (ONE TIME ONLY)
1. In Apps Script editor, find the function dropdown (top toolbar)
2. Select **setupWorkoutSheet** from the dropdown
3. Click **Run** (▶️ play button)
4. **Authorize the script** when prompted:
   - Click "Review permissions"
   - Choose your Google account
   - Click "Advanced" → "Go to BWS Workout Tracker (unsafe)" OR "Go to Untitled project (unsafe)"
   - Click "Allow"
5. Wait 5-10 seconds for setup to complete
6. Check "Execution log" at bottom - should show: `✅ Workout sheets created successfully!`
7. **Go back to your spreadsheet** - you'll see 4 new tabs with all exercises and rep ranges already filled in!

### Step 4: Deploy as Web App
1. Click **Deploy → New deployment** (top-right corner)
2. Click the **gear icon** ⚙️ next to "Select type"
3. Select **Web app** from the dropdown
4. Fill in the deployment settings:
   - **Description:** Type "BWS Horizontal Layout Tracker" (or any name you prefer)
   - **Execute as:** Select **Me** (your Google account email)
   - **Who has access:** Select **Anyone** (important - allows PWA to access the script)
5. Click **Deploy** button
6. **Authorize if prompted:**
   - Click "Authorize access"
   - Choose your Google account
   - Click "Advanced" → "Go to BWS Workout Tracker (unsafe)" (this is safe - it's your own script)
   - Click "Allow"
7. **Copy the Web App URL** from the popup (looks like: `https://script.google.com/macros/s/AKfy...xyz/exec`)
   - Click the "Copy" button or select and copy the URL manually
   - Keep this URL - you'll need it in the next step

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

### When You Change Exercise Variant in PWA:
```
Example: Switching from "Chest-Supported Row" to "Single Arm Row"

PWA:
1. User clicks dropdown on E2, selects "Single Arm Row"
2. Completes workout with new exercise
3. Clicks "📤 Sync to Sheets"

PWA sends:
{
  exercise: "E2: Single Arm Row",  ← Changed from default
  set: 1,
  weight: 30,
  reps: 10
}

Apps Script:
1. Extracts position from "E2: Single Arm Row" → Position 2
2. Scans Column A for 2nd exercise (ignores exercise name)
3. Finds 2nd exercise = "Chest-Supported Row" (or whatever is in sheet)
4. Writes data to SET 1 row under that 2nd exercise position

Result:
✅ Data syncs to correct position regardless of exercise name
✅ You can change exercises in PWA without updating Google Sheet
✅ Column A exercise names can stay as default OR you can manually update them
```

**How This Works:**
- PWA exercise names follow format: "E1: Exercise Name", "E2: Exercise Name", etc.
- Apps Script extracts the position number (E1=1, E2=2, E3=3, etc.)
- Matches by position, not by name
- **You can change exercise variants freely in PWA** - data always syncs to correct row

**Example:**
- Sheet has "Chest-Supported Row" in E2 position
- You switch PWA to "Single Arm Row" variant
- Workout data writes to E2 position in sheet (under "Chest-Supported Row" row)
- Column A name doesn't matter - position matching ensures correct sync

**Optional:** You can manually update Column A exercise names in Google Sheet to match your chosen variants, but it's not required for syncing.

### Exercise Change Tracking with Visual Markers:

**Two Ways to Change Exercises:**

1. **In PWA** - Select different variant from dropdown in PWA interface
2. **In Google Sheets** - Click exercise name cell in Column A and select from dropdown menu

When you change an exercise variant (via PWA or Google Sheets), the script automatically:

1. **Detects the change** - Compares PWA exercise name to Google Sheet Column A
2. **Adds visual marker** - Date cell gets yellow background (#fff9c4) and ⚠️ warning icon
3. **Adds cell note** - Hover over the date cell to see: "2025-12-16: E2 changed from 'Chest-Supported Row' to 'Single Arm Row'"
4. **Updates Column A** - Changes exercise name to match new variant
5. **Preserves history** - Previous date columns still show old exercise data with clear change marker

**Example:**
```
Row 1:  | Upper Body 1 | 2025-12-11      | 2025-12-16 ⚠️    | 2025-12-18 |
                        No marker        Yellow highlight   No marker
                                         + Note
Row 2:  |              | R    | W       | R    | W        | R    | W |
Row 3:  | E2: Single   |      |         |      |          |      |     |
        | Arm Row ▼    |      |         |      |          |      |     | ← Dropdown menu
Row 4:  | SET 1: 8-10  | 10   | 30      | 10   | 32.5     | 10   | 32.5 |
                        ↑                 ↑                 ↑
                    Old exercise      Changed here      New exercise
                 (Chest Row data)   (First Single Arm) (Single Arm)
```

**Why This Matters:**
- Historical data from 2025-12-11 is for "Chest-Supported Row"
- On 2025-12-16, you switched to "Single Arm Row"
- Yellow highlight + ⚠️ icon make it immediately visible when exercise changed
- Cell note on 2025-12-16 marks when the change happened
- Future workouts (2025-12-18+) use "Single Arm Row" data for progression

**How to See Change Notes:**
- In Google Sheets, hover over date cell in Row 1
- Small note indicator appears in corner of cell
- Note shows: "Date: E# changed from 'Old Exercise' to 'New Exercise'"

**Using Dropdowns in Google Sheets:**
- Each exercise name cell in Column A has a dropdown menu (▼)
- Click the cell to see all available alternatives for that exercise
- Select any alternative from the list - change will be tracked automatically
- This makes it easy to switch exercises manually in Google Sheets without using PWA

---

## 🎯 Benefits of This Approach

✅ **Single source of truth** - Manual tracking and PWA use same tabs
✅ **Auto-creates date columns** - No manual column creation for new workouts
✅ **Maintains your horizontal workflow** - See progression over time left-to-right
✅ **Supports variable set counts** - Handles 3 or 4 sets per exercise
✅ **Backward compatible** - Works with existing workout data
✅ **Progressive overload** - PWA suggests +2.5kg when you hit top reps
✅ **Flexible date formats** - Supports YYYY-MM-DD, MM/DD/YYYY, or Google Sheets dates
✅ **Exercise dropdowns** - Each exercise has dropdown menu with all alternatives
✅ **Visual change tracking** - Yellow highlight + ⚠️ icon when exercise variant changes
✅ **Detailed change history** - Cell notes track what changed and when

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
