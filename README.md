# SQL-project
/* 1.	Create a database named employee, then import data_science_team.csv proj_table.csv and
 emp_record_table.csv into the employee database from the given resources. */
create database employee;
use employee;
SELECT 
    *
FROM
    data_science_team;
SELECT 
    *
FROM
    emp_record_table;
SELECT 
    *
FROM
    proj_table;

-- 2.	Create an ER diagram for the given employee database.

/* 3.	Write a query to fetch EMP_ID, FIRST_NAME, LAST_NAME, GENDER, and DEPARTMENT from the employee record table, 
and make a list of employees and details of their department. */
select EMP_ID,FIRST_NAME,LAST_NAME,GENDER,DEPT from emp_record_table ;

/* 4.	Write a query to fetch EMP_ID, FIRST_NAME, LAST_NAME, GENDER, DEPARTMENT, and EMP_RATING if the EMP_RATING is: 
●	less than two
●	greater than four 
●	between two and four */
select EMP_ID,FIRST_NAME,LAST_NAME,GENDER,DEPT as 'DEPARTMENT' ,EMP_RATING
 from emp_record_table  
 where EMP_RATING < 2;

select EMP_ID,FIRST_NAME,LAST_NAME,GENDER,DEPT as 'DEPARTMENT' ,EMP_RATING
 from emp_record_table  
 where EMP_RATING >4;

select EMP_ID,FIRST_NAME,LAST_NAME,GENDER,DEPT as 'DEPARTMENT' ,EMP_RATING
 from emp_record_table  
 where EMP_RATING between 2 and 4;
 
/*  5.	Write a  query to concatenate the FIRST_NAME and the LAST_NAME of employees in the Finance department
 from the employee table and then give the resultant column alias as NAME. */
 
 select EMP_ID, DEPT as 'department', concat(trim(FIRST_NAME),' ',(LAST_NAME)) as 'Name'
 from emp_record_table 
 where DEPT = 'FINANCE';
 
select EMP_ID, DEPT as 'department', upper(concat(trim(FIRST_NAME),' ',(LAST_NAME))) as 'Name'
 from emp_record_table 
 where DEPT = 'FINANCE';
 
 /* 6.	Write a query to list only those employees who have someone reporting to them. 
 Also, show the number of reporters (including the President). */
select e.EMP_ID as 'manager id', count(e.EMP_ID) as 'no of reports'
from emp_record_table e 
inner join emp_record_table m 
on e.EMP_ID = m.MANAGER_ID 
group by 1
order by 1;

/* 7. Write a query to list down all the employees from the healthcare and finance departments using union.
 Take data from the employee record table. */
 select * from emp_record_table 
 where DEPT in ('FINANCE','HEALTHCARE');     --- (1)method 
 
 select EMP_ID,FIRST_NAME,LAST_NAME,DEPT
 from emp_record_table 
where DEPT in ('FINANCE','HEALTHCARE')
order by 4;                                 --- (2)method

select * from emp_record_table 
where  DEPT = 'FINANCE'
union 
select * from emp_record_table 
where  DEPT = 'HEALTHCARE';                  --- using union 

/* 8.	Write a query to list down employee details such as EMP_ID, FIRST_NAME, LAST_NAME, ROLE, DEPARTMENT, and EMP_RATING grouped by dept.
 Also include the respective employee rating along with the max emp rating for the department. */
select EMP_ID,FIRST_NAME,LAST_NAME,ROLE,DEPT,EMP_RATING ,
max(EMP_RATING) over(partition by DEPT)  As 'max_emp_rating'
from emp_record_table ;

/* 9.	Write a query to calculate the minimum and the maximum salary of the employees in each role. 
Take data from the employee record table */
use employee;
select *,
max(SALARY) over(partition by ROLE) as 'max salary',
min(SALARY) over(partition by ROLE) as 'min salary'
from emp_record_table;

select ROLE, max(SALARY) AS 'Max_Salary', MIN(SALARY) AS 'Min_SAlary'
from emp_record_table
group by 1 ;

