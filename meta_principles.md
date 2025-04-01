## Meta Principles
- Be engaged
- Know how to use AI
- Be a tinkerer
- Figure out what people want
- Be useful
- Be resilient
- Be adaptable
- Learn how you learn
- Figure things out

## Work Principles
- Be on time
- Look like you can take care of peoples' things
- Make eye contact
- Say peoples' names
- Ask questions
- Use open hand gestures
- When in doubt speak in short simple sentences
- Leave notes that make it easy for people to follow what you did
- Understand the KPIs

```javascript
function deleteFilteredRows() {
  const nextThursday = new Date(getNextThursday());

  const sourceSheet = getSheetByName('TimeOffRequests');
  const destinationSheet = getSheetByName('FilteredAbsences');

  const rowsToDelete = []; // Store actual row numbers to delete

  roleConfirmationsFormResponsesData.forEach((subArray, i) => {
    const rawDates = subArray.slice(2).join(', ');
    const dateList = rawDates.split(',').map(date => date.trim());

    const hasFutureDate = dateList.some(dateStr => {
      const parsedDate = new Date(dateStr);
      return parsedDate > nextThursday;
    });

    if (hasFutureDate) {
      rowsToDelete.push(100 + i + 1); // Convert from slice index to 1-based row number
    }
  });

  // Delete from source sheet (bottom to top!)
  if (rowsToDelete.length > 0) {
    rowsToDelete
      .sort((a, b) => b - a) // Descending order
      .forEach(rowNum => {
        sourceSheet.deleteRow(rowNum);
      });
  }
}
```