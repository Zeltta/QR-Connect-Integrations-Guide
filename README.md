
To create dashboards and reports that go beyond QR Connect's current features, you can import your data into data analysis programs using an API request.

This guide uses Google sheets but the basic principles will also apply to other data analysis programs like Excel and Power BI.

<br/>

## Contents

- [Creating an API Key](#creating-an-API-key)
- [Importing data into Google Sheets](#importing-data-into-Google-Sheets)

<br/>

# Create an API Key

> API Keys are random combinations of letters and numbers that are extremely hard to guess. They enable secure access your QR Connect data from applications like Google Sheets, Excel etc.

1. Go to your company page and click on `Integrations` to navigate to the integrations screen.
2. Click the `Create new API key` button
3. Give your key a name that will help you remember what it is being used for.
4. Click `Create API key`
5. You will copy and paste this key later on to enable secure access to your data.

<br/>

# Import data into Google Sheets

1. Create a new google sheet.

2. In your new google sheet, open the `Tools` menu and then click on `Script Editor`

3. Copy and paste the code below into your apps script:

```js
const API_KEY = "REPLACE_WITH_YOUR_API_KEY"; // <-- replace with your key that you just created. Keep the double quotes (") on each side.
const API_URL = "https://api.qrconnect.nz/views";

// Function that adds 'Fetch data' button to spreadsheet.
function onOpen() {
  const ui = SpreadsheetApp.getUi();
  ui.createMenu("QR Connect").addItem("Fetch data", "fetchData").addToUi();
}

// Function that fetches the data
function fetchData() {
  try {
    const response = UrlFetchApp.fetch(API_URL, {
      method: "post",
      mode: "cors",
      headers: {
        "Content-Type": "application/json",
      },
      payload: JSON.stringify({
        apiKey: API_KEY,
        projectId: "", // <-- restrict to submissions from a specific folder (or leave blank)
        pageId: "", // <-- restrict to submisisons from a specific page (or leave blank)
        submissionsOnly: false, // <-- set to true to only get views that resulted in form submissions.
      }),
    });
    const json = response.getContentText();
    const data = JSON.parse(json);

    Logger.log(data);
    printDataToRows(data);
  } catch (e) {
    Logger.log(e);
  }
}

// function that prints the data to the spreadsheet
function printDataToRows(data) {
  const sheet = SpreadsheetApp.getActiveSheet();

  for (let i = 0; i < data.length; i += 1) {
    const {
      id,
      createdAt,
      pageId,
      userId,
      lat,
      lng,
      user,
      locationAccuracy,
      submission,
      page: {
        title,
        project: { name },
      },
    } = data[i];
    const time = new Date(createdAt).toDateString();

    let fullName = "n/a";
    if (user) fullName = user.firstName + " " + user.lastName;

    let pageContent = "n/a";
    if (submission) {
      pageContent = submission.content
        .map((a) => {
          return `${JSON.stringify(a.data)}`;
        })
        .join(", \n")
        .replaceAll("{", "")
        .replaceAll("}", "")
        .replaceAll("[", "")
        .replaceAll("]", "");
    }

    const row = [
      time,
      fullName,
      title,
      name,
      lat ? lat : "n/a",
      lng ? lng : "n/a",
      locationAccuracy ? locationAccuracy : "n/a",
      pageContent,
    ];

    const columnLabels = [
      "Time",
      "Full name",
      "Page title",
      "Folder name",
      "Lat",
      "Lng",
      "Location accuracy",
      "Submission",
    ];
    if (i === 0) {
      sheet.getRange(i + 1, 1, 1, 8).setValues([columnLabels]);
    }
    sheet.getRange(i + 2, 1, 1, 8).setValues([row]);
  }
}
```

4. You only need to make one change. Paste in your new API key into the top line to replace the `REPLACE_WITH_YOUR_API_KEY` bit.

5. Save your script and run it to make sure it works. You will be asked for permission to run it, accept the permissions.

6. Now go back to your sheet. You will now see a new menu item called `QR Connect`. Open the new menu and click on `Fetch data`.

7. All done! You can now use fetch this data and use it however you wish 🎉. 

## Bonus tip

You can also set up your script to fetch automatically at specific intervals by going back into your script and setting up a trigger.
