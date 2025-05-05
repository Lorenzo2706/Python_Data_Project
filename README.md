# Overview
This project aims to identify the optimal skills to learn for securing a job as a data analyst, exploiting Python for analytics and data visualization. Leveraging a dataset of data science job postings from Hugging Face, the project explores which skills are most in demand, how those demands are changing, and which skills command the highest salaries.

However, the main goal of this project is to consolidate my skills in Python, showcasing my ability to leverage multiple modules and libraries to analyze data, find insights and create a story

# The Questions
To achieve my goal, I aim to answer five main questions:
1. In **which country** shall I focus my analysis? 
2. What are the **skills most in demand** for the top 3 most popular data roles?
3. How are in-demand skills **trending for Data Analysts?**
4. How well do jobs and **skills pay** for Data Analysts?
5. What are the **optimal skills for data analysts to learn?** (High Demand AND High Paying)

# Tools I Used
To conduct an in-depth exploration of the data analyst job market, I utilized a range of powerful tools:
- **Python** served as the core of my analysis, enabling data processing and insight generation. Key libraries included:
    - Pandas for data manipulation and analysis
    - Matplotlib for creating visualizations
    - Seaborn for producing more sophisticated, insightful charts
- **Jupyter Notebooks** provided an interactive environment to run code, document insights, and structure the analysis effectively.
- **Visual Studio Code** was my primary editor for writing and testing Python scripts.
- **Git and GitHub** were integral for version control, collaboration, and maintaining a transparent development history of the project.

# Data Preparation & Cleanup
- The dataset was loaded from Hugging Face (lukebarousse/data_jobs).
- Key steps for preparing and cleaning data included:
    1. Downloading and importing libraries
    2. Converting date columns to datetime objects.
    3. Extracting the job posting month from the converted datetime column.
    4. Parsing the job_skills column into lists for analysis (priorly available as a string).
```python
# Import Data & Libraries
# 1) import libraries & dataset
from datasets import load_dataset
from datetime import datetime
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import seaborn as sns
import ast
dataset = load_dataset('lukebarousse/data_jobs')
df = dataset['train'].to_pandas()

# 2) format job_posted date in datetime (it was string)
df.job_posted_date = pd.to_datetime(df.job_posted_date)

# 3) add column job posted month to convert job posted datetime
df['job_posted_month'] = df.job_posted_date.dt.month

# 4) convert 'job_skills' into list
df['job_skills'] = df['job_skills'].apply(lambda skill_list: ast.literal_eval(skill_list) if pd.notna(skill_list) else skill_list)
```
# The Analysis
This chapter provides an in-depth walkthrough of each step and file in the analysis, including code snippets and key insights derived from the data.

## 1. In which country shall I focus my analysis? 
#### Exploratory Data Analysis (EDA):
Brief introductory analysis to understand the structure of the dataset and decide where to focus the analysis. The main goal at this stage is to select a focus country, choosing according by data availabilty (how many job posting and how many salary listed) and relevance (top salaries only)

View my detailed analysis in my worbook [1_EDA_Intro](Project\1_EDA_Intro.ipynb)

```python
# Compute total job counts per country
total_counts = df['job_country'].value_counts().rename('total_count')

# Compute counts where salary_year_avg is not null
salary_counts = df[df['salary_year_avg'].notna()]['job_country'].value_counts().rename('salary_count')

# Compute the percentage of job postings with salary data per country
salary_percentage = (salary_counts / total_counts * 100).rename('salary_percentage')

# get median salary for each country
median_salary = df.groupby('job_country')['salary_year_avg'].median().rename('median_salary')

# Merge the two counts into a single DataFrame
country_counts = pd.concat([total_counts, salary_percentage, median_salary], axis=1).fillna(0)

# Keep only the top 10 countries by total number of job postings
country_counts = country_counts.head(20)

#Plotting
plt.figure(figsize=(10, 6))
sns.scatterplot(
        data=country_counts, 
        x='median_salary', 
        y='salary_percentage', 
        hue=country_counts.index, 
        palette='light:b', 
        legend=False,
)
plt.show()
```
![Fig_1.png](Assets\Fig_1.png)
*Scatter Plot visualizing the top countries by yearly salary and availabilty of salary data over the total of job postings*

#### Insights: 
The **United States was chosen** for deeper analysis due to its high job volume and reliable salary data, while countries like Sudan were excluded due to data quality concerns.

For what concerns the role, since my interest regards Data Analyst, I will mainly focus on this position only

