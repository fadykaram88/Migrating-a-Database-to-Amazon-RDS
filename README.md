
# Project 7 – Migrating a Database to Amazon RDS
![Migrating Database to RDS Architecture](https://github.com/fadykaram88/Migrating-a-Database-to-Amazon-RDS/blob/main/m6ch-lab-end-arch.png?raw=true)

## Task 1: Create RDS Instance for the Cafe App

1. In the AWS Console, search for **RDS** > click **Create database**
   - **Engine options**: MariaDB
   - **Templates**: Dev/Test

2. **Settings**:
   - **DB instance identifier**: `CafeDatabase`
   - **Master username**: `admin`
   - **Credentials management**: Self managed
   - **Master password**: `Caf3DbPassw0rd!`  
   ⚠️ *Important: Save the password, you'll need it later.*

3. **Instance Configuration**:
   - **DB instance class**: Burstable classes (`db.t3.micro`)

4. **Storage**:
   - **Type**: General Purpose SSD (gp2)
   - **Allocated Storage**: 20 GB

5. **Availability & Durability**:
   - Do NOT create a standby instance

6. **Connectivity**:
   - **VPC**: `Lab VPC`
   - **DB subnet group**: `lab-db-subnet-group`
   - **Public Access**: `No`
   - **VPC Security Group**: Choose existing group `dbSG` (not the default)

   📝 *Need help? Review Project 3 for creating VPC and Security Group*

   - **Availability Zone**: Choose one ending with `(a)`

7. **Additional Configuration**:
   - **Database port**: `3306`

8. **Monitoring**:
   - Disable Enhanced Monitoring

9. Click **Create database**

---

## Task 2: Analyze the Existing Cafe App Deployment

10. A pre-created EC2 instance `CafeServer` is running the app

11. Test the app in your browser:  
   `http://3.84.143.82/cafe`

12. The page shows past orders (data stored in local DB).  
    You will now migrate this to Amazon RDS.

13. Connect to EC2 using **Session Manager**:
    - Go to EC2 instance `CafeServer`
    - Click **Connect** > **Session Manager** > **Connect**

14. In terminal, run:
```bash
sudo su
su ec2-user
whoami
cd /home/ec2-user/
```

📝 *Note: Use Amazon Linux 2 AMI and attach IAM policy `AmazonSSMManagedInstanceCore` to use Session Manager.*

---

## Task 3: Export Data from Old DB & Connect to RDS

15. Check local MariaDB service:
```bash
service mariadb status
mysql --version
```

16. In AWS Console, go to **Secrets Manager** > secret `/cafe/dbpassword`  
    - Click **Retrieve secret value**
    - Copy the plaintext password

17. Back in terminal, connect to local DB:
```bash
mysql -u root -p
```
Enter the password from the secret. Then run:
```sql
show databases;
use cafe_db;
show tables;
select * from `order`;
exit;
```

18. Dump the database:
```bash
mysqldump -u root -p cafe_db > CafeDbDump.sql
```
Verify:
```bash
ls
cat CafeDbDump.sql
```

---

## Task 4: Import Data into RDS

19. Ensure RDS status is **Available**

20. In AWS Console, open the RDS instance > copy **Endpoint** and **Port**

21. Back in terminal:
```bash
mysql -u admin -p --host <rds-endpoint>
```
Replace `<rds-endpoint>` with actual value (e.g. `cafedatabase.xxxxxx.rds.amazonaws.com`)  
Enter password: `Caf3DbPassw0rd!`

⚠️ *If connection fails: check security group inbound rules. Do NOT make it public — allow only specific SG sources.*

22. Run:
```sql
show databases;
exit;
```

---

## Task 5: Import the Data to RDS Instance

23. Run:
```bash
mysql -u admin -p --host <rds-endpoint> < CafeDbDump.sql
```

24. Verify data:
```bash
mysql -u admin -p --host <rds-endpoint>
```
Then:
```sql
show databases;
use cafe_db;
show tables;
select * from `order`;
exit;
```

---

## Task 6: Link Cafe App to New Database

25. Stop local MariaDB to disconnect the app from EC2:
```bash
sudo service mariadb stop
```

---

## ✅ Project Complete

You have successfully migrated your application database to Amazon RDS.  
👏 Great job finishing a medium-difficulty project!

See you in the next one!
