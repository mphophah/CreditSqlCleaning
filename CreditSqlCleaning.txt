-- Create two tables Subscribers and CreditProfiles 

CREATE TABLE Subscribers (
    SubscriberID INT PRIMARY KEY,
    First_Name VARCHAR(255),
    Last_Name VARCHAR(255),
    EmailPhysicalAddress VARCHAR(50),
    PhoneNum VARCHAR(255),
    PhysicalAddress VARCHAR(255)
);

CREATE TABLE CreditProfiles (
    CreditProfileID INT PRIMARY KEY,
    SubscriberID INT,
    SubmissionDate DATE,
    FundingAmount DECIMAL(20, 2),
    ProcessingStatus VARCHAR(50),
    FOREIGN KEY (SubscriberID) REFERENCES Subscribers(SubscriberID)
);

-- Insert data inside Subscribers table
INSERT INTO Subscribers (SubscriberID, First_Name, Last_Name, EmailPhysicalAddress, PhoneNum, PhysicalAddress)
VALUES
    (1, 'John', 'Doe', 'john.doe@gmail.com', '123-456-7890', '123 Main St'),
    (2, 'Jane', 'Smith', 'jane.smith@gmail.com', '123-774-6467', '456 Elm St');

-- Insert data inside CreditProfiles table
INSERT INTO CreditProfiles (CreditProfileID, SubscriberID, SubmissionDate, FundingAmount, ProcessingStatus)
VALUES
    (1, 1, '2023-08-01', 1500.00, 'Pending'),
    (2, 2, '2023-08-05', 2000.00, 'Approved'),
    (3, 1, '2023-08-10', 1000.00, 'Rejected');


--1.List all Subscribers along with their credit application ProcessingStatuses:

SELECT c.SubscriberID, c.First_Name, c.Last_Name, ca.ProcessingStatus
FROM Subscribers c
LEFT JOIN CreditProfiles ca ON c.SubscriberID = ca.SubscriberID;

--2. Calculate the total requested amount for each customer:

SELECT c.SubscriberID, c.First_Name, c.Last_Name, SUM(ca.FundingAmount) AS TotalFundingAmount
FROM Subscribers c
LEFT JOIN CreditProfiles ca ON c.SubscriberID = ca.SubscriberID
GROUP BY c.SubscriberID, c.First_Name, c.Last_Name;

--3. List the applications that are still pending:

SELECT *
FROM CreditProfiles
WHERE ProcessingStatus = 'Pending';

--4. Find the average requested amount across all applications:

SELECT AVG(FundingAmount) AS AverageFundingAmount
FROM CreditProfiles;

--5. Identify Subscribers with approved applications and their total requested amount:

SELECT c.SubscriberID, c.First_Name, c.Last_Name, SUM(ca.FundingAmount) AS TotalFundingAmount
FROM Subscribers c
INNER JOIN CreditProfiles ca ON c.SubscriberID = ca.SubscriberID
WHERE ca.ProcessingStatus = 'Approved'
GROUP BY c.SubscriberID, c.First_Name, c.Last_Name;

--6. List Subscribers whose applications have been rejected:

SELECT c.SubscriberID, c.First_Name, c.Last_Name
FROM Subscribers c
LEFT JOIN CreditProfiles ca ON c.SubscriberID = ca.SubscriberID
WHERE ca.ProcessingStatus = 'Rejected';

--7. Calculate the percentage of approved applications:

SELECT
    (COUNT(CASE WHEN ProcessingStatus = 'Approved' THEN 1 ELSE NULL END) * 100.0 / COUNT(*)) AS ApprovalRate
FROM CreditProfiles;

--8. Retrieve the latest credit application date for each customer:

SELECT c.SubscriberID, c.First_Name, c.Last_Name, MAX(ca.SubmissionDate) AS LatestSubmissionDate
FROM Subscribers c
LEFT JOIN CreditProfiles ca ON c.SubscriberID = ca.SubscriberID
GROUP BY c.SubscriberID, c.First_Name, c.Last_Name;

