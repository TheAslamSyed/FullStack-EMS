# FullStack EMS – Employee Management System

**A full-stack MERN web application for managing a company's workforce. Admins manage employees, approve leave and generate payslips. Employees check in and out, apply for leave, and view and print their payslips. Background jobs send automatic email reminders and handle missed check-outs.**

<p align="center">
  <img src="https://img.shields.io/badge/MongoDB-Database-47A248?logo=mongodb&logoColor=white" />
  <img src="https://img.shields.io/badge/Express-5-000000?logo=express&logoColor=white" />
  <img src="https://img.shields.io/badge/React-19-61DAFB?logo=react&logoColor=black" />
  <img src="https://img.shields.io/badge/Node.js-Backend-339933?logo=node.js&logoColor=white" />
  <img src="https://img.shields.io/badge/Tailwind%20CSS-4-06B6D4?logo=tailwindcss&logoColor=white" />
  <img src="https://img.shields.io/badge/Auth-JWT-orange" />
  <img src="https://img.shields.io/badge/Jobs-Inngest-purple" />
</p>

<!-- Add a screenshot or the live demo link here, for example: ![Dashboard](assets/dashboard.png) -->

---

## Table of Contents

1. [Project Overview](#1-project-overview)
2. [Key Features](#2-key-features)
3. [Tools and Technologies](#3-tools-and-technologies)
4. [System Architecture](#4-system-architecture)
5. [User Roles and Access Control](#5-user-roles-and-access-control)
6. [Database Design](#6-database-design)
7. [Business Logic](#7-business-logic)
8. [Background Jobs and Email Automation](#8-background-jobs-and-email-automation)
9. [REST API Endpoints](#9-rest-api-endpoints)
10. [Project Structure](#10-project-structure)
11. [How to Run the Project](#11-how-to-run-the-project)
12. [Environment Variables](#12-environment-variables)
13. [Deployment](#13-deployment)
14. [Limitations and Future Improvements](#14-limitations-and-future-improvements)
15. [Author](#15-author)

---

## 1. Project Overview

Small and medium-sized companies often track attendance, leave and salaries across spreadsheets, emails and paper forms. This makes it hard to see who is present today, which leave requests are waiting, or what an employee was paid last month.

**FullStack EMS** brings these tasks into one web application with two separate portals:

- **Admin Portal:** manage employees across 10 departments, approve or reject leave, generate payslips, and see company-wide statistics.
- **Employee Portal:** check in and check out, apply for leave, view attendance history, and view or print payslips.

The application is built on the **MERN stack** (MongoDB, Express, React, Node.js). It uses **JWT authentication** with role-based access control and **Inngest** background jobs that automate reminders and missed check-outs by email.

## 2. Key Features

### Admin

| Feature | Description |
|---|---|
| Dashboard | Total employees, number of departments, today's attendance, and pending leave requests |
| Employee management | Add, edit and deactivate employees, including salary details, department, position and login credentials |
| Search and filter | Search employees by name or position and filter by department |
| Leave management | View all leave applications, filter by status, and approve or reject them |
| Payroll | Generate monthly payslips; net salary is calculated automatically |
| Soft delete | Deactivated employees keep their records but cannot check in, apply for leave or edit their profile |

### Employee

| Feature | Description |
|---|---|
| Dashboard | This month's attendance, pending leave requests, and latest payslip |
| Check-in / check-out | One click to clock in or out; working hours and day type are calculated automatically |
| Attendance history | Past attendance records with days present and late arrivals |
| Leave applications | Apply for Sick, Casual or Annual leave and track the status of each request |
| Payslips | View payslip history and print any payslip (or save it as PDF from the browser) |
| Profile settings | Update bio and change password |

### Automation

| Feature | Description |
|---|---|
| Daily attendance reminder | At 11:30 AM IST, employees who have not checked in (and are not on approved leave) get a reminder email |
| Check-out reminder | 9 hours after check-in, an employee who has not checked out gets a reminder email |
| Auto check-out | If the employee still has not checked out an hour later, the system checks them out as a Half Day, marked Late |
| Leave reminder for admin | If a leave request is still pending after 24 hours, the admin gets a reminder email |

## 3. Tools and Technologies

| Category | Tools |
|---|---|
| Frontend | React 19, Vite, React Router 7 |
| Styling | Tailwind CSS 4, Lucide React icons |
| HTTP Client | Axios (with an interceptor that attaches the JWT) |
| Notifications (UI) | React Hot Toast |
| Date Handling | date-fns |
| Backend | Node.js, Express 5 |
| Database | MongoDB, Mongoose |
| Authentication | JSON Web Tokens (JWT), bcrypt password hashing |
| Background Jobs | Inngest (event-driven functions and cron jobs) |
| Email | Nodemailer with Brevo SMTP |
| Deployment | Vercel (frontend and backend) |
| Version Control | Git, GitHub |

## 4. System Architecture

```text
┌──────────────────────────────┐         ┌──────────────────────────────────┐
│     React + Vite (client)    │  HTTPS  │     Express REST API (server)    │
│                              │ ──────► │                                  │
│  Admin Portal / Employee     │  JSON   │  protect      → verify JWT       │
│  Portal, Axios + JWT header  │ ◄────── │  protectAdmin → ADMIN role only  │
└──────────────────────────────┘         │                                  │
                                         │  Controllers: auth, employees,   │
                                         │  attendance, leave, payslips,    │
                                         │  profile, dashboard              │
                                         └───────┬───────────────┬──────────┘
                                                 │               │
                                          Mongoose          inngest.send()
                                                 │               │
                                                 ▼               ▼
                                     ┌───────────────┐   ┌──────────────────┐
                                     │   MongoDB     │◄──│ Inngest functions│
                                     │               │   │ (sleep / cron)   │
                                     └───────────────┘   └────────┬─────────┘
                                                                  │
                                                       Nodemailer (Brevo SMTP)
                                                                  │
                                                                  ▼
                                                        Reminder emails
```

## 5. User Roles and Access Control

The application has two roles: **ADMIN** and **EMPLOYEE**.

### Authentication flow

1. The user logs in through the **Admin Portal** or the **Employee Portal**.
2. The server checks that the account's role matches the portal, then compares the password with the **bcrypt** hash.
3. On success, the server returns a **JWT** (valid for 7 days) containing the user ID, role and email.
4. The client stores the token and the Axios interceptor adds it to every request as `Authorization: Bearer <token>`.

### Middleware

| Middleware | Purpose |
|---|---|
| `protect` | Verifies the JWT on every protected route and attaches the session to the request |
| `protectAdmin` | Allows the request only if the session role is `ADMIN` (returns 403 otherwise) |

The same endpoint can return different data depending on the role. For example, `GET /api/leave` returns **all** leave applications for an admin, but only the logged-in employee's own applications for an employee.

## 6. Database Design

The database has **5 MongoDB collections**.

```text
User (login account)  1 ──── 1  Employee (profile and salary)
                                      │
                    ┌─────────────────┼──────────────────┐
                    │ 1..N            │ 1..N             │ 1..N
                    ▼                 ▼                  ▼
               Attendance     LeaveApplication       Payslip
```

### User

| Field | Type | Notes |
|---|---|---|
| email | String | Required, unique |
| password | String | bcrypt hash |
| role | String | `ADMIN` or `EMPLOYEE` (default) |

### Employee

| Field | Type | Notes |
|---|---|---|
| userId | ObjectId → User | Required, unique |
| firstName, lastName | String | Required |
| email, phone, position | String | Required |
| department | String | One of 10 departments (Engineering, HR, Marketing, Sales, Finance, Operations, IT Support, Customer Success, Product Management, Design) |
| basicSalary, allowances, deductions | Number | Default 0 |
| employmentStatus | String | `ACTIVE` or `INACTIVE` |
| joinDate | Date | Required |
| isDeleted | Boolean | Soft-delete flag |
| bio | String | Editable by the employee |

### Attendance

| Field | Type | Notes |
|---|---|---|
| employeeId | ObjectId → Employee | Required |
| date | Date | The day of attendance |
| checkIn, checkOut | Date | Timestamps |
| status | String | `PRESENT`, `ABSENT` or `LATE` |
| workingHours | Number | Calculated at check-out |
| dayType | String | `Full Day`, `Three Quarter Day`, `Half Day` or `Short Day` |

A **unique compound index** on `(employeeId, date)` ensures an employee can have only one attendance record per day.

### LeaveApplication

| Field | Type | Notes |
|---|---|---|
| employeeId | ObjectId → Employee | Required |
| type | String | `SICK`, `CASUAL` or `ANNUAL` |
| startDate, endDate | Date | Required |
| reason | String | Required |
| status | String | `PENDING` (default), `APPROVED` or `REJECTED` |

### Payslip

| Field | Type | Notes |
|---|---|---|
| employeeId | ObjectId → Employee | Required |
| month, year | Number | Pay period |
| basicSalary, allowances, deductions | Number | Salary components |
| netSalary | Number | Calculated by the server |

All collections store `createdAt` and `updatedAt` timestamps automatically.

## 7. Business Logic

### Attendance: check-in and check-out

The same button handles both actions:

- **No record for today:** a new attendance record is created with the check-in time. Check-ins after 9 AM are marked **LATE**; earlier ones are **PRESENT**.
- **Checked in but not out:** the check-out time is saved and the working hours are calculated.
- **Already checked out:** the existing record is returned without changes.

| Working hours | Day type |
|---|---|
| 8 or more | Full Day |
| 6 to 8 | Three Quarter Day |
| 4 to 6 | Half Day |
| Less than 4 | Short Day |

### Leave applications

- All fields (type, start date, end date, reason) are required.
- Both dates must be **in the future**, and the end date cannot be before the start date.
- New applications start as **PENDING**. Only an admin can change the status.
- Deactivated employees cannot apply for leave.

### Payroll

```text
Net Salary = Basic Salary + Allowances − Deductions
```

The net salary is always calculated on the server, so the client cannot send a wrong value.

### Soft delete

Deleting an employee sets `isDeleted = true` and `employmentStatus = INACTIVE` instead of removing the record. Their attendance, leave and payslip history is kept for records.

## 8. Background Jobs and Email Automation

Background jobs are built with **Inngest**. Events are sent from the API, and Inngest runs the functions reliably in the background, including long waits of several hours. Emails are sent through **Nodemailer** using Brevo SMTP.

| Function | Trigger | What it does |
|---|---|---|
| `auto-check-out` | Event `employee/check-out` (sent on check-in) | Waits 9 hours. If the employee has not checked out, emails a reminder. Waits 1 more hour; if there is still no check-out, records a check-out 4 hours after check-in, sets the day type to **Half Day** and the status to **LATE**. |
| `leave-application-reminder` | Event `leave/pending` (sent when leave is applied for) | Waits 24 hours. If the application is still pending, emails the admin a reminder. |
| `attendance-reminder-cron` | Cron, every day at **11:30 AM IST** | Finds active employees who have not checked in and are not on approved leave today, and emails each of them a reminder. |

Each step of the cron job runs as a separate Inngest `step.run()`, so a failed step can be retried without repeating the earlier steps.

## 9. REST API Endpoints

Base URL: `/api`

| Method | Endpoint | Access | Description |
|---|---|---|---|
| POST | `/auth/login` | Public | Log in as admin or employee |
| GET | `/auth/session` | Logged in | Get the current session |
| POST | `/auth/change-password` | Logged in | Change password |
| GET | `/dashboard` | Logged in | Role-based dashboard statistics |
| GET | `/employees` | Admin | List employees (optional `?department=` filter) |
| POST | `/employees` | Admin | Create an employee and their login account |
| PUT | `/employees/:id` | Admin | Update an employee |
| DELETE | `/employees/:id` | Admin | Deactivate an employee (soft delete) |
| POST | `/attendance` | Logged in | Check in or check out |
| GET | `/attendance` | Logged in | Own attendance history (optional `?limit=`, default 30) |
| POST | `/leave` | Logged in | Apply for leave |
| GET | `/leave` | Logged in | Admin: all applications (optional `?status=`); employee: own applications |
| PATCH | `/leave/:id` | Admin | Approve or reject a leave application |
| POST | `/payslips` | Admin | Generate a payslip |
| GET | `/payslips` | Logged in | Admin: all payslips; employee: own payslips |
| GET | `/payslips/:id` | Logged in | Get one payslip (used for printing) |
| GET | `/profile` | Logged in | Get own profile |
| POST | `/profile` | Logged in | Update own bio |
| — | `/inngest` | Inngest | Endpoint used by Inngest to run background functions |

## 10. Project Structure

```text
FullStack-EMS/
│
├── client/                          # React frontend (Vite)
│   ├── src/
│   │   ├── api/axios.js             # Axios instance + JWT interceptor
│   │   ├── context/AuthContext.jsx  # Login state, token storage, session check
│   │   ├── pages/                   # Dashboard, Employees, Attendance, Leave,
│   │   │                            # Payslips, PrintPayslip, Settings, Login
│   │   ├── components/
│   │   │   ├── attendance/          # Check-in button, stats, history
│   │   │   ├── leave/               # Apply-leave modal, leave history
│   │   │   ├── payslip/             # Generate form, payslip list
│   │   │   └── ...                  # Dashboards, forms, sidebar, modals
│   │   ├── App.jsx                  # Routes
│   │   └── main.jsx
│   ├── vercel.json                  # SPA rewrites for Vercel
│   └── package.json
│
├── server/                          # Express backend
│   ├── config/
│   │   ├── db.js                    # MongoDB connection
│   │   └── nodemailer.js            # SMTP email sender
│   ├── constants/departments.js     # List of departments
│   ├── controllers/                 # Business logic for each resource
│   ├── inngest/index.js             # Background jobs and cron
│   ├── middleware/auth.js           # protect and protectAdmin
│   ├── models/                      # Mongoose schemas
│   ├── routes/                      # Express routers
│   ├── seed.js                      # Creates the first admin account
│   ├── server.js                    # App entry point
│   ├── vercel.json
│   └── package.json
│
└── .gitignore
```

## 11. How to Run the Project

### Prerequisites

- **Node.js** 20 or later and npm
- A **MongoDB** database (local MongoDB or a free [MongoDB Atlas](https://www.mongodb.com/atlas) cluster)
- (Optional, for emails) A free [Brevo](https://www.brevo.com/) account for SMTP credentials

### Step 1: Clone the repository

```bash
git clone https://github.com/TheAslamSyed/FullStack-EMS.git

cd FullStack-EMS
```

### Step 2: Set up the backend

```bash
cd server
npm install
```

Create a `server/.env` file (see [Environment Variables](#12-environment-variables)):

```env
MONGODB_URI=your_mongodb_connection_string
JWT_SECRET=a_long_random_secret
ADMIN_EMAIL=admin@example.com
SMTP_USER=your_brevo_smtp_login
SMTP_PASS=your_brevo_smtp_key
SENDER_EMAIL=your_verified_sender@example.com
```

### Step 3: Create the admin account

```bash
npm run seed
```

This creates an admin user with the email from `ADMIN_EMAIL` and the temporary password `admin123`. **Change this password from the Settings page after your first login.**

### Step 4: Start the backend

```bash
npm run server      # development, with auto-restart (nodemon)
# or
npm start           # production
```

The API runs at `http://localhost:4000`.

### Step 5: Start the Inngest Dev Server

In a new terminal, run the Inngest Dev Server so background jobs and reminder emails work locally:

```bash
npx inngest-cli@latest dev -u http://localhost:4000/api/inngest
```

The Inngest dashboard opens at `http://localhost:8288`, where you can see every event and function run.

### Step 6: Set up and start the frontend

```bash
cd ../client
npm install
```

Create a `client/.env` file:

```env
VITE_BASE_URL=http://localhost:4000
```

Then start the app:

```bash
npm run dev
```

Open `http://localhost:5173`, choose the **Admin Portal**, and log in with the admin account. Add employees from the **Employees** page; each employee can then log in through the **Employee Portal** with the email and password you set.

## 12. Environment Variables

### Server (`server/.env`)

| Variable | Required | Description |
|---|---|---|
| `MONGODB_URI` | Yes | MongoDB connection string |
| `JWT_SECRET` | Yes | Secret key used to sign and verify JWTs |
| `PORT` | No (default `4000`) | Port for the API server |
| `ADMIN_EMAIL` | Yes | Email of the first admin (used by `seed.js`) and recipient of leave reminders |
| `SMTP_USER` | For emails | Brevo SMTP login |
| `SMTP_PASS` | For emails | Brevo SMTP key |
| `SENDER_EMAIL` | For emails | Verified sender address for outgoing emails |
| `INNGEST_EVENT_KEY` | In production | Inngest key for sending events |
| `INNGEST_SIGNING_KEY` | In production | Inngest key for securing the `/api/inngest` endpoint |

### Client (`client/.env`)

| Variable | Required | Description |
|---|---|---|
| `VITE_BASE_URL` | No (default `http://localhost:4000`) | URL of the backend API |

**Never commit `.env` files.** They are already listed in `.gitignore`.

## 13. Deployment

Both apps include a `vercel.json` and can be deployed to **Vercel** as two separate projects:

1. **Backend:** import the repository with `server` as the root directory and add all server environment variables, including the Inngest keys.
2. **Inngest:** in the Inngest dashboard, sync your app using the deployed URL `https://<your-backend>.vercel.app/api/inngest`.
3. **Frontend:** import the repository again with `client` as the root directory, and set `VITE_BASE_URL` to the backend URL.

The client's `vercel.json` rewrites all paths to `index.html`, so React Router routes such as `/dashboard` work after a page refresh.

## 14. Limitations and Future Improvements

### Current Limitations

- **Payslip access:** `GET /api/payslips/:id` does not check that the payslip belongs to the logged-in employee, so any logged-in user who knows a payslip's ID can open it. It should allow only the owner or an admin.
- **Late check-in rule:** the check uses `hour >= 9 && minutes > 0`, so a check-in at exactly 10:00 or 11:00 is not marked late. It also uses the server's time zone, which is UTC on Vercel, not IST.
- **Average working hours** on the attendance page is a fixed value (8.5 hrs), not calculated from the records.
- **Same-day leave is not allowed**, because leave dates must be in the future. An employee cannot apply for sick leave on the day they fall sick.
- **No leave balance:** there is no limit on how many days of each leave type an employee can take.
- **No pagination** on employee, leave and payslip lists.
- The JWT is stored in `localStorage` and is valid for 7 days, with no way to revoke it early.
- The cron job does not wait for its emails to finish sending before reporting success.

### Future Improvements

- [ ] Ownership check on single payslip access
- [ ] Configurable office start time and time zone for late check-ins
- [ ] Leave balances per type, with automatic deduction on approval
- [ ] Half-day and same-day leave requests
- [ ] Calculate payslip deductions from unpaid leave and absences
- [ ] Downloadable PDF payslips generated on the server
- [ ] Monthly attendance reports and charts for the admin
- [ ] Pagination and server-side search
- [ ] Refresh tokens stored in HTTP-only cookies
- [ ] Input validation with a schema library (for example, Zod or Joi)
- [ ] Unit and API tests

## 15. Author

**Syed Abdul Aslam Shareef**

B.Tech, National Institute of Technology Warangal

- **GitHub:** [@TheAslamSyed](https://github.com/TheAslamSyed)
- **LinkedIn:** [Aslam Syed](https://www.linkedin.com/in/aslam-syed-8a9b7b303)

---

*This project demonstrates practical skills in full-stack web development with the MERN stack, REST API design, JWT authentication and role-based access control, MongoDB schema design, event-driven background jobs, email automation, and deployment.*

If you found this project useful, consider giving it a ⭐
