# Software Course Database
* https://course_database.in
## features
* users should be able to view all the course details
### feature 1: list the course details



create table course_details(
    id number primary key,
    name VARCHAR2(25) not null,     
    course VARCHAR2(20) not null,
    course_cost number, 
    constraint ccost_chk check(ccost>0));
    
    insert into studies (id,name,course,ccost) values('santhosh','Pgdca', 45000);
    insert into studies (id,name,course,ccost) values('sam', 'Dca', 35000));
    
    
    query: select * from course_details;