Therefore, the filtered Data Frame will be: 
```python
df_US = df[df['job_country'] == 'United States']
```

##  2. What are the skills most in demand for the top 3 most popular data roles?
To find the most in-demand skills for the top three data jobs, I first looked at which job titles appeared the most. Then, I found the top five skills mentioned for each of these roles. This helped me see which jobs are the most popular and which skills are most important for each one.

View my notebook with detailed steps here: [2_Skill_Demand](Project/2_Skill_Demand.ipynb)

```python
import seaborn as sns

# plotting 
fig, ax = plt.subplots(len(job_titles),1) 

# for loop using enumerate function to iterate the index
for i, job_title in enumerate(job_titles):
    df_plot = df_skills_new[df_skills_new['job_title_short'] == job_title].head(5)
    sns.barplot(data=df_plot, x='skill_ratio', y='job_skills', ax=ax[i], hue='skill_count', palette='dark:b_r')

plt.show()
```
![Fig_2.png](Assets\Fig_2.png)

#### Insights:
- SQL is the most frequently required skill for Data Analyst roles (present in over 50% of postings).
- Excel and Python are also highly demanded for Data Analysts.
- For Data Scientists, Python and R are particularly prominent.

## 3. How are in-demand skills trending for Data Analysts?
#### Purpose:
Track how the demand for top skills among Data Analyst roles changes over time.
#### Methodology:
- Aggregate skill counts monthly.
- Normalize by total job postings per month.
- Visualize trends.

Detailed step by step at this link: [3_Skill_Trends](Project\3_Skill_Trends.ipynb)

```python
from matplotlib.ticker import PercentFormatter

df_plot = df_DA_US_percent.iloc[:, :5]
sns.lineplot(data=df_plot, dashes=False, legend='full', palette='tab10')

plt.gca().yaxis.set_major_formatter(PercentFormatter(decimals=0))

plt.show()
```
![Fig_3.png](Assets\Fig_3.png)
*Main Skills treding over 2023*

#### Insights:
- SQL remains the most consistently demanded skill throughout the year, although it shows a gradual decrease in demand.
- Excel experienced a significant increase in demand starting around September, surpassing both Python and Tableau by the end of the year.
- Both Python and Tableau show relatively stable demand throughout the year with some fluctuations but remain essential skills for data analysts. Power BI, while less demanded compared to the others, shows a slight upward trend towards the year's end.

## 4. How well do jobs and skills pay for Data Analysts?
To answer this question, first I compared salary distribution for Data Analyst with the distribution of the other main Data roles, leveraging box plots. Then, focusing on Data Analyst role only, I deep dived the skills associated with the highest salaries and those that are more in demand, leveraging bar chart.

View my detailed analysis here: [4_Skill_Analysis](Project\4_Skill_Analysis.ipynb)

#### - Salary Comparison
```python
sns.boxplot(data=df_US_top6, x='salary_year_avg', y='job_title_short', order=job_order)

ticks_x = plt.FuncFormatter(lambda y, pos: f'${int(y/1000)}K')
plt.gca().xaxis.set_major_formatter(ticks_x)
plt.show()
```
![Fig_4.png](Assets\Fig_4.png)
*Salary Distribution of the top 6 Data Science jobs*

#### Insights:
- **Seniority Drives Pay**: Senior roles (Senior Data Scientist, Senior Data Engineer, Senior Data Analyst) consistently show higher median salaries and overall salary ranges compared to their non-senior counterparts. In any case, Senior Data Analyst pays less than non senior Data Engineer and Data Scientist, suggesting more career potential in those fields
- **Role Differentiation**: Data Analyst roles generally have a lower salary distribution than Data Engineer and Data Scientist roles, suggesting a different market valuation for these skill sets.
- **Significant Variability**: Within each role, there's a wide range of salaries (as indicated by the box size and outliers), highlighting that factors beyond just the job title, such as experience, company size, and location, play a significant role in determining compensation.

