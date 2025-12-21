# How to Connect Your Contact Form to Google Sheets

Follow these steps to save your website's contact form submissions directly into a Google Sheet.

## Step 1: Create the Google Sheet
1. Go to [Google Sheets](https://docs.google.com/spreadsheets/) and create a **Blank spreadsheet**.
2. Name it something like "Website Contact Form Responses".
3. In the first row, create the following headers exactly:
   - **Column A:** `timestamp`
   - **Column B:** `name`
   - **Column C:** `email`
   - **Column D:** `phone`
   - **Column E:** `course`
   - **Column F:** `message`

## Step 2: Create the Google Apps Script
1. In your Google Sheet, click on **Extensions** > **Apps Script**.
2. Delete any code currently in the editor (like `function myFunction() {...}`).
3. Copy and paste the following code exactly:

```javascript
var sheetName = 'Sheet1';
var scriptProp = PropertiesService.getScriptProperties();

function intialSetup() {
  var activeSpreadsheet = SpreadsheetApp.getActiveSpreadsheet();
  scriptProp.setProperty('key', activeSpreadsheet.getId());
}

function doPost(e) {
  var lock = LockService.getScriptLock();
  lock.tryLock(10000);

  try {
    var doc = SpreadsheetApp.openById(scriptProp.getProperty('key'));
    var sheet = doc.getSheetByName(sheetName);

    var headers = sheet.getRange(1, 1, 1, sheet.getLastColumn()).getValues()[0];
    var nextRow = sheet.getLastRow() + 1;

    var newRow = headers.map(function(header) {
      if(header === 'timestamp') {
        return new Date();
      } else if(header === 'course') {
         // Handle Select logic if needed or just pass value
         return e.parameter[header];
      } else {
         return e.parameter[header];
      }
    });

    sheet.getRange(nextRow, 1, 1, newRow.length).setValues([newRow]);

    return ContentService
      .createTextOutput(JSON.stringify({ 'result': 'success', 'row': nextRow }))
      .setMimeType(ContentService.MimeType.JSON);
  }

  catch (e) {
    return ContentService
      .createTextOutput(JSON.stringify({ 'result': 'error', 'error': e }))
      .setMimeType(ContentService.MimeType.JSON);
  }

  finally {
    lock.releaseLock();
  }
}
```

4. Click the **Save** icon (floppy disk).

## Step 3: Run Setup
1. In the toolbar, ensure `intialSetup` is selected in the function dropdown.
2. Click **Run**.
3. A "Review Permissions" window will pop up. Click **Review Permissions**.
4. Choose your Google Account.
5. You might see a warning "Google hasn't verified this app". Click **Advanced** > **Go to (Script Name) (unsafe)**.
6. Click **Allow**.

## Step 4: Deploy as Web App
1. Click on the blue **Deploy** button (top right) > **New deployment**.
2. Click the specific **Select type** gear icon > **Web app**.
3. **Description:** "Contact Form".
4. **Execute as:** `Me (your email)`.
5. **Who has access:** `Anyone` (IMPORTANT: It must be "Anyone", or the form won't work).
6. Click **Deploy**.
7. Copy the **Web app URL** that is generated. It looks like `https://script.google.com/macros/s/.../exec`.

## Step 5: Connect to Your Website
1. Open the file `js/script.js` in your project.
2. Find the line:
   ```javascript
   const GOOGLE_SCRIPT_URL = "YOUR_GOOGLE_SCRIPT_URL_HERE";
   ```
3. Replace `"YOUR_GOOGLE_SCRIPT_URL_HERE"` with the URL you copied in Step 4.
   - Example:
   ```javascript
   const GOOGLE_SCRIPT_URL = "https://script.google.com/macros/s/AKfycbz_rXy7.../exec";
   ```
4. Save the file.

**Done!** Your form submissions should now appear in your Google Sheet.
