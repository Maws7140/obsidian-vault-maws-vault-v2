---
title: Ds410temp
created: 2025-09-07
modified: 2025-12-23
---
<%*
//////////////////////////////////////////////////////////////////////////////
// New note template — auto-creates & opens the new file instantly
//////////////////////////////////////////////////////////////////////////////

// Prompt for the descriptive title that follows the date
const logTitle = await tp.system.prompt("Note title");
if (!logTitle) { throw new Error("Need a title to continue!"); }  // Stop if empty

// Build today’s date string (e.g., "7-8-2025")
const dateStr = tp.date.now("M-D-YYYY");

// Assemble the destination file path
const baseFolder = "School/Class 4";
const destFile   = `${baseFolder}/${dateStr} ${logTitle}`; // you can add ".md" if you prefer

// Compose the Markdown body of the new note
const body = `# ${dateStr} ${logTitle}
---
title: ${logTitle}
created: ${isoDate}
modified: ${isoDate}
tags: class4
color: #0f0cdf 
---
#class4
`;

// Create the file (parent folders are created if they don't exist)
const newFile = await tp.file.create_new(body, destFile);

// Open the freshly-created note in a new pane
await app.workspace.getLeaf(true).openFile(newFile);

//////////////////////////////////////////////////////////////////////////////
// End of Templater script
//////////////////////////////////////////////////////////////////////////////
-%>
