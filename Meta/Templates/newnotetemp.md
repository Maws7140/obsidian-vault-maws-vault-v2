---
title: newnotetemp
created: 2025-07-31
modified: 2025-12-19
---
<%*
//////////////////////////////////////////////////////////////////////////////
// New note template — auto‑creates & opens the new file instantly
//////////////////////////////////////////////////////////////////////////////

// Prompt for the descriptive title that follows the date
const logTitle = await tp.system.prompt("Note title");
if (!logTitle) { throw new Error("Need a title to continue!"); }  // Stop if empty

// Build today’s date string (e.g., "7-8-2025")
const dateStr = tp.date.now("M-D-YYYY");

// Assemble the destination file path
const baseFolder = "Quick notes";
const destFile   = `${baseFolder}/${dateStr} ${logTitle}`;

// Compose the Markdown body of the new note
const body = ` 

`;

// Create the file (parent folders are created if they don't exist)
const newFile = await tp.file.create_new(body, destFile);

// Open the freshly‑created note in a new pane
await app.workspace.getLeaf(true).openFile(newFile);

//////////////////////////////////////////////////////////////////////////////
// End of Templater script
//////////////////////////////////////////////////////////////////////////////
-%>
