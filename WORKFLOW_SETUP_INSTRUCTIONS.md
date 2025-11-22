# Sand & Gravel PDF Extraction Workflow - Setup Instructions

## Overview

This modified workflow extracts contact information from PDF permits and updates your Google Sheet "Sand & Gravel Sites" with the extracted data, then creates Gmail drafts for outreach.

## Key Modifications from Original Workflow

### 1. **Enhanced PDF Extraction**
The Claude AI node now extracts:
- ✅ Legally Responsible Party Name
- ✅ Legally Responsible Party Email
- ✅ Permittee Name
- ✅ Permittee Email
- ✅ Site Contact Name
- ✅ Site Contact Email
- ✅ Permit Number (for matching existing rows)
- ✅ Facility Name
- ✅ NAICS Code(s) (from "Location of Sampling Locations" section)
- ✅ Type of Discharge (from "Location of Sampling Locations" section)
- ✅ Outfall Type (from "Location of Discharge into Surface Waterbody" section)

### 2. **Smart Row Matching**
The workflow now:
- **Matches by Permit Number** to find existing rows in your sheet
- **Updates only the extracted fields** (NAICS, Type of Discharge, Outfall Type, Legal Name, Legal Email, Permittee Name, Permittee Email, Site Contact Name, Site Contact Email)
- **Preserves existing data** in other columns (Facility Name, Address, City, Effective Date, Expiration Date)

### 3. **Updated Google Sheets Integration**
- Points to "Sand & Gravel Sites" sheet (instead of "PARIS Violations Tracker")
- Uses "appendOrUpdate" operation to update existing rows or add new ones if no match found
- Maps extracted data to correct columns matching your sheet structure

## Setup Instructions

### Step 1: Import into N8N

1. Open your N8N instance
2. Click **"Add Workflow"** or the **"+"** button
3. Click **"Import from File"** or **"Import from URL"**
4. Select the file: `SAND&GRAVEL_RENEWAL_MODIFIED.json`
5. Click **"Import"**

### Step 2: Configure Google Sheets Node

**IMPORTANT:** You need to update the Google Sheet ID to point to your "Sand & Gravel Sites" sheet.

1. Click on the **"Update Sand & Gravel Sites Sheet"** node
2. In the **"Document"** field:
   - Click the dropdown
   - Select **"Sand & Gravel Sites"** from your Google Sheets
   - OR manually enter your Google Sheet ID (found in the URL of your sheet)
3. In the **"Sheet"** field:
   - Select the correct sheet tab name
   - Default is usually "Sheet1" or the name of your tab

**To find your Google Sheet ID:**
- Open your "Sand & Gravel Sites" Google Sheet
- Look at the URL: `https://docs.google.com/spreadsheets/d/YOUR_SHEET_ID_HERE/edit`
- Copy the ID between `/d/` and `/edit`

### Step 3: Verify Credentials

Ensure these credentials are configured:

1. **Anthropic API** (for Claude AI extraction)
   - Node: "Claude AI Extract"
   - You can reuse your existing "PARISPDFCLAUDEAPI" credential

2. **Gmail OAuth2** (for creating drafts)
   - Node: "Create Gmail Draft"
   - You can reuse your existing "Gmail account BLUE" credential

3. **Google Sheets OAuth2** (for updating the sheet)
   - Node: "Update Sand & Gravel Sites Sheet"
   - You can reuse your existing "Google Sheets account" credential

### Step 4: Test the Workflow

1. Click **"Execute Workflow"** or **"Test Workflow"**
2. Upload a sample PDF using the webhook URL
3. Verify:
   - ✅ PDF text is extracted
   - ✅ Claude extracts all 8 fields correctly
   - ✅ Email draft is created in Gmail
   - ✅ Google Sheet row is updated (matched by Permit Number)

### Step 5: Activate the Workflow

1. Click the toggle switch to **"Active"**
2. The workflow will now process PDFs automatically when uploaded via webhook

## How the Workflow Works

