---
modified: 2025-12-23
cssclasses:
  - style-wide
banner: "[[Untitled.png]]"
banner_position: "43"
mode: read_only
---
<h1><center> Good Afternoon</center></h1>

![[notionrec.base]]
![[Classes.base]]
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
![[Projects Base.base]]
![[Projects Read later.base]]
