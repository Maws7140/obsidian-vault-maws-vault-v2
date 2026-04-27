---
foldernote: true
modified: 2026-04-27
banner: "[[school ban1.png]]"
cover: "[[school cov1.png]]"
mode: read_only
title: Time for skoo
created: 2025-10-14
cssclasses:
  - bases-max
  - style-wide 
"":
---
```search-bar
only search bar
```
> [!multi-column|no-wrap]
>
>> [!blank-container|wide-0 min-0]+
>> ```tasks
>> folder includes Time for skoo
>> has created date
>> not done
>> sort by urgency reverse
>>sort by priority
>> ```
>> ```button
>> 
>> name Math 230 schedule
>> type link
>> action obsidian://open?vault=Vault&file=Time%20for%20skoo%2FMath%20230%2FSchedule
>> class pill school
>> ```
>> ```button
>> name Math 230
>> type command
>> action Templater: Insert math230temp
>> templater true
>> class pill work
>> ^button-s3b7
>> ```
>> ```button
>> name Stat 380
>> type command
>> action Templater: Insert Stat380temp
>> templater true
>>class pill expenses
>> ^button-7xg2
>> ```
>> ```button
>> name Stat 415
>> type command
>> action Templater: Insert stat415temp
>> templater true
>> class pill school
>> ^button-kcgd
>> ```
>> ```button
>> name Stat 184
>> type command
>> action Templater: Insert Stat184temp
>> templater true
>> class pill work
>> ^button-3h4c
>> ```
>> ```button
>> name DS 410
>> type command
>> action Templater: Insert Ds410temp
>> templater true
>> class pill expenses
>> ^button-tu8b
>> ```
>> ```button
>> name Cmpsc 461
>> type command
>> action Templater: Insert cmpsc461temp
>> templater true
>> class pill school
>> ^button-wzdr
>> ```
>
>> [!blank-container|wide-4]+ Resources
>> ![[Classes.base]]
>> ```dataviewjs
>> // === edit me ===
>> const FOLDER = "School/Daily";   // where your daily notes live
>> const KEY    = "study_mins";            // frontmatter field for total minutes
>> const SCALE  = 180;                     // max minutes for darkest color (tweak)
>> // =================
>> function minutes(p){
>>   // Support either a single total, or an array like:
>>   // studies:
>>   //   - course: Math 230
>>   //     mins: 45
>>   if (Array.isArray(p.studies)) {
>>     return p.studies.reduce((s,x)=> s + Number(x.mins||x.minutes||0), 0);
>>   }
>>   return Number(p[KEY] ?? 0);
>> }
>> const pages = dv.pages(`"${FOLDER}"`)
>>   .where(p => p.file.name?.match(/^\d{4}-\d{2}-\d{2}$/));
>> const trackerData = {
>>   entries: [],
>>   heatmapTitle: "Study Tracker",
>>   heatmapSubtitle: "Lock in chart",
>>   separateMonths: false,
>>   basePath: FOLDER,         // click empty day → create note in this folder
>>   intensityScaleStart: 0,   // optional scale for color mapping
>>   intensityScaleEnd: SCALE
>> };
>> for (const p of pages) {
>>   const m = minutes(p);
>>   if (m > 0) {
>>     trackerData.entries.push({
>>       date: p.file.name,   // relies on YYYY-MM-DD.md
>>       filePath: p.file.path,
>>       intensity: m
>>     });
>>   }
>> }
>> renderHeatmapTracker(this.container, trackerData);
>> ```
>> 
>
>> [!blank-container|wide-2 min-0]+ To-Do
>> ```cards-name
>> @card [color-blue]
>> name: Maws Vault v1
>> tags: Student, Cloud engineer, Data scientist  
>> remark: Vault template
>> ```
>> ![[school 2.base]]

![[Dailystudy.base]]
