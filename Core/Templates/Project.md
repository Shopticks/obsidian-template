<%*
let projectName = await tp.system.prompt("Введите название проекта", "Проект");
const folderPath = "00 - Projects/" + projectName;

try {
    await app.vault.createFolder(folderPath);
} catch (e) {
    if (!e.message.includes("already exists")) throw e;
}

const allImages = tp.app.vault.getFiles().filter(f => 
    ['jpg', 'jpeg', 'png', 'gif', 'webp'].includes(f.extension)
);
const bannerFile = await tp.system.suggester((f) => f.path, allImages);
const banner = bannerFile ? bannerFile.path : "";

const status = await tp.system.suggester(
    ["Active", "Planning", "Paused", "Completed", "Rejected"], 
    ["Active", "Planning", "Paused", "Completed", "Rejected"]
);

let progressNum = await tp.system.prompt("Прогресс (0-100)", "0");
let progress = Math.min(Math.max(parseInt(progressNum) || 0, 0), 100);

let daysOffset = await tp.system.prompt("Дедлайн (через сколько дней?)", "7");
const dueDate = tp.date.now("YYYY-MM-DD", parseInt(daysOffset) || 0);

const baseContent = `
filters:
  and:
    - file.folder == "${folderPath}"
    - type == ["Task"]
formulas:
  Sorted priority: if(priority == "High", 1, if(priority == "Medium", 2, if(priority == "Low", 3, 4)))
  Sorted status: if(status == "In-Progress", 1, if(status == "ToDo", 2, 3))
views:
  - type: table
    name: Table
    order: [file.name, priority, status, due_date]
    sort:
      - property: due_date
        direction: ASC
      - property: formula.Sorted priority
        direction: ASC
`;

const baseName = "Base Of Tasks for " + projectName + ".base";
await tp.app.vault.create(folderPath + "/" + baseName, baseContent);
await tp.file.move(folderPath + "/" + projectName);
%>---
banner: "<% banner %>"
type: Project
status:
  - <% status %>
start_date: <% tp.date.now("YYYY-MM-DD") %>
due_date: <% dueDate %>
progress: <% progress %>
tags: [project]
created: <% tp.date.now("YYYY-MM-DD") %>
---

# <% projectName %>

Прогресс: `$=dv.el("progress", "", {attr: {max: 100, value: dv.current().progress}, style: "width:150px;height:20px;vertical-align:middle;"})` `$= dv.current().progress + "%"`

> [!abstract] Сводка времени
> **Дедлайн:** <% dueDate %> (`$= (()=>{ const d = dv.current()?.due_date; if (!d) return "—"; const diff = dv.date(d).diff(dv.date('today'), 'days').days; return Math.round(diff); })()` дн.)
> **Статус:** #project/<% status %>

## 🎯 Цели и описание
- [ ] Главная цель проекта
- Краткое описание контекста или желаемого результата.

## 🛠 Ресурсы и ссылки
- **Материалы:** 
- **Полезные ссылки:** 

## 📋 Основные задачи
![[<% baseName %>]]

## 📂 Недавние файлы проекта
```dataview
list from "<% folderPath %>"
where file.name != this.file.name
sort file.mday desc
limit 5