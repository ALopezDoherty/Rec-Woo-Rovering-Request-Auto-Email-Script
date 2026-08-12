```markdown
```
---
# Auto Email Materials – Recreation Worcester Rover Dispatch

This Google Apps Script listens for a Google Form submission and automatically sends a formatted supply request email to a designated Rover recipient. It was originally built for the City of Worcester Recreation program but is intentionally shared without access to the original form, spreadsheet, or recipient list.

> **Important:**  
> This repository does **not** include access to any live Google Form, Google Sheet, or recipient email address. To use this script, you must create your own Google Form and Apps Script project using the instructions below.

---

## How It Works

1. A staff member submits a Google Form indicating the park site and the curriculum week/activity.
2. The script parses the form submission data (`e.namedValues`).
3. It identifies the selected activity and looks up required supplies from a built-in inventory catalog.
4. It composes a formatted HTML email and sends it to the configured Rover recipient(s).

The script does **not** write to a spreadsheet or create any records. It only sends email notifications.

---

## Requirements

- A Google account with access to Google Forms and Google Apps Script.
- Permission to send email from that account.
- A Google Form that matches the expected question titles described below.

---

## Setup

### 1. Create the Google Form

Create a new Google Form with the following question titles **exactly** as written. These titles are used by the script to read form responses.

| Question Title | Type | Notes |
|----------------|------|-------|
| `Select Park` | Short answer or Dropdown | Name of the park/site. |
| `Week` | Multiple choice or Dropdown | Must contain one of these values: `Weekly`, `Recurring`, `Other`, `Week 1`, `Week 2`, `Week 3`, `Week 4`, `Week 5`, `Week 6`. |
| `Select Weekly Activity` | Short answer / Paragraph / Checkbox | Used when `Week` contains `Weekly` or `Recurring`. |
| `General Supplies / Other` | Short answer / Paragraph / Checkbox | Used when `Week` is exactly `Other`. |
| `Select Week 1 Activity` | Short answer / Paragraph / Checkbox | Used when `Week` is `Week 1`. |
| `Select Week 2 Activity` | Short answer / Paragraph / Checkbox | Used when `Week` is `Week 2`. |
| `Select Week 3 Activity` | Short answer / Paragraph / Checkbox | Used when `Week` is `Week 3`. |
| `Select Week 4 Activity` | Short answer / Paragraph / Checkbox | Used when `Week` is `Week 4`. |
| `Select Week 5 Activity` | Short answer / Paragraph / Checkbox | Used when `Week` is `Week 5`. |
| `Select Week 6 Activity` | Short answer / Paragraph / Checkbox | Used when `Week` is `Week 6`. |

**Recommended form structure:**

- Section 1: `Select Park` and `Week`.
- Use Google Forms **“Go to section based on answer”** logic to send respondents to the correct activity section based on the `Week` value.
- In each activity section, include only the relevant activity question.
- In the activity question help text, list the exact activity names available for that week so respondents can copy/paste them.

**Important about activity field type:**

The script expects the activity field value to be a **string** containing one or more activity names separated by commas, for example:

```
Hero Emblem Banner, Superhero Symbol, Comic Creators
```

You can achieve this by using:

- A **Short answer** or **Paragraph** question with clear instructions to enter activity names exactly as shown, separated by commas.
- A **Checkbox** question **only if you modify the script** to join all selected values (see Customization below).

If you use a checkbox question without modifying the script, the script may only process the first selected activity.

---

### 2. Add the Script

1. Open your Google Form.
2. Click the **three-dot menu** in the top-right and select **Script editor**.
3. Delete any placeholder code and paste the full script.
4. Locate the configuration line at the top:
   ```javascript
   var ROVER_RECIPIENTS = "exampleEmail@WorcesterMA.gov";
   ```
5. Replace the email address with the actual recipient address.  
   To send to multiple recipients, separate email addresses with commas:
   ```javascript
   var ROVER_RECIPIENTS = "rover1@example.gov,rover2@example.gov";
   ```

---

### 3. Install the Trigger

The script must be run as an **installable trigger** because it sends email and requires authorization.

1. In the Apps Script editor, click the **clock icon** on the left (Triggers).
2. Click **Add Trigger**.
3. Configure:
   - **Choose function:** `autoEmailMaterials`
   - **Choose deployment:** `Head`
   - **Event source:** `From form`
   - **Event type:** `On form submit`
