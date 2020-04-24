# Pewlett-Hackard-Analysis

# Project Summary 

The company Pewlett Hackard has several employees approaching retirement age. This analysis uses SQL queries to identify these employees and the employees ready to mentor new employees. The entity relationship diagram below show the end state of the database required to complete the challenge. The queries and sample output section shows how we approached the challenge and the result of the queries.

![Database Diagram](https://github.com/raven-rivas/Pewlett-Hackard-Analysis/blob/master/Pewlett-Hackard-Analysis%20Folder/EmployeeDB.png)

# Number of Employees Retiring by Title
This query uses inner joins to the employees, titles and salary tables to extract the required data. It also uses an inner join to dept_emp to include current employees only. The employees table has a one to many relationship with the title table which creates duplicates in the dataset.

This query returned 54722 rows of retiring employees as we see multiple rows for some employees. Further analysis should focus on the areas of expertise of the retiring employees and how they relate to employees not retiring. Are the retiring employees more likely found in a particular area?

Query
select ce.emp_no, ce.first_name, ce.last_name, t.title, t.from_date, s.salary into challenge_emp_info from current_emp ce inner join titles t on ce.emp_no = t.emp_no inner join salaries s on ce.emp_no = s.emp_no inner 

Output
number challenge_emp_info with names that have duplicates because of titles.
![Number of Employees](https://github.com/raven-rivas/Pewlett-Hackard-Analysis/blob/master/Pewlett-Hackard-Analysis%20Folder/Employee_name_dup.PNG)

Only the Most Recent Titles
In this query we remove the duplicates from the dataset above using a partion statement. We used a CTE to facilitate the frequency count of employee titles. We made an assumption that the count should only include current title and not titles previously held by the same employee.

This query returns 33118 rows of retiring employees excluding the previous titles of the employee. As with the above query, further analysis could seek to understand the company's current staffing in the areas of the retiree's also, it would be intersting to understand if the employees have been working full time and if the company currently contracts for the roles with staff augmentation agencies. It could be beneficial to replace retiring employees with contract consultant to reduce liabilities in employee benefits.

Query
with SELECT emp_no, first_name, last_name, title, from_date, salary,  INTO current_title_info From (SELECT*, ROW_NUMBER() OVER (PARTITION BY (cei.employee_number, cei.first_name, cei.last_name)ORDER BY cei.from_date DESC) AS emp_row_number FROM challenge_emp_info AS cei) AS unique_employee WHERE emp_row_number =1;	

-- Get frequency count of employee titles 
SELECT *, count(ct.Employee_number) OVER (PARTITION BY ct.title ORDER BY ct.from_date DESC) AS emp_count INTO challenge_title_info   FROM current_title_info AS ct;
-- get total count per title group
SELECT COUNT(employee_number), title FROM challenge_title_info GROUP BY title;

Output
![current_title_info](https://github.com/raven-rivas/Pewlett-Hackard-Analysis/blob/master/Pewlett-Hackard-Analysis%20Folder/Employee_name_wo_dup.PNG)


Who’s Ready to be a Mentor?
The company is looking to identify employees ready to share work experience through mentorships. The query below identifies current employees born in 1965. We use a subquery instead of a CTE to eliminate duplicates since we did not need to reuse the dataset as in the above query.

This query returns 1549 employees ready to mentor base don their birth date in the year 1965. Further analysis should look at the time in the current position of the employees identified for mentorship. A good question to ask is if all of these employees posses the level of experience in their current role to teach others?

Query
select tm.emp_no, tm.first_name, tm.last_name, tm.title, tm.from_date , tm.to_date into ready_for_mentor from ( SELECT e.emp_no, e.first_name, e.last_name, t.title, t.from_date , t.to_date, ROW_NUMBER() OVER (PARTITION BY (first_name, last_name) ORDER BY t.from_date DESC) rn FROM employees e inner join titles t on e.emp_no = t.emp_no inner join dept_emp de on e.emp_no = de.emp_no WHERE e.birth_date BETWEEN '1965-01-01' AND '1965-12-31' and de.to_date = '9999-01-01' ) tm where tm.rn = 1 order by tm.last_name

ORDER BY tm.last_name
Output
![Ready to Mentor](https://github.com/raven-rivas/Pewlett-Hackard-Analysis/blob/master/Pewlett-Hackard-Analysis%20Folder/Employee_Ready_Mentor.PNG)
