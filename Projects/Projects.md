---
foldernote: true
modified: 2026-04-27
banner: "[[proj ban1.png]]"
cover: "[[proj cov1.png]]"
mode: read_only
created: 2025-08-08
title: ${logTitle}
cssclasses:
  - bases-max
  - table-max
Status:
  - In progress
---

```search-bar
only search bar
```

````columns
id: _VSEa66zeRkueC_R8swp6
===
```dataviewjs
/*
NEO TOKYO CLOCK
DataviewJS version

HOW TO EDIT LATER
- Change locale in toLocaleDateString if you want a different style
- Change hour12 to true if you want 12-hour time
- Edit getTimeMode() if you want different day-part ranges
*/

const host = dv.el("div", "", { cls: "ntd-clock-card" });
const content = host.createDiv({ cls: "ntd-clock-content" });

const badge = host.createDiv({ cls: "ntd-clock-badge" });
const timeEl = content.createDiv({ cls: "ntd-clock-time" });
const dateEl = content.createDiv({ cls: "ntd-clock-date" });
const subtleEl = content.createDiv({ cls: "ntd-clock-subtle" });

function getTimeMode(hour) {
  if (hour >= 5 && hour < 8) {
    return { mode: "dawn", icon: "☀", label: "Dawn" };
  }
  if (hour >= 8 && hour < 18) {
    return { mode: "day", icon: "☀", label: "Day" };
  }
  if (hour >= 18 && hour < 21) {
    return { mode: "dusk", icon: "☾", label: "Dusk" };
  }
  return { mode: "night", icon: "☾", label: "Night" };
}

function renderClock() {
  const now = new Date();
  const hour = now.getHours();
  const meta = getTimeMode(hour);

  timeEl.textContent = now.toLocaleTimeString([], {
    hour: "2-digit",
    minute: "2-digit",
    hour12: false
  });

  dateEl.textContent = now.toLocaleDateString("en-US", {
    weekday: "long",
    month: "long",
    day: "numeric",
    year: "numeric"
  });

  subtleEl.textContent = meta.label;

  badge.textContent = meta.icon;
  badge.dataset.mode = meta.mode;
  badge.setAttribute("aria-label", meta.label);
  badge.setAttribute("title", meta.label);
}

renderClock();

const interval = window.setInterval(renderClock, 1000);

/*
Clean up the interval if the block gets removed
*/
const observer = new MutationObserver(() => {
  if (!document.body.contains(host)) {
    clearInterval(interval);
    observer.disconnect();
  }
});

observer.observe(document.body, { childList: true, subtree: true });
```
===
![[Projects2Base 1 1.base]]
````
````columns
id: fvvxwN3mXRgwc21tL_uAw
===
```tasks
not done
folder includes Projects
has created date
sort by urgency reverse
sort by priority
```
```button
name New Project
type command
action Templater: Insert newprojecttemp
templater true
class make-it-wider
^button-5d9m
```
````
````columns
id: MDoGn4OWOF1_OI2u5IQPD
===
![[Projects2Base 1.base]]
````





