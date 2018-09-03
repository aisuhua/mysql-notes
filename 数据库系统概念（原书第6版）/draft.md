SQL

```sql
select course_id from course where  exists (select course_id from section where course.course_id = section.course_id and year = 2009 having count(section.course_id) = 1);
```
