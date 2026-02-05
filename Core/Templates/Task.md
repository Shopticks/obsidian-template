<%*
const taskName = await tp.system.prompt("Название задачи", "Новая задача");
const status = await tp.system.suggester(["ToDo", "In-Progress", "Done"], ["ToDo", "In-Progress", "Done"]);
const priority = await tp.system.suggester(["Low", "Medium", "High"], ["Low", "Medium", "High"]);
const daysOffset = await tp.system.prompt("Дедлайн (дней от сегодня)", "0");
const dueDate = tp.date.now("YYYY-MM-DD", parseInt(daysOffset) || 0);

const projectFolders = app.vault.getAbstractFileByPath("00 - Projects")?.children
    .filter(f => f instanceof tp.obsidian.TFolder)
    .map(f => f.name) || [];

let projectName = await tp.system.suggester(["Без проекта", "+ Создать новый...", ...projectFolders], ["none", "new", ...projectFolders]);

if (projectName === "new") {
    projectName = await tp.system.prompt("Введите название нового проекта");
}

let folderPath = "";
if (projectName && projectName !== "none") {
    folderPath = "00 - Projects/" + projectName;
    if (!(await app.vault.adapter.exists(folderPath))) {
        await app.vault.createFolder(folderPath);
    }
}

await tp.file.rename(taskName);
if (folderPath) {
    await tp.file.move(folderPath + "/" + taskName);
}

const content = `---
type: Task
${projectName && projectName !== "none" ? `project: "[[${projectName}]]"` : ""}
status:
  - ${status}
priority:
  - ${priority}
due_date: ${dueDate}
created: ${tp.date.now("YYYY-MM-DD")}
---

# ${taskName}

> [!info] Инфо
> **Проект:** ${projectName && projectName !== "none" ? `[[${projectName}]]` : "—"}
> **Приоритет:** ${priority}
> **Дедлайн:** ${dueDate}

## 🎯 Суть задачи
*Краткое описание того, что именно нужно сделать и зачем.*

## ✅ Критерии готовности (DoD)
- [ ] 

## 🛠 Действия
- [ ] 

## 📎 Ресурсы и материалы
- 

## 🏁 Результат
*Заметки по итогу выполнения.*
`;

tR += content;
%>