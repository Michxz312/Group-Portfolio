---
jupyter:
  jupytext:
    text_representation:
      extension: .md
      format_name: markdown
      format_version: '1.3'
      jupytext_version: 1.16.1
  kernelspec:
    display_name: Python 3 (ipykernel)
    language: python
    name: python3
---

# MATH 441 Group 5 Project

**Aziz, Mika, Spock**


## Project Title: Employee Scheduling for a Financial Institution

**Problem Statement:**

How can we optimize the scheduling of employees in a financial institution to minimize labor costs while ensuring customer demand is met and considering employee preferences and skill levels?

**Relevant Real-world Examples:**

* Study employee scheduling practices in banks or financial institutions.
* Explore existing optimization algorithms applied to employee scheduling problems.

**Data and Computations:**

 Data:
* Employee availability and preferences.
* Historical customer demand patterns.
* Employee skill levels and qualifications.
* Labor costs for different time slots.

**Note**: In search of a suitable dataset to apply this optimization on, we will attempt to find relevant employee scheduling information from local organizations like banks, investment firms and other financial institutions. There are of course some difficulties when it comes to obtaining said data:
1. Companies might not release such data publicly
2. Needed data might not be formatted as expected
3. A lack of data in general on employee scheduling

We will attempt to remedy this through a few methods, such as reaching out to companies to obtain anonymous data, generate data based on a few known paramater distributions etc.

 Computations:
* Formulate the scheduling problem as an optimization model, considering constraints such as employee availability, customer demand, and skill requirements.
* Try to explore and implement optimization algorithms in order to find the optimal schedule.
* Validate the model using real or simulated data.


**Parameters:**

* There are $t$ time slots in a day $k = 0, \dots, t-1$.
* There are $d$ days in the scheduling period $l = 0, \dots, d-1$.
* There are $n$ employees $i = 0, \dots, n-1$.
* There are $s$ types of shifts $j = 0, \dots, s-1$. The shifts starts from 9am to 5pm so $s=8$.
* $P = [p_{ij}]$ where $p_{ij}$ is the preference score of employee $i$ for shift type $j$.

**Decision variables:** Let $x_{ikl} \in {0,1}$ be a binary variable, where $x_{ikl} = 1$ if employee $i$ is scheduled to work on day $l$ at time $k$, and 0 otherwise.

**Objective:** Minimize the total labor cost:
$$
\sum_l \sum_k \sum_i \sum_j p_{ij}\cdot x_{ikl}
$$

**Constraints:**

Each employee is scheduled exactly once per day:
$$
\sum_k x_{ikl} = 1 \ , \ \ \text{for each} \ i,l 
$$

All shifts must be covered:
$$
\sum_i \sum_l x_{ikl}  \geq  \text{Demand}_{jk} \ , \ \ \text{for each} \ k,j 
$$

Employee availability constraint:
$$
x_{ikl} = 0 \ , \ \ \text{if $i$ is not available at time $k$ on day $l$}
$$

Consistency with employee preferences:
$$
\sum_k p_{ij}\cdot x_{ikl} \geq \text{Preference}_{il} \ , \ \ \text{for each} \ i,l
$$

No overlapping shifts for employees:
$$
\sum_j x_{ikl} \leq 1 \ , \ \ \text{for each} \ i,k,l
$$






### Modifying initial formula


#### Simplified Scheduling problem

Objective: 
Minimize the total labor cost:
- $C_ {i,j}$ = cost for assigning an employee i to job j
- $X_{i,j}$ = $\begin{cases} 
          1 & \text{if employee i works at job j} \\
          0 & \text{otherwise}
        \end{cases}$
$$ \text{objective :} \displaystyle \min \sum_i \sum_j C_{i,j} X_{i,j} $$

constraints: 
- Each employee is schedule exactly once a day:
$$ \sum_j X_{i,j} = 1 $$

- Each job gets assigned to an employee: 
$$ \sum_i X_{i,j} = 1 $$

(source) https://web.mit.edu/15.053/www/AMP-Chapter-09.pdf

