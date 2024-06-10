```
SELECT 
    s.task_id,
    n.new_time AS start_queue,
    s.suspended_time AS left_queue
FROM 
    (
        SELECT 
            t.task_id,
            h.action_time AS suspended_time
        FROM 
            task t
        JOIN 
            history h ON t.task_id = h.task_id
        WHERE 
            t.stage_id = 1
            AND h.action = 'SUSPENDED'
    ) s
JOIN 
    (
        SELECT 
            t.task_id,
            MIN(h.action_time) AS new_time
        FROM 
            task t
        JOIN 
            history h ON t.task_id = h.task_id
        WHERE 
            t.stage_id = 1
            AND h.action = 'NEW'
        GROUP BY 
            t.task_id
    ) n ON s.task_id = n.task_id
LEFT JOIN 
    history h_assigned ON s.task_id = h_assigned.task_id 
    AND h_assigned.action = 'ASSIGNED' 
    AND h_assigned.action_time > s.suspended_time 
    AND h_assigned.action_time < n.new_time
WHERE 
    h_assigned.task_id IS NULL
ORDER BY 
    s.task_id;
```
