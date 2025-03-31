# **Project: R Shiny + Amazon RDS Integration with PostgreSQL**

## **Repo Name**: `shiny-postgres-integration`

## **Overview:**

This project demonstrates how to integrate R Shiny with Amazon RDS, using a PostgreSQL database, to create a dynamic web application that interacts with data from multiple tables. The entire process was accomplished with the help of ChatGPT/AI. Here's an overview of what was done:

- **Creating a PostgreSQL Database on Amazon RDS**: Set up the database instance, configure security groups, and create tables in Amazon RDS.
- **Data Integration**: Populated mock construction data into the PostgreSQL database using SQL scripts.
- **R Shiny Integration**: Built a Shiny web application that connects to the PostgreSQL database, queries data from multiple tables (`projects`, `workers`, `tasks`), and displays them in the application.
- **Authentication and Security**: Integrated authentication mechanisms in the Shiny app to restrict access.
  
The process was made simple using AI assistance to streamline database setup, table creation, and Shiny app development.

---

## **Steps to Setup and Configure Amazon RDS:**

### **Step 1: Create an Amazon RDS Instance (PostgreSQL)**

1. **Log in to AWS Console**:
   - Navigate to the Amazon RDS service in the AWS Management Console.

2. **Create a PostgreSQL Instance**:
   - Go to the **RDS dashboard** and click on **Create Database**.
   - Choose **PostgreSQL** as the engine.
   - Under **Settings**, provide your DB instance name (e.g., `shinyapp-db`), username (e.g., `postgres`), and password.
   - Set the **Instance Class** and **Storage** based on your needs (you can choose the free tier for testing purposes).
   - Make sure **Publicly Accessible** is set to **Yes** for remote access.

3. **Configure the Security Group**:
   - Set up a security group that allows inbound connections on port 5432 (default PostgreSQL port). Ensure your local IP or desired IP range is added to the inbound rules for secure access.

4. **Wait for the Instance to be Available**:
   - The instance will take a few minutes to be created. Once it's ready, take note of the **Endpoint** URL, which you will use as the host in your R code.

---

### **Step 2: Set Up Tables and Data in PostgreSQL**

Once the RDS instance is created and accessible, you can log into the database using a PostgreSQL client (such as pgAdmin, DBeaver, or directly from R) and create the necessary tables.

Here's the SQL code to create the `projects`, `workers`, and `tasks` tables and insert mock data into them:

#### **SQL Code to Create Tables:**

```sql
-- Create the projects table
CREATE TABLE projects (
    project_id SERIAL PRIMARY KEY,
    project_name VARCHAR(255),
    start_date DATE,
    end_date DATE
);

-- Create the workers table
CREATE TABLE workers (
    worker_id SERIAL PRIMARY KEY,
    worker_name VARCHAR(255),
    role VARCHAR(255),
    project_id INT,
    FOREIGN KEY (project_id) REFERENCES projects(project_id)
);

-- Create the tasks table
CREATE TABLE tasks (
    task_id SERIAL PRIMARY KEY,
    task_name VARCHAR(255),
    worker_id INT,
    status VARCHAR(50),
    FOREIGN KEY (worker_id) REFERENCES workers(worker_id)
);

-- Insert mock data into the projects table
INSERT INTO projects (project_name, start_date, end_date) VALUES
('Construction of Office Building', '2022-01-01', '2023-12-31'),
('Road Construction', '2022-06-01', '2023-08-01'),
('Bridge Building', '2022-09-01', '2024-02-28');

-- Insert mock data into the workers table
INSERT INTO workers (worker_name, role, project_id) VALUES
('John Doe', 'Engineer', 1),
('Jane Smith', 'Architect', 1),
('Emily Johnson', 'Project Manager', 2),
('Mike Brown', 'Engineer', 2);

-- Insert mock data into the tasks table
INSERT INTO tasks (task_name, worker_id, status) VALUES
('Design Blueprints', 1, 'Completed'),
('Foundation Construction', 2, 'In Progress'),
('Surveying the Site', 3, 'Completed'),
('Building the Structure', 4, 'In Progress');
```

