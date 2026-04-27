---
title: Home
created: 2026-04-19
modified: 2026-04-27
tags: dashboard
cssclasses:
icon: layout-dashboard
banner: "[[logowhitetransparent.png]]"
mode: read_only
---
```datacorejsx
return function Greeting() {
    const hour = new Date().getHours();

    let greeting = "Good evening";
    if (hour < 12) greeting = "Good morning";
    else if (hour < 18) greeting = "Good afternoon";

    return (
        <div style={{ textAlign: "center" }}>
            <div style={{ fontSize: "1.4rem", fontWeight: "700" }}>
                {greeting}
            </div>
        </div>
    );
}
```

```datacorejsx
return function Clock() {
    const time = new Date();
    const hours = time.getHours().toString().padStart(2, '0');
    const minutes = time.getMinutes().toString().padStart(2, '0');
    const dateStr = time.toLocaleDateString('en-US', { weekday: 'long', month: 'long', day: 'numeric', year: 'numeric' });

    return (
        <div style={{ textAlign: "center", marginBottom: "20px" }}>
            <div style={{ fontSize: "2.5rem", fontWeight: "900", fontFamily: "monospace", letterSpacing: "2px", color: "white" }}>
                <span style={{backgroundColor:"#222", padding:"4px 8px", borderRadius:"5px", border:"1px solid #444"}}>{hours}</span>
                <span style={{margin:"0 4px"}}>:</span>
                <span style={{backgroundColor:"#222", padding:"4px 8px", borderRadius:"5px", border:"1px solid #444"}}>{minutes}</span>
            </div>
            <div style={{ fontSize: "0.8rem", fontWeight: "700", marginTop: "10px", textTransform: "uppercase", letterSpacing: "1px", color: "#a0a0a0" }}>
                {dateStr}
            </div>
        </div>
    );
}
```

![[Recents notion.base]]

```dataviewjs
const DAYS_AHEAD = 14;
const SOURCES = ['"TaskNotes/Tasks"', '"School"'];
const PRIORITY_COLORS = { high: "pink", medium: "orange", low: "blue" };

const today = moment().startOf("day");
const end = moment(today).add(DAYS_AHEAD, "days").endOf("day");

function toMoment(v) {
  if (!v) return null;
  const m = moment(v.toJSDate ? v.toJSDate() : v);
  return m.isValid() ? m : null;
}

let items = [];
for (const src of SOURCES) {
  for (const p of dv.pages(src)) {
    const s = String(p.status ?? "").toLowerCase();
    if (s === "done" || s === "completed" || s === "cancelled") continue;
    const date = toMoment(p.due) || toMoment(p.scheduled) || toMoment(p.start);
    if (!date || !date.isSameOrBefore(end)) continue;
    const color = p.color ? String(p.color).replace(/"/g, "") : PRIORITY_COLORS[String(p.priority ?? "medium")] ?? "orange";
    const proj = p.projects ? String(p.projects) : p.class ? String(p.class).replace(/\[\[|\]\]/g, "") : "";
    items.push({ title: p.title ?? p.file.name, date, color, proj, path: p.file.path });
  }
}
items.sort((a, b) => a.date.valueOf() - b.date.valueOf());

const days = {};
for (const it of items) {
  const key = it.date.format("YYYY-MM-DD");
  if (!days[key]) days[key] = { date: it.date, items: [] };
  days[key].items.push(it);
}

const root = dv.el("div", "", { cls: "upcoming-events" });

root.innerHTML = `<div class="ue-header">
  <svg viewBox="0 0 24 24"><rect x="3" y="4" width="18" height="18" rx="2"></rect>
  <path d="M16 2v4"></path><path d="M8 2v4"></path><path d="M3 10h18"></path></svg>
  Upcoming events</div><div class="ue-body"></div>`;

const body = root.querySelector(".ue-body");
const dayKeys = Object.keys(days).sort();

if (!dayKeys.length) {
  body.innerHTML = '<div class="ue-empty">No upcoming events.</div>';
} else {
  for (const key of dayKeys) {
    const { date, items: dayItems } = days[key];
    const isToday = date.isSame(today, "day");
    const isOverdue = date.isBefore(today);
    const labelClass = isToday ? "ue-day-label-today" : isOverdue ? "ue-day-label-overdue" : "ue-day-label-future";
    const labelText = isToday ? "Today " + date.format("MMMM D") : isOverdue ? "Overdue " + date.format("MMMM D") : date.format("dddd MMMM D");

    const day = document.createElement("div");
    day.className = "ue-day";
    day.innerHTML = `<div class="ue-day-label ${labelClass}">${labelText}</div><div class="ue-day-events"></div>`;
    const eventsCol = day.querySelector(".ue-day-events");

    for (const it of dayItems) {
      const ev = document.createElement("div");
      ev.className = "ue-event";
      ev.dataset.color = it.color;
      ev.innerHTML = `<div class="ue-event-top"><div class="ue-dot"></div><div class="ue-event-title">${it.title}</div></div>` +
        (it.proj ? `<div class="ue-event-meta"><span class="ue-tag">${it.proj}</span></div>` : "");
      ev.onclick = () => app.workspace.openLinkText(it.path, "", false);
      eventsCol.appendChild(ev);
    }
    body.appendChild(day);
  }
}
```