-- 10.	Write a query to assign ranks to each employee based on their experience. Take data from the employee record table.
select *,
rank() over() as r1 ,
rank() over(order by EXP desc) as 'ranks'
from emp_record_table;

select *,
dense_rank() over() as r1 ,
dense_rank() over(order by EXP desc) as 'denseranks'
from emp_record_table;                      -- use dense rank preffred
 
/* 11.	Write a query to create a view that displays employees in various countries whose salary is more than six thousand. 
Take data from the employee record table. */
 
select COUNTRY,SALARY 
from emp_record_table 
where SALARY > 6000;        -- by filtering column 
 
select * from emp_record_table 
where SALARY > 6000;            -- selecting whole table

create view emp_salary_view
as 
select * from emp_record_table 
where SALARY > 6000;                
select * from employeesalaryview ;

/* 12.	Write a nested query to find employees with experience of more than ten years. 
Take data from the employee record table.  */
-- sub query
 select EMP_ID from emp_record_table 
 where EXP > 10 ;
-- main query 
select * from emp_record_table where EMP_ID in 
(select EMP_ID from emp_record_table where EXP > 10) ;
 
 /* 13.	Write a query to create a stored procedure to retrieve the details of the employees whose experience is more than three years. 
 Take data from the employee record table. */

delimiter &&
create procedure expof3_years()
BEGIN
select * from emp_record_table where EXP > 3;
END &&
delimiter ;

call expof3_years();

/* 14.	Write a query using stored functions in the project table to 
check whether the job profile assigned to each employee in the data science team matches the organization’s set standard.
The standard being:
For an employee with experience less than or equal to 2 years assign 'JUNIOR DATA SCIENTIST',
For an employee with the experience of 2 to 5 years assign 'ASSOCIATE DATA SCIENTIST',
For an employee with the experience of 5 to 10 years assign 'SENIOR DATA SCIENTIST',
For an employee with the experience of 10 to 12 years assign 'LEAD DATA SCIENTIST',
For an employee with the experience of 12 to 16 years assign 'MANAGER' */
use employee;

delimiter &&
create function job_prof_assg(EXP int )
returns varchar(50)
deterministic 
BEGIN
declare job_profile varchar(50);
if EXP <= 2 THEN set job_profile =  'JUNIOR DATA SCIENTIST';
elseif EXP >2 and EXP <= 5 THEN set job_profile = 'ASSOCIATE DATA SCIENTIST';
elseif EXP >5 AND EXP <= 10 THEN set job_profile = 'SENIOR DATA SCIENTIST';
elseif EXP > 10 AND EXP <= 12 THEN set job_profile = 'LEAD DATA SCIENTIST';
else SET job_profile = 'MANAGER' ;
end IF ;

return(job_profile) ;

END &&

delimiter ;  

SELECT EMP_ID,EXP, job_prof_assg(EXP)
FROM data_science_team ;
/* 15.	Create an index to improve the cost and performance of the query to
 find the employee whose FIRST_NAME is ‘Eric’ in the employee table after checking the execution plan. */
 
 select * from emp_record_table where FIRST_NAME like '%Eric';
create index idx_first_name
on emp_record_table(FIRST_NAME);

CREATE INDEX idx_first_name 
ON emp_record_table (FIRST_NAME(50));

 /* 16.	Write a query to calculate the bonus for all the employees, 
 based on their ratings and salaries (Use the formula: 5% of salary * employee rating). */
 select EMP_ID,FIRST_NAME,EMP_RATING,SALARY ,
 (0.05* SALARY *EMP_RATING) as 'Bonus'
 from emp_record_table ;
 
/* 17.	Write a query to calculate the average salary distribution based on the continent and country.
 Take data from the employee record table. */
 select CONTINENT,COUNTRY, round(avg(SALARY)) as 'avg salary'
 from emp_record_table 
 group by 1,2
 order by 3 desc ;
 
