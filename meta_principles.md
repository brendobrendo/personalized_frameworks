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
function getEmailTemplate(cssContent, htmlContent) {
  return `
    <!DOCTYPE html>
    <html>
    <head>
      <link rel="stylesheet" href="https://fonts.googleapis.com/css2?family=Material+Symbols+Outlined:opsz,wght,FILL,GRAD@24,400,0,0" />
      <style>
        ${cssContent}
      </style>
    </head>
    <body style="font-family: Arial; background-color: #f8f1e6; border-radius: 10px">
      <!-- Common header here -->
      ${htmlContent}
      <!-- Common footer here -->
    </body>
    </html>
  `;
}

function getConfirmAndAbsentButtonContent() {
  const cssContent = `
    .confirm-button {
      background-color: #772432;
      color: #FFFFFF !important;
      font-weight: bold;
      padding: 15px 25px;
      border: none;
      border-radius: 5px;
      cursor: pointer;
      text-decoration: none;
      display: inline-block;
    }
    .absence-button {
      background-color: #FFEB88;
      color: black !important;
      font-weight: bold;
      padding: 15px 25px;
      border: none;
      border-radius: 5px;
      cursor: pointer;
      text-decoration: none;
      display: inline-block;
    }
    .button-container {
      display: flex;
      justify-content: center;
      margin: 20px 0;
    }
  `

  const htmlContent = `
    <div class="button-container">
      <div style="text-align: center; margin: 10px;">
        <a href="https://docs.google.com/forms/d/e/1FAIpQLSfyku3vNUYS_58DLrhDCUXNB9n1_rT8smZVgG7vOlYP6wUnCg/viewform" class="confirm-button">Confirm Roles</a>
      </div>
      <div style="text-align: center; margin: 10px;">
        <a href="https://docs.google.com/forms/d/e/1FAIpQLScVOhNKlIvd7ew8bkDBI7uJZiHuEkSpVw2kUFM02mZLY8RBoA/viewform?usp=sf_link" class="absence-button">Enter Absence</a>
      </div>
    </div>
  `

  return { cssContent, htmlContent };
}

function getSpecialNoticeContent() {
  const htmlContent = `
    <div style="padding: 5px 10px;">
      <div style="background-color: #FFEB88; padding: 10px; border-radius: 5px; margin: 20px 0;">
        <h2 style="color: #772432;">Important Club News</h2>
        <p>The location for the Division Contest has been finalized! It will take place on <strong>Saturday, April 26th from 1-3 PM</strong> at the <strong>Crossroads Community Center in Bellevue, WA</strong>.</p>
        <p><strong>Andres Garcia</strong> and <strong>Diana Lillig</strong> will be competing in their respective contests — International Speech and Table Topics. Let’s cheer them on!</p>
        <p>Register to attend and support our club members by clicking the link below:</p>
        <p><a href="https://d2tm.org/event/division-c-contest/" target="_blank" style="color: #3366CC;">Click here to register for the Division Contest</a></p>
      </div>
    </div>
  `
  return { htmlContent }
}

/**
 * Helper function for generateKeyRoleConfirmationEmail
 */
function findMemberInfo(memberName) {

  for (const member of memberData) {
    if (member[1] === memberName) {
      return [member[2], member[12]];
    }
  }
}

/**
 * Generates and sends the html email for the newly generated key roles (4 weeks out).
 */
