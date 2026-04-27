---
modified: 2025-12-23
title: ${logTitle}
created: 2025-08-08
---
 

<%*
//////////////////////////////////////////////////////////////////////////////
// New SWE template - auto-creates & opens the new file instantly
//////////////////////////////////////////////////////////////////////////////

// Prompt for the descriptive title that follows the date
const rawTitle = await tp.system.prompt("Project Name");
if (!rawTitle) {
  throw new Error("Need a title to continue!");
}
const logTitle = rawTitle.trim();

// Build today's date string (e.g., "7-8-2025")
const dateStr = tp.date.now("M-D-YYYY");
const isoDate = tp.date.now("YYYY-MM-DD");

// Assemble the destination file path
const baseFolder = "Projects";
const safeTitle = logTitle.replace(/[\\\\/:]/g, "-");
const destFile = `${baseFolder}/${dateStr} ${safeTitle}.md`;

// Compose the Markdown body of the new note
const body = `---
title: ${logTitle}
created: ${isoDate}
modified: ${isoDate}
tags: project
Status: 
Phase: 
Project:

---
<progress value="0" max="100"></progress>
#project
# ${logTitle}

## Details 


## Goals & Milestones
- [ ] Goal 1:  <!-- task-id: mgx1a8fz -->
- [ ] Goal 2:  <!-- task-id: mgx1a913 -->
- [ ] Goal 3: <!-- task-id: mgx1a9m1 -->
- [ ] Goal 4: <!-- task-id: mgx1a9zy -->
- [ ] Goal 5: <!-- task-id: mgx1aasg -->
- [ ] Goal 6: <!-- task-id: mgx1abd8 -->
- [ ] Goal 7:  <!-- task-id: mgx1ac5j -->
- [ ] Goal 8:  <!-- task-id: mgx1acqh -->
- [ ] Goal 9:  <!-- task-id: mgx1adit -->
- [ ] Goal 10: <!-- task-id: mgx1aebd -->

## Technical Roadmap 


## Notes
 



### Resources & Links

# Timeline 
~~~cards-timeline-h
@card
time: 2024-12-12
title: Beta
content:
Plug in launched into beta

@card [color-red]
time: 2024-12-12
title: Launch
content:
Plug accepted into obsidian store 

@card [color-green]
time: 2024-12-12
title: Launch
content:
Plug accepted into obsidian store 

@card [color-blue]
time: 2024-12-12
title: Launch
content:
Plug accepted into obsidian store 
~~~

---

`;

// Create the file (parent folders are created if they don't exist)
const newFile = await tp.file.create_new(body, destFile);

// Open the freshly-created note in a new pane
await app.workspace.getLeaf(true).openFile(newFile);

//////////////////////////////////////////////////////////////////////////////
// End of Templater script
//////////////////////////////////////////////////////////////////////////////
-%>