#### - Highest paying skills & most demanded ones
```python
fig, ax = plt.subplots(2, 1)  

# Top 10 Highest Paid Skills for Data Analysts
sns.barplot(data=df_DA_top_pay, x='median', y=df_DA_top_pay.index, hue='median', ax=ax[0], palette='dark:b_r')

# Top 10 Most In-Demand Skills for Data Analystsr')
sns.barplot(data=df_DA_skills, x='median', y=df_DA_skills.index, hue='median', ax=ax[1], palette='light:b')

plt.show()
```
![Fig_5.png](Assets\Fig_5.png)
*Yearly salary per skill* 
#### Insights:
- **Specialized Skills Command High Salaries**: The top chart reveals that niche and often more technical skills like dpylr, bitbucket, gitlab, solidity, and hugging face are associated with the highest median salaries for data analysts, even if the number of positions requiring these skills might be limited.
- **Core Analytical Skills are In-Demand but Offer Moderate Pay**: The bottom chart shows that fundamental data analysis skills such as python, tableau, r, sql server, sql, sas, power bi, powerpoint, excel, and word are highly sought after (optimal skills), but their corresponding median salaries are generally lower compared to the specialized skills in the top chart. This suggests these are foundational skills with a larger supply of professionals.
- **Potential Career Growth Areas** : Data analysts looking to significantly increase their earning potential might consider acquiring one or more of the top paying specialized skills. While the demand for these might be smaller, the financial reward appears to be considerably higher than focusing solely on the core, albeit highly demanded, analytical toolkit.

## 5. What are the optimal skills for data analysts to learn?(High Demand AND High Paying)
To answer this question, I calculated the median salary associated with each skill and the % of appeareance in job postings related to Data Analyst roles in US. Moreover, I deep dived the skill type associated with those skills to identify any trend. 

Detailed calculations here: [5_Optimal_Skills](Project\5_Optimal_Skills.ipynb)
```python
from matplotlib.ticker import PercentFormatter

# Create a scatter plot
scatter = sns.scatterplot(
    data=df_DA_skills_tech_high_demand,
    x='skill_percent',
    y='median_salary',
    hue='technology',  # Color by technology
    palette='bright',  # Use a bright palette for distinct colors
    legend='full'  # Ensure the legend is shown
)
plt.show()
```
![Fig_6.png](Assets\Fig_6.png)
*Optimal Skills with related technology type*

#### Insights:
- **Core Programming and Database Skills Show Strong Presence and Good Pay**: Skills like python and sql appear frequently in job postings (high % of appearance) and offer relatively high median salaries (above $90K). This reinforces their importance as foundational skills for data analysts.
- **Visualization Tools are In-Demand with Decent Salaries**: Tableau also shows a significant presence and a median salary in the low to mid $90K range, highlighting the value placed on data visualization skills in the market.
- **Office Suite Skills are Common but Lower Paying**: Skills like excel, powerpoint, and word appear in a notable percentage of job postings but are associated with lower median salaries (below $85K). This suggests they are expected baseline skills rather than high-value differentiators in terms of compensation.

# What I learned
This project deepened my understanding of the data analyst job market and enhanced my Python technical skills, specifically in data manipulation and visualization. I learned the following:

- **Advanced Python Libraries**: to increase speed and efficiency, I utilized libraries such as *Pandas* for data manipulation and *Seaborn* and *Matplotlib* for data visualization. These tools enabled me to perform complex data analysis tasks more efficiently and effectively.
- **The importance of Data Cleansing**: I recognized that data cleaning and preparation are crucial steps to undertake before any analysis. This ensures the accuracy of insights derived from the data.
- **Strategic Skill Alignment Matters**: The project emphasized the importance of aligning one's skills with market demand. Understanding the relationship between skill demand, salary, and job availability allows for more strategic career planning in the tech industry.

# Insights
At the end of the analysis, there are three key takeways
1. **SQL, Excel, and Python are the core skills for Data Analysts**. SQL is the most frequently required skill in job postings, with Excel and Python also appearing consistently among the top demands. Mastery of these three tools forms the foundation for most data analyst positions.

2. **Python delivers the strongest combination of salary and demand**. While SQL is requested most often, Python is linked to higher median salaries and is increasingly in demand. Data Analysts who add Python to their skillset can access more opportunities and better compensation.

3. **Data visualization skills are rising in importance and value**. Tools like Tableau and Power BI are showing upward trends in job postings and are associated with competitive salaries. Developing proficiency in these tools can further differentiate candidates in a dynamic job market.

# Challenges I Faced
Three factors represented the main challenges in this project:
- Many job postings lacked salary data, limiting the scope of salary-based analysis.
- Some countries (e.g., Sudan) had unreliable or corrupted data, necessitating a focus on the U.S. market for accuracy.
- Parsing and standardizing skill names required careful data cleaning due to inconsistent formatting.

# Conclusion
To maximize employability and salary as a Data Analyst, focus on mastering SQL, Excel, and Python. These skills are both highly demanded and, in the case of Python, linked to higher salaries. Staying updated with trends in data visualization tools can further boost job prospects. The U.S. remains the most data-rich and reliable market for such analysis.