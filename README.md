```
WITH filtered_tasks AS (
    SELECT 
        t.task_id,
        h.action,
        h.action_time
    FROM 
        task t
    JOIN 
        history h ON t.task_id = h.task_id
    WHERE 
        t.stage_id = 1
),
new_actions AS (
    SELECT 
        task_id,
        action_time AS new_time
    FROM 
        filtered_tasks
    WHERE 
        action = 'NEW'
),
suspended_actions AS (
    SELECT 
        task_id,
        action_time AS suspended_time
    FROM 
        filtered_tasks
    WHERE 
        action = 'SUSPENDED'
),
assigned_actions AS (
    SELECT 
        task_id,
        action_time AS assigned_time
    FROM 
        filtered_tasks
    WHERE 
        action = 'ASSIGNED'
),
suspended_without_assigned AS (
    SELECT 
        s.task_id,
        s.suspended_time,
        n.new_time
    FROM 
        suspended_actions s
    JOIN 
        new_actions n ON s.task_id = n.task_id
    LEFT JOIN 
        assigned_actions a ON s.task_id = a.task_id 
        AND a.assigned_time > s.suspended_time 
        AND a.assigned_time < n.new_time
    WHERE 
        a.task_id IS NULL
)
SELECT 
    task_id,
    new_time AS start_queue,
    suspended_time AS left_queue
FROM 
    suspended_without_assigned
ORDER BY 
    task_id;
```
