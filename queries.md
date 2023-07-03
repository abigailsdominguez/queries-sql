# Queries (SQL)
(source: [Leetcode](https://leetcode.com/problemset/all/))
##### Problem 1: Write an SQL query that reports the average experience years of all the employees for each project, rounded to 2 digits. Return the result table in any order.
Table: Project
| Column Name | Type |
| ------ | ------ |
| project_id | int|
| employee_id | int |
> project_id is the primary key of this table.
> employee_id is a foreign key to Employee table.
> Each row of this table indicates that the employee with employee_id is working on the project with project_id.

Table: Employee
| Column Name | Type |
| ------ | ------ |
| employee_id | int|
| name | varchar |
| experience_years | int|
> employee_id is the primary key of this table.
> It's guaranteed that experience_years is not NULL.
> Each row of this table contains information about one employee.

#### Solution:
```sh
select
    distinct p.project_id,
    round(avg(e.experience_years) over (partition by p.project_id),2) as average_years
from Project p
join Employee e
on p.employee_id = e.employee_id;
```

##### Problem 2: Write an SQL query to find employees who have the highest salary in each of the departments. Return the result table in any order.
Table: Employee
| Column Name | Type |
| ------ | ------ |
| id | int |
| name | varchar |
| salary | int |
| departmentId | int |
> id is the primary key column for this table.
> departmentId is a foreign key of the ID from the Department table.
> Each row of this table indicates the ID, name, and salary of an employee. It also contains the ID of their department.

Table: Department
| Column Name | Type |
| ------ | ------ |
| id | int|
| name | varchar |
> id is the primary key column for this table. It is guaranteed that department name is not NULL.
> Each row of this table indicates the ID of a department and its name.

#### Solution:
```sh
select 
    Department,
    Employee, 
    Salary
from
    (select
        d.name as Department,
        e.name as Employee,
        e.salary as Salary,
        dense_rank() over (partition by e.departmentID order by e.salary desc) as rnk
    from employee e
    left join department d
    on e.departmentId = d.id) x
where rnk = 1;
```

##### Problem 3: Write an SQL query to report the second highest salary from the Employee table. If there is no second highest salary, the query should report null.
Table: Employee
| Column Name | Type |
| ------ | ------ |
| id | int |
| salary | int |
> id is the primary key column for this table.
> Each row of this table contains information about the salary of an employee.

#### Solution:
```sh
select
    case 
        when count(salary) = 0 then null 
        else salary 
    end as SecondHighestSalary
from Employee
where salary = (
                select salary from Employee
                order by salary desc 
                limit 1
                offset 1);
```

##### Problem 4: Write an SQL query to rank the scores. The ranking should be calculated according to the following rules:
- The scores should be ranked from the highest to the lowest.
- If there is a tie between two scores, both should have the same ranking.
- After a tie, the next ranking number should be the next consecutive integer value. In other words, there should be no holes between ranks.

Return the result table ordered by score in descending order.

Table: Scores
| Column Name | Type |
| ------ | ------ |
| id | int |
| score | decimal |
> id is the primary key column for this table.
> Each row of this table contains the score of a game. Score is a floating point value with two decimal places.

#### Solution:
```sh
select
    score,
    dense_rank() over (order by score desc) as 'rank'
from scores;
```

##### Problem 5: Write an SQL query to find for each month and country, the number of transactions and their total amount, the number of approved transactions and their total amount. Return the result table in any order.

Table: Transactions
| Column Name | Type |
| ------ | ------ |
| id | int |
| country | varchar |
| state | enum  |
| amount | int |
| trans_date | date |
> id is the primary key column for this table.
> The table has information about incoming transactions.
> The state column is an enum of type ["approved", "declined"].

#### Solution:
```sh
select
    concat(year(trans_date),"-",lpad(month(trans_date),2,0)) as month,
    country,
    count(id) as trans_count,
    sum(case when state = 'approved' then 1 else 0 end) as approved_count,
    sum(amount) as trans_total_amount,
    sum(case when state = 'approved' then amount else 0 end) as approved_total_amount
from Transactions
group by
    year(trans_date), 
    month(trans_date),
    country;
```

