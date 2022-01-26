# SQL query-based analysis
Below is some analysis and demonstration of various query techniques created for learning purposes.

### Which workouts have booked a room too small with respect to workout max_attendance? (JOIN)
```
SELECT "Workout_ID","Max_attendance","Size" room_size FROM
(SELECT "Workout_ID","Max_attendance","Room_ID" FROM "Workouts") wo
JOIN
(SELECT "Room_ID","Size" FROM "Rooms") ro
ON wo."Room_ID"=ro."Room_ID"
WHERE "Max_attendance">"Size"/5
```
### Which workouts have booked the same rooms at the same time? (JOIN, ORDER)
```
SELECT "Workout_ID",wo."Room_ID","Start_time" FROM
(SELECT "Workout_ID","Room_ID","Start_time" FROM "Workouts") wo
JOIN
(SELECT "Room_ID" FROM "Rooms") ro
ON wo."Room_ID"=ro."Room_ID"
ORDER BY "Start_time","Room_ID"
```
### For a given time slot some workouts have booked the same room. How many workouts need rebooking? (JOIN, GROUP, aggregation)
```
SELECT COUNT(ro."Room_ID")-COUNT(DISTINCT ro."Room_ID") need_rebooking,"Start_time" FROM
(SELECT "Workout_ID","Room_ID","Start_time" FROM "Workouts") wo
JOIN
(SELECT "Room_ID" FROM "Rooms") ro
ON wo."Room_ID"=ro."Room_ID"
GROUP BY "Start_time"
```
### Which workouts have registered members larger than max_attendance  (JOIN, WHERE, common table expressions (CTE))
```
WITH plan AS
(SELECT "Workout_ID", COUNT("Member_ID") registered FROM "Plan"
GROUP BY "Workout_ID"),
wouts AS
(SELECT "Workout_ID","Max_attendance" FROM "Workouts")
SELECT pl."Workout_ID","Max_attendance",registered FROM plan pl
JOIN
wouts wo
ON pl."Workout_ID"=wo."Workout_ID"
WHERE "Max_attendance"<registered
```
### Identify members with simultaneous workouts (HAVING)
```
SELECT "Member_ID",COUNT(*) simult_workouts,"Start_time" FROM
(SELECT "Workout_ID", "Member_ID" FROM "Plan") pl
JOIN
(SELECT "Workout_ID","Start_time" FROM "Workouts") wo
ON pl."Workout_ID"=wo."Workout_ID"
GROUP BY "Member_ID","Start_time"
HAVING COUNT(*)>1
ORDER BY "Member_ID"
```
### How often particular member (Member_ID=321) comes to workouts and individual trainings (View, user defined function (UDF), Window Function)
```
Create View to show selected member times of entering (0) and leaving (1) the gym
CREATE VIEW mem_plan AS
SELECT "Member_ID", "Time", (ROW_NUMBER() OVER (ORDER BY "Time")-1)%2 AS enters_leaves FROM
(SELECT "Username","Member_ID" FROM "Member") me
JOIN
(SELECT "Username","Time" FROM "Time") ti
ON me."Username"=ti."Username"
WHERE "Member_ID"=321
ORDER BY "Time"
```
We modify this view to show entering and leaving times as separate columns
```
CREATE VIEW mem_entries AS
SELECT enters, leaves FROM
(
SELECT "Time" enters, ROW_NUMBER() OVER (ORDER BY "Time") AS i FROM mem_plan
WHERE enters_leaves=0
) entering
JOIN
(
SELECT "Time" leaves, ROW_NUMBER() OVER (ORDER BY "Time") AS i FROM mem_plan
WHERE enters_leaves=1) leaving
ON entering.i=leaving.i
```

Next we create the function to identify whether interval between member enters and leaves the gym does include interval from wstart till wend
CREATE OR REPLACE function came_for_workout(wstart timestamp, wend timestamp) 
RETURNS int LANGUAGE plpgsql 
AS
$$ 
DECLARE
wtimes int; 
BEGIN 
 SELECT COUNT(*) INTO wtimes FROM mem_entries
 WHERE enters<wstart AND leaves>wend;  
 RETURN wtimes; 
END; 
$$; 

