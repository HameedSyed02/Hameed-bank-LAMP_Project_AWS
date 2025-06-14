
# 💳 LAMP Stack Banking App on Amazon EC2

A simple banking application running on a **LAMP stack** hosted on an **Amazon EC2 (Amazon Linux)** instance.  
Features include:
- Secure **login page**
- **Balance check** feature from a MariaDB database
- Apache serving PHP web pages
- GitHub-ready project with troubleshooting notes

---

## 🏗️ Project Architecture

```
Client (Browser)
      ↓
Apache (Web Server on EC2)
      ↓
PHP (Processes login & balance logic)
      ↓
MariaDB (Stores user & balance data)
```

---

## 🚀 Step-by-Step Deployment Guide

### 1. 🔧 Launch EC2 Instance
- AMI: **Amazon Linux 2** or **Amazon Linux 2023**
- Security Group: Allow ports **22 (SSH), 80 (HTTP), 443 (HTTPS)**

### 2. 🖥️ SSH Into EC2
```bash
ssh -i your-key.pem ec2-user@your-ec2-public-ip
```

### 3. 🧱 Install LAMP Stack
```bash
# Update packages
sudo yum update -y

# Install Apache
sudo yum install -y httpd
sudo systemctl start httpd
sudo systemctl enable httpd

# Install PHP
sudo amazon-linux-extras enable php8.2
sudo yum clean metadata
sudo yum install -y php php-mysqlnd

# Install MariaDB (Amazon Linux 2)
sudo yum install -y mariadb105-server mariadb105
sudo systemctl start mariadb
sudo systemctl enable mariadb
```

### 4. 🛠️ Configure Database
```bash
# Secure installation
sudo mysql_secure_installation

# Login to MariaDB
mysql -u root -p
```

```sql
CREATE DATABASE bankapp;
USE bankapp;

CREATE TABLE users (
  id INT AUTO_INCREMENT PRIMARY KEY,
  username VARCHAR(50) UNIQUE,
  password VARCHAR(255),
  balance DECIMAL(10,2)
);

-- Insert test user (password is hashed in PHP)
INSERT INTO users (username, password, balance)
VALUES ('user1', SHA2('password123', 256), 1500.00);

-- Create limited user (optional)
CREATE USER 'bankuser'@'localhost' IDENTIFIED BY 'strongpass';
GRANT SELECT, INSERT ON bankapp.* TO 'bankuser'@'localhost';
FLUSH PRIVILEGES;
```

### 5. 🌐 Deploy PHP Files
```bash
cd /var/www/html
sudo rm -f index.html
sudo nano login.php
```

Paste this minimal login code:

```php
<?php
session_start();
$conn = new mysqli("localhost", "root", "your_password", "bankapp");
if ($conn->connect_error) die("Connection failed.");

if ($_SERVER["REQUEST_METHOD"] == "POST") {
    $user = $_POST["username"];
    $pass = hash('sha256', $_POST["password"]);

    $sql = "SELECT * FROM users WHERE username='$user' AND password='$pass'";
    $result = $conn->query($sql);

    if ($result->num_rows == 1) {
        $_SESSION["username"] = $user;
        header("Location: balance.php");
    } else {
        echo "Invalid login";
    }
}
?>

<form method="post">
    Username: <input name="username"><br>
    Password: <input type="password" name="password"><br>
    <button type="submit">Login</button>
</form>
```

Then create `balance.php`:
```bash
sudo nano balance.php
```

```php
<?php
session_start();
if (!isset($_SESSION["username"])) {
    header("Location: login.php"); exit;
}

$conn = new mysqli("localhost", "root", "your_password", "bankapp");
$user = $_SESSION["username"];
$result = $conn->query("SELECT balance FROM users WHERE username='$user'");
$row = $result->fetch_assoc();
echo "<h1>Welcome, $user</h1>";
echo "<p>Your Balance: $" . $row["balance"] . "</p>";
echo '<a href="logout.php">Logout</a>';
?>
```

Then create `logout.php`:
```php
<?php
session_start();
session_destroy();
header("Location: login.php");
?>
```

### 6. 🔐 Set Permissions
```bash
sudo chown -R apache:apache /var/www/html
sudo chmod -R 755 /var/www/html
```

### 7. 🌍 Access the App
Visit:  
```
http://<your-ec2-public-ip>/login.php
```

---

## ✅ Features

- 🔐 Secure session-based login
- 💰 Balance fetch via database
- 🛡️ Enforced basic security (password hashing, input check)
- 🌐 Apache + PHP + MariaDB stack
- 📦 Ready for WAF, Cloudflare, Grafana integrations

---

## 🧪 Sample Test User
| Username | Password     | Balance |
|----------|--------------|---------|
| user1    | password123  | $1500   |

---

## 🚨 Troubleshooting Summary

| Problem                             | Cause                       | Fix                                                                                   |
|-------------------------------------|-----------------------------|----------------------------------------------------------------------------------------|
| `mkdir: no such file or directory` | Missing Loki folders        | `sudo mkdir -p /tmp/loki/{index,cache,chunks,compactor}`                              |
| `schema v13 is required`           | Outdated schema in config   | Set `schema: v13` under `schema_config:` in `/etc/loki-config.yaml`                   |
| `curl localhost:3100/ready` → not ready | Loki crashed or not running | Run `cat /var/log/loki.log` to see error logs                                         |
| No logs showing in Grafana         | Promtail misconfigured      | Verify Promtail config and run `cat /var/log/promtail.log`                            |
| Apache log path not found          | OS uses different log path  | Use `/var/log/httpd/access_log` (Amazon Linux) or `/var/log/apache2/access.log` (Ubuntu) |
| Grafana can't connect to Loki      | Loki URL incorrect or down  | Set URL to `http://localhost:3100` in Grafana’s Loki data source settings             |

---

## 📂 Project Structure

```
/var/www/html/
├── login.php
├── balance.php
└── logout.php
```

---

## 📘 Next Steps

- [ ] Add HTTPS with Let's Encrypt
- [ ] Connect with Cloudflare & WAF
- [ ] Monitor logs using Promtail + Loki + Grafana
- [ ] Add 2FA, user registration, and password reset
- [ ] Host code on GitHub with CI/CD

---

## 📜 License

MIT License. Free to use, modify, and share.
