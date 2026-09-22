# Introduction
This project analyzes Data Analyst job postings from 2023 to identify trends in salaries, job demand, and the skills associated with Data Analyst roles. Using SQL, I explored the dataset to uncover insights that can help understand what employers were looking for and which skills were associated with higher-paying opportunities.

SQL queries? Check them out here: [project_sql folder](/project_sql/)
# Background
I created this project as part of my journey to develop practical SQL and data analysis skills. The goal was to move beyond learning SQL syntax and apply it to a real-world dataset.

The analysis focused on questions such as:
1. What are the top-paying Data Analyst roles?
2. Which skills are most commonly requested for these top-paying jobs?
3. Which skills are most in demand for data analysts?
4. Which skills are associated with higher salaries?
5. What are the most optimal skills to learn?
# Tools I Used
- **PostgreSQL** – Used to store, query, and analyze the job-postings dataset.
- **SQL** – Used for data filtering, aggregation, joins, subqueries, CTEs, and conditional analysis.
- **VS Code** – Used as the development environment for writing and managing SQL queries.
- **Git & GitHub** – Used for version control and documenting the project.
# The Analysis
Each query for this project aimed at investigating specific aspects of the data analyst job market. Here is how i appproached each questions:

### 1. Top Paying Data Analyst Jobs
To identify the highest-paying roles, I filtered data analyst positions by average yearly salary and location, focusing on remote jobs. This query highlights the high paying opportunities in the field.

```sql
SELECT
      job_id,
      job_title,
      name AS company_name,
      job_location,
      job_schedule_type,
      salary_year_avg,
      job_posted_date
FROM
      job_postings_fact
LEFT JOIN company_dim ON job_postings_fact.company_id = company_dim.company_id
WHERE 
      job_title_short = 'Data Analyst' AND job_location = 'Anywhere' AND salary_year_avg IS NOT NULL
ORDER BY
      salary_year_avg DESC
LIMIT 10;
```
Here's the breakdown of the data analyst jobs in 2023:
- **Wide Salary Range:** Top 10 paying data analyst roles span from $184,000 to $650,000, indicating significant salary potential in the field.
- **Diverse Employers:** Companies like SmartAsset, Meta, and AT&T are among those offering high salries, showing a broad interest across different industries.
- **Job Title Variety:** There's a high diversity in the job titles, from Data Analyst to Director of Analytics, reflecting varied roles and specia lizations within data analytics.

### 2. Top Paying Job Skills
To identify the skills associated with the highest-paying data analyst positions, I analyzed the skills required by the top-paying jobs and compared them based on the average salary offered.
```sql
WITH top_paying_jobs AS (
      SELECT
            job_id,
            job_title,
            name AS company_name,
            salary_year_avg
      FROM
            job_postings_fact
      LEFT JOIN company_dim ON job_postings_fact.company_id = company_dim.company_id
      WHERE 
            job_title_short = 'Data Analyst' AND job_location = 'Anywhere' AND salary_year_avg IS NOT NULL
      ORDER BY
            salary_year_avg DESC
      LIMIT 10
)

SELECT 
      top_paying_jobs.*,
      skills
FROM top_paying_jobs
INNER JOIN skills_job_dim ON top_paying_jobs.job_id = skills_job_dim.job_id
INNER JOIN skills_dim ON skills_job_dim.skill_id = skills_dim.skill_id
ORDER BY
      salary_year_avg DESC
```
Here's the breakdown of the most valuable skills for data analyst roles in 2023:

- **High-Value Technical Skills:** Skills such as Python, SQL, and specialized data technologies appear frequently among higher-paying positions, showing the value of strong technical abilities in the data analytics field.
- **Advanced Skills Increase Earning Potential:** The higher-paying roles tend to require more advanced technical and analytical skills, suggesting that developing beyond basic spreadsheet skills can open access to more specialized opportunities.
- **Combination of Skills:** The results show that employers often seek candidates with a combination of programming, database, visualization, and analytical skills, rather than relying on a single skill.

### 3. Top Demanded Skills
To determine which skills are most frequently requested by employers, I analyzed the number of job postings associated with each skill. This helped identify the skills that are most commonly required in the data analyst job market.

```sql
SELECT 
      skills,
      COUNT(skills_job_dim.job_id) AS demand_count
FROM job_postings_fact
INNER JOIN skills_job_dim ON job_postings_fact.job_id = skills_job_dim.job_id
INNER JOIN skills_dim ON skills_job_dim.skill_id = skills_dim.skill_id
WHERE
        job_title_short = 'Data Analyst' AND job_work_from_home = True
GROUP BY
      skills
ORDER BY
      demand_count DESC
LIMIT 5
```
Here's the breakdown of the most demanded skills in 2023:
- **Strong Demand for SQL:** SQL stands out as one of the most frequently requested skills, highlighting the importance of database querying and data extraction.
- **Importance of Excel and Visualization:** Skills such as Excel and data visualization tools are also commonly requested, showing that employers value the ability to analyze and communicate data effectively.
- **Growing Demand for Programming:** Python appears as an important technical skill, demonstrating that programming is increasingly useful for data analysts, particularly for data manipulation and more advanced analysis.
- **Broad Skill Requirements:** The results suggest that data analyst roles require a mixture of database, spreadsheet, programming, and visualization skills.

