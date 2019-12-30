# Software Course Database
* https://course_database.in
## features
* users should be able to view all the course details
### feature 1: list the course details


```sql
create table course_details(
    id number primary key,     
    course_name VARCHAR2(20) not null,
    course_cost number not null, 
    constraint course_cost_chk check(course_cost>0),
    constraint course_course_name_uk unique(course_name));
    insert into course_details (id,name,course,course_cost) values('santhosh','Pgdca', 45000);
    insert into course_details (id,name,course,course_cost) values('sam', 'Dca', 35000));
    
    
    query: select * from course_details;
```