---

### **Step 3: Connect PostgreSQL Database with R Shiny**

Now, set up the R Shiny app to interact with the PostgreSQL database. Here's the R code used to connect to the RDS instance and query data from the `projects`, `workers`, and `tasks` tables.

```r
# Load necessary libraries
library(shiny)
library(RPostgres)
library(DT)  # For rendering DataTable

# Set up PostgreSQL connection details
db_host <- "shinyapp-1.czckwowae2z2.us-east-2.rds.amazonaws.com"  # Your RDS instance hostname
db_port <- 5432  # Default PostgreSQL port
db_name <- "postgres"  # Your database name (replace with your actual db name if different)
db_user <- "postgres"  # Your master username
db_password <- "$M03andB1n316"  # Your master password

# Connect to the PostgreSQL database
con <- dbConnect(RPostgres::Postgres(),
                 host = db_host,
                 port = db_port,
                 dbname = db_name,
                 user = db_user,
                 password = db_password)

# Define UI for the Shiny app
ui <- fluidPage(
  titlePanel("Project Management Data from PostgreSQL"),
  
  # Display tables from multiple queries
  h3("Projects Table"),
  DTOutput("projects_table"),
  
  h3("Workers Table"),
  DTOutput("workers_table"),
  
  h3("Tasks Table"),
  DTOutput("tasks_table")
)

# Define the server logic for the Shiny app
server <- function(input, output) {
  
  # Query data from the 'projects' table
  projects_query <- "SELECT * FROM projects;"
  projects_df <- dbGetQuery(con, projects_query)
  
  # Query data from the 'workers' table
  workers_query <- "SELECT * FROM workers;"
  workers_df <- dbGetQuery(con, workers_query)
  
  # Query data from the 'tasks' table
  tasks_query <- "SELECT * FROM tasks;"
  tasks_df <- dbGetQuery(con, tasks_query)
  
  # Render the projects table
  output$projects_table <- renderDT({
    projects_df
  })
  
  # Render the workers table
  output$workers_table <- renderDT({
    workers_df
  })
  
  # Render the tasks table
  output$tasks_table <- renderDT({
    tasks_df
  })
}

# Run the application
shinyApp(ui = ui, server = server)

# Disconnect from the database when the app is closed
onStop(function() {
  dbDisconnect(con)
})
```

---

## **Why This Matters**

Integrating R Shiny with Amazon RDS enables businesses and data analysts to build interactive, web-based data applications that can dynamically query and display real-time data from cloud databases. In this project, using Amazon RDS provides a scalable, managed relational database service for handling large amounts of construction project data. The ability to integrate such applications with PostgreSQL allows for seamless data access, and the R Shiny app ensures users can interact with the data in a meaningful and visually appealing way.

This combination of technologies streamlines the data analysis and visualization process, empowering stakeholders to make data-driven decisions, track project progress, and monitor workers and tasks in real time. This approach is particularly valuable in fields such as construction, where tracking various aspects of a project’s progress is crucial for success.

---

## **Conclusion**

In this project, we:
- Set up an Amazon RDS PostgreSQL instance to host mock construction data.
- Created tables (`projects`, `workers`, `tasks`) and populated them with sample data using SQL.
- Developed an R Shiny application that connects to the PostgreSQL database to query and display the data.
- Demonstrated how AI (ChatGPT) can help streamline the process of setting up and integrating databases, making it easier to build data-driven applications.

This project showcases how the integration of cloud technologies (like AWS RDS) and open-source data science tools (like R Shiny) can help solve real-world business problems by providing dynamic and interactive data-driven solutions.

---

## **Suggested Repository Name**: `shiny-postgres-integration`

This guide can be used as the README for the repository, allowing others to replicate and learn from the process of integrating R Shiny with Amazon RDS for data visualization.
