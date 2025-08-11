# SQL_EmployeeDatabase Analysis using CTE
This project contains a three-table employee database (Departments, Job_Titles, Employee_Records) with over 120 manually generated employee records. It is designed for MS SQL Server practice, focusing on Common Table Expressions (CTEs) from basic to advanced levels,  including:
- Filtering and transforming employee data
- Ranking and aggregation using window functions
- Department and job title analysis
- Salary and performance comparisons
The repository includes:
- CSV files for all three tables (ready for import into SQL Server)
- Example SQL queries with answers for beginner, intermediate, and advanced CTE scenarios

```SQL
SELECT * FROM Departments
SELECT * FROM Employees
SELECT * FROM Jobtitles

--Q1) Get full names and job titles of all employees
WITH emp_fullname AS 
(
	SELECT
		EmployeeID,
		JobTitleID,
		(FirstName+ ' ' + LastName) AS Full_Name
	FROM
		Employees
)
SELECT
	emp_fullname.EmployeeID,
	emp_fullname.Full_Name,
	Jobtitles.TitleName
FROM
	emp_fullname
JOIN
	Jobtitles ON Jobtitles.JobTitleID = emp_fullname.JobTitleID

--Q2) List employees hired after 2020
WITH emp_details AS
(
	SELECT
		EmployeeID,
		FirstName,
		LastName,
		YEAR(HireDate) AS year
	FROM
		Employees
)
SELECT *
FROM
	emp_details
WHERE
	year >= 2020
ORDER BY
	year ASC
	
--Q3) Find departments with more than 5 employees
WITH Department_Count AS
(
	SELECT
		DepartmentID,
		COUNT(*) AS number_of_employees
	FROM
		Employees
	GROUP BY
		DepartmentID
)
SELECT
	Departments.DepartmentName,
	number_of_employees
FROM
	Department_Count
JOIN
	Departments ON Departments.DepartmentID = Department_Count.DepartmentID
WHERE
	number_of_employees > 5

--Q4) Get average salary by department
WITH avg_dept_salary AS 
(
	SELECT
		DepartmentID,
		AVG(BaseSalary + Bonus) AS avg_salary
	FROM
		Employees
	GROUP BY
		DepartmentID
)
SELECT
	Departments.DepartmentName,
	avg_dept_salary.avg_salary
FROM
	Departments
JOIN
	avg_dept_salary ON Departments.DepartmentID = avg_dept_salary.DepartmentID
ORDER BY
	avg_salary DESC

--Q5) Find employees with salary above the average of their department

WITH avg_dept_salary AS
(
	SELECT
		DepartmentID,
		AVG(BaseSalary) AS avg_salary
	FROM
		Employees
	GROUP BY
		DepartmentID
)
SELECT
	Employees.FirstName,
	Employees.LastName,
	Departments.DepartmentName,
	avg_dept_salary.avg_salary,
	(Employees.BaseSalary) AS salary
FROM
	Employees
JOIN
	Departments On Departments.DepartmentID = Employees.DepartmentID
JOIN
	avg_dept_salary ON Employees.DepartmentID = avg_dept_salary.DepartmentID
WHERE
	(Employees.BaseSalary) > avg_dept_salary.avg_salary
ORDER BY
	Departments.DepartmentName

--Q6) Rank employees by performance within their department
--Method 1
SELECT
	Departments.DepartmentName,
	Employees.FirstName,
	Employees.LastName,
	Employees.PerformanceRating,
	RANK() OVER(PARTITION BY Departments.DepartmentName ORDER BY Employees.PerformanceRating DESC) AS rank
FROM
	Departments
JOIN
	Employees
ON
	Departments.DepartmentID = Employees.DepartmentID

--Method 2 (Using CTE)
WITH employee_rank AS
(
SELECT
	DepartmentID,
	FirstName,
	LastName,
	PerformanceRating,
	RANK() OVER (PARTITION BY DepartmentID ORDER BY PerformanceRating DESC) AS Rank
FROM
	Employees
)
SELECT
	Departments.DepartmentName,
	employee_rank.FirstName,
	employee_rank.LastName,
	employee_rank.PerformanceRating,
	employee_rank.Rank
FROM
	employee_rank
JOIN
	Departments ON Departments.DepartmentID = employee_rank.DepartmentID

--Q7) Get employee count by country and average salary using CTE
SELECT
	Country,
	COUNT(DISTINCT EmployeeID) as number_of_employees,
	AVG(BaseSalary + Bonus) AS avg_alary
FROM
	Employees
GROUP BY
	Country

--Q8) List employees over 40 years old and working in 'Engineering'
WITH older_employees AS
(
	SELECT
		*
	FROM
		Employees
	WHERE
		Age > 40
)
SELECT
	older_employees.FirstName,
	Older_employees.LastName,
	older_employees.Age,
	Departments.DepartmentName
FROM
	Departments
JOIN
	older_employees ON Departments.DepartmentID = older_employees.DepartmentID
WHERE
	Departments.DepartmentName = 'Engineering'

--Q9) Find the number of active and terminated employees per department

WITH Terminated_Employees AS
(
	SELECT
		Departments.DepartmentID,
		Departments.DepartmentName,
		COUNT(*) AS number_of_terminated
	FROM
		Departments
	JOIN
		Employees
	ON
		Departments.DepartmentID = Employees.DepartmentID
	WHERE
		Employees.IsActive = 0
	GROUP BY
		Departments.DepartmentID,
		Departments.DepartmentName
),
Active_Employees AS
(
	SELECT
		Departments.DepartmentID,
		Departments.DepartmentName,
		COUNT(*) AS active_employees
	FROM
		Departments
	JOIN
		Employees
	ON
		Departments.DepartmentID = Employees.DepartmentID
	WHERE
		Employees.IsActive = 1
	GROUP BY
		Departments.DepartmentID,
		Departments.DepartmentName
)
SELECT
	Departments.DepartmentName,
	Active_Employees.active_employees,
	Terminated_Employees.number_of_terminated
FROM
	Departments
JOIN
	Active_Employees
ON
	Departments.DepartmentID = Active_Employees.DepartmentID
JOIN
	Terminated_Employees
ON
	Terminated_Employees.DepartmentID = Departments.DepartmentID

--Q10) Get employees whose bonus is more than 15% of their base salary
WITH high_bonus AS
(
	SELECT
		*,
		ROUND((CAST(Bonus AS FLOAT)/NULLIF(BaseSalary,0)) * 100,2) AS percentage_of_bonus_on_basesalary
	FROM
		Employees
)
SELECT
	FirstName,
	LastName,
	BaseSalary,
	Bonus,
	percentage_of_bonus_on_basesalary
FROM
	high_bonus
WHERE
	percentage_of_bonus_on_basesalary > 15

--Q11) Get job titles where average salary is below $70,000
WITH avg_salary AS
(
	SELECT
		Jobtitles.TitleName,
		AVG(BaseSalary) AS Avg_Salary
	FROM
		Jobtitles
	JOIN
		Employees
	ON
		Employees.JobTitleID = Jobtitles.JobTitleID
	GROUP BY
		Jobtitles.TitleName
)
SELECT
	TitleName,
	avg_salary
FROM
	avg_salary
WHERE
	Avg_Salary < 70000


	https://chatgpt.com/c/688bb0a3-b460-800d-a03c-622ece9736e8
--Q12) Find employees who joined earlier than the average hire date
SELECT
	FirstName,
	LastName,
	YEAR(HireDate) AS Hire_Year
FROM
	Employees
WHERE
	YEAR(HireDate) < (SELECT AVG(YEAR(HireDate)) FROM Employees)

--Q13) Compare each employee's salary to the average salary of their department
WITH dept_avg_salary AS
(
	SELECT
		Departments.DepartmentID,
		Departments.DepartmentName,
		AVG(Employees.BaseSalary) AS avg_salary
	FROM
		Departments
	JOIN
		Employees
	ON
		Departments.DepartmentID = Employees.DepartmentID
	GROUP BY
		Departments.DepartmentID,
		Departments.DepartmentName
)
SELECT
	Employees.FirstName,
	Employees.LastName,
	dept_avg_salary.DepartmentName,
	Employees.BaseSalary,
	dept_avg_salary.avg_salary,
	CASE
		WHEN Employees.BaseSalary > dept_avg_salary.avg_salary THEN 'Above Average'
		WHEN Employees.BaseSalary < dept_avg_salary.avg_salary THEN 'Below Average'
		ELSE
			'Eql to Average'
		END AS avg_sala_status
FROM
	Employees
JOIN
	dept_avg_salary ON Employees.DepartmentID = dept_avg_salary.DepartmentID
ORDER BY
	dept_avg_salary.DepartmentName DESC

--Q14) Find the most common job title in each department
WITH job_title_rank AS
(
	SELECT
		Departments.DepartmentID,
		Departments.DepartmentName,
		Jobtitles.TitleName,
		COUNT(Jobtitles.JobTitleID) AS number_of_titles,
		RANK() OVER (PARTITION BY Departments.DepartmentName ORDER BY COUNT(Jobtitles.JobTitleID) DESC) AS rank
	FROM
		Departments
	JOIN
		Employees ON Employees.DepartmentID = Departments.DepartmentID
	JOIN
		Jobtitles ON Jobtitles.JobTitleID = Employees.JobTitleID
	GROUP BY
		Departments.DepartmentID,
		Departments.DepartmentName,
		Jobtitles.TitleName
)
SELECT
	*
FROM
	job_title_rank
WHERE
	rank = 1

--Q15) Find average age and performance rating per department-country combination
SELECT * FROM Departments
SELECT * FROM Employees
SELECT * FROM Jobtitles

WITH DeptCountryStats AS (
    SELECT DepartmentID, Country,
           AVG(Age) AS AvgAge,
           AVG(PerformanceRating) AS AvgRating
    FROM Employees
    GROUP BY DepartmentID, Country
)
SELECT D.DepartmentName, DS.Country, DS.AvgAge, DS.AvgRating
FROM DeptCountryStats DS
JOIN Departments D ON DS.DepartmentID = D.DepartmentID;














	





