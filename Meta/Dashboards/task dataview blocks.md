---
modified: 2025-12-23
---
Dataviews for tasks 
---

### Option 1: Centered Kanban (Clickable)

```dataviewjs
// ============================================================
// CONFIGURATION SECTION
// ============================================================
// Set the height of the scrollable area
const height = "400px";
// Set the width of each column
const cardWidth = "320px";

// ============================================================
// DATA GATHERING
// ============================================================
// Get all tasks from all pages in the vault
const tasks = dv.pages().file.tasks;

// Organize tasks into groups (To Do vs Done)
const groups = [
    // Filter for incomplete tasks
    { name: "📋 To Do", tasks: tasks.where(t => !t.completed) },
    // Filter for completed tasks
    { name: "✅ Done", tasks: tasks.where(t => t.completed) }
];

// ============================================================
// UNIQUE ID GENERATION
// ============================================================
// Generate a random ID (e.g., "kanban-xyz123") to prevent CSS conflicts
// if you use this script multiple times in one note.
const id = "kanban-center-" + Math.random().toString(36).substring(2, 9);

// ============================================================
// CSS STYLING
// ============================================================
const css = `
  /* The Main Container */
  #${id} {
    display: flex;              /* Layout children in a row */
    gap: 20px;                  /* Space between columns */
    overflow-x: auto;           /* Allow horizontal scrolling if needed */
    padding: 10px 0;            /* Top/Bottom padding */
    justify-content: center;    /* Center the columns in the middle of the page */
    min-width: 100%;            /* Ensure it takes full width */
  }

  /* The Column (To Do / Done) */
  #${id} .column {
    flex: 0 0 ${cardWidth};     /* Fixed width, do not shrink */
    display: flex;              /* Flex layout for header/content inside */
    flex-direction: column;     /* Stack items vertically */
    background-color: var(--background-secondary); /* Obsidian secondary color */
    border: 1px solid var(--background-modifier-border); /* Subtle border */
    border-radius: 8px;         /* Rounded corners */
    height: ${height};          /* Fixed height */
    box-shadow: 0 4px 6px rgba(0,0,0,0.05); /* Slight shadow */
  }

  /* The Column Header */
  #${id} .header {
    padding: 12px 16px;
    border-bottom: 1px solid var(--background-modifier-border);
    font-weight: 600;
    display: flex;
    justify-content: space-between; /* Push name and count to edges */
    align-items: center;
    background-color: var(--background-secondary-alt);
    border-radius: 8px 8px 0 0; /* Round only top corners */
  }

  /* The Task Counter Pill */
  #${id} .count {
    background: var(--background-primary);
    padding: 2px 8px;
    border-radius: 12px;
    font-size: 0.75em;
    border: 1px solid var(--background-modifier-border);
  }

  /* The Scrollable List Area */
  #${id} .task-list {
    overflow-y: auto;           /* Allow vertical scrolling inside column */
    padding: 12px;
    flex-grow: 1;               /* Take up remaining height */
    scrollbar-width: thin;      /* Thin scrollbar for Firefox */
  }

  /* The Individual Task Card */
  #${id} .task-card {
    background-color: var(--background-primary);
    padding: 12px;
    margin-bottom: 10px;        /* Space between cards */
    border-radius: 6px;
    border: 1px solid var(--background-modifier-border);
    font-size: 0.9em;
    transition: transform 0.15s ease-in-out, border-color 0.15s; /* Smooth hover animation */
    cursor: pointer;            /* Shows the hand icon to indicate clicking works */
  }

  /* Hover Effect for Card */
  #${id} .task-card:hover {
    border-color: var(--interactive-accent); /* Highlight border color */
    transform: translateY(-1px); /* Move up slightly */
    box-shadow: 0 2px 4px rgba(0,0,0,0.1);
  }

  /* The Source Link Text */
  #${id} .task-source {
    display: block;
    margin-top: 6px;
    font-size: 0.8em;
    color: var(--text-muted);   /* Greyed out text */
    font-style: italic;
  }
