---
cssclasses:
  - wide
  - max
  - button-grid-square
  - button-grid
banner: https://i.ibb.co/BK6dNQRK/13ce4b01ae7e.png
modified: 2026-04-27
banner_position:
mode: read_only
title: newnotetemp
created: 2025-07-31
---

````columns
id: OFOq2VparEzlBKfybDW6D
===

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
===
# Homepage
---
````

```columns
id: JI9AU_tArCt_v6x9MF16M
===
![[Misc/Untitled 5.base]]
===

## Quick Launch

> [!button-grid] Actions
> ```button
> name New Note
> type command
> action Templater: Insert newnotetemp
> class make-it-wider btn-3
> ```
> ```button
> name New Project
> type command
> action Templater: Insert newprojecttemp
> class make-it-wider btn-4
> ```
> ```button
> name Daily note
> type command
> action Templater: Insert Interactive Daily Note
> class make-it-wider btn-2

```datacorejsx
return function TightTaskDashboard() {
    const allItems = dc.useQuery("@task");
    const [tab, setTab] = dc.useState("Open");
    const [search, setSearch] = dc.useState("");
    const [pathFilter, setPathFilter] = dc.useState("");
    const [showSubtasks, setShowSubtasks] = dc.useState(true);
    const [sortMode, setSortMode] = dc.useState("due");
    const [busyKey, setBusyKey] = dc.useState("");
    const [refreshTick, setRefreshTick] = dc.useState(0);

    function toDateKey(value) {
        if (!value) return null;
        try {
            if (typeof value === "string") return value.slice(0, 10);
            if (value && typeof value.toISODate === "function") return value.toISODate();
            if (value instanceof Date) return value.toISOString().slice(0, 10);
            if (value && value.ts) return new Date(value.ts).toISOString().slice(0, 10);
            return String(value).slice(0, 10);
        } catch {
            return null;
        }
    }

    const todayKey = toDateKey(new Date());

    function lower(value) {
        return String(value ?? "").toLowerCase();
    }

    function getRawTaskText(task) {
        return String(task?.text ?? task?.$text ?? task?.$name ?? task?.name ?? "");
    }

    function getTaskText(task) {
        let text = getRawTaskText(task);
        text = text.replace(/^\[[^\]]+\]\s*/, "").trim();
        text = text.replace(/(^|\s)#task\b/gi, " ").replace(/\s+/g, " ").trim();
        text = text.replace(/^~~(.*)~~$/, "$1").trim();
        return text;
    }

    function getTaskPath(task) {
        return String(task?.$path ?? task?.path ?? task?.file?.path ?? task?.$file ?? "");
    }

    function getTaskStatusObject(task) {
        return task?.status ?? task?.$status ?? null;
    }

    function getTaskStatusName(task) {
        const s = getTaskStatusObject(task);
        return lower(s?.name ?? s?.label ?? s?.text ?? s?.type ?? s ?? "");
    }

    function getTaskStatusSymbol(task) {
        const s = getTaskStatusObject(task);
        return String(s?.symbol ?? s?.marker ?? s?.raw ?? "");
    }

    function getDue(task) {
        return toDateKey(task?.$due ?? task?.due);
    }

    function getScheduled(task) {
        return toDateKey(task?.$scheduled ?? task?.scheduled);
    }

    function getPriority(task) {
        return String(task?.priority ?? task?.$priority ?? "");
    }

    function hasTaskTag(task) {
        return /(^|\s)#task\b/i.test(getRawTaskText(task));
    }

    function isCancelled(task) {
        const raw = getRawTaskText(task).trim();
        return /^~~.*~~$/.test(raw);
    }

    function isDone(task) {
        return !!(
            task?.$completed === true ||
            task?.completed === true ||
            task?.checked === true ||
            task?.$checked === true ||
            task?.completion ||
            getTaskStatusName(task).includes("done") ||
            getTaskStatusName(task).includes("complete") ||
            ["x", "X", "c", "C"].includes(getTaskStatusSymbol(task))
        );
    }

    function isOverdue(task) {
        if (isDone(task)) return false;
        const due = getDue(task);
        return !!(due && due < todayKey);
    }

    function isToday(task) {
        if (isDone(task)) return false;
        const due = getDue(task);
        const scheduled = getScheduled(task);
        return due === todayKey || scheduled === todayKey;
    }

    function isWaiting(task) {
        const status = getTaskStatusName(task);
        const raw = lower(getRawTaskText(task));
        return (
            status.includes("waiting") ||
            status.includes("hold") ||
            raw.includes("#waiting") ||
            raw.includes("#hold")
        );
    }

    function isInProgress(task) {
        if (isDone(task)) return false;
        const status = getTaskStatusName(task);
        const symbol = getTaskStatusSymbol(task);
        const raw = lower(getRawTaskText(task));

        return (
            status.includes("progress") ||
            status.includes("doing") ||
            status.includes("active") ||
            symbol === "/" ||
            symbol === ">" ||
            raw.includes("#doing") ||
            raw.includes("#in-progress") ||
            raw.includes("#active")
        );
    }

    function isOpen(task) {
        return !isDone(task) && !isWaiting(task) && !isInProgress(task);
    }

    const tasksOnly = allItems
        .filter(hasTaskTag)
        .filter(task => !isCancelled(task));

    function matchesTab(task) {
        switch (tab) {
            case "All": return true;
            case "Open": return isOpen(task);
            case "Doing": return isInProgress(task);
            case "Waiting": return isWaiting(task);
            case "Today": return isToday(task);
            case "Overdue": return isOverdue(task);
            case "Done": return isDone(task);
            default: return true;
        }
    }

    function matchesSearch(task) {
        if (!search.trim()) return true;
        const haystack = [
            getTaskText(task),
            getTaskPath(task),
            task?.$tags?.join?.(" ") ?? "",
            getTaskStatusName(task)
        ].join(" ").toLowerCase();

        return haystack.includes(search.trim().toLowerCase());
    }

    function matchesPath(task) {
        if (!pathFilter.trim()) return true;
        return getTaskPath(task).toLowerCase().includes(pathFilter.trim().toLowerCase());
    }

    function parentDepth(task) {
        return Number(task?.parentCount ?? task?.$parentCount ?? task?.depth ?? 0);
    }

    function matchesSubtaskMode(task) {
        if (showSubtasks) return true;
        return parentDepth(task) === 0;
    }

    function dateRank(task) {
        return getDue(task) ?? getScheduled(task) ?? "9999-12-31";
    }

    function alphaRank(task) {
        return getTaskText(task).toLowerCase();
    }

    function statusRank(task) {
        if (isOverdue(task)) return 0;
        if (isToday(task)) return 1;
        if (isInProgress(task)) return 2;
        if (isWaiting(task)) return 3;
        if (isOpen(task)) return 4;
        if (isDone(task)) return 5;
        return 6;
    }

    const visibleTasks = tasksOnly
        .filter(matchesTab)
        .filter(matchesSearch)
        .filter(matchesPath)
        .filter(matchesSubtaskMode)
        .sort((a, b) => {
            if (sortMode === "alpha") return alphaRank(a).localeCompare(alphaRank(b));
            if (sortMode === "status") {
                const s = statusRank(a) - statusRank(b);
                if (s !== 0) return s;
                return dateRank(a).localeCompare(dateRank(b));
            }
            const d = dateRank(a).localeCompare(dateRank(b));
            if (d !== 0) return d;
            return alphaRank(a).localeCompare(alphaRank(b));
        });

    const counts = {
        All: tasksOnly.length,
        Open: tasksOnly.filter(isOpen).length,
        Doing: tasksOnly.filter(isInProgress).length,
        Waiting: tasksOnly.filter(isWaiting).length,
        Today: tasksOnly.filter(isToday).length,
        Overdue: tasksOnly.filter(isOverdue).length,
        Done: tasksOnly.filter(isDone).length,
    };

    const tabs = ["All", "Open", "Doing", "Waiting", "Today", "Overdue", "Done"];

    function taskKey(task) {
        return getTaskPath(task) + "::" + getRawTaskText(task);
    }

    function escapeRegExp(str) {
        return String(str).replace(/[.*+?^${}()|[\]\\]/g, "\\$&");
    }

    async function toggleTask(task) {
        const path = getTaskPath(task);
        const rawText = getRawTaskText(task).trim();
        if (!path || !rawText) return;

        const key = taskKey(task);
        setBusyKey(key);

        try {
            const file = app.vault.getAbstractFileByPath(path);
            if (!file) throw new Error("File not found: " + path);

            const content = await app.vault.read(file);
            const lines = content.split("\n");

            const textPattern = escapeRegExp(rawText);
            const unchecked = new RegExp("^(\\s*[-*+]\\s+\\[ \\]\\s+.*" + textPattern + ".*)$");
            const checked = new RegExp("^(\\s*[-*+]\\s+\\[[xX]\\]\\s+.*" + textPattern + ".*)$");

            let changed = false;

            for (let i = 0; i < lines.length; i++) {
                if (!changed && unchecked.test(lines[i])) {
                    lines[i] = lines[i].replace(/^(\s*[-*+]\s+\[)\s(\]\s+)/, "$1x$2");
                    changed = true;
                    break;
                }
                if (!changed && checked.test(lines[i])) {
                    lines[i] = lines[i].replace(/^(\s*[-*+]\s+\[)[xX](\]\s+)/, "$1 $2");
                    changed = true;
                    break;
                }
            }

            if (changed) {
                await app.vault.modify(file, lines.join("\n"));
                setRefreshTick(v => v + 1);
            } else {
                new Notice("Could not find matching task line to toggle.");
            }
        } catch (err) {
            console.error(err);
            new Notice("Task toggle failed.");
        } finally {
            setBusyKey("");
        }
    }

    async function openOriginalNote(task) {
        const path = getTaskPath(task);
        if (!path) return;

        try {
            await app.workspace.openLinkText(path, "", false);
        } catch (err) {
            console.error(err);
            new Notice("Could not open original note.");
        }
    }

    function pillStyle(active) {
        return {
            padding: "4px 10px",
            borderRadius: "999px",
            border: "1px solid var(--background-modifier-border)",
            background: active ? "var(--interactive-accent)" : "var(--background-secondary)",
            color: active ? "var(--text-on-accent)" : "var(--text-normal)",
            fontSize: "11px",
            fontWeight: "600",
            cursor: "pointer",
            lineHeight: "1.2"
        };
    }

    function metaChipStyle() {
        return {
            padding: "2px 7px",
            borderRadius: "999px",
            border: "1px solid var(--background-modifier-border)",
            background: "var(--background-secondary)",
            color: "var(--text-muted)",
            fontSize: "10px",
            lineHeight: "1.25"
        };
    }

    function cardTone(task) {
        if (isOverdue(task)) return "rgba(255, 90, 90, 0.10)";
        if (isToday(task)) return "rgba(255, 180, 0, 0.08)";
        if (isInProgress(task)) return "rgba(90, 140, 255, 0.08)";
        if (isWaiting(task)) return "rgba(150, 150, 150, 0.07)";
        if (isDone(task)) return "rgba(90, 200, 120, 0.08)";
        return "var(--background-primary)";
    }

    function statusLabel(task) {
        if (isDone(task)) return "Done";
        if (isOverdue(task)) return "Overdue";
        if (isToday(task)) return "Today";
        if (isInProgress(task)) return "Doing";
        if (isWaiting(task)) return "Waiting";
        return "Open";
    }

    function truncatePath(path) {
        if (!path) return "";
        if (path.length <= 46) return path;
        return "…" + path.slice(-45);
    }

    refreshTick;

    return (
        <div style={{ display: "grid", gap: "8px" }}>
            <div
                style={{
                    border: "1px solid var(--background-modifier-border)",
                    borderRadius: "14px",
                    padding: "8px",
                    background: "var(--background-primary)",
                    display: "grid",
                    gap: "8px"
                }}
            >
                <div style={{ fontWeight: "700", fontSize: "15px" }}>Tasks</div>

                <div style={{ display: "flex", flexWrap: "wrap", gap: "6px" }}>
                    {tabs.map(name => (
                        <button key={name} onClick={() => setTab(name)} style={pillStyle(tab === name)}>
                            {name} {counts[name] ?? 0}
                        </button>
                    ))}
                </div>

                <div
                    style={{
                        display: "grid",
                        gridTemplateColumns: "minmax(0,1.2fr) minmax(0,1fr) auto auto",
                        gap: "6px",
                        alignItems: "center"
                    }}
                >
                    <input
                        type="text"
                        value={search}
                        placeholder="Search tasks"
                        onInput={(e) => setSearch(e.target.value)}
                        style={{
                            padding: "6px 8px",
                            borderRadius: "8px",
                            border: "1px solid var(--background-modifier-border)",
                            background: "var(--background-primary-alt)",
                            color: "var(--text-normal)",
                            fontSize: "12px",
                            minWidth: 0
                        }}
                    />

                    <input
                        type="text"
                        value={pathFilter}
                        placeholder="Filter folder"
                        onInput={(e) => setPathFilter(e.target.value)}
                        style={{
                            padding: "6px 8px",
                            borderRadius: "8px",
                            border: "1px solid var(--background-modifier-border)",
                            background: "var(--background-primary-alt)",
                            color: "var(--text-normal)",
                            fontSize: "12px",
                            minWidth: 0
                        }}
                    />

                    <select
                        value={sortMode}
                        onChange={(e) => setSortMode(e.target.value)}
                        style={{
                            padding: "6px 8px",
                            borderRadius: "8px",
                            border: "1px solid var(--background-modifier-border)",
                            background: "var(--background-primary-alt)",
                            color: "var(--text-normal)",
                            fontSize: "12px"
                        }}
                    >
                        <option value="due">Due</option>
                        <option value="status">Status</option>
                        <option value="alpha">A-Z</option>
                    </select>

                    <label
                        style={{
                            display: "flex",
                            alignItems: "center",
                            gap: "5px",
                            padding: "6px 8px",
                            borderRadius: "8px",
                            border: "1px solid var(--background-modifier-border)",
                            background: "var(--background-primary-alt)",
                            fontSize: "12px",
                            whiteSpace: "nowrap"
                        }}
                    >
                        <input
                            type="checkbox"
                            checked={showSubtasks}
                            onChange={(e) => setShowSubtasks(e.target.checked)}
                        />
                        Subtasks
                    </label>
                </div>
            </div>

            <div style={{ display: "grid", gap: "6px" }}>
                {visibleTasks.map(task => {
                    const key = taskKey(task);
                    const due = getDue(task);
                    const scheduled = getScheduled(task);
                    const path = getTaskPath(task);

                    return (
                        <div
                            key={key}
                            style={{
                                display: "flex",
                                alignItems: "flex-start",
                                gap: "10px",
                                border: "1px solid var(--background-modifier-border)",
                                borderRadius: "12px",
                                padding: "8px 10px",
                                background: cardTone(task),
                                minWidth: 0
                            }}
                        >
                            <input
                                type="checkbox"
                                checked={isDone(task)}
                                disabled={busyKey === key}
                                onChange={() => toggleTask(task)}
                                style={{
                                    marginTop: "2px",
                                    flex: "0 0 auto"
                                }}
                            />

                            <div style={{ minWidth: 0, flex: 1, display: "grid", gap: "5px" }}>
                                <div
                                    style={{
                                        fontSize: "13px",
                                        lineHeight: "1.35",
                                        wordBreak: "break-word",
                                        textDecoration: isDone(task) ? "line-through" : "none",
                                        opacity: isDone(task) ? 0.72 : 1
                                    }}
                                >
                                    {getTaskText(task)}
                                </div>

                                <div
                                    style={{
                                        display: "flex",
                                        flexWrap: "wrap",
                                        gap: "4px",
                                        alignItems: "center"
                                    }}
                                >
                                    <span style={metaChipStyle()}>{statusLabel(task)}</span>
                                    {due ? <span style={metaChipStyle()}>Due {due}</span> : null}
                                    {!due && scheduled ? <span style={metaChipStyle()}>Sched {scheduled}</span> : null}
                                    {getPriority(task) ? <span style={metaChipStyle()}>{getPriority(task)}</span> : null}
                                    {path ? (
                                        <span
                                            title={path}
                                            style={{
                                                ...metaChipStyle(),
                                                maxWidth: "320px",
                                                overflow: "hidden",
                                                textOverflow: "ellipsis",
                                                whiteSpace: "nowrap"
                                            }}
                                        >
                                            {truncatePath(path)}
                                        </span>
                                    ) : null}
                                </div>
                            </div>

                            <button
                                onClick={() => openOriginalNote(task)}
                                style={{
                                    padding: "5px 8px",
                                    borderRadius: "8px",
                                    border: "1px solid var(--background-modifier-border)",
                                    background: "var(--background-primary-alt)",
                                    color: "var(--text-muted)",
                                    fontSize: "11px",
                                    whiteSpace: "nowrap",
                                    cursor: "pointer",
                                    flex: "0 0 auto"
                                }}
                            >
                                ↗ Note
                            </button>
                        </div>
                    );
                })}
            </div>
        </div>
    );
}
```

````columns
id: dispatcher-mid

## >> G_ORACLE

<div class="g-oracle-box">
  <div class="highlight">>> The Bremen Musicians settled before their destination.</div>
  ブレーメンの音楽隊は、目的地に辿り着く前に定住を選んだ。<br><br>
  System Log:<br>
  Teams that forget the KPI rot in the comfort of "status quo."<br>
  当初のKPIを忘れ、居心地の良い現状維持に逃げ込むチームの末路。
</div>

===
![[Articles.base]]
===
Whatever content you want
````

![[Misc/Untitled 6.base]]