```
1. PDF Upload (Webhook)
   ↓
2. Extract Text from PDF
   ↓
3. Claude AI Extraction
   - Legal Name
   - Legal Email
   - Permittee Name
   - Permittee Email
   - Site Contact Name
   - Site Contact Email
   - Permit Number
   - Facility Name
   - NAICS
   - Type of Discharge
   - Outfall Type
   ↓
4. Prepare Email & Data (JavaScript)
   - Formats extracted data
   - Builds email draft
   ↓
5. Two Parallel Actions:
   a) Create Gmail Draft → Sent to Permittee/Legal contact
   b) Update Google Sheet → Matches by Permit Number, updates row
```

## Important Notes

### Matching Logic
- The workflow **matches rows by Permit Number**
- If the Permit Number is found in your sheet, it **updates that row**
- If the Permit Number is NOT found, it **creates a new row**
- Only the extracted columns are updated; existing data is preserved

### Column Mapping

| PDF Field | PDF Section | Google Sheet Column |
|-----------|-------------|---------------------|
| NAICS | Location of Sampling Locations (Monitoring Point) | NAICS |
| Type of Discharge | Location of Sampling Locations (Monitoring Point) | Type of Discharge |
| Outfall Type | Location of Discharge into Surface Waterbody (Outfall Location) | Outfall Type |
| Permit Number | Main permit information | Permit Number (used for matching) |
| Legal Name | Legal Responsible Party section | Legal Name |
| Legal Email | Legal Responsible Party section | Legal Email |
| Permittee Name | Permittee section | Permittee Name |
| Permittee Email | Permittee section | Permittee Email |
| Site Contact Name | Site Contact section | Site Contact Name |
| Site Contact Email | Site Contact section | Site Contact Email |

### Email Draft Behavior
- **To:** Permittee Email (or Legal Email if Permittee Email not found)
- **CC:** Site Contact Email
- **Subject:** "[Facility Name] - Stormwater Permit Information"
- **Body:** Professional outreach email with your signature

## Troubleshooting

### Issue: Sheet rows not updating
- **Check:** Verify the Permit Number extracted from PDF matches exactly with the value in your sheet
- **Check:** Ensure "Permit Number" column exists and is spelled correctly in your sheet
- **Fix:** The matching is case-sensitive and must be exact

### Issue: Wrong sheet being updated
- **Check:** Verify you updated the Google Sheet ID in the "Update Sand & Gravel Sites Sheet" node
- **Fix:** Click the node, select the correct document from the dropdown

### Issue: Claude not extracting all fields
- **Check:** Review the PDF structure - field names and section headers may vary
- **Fix:** Update the Claude AI prompt to include alternative field names or section names
  - For NAICS: Look for "NAICS Code", "Industry Code", "SIC Code" as alternatives
  - For Type of Discharge: Look for "Discharge Type", "Outfall Type", or similar
  - For Outfall Type: Check "Outfall Location", "Discharge Point", or similar sections
  - For contact info: Try "Legal Responsible Person" vs "Legally Responsible Party"

### Issue: Duplicate rows being created
- **Check:** Ensure Permit Number is being extracted correctly
- **Check:** Verify the Permit Number column in your sheet has values
- **Fix:** The workflow needs a valid Permit Number to match; if PDFs don't have permit numbers, you may need to match by "Facility Name" instead

## Customization Options

### Change Matching Column
If you want to match by "Facility Name" instead of "Permit Number":

1. In the "Update Sand & Gravel Sites Sheet" node
2. Change `"matchingColumns": ["Permit Number"]` to `"matchingColumns": ["Facility Name"]`
3. Ensure you're also extracting and passing the Facility Name in the data

### Modify Email Template
1. Click on the "Prepare Email & Data" node
2. Edit the `message` variable in the JavaScript code
3. Customize the email text, subject, or formatting

### Add More Fields to Extract
1. Update the Claude AI prompt to include new fields
2. Add the fields to the JavaScript code in "Prepare Email & Data"
3. Add the columns to the Google Sheets mapping

## Support

If you encounter issues:
1. Check the execution log in N8N for error messages
2. Verify credentials are properly configured
3. Test each node individually to isolate the problem
4. Ensure your Google Sheet column names match exactly (case-sensitive)

---

**File:** `SAND&GRAVEL_RENEWAL_MODIFIED.json`
**Created:** 2025-11-22
**Purpose:** Extract PDF permit information and update Sand & Gravel Sites tracking sheet
