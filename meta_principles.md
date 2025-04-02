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
- Blink
- Smile
- Say peoples' names
- Ask questions
- Use open hand gestures
- When in doubt speak in short simple sentences
- Leave notes that make it easy for people to follow what you did
- Understand the KPIs

```javascript
function addConfirmationsToTable() {
  
  const emailToNameMap = {};
  memberData.forEach((memberRow) => {
    const name = memberRow[1];
    const email = memberRow[12];
    if (email && name) {
      emailToNameMap[email.trim()] = name.trim();
    }
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
  // const oneWeekOut = new Date(roleConfirmationsFormResponsesData[0][0]);

  // roleConfirmationsFormResponsesData.forEach((subArray) => {
  //   const rawDates = subArray.slice(2).join(', ');
  //   const dateList = rawDates.split(',').map(date => date.trim());

  //   dateList.forEach((date) => {
  //     let formattedDate = new Date(date);
  //     if (formattedDate.getTime() === oneWeekOut.getTime()) {
  //       Logger.log('The date matches!');
  //     }
  //   })
  // })
}
```