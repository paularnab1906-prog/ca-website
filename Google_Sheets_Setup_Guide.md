# Google Sheets Form Submission Setup Guide

This guide explains how to connect both your **Contact Enquiry Form** and **Careers Application Form** directly to a Google Sheet. Whenever someone fills out either form on your website, a new row will automatically be added to your spreadsheet.

---

## Step 1: Create Your Google Sheets

1. Go to [Google Sheets](https://sheets.google.com) and create a **New Blank Spreadsheet**.
2. Rename the spreadsheet (e.g., `Abinash & Co. Website Submissions`).
3. You will want two sheets (tabs) in this file—one for Enquiries and one for Careers:
   * Rename **Sheet1** to `Enquiries`.
   * Create a second sheet by clicking the `+` sign at the bottom-left, and rename it to `Careers`.
4. In the first row of each sheet, type the exact headers that the forms will send (headers are case-sensitive):
   * **In the `Enquiries` sheet, set row 1 to:**
     `Date` | `Name` | `Phone` | `Email` | `Message`
   * **In the `Careers` sheet, set row 1 to:**
     `Date` | `Name` | `Phone` | `Email` | `Position` | `Summary`

---

## Step 2: Add Google Apps Script

1. In your Google Sheet, click **Extensions** -> **Apps Script** in the top menu bar.
2. Delete any code that is currently in the editor.
3. Copy and paste the following script into the editor:

```javascript
// Google Apps Script to write HTML form submissions to Google Sheets

function doPost(e) {
  const lock = LockService.getScriptLock();
  lock.tryLock(10000); // Prevent concurrent write errors

  try {
    const activeSpreadsheet = SpreadsheetApp.getActiveSpreadsheet();
    
    // Determine which form submitted based on parameter fields
    let sheetName = 'Enquiries';
    if (e.parameter.Position) {
      sheetName = 'Careers';
    }
    
    const sheet = activeSpreadsheet.getSheetByName(sheetName);
    if (!sheet) {
      return ContentService
        .createTextOutput(JSON.stringify({ 'result': 'error', 'error': 'Sheet not found: ' + sheetName }))
        .setMimeType(ContentService.MimeType.JSON);
    }

    // Read headers from row 1
    const headers = sheet.getRange(1, 1, 1, sheet.getLastColumn()).getValues()[0];
    const nextRow = sheet.getLastRow() + 1;

    // Map parameters to header columns
    const newRow = headers.map(function(header) {
      if (header === 'Date') return new Date().toLocaleString();
      return e.parameter[header] || '';
    });

    // Write row
    sheet.getRange(nextRow, 1, 1, newRow.length).setValues([newRow]);

    return ContentService
      .createTextOutput(JSON.stringify({ 'result': 'success', 'row': nextRow }))
      .setMimeType(ContentService.MimeType.JSON);

  } catch (error) {
    return ContentService
      .createTextOutput(JSON.stringify({ 'result': 'error', 'error': error.toString() }))
      .setMimeType(ContentService.MimeType.JSON);
  } finally {
    lock.releaseLock();
  }
}
```

4. Click the **Save** icon (disk symbol) or press `Ctrl + S`.

---

## Step 3: Deploy the Script as a Web App

To make the script accessible to your website, you must deploy it as a public web app:

1. In the top right corner of the Apps Script page, click **Deploy** -> **New deployment**.
2. Click the gear icon next to "Select type" and choose **Web app**.
3. Fill out the configuration fields:
   * **Description:** `Website Form Submission Web App`
   * **Execute as:** `Me (your-email@gmail.com)`
   * **Who has access:** **`Anyone`** *(CRITICAL: Must select "Anyone" so visitors to your website can submit forms).*
4. Click **Deploy**.
5. Google will ask you to **Authorize Access**. Click *Authorize access*, choose your Google account, click *Advanced* (bottom left of dialog), and click *Go to Untitled project (unsafe)*, then click *Allow*.
6. Once deployed, copy the **Web App URL** shown in the dialog box (it will look like: `https://script.google.com/macros/s/AKfycb.../exec`).

---

## Step 4: Link the URL to Your HTML Files

Now, you just need to paste this URL into your website code:

### 1. In `Abinash & Co - 1 Classic.html` (Enquiry Form)
1. Open the [Abinash & Co - 1 Classic.html](file:///e:/CA%20Website/Abinash%20&%20Co%20-%201%20Classic.html) file in a text editor (e.g. VS Code, Notepad++).
2. Search for the placeholder string: `'YOUR_GOOGLE_SHEET_CONTACT_URL_HERE'`.
3. Replace that placeholder string (including the quotes) with your Web App URL.
   * *Example:* `const scriptUrl = 'https://script.google.com/macros/s/YOUR_ACTUAL_ID/exec';`
4. Save the file.

### 2. In `careers.html` (Careers Form)
1. Open [careers.html](file:///e:/CA%20Website/careers.html) in your text editor.
2. Search for the placeholder string: `'YOUR_GOOGLE_SHEET_CAREERS_URL_HERE'`.
3. Replace that placeholder with the exact same Web App URL.
4. Save the file.

---

## How to Test
1. Open your website locally or visit the hosted link.
2. Fill out the **Send Enquiry** form and submit. It should change the button to "Sending..." and then show the thank you screen. Check your Google Sheet under the **Enquiries** tab—the data will be recorded!
3. Go to the **Careers** page, fill out the application form, and submit. Check your Google Sheet under the **Careers** tab—the application will be recorded there too!