--9. List Subscribers who have multiple pending applications:

SELECT c.SubscriberID, c.First_Name, c.Last_Name, COUNT(ca.CreditProfileID) AS PendingApplications
FROM Subscribers c
LEFT JOIN CreditProfiles ca ON c.SubscriberID = ca.SubscriberID
WHERE ca.ProcessingStatus = 'Pending'
GROUP BY c.SubscriberID, c.First_Name, c.Last_Name
HAVING COUNT(ca.CreditProfileID) > 1;

--10. Calculate the average time taken to process approved applications:

SELECT
    AVG(DATEDIFF(DAY, ca.SubmissionDate, GETDATE())) AS AvgProcessingTime
FROM CreditProfiles ca
WHERE ca.ProcessingStatus = 'Approved';

--11. Find the customer with the highest requested credit amount:

SELECT c.SubscriberID, c.First_Name, c.Last_Name, MAX(ca.FundingAmount) AS HighestFundingAmount
FROM Subscribers c
INNER JOIN CreditProfiles ca ON c.SubscriberID = ca.SubscriberID
GROUP BY c.SubscriberID, c.First_Name, c.Last_Name;

--12. List Subscribers who have never applied for credit:

SELECT c.SubscriberID, c.First_Name, c.Last_Name
FROM Subscribers c
LEFT JOIN CreditProfiles ca ON c.SubscriberID = ca.SubscriberID
WHERE ca.CreditProfileID IS NULL;

--13. Calculate the total number of applications for each ProcessingStatus:

SELECT ProcessingStatus, COUNT(*) AS NumberOfApplications
FROM CreditProfiles
GROUP BY ProcessingStatus;

--14. Identify Subscribers with a total requested amount exceeding a certain threshold:

SELECT c.SubscriberID, c.First_Name, c.Last_Name, SUM(ca.FundingAmount) AS TotalFundingAmount
FROM Subscribers c
INNER JOIN CreditProfiles ca ON c.SubscriberID = ca.SubscriberID
GROUP BY c.SubscriberID, c.First_Name, c.Last_Name
HAVING SUM(ca.FundingAmount) > 2500.00;

--15. Calculate the average requested amount for approved applications:

SELECT AVG(FundingAmount) AS AvgFundingAmount
FROM CreditProfiles
WHERE ProcessingStatus = 'Approved';

--16. List Subscribers who have applied for credit more than once in the past month:

SELECT c.SubscriberID, c.First_Name, c.Last_Name, COUNT(ca.CreditProfileID) AS ApplicationsInPastMonth
FROM Subscribers c
INNER JOIN CreditProfiles ca ON c.SubscriberID = ca.SubscriberID
WHERE ca.SubmissionDate >= DATEADD(MONTH, -1, GETDATE())
GROUP BY c.SubscriberID, c.First_Name, c.Last_Name
HAVING COUNT(ca.CreditProfileID) > 1;

--17. Calculate the median requested amount across all applications:

SELECT
    PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY FundingAmount) OVER() AS MedianFundingAmount
FROM CreditProfiles;


--18. List the top 5 Subscribers with the highest total requested amount:

SELECT TOP 5 c.SubscriberID, c.First_Name, c.Last_Name, SUM(ca.FundingAmount) AS TotalFundingAmount
FROM Subscribers c
INNER JOIN CreditProfiles ca ON c.SubscriberID = ca.SubscriberID
GROUP BY c.SubscriberID, c.First_Name, c.Last_Name
ORDER BY TotalFundingAmount DESC;

--19. Calculate the average processing time for pending applications:

SELECT
    AVG(DATEDIFF(DAY, ca.SubmissionDate, GETDATE())) AS AvgPendingProcessingTime
FROM CreditProfiles ca
WHERE ca.ProcessingStatus = 'Pending';
