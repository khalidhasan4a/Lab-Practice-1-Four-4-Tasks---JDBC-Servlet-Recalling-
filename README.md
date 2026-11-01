# ICT 3108 – Software Engineering Project Management (SEPM)
## Lab Practice: Java, JDBC, Apache Tomcat & Java Servlet

**Department of ICT**  
**Mawlana Bhashani Science and Technology University (MBSTU)**

---

## Student Information

| Field | Information |
|---|---|
| **Full Name** |Md. Khalid Hasan|
| **Student ID** |IT-24013|
| **Course** | ICT 3108 – Software Engineering Project Management |
| **Lab Date** | 1 November 2026 |
| **Development Environment** | Windows 10/11, IntelliJ IDEA 2026.2.1 |
| **Java Version** | Java 21 (JDBC project); Java 8 source/target configuration (Servlet project) |
| **Application Server** | Apache Tomcat 10.1.57 |
| **Build Tool** | Maven |
| **Database** | MySQL |

> Replace the student information placeholders above before submitting.

---

## Table of Contents

1. [Lab Overview](#lab-overview)
2. [Task 1 – Install IntelliJ IDEA Ultimate](#task-1--install-intellij-idea-ultimate)
3. [Task 2 – JDBC Database Connectivity: N-th Prime Number](#task-2--jdbc-database-connectivity-n-th-prime-number)
4. [Task 3 – Configure Apache Tomcat Server](#task-3--configure-apache-tomcat-server)
5. [Task 4 – Browser-Based Servlet Applications](#task-4--browser-based-servlet-applications)
6. [Project Structure](#project-structure)
7. [Troubleshooting Notes](#troubleshooting-notes)
8. [Submission Checklist](#submission-checklist)
9. [GitHub Repository](#github-repository)

---

## Lab Overview

The purpose of this lab was to gain practical experience with Java development, JDBC database connectivity, Apache Tomcat configuration, and Java Servlet development.

The work covers the following:

- Installing and verifying IntelliJ IDEA.
- Finding the n-th prime number and saving the result in MySQL using JDBC.
- Downloading and configuring Apache Tomcat 10.1.57.
- Building and deploying Java Servlet applications and testing them in a browser.

---

## Task 1 – Install IntelliJ IDEA Ultimate

### Objective

Install IntelliJ IDEA Ultimate, activate the JetBrains educational licence, and verify that the IDE is working.

### Implementation

1. Downloaded and installed IntelliJ IDEA.
2. Opened IntelliJ IDEA and verified that the application launched successfully.
3. The installed version shown on the system is **IntelliJ IDEA 2026.2.1**.
4. The JetBrains Student Programme can be used to apply for an educational licence with a valid university email address.

**Student Programme:**  
https://www.jetbrains.com/community/education/#students

### Screenshot

<img width="966" height="736" alt="image" src="https://github.com/user-attachments/assets/923ca450-275f-4716-8821-857179e9d736" />


*Figure 1: IntelliJ IDEA 2026.2.1 appears in Windows Search and the application is available to open.*

> Licence activation should be verified in IntelliJ IDEA under **Help → Register** (the menu wording may vary by version). Add a screenshot of the activated educational licence here if required by the instructor.

---

## Task 2 – JDBC Database Connectivity: N-th Prime Number

### Objective

Create a Java program that accepts an integer `n`, calculates the n-th prime number, inserts the result into a MySQL database using JDBC, and displays a confirmation message.

### Technologies

- Java 21
- Maven
- MySQL
- MySQL Connector/J 8.3.0
- JDBC

### Database Setup

Create a database and the required table in MySQL:

```sql
CREATE DATABASE primedb;

USE primedb;

CREATE TABLE prime_store (
    id INT AUTO_INCREMENT PRIMARY KEY,
    n INT NOT NULL,
    nth_prime INT NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

The table stores the input position (`n`) and its corresponding prime number (`nth_prime`).

### Maven Configuration

**File:** `pom.xml`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
         http://maven.apache.org/xsd/maven-4.0.0.xsd">

    <modelVersion>4.0.0</modelVersion>

    <groupId>org.example</groupId>
    <artifactId>store-nth-prime</artifactId>
    <version>1.0-SNAPSHOT</version>

    <properties>
        <maven.compiler.source>21</maven.compiler.source>
        <maven.compiler.target>21</maven.compiler.target>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    </properties>

    <dependencies>
        <dependency>
            <groupId>com.mysql</groupId>
            <artifactId>mysql-connector-j</artifactId>
            <version>8.3.0</version>
        </dependency>
    </dependencies>

</project>
```

Maven downloads the MySQL JDBC driver automatically from the dependency declared above.

### Java Source Code

**File:** `src/main/java/org/example/PrimeJDBC.java`

```java
package org.example;

import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.PreparedStatement;
import java.sql.SQLException;
import java.util.Scanner;

public class PrimeJDBC {

    // MySQL database connection information
    static final String URL = "jdbc:mysql://localhost:3306/primedb";
    static final String USER = "root";
    static final String PASS = "12345";

    public static void main(String[] args) throws SQLException {

        Scanner sc = new Scanner(System.in);

        System.out.print("Enter n: ");
        int n = sc.nextInt();

        // Validate the input
        if (n < 1) {
            System.out.println("Please enter a positive integer.");
            sc.close();
            return;
        }

        // Find the n-th prime number
        int count = 0;
        int num = 1;

        while (count < n) {
            num++;

            boolean prime = true;

            // Check whether the current number is prime
            for (int i = 2; i * i <= num; i++) {
                if (num % i == 0) {
                    prime = false;
                    break;
                }
            }

            if (prime) {
                count++;
            }
        }

        // Insert the result into the database
        String sql = "INSERT INTO prime_store (n, nth_prime) VALUES (?, ?)";

        try (Connection con = DriverManager.getConnection(URL, USER, PASS);
             PreparedStatement ps = con.prepareStatement(sql)) {

            ps.setInt(1, n);
            ps.setInt(2, num);

            ps.executeUpdate();

            System.out.println(n + "th prime = " + num + " -> saved!");
        }

        sc.close();
    }
}
```

### Code Explanation

1. `Scanner` reads the value of `n` from the keyboard.
2. The `while` loop checks consecutive integers until it finds `n` prime numbers.
3. The `for` loop checks divisibility from `2` up to the square root of the current number.
4. When a prime number is found, the counter is increased.
5. After finding the n-th prime, `DriverManager.getConnection()` establishes the JDBC connection.
6. `PreparedStatement` inserts the input and calculated prime number into `prime_store`.
7. `executeUpdate()` executes the SQL insertion.

### Run the Program

Run `PrimeJDBC.java` from IntelliJ IDEA. Example:

```text
Enter n: 10
10th prime = 29 -> saved!
```

### Verify the Database Record

Execute:

```sql
USE primedb;

SELECT * FROM prime_store;
```

Example expected result for input `10`:

mysql> SELECT * FROM prime_store;
+----+-----+-----------+
| id | n   | nth_prime |
+----+-----+-----------+
|  1 |   5 |        11 |
|  2 |  10 |        29 |
|  3 |  50 |       229 |
|  4 | 150 |       863 |
+----+-----+-----------+

The `created_at` column is generated automatically by MySQL.

### Important Configuration Note

Update `URL`, `USER`, and `PASS` in `PrimeJDBC.java` to match the local MySQL configuration. The database name and table/column names in the SQL statement must also match the database schema.

---

## Task 3 – Configure Apache Tomcat Server

### Objective

Download Apache Tomcat, configure it as an application server, and verify that it can be used to deploy Java web applications.

### Environment

| Component | Version / Path |
|---|---|
| Apache Tomcat | 10.1.57 |
| Downloaded archive | `apache-tomcat-10.1.57-windows-x64.zip` |
| Extracted folder | `apache-tomcat-10.1.57` |
| Tomcat installation path used | `E:\Tools\apache-tomcat-10.1.57` |

### Installation Steps

1. Downloaded the Windows ZIP distribution of Apache Tomcat 10.1.57.
2. Extracted the ZIP archive.
3. Placed the extracted Tomcat folder in the tools directory.
4. In IntelliJ IDEA Ultimate, opened **Settings → Build, Execution, Deployment → Application Servers**.
5. Added a new **Tomcat Server** configuration and selected the extracted Tomcat installation directory.
6. Created a Tomcat run configuration and selected the required deployment artifact (WAR exploded or WAR, depending on the project).
7. Started the server from IntelliJ IDEA.

**Official website:**  
https://tomcat.apache.org/

### Installation Screenshot

<img width="982" height="608" alt="image" src="https://github.com/user-attachments/assets/3256aa56-37c7-4d96-9da9-fa39ccc4a3c5" />


*Figure 2: Apache Tomcat 10.1.57 ZIP archive and extracted folder visible in Windows Search.*

### Verify Tomcat

When Tomcat is running, open:

```text
http://localhost:8080/
```

The Tomcat welcome page should appear if the server is running on the default port.

### Tomcat Configuration Used

The Tomcat installation directory used for the project was:

```text
E:\Tools\apache-tomcat-10.1.57
```

The project was run locally through IntelliJ IDEA using the configured Tomcat server.

---

## Task 4 – Browser-Based Servlet Applications

### Objective

Develop Java Servlet applications, deploy them on Apache Tomcat, and access them through a web browser.

Two browser-based applications were tested:

1. Hello World Servlet.
2. Prime Checker Servlet.

### 4.1 Hello World Servlet

#### Maven Configuration

**File:** `pom.xml`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
         https://maven.apache.org/xsd/maven-4.0.0.xsd">

    <modelVersion>4.0.0</modelVersion>

    <groupId>org.example</groupId>
    <artifactId>HelloServletApp</artifactId>
    <version>1.0-SNAPSHOT</version>
    <name>HelloServletApp</name>
    <packaging>war</packaging>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <maven.compiler.target>8</maven.compiler.target>
        <maven.compiler.source>8</maven.compiler.source>
        <junit.version>5.10.2</junit.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>jakarta.servlet</groupId>
            <artifactId>jakarta.servlet-api</artifactId>
            <version>6.0.0</version>
            <scope>provided</scope>
        </dependency>

        <dependency>
            <groupId>org.junit.jupiter</groupId>
            <artifactId>junit-jupiter-api</artifactId>
            <version>${junit.version}</version>
            <scope>test</scope>
        </dependency>

        <dependency>
            <groupId>org.junit.jupiter</groupId>
            <artifactId>junit-jupiter-engine</artifactId>
            <version>${junit.version}</version>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-war-plugin</artifactId>
                <version>3.4.0</version>
            </plugin>
        </plugins>
    </build>

</project>
```

#### Servlet Source Code

**File:** `src/main/java/org/example/helloservletapp/HelloServlet.java`

```java
package org.example.helloservletapp;

import java.io.IOException;
import java.io.PrintWriter;

import jakarta.servlet.annotation.WebServlet;
import jakarta.servlet.http.HttpServlet;
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;

@WebServlet(name = "helloServlet", value = "/hello-servlet")
public class HelloServlet extends HttpServlet {

    private String message;

    @Override
    public void init() {
        message = "Hello World!";
    }

    @Override
    protected void doGet(HttpServletRequest request,
                         HttpServletResponse response) throws IOException {

        response.setContentType("text/html;charset=UTF-8");

        // Send the response to the browser
        PrintWriter out = response.getWriter();

        out.println("<html><body>");
        out.println("<h1>" + message + "</h1>");
        out.println("</body></html>");
    }

    @Override
    public void destroy() {
        // Servlet cleanup can be performed here
    }
}
```

#### JSP Home Page

**File:** `src/main/webapp/index.jsp`

```jsp
<%@ page contentType="text/html; charset=UTF-8" pageEncoding="UTF-8" %>
<!DOCTYPE html>
<html>
<head>
    <title>JSP - Hello World</title>
</head>
<body>
    <h1><%= "Khalid" %></h1>
    <br/>
    <a href="hello-servlet">Hello Servlet</a>
</body>
</html>
```

#### How It Works

- `@WebServlet` maps the servlet to `/hello-servlet`.
- When the browser requests that URL, Tomcat calls the servlet's `doGet()` method.
- `response.setContentType()` sets the response type to HTML.
- `PrintWriter` writes the HTML response to the browser.
- The JSP page displays the name and provides a link to the servlet.

#### Browser URLs

Assuming the application context path is `/HelloServletApp`:

```text
http://localhost:8080/HelloServletApp/
http://localhost:8080/HelloServletApp/hello-servlet
```

Expected servlet output:

```text
Hello World!
```

---

### 4.2 Prime Checker Servlet

#### Objective

Accept an integer from the browser and determine whether it is a prime number.

#### Servlet Source Code

**File:** `src/main/java/org/example/helloservletapp/PrimeServlet.java`

```java
package org.example.helloservletapp;

import java.io.IOException;
import java.io.PrintWriter;

import jakarta.servlet.annotation.WebServlet;
import jakarta.servlet.http.HttpServlet;
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;

@WebServlet(name = "primeServlet", value = "/prime-check")
public class PrimeServlet extends HttpServlet {

    // Checks whether a number is prime
    private boolean isPrime(int number) {

        if (number <= 1) {
            return false;
        }

        for (int i = 2; i * i <= number; i++) {
            if (number % i == 0) {
                return false;
            }
        }

        return true;
    }

    @Override
    protected void doGet(HttpServletRequest request,
                         HttpServletResponse response) throws IOException {

        response.setContentType("text/html;charset=UTF-8");

        PrintWriter out = response.getWriter();

        String input = request.getParameter("number");

        out.println("<!DOCTYPE html>");
        out.println("<html><head><title>Prime Checker</title></head><body>");
        out.println("<h2>Prime Number Checker</h2>");

        // Display the input form
        out.println("<form method='get' action='prime-check'>");
        out.println("Enter a number: ");
        out.println("<input type='number' name='number' required>");
        out.println("<button type='submit'>Check</button>");
        out.println("</form>");

        // Process the submitted number
        if (input != null && !input.trim().isEmpty()) {
            try {
                int number = Integer.parseInt(input);

                if (isPrime(number)) {
                    out.println("<h3>" + number + " is a Prime Number.</h3>");
                } else {
                    out.println("<h3>" + number + " is not a Prime Number.</h3>");
                }

            } catch (NumberFormatException e) {
                out.println("<h3>Please enter a valid integer.</h3>");
            }
        }

        out.println("</body></html>");
    }
}
```

#### How It Works

1. The browser sends an integer through the `number` request parameter.
2. The servlet reads the value using `request.getParameter("number")`.
3. `Integer.parseInt()` converts the input into an integer.
4. The `isPrime()` method checks whether the number is divisible by any integer from `2` through its square root.
5. The servlet generates an HTML response showing whether the number is prime.

#### Browser URL

Assuming the context path is `/HelloServletApp`:

```text
http://localhost:8080/HelloServletApp/prime-check
```

#### Test Cases

| Input | Expected Output |
|---:|---|
| 2 | 2 is a Prime Number. |
| 7 | 7 is a Prime Number. |
| 10 | 10 is not a Prime Number. |
| 1 | 1 is not a Prime Number. |
| 0 | 0 is not a Prime Number. |

### Servlet Deployment

1. Open the Servlet project in IntelliJ IDEA.
2. Ensure the project uses `war` packaging.
3. Ensure the Jakarta Servlet API dependency is available with `provided` scope.
4. Add the Tomcat 10.1.57 server configuration in IntelliJ IDEA.
5. Add the project artifact to the Tomcat deployment tab.
6. Run the Tomcat configuration.
7. Open the application URLs in a browser and test both servlet applications.

> Tomcat 10.1 uses the `jakarta.servlet.*` package. Do not replace these imports with the older `javax.servlet.*` package when using Tomcat 10.1.

---

## Project Structure

### JDBC Project

```text
store-nth-prime/
├── pom.xml
└── src/
    └── main/
        └── java/
            └── org/
                └── example/
                    └── PrimeJDBC.java
```

### Servlet Project

```text
HelloServletApp/
├── pom.xml
└── src/
    └── main/
        ├── java/
        │   └── org/
        │       └── example/
        │           └── helloservletapp/
        │               ├── HelloServlet.java
        │               └── PrimeServlet.java
        └── webapp/
            └── index.jsp
```

### Suggested Screenshot Folder

Keep the screenshots in the repository using these filenames so the Markdown images render correctly:

```text
screenshots/
├── intellij-idea-2026.2.1.png
├── apache-tomcat-10.1.57.png
├── jetbrains-student-licence.png
├── jdbc-console-output.png
├── mysql-table-output.png
├── tomcat-running.png
├── hello-world-browser.png
└── prime-checker-browser.png
```

The first two screenshot files correspond to the images included with this lab documentation. Add the remaining screenshots from your own successful tests before submission.

---

## Troubleshooting Notes

| Problem | What to check |
|---|---|
| JDBC connection refused | Make sure the MySQL server is running and the JDBC URL/port are correct. |
| Access denied for MySQL user | Verify the username and password in `PrimeJDBC.java`. |
| Database or table not found | Create `primedb` and `prime_store` using the SQL script above. |
| MySQL driver not found | Reload Maven and verify the MySQL Connector/J dependency. |
| Tomcat does not start | Check the configured Tomcat installation path and whether port `8080` is already in use. |
| HTTP 404 | Check the deployed application context path and servlet URL mapping. |
| Servlet class/import error | Verify the Jakarta Servlet API dependency and use Tomcat 10.1 or a compatible server. |

---

## Submission Checklist

- [x] IntelliJ IDEA 2026.2.1 installed and launched.
- [ ] JetBrains Student Licence activation screenshot added.
- [x] JDBC program written to calculate the n-th prime number.
- [x] JDBC insertion implemented using `PreparedStatement`.
- [ ] MySQL table output screenshot added.
- [x] Apache Tomcat 10.1.57 downloaded and extracted.
- [ ] Tomcat running screenshot added.
- [x] Hello World Servlet implemented.
- [x] Prime Checker Servlet implemented.
- [ ] Browser screenshots for both servlet applications added.
- [ ] Student information and GitHub repository link completed.
- [ ] All screenshots uploaded to the `screenshots/` directory.

---

## GitHub Repository

**Repository Link:** `ADD YOUR GITHUB REPOSITORY URL HERE`

---

**Submitted to:** Department of ICT  
**University:** Mawlana Bhashani Science and Technology University (MBSTU)  
**Course:** ICT 3108 – Software Engineering Project Management  
**Lab Date:** 1 November 2026
