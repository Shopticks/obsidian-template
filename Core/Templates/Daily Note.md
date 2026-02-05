<%*
const filePath = "04 - Daily/" + tp.date.now("YYYY-MM-DD");
await tp.file.move(filePath);
%>---
type: Daily
date: <% tp.date.now("YYYY-MM-DD") %>
day_of_week: <% tp.date.now("dddd", 0, tp.date.now("YYYY-MM-DD")) %>
tags: [journal]
created: <% tp.date.now("YYYY-MM-DD HH:mm") %>
---
## 🧠 Мысли дня

> Что сегодня думаешь\планируешь делать?
## 📌 Задачи

- [ ] С чего начнем?
## 📚 Обучение / Прогресс

Что изучил нового?
## 🔗 Связи
- Предыдущая: [[<% tp.date.now("YYYY-MM-DD", -1) %>]]
- Следующая: [[<% tp.date.now("YYYY-MM-DD", +1) %>]]