function generateKeyRoleConfirmationEmail() {
  const subject = 'Key Role Confirmation Email'
  const upcomingMeetingRoles = upcomingMeetings.getRange('A2:P' + 1).getValues()[1];
  // Get fullnames for TM and each speaker 
  const toastmaster = upcomingMeetingRoles[1];
  let toastmasterInfo = findMemberInfo(toastmaster);
  const speaker1 = upcomingMeetingRoles[4];
  const speaker1Info = findMemberInfo(speaker1);
  const speaker2 = upcomingMeetingRoles[5];
  const speaker2Info = findMemberInfo(speaker2);
  const speaker3 = upcomingMeetingRoles[6];
  const speaker3Info = findMemberInfo(speaker3);

  // format date
  const dateString = upcomingMeetingRoles[0];
  const formattedDate = formatDate(dateString);

  // Get Confirm and Enter Absence buttons content
  const confirmAndAbsentButtonContent = getConfirmAndAbsentButtonContent();

  // non-template html
  const nonTemplateHtmlBody = `
    <div style="background-color: #772432; color: #FFEB88; font-weight: bold; padding: 10px; border-top-left-radius: 10px; border-top-right-radius: 10px;">
      <h1>Key Role Confirmations</h1>
      <h3>For the week of Thursday ${formattedDate}</h3>
    </div>
    <div style="padding: 5px 10px;">
      <p>Hi ${toastmasterInfo[0]}, ${speaker1Info[0]}, ${speaker2Info[0]}, and ${speaker3Info[0]}</p>
      <p>Please confirm if you can fulfill your role on ${formattedDate} or log your absence using the buttons below. We're here to help with any questions or reservations! A quick response will allow us to finalize the Weekly Update email.</p>
      <ul>
        <li>Toastmaster: ${toastmaster}</li>
        <li>Speaker 1: ${speaker1}</li>
        <li>Speaker 2: ${speaker2}</li>
        <li>Speaker 3: ${speaker3}</li>
      </ul>
      <p>Thanks,<p>
      <p>Brendan</p>
    </div>
    ${confirmAndAbsentButtonContent.htmlContent}
  `;

  htmlBody = getEmailTemplate(confirmAndAbsentButtonContent.cssContent, nonTemplateHtmlBody);
  
  const emails = [toastmasterInfo[1], speaker1Info[1], speaker2Info[1], speaker3Info[1]]
  
  emails.forEach((email) => {
    GmailApp.sendEmail(email, subject, "", {
      htmlBody: htmlBody
    });
  })
}

/**
 * Sends email with individualized role assignments.
 */
function sendHighlightedAssignmentEmails() {
  memberData.forEach((row) => {
    if (row[14] === 'Yes') {
      Logger.log(row[1]);
      generateHighlightedRoleAssignmentEmail(row[12], row[1], row[2]);
    }
  })
}

/**
 * Generate html email with individualied role assignments
 * @param {string} recipient - Recipient's email address
 * @param {string} specialName - Recipient's first and last name
 * @param {string} specialFirstName - Recipient's first name
 */
