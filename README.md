// =========================================================================
// GOOGLE APPS SCRIPT BACKEND FOR 12-WEEK GYM TRACKER
// Handles Dashboard Data Extraction (GET) and Mobile App Logging (POST)
// =========================================================================

const DAY_SHEETS = [
  "Day 1 - Lower A", 
  "Day 2 - Upper A", 
  "Day 3 - Condition", 
  "Day 4 - Zone 2", 
  "Day 5 - Lower B", 
  "Day 6 - Upper B"
];

// 1. GET REQUEST: Fetches all data from day sheets to power the web dashboard
function doGet(e) {
  var ss = SpreadsheetApp.getActiveSpreadsheet();
  var payload = { data: {}, muscleGroups: {} };
  var maxTrackedWeek = 1;
  
  DAY_SHEETS.forEach(function(sheetName) {
    var sheet = ss.getSheetByName(sheetName);
    if (!sheet) return;
    
    var values = sheet.getDataRange().getValues();
    payload.data[sheetName] = [];
    var currentWeekNum = null;
    
    // Parse structural rows to compile clean JSON
    for (var i = 0; i < values.length; i++) {
      var row = values[i];
      var firstCell = row[0] ? row[0].toString().trim() : "";
      
      // Track which week section we are currently reading
      if (firstCell.toUpperCase().indexOf("WEEK ") === 0) {
        currentWeekNum = parseInt(firstCell.replace(/[^0-9]/g, ''));
        if (currentWeekNum > maxTrackedWeek) maxTrackedWeek = currentWeekNum;
      }
      
      if (currentWeekNum && row[0] && i > 6) {
        var cleanHeader = firstCell.toUpperCase();
        if (cleanHeader.indexOf("WEEK") === -1 && cleanHeader.indexOf("SUMMARY") === -1 && firstCell !== "Exercise") {
          
          var hasData = (row[3] !== "" || row[7] !== "");
          if (hasData && currentWeekNum > maxTrackedWeek) {
            maxTrackedWeek = currentWeekNum;
          }

          payload.data[sheetName].push({
            week: currentWeekNum,
            name: row[0],
            category: row[1] || "Other",
            target: row[2] || "",
            sets: [
              { weight: row[3], reps: row[7], rpe: row[11] },
              { weight: row[4], reps: row[8], rpe: row[12] },
              { weight: row[5], reps: row[9], rpe: row[13] },
              { weight: row[6], reps: row[10], rpe: row[14] }
            ],
            est1RM: row[15] || 0,
            duration: row[16] || "",
            notes: row[17] || ""
          });
        } else if (cleanHeader.indexOf("SUMMARY") !== -1) {
          var sessionRpeVal = "";
          for (var c = 0; c < row.length; c++) {
            if (row[c] && row[c].toString().trim().toLowerCase() === "session rpe:") {
              sessionRpeVal = row[c+1];
              break;
            }
          }
          payload.data[sheetName].push({
            week: currentWeekNum,
            isSummary: true,
            sessionRPE: sessionRpeVal
          });
        }
      }
    }
  });
  
  payload.currentWeek = maxTrackedWeek;
  
  return ContentService.createTextOutput(JSON.stringify(payload))
    .setMimeType(ContentService.MimeType.JSON)
    .setHeaders({ 'Access-Control-Allow-Origin': '*' });
}

// 2. POST REQUEST: Takes log data submitted from your webpage and writes it to Sheets row coordinates
function doPost(e) {
  try {
    var params = JSON.parse(e.postData.contents);
    var sheetName = params.daySheet;
    var weekTarget = "WEEK " + params.week;
    
    var ss = SpreadsheetApp.getActiveSpreadsheet();
    var sheet = ss.getSheetByName(sheetName);
    if (!sheet) {
      return createCorsResponse({status: "error", message: "Sheet '" + sheetName + "' not found."});
    }
    
    var data = sheet.getDataRange().getValues();
    var startRow = -1;
    
    // Find the exact Week Block row coordinate
    for (var i = 0; i < data.length; i++) {
      if (data[i][0] && data[i][0].toString().trim().toUpperCase() === weekTarget.toUpperCase()) {
        startRow = i;
        break;
      }
    }
    
    if (startRow === -1) {
      return createCorsResponse({status: "error", message: "Block " + weekTarget + " not found in sheet."});
    }
    
    var updatedCount = 0;
    
    // Scan exercises below the matched Week row
    for (var i = startRow + 1; i < data.length; i++) {
      var rowHeader = data[i][0] ? data[i][0].toString().trim() : "";
      
      // Stop scanning if we accidentally cross into the next week block
      if (rowHeader.toUpperCase().indexOf("WEEK") === 0) break;
      
      // Match submitted payload exercises with row name
      for (var j = 0; j < params.workouts.length; j++) {
        var exData = params.workouts[j];
        if (rowHeader.toLowerCase() === exData.name.toLowerCase()) {
          var rowNum = i + 1;
          
          // Columns Map: D-G (Weights), H-K (Reps), L-O (RPEs)
          for (var s = 0; s < 4; s++) {
            if (exData.sets[s]) {
              if (exData.sets[s].weight !== undefined && exData.sets[s].weight !== "") {
                sheet.getCell(rowNum, 4 + s).setValue(Number(exData.sets[s].weight));
              }
              if (exData.sets[s].reps !== undefined && exData.sets[s].reps !== "") {
                sheet.getCell(rowNum, 8 + s).setValue(Number(exData.sets[s].reps));
              }
              if (exData.sets[s].rpe !== undefined && exData.sets[s].rpe !== "") {
                sheet.getCell(rowNum, 12 + s).setValue(Number(exData.sets[s].rpe));
              }
            }
          }
          if (exData.notes) sheet.getCell(rowNum, 18).setValue(exData.notes); // Col R
          updatedCount++;
        }
      }
      
      // Write general session metrics if it hits the Summary Row row header
      if (rowHeader.toLowerCase().indexOf("summary") !== -1 && params.sessionRPE) {
        for (var c = 0; c < data[i].length; c++) {
          if (data[i][c] && data[i][c].toString().trim().toLowerCase() === "session rpe:") {
            sheet.getCell(i + 1, c + 2).setValue(Number(params.sessionRPE));
            break;
          }
        }
      }
    }
    
    return createCorsResponse({status: "success", updatedExercises: updatedCount});
  } catch (err) {
    return createCorsResponse({status: "error", message: err.toString()});
  }
}

function createCorsResponse(obj) {
  return ContentService.createTextOutput(JSON.stringify(obj))
    .setMimeType(ContentService.MimeType.JSON)
    .setHeaders({
      'Access-Control-Allow-Origin': '*',
      'Access-Control-Allow-Methods': 'GET, POST, OPTIONS',
      'Access-Control-Allow-Headers': 'Content-Type'
    });
}
function doOptions(e) {
  return createCorsResponse({status: "cors_ok"});
