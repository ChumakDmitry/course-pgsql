### Домашнее задание №10

1. Проанализировать данные о зарплатах сотрудников с использованием оконных функций.
а) На сколько было увеличение с предыдущей зарплатой
б) если это первая зарплата - вместо NULL вывести 0

```
WITH last_salary AS (
	SELECT 
  		fk_employee,
  		amount,
  		from_date,
  		LAG(amount) OVER (PARTITION BY fk_employee ORDER BY from_date) AS last_salary
  	FROM salary
)
SELECT fk_employee, amount, from_date, coalesce(amount - last_salary, 0) AS salary_increase
FROM last_salary
ORDER BY fk_employee;

https://www.db-fiddle.com/f/eQ8zNtAFY88i8nB4GRB65V/1
```
