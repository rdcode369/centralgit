## 📦 Project Title: Deploying PHP MySQL Web App on EC2 with Apache (EC2 LAMP Stack Deployment)

This project shows how to set up a simple web app on an EC2 Linux server using Apache2 and MySQL. It includes installing the server, database, and getting everything working together to run your PHP app.

## 📌 Step 1: Set Up an EC2 Instance

1. **Log in to AWS Console**

   - Go to the [AWS Management Console](https://aws.amazon.com/console/).
   - Navigate to the **EC2** service.

2. **Launch an EC2 Instance**

   - Click **Launch Instance**.
   - Choose an Amazon Machine Image (AMI), such as **Amazon Linux**.
   - Select an instance type (e.g., `t2.micro` for free tier).
   - Configure instance details (default settings are fine).
   - Add storage (default 8GB is sufficient).
   - Add tags (optional).
   - Configure the security group:
     - Allow **HTTP (80)**, **HTTPS (443)**, **SSH (22)**, and **MySQL (3306)**.
   - Review and launch the instance.
   - Create and download a key pair (`webserver-db-amazon-key.pem`) for SSH access.

3. **Connect to the EC2 Instance**

   ```bash
   cd Downloads
   chmod 400 webserver.pem
   ssh -i "webserver-db-amazon-key.pem" ec2-user@<your-ec2-public-ip>
   ```

## 📌 Step 2: Install Apache2, PHP, and MySQL

1. **Update the System**

   ```bash
   # for Amazon Linux
   sudo dnf update -y
   sudo dnf upgrade -y
   sudo passwd

   # for Ubuntu Linux
   sudo apt update
   sudo apt upgrade -y
   sudo -i passwd
   ```

2. **Install PHP**

   ```bash
   # for Amazon Linux
   sudo dnf install -y php-fpm php-mysqli php-json php-devel mariadb105-server

   # for Ubuntu Linux
   sudo apt install php libapache2-mod-php php-mysql -y
   # or
   sudo apt install -y php-fpm php-mysql php-json php-dev mariadb-server
   ```

3. **Install Apache2**

   ```bash
   # for Amazon Linux
   sudo dnf install httpd -y

   # for Ubuntu Linux
   sudo apt install apache2 -y
   ```

4. **Start and Enable Services**

   ```bash
   # for Amazon Linux
   sudo systemctl start httpd
   sudo systemctl enable httpd
   sudo systemctl status httpd

   sudo systemctl start mariadbmariadbmariadb
   sudo systemctl enable mariadbmariadb
   sudo systemctl status mariadb

   # for Ubuntu Linux
   sudo systemctl start apache2
   sudo systemctl enable apache2
   sudo systemctl status apache2

   sudo systemctl start mysql
   sudo systemctl enable mysql
   sudo systemctl status mysql
   ```

5. **Secure MySQL Installation**

   ```bash
   sudo mysql_secure_installation
   ```
   
6. **Check MySQL (MariaDB) version**

   ```bash
   mysql --version
   ```
7. **Ensure PHP is set up with Apache**
	* Amazon Linux 2023 uses php-fpm by default. Make sure both services are running:

	```bash
   sudo systemctl start php-fpm
   sudo systemctl enable php-fpm
   ```
8. **Verify Apache Installation**

   * Open browser: `http://<your-ec2-public-ip>`
   * You should see Apache's default landing page.

## 📌 Step 3: Set Up MySQL Database

1. **Log in to MySQL**

   ```bash
   sudo mysql -u root -p
   ```

2. **Create a Database and User**

   ```sql
   CREATE DATABASE myproject;
   CREATE USER 'myuser'@'localhost' IDENTIFIED BY 'mypassword';
   GRANT ALL PRIVILEGES ON myproject.* TO 'myuser'@'localhost';
   FLUSH PRIVILEGES;
   ```

3. **Create a Table**

   ```sql
   USE myproject;
   CREATE TABLE users (
       id INT AUTO_INCREMENT PRIMARY KEY,
       name VARCHAR(50) NOT NULL,
       email VARCHAR(100) NOT NULL,
	   country VARCHAR(100) NOT NULL
   );
   EXIT;
   ```


## 📌 Step 4: Deploy a PHP Web Application

1. **Create Project Directory**

   ```bash
   sudo mkdir /var/www/html/myproject
   sudo chown -R $USER:$USER /var/www/html/myproject
   sudo chmod 755 /var/www/html/
   cd /var/www/html
   sudo rm index.html
   sudo touch index.php
   sudo nano index.php
   ```

2. **Add PHP Code**

   Paste the following code into `index.php`:

   ```php
   <?php
    // Enable error reporting
    ini_set('display_errors', 1);
    ini_set('display_startup_errors', 1);
    error_reporting(E_ALL);
    
    // Database connection
    $servername = "localhost";
    $username = "myuser";
    $password = "mypassword";
    $dbname = "myproject";
    $conn = new mysqli($servername, $username, $password, $dbname);
    if ($conn->connect_error) die("Connection failed: " . $conn->connect_error);
    
    // Status message
    $status = $_GET['status'] ?? null;

    // Handle Add User
    if ($_SERVER["REQUEST_METHOD"] == "POST" && isset($_POST['submit'])) {
        $name = $_POST['name'];
        $email = $_POST['email'];
        $country = $_POST['country'];

    $stmt = $conn->prepare("INSERT INTO users (name, email, country) VALUES (?, ?, ?)");
    $stmt->bind_param("sss", $name, $email, $country);
    $stmt->execute();
    $stmt->close();

    header("Location: " . $_SERVER['PHP_SELF'] . "?status=added");
    exit();
    }
    ?>
    
    <!DOCTYPE html>
    <html lang="en">
    <head>
        <meta charset="UTF-8">
        <title>My EC2 Project</title>
        <meta name="viewport" content="width=device-width, initial-scale=1.0">
        <style>
            body {
                background-color: #1e1e2f;
                color: #f0f0f0;
                font-family: 'Segoe UI', sans-serif;
                margin: 0;
                padding: 20px;
            }

        h1 {
            text-align: center;
            color: #66ccff;
        }

        .flex-container {
            display: flex;
            flex-wrap: wrap;
            justify-content: center;
            align-items: flex-start;
            gap: 20px;
            margin: 20px auto;
            max-width: 1200px;
        }

        .form-box, .message-box {
            background-color: #2e2e40;
            border-radius: 8px;
            padding: 20px;
            box-shadow: 0 4px 10px rgba(0,0,0,0.4);
        }

        .form-box {
            flex: 1 1 300px;
            max-width: 500px;
        }

        .message-box {
            flex: 1 1 200px;
            max-width: 400px;
            color: #bafac7;
            background-color: #2e5731;
            border-left: 5px solid #00cc66;
            opacity: 1;
            transition: opacity 1s ease-in-out;
        }

        .message-box.hidden {
            opacity: 0;
        }

        .form-box h2 {
            color: #66ccff;
            margin-bottom: 20px;
            border-bottom: 1px solid #444;
            padding-bottom: 10px;
        }

        input[type="text"], input[type="email"] {
            width: 96%;
            padding: 10px;
            margin: 10px 0 20px;
            border: 1px solid #555;
            border-radius: 4px;
            background-color: #1a1a2a;
            color: #fff;
        }

        input[type="submit"] {
            background-color: #66ccff;
            color: #1e1e2f;
            padding: 10px 20px;
            border: none;
            border-radius: 4px;
            font-weight: bold;
            cursor: pointer;
        }

        input[type="submit"]:hover {
            background-color: #4db8e6;
        }

        .table-box {
            background-color: #2e2e40;
            border-radius: 8px;
            padding: 20px;
            margin: 20px auto;
            width: 100%;
            max-width: 1200px;
            box-shadow: 0 4px 10px rgba(0,0,0,0.4);
            overflow-x: auto;
        }

        .table-box h2 {
            color: #66ccff;
            border-bottom: 1px solid #444;
            padding-bottom: 10px;
            margin-bottom: 20px;
        }

        table {
            width: 100%;
            border-collapse: collapse;
            color: #fff;
            word-break: break-word;
        }

        th, td {
            padding: 10px;
            text-align: left;
            border-bottom: 1px solid #444;
        }

        th {
            background-color: #444;
        }

        @media (max-width: 768px) {
            .flex-container {
                flex-direction: column;
                align-items: stretch;
            }
        }
    </style>
    </head>
    <body>
        <h1>Welcome to My EC2 Web Server!</h1>
    <p style="text-align:center;">Simple PHP + MySQL App</p>
        <!-- Add Form and Message -->
        <div class="flex-container">
            <div class="form-box">
                <h2>Add User</h2>
                <form method="POST" action="">
                    <input type="text" name="name" required placeholder="Name">
                    <input type="email" name="email" required placeholder="Email">
                    <input type="text" name="country" required placeholder="Country">
                    <input type="submit" name="submit" value="Add User">
                </form>
            </div>

        <?php if ($status == 'added'): ?>
        <div class="message-box" id="status-message">
            <h3>Status</h3>
            <p>✅ User added successfully.</p>
        </div>
        <?php endif; ?>
    </div>

    <!-- User List -->
    <div class="table-box">
        <h2>User List</h2>
        <?php
        $sql = "SELECT name, email, country FROM users ORDER BY id DESC";
        $result = $conn->query($sql);

        if ($result && $result->num_rows > 0) {
            echo "<table>";
            echo "<tr><th>Name</th><th>Email</th><th>Country</th></tr>";
            while ($row = $result->fetch_assoc()) {
                echo "<tr>";
                echo "<td>" . htmlspecialchars($row["name"]) . "</td>";
                echo "<td>" . htmlspecialchars($row["email"]) . "</td>";
                echo "<td>" . htmlspecialchars($row["country"]) . "</td>";
                echo "</tr>";
            }
            echo "</table>";
        } else {
            echo "<p>No users found.</p>";
        }

        $conn->close();
        ?>
    </div>

    <!-- Auto-hide success message -->
    <script>
        setTimeout(() => {
            const msg = document.getElementById('status-message');
            if (msg) {
                msg.classList.add('hidden');
                setTimeout(() => msg.style.display = 'none', 1000); // Hide after fade
            }
        }, 4000);
    </script>
    </body>
    </html>

   ```

3. **Restart Apache**

   ```bash
   # for Amazon Linux
   sudo systemctl restart httpd

   # for Ubuntu Linux
   sudo systemctl restart apache2
   ```

4. **Test Application**

   * Visit `http://<your-ec2-public-ip>` in a browser.
   * Use the form to add users and view the user list.


## 📌 Step 5: Clean Up

* **Stop/Terminate EC2 Instance** when the project is complete to avoid charges.


## ✅ Conclusion

This project demonstrates how to set up an **EC2 instance**, install **Apache2** and **MySQL**, and deploy a **PHP web application** connected to a database.

