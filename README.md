# 🏥 Hospital Database Management System

A normalized relational database for hospital operations with advanced SQL analytics demonstrating database design, complex queries, and healthcare data management.

![Database Schema](screenshots/er_diagram.png)

## 📊 Project Overview

This project showcases database design and SQL query optimization for healthcare data management. Built with SQLite, it includes patient management, appointment scheduling, billing systems, and treatment tracking with proper normalization and referential integrity.

**Key Metrics:**
- **10 normalized tables** following 3rd Normal Form (3NF)
- **237 records** of realistic healthcare data
- **9 foreign key relationships** ensuring data integrity
- **4 complex SQL queries** demonstrating advanced techniques

**Live Demo:** [View SQL Project](https://himansh-11.github.io/hospital-database-sql/)

---

## 🗄️ Database Schema

### Entity-Relationship Diagram

![ER Diagram](screenshots/er_diagram.png)

### Database Structure

The database consists of 10 interconnected tables:

| Table | Records | Description |
|-------|---------|-------------|
| **departments** | 7 | Hospital organizational structure (Cardiology, Emergency, Pediatrics, etc.) |
| **doctors** | 15 | Physician information including specialty and contact details |
| **docinfo** | 15 | Extended doctor credentials, licenses, and education history |
| **salary** | 38 | Compensation records with historical tracking |
| **patients** | 45 | Patient demographics, insurance, and health status |
| **appointments** | 43 | Scheduling with status tracking (Completed, Cancelled, No-Show) |
| **treatments** | 15 | Medical procedures and interventions (ECG, MRI, Surgery, etc.) |
| **prescriptions** | 10 | Medication records with dosage and frequency |
| **billing** | 39 | Financial transactions (Paid: 75%, Pending: 15%, Overdue: 10%) |
| **rooms** | 10 | Facility management and bed assignments |

### Key Relationships

- **Departments → Doctors** (1:N): One department has many doctors
- **Doctors → DocInfo** (1:1): Each doctor has detailed credentials
- **Doctors → Salary** (1:N): Salary history tracking over time
- **Doctors → Patients** (1:N): One doctor treats many patients
- **Patients → Appointments** (1:N): Patients schedule multiple visits
- **Appointments → Treatments/Prescriptions/Billing** (1:N): Each visit generates multiple records

---

## 💡 SQL Queries & Analysis

### Query 1: Doctor Workload Analysis

**Business Question:** Which doctors have the highest patient loads and revenue generation for capacity planning?

```sql
SELECT 
    d.DoctorID,
    d.FirstName || ' ' || d.LastName AS DoctorName,
    d.Specialty,
    dept.DepartmentName,
    COUNT(DISTINCT p.PatientID) AS TotalPatients,
    COUNT(DISTINCT a.AppointmentID) AS TotalAppointments,
    ROUND(AVG(b.TotalAmount), 2) AS AvgBilling,
    SUM(b.TotalAmount) AS TotalRevenue
FROM doctors d
LEFT JOIN departments dept ON d.DepartmentID = dept.DepartmentID
LEFT JOIN patients p ON d.DoctorID = p.DoctorID
LEFT JOIN appointments a ON d.DoctorID = a.DoctorID
LEFT JOIN billing b ON a.AppointmentID = b.AppointmentID
GROUP BY d.DoctorID, d.FirstName, d.LastName, d.Specialty, dept.DepartmentName
ORDER BY TotalRevenue DESC;
```

**Skills Demonstrated:** Multi-table JOINs (4 tables), aggregate functions, GROUP BY, NULL handling

**Key Insight:** Dr. Williams (Orthopedics) generates the highest revenue ($41,200) with surgical procedures, while Dr. Johnson (Pediatrics) maintains the highest patient volume (4 patients) with lower per-visit costs, revealing different specialization models requiring distinct resource allocation strategies.

![Query 1 Results](screenshots/query1_results.png)

---

### Query 2: Patient Status Distribution

**Business Question:** What's the distribution of patient health statuses and their appointment completion rates?

```sql
SELECT 
    p.PatientStatus,
    COUNT(DISTINCT p.PatientID) AS NumPatients,
    ROUND(COUNT(DISTINCT p.PatientID) * 100.0 / 
          SUM(COUNT(DISTINCT p.PatientID)) OVER (), 2) AS PercentageOfTotal,
    COUNT(DISTINCT a.AppointmentID) AS TotalAppointments,
    ROUND(CAST(COUNT(DISTINCT a.AppointmentID) AS FLOAT) / 
          COUNT(DISTINCT p.PatientID), 2) AS AvgAppointmentsPerPatient,
    SUM(CASE WHEN a.AppointmentStatus = 'Completed' THEN 1 ELSE 0 END) AS CompletedAppointments,
    SUM(CASE WHEN a.AppointmentStatus = 'Cancelled' THEN 1 ELSE 0 END) AS CancelledAppointments
FROM patients p
LEFT JOIN appointments a ON p.PatientID = a.PatientID
GROUP BY p.PatientStatus
ORDER BY NumPatients DESC;
```

**Skills Demonstrated:** Window functions (SUM OVER), CASE statements, conditional aggregation, percentage calculations

**Key Insight:** Critical patients show 100% appointment completion (no cancellations), while Stable patients and Recovering Patients have 94.12% and 93.75% completion rates, respectively, suggesting higher engagement when health stakes are perceived as higher.

![Query 2 Results](screenshots/query2_results.png)

---

### Query 3: Department Financial Performance

**Business Question:** Which departments should receive investment based on revenue performance and collection efficiency?

```sql
WITH DepartmentMetrics AS (
    SELECT 
        dept.DepartmentName,
        dept.DepartmentHead,
        COUNT(DISTINCT d.DoctorID) AS NumDoctors,
        COUNT(DISTINCT a.AppointmentID) AS TotalAppointments,
        SUM(b.TotalAmount) AS TotalRevenue,
        ROUND(AVG(b.TotalAmount), 2) AS AvgRevenuePerAppointment,
        SUM(CASE WHEN b.PaymentStatus = 'Paid' THEN b.TotalAmount ELSE 0 END) AS CollectedRevenue
    FROM departments dept
    LEFT JOIN doctors d ON dept.DepartmentID = d.DepartmentID
    LEFT JOIN appointments a ON d.DoctorID = a.DoctorID
    LEFT JOIN billing b ON a.AppointmentID = b.AppointmentID
    GROUP BY dept.DepartmentName, dept.DepartmentHead
)
SELECT 
    DepartmentName,
    DepartmentHead,
    NumDoctors,
    TotalAppointments,
    TotalRevenue,
    AvgRevenuePerAppointment,
    CollectedRevenue,
    ROUND(CollectedRevenue * 100.0 / NULLIF(TotalRevenue, 0), 2) AS CollectionRate
FROM DepartmentMetrics
WHERE TotalRevenue > 0
ORDER BY TotalRevenue DESC;
```

**Skills Demonstrated:** Common Table Expressions (CTEs), window functions, complex aggregations, financial metrics

**Key Insight:** Orthopedics leads with $12,600 total revenue and 100% collection rate, while Oncology shows $20,600 revenue but only a 66.5% collection rate, indicating a need for improved billing processes. Emergency Medicine handles the highest volume (5 appointments) but lower per-visit revenue ($440), suggesting different business models requiring tailored strategies.

![Query 3 Results](screenshots/query3_results.png)

---

### Query 4: High-Value Patient Analysis

**Business Question:** Who are the highest-spending patients and what treatments drive their costs?

```sql
SELECT 
    p.PatientID,
    p.FirstName || ' ' || p.LastName AS PatientName,
    p.PatientStatus,
    d.FirstName || ' ' || d.LastName AS AssignedDoctor,
    d.Specialty,
    COUNT(DISTINCT a.AppointmentID) AS TotalAppointments,
    COUNT(DISTINCT t.TreatmentID) AS TotalTreatments,
    SUM(b.TotalAmount) AS TotalSpending,
    ROUND(AVG(b.TotalAmount), 2) AS AvgSpendingPerVisit,
    MIN(a.AppointmentDate) AS FirstVisit,
    MAX(a.AppointmentDate) AS LastVisit
FROM patients p
LEFT JOIN doctors d ON p.DoctorID = d.DoctorID
LEFT JOIN appointments a ON p.PatientID = a.PatientID
LEFT JOIN treatments t ON a.AppointmentID = t.AppointmentID
LEFT JOIN billing b ON a.AppointmentID = b.AppointmentID
GROUP BY p.PatientID, p.FirstName, p.LastName, p.PatientStatus, 
         d.FirstName, d.LastName, d.Specialty
HAVING SUM(b.TotalAmount) > 0
ORDER BY TotalSpending DESC
LIMIT 15;
```

**Skills Demonstrated:** Complex multi-table JOINs (6 tables), HAVING clause, date functions, TOP N analysis

**Key Insight:** Top patient Christopher Jackson (Oncology) spent $19,400 over 4 appointments with chemotherapy treatments, while Richard Gonzalez (Orthopedics) spent $12,050 on surgical procedures.

![Query 4 Results](screenshots/query4_results.png)

---

## 🛠️ Technical Implementation

### Database Design Principles

- **Normalization:** All tables follow 3rd Normal Form (3NF) to eliminate data redundancy
- **Referential Integrity:** Foreign key constraints ensure data consistency across related tables
- **Data Types:** Optimized column types (VARCHAR, INT, DATE, DECIMAL) for storage efficiency
- **Indexing:** Primary keys on all tables for query performance
- **Constraints:** NOT NULL and UNIQUE constraints where appropriate

### SQL Techniques Demonstrated

✅ **Join Operations:** INNER JOIN, LEFT JOIN, multiple table joins (up to 6 tables)  
✅ **Aggregate Functions:** COUNT, SUM, AVG, MIN, MAX with DISTINCT  
✅ **Window Functions:** SUM() OVER(), RANK(), DENSE_RANK()  
✅ **Common Table Expressions (CTEs):** For complex query organization  
✅ **Conditional Logic:** CASE statements, NULLIF, COALESCE  
✅ **Subqueries:** Nested SELECT statements  
✅ **Date Functions:** Date arithmetic and formatting  
✅ **String Operations:** Concatenation with ||  
✅ **GROUP BY & HAVING:** Data aggregation and filtering  

### Technology Stack

- **Database:** SQLite 3.x (portable, no server required)
- **Tools:** DB Browser for SQLite
- **Data Format:** CSV files for easy portability
- **Version Control:** Git & GitHub

---

## 🚀 Getting Started

### Prerequisites

- DB Browser for SQLite ([Download](https://sqlitebrowser.org/dl/))
- Or any SQLite-compatible database tool

### Installation & Setup

**Option 1: Use Pre-built Database (Recommended)**

1. Clone the repository
```bash
git clone https://github.com/himansh-11/hospital-database-sql.git
cd hospital-database-sql
```

2. Open `hospital.db` in DB Browser for SQLite

3. Navigate to "Execute SQL" tab and run queries from `/sql_queries` folder

**Option 2: Build from CSV Files**

1. Download all CSV files from `/data` folder

2. Open DB Browser for SQLite → Create New Database

3. Import each CSV file:
   - File → Import → Table from CSV file
   - Select CSV, name table (lowercase), check "Column names in first line"
   - Repeat for all 10 tables

4. Run the queries!

---

## 📸 Sample Results

### Database Schema
![Database Structure](screenshots/database_schema.png)

### Query Results
![Doctor Workload Analysis](screenshots/query1_results.png)
![Patient Distribution](screenshots/query2_results.png)
![Department Performance](screenshots/query3_results.png)
![High-Value Patients](screenshots/query4_results.png)

---

## 💼 Business Applications

This database supports critical healthcare operations:

- **Patient Management:** Track demographics, medical history, and care continuity
- **Appointment Scheduling:** Optimize doctor availability and reduce no-shows
- **Financial Operations:** Monitor revenue, track receivables, and analyze profitability
- **Resource Allocation:** Identify capacity constraints and workload imbalances
- **Quality Metrics:** Analyze appointment completion rates and patient engagement
- **Strategic Planning:** Guide department investment and hiring decisions

### Real-World Use Cases

1. **Capacity Planning:** Identify overloaded doctors needing support or redistribution
2. **Revenue Optimization:** Target departments with low collection rates for process improvement
3. **Patient Retention:** Analyze high-value patients for VIP care programs
4. **Operational Efficiency:** Compare appointment completion rates across patient groups
5. **Financial Forecasting:** Track historical salary trends for budget planning

---

## 📊 Data Highlights

### Financial Metrics
- **Total Revenue:** $45,280 across all departments
- **Average Bill:** $1,161 per appointment
- **Collection Rate:** 87% overall (varies by department)
- **Payment Distribution:** 75% Paid, 15% Pending, 10% Overdue

### Operational Metrics
- **Appointment Completion:** 88% overall
- **No-Show Rate:** 2.3%
- **Average Patient Load:** 3 patients per doctor
- **Treatment Complexity:** Ranges from $45 (A1C test) to $8,500 (surgery)

### Healthcare Providers
- **15 doctors** across 7 specialties
- **Salary Range:** $238K - $385K annually
- **Experience:** Average 10.5 years, range 6-15 years
- **Top Medical Schools:** Johns Hopkins, Harvard, Stanford, Mayo Clinic

---

## 🎯 Project Goals

This project demonstrates:

1. **Database Design Expertise:** Proper normalization, relationship modeling, and constraint implementation
2. **Advanced SQL Skills:** Complex queries using joins, CTEs, window functions, and aggregations
3. **Healthcare Domain Knowledge:** Understanding of medical workflows, billing processes, and operational metrics
4. **Business Analysis:** Translating data into actionable insights for strategic decision-making
5. **Data Quality:** Creating realistic, consistent datasets with proper relationships


---

## 📫 Contact

**Himansh**  
Data Specialist | Data Analytics | SQL & Database Design

- **Portfolio:** [himansh-11.github.io/healthcare-analytics-portfolio](https://himansh-11.github.io/healthcare-analytics-portfolio/)
- **LinkedIn:** [linkedin.com/in/himansh11](https://linkedin.com/in/himansh-rajak)
- **GitHub:** [github.com/himansh-11](https://github.com/himansh-11)
- **Email:** himanshr1107@gmail.com

---

## 📄 License

This project is available for educational and portfolio purposes. All data is synthetic and created specifically for this demonstration.

---

## 🙏 Acknowledgments

- Database design follows industry best practices for healthcare data management
- Inspired by real-world hospital information systems (HIS) architecture
- Data structures aligned with HIPAA compliance principles for proper data separation

---

*Last Updated: November 2024*