### 4. Top Paying Skills
To identify the individual skills associated with the highest salaries, I analyzed the average salary of data analyst jobs requiring each skill. This helped determine which technical skills were associated with greater earning potential.

```sql
SELECT 
      skills,
      ROUND(AVG(salary_year_avg), 0) AS avg_salary
FROM job_postings_fact
INNER JOIN skills_job_dim ON job_postings_fact.job_id = skills_job_dim.job_id
INNER JOIN skills_dim ON skills_job_dim.skill_id = skills_dim.skill_id
WHERE
      job_title_short = 'Data Analyst' AND salary_year_avg IS NOT NULL AND job_work_from_home = TRUE
GROUP BY
      skills
ORDER BY
      avg_salary DESC
LIMIT 25
```
Here's the breakdown of the highest-paying skills in 2023:
- **Specialized Skills Command Higher Salaries:** Some less commonly requested technical skills are associated with higher average salaries, suggesting that specialization can provide access to higher-paying opportunities.
- **Advanced Technical Knowledge:** Skills involving advanced programming, cloud technologies, and specialized data platforms tend to appear among the higher-paying skills.
- **Demand and Salary Are Not Always the Same:** A skill can be highly demanded without being among the highest-paying skills. This shows that frequency of demand and earning potential are two different factors when choosing which skills to develop.

### 5. Optimal Skills to Learn
To identify the most valuable skills for someone pursuing a data analyst career, I combined skill demand and salary data. This allowed me to identify skills that provide a balance between being frequently requested by employers and being associated with higher salaries.

```sql
SELECT 
      skills_dim.skill_id,
      skills_dim.skills,
      COUNT(skills_job_dim.job_id) AS demand_count,
      ROUND(AVG(job_postings_fact.salary_year_avg), 0) AS avg_salary
FROM job_postings_fact
INNER JOIN skills_job_dim ON job_postings_fact.job_id = skills_job_dim.job_id
INNER JOIN skills_dim ON skills_job_dim.skill_id = skills_dim.skill_id
WHERE
     job_title_short = 'Data Analyst'
     AND salary_year_avg IS NOT NULL
     AND job_work_from_home = True
GROUP BY
      skills_dim.skill_id
HAVING
     COUNT(skills_job_dim.job_id) > 10
ORDER BY
      avg_salary DESC,
      demand_count DESC
LIMIT 25;
```
Here's the breakdown of the optimal skills to learn in 2023:
- **Balance Between Demand and Salary:** The analysis shows that the most useful skills are not necessarily the skills with the highest salary alone. Skills that are both frequently requested and well-compensated provide a stronger career opportunity.
- **SQL and Python Stand Out:** SQL is highly valuable because of its strong demand across data analyst positions, while Python provides additional capabilities for data manipulation, automation, and advanced analytics.
- **Visualization Skills Add Value:** Data visualization tools help analysts transform their findings into information that can be easily understood by businesses and decision-makers.
- **Developing a Combination of Skills:** The analysis suggests that becoming proficient in a combination of SQL, Excel, Python, and visualization tools can provide a strong foundation for entering the data analytics job market.
- **Specialization Can Increase Earning Potential:** After developing the core skills, learning more specialized technologies can help differentiate a candidate and potentially provide access to higher-paying roles.

# What I Learned
This project helped me understand how SQL can be used to answer real business and career-related questions from a large dataset.

Some of the key concepts I practiced include:

- Filtering data using `WHERE`
- Aggregating data using functions such as `COUNT()` and `AVG()`
- Grouping results using `GROUP BY`
- Filtering aggregated results using `HAVING`
- Combining tables using different types of `JOIN`
- Creating conditional categories using `CASE`
- Using subqueries and Common Table Expressions (CTEs)
- Extracting information from dates using `EXTRACT()`
- Sorting and limiting results
- Using SQL to identify patterns and communicate insights

Beyond SQL syntax, I learned that effective data analysis is not just about writing queries. It involves asking useful questions, interpreting the results, and turning raw data into meaningful insights.

# Conclusions
This project showed that the Data Analyst role can involve much more than basic reporting and spreadsheets. In the job postings analyzed, higher-paying opportunities were often associated with a combination of SQL, Python, and more advanced data technologies.

The analysis also reinforced the importance of continuously developing technical skills. SQL provides a strong foundation for working with data, while Python, visualization, cloud platforms, and data engineering tools can expand the types of problems an analyst can solve.

Overall, this project strengthened my practical SQL skills and gave me experience applying data analysis techniques to a real-world dataset.