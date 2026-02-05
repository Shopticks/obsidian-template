<%*
let projectName = await tp.system.prompt("Введите название проекта: ", "Проект");

const folderPath = "00 - Projects/" + projectName;

try {
    await app.vault.createFolder(folderPath);
} catch (e) {
    if (!e.message.includes("already exists")) throw e;
}

await tp.file.move(folderPath + "/" + projectName);

<!-- ADDING BANNER -->

function getAllImagesHelper(files) { return files.filter(file => { if (file.name) { const ext = file.name.split('.').pop().toLowerCase(); return ['jpg', 'jpeg', 'png', 'gif', 'bmp', 'webp', 'tiff'].includes(ext); } return false; }); }

const files_path = tp.app.vault.getFiles();
const allImages = getAllImagesHelper(files_path);

const banner = await tp.system.suggester(allImages, allImages);

<!-- GET YAML PROPERTIES -->

const status = await tp.system.suggester(["Active", "Planning", "Paused", "Completed", "Rejected"], ["Active", "Planning", "Paused", "Completed", "Rejected"]);

let progressNum = await tp.system.prompt("Введите прогресс проекта (0-100)", "0");

let progress = parseFloat(progressNum);
if (isNaN(progress) || progress < 0 || progress > 100) { progress = 0; } 
else { progress = Math.round(progress); }

let daysOffset = await tp.system.prompt("Дедлайн: через сколько дней, начиная от сегодняшнего? (0 = сегодня)", "0");
const dueDate = tp.date.now("YYYY-MM-DD", parseInt(daysOffset) || 0);

<!-- PROJECT BASE FOR TASKS -->
const content = `
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
    order:
      - file.name
      - priority
      - status
      - due_date
    sort:
      - property: due_date
        direction: ASC
      - property: formula.Sorted priority
        direction: ASC
      - property: formula.Sorted status
        direction: ASC
      - property: file.name
        direction: ASC
`;
const baseName = "Base Of Tasks for " + projectName + ".base";
await tp.app.vault.create(folderPath + "/" + baseName, content);
%>---
banner: "<% banner %>"
type: Project
status:
  - <% status %>
# planning | active | paused | completed | rejected
start_date: <% tp.date.now("YYYY-MM-DD") %>
due_date: <% dueDate %>
progress: <% progress %>
tags: [project]
created: <% tp.date.now("YYYY-MM-DD") %>
---

# <% projectName %>
Прогресс: `$=dv.el("progress", "", {attr: {max: 100, value: dv.current().progress}, style: "width:150px;height:20px;vertical-align:middle;"})` `$= dv.current().progress + "%"`


## 🎯 Цель проекта

- [ ] цель 1

## 📋 Основные задачи
![[<% baseName %>]]