function generateHighlightedRoleAssignmentEmail(recipient, specialName, specialFirstName) {
  const subject = 'Weekly Update Email'
  const body = getDraftEmailContent();

  // Split the content into paragraphs
  let paragraphs = body.split('\n');

  paragraphs[1] = paragraphs[1].replace("Absence Form", `<a href="https://docs.google.com/forms/d/e/1FAIpQLScVOhNKlIvd7ew8bkDBI7uJZiHuEkSpVw2kUFM02mZLY8RBoA/viewform?usp=sf_link">Absence Form</a>`);

  // Convert paragraphs starting from the second to numbered bullet points
  const numberedPoints = paragraphs.slice(1)
    .map((point, index) => `<li>${point}</li>`)
    .join('');

  // Insert these bullet points into an ordered list
  const orderedListHtml = `<ol>${numberedPoints}</ol>`;

  let topSection = '<ul>';
  let rolesSection = '';
  const upcomingMeetings = getUpcomingMeetingDataUpdated();

  // Define role names corresponding to the positions in the sub-array
  const roleNames = ['Toastmaster', 'General Evaluator', 'Topics Master',
                      'Speaker 1', 'Speaker 2', 'Speaker 3', 'Evaluator 1',
                      'Evaluator 2', 'Evaluator 3', 'Grammarian', 'Ah Counter',
                      'Timer', 'Vote Counter', 'Host 1', 'Host 2'];

  // Define the specific name to be highlighted
  const starEmoji = '⭐';

  // Process each meeting
  for (let i = upcomingMeetings.length - 1; i >= 0; i--) {
    // Extract meeting date and roles
    const [dateString, ...roles] = upcomingMeetings[i];

    const formattedDate = formatDate(dateString);

    // Generate HTML for role assignments with star emoji for special name
    const rolesHtml = roles.map((name, index) => {
      const roleText = name === specialName ? `${name} ${starEmoji}`: name;
      return `<li><strong>${roleNames[index]}:</strong> ${roleText}</li>`;
    }).join('');

    let assignedRolesForWeek = '';

    for (let i = 0; i < roles.length; i++) {
      roles[i] === specialName ? assignedRolesForWeek += roleNames[i] + ', ' : null;
    }

    if (assignedRolesForWeek === '') {
      assignedRolesForWeek += 'No Role Assigned'
    } else {
      assignedRolesForWeek.slice(0, -6);
    }

    // Wrap roles in an unordered list and a section for each meeting
    const meetingSection = `<div style="padding: 5px 10px;">
      <h3>Meeting of ${formattedDate}</h3>
      <ul>${rolesHtml}</ul>
    </div>`;

    // Add this section to the roles section
    rolesSection += meetingSection;
    topSection += `<li><strong>${formattedDate}:</strong> ${assignedRolesForWeek}`
  };

  topSection += `</ul>`

  // Replace the original paragraphs from the second onward with the formatted HTML list
  paragraphs = [paragraphs[0], orderedListHtml];

  const goingIntoWeekOfDate = getNextThursday();

  const confirmAndAbsentButtonContent = getConfirmAndAbsentButtonContent();
  const specialNoticeContent = getSpecialNoticeContent();

  // Use template literals for HTML content to enhance readability and maintainability
  const nonHtmlBody = `
      <div style="background-color: #772432; color: #FFEB88; font-weight: bold; padding: 10px; border-top-left-radius: 10px; border-top-right-radius: 10px;">
        <h1>WEEKLY UPDATE</h1>
        <h3>Going into the week of Thursday ${goingIntoWeekOfDate}</h3>
      </div>
      <div style="padding: 5px 10px;">
        <h3>Hi ${specialFirstName},</h3>
        <p>Here are your upcoming role assignments. Visit our
          <a href="https://sites.google.com/site/toastmasters4741/home/meeting-roles" style="color: #772432; text-decoration: underline;">internal club website</a> for helpful tips to prepare for them.
        </p>
        ${topSection}
      </div>
      ${confirmAndAbsentButtonContent.htmlContent}
      ${specialNoticeContent.htmlContent}
      <h2 style="padding: 5px 10px;"><strong>Upcoming Role Assignments</strong></h2>
      ${rolesSection}
  `;
  
  const htmlBody = getEmailTemplate(confirmAndAbsentButtonContent.cssContent, nonHtmlBody);

  GmailApp.sendEmail(recipient, subject, "", {
    htmlBody: htmlBody
  });
}

const extractMemberDataForOfficers = (memberDataArray) => {
  return memberDataArray.map(function(memberDataRow) { // Relevant member data is name, eligibleForPreparedRole, attendanceFrequencyCategory
    return [memberDataRow[1], memberDataRow[15], memberDataRow[16]]
  })
}

const  createMemberCategorizationObjectForOfficers = (extractedMemberArray) => {
  const regularAttendingMemberEligibleForSpeakerRole = [];
  const upAndComingMember = [];
  const onTheFenceMember = [];
  const formerMember = [];
  const newMember = [];

  extractedMemberArray.forEach(subarray => {
    const secondElement = subarray[1];
    const thirdElement = subarray[2];

    if (thirdElement === 'Former Member') {
      formerMember.push(subarray);
    } else if (thirdElement === 'New Member') {
      newMember.push(subarray);
    } else if (secondElement === 'Yes' && thirdElement !== 'Former Member' && thirdElement !== 'Never') {
      regularAttendingMemberEligibleForSpeakerRole.push(subarray);
    } else if (secondElement === 'No' && thirdElement !== 'Former Member' && thirdElement !== 'Never') {
      upAndComingMember.push(subarray);
    } else if (secondElement === 'Maybe' && thirdElement !== 'Former Member' && thirdElement !== 'Never') {
      onTheFenceMember.push(subarray);
    }
  });

  // Eligible prepared speaker categories
  const attendanceOrder = {
    'Always': 1,
    'Frequent': 2,
    'Occasional': 3,
    'Returning Member': 4,
    'Rare': 5
  }

  // Sort the 'regularAttendingMemberEligibleForSpeakerRole' array
  regularAttendingMemberEligibleForSpeakerRole.sort((a, b) => {
    const aValue = attendanceOrder[a[2]] || 99; // Default to a high number if not found.
    const bValue = attendanceOrder[b[2]] || 99;
    return aValue - bValue;
  });

  return {
    regularAttendingMemberEligibleForSpeakerRole,
    upAndComingMember,
    onTheFenceMember,
    formerMember,
    newMember
  };
};

