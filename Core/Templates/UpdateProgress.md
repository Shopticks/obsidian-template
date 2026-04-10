<%*
const rootFolder = "00 - Projects/";
const files = app.vault.getMarkdownFiles();

const projectFiles = [];
const taskFiles = [];

// 1. Собираем файлы
files.forEach(file => {
    if (!file.path.startsWith(rootFolder)) return;
    const cache = app.metadataCache.getFileCache(file);
    const type = cache?.frontmatter?.type;
    const hasType = (t) => Array.isArray(type) ? type.includes(t) : type === t;

    if (hasType("Project")) projectFiles.push(file);
    else if (hasType("Task")) taskFiles.push(file);
});

// 2. Обработка и обновление проектов
for (const project of projectFiles) {
    const projectDir = project.parent.path;
    const projectTasks = taskFiles.filter(t => t.path.startsWith(projectDir));
    
    if (projectTasks.length > 0) {
        const total = projectTasks.reduce((sum, task) => {
            return sum + (app.metadataCache.getFileCache(task)?.frontmatter?.progress || 0);
        }, 0);
        
        const avgProgress = Math.ceil(total / projectTasks.length);

        // Записываем данные в файл проекта
        await app.fileManager.processFrontMatter(project, (fm) => {
            fm["progress"] = avgProgress;
        });
    }
}

new Notice("✅ Прогресс проектов пересчитан!");

// 3. УДАЛЕНИЕ ТЕКУЩЕГО ФАЙЛА
// Получаем объект текущего файла через Templater
// ... (весь код расчета остается прежним) ...

const fileToDelete = tp.config.target_file;

if (fileToDelete) {
    // 1. Создаем новую пустую вкладку, чтобы текущая не была активной
    await app.workspace.getLeaf(true).setViewState({ type: "empty" });

    // 2. Небольшая пауза для переключения фокуса
    window.setTimeout(async () => {
        try {
            // Находим и закрываем вкладку с ненужным файлом
            const leaves = app.workspace.getLeavesOfType("markdown");
            const leaf = leaves.find(l => l.view.file?.path === fileToDelete.path);
            if (leaf) leaf.detach();

            // Удаляем временный файл
            await app.vault.trash(fileToDelete, true);
        } catch (e) {
            console.error(e);
        }
    }, 500);
}
%>