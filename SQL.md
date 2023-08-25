# Примеры задач на SQL
Здравствуйте, здесь представлены 5 задач, решенные мною
на различных сайтах-тренажерах. Они призваны показать
уровень владения SQL и его инструментами.

## Department Highest Salary

Write a solution to find employees who have the highest salary in each of the departments.

Return the result table in any order.

```
SELECT Department.name AS Department, Employee.name AS Employee, salary AS Salary
FROM Department JOIN Employee ON Department.id = Employee.departmentId
WHERE (Employee.departmentId, Employee.salary) IN(
    SELECT departmentId, MAX(salary)FROM Employee
    GROUP BY departmentId)
```

## Nth Highest Salary

Write a solution to find the nth highest salary from the Employee table. If there is no nth highest salary, return null.

```
CREATE FUNCTION getNthHighestSalary(N INT) RETURNS INT
BEGIN
SET N = N - 1;
    RETURN (
        IFNULL(
            (SELECT DISTINCT salary FROM Employee
            ORDER BY salary DESC
            LIMIT 1 OFFSET N),
            NULL
    ) 
  );
END
```

## Premium vs Freemium
Find the total number of downloads for paying and non-paying users by date. Include only records where non-paying customers have more downloads than paying customers. The output should be sorted by earliest date first and contain 3 columns date, non-paying downloads, paying downloads.

```
select * from(
    select date, sum(case
        when paying_customer = 'yes'
        then downloads
        end) as paying,
        sum(case
        when paying_customer = 'no'
        then downloads
        end) as non_paying from(
        select * from
        (ms_download_facts as loads join ms_user_dimension as users on loads.user_id = users.user_id) as load_by_user join ms_acc_dimension as acc on load_by_user.acc_id = acc.acc_id) as joined
    group by date) as counted
where paying < non_paying
order by date
```
## Marketing Campaign Success [Advanced]

You have a table of in-app purchases by user. Users that make their first in-app purchase are placed in a marketing campaign where they see call-to-actions for more in-app purchases. Find the number of users that made additional in-app purchases due to the success of the marketing campaign.


The marketing campaign doesn't start until one day after the initial in-app purchase so users that only made one or multiple purchases on the first day do not count, nor do we count users that over time purchase only the products they purchased on the first day.

```
select count(*) from (
    select distinct user_id from marketing_campaign
    where concat(user_id, product_id) not in (
        select concat(user_id, product_id) from marketing_campaign
        where concat(user_id, date(created_at)) in (
            select concat(user_id, date(min)) from (
                select user_id, min(created_at) from marketing_campaign
                group by user_id) as start))) as id_success;
```

## Host Popularity Rental Prices

You’re given a table of rental property searches by users. The table consists of search results and outputs host information for searchers. Find the minimum, average, maximum rental prices for each host’s popularity rating. The host’s popularity rating is defined as below:
0 reviews: New
1 to 5 reviews: Rising
6 to 15 reviews: Trending Up
16 to 40 reviews: Popular
more than 40 reviews: Hot


Tip: The id column in the table refers to the search ID. You'll need to create your own host_id by concating price, room_type, host_since, zipcode, and number_of_reviews.


Output host popularity rating and their minimum, average and maximum rental prices.

```
with formated as (
    select distinct price, host_id, popularity from (
        select *,concat(price,room_type,host_since,zipcode,number_of_reviews) as host_id, (case
            when number_of_reviews = 0 then 'New'
            when number_of_reviews between 1 and 5 then 'Rising'
            when number_of_reviews between 6 and 15 then 'Trending Up'
            when number_of_reviews between 16 and 40 then 'Popular'
            when number_of_reviews > 40 then 'Hot'
            end) as popularity from airbnb_host_searches) popularity_ranked)

select popularity, min(price), avg(price), max(price) from formated
group by popularity
```