By applying the function we see that Member was in gym for individual trainings not workouts
SELECT "Start_time","End_time",came_for_workout("Start_time","End_time") FROM
(SELECT * FROM "Plan"
WHERE "Member_ID"=321) pl
JOIN
(SELECT * FROM "Workouts") wo
ON pl."Workout_ID"=wo."Workout_ID"

To drop views created
DROP VIEW mem_entries
DROP VIEW mem_plan

###  Human-readable data on members and trainers (UNION)
SELECT "F_name","L_name","Email", "Address","Gender","Y_birth" FROM "Member"
UNION
SELECT "F_name","L_name","Email", "Address","Gender","Y_birth" FROM "Trainer"

### How much each member has to pay extra to cover all his memberships? (UDF)
Create the function to show the price for given membership_ID
CREATE OR REPLACE function membership_price(mem_id int) 
RETURNS int LANGUAGE plpgsql 
AS
$$ 
DECLARE
pr int; 
BEGIN 
 SELECT "Price" INTO pr FROM "Memberships"
 WHERE "Membership_ID"=mem_id;  
 RETURN pr; 
END; 
$$; 

By applying this function we obtain debts(+) and overpayments(-) of the members
SELECT "Member_ID",SUM(membership_price("Membership_ID"))-SUM("Received_amount") AS needs_to_be_paid FROM "Receivables"
GROUP BY "Member_ID"

### Dynamically change member sum of payments when new payment arrived (Trigger, View)
From view paid_table (previous task) we create toy table paid_table_beginning to play with
CREATE VIEW paid_table AS
SELECT "Member_ID",SUM(membership_price("Membership_ID"))-SUM("Received_amount") AS needs_to_be_paid FROM "Receivables"
GROUP BY "Member_ID"
ORDER BY "Member_ID"

CREATE TABLE paid_table_beginning AS 
SELECT * FROM paid_table
LIMIT 5

The trigger function below decreases the value in the field needs_to_be_paid in our toy table for the member who made payment
CREATE OR REPLACE FUNCTION member_paid()
 RETURNS TRIGGER 
 LANGUAGE PLPGSQL
 AS
$$
BEGIN
	UPDATE paid_table_beginning
	SET needs_to_be_paid=needs_to_be_paid-NEW."Received_amount"
	WHERE "Member_ID"=NEW."Member_ID";
	RETURN NEW;
END;
$$

CREATE TRIGGER something_paid
  BEFORE INSERT
  ON "Receivables"
  FOR EACH ROW
  EXECUTE PROCEDURE member_paid();

Trigger invokes function  member_paid  if something is inserted into table “Receivables”. For instance, 
INSERT INTO "Receivables" 
("Received_ID","Membership_ID","Member_ID","Date","Received_amount")
VALUES 
	(10001,5,1,'2022-01-07 01:00:00',20);

Member with Member_ID=1 pays 20. Due to the trigger, corresponding value in the toy table changes automatically, see 
SELECT * FROM paid_table_beginning

To remove view, table and row created do
DROP VIEW paid_table
DROP TABLE paid_table_beginning

DELETE FROM "Receivables"
WHERE "Received_ID"=10001

### How many visits were done by the member after the last payment? Can be used to identify whether member’s membership is expired or not (Indexes, UDF, CTE, subquery)
CREATE OR REPLACE function mem_last_pay(mem_id int) 
RETURNS timestamp LANGUAGE plpgsql 
AS
$$ 
DECLARE
lp timestamp ; 
BEGIN 
   SELECT MAX("Date") INTO lp FROM "Receivables"
   WHERE "Member_ID"=mem_id;  
   RETURN lp; 
END; 
$$; 

This function finds member’s last payment timestamp in the “Receivables” table. For all members, it executes for 865ms, see
EXPLAIN ANALYZE SELECT "Member_ID",mem_last_pay("Member_ID") FROM "Member"

CREATE INDEX idx_rec_date 
ON "Receivables"("Date")

With index created, the execution time is reduced to 151 ms, see
EXPLAIN ANALYZE SELECT "Member_ID",mem_last_pay("Member_ID") FROM "Member"

We use subquery to show number of gym visits after last payment of the member with Member_ID=15

SELECT COUNT("Time") FROM "Time"
WHERE "Username" = (
SELECT "Username" FROM "Member"
WHERE "Member_ID"=15)
AND "Time">=mem_last_pay(15)

To removes index created do
DROP INDEX idx_rec_date