<!-- #region -->
### To Do List:

**Defining the types of shift S and what skill levels are needed**

1. Account opening - entry 
2. Credit card application - entry 
3. Loan Application - junior 
4. Mortgage Consultation - senior 
5. retirement planning - senior
6. financial advising - senior
7. wealth management - manager

**Defining employee skill levels**

* 0 - entry-level
* 1 - junior
* 2 - senior
* 3 - manager

**Constraints**

- An employee with entry-level skill cannot do a shift that requires junior level or higher
- An employee with junior-level skill cannot do a shift that requires senior level or higher
- An employee with senior-level skill cannot do a shift that requires manager level

**Defining labor cost or each shift types** 
1. Account opening - \$17 
2. Credit card application - \$17 
3. Loan Application - \$18 
4. Mortgage Consultation - \$25 
5. retirement planning - \$27
6. financial advising - \$30
7. wealth management - \$37

**Updating the formulation to include matching of shift S and the employee's skill level**


**Generate example**
<!-- #endregion -->

## Data


Before we can begin with the sovling of the LP problem at the core of our project, we have to obtain the appropriate data. Obtaining the data for such a project is qutie difficult. So what we will resolve to do is generate the data ourselves.


Row is an employee, where first column is their tier (0-3, junior, manager etc), second column is their vector of preferences. If they're unauthorized to do a specific task, that preference will be a 0.


TODO: Assumptions section


TODO: Come back to availablity discussion (hourly vs daily etc)

```python
import numpy as np
import random
import csv

def generate_employee_data_custom_distribution(num_employees, skill_levels, tasks_with_min_levels, distribution):
    employees_data = []
    
    # Calculate the number of employees in each skill level based on the distribution
    num_employees_distribution = {level: int(pct * num_employees) for level, pct in distribution.items()}
    
    # Adjust for any rounding differences to ensure the total count matches num_employees
    while sum(num_employees_distribution.values()) < num_employees:
        num_employees_distribution[random.choice(list(num_employees_distribution.keys()))] += 1
        
    # Generate data for each employee based on the distribution
    for skill_level_label, count in num_employees_distribution.items():
        skill_level = skill_levels[skill_level_label]
        for _ in range(count):
            task_preferences = []
            for task, min_level in tasks_with_min_levels.items():
                if skill_level >= min_level:
                    preference_level = random.randint(1, 5)  # Assuming preference levels range from 1 to 5
                else:
                    preference_level = 0
                task_preferences.append(preference_level)
            employees_data.append((skill_level_label, task_preferences))
    
    # Shuffle the data to mix skill levels
    random.shuffle(employees_data)
    
    return employees_data

# Example usage
skill_levels = {"entry-level": 0, "junior": 1, "senior": 2, "manager": 3}
tasks_with_min_levels = {
    "Account opening": 0,
    "Credit card application": 0,
    "Loan Application": 1,
    "Mortgage Consultation": 2,
    "Retirement planning": 2,
    "Financial advising": 2,
    "Wealth management": 3
}

distribution = {
    "entry-level": 0.4,
    "junior": 0.3,
    "senior": 0.2,
    "manager": 0.1
}

num_employees = 100
employees_data = generate_employee_data_custom_distribution(num_employees, skill_levels, tasks_with_min_levels, distribution)

# Output the first and last five employees
first_five = employees_data[:5]
last_five = employees_data[-5:]

print("First five employees:")
for i, (skill, preferences) in enumerate(first_five):
    print(f"Employee {i+1}: Skill Level - {skill}, Task Preferences - {preferences}")

print("\nLast five employees:")
for i, (skill, preferences) in enumerate(last_five):
    print(f"Employee {i+1}: Skill Level - {skill}, Task Preferences - {preferences}")

# Save to CSV file
csv_file_path = "employees_data.csv"
with open(csv_file_path, mode='w', newline='') as file:
    writer = csv.writer(file)
    writer.writerow(["Skill Level", "Task Preferences"])
    for skill, preferences in employees_data:
        writer.writerow([skill, preferences])

```