```dataviewjs
const DAYS_AHEAD = 30;
const SOURCE = '"TaskNotes/Tasks"';

const today = moment().startOf("day");
const end = moment(today).add(DAYS_AHEAD, "days").endOf("day");

function toMoment(value) {
  if (!value) return null;
  const raw = value.toJSDate ? value.toJSDate() : value;
  const m = moment(raw);
  return m.isValid() ? m : null;
}

function pageDate(p) {
  return toMoment(p.due) || toMoment(p.scheduled) || toMoment(p.start);
}

function pageArea(p) {
  const tags = p.file.tags ?? [];
  if (tags.includes("#work")) return "Work";
  if (tags.includes("#personal")) return "Personal";
  if (p.category) return String(p.category);
  if (p.area) return String(p.area);
  return "Inbox";
}

const pages = dv.pages(SOURCE)
  .where(p => p.status !== "done" && p.status !== "completed" && p.status !== "cancelled")
  .map(p => ({ page: p, date: pageDate(p) }))
  .where(x => x.date && x.date.isSameOrBefore(end))
  .sort(x => x.date.valueOf());

const root = dv.el("div", "", { cls: "notion-task-widget" });

root.innerHTML = `
  <div class="nt-left">
    <div class="nt-icon" aria-hidden="true">
      <svg viewBox="0 0 24 24" class="nt-lucide-icon">
        <rect x="3" y="5" width="6" height="6" rx="1.5"></rect>
        <path d="M3 17l2 2 4-4"></path>
        <path d="M13 6h8"></path>
        <path d="M13 12h8"></path>
        <path d="M13 18h8"></path>
      </svg>
    </div>
    <div class="nt-empty-title">See all your tasks across your vault in one place.</div>
    <div class="nt-link" style="cursor:pointer">Open task folder</div>
  </div>
  <div class="nt-right"></div>