`;

// ============================================================
// RENDER LOGIC
// ============================================================
const container = dv.container;

// Inject CSS
const styleEl = container.createEl('style');
styleEl.textContent = css;

// Create Main Wrapper
const wrapper = container.createEl('div', { attr: { id: id } });

// Loop through our groups (To Do / Done)
groups.forEach(group => {
    // Create Column
    const col = wrapper.createEl('div', { cls: 'column' });
    
    // Create Header
    const header = col.createEl('div', { cls: 'header' });
    header.createEl('span', { text: group.name });
    header.createEl('span', { cls: 'count', text: `${group.tasks.length}` });

    // Create List Container
    const list = col.createEl('div', { cls: 'task-list' });
    
    // Handle Empty State
    if (group.tasks.length === 0) {
        list.createEl('div', { 
            text: "All caught up!", 
            attr: { style: "color: var(--text-muted); text-align: center; padding: 20px; font-style: italic;" }
        });
    }

    // Loop through tasks (Limit to 50 to prevent freezing on huge vaults)
    for (let t of group.tasks.limit(50)) { 
        // Create the Card
        const card = list.createEl('div', { cls: 'task-card' });
        
        // Clean the text (remove [ ] or [x] checkboxes)
        const cleanText = t.text.replace(/\[.\]/, "").trim(); 
        
        // Render the task text
        dv.span(cleanText, { container: card });
        
        // Render the source file name at bottom
        card.createEl('div', { 
            cls: 'task-source', 
            text: "📄 " + t.link.path.replace(".md", "") 
        });

        // ============================================================
        // CLICK HANDLER
        // ============================================================
        card.addEventListener('click', async (e) => {
            // Check if Control (Win) or Command (Mac) key is held down
            const newTab = e.ctrlKey || e.metaKey;
            
            // Use Obsidian's internal API to open the link
            // t.link.path contains the file path (e.g., "Folder/Note.md")
            await app.workspace.openLinkText(t.link.path, t.link.path, newTab);
        });
    }
});
```

---

### Option 2: Tabbed Panels (Clickable Rows)

```dataviewjs
// ============================================================
// CONFIGURATION
// ============================================================
const maxFiles = 6; // Maximum number of tabs to display

// ============================================================
// DATA GATHERING
// ============================================================
// Find pages that have at least one incomplete task
const pages = dv.pages().where(p => p.file.tasks.some(t => !t.completed));
const tasksByFile = [];

// Group tasks by file
for (let page of pages) {
    const tasks = page.file.tasks.where(t => !t.completed);
    if(tasks.length > 0) {
        tasksByFile.push({ 
            name: page.file.name, // The file name
            path: page.file.path, // Full path for linking
            tasks: tasks          // The task objects
        });
    }
}
// Sort by which file has the most tasks, then take top X files
const topFiles = tasksByFile.sort((a,b) => b.tasks.length - a.tasks.length).slice(0, maxFiles);

// ============================================================
// UNIQUE ID
// ============================================================
const id = "tabs-" + Math.random().toString(36).substring(2, 9);

// ============================================================
// CSS STYLING
// ============================================================
const css = `
  /* Main Container */
  #${id} {
    border: 1px solid var(--background-modifier-border);
    border-radius: 6px;
    background-color: var(--background-primary);
    display: flex;
    flex-direction: column;
    height: 450px;
    overflow: hidden;
  }
  
  /* Tab Navigation Bar */
  #${id}-nav {
    display: flex;
    background-color: var(--background-secondary);
    border-bottom: 1px solid var(--background-modifier-border);
    overflow-x: auto; /* Allow scrolling tabs horizontally */
  }
  
  /* Individual Tab Buttons */
  #${id} .tab-btn {
    padding: 10px 15px;
    cursor: pointer;
    border: none;
    background: transparent;
    color: var(--text-muted);
    font-size: 0.9em;
    border-right: 1px solid var(--background-modifier-border);
    white-space: nowrap;
    transition: all 0.2s;
  }
  
  /* Tab Hover State */
  #${id} .tab-btn:hover {
    color: var(--text-normal);
    background-color: var(--background-modifier-hover);
  }
  
  /* Active Tab State */
  #${id} .tab-btn.active {
    background-color: var(--background-primary);
    color: var(--interactive-accent);
    font-weight: bold;
    border-bottom: 2px solid var(--interactive-accent); /* Underline */
    margin-bottom: -1px; /* Blend with content area */
  }
  
  /* Content Area */
  #${id}-content {
    flex-grow: 1;
    overflow-y: auto;
    padding: 20px;
  }
  
  /* Hide panes by default */
  #${id} .tab-pane {
    display: none;
    animation: fadein 0.3s;
  }
  
  /* Show active pane */
  #${id} .tab-pane.active {
    display: block;
  }
  
  /* Simple Fade Animation */
  @keyframes fadein {
    from { opacity: 0; }
    to   { opacity: 1; }
  }
  
  /* The Task Row */
  #${id} .task-row {
    padding: 8px 10px;
    border-bottom: 1px solid var(--background-modifier-border);
    display: flex;
    gap: 10px;
    font-size: 0.95em;
    cursor: pointer; /* Interaction hint */
    border-radius: 4px;
    transition: background 0.2s;
  }
  
  /* Row Hover Effect */
  #${id} .task-row:hover {
    background-color: var(--background-secondary);
  }
