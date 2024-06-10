```
SELECT 
    c.task_id,
    c.cancelled_time AS left_queue,
    n.new_time AS start_queue
FROM 
    (
        SELECT 
            t.task_id,
            h.action_time AS cancelled_time
        FROM 
            task t
        JOIN 
            history h ON t.task_id = h.task_id
        WHERE 
            t.stage_id = 1
            AND h.action = 'CANCELLED'
    ) c
JOIN 
    (
        SELECT 
            t.task_id,
            h.action_time AS new_time
        FROM 
            task t
        JOIN 
            history h ON t.task_id = h.task_id
        WHERE 
            t.stage_id = 1
            AND h.action = 'NEW'
    ) n ON c.task_id = n.task_id
LEFT JOIN 
    history h_assigned ON c.task_id = h_assigned.task_id 
    AND h_assigned.action = 'ASSIGNED' 
    AND h_assigned.action_time > c.cancelled_time 
    AND h_assigned.action_time < n.new_time
WHERE 
    h_assigned.task_id IS NULL
    AND n.new_time = (
        SELECT MIN(h_new.action_time)
        FROM history h_new
        WHERE h_new.task_id = c.task_id
        AND h_new.action = 'NEW'
        AND h_new.action_time > c.cancelled_time
    )
ORDER BY 
    c.task_id;

```
