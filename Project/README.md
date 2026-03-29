
# 🩸  Blood Bank

A web-based blood bank management system built with PHP, Apache, and MySQL (AWS RDS). Supports donor registration, blood group search, user authentication, and admin management.

---

##  Table of Contents

- [Prerequisites](#prerequisites)
- [Server Setup](#server-setup)
- [Database Setup](#database-setup)
- [Configuration](#configuration)
- [Git Workflow](#git-workflow)
- [CI/CD with Jenkins](#cicd-with-jenkins)
- [CloudWatch Logging](#cloudwatch-logging)
- [Useful SQL Commands](#useful-sql-commands)
- [Important Notes](#important-notes)

---

## Prerequisites

- Ubuntu EC2 Instance
- AWS RDS MySQL Instance
- Git

---

## Server Setup

```bash
# Update packages
sudo apt-get update -y

# Install Apache
sudo apt-get install apache2 -y
sudo systemctl restart apache2
sudo systemctl enable apache2

# Install PHP and required extensions
sudo apt-get install php libapache2-mod-php php-mysql php-curl php-gd php-json php-zip php-mbstring -y

# Install MySQL client
sudo apt-get install mysql-server -y
```

---

## Database Setup

### Connect to RDS

```bash
mysql -h <your-rds-endpoint> -u admin -p
```

### Create Database

```sql
CREATE DATABASE customers;
USE customers;
```

### Create Tables

**Donors Table**
```sql
CREATE TABLE donors (
  id          INT AUTO_INCREMENT PRIMARY KEY,
  fname       VARCHAR(255) NOT NULL,
  lname       VARCHAR(255) NOT NULL,
  mobileno    BIGINT UNIQUE,
  city        VARCHAR(255) NOT NULL,
  bfrom       DATE,
  bto         DATE,
  dob         DATE,
  bloodgroup  VARCHAR(255) NOT NULL
);
```

**Users Table**
```sql
CREATE TABLE users (
  username  VARCHAR(80) NOT NULL,
  name      VARCHAR(80) NOT NULL,
  password  VARCHAR(80) NOT NULL
) ENGINE=InnoDB DEFAULT CHARSET=latin1;
```

**Admin Table**
```sql
CREATE TABLE admin (
  username  VARCHAR(80) NOT NULL,
  name      VARCHAR(80) NOT NULL,
  password  VARCHAR(80) NOT NULL
) ENGINE=InnoDB DEFAULT CHARSET=latin1;
```

### Insert Sample Data

**Donors**
```sql
INSERT INTO donors (fname, lname, mobileno, city, bfrom, bto, dob, bloodgroup) VALUES
  
  ('Srinivas',  'Thota',    '9812723411', 'Mumbai',    '2022-04-19', '2022-10-07', '1992-07-22', 'B_Positive'),
  ('Zaheer',    'Khan',     '7788678987', 'Chennai',   '2022-09-11', '2022-12-19', '1998-11-11', 'A_Positive');
```

**Users**
```sql
INSERT INTO users (username, name, password) VALUES
  
  ('prashanth',  'Prashanth Katkam',   '12345'),
  ('vijay',      'Vijay Mourya',       '12345');
```

**Admin**
```sql
INSERT INTO admin (username, name, password) VALUES ('admin', 'admin', '12345');
```

### Grant Permissions

```sql
GRANT ALL PRIVILEGES ON customers.* TO 'root'@'%' IDENTIFIED BY 'admin123';
FLUSH PRIVILEGES;
```

---

## Configuration

Update the **RDS endpoint** in the following files:

| File | Purpose |
|------|---------|
| `config.php` | Global DB connection config |
| `donate-blood.php` | Donor registration |
| `find-donor.php` | Donor search |
| `search.php` | Search functionality |
| `signup.php` | User signup |
| `deletedata.php` | Delete records |

> ⚠️ **Important:**
> - Add the **donors** table name to `index.php`
> - Add the **admin** table name to `indexadmin.php`
> - If the database or table name changes, update all referenced files accordingly

### RDS Connectivity

If the EC2 instance cannot connect to the RDS database, add an **inbound rule** to the RDS Security Group:
- **Type:** Aurora / MySQL
- **Source:** Security Group of the EC2 instance

---

## Git Workflow

### Clone Repository

```bash
# Clone default branch
git clone <repo-url>

# Clone a specific branch
git clone --branch <branch-name> <repo-url>
```

### Push Changes

```bash
sudo git init
sudo git remote add origin "https://github.com/prashanthkatam/Hackathon.git"
sudo git remote -v
sudo git add .
sudo git commit -m "your commit message"
sudo git push origin master
```

### Update Remote URL (with token)

```bash
git remote set-url origin https://<your-token>@github.com/prashanthkatam/Hackathon.git
```

---

## CI/CD with Jenkins

```bash
# Update and install Java
sudo apt-get update
sudo apt-get install openjdk-8-jdk

# Add Jenkins repo and install
wget -q -O - https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo apt-key add -
sudo sh -c 'echo deb https://pkg.jenkins.io/debian-stable binary/ > /etc/apt/sources.list.d/jenkins.list'
sudo apt-get update
sudo apt-get install jenkins

# Install Git
sudo apt install git
```

---

## CloudWatch Logging

Push Apache access logs to AWS CloudWatch for monitoring.

### Steps

1. **Create an EC2 Instance**
2. **Attach IAM Role** with `CloudWatchAgentServerPolicy`
3. **Install dependencies**

```bash
sudo apt-get update
sudo apt-get install apache2 -y

# Download and install CloudWatch Agent
sudo wget https://s3.amazonaws.com/amazoncloudwatch-agent/ubuntu/amd64/latest/amazon-cloudwatch-agent.deb
sudo dpkg -i -E ./amazon-cloudwatch-agent.deb
```

4. **Create agent config**

```bash
sudo vi /opt/aws/amazon-cloudwatch-agent/bin/config.json
```

```json
{
  "agent": {
    "run_as_user": "root"
  },
  "logs": {
    "logs_collected": {
      "files": {
        "collect_list": [
          {
            "file_path": "/var/log/apache2/access.log",
            "log_group_name": "myapache-error-log",
            "log_stream_name": "{instance_id}"
          }
        ]
      }
    }
  }
}
```

5. **Start the CloudWatch Agent**

```bash
sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl \
  -a fetch-config \
  -m ec2 \
  -c file:/opt/aws/amazon-cloudwatch-agent/bin/config.json \
  -s
```

6. In the **AWS Console**, navigate to **CloudWatch → Log Groups** to view logs under the group name specified in `config.json`.

> ⚠️ Double-check `config.json` for correct bracket syntax and ensure the `CloudWatchAgentServerPolicy` is attached to the instance.

---

## Useful SQL Commands

```sql
-- View all columns in a table
SHOW COLUMNS FROM donors;

-- Delete all rows from a table
DELETE FROM donors;

-- Show all databases
SHOW DATABASES;

-- Use a specific database
USE customers;
```

---

## Important Notes

- Always update the **DB endpoint** in `config.php` and any file that connects to the database.
- The **Hackathon repo** contains the latest version of the code.
- Verify **Security Group rules** if database connectivity fails from EC2.
- Changes pushed to GitHub will be automatically reflected via the CI/CD pipeline.