`;

// ============================================================
// RENDER LOGIC
// ============================================================
const container = dv.container;
container.createEl('style').textContent = css;
const wrapper = container.createEl('div', { attr: { id: id } });

// Create Navigation Bar
const nav = wrapper.createEl('div', { attr: { id: `${id}-nav` } });
// Create Content Area
const contentArea = wrapper.createEl('div', { attr: { id: `${id}-content` } });

topFiles.forEach((file, index) => {
    // 1. Create Tab Button
    const btn = nav.createEl('button', { 
        cls: `tab-btn ${index === 0 ? 'active' : ''}`, 
        text: `${file.name} (${file.tasks.length})` 
    });
    // Add Click Event to Switch Tabs
    btn.onclick = (e) => switchTab(index, e);

    // 2. Create Content Pane
    const pane = contentArea.createEl('div', { 
        cls: `tab-pane ${index === 0 ? 'active' : ''}`,
        attr: { 'data-index': index }
    });

    // 3. Add Header
    dv.header(4, `Tasks in [[${file.name}]]`, { container: pane });

    // 4. List Tasks
    for(let t of file.tasks) {
        const row = pane.createEl('div', { cls: 'task-row' });
        
        // Add text to row
        const textSpan = row.createEl('span', { text: t.text.replace(/\[.\]/, "").trim() });

        // ============================================================
        // CLICK HANDLER
        // ============================================================
        row.addEventListener('click', async (e) => {
            const newTab = e.ctrlKey || e.metaKey;
            // Open the specific note
            await app.workspace.openLinkText(t.link.path, t.link.path, newTab);
        });
    }
});

// ============================================================
// HELPER FUNCTION: Tab Switching Logic
// ============================================================
function switchTab(index, event) {
    // Remove 'active' class from all buttons
    const buttons = nav.querySelectorAll('.tab-btn');
    buttons.forEach(b => b.classList.remove('active'));
    
    // Add 'active' class to clicked button
    event.target.classList.add('active');

    // Remove 'active' class from all panes
    const panes = contentArea.querySelectorAll('.tab-pane');
    panes.forEach(p => {
        p.classList.remove('active');
        // Add 'active' class only to the matching pane
        if(p.getAttribute('data-index') == index) p.classList.add('active');
    });
}
```

---

### Option 3: Compact Grid (Clickable Cards)

```dataviewjs
// ============================================================
// CONFIGURATION
// ============================================================
const maxTasks = 20; // Maximum number of cards to show

// ============================================================
// DATA GATHERING
// ============================================================
// Get incomplete tasks, limited by maxTasks
const tasks = dv.pages().file.tasks.where(t => !t.completed).limit(maxTasks);

// ============================================================
// UNIQUE ID
// ============================================================
const id = "grid-" + Math.random().toString(36).substring(2, 9);

// ============================================================
// CSS STYLING
// ============================================================
const css = `
  /* Grid Container */
  #${id} {
    display: grid;
    /* responsive columns: fill row with as many 200px columns as fit */
    grid-template-columns: repeat(auto-fill, minmax(200px, 1fr));
    gap: 12px;
    margin-top: 10px;
  }
  
  /* Individual Card */
  #${id} .card {
    background-color: var(--background-secondary);
    border: 1px solid var(--background-modifier-border);
    border-radius: 8px;
    padding: 12px;
    display: flex;
    flex-direction: column;
    justify-content: space-between;
    transition: transform 0.2s, box-shadow 0.2s; /* Smooth animations */
    cursor: pointer; /* Clickable cursor */
  }
  
  /* Hover Effects */
  #${id} .card:hover {
    box-shadow: 0 4px 10px rgba(0,0,0,0.1); /* Add drop shadow */
    border-color: var(--interactive-accent); /* Color the border */
    transform: translateY(-2px); /* Lift card up */
  }
  
  /* Text Content */
  #${id} .card-content {
    font-size: 0.9em;
    margin-bottom: 10px;
    line-height: 1.4;
    /* Ellipsis for long text (prevents giant cards) */
    overflow: hidden;
    display: -webkit-box;
    -webkit-line-clamp: 4; /* Limit to 4 lines */
    -webkit-box-orient: vertical;
  }
  
  /* Footer (Source Link) */
  #${id} .card-footer {
    padding-top: 8px;
    border-top: 1px solid var(--background-modifier-border);
    font-size: 0.75em;
    color: var(--text-muted);
    display: flex;
    align-items: center;
  }
`;

// ============================================================
// RENDER LOGIC
// ============================================================
dv.container.createEl('style').textContent = css;
const grid = dv.container.createEl('div', { attr: { id: id } });

for (let t of tasks) {
    // Create Card
    const card = grid.createEl('div', { cls: 'card' });
    
    // Create Content Block
    const content = card.createEl('div', { cls: 'card-content' });
    dv.span(t.text, { container: content });
    
    // Create Footer Block
    const footer = card.createEl('div', { cls: 'card-footer' });
    // Add icon and filename
    footer.innerHTML = `<span style="margin-right:4px">📄</span> ${t.link.path.replace('.md', '')}`;

    // ============================================================
    // CLICK HANDLER
    // ============================================================
    card.addEventListener('click', async (e) => {
        // Handle Ctrl+Click / Cmd+Click for new tab
        const newTab = e.ctrlKey || e.metaKey;
        
        // Open the note
        await app.workspace.openLinkText(t.link.path, t.link.path, newTab);
    });
}
```
