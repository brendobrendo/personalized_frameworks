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
function processRoleConfirmations() {
  const formSheet = getSheetByName('Role Confirmation Responses');
  const meetingsSheet = getSheetByName('Meeting Confirmations');
  const memberInfoSheet = getSheetByName('Member Info');

  const formData = formSheet.getRange(2, 1, formSheet.getLastRow() - 1, 3).getValues();
  const meetingsData = meetingsSheet.getRange(2, 1, meetingsSheet.getLastRow() - 1, 2).getValues();
  const memberInfo = memberInfoSheet.getRange(2, 1, memberInfoSheet.getLastRow() - 1, 2).getValues();

  const emailToNameMap = {};
  memberInfo.forEach(([email, name]) => {
    emailToNameMap[email.trim().toLowerCase()] = name.trim();
  });

  formData.forEach(([timestamp, email, dateListRaw]) => {
    const name = emailToNameMap[email.trim().toLowerCase()];
    if (!name) {
      Logger.log(`No name found for email: ${email}`);
      return;
    }

    const confirmedDates = dateListRaw
      .split(',')
      .map(d => new Date(d.trim()))
      .map(d => new Date(d.getFullYear(), d.getMonth(), d.getDate())); // Strip time

    confirmedDates.forEach(date => {
      for (let i = 0; i < meetingsData.length; i++) {
        const [meetingDate, memberList] = meetingsData[i];
        const normalizedMeetingDate = new Date(meetingDate.getFullYear(), meetingDate.getMonth(), meetingDate.getDate());

        if (date.getTime() === normalizedMeetingDate.getTime()) {
          const existingNames = memberList ? memberList.split(',').map(n => n.trim().toLowerCase()) : [];

          if (!existingNames.includes(name.toLowerCase())) {
            existingNames.push(name);
            const formattedNames = existingNames.map(n => {
              // Format the names back to proper case using memberInfo
              for (let email in emailToNameMap) {
                if (emailToNameMap[email].toLowerCase() === n) {
                  return emailToNameMap[email];
                }
              }
              return n; // fallback
            });

            meetingsSheet.getRange(i + 2, 2).setValue(formattedNames.join(', '));
            Logger.log(`✅ Added ${name} to ${date.toLocaleDateString()}`);
          } else {
            Logger.log(`⚠️  Skipped duplicate: ${name} already confirmed for ${date.toLocaleDateString()}`);
          }

          break;
        }
      }
    });
  });
}
```