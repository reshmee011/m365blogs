---
title: "Use Open AI in Excel"
date: 2024-02-04T11:15:28Z
tags: ["OpenAi","Excel","Script", "Google Spreadsheet"]
draft: true
---

# Use Open AI in Excel

## Google Spreadsheet Apps Script

```typescript
async function main(workbook: ExcelScript.Workbook) {
    // Get the active cell and worksheet.
    let selectedCell = workbook.getActiveCell();
    let selectedSheet = workbook.getActiveWorksheet();

    // Get the text content of the selected cell.
    let cellText = selectedCell.getValues()[0][0];

    // Define the chatAPI function within the main function.
    async function chatAPI(cellText: string): Promise<string> {
        const url = "https://api.openai.com/v1/chat/completions";
        const apiKey = "API Key"; // Replace with your OpenAI API key

        const requestOptions: RequestInit = {
            method: "POST",
            headers: {
                "Content-Type": "application/json",
                "Authorization": "Bearer " + apiKey,
            },
            body: JSON.stringify({
                "model": "gpt-3.5-turbo",
                "messages": [
                    {
                        "role": "system",
                        "content": "You are a helpful assistant."
                    },
                    {
                        "role": "user",
                        "content": cellText + " in French is:"
                    }
                ]
            }),
        };

        try {
            const response: Response = await fetch(url, requestOptions);
            const data: { choices: { message: { content: string } }[] } = await response.json();
            const message: string = data.choices[0].message.content; // Accessing the response content

            return message;
        } catch (error) {
            console.log("Error fetching data:", error);
            throw error;
        }
    }

    // Call the chatAPI function with the cell text.
    try {
        const response  = await chatAPI(cellText.toString());

        // TODO: Handle the response, for example, you can insert the response into a nearby cell.
        const targetCell = selectedSheet.getCell(selectedCell.getRowIndex(), selectedCell.getColumnIndex() + 1);
        targetCell.setValues([[response]]);
    } catch (error) {
        // Handle errors here, e.g., log or display an error message.
        console.log("Error:", error);
    }
}

```

## References


