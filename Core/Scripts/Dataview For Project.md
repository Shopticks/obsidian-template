```dataview
TABLE priority, status, due_date
FROM "00 - Projects/"
WHERE type = "Task" AND status != "Completed" AND status != "Rejected"
SORT 
  due_date ASC,
  choice(priority = "High", 1, choice(priority = "Medium", 2, choice(priority = "Low", 3, 4))) ASC,
  choice(status = "Active", 1, choice(status = "Planning", 2, choice(status = "Paused", 3, 4))) ASC
```