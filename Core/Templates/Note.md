<%*
let cat = await tp.system.multi_suggester(
  item => item, Object.keys(tp.app.metadataCache.getTags()).map(x => x.replace("#", ""))
);
-%>---
type: Note
tags: [<% cat %>]
created: <% tp.date.now("YYYY-MM-DD") %>
---
## 💡 Краткое описание

Краткое описание заметки
## 📖 Подробное описание

Подробное описание заметки
## 🔗 Связанные концепции

- Концепция 1