4. Click **Save**.
5. Authorize the script when prompted.

> **Note:** Do not rely on a simple `onFormSubmit` trigger. Simple triggers cannot access services that require authorization, such as `MailApp.sendEmail`.

---

### 4. Test

1. Submit a test response through your Google Form.
2. Check the recipient inbox for an email with the subject:  
   `NEW SUPPLY REQUEST: [Park Name] [Week Value]`
3. Open the Apps Script editor and view **Executions** or **Logs** to debug if no email arrives.

---

## Inventory Catalog

The script contains a built-in inventory catalog in the `rawCatalog` object. It maps exact activity names to the materials needed for that activity.

```javascript
var rawCatalog = {
  "Hero Emblem Banner": "Construction paper, Pre-cut shield/star/circle templates, Markers/crayons, ...",
  "Superhero Symbol": "Felt, scissors, glue, safety pins",
  ...
};
```

### Updating the Catalog

- Add or edit entries inside `rawCatalog` to reflect current activities and supplies.
- The activity name **key** must exactly match the activity name submitted in the form.
- Matching is **case-insensitive**, so `"hero emblem banner"` will still match `"Hero Emblem Banner"`.
- If an activity is submitted but not found in the catalog, the email will display:  
  `No specialized items listed in system.`

---

## Customization

### Change Email Recipient

Modify this line at the top of the script:

```javascript
var ROVER_RECIPIENTS = "yourEmail@example.gov";
```

### Support Checkbox Activity Fields

If your activity questions are **checkbox** questions and you want to process all selected activities, replace every occurrence of `[0]` when assigning `rawActivities` with `.join(", ")`.

For example, change:

```javascript
rawActivities = values['Select Weekly Activity'] ? values['Select Weekly Activity'][0] : "";
```

to:

```javascript
rawActivities = values['Select Weekly Activity'] ? values['Select Weekly Activity'].join(", ") : "";
```

Apply the same change to the `General Supplies / Other` and dynamic `Week X Activity` assignments.

### Add More Weeks

1. Add the new week option to the `Week` question in your form.
2. Add the corresponding activity question, e.g., `Select Week 7 Activity`.
3. Add activity entries to the `rawCatalog` object in the script.

### Change Email Branding

The HTML email template is built in the `htmlBody` variable. You can edit the colors, text, and layout there.

---

## Troubleshooting

| Issue | Likely Cause | Solution |
|-------|--------------|----------|
| No email sent | Trigger not installed or not authorized | Install an **installable form submit trigger** and authorize the script. |
| Email sent to wrong address | `ROVER_RECIPIENTS` not updated | Update the recipient email in the script. |
| Log says `CRITICAL ERROR: Could not find 'Select Park' or 'Week'.` | Form question titles do not match exactly | Rename form questions to exactly `Select Park` and `Week`. |
| Log says `Exited safely: No matching activity selection found` | The activity field for the selected week was empty or the week value did not match expected patterns | Check that the activity question for the selected week is filled out and that the `Week` value is one of: `Weekly`, `Recurring`, `Other`, `Week 1`–`Week 6`. |
| Email shows `No specialized items listed in system.` | Activity name submitted does not exist in `rawCatalog` | Add the exact activity name to the catalog or correct the form response. |

To view logs:

1. Open the Apps Script editor.
2. Click **Executions** in the left menu.
3. Select an execution to see logs and error messages.

---

## Limitations

- The script only sends email; it does not create or update any spreadsheet.
- The recipient list is hardcoded. To make it dynamic, you would need to modify the script.
- The activity field should contain a comma-separated string of activity names. Checkbox fields require a small script modification to work correctly with multiple selections.
- The script currently matches activity names by searching the submitted response for known catalog keys. This works well for comma-separated lists but may fail if the form returns the activity names in an unexpected format.

---

## Security & Privacy

- This script sends email from the Google account that created the trigger.
- Do not commit live form response data or recipient email addresses to a public repository.
- Only share the script with users who need to maintain it.
- The original form and tracking sheet are intentionally not included here.

---

## License

This script is provided as-is for internal use by the Recreation Worcester program. Modify freely to fit your needs.
```

This README will help future users set up and maintain the script without requiring access to your original form or spreadsheet.
