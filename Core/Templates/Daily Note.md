<%*
<!-- Настройки -->
const dailyDirName = "04 - Daily";
const currentDate = tp.date.now("YYYY-MM-DD");
const currentFileName = currentDate + ".md";

const targetPath = `${dailyDirName}/${currentDate}`;
if (tp.file.path(true) !== targetPath + ".md") {
    await tp.file.move(targetPath);
}

const allFiles = app.vault.getMarkdownFiles();
const dailyFiles = allFiles.filter(file => {
    const isInFolder = file.path.startsWith(dailyDirName);
    const isDateName = /^\d{4}-\d{2}-\d{2}\.md$/.test(file.name);
    const isNotCurrent = file.name !== currentFileName;
    return isInFolder && isDateName && isNotCurrent;
});

dailyFiles.sort((a, b) => b.basename.localeCompare(a.basename));

let prevLinkStr = "..."; 
let nextLinkStr = "ожидание"; 

if (dailyFiles.length > 0) {
    const latestPrevFile = dailyFiles[0]; 
    
    prevLinkStr = `[[${latestPrevFile.basename}|прошлый]]`;

    try {
        const content = await app.vault.read(latestPrevFile);
        
        const regex = /(\|\|\s*)(ожидание)(\s*>)/;
        
        if (regex.test(content)) {
            const newContent = content.replace(regex, `$1[[${currentDate}|следующий]]$3`);
            
            if (content !== newContent) {
                await app.vault.modify(latestPrevFile, newContent);
                console.log(`Обновлена навигация в файле: ${latestPrevFile.basename}`);
            }
        }
    } catch (e) {
        console.error("Ошибка при обновлении предыдущей заметки:", e);
    }
}

_prevLink = prevLinkStr;
_nextLink = nextLinkStr;
%>---
type: Daily
date: <% tp.date.now("YYYY-MM-DD") %>
day_of_week: <% tp.date.now("dddd", 0, tp.date.now("YYYY-MM-DD")) %>
tags: [journal]
created: <% tp.date.now("YYYY-MM-DD HH:mm") %>
---
> [!dailynav]
> < <% _prevLink %> || <% _nextLink %> >

## 🧠 Мысли дня

> Что сегодня думаешь/планируешь делать?

## 📌 Задачи

- [ ] С чего начнем?

## 📚 Обучение / Прогресс

Что изучил нового?