/**
 * @param {string} recipient - Email address for the intended recipient.
 */
function sendMemberStatusEmail(recipient, categorizedObject) {
  const subject = 'Member Status Update' 


  const nonTemplateHtmlBody = `
    <div style="font-family: Arial; background-color: #f8f1e6; border-radius: 10px">
      <div style="background-color: #772432; color: #FFEB88; font-weight: bold; padding: 10px; border-top-left-radius: 10px; border-top-right-radius: 10px;">
        <h2>MEMBER STATUS</h2>
      </div>
      <div style="padding: 10px">
        <p>For the meeting later today...</p>
        <h3>Main Speaker Queue (${categorizedObject['regularAttendingMemberEligibleForSpeakerRole'].length})</h3>
        <table style="width: 100%; border-collapse: collapse;">
          <thead>
            <tr>
              <th style="border: 1px solid #ddd; padding: 8px;">Member Name</th>
              <th style="border: 1px solid #ddd; padding: 8px;">Attendance Pattern</th>
            </tr>
          </thead>
          <tbody>
            ${categorizedObject['regularAttendingMemberEligibleForSpeakerRole'].map(member => 
              `<tr>
                <td style="border: 1px solid #ddd; padding: 8px;">${member[0]}</td>
                <td style="border: 1px solid #ddd; padding: 8px;">${member[2]}</td>
              </tr>
            `).join('')}
          </tbody>
        </table>
        <h3>Nearly Ready Members (${categorizedObject['onTheFenceMember'].length})</h3>
        <table style="width: 100%; border-collapse: collapse;">
          <thead>
            <tr>
              <th style="border: 1px solid #ddd; padding: 8px;">Member Name</th>
              <th style="border: 1px solid #ddd; padding: 8px;">Attendance Pattern</th>
            </tr>
          </thead>
          <tbody>
            ${categorizedObject['onTheFenceMember'].map(member => 
              `<tr>
                <td style="border: 1px solid #ddd; padding: 8px;">${member[0]}</td>
                <td style="border: 1px solid #ddd; padding: 8px;">${member[2]}</td>
              </tr>
            `).join('')}
          </tbody>
        </table>
        <h3>New Members (${categorizedObject['newMember'].length})</h3>
        <table style="width: 100%; border-collapse: collapse;">
          <thead>
            <tr>
              <th style="border: 1px solid #ddd; padding: 8px;">Member Name</th>
              <th style="border: 1px solid #ddd; padding: 8px;">Attendance Pattern</th>
            </tr>
          </thead>
          <tbody>
            ${categorizedObject['newMember'].map(member => 
              `<tr>
                <td style="border: 1px solid #ddd; padding: 8px;">${member[0]}</td>
                <td style="border: 1px solid #ddd; padding: 8px;">${member[2]}</td>
              </tr>
            `).join('')}
          </tbody>
        </table>
        <h3>Other (${categorizedObject['upAndComingMember'].length})</h3>
        <table style="width: 100%; border-collapse: collapse;">
          <thead>
            <tr>
              <th style="border: 1px solid #ddd; padding: 8px;">Member Name</th>
              <th style="border: 1px solid #ddd; padding: 8px;">Attendance Pattern</th>
            </tr>
          </thead>
          <tbody>
            ${categorizedObject['upAndComingMember'].map(member => 
              `<tr>
                <td style="border: 1px solid #ddd; padding: 8px;">${member[0]}</td>
                <td style="border: 1px solid #ddd; padding: 8px;">${member[2]}</td>
              </tr>
            `).join('')}
          </tbody>
        </table>
      </div>
    </div>
  `;

  const htmlBody = getEmailTemplate('', nonTemplateHtmlBody);

  GmailApp.sendEmail(recipient, subject, "", {
    htmlBody: htmlBody
  });
}
```