`;

const TASK_FOLDER = "TaskNotes/Tasks";

const linkEl = root.querySelector(".nt-link");
linkEl.onclick = () => app.workspace.openLinkText(TASK_FOLDER, "", false);

const right = root.querySelector(".nt-right");

if (!pages.length) {
  right.innerHTML = `<div class="nt-none">No upcoming tasks.</div>`;
} else {
  for (const { page, date } of pages) {
    const row = document.createElement("div");
    row.className = "nt-task-row";
    row.style.cursor = "pointer";

    const text = page.title ?? page.file.name;
    const area = pageArea(page);
    const isOverdue = date.isBefore(today);
    const dateText = (isOverdue ? "Overdue \u00b7 " : "") + date.format("MMM D, YYYY");

    row.innerHTML = `
      <div class="nt-checkbox"></div>
      <div class="nt-task-title">${text}</div>
      <div class="nt-task-date" ${isOverdue ? 'style="color:#e74c3c"' : ''}>${dateText}</div>
      <div class="nt-task-area">${area}</div>
    `;

    row.onclick = () => app.workspace.openLinkText(page.file.path, "", false);
    right.appendChild(row);
  }

  const add = document.createElement("div");
  add.className = "nt-new-task";
  add.style.cursor = "pointer";
  add.innerHTML = `<span class="nt-plus">+</span><span>New task</span>`;
  add.onclick = () => {
    const overlay = document.createElement("div");
    overlay.className = "nt-modal-overlay";
    const modal = document.createElement("div");
    modal.className = "nt-modal";
    modal.innerHTML = `
      <div class="nt-modal-title">New Task</div>
      <label class="nt-modal-field"><div class="nt-modal-label">Title</div>
        <input id="nt-title" class="nt-modal-input" type="text" placeholder="Task name"></label>
      <div class="nt-modal-row">
        <label><div class="nt-modal-label">Priority</div>
          <select id="nt-priority" class="nt-modal-select">
            <option value="low">Low</option><option value="medium" selected>Medium</option><option value="high">High</option></select></label>
        <label><div class="nt-modal-label">Due date</div>
          <input id="nt-due" class="nt-modal-input" type="date" value="${moment().add(1,'days').format('YYYY-MM-DD')}"></label>
      </div>
      <div class="nt-modal-row">
        <label><div class="nt-modal-label">Project</div>
          <input id="nt-project" class="nt-modal-input" type="text" placeholder="Optional"></label>
        <label><div class="nt-modal-label">Context</div>
          <input id="nt-context" class="nt-modal-input" type="text" value="computer"></label>
      </div>
      <div class="nt-modal-actions">
        <button id="nt-cancel" class="nt-modal-btn">Cancel</button>
        <button id="nt-create" class="nt-modal-btn-primary">Create</button>
      </div>`;
    overlay.appendChild(modal);
    document.body.appendChild(overlay);
    const titleInput = modal.querySelector("#nt-title");
    titleInput.focus();
    overlay.onclick = (e) => { if (e.target === overlay) overlay.remove(); };
    modal.querySelector("#nt-cancel").onclick = () => overlay.remove();
    const create = () => {
      const title = titleInput.value.trim() || "Untitled task " + moment().format("YYYY-MM-DD-HHmmss");
      const priority = modal.querySelector("#nt-priority").value;
      const due = modal.querySelector("#nt-due").value || moment().add(1,"days").format("YYYY-MM-DD");
      const project = modal.querySelector("#nt-project").value.trim();
      const context = modal.querySelector("#nt-context").value.trim() || "computer";
      const todayStr = moment().format("YYYY-MM-DD");
      const projects = project ? "[" + project + "]" : "[]";
      const content = [
        "---","title: " + title,"created: " + todayStr,"modified: " + todayStr,
        "tags: [task]","status: open","priority: " + priority,"due: " + due,
        "scheduled: " + todayStr,"projects: " + projects,"contexts: [" + context + "]",
        "recurrence: null","complete_instances: []","---","# " + title,""
      ].join("\n");
      overlay.remove();
      app.vault.create(TASK_FOLDER + "/" + title + ".md", content)
        .then(f => app.workspace.openLinkText(f.path, "", false));
    };
    modal.querySelector("#nt-create").onclick = create;
    titleInput.addEventListener("keydown", (e) => { if (e.key === "Enter") create(); });
  };
  right.appendChild(add);
}
```

![[Notionclasses.base]]

![[Projects2Base.base]]