# Hospital Database Design & Implementation

A comprehensive MySQL database schema for hospital management, designed to handle patient records, doctor schedules, appointments, treatments, and billing.

## 📊 Project Overview

This project demonstrates database design and SQL implementation for a healthcare management system. The database supports core hospital operations, including patient registration, appointment scheduling, treatment tracking, and billing.

## 🎯 Project Objectives

- Design a normalized relational database for hospital operations
- Implement entity relationships following healthcare data standards
- Create an efficient schema for patient care workflow
- Demonstrate SQL DDL (Data Definition Language) skills

## 🗄️ Database Schema

### Tables Overview

**Core Entities:**
1. **Patients** - Patient demographics and contact information
2. **Doctors** - Medical staff details and specializations
3. **Appointments** - Scheduling system linking patients and doctors
4. **Treatments** - Medical procedures and interventions
5. **Prescriptions** - Medication records
6. **Billing** - Financial transactions and insurance claims
7. **Departments** - Hospital organizational structure
8. **Rooms** - Facility management and bed assignments

### Key Relationships

- **Patients ↔ Appointments** (One-to-Many)
- **Doctors ↔ Appointments** (One-to-Many)
- **Appointments ↔ Treatments** (One-to-Many)
- **Departments ↔ Doctors** (One-to-Many)
- **Patients ↔ Billing** (One-to-Many)

## 🛠️ Technical Implementation

### Database Features

- **Normalization:** 3rd Normal Form (3NF) to reduce redundancy
- **Primary Keys:** Auto-incrementing IDs for all entities
- **Foreign Keys:** Enforced referential integrity
- **Data Types:** Optimized for healthcare data (VARCHAR, DATE, DECIMAL)
- **Constraints:** NOT NULL, UNIQUE where applicable

### Sample Table Structure
```sql
CREATE TABLE Patients (
    PatientID INT PRIMARY KEY AUTO_INCREMENT,
    FirstName VARCHAR(50) NOT NULL,
    LastName VARCHAR(50) NOT NULL,
    DateOfBirth DATE NOT NULL,
    Gender ENUM('M', 'F', 'Other'),
    Phone VARCHAR(15),
    Email VARCHAR(100),
    Address VARCHAR(255),
    InsuranceProvider VARCHAR(100),
    InsuranceNumber VARCHAR(50)
);

CREATE TABLE Doctors (
    DoctorID INT PRIMARY KEY AUTO_INCREMENT,
    FirstName VARCHAR(50) NOT NULL,
    LastName VARCHAR(50) NOT NULL,
    Specialization VARCHAR(100),
    Phone VARCHAR(15),
    Email VARCHAR(100),
    DepartmentID INT,
    FOREIGN KEY (DepartmentID) REFERENCES Departments(DepartmentID)
);

CREATE TABLE Appointments (
    AppointmentID INT PRIMARY KEY AUTO_INCREMENT,
    PatientID INT,
    DoctorID INT,
    AppointmentDate DATE NOT NULL,
    AppointmentTime TIME NOT NULL,
    Status ENUM('Scheduled', 'Completed', 'Cancelled') DEFAULT 'Scheduled',
    Notes TEXT,
    FOREIGN KEY (PatientID) REFERENCES Patients(PatientID),
    FOREIGN KEY (DoctorID) REFERENCES Doctors(DoctorID)
);
```

## 📈 Sample Queries & Use Cases

### 1. Patient Appointment History
```sql
SELECT 
    p.FirstName, 
    p.LastName, 
    a.AppointmentDate, 
    d.FirstName AS DoctorFirstName,
    d.LastName AS DoctorLastName,
    d.Specialization
FROM Patients p
JOIN Appointments a ON p.PatientID = a.PatientID
JOIN Doctors d ON a.DoctorID = d.DoctorID
WHERE p.PatientID = 1
ORDER BY a.AppointmentDate DESC;
```

### 2. Doctor Schedule for a Specific Date
```sql
SELECT 
    d.FirstName,
    d.LastName,
    a.AppointmentTime,
    p.FirstName AS PatientFirstName,
    p.LastName AS PatientLastName,
    a.Status
FROM Doctors d
JOIN Appointments a ON d.DoctorID = a.DoctorID
JOIN Patients p ON a.PatientID = p.PatientID
WHERE d.DoctorID = 5 
  AND a.AppointmentDate = '2024-11-20'
ORDER BY a.AppointmentTime;
```

### 3. Department Workload Analysis
```sql
SELECT 
    dep.DepartmentName,
    COUNT(DISTINCT d.DoctorID) AS TotalDoctors,
    COUNT(a.AppointmentID) AS TotalAppointments
FROM Departments dep
LEFT JOIN Doctors d ON dep.DepartmentID = d.DepartmentID
LEFT JOIN Appointments a ON d.DoctorID = a.DoctorID
GROUP BY dep.DepartmentName
ORDER BY TotalAppointments DESC;
```

### 4. Billing Summary by Patient
```sql
SELECT 
    p.FirstName,
    p.LastName,
    SUM(b.TotalAmount) AS TotalBilled,
    SUM(b.AmountPaid) AS TotalPaid,
    SUM(b.TotalAmount - b.AmountPaid) AS Outstanding
FROM Patients p
JOIN Billing b ON p.. PatientID = b.PatientID
GROUP BY p.PatientID
HAVING Outstanding > 0;
```

## 💡 Key Insights & Analytics

**Database can answer:**
- ✅ How many appointments does each doctor have per day/week?
- ✅ What's the average patient wait time between appointment request and scheduled date?
- ✅ Which departments have the highest patient volume?
- ✅ What's the total outstanding billing amount?
- ✅ Which treatments are most frequently prescribed?
- ✅ Patient retention rates and appointment completion rates

## 🎓 Skills Demonstrated

- **Database Design:** ER modeling, normalization, schema design
- **SQL DDL:** CREATE TABLE, ALTER TABLE, constraints
- **SQL DML:** INSERT, UPDATE, DELETE operations
- **SQL Queries:** JOIN operations, aggregations, subqueries
- **Healthcare Domain:** Understanding of medical data structures
- **Data Integrity:** Primary/foreign key relationships, constraints

## 🚀 Future Enhancements

- [ ] Add stored procedures for common operations
- [ ] Implement triggers for audit logging
- [ ] Create views for frequently accessed data
- [ ] Add indexes for query optimization
- [ ] Implement role-based access control
- [ ] Add table for medical history and lab results
- [ ] Create reporting dashboard queries

## 📁 Repository Contents

- `hospital_schema` - Complete database schema
- `sample_data.sql` - Sample INSERT statements
- `queries.sql` - Example analytical queries
- `README.md` - This documentation

## 🔗 Related Projects

- [Healthcare Analytics Dashboard](https://Himansh-11.github.io/healthcare-dashboard) - Power BI visualization using similar healthcare data

## 📧 Contact

**Himansh Rajak**  
Data Analyst | SQL | Healthcare Analytics  
📧 himanshr1107@gmail.com  
💼 [LinkedIn](https://linkedin.com/in/himansh-rajak)  
🌐 [Portfolio](https://Himansh-11.github.io)

---

**Technologies:** MySQL, SQL, Database Design, Healthcare Data Modeling
