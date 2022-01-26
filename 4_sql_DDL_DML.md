# DDL and DML commands
Below are Data Definition Language (DDL) and Data Manipulation Language (DML) commands used to create 11 tables and fill them by data generated

## DDL
```
CREATE TABLE "Login" (
"Username" varchar(10) not null PRIMARY KEY,
"Password" varchar(20) not null );


CREATE TABLE "Trainer" (
"Trainer_ID" integer not null PRIMARY KEY,
"Username" varchar(10) not null,
"Email" varchar(30) not null,
"Address" varchar(30) not null,
"Gender" boolean not null,
"Salary" integer not null,
"Y_birth" date not null,
"Acc_nr" varchar(30) not null,
"F_name" varchar(10) not null,
"L_name" varchar(10) not null);


CREATE TABLE "Member" (
"Member_ID" integer not null PRIMARY KEY,
"Username" varchar(10) not null,
"Email" varchar(30) not null,
"Address" varchar(30) not null,
"Gender" boolean not null,
"Y_birth" date not null,
"F_name" varchar(10) not null,
"L_name" varchar(10) not null);


CREATE TABLE "Time" (
"Time_ID" integer not null PRIMARY KEY,
"Username" varchar(10) not null,
"Time" timestamp not null );


CREATE TABLE "Announcements" (
"Ann_ID" integer not null PRIMARY KEY,
"Username" varchar(10) not null,
"Description" varchar(255) not null,
"Date" timestamp not null );


CREATE TABLE "Receivables" (
"Received_ID" integer not null PRIMARY KEY,
"Membership_ID" integer not null,
"Member_ID" integer not null,
"Date" timestamp not null,
"Received_amount" integer not null);


CREATE TABLE "Memberships" (
"Membership_ID" integer not null PRIMARY KEY,
"Type" smallint not null,
"Price" integer not null );


CREATE TABLE "Rooms" (
"Room_ID" integer not null PRIMARY KEY,
"Name" varchar(20) not null,
"Size" smallint not null,
"Availability" boolean not null );


CREATE TABLE "Plan" (
"Plan_ID" integer not null PRIMARY KEY,
"Member_ID" integer not null,
"Workout_ID" integer not null);


CREATE TABLE "Workouts" (
"Workout_ID" integer not null PRIMARY KEY,
"Trainer_ID" integer not null,
"Room_ID" integer not null,
"Max_attendance" smallint not null,
"Type" smallint not null,
"Start_time" timestamp not null,
"End_time" timestamp not null);


CREATE TABLE "Payables" (
"Paid_ID" integer not null PRIMARY KEY,
"Workout_ID" integer not null,
"Date" timestamp not null,
"Amount_paid" integer not null);
```


## DML

`COPY "Login"`
`FROM 'Login.csv'`
`DELIMITER ','`
`CSV HEADER;`

`COPY "Trainer"`
`FROM 'Trainer.csv'`
`DELIMITER ','`
`CSV HEADER;`

`COPY "Member"`
`FROM 'Member.csv'`
`DELIMITER ','`
`CSV HEADER;`

`COPY "Time"`
`FROM 'Time.csv'`
`DELIMITER ','`
`CSV HEADER;`

`COPY "Announcements"`
`FROM 'Announcement.csv'`
`DELIMITER ','`
`CSV HEADER;`

`COPY "Plan"`
`FROM 'Plan.csv'`
`DELIMITER ','`
`CSV HEADER;`

`COPY "Workouts"`
`FROM 'Workout.csv'`
`DELIMITER ','`
`CSV HEADER;`

`COPY "Receivables"`
`FROM 'Receivables.csv'`
`DELIMITER ','`
`CSV HEADER;`

`COPY "Payables"`
`FROM 'Payables.csv'`
`DELIMITER ','`
`CSV HEADER;`

`INSERT INTO "Memberships" ("Membership_ID", "Type","Price")`
`VALUES `
`	(1,3,30),`
`	(2,1,10),`
`	(3,2,20),`
`	(4,4,40),`
`	(5,5,50);`

`INSERT INTO "Rooms" ("Room_ID","Name","Size","Availability")`
`VALUES `
`	(1,'Nmloch',50,true),`
`	(2,'Mfzzje',200,true),`
`	(3,'Sjtlfn',50,true),`
`	(4,'Nbhopy',100,true),`
`	(5,'Jvquwr',30,false);`