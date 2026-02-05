<%*
const taskName = await tp.system.prompt("Название задачи:", "Новая задача");
await tp.file.rename(taskName);

const status = await tp.system.suggester(["ToDo", "In-Progress", "Done"], ["ToDo", "In-Progress", "Done"]);
const priority = await tp.system.suggester(["Low", "Medium", "High"], ["Low", "Medium", "High"]);
const daysOffset = await tp.system.prompt("Дедлайн: через сколько дней, начиная от сегодняшнего? (0 = сегодня)", "0");
const dueDate = tp.date.now("YYYY-MM-DD", parseInt(daysOffset) || 0);
const projectName = await tp.system.prompt("Название проекта (оставьте пустым, если без проекта):", "");

let targetPath = null;

if (projectName && projectName.trim() !== "") {
    const cleanProjectName = projectName.trim();
    const folderPath = "00 - Projects/" + cleanProjectName;
    
    try {
        await app.vault.createFolder(folderPath);
    } catch (e) {
        if (!e.message.includes("already exists")) throw e;
    }
    
    targetPath = folderPath + "/" + taskName;
}

// Формируем контент с переносами строк и отступами для списка
const content = `---
type: Task${projectName && projectName.trim() !== "" ? `\nproject: ${projectName.trim()}` : ""}
status:
  - ${status}
priority:
  - ${priority}
due_date: ${dueDate}
created: ${tp.date.now("YYYY-MM-DD")}
---
## 🎯 Цель

Описание твоей цели.

## 🛠️ Действия к решению

Что нужно сделать?

## ✅ Решение / Итог

Как достичь решения задачи?

## 📌 Связи
- Проекты:${projectName && projectName.trim() !== "" ? `\n  - [[${projectName.trim()}]]` : "\n  - Проект 1"}
- Заметки:
  - Заметка 1
`;

if (targetPath) {
    await tp.file.move(targetPath);
}

tR += content;
%>