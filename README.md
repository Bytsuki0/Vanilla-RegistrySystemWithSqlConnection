<h1>Vanilla Registry System with SQL Connection</h1>

<h2>Overview</h2>

<p><strong>Vanilla Registry System with SQL Connection</strong> is a Java project that implements a simple registry system with persistence in a relational database via SQL connection. The goal is to provide a clean and “vanilla” (no heavy frameworks) foundation to demonstrate basic CRUD patterns, JDBC connection management, minimal layer organization (DAO/service), and database configuration examples.</p>

<hr>

<h2>Main Features (Expected / Recommended)</h2>

<ul>
  <li>CRUD operations (Create, Read, Update, Delete) for registry entities (e.g., <code>User</code>, <code>Customer</code>, <code>Reservation</code>).</li>
  <li>Data access abstraction via DAO (Data Access Object) using plain JDBC.</li>
  <li>SQL database connection management (MySQL, PostgreSQL, SQLite, or SQL Server) via <code>DataSource</code> or <code>DriverManager</code>.</li>
  <li>External configuration file for credentials and connection parameters (<code>config.properties</code> or environment variables).</li>
  <li>SQL scripts for schema creation and sample data.</li>
  <li>Execution via Maven/Gradle or plain <code>javac/java</code>.</li>
</ul>

<hr>

<h2>Suggested Architecture</h2>

<p>Minimal project structure (suggestion):</p>

<pre><code>Vanilla-RegistrySystemWithSqlConnection/
├─ README.md
├─ pom.xml (or build.gradle)
├─ src/
│  ├─ main/
│  │  ├─ java/
│  │  │  ├─ com.example.registry/
│  │  │  │  ├─ App.java
│  │  │  │  ├─ config/
│  │  │  │  │  └─ DBConfig.java
│  │  │  │  ├─ dao/
│  │  │  │  │  ├─ BaseDAO.java
│  │  │  │  │  └─ UserDAO.java
│  │  │  │  ├─ model/
│  │  │  │  │  └─ User.java
│  │  │  │  └─ service/
│  │  │  │     └─ UserService.java
│  ├─ resources/
│  │  ├─ config.properties
│  │  └─ schema.sql
└─ sql/
   └─ schema.sql
</code></pre>

<hr>

<h2>Technologies and Dependencies</h2>

<ul>
  <li>Java 11+ (Java 17 LTS recommended)</li>
  <li>JDBC (database driver: MySQL Connector/J, PostgreSQL JDBC, etc.)</li>
  <li>Build tool: Maven or Gradle (include JDBC driver dependency in <code>pom.xml</code> or <code>build.gradle</code>)</li>
  <li>(Optional) HikariCP for connection pooling in production</li>
  <li>(Optional) JUnit 5 for unit testing</li>
</ul>

<hr>

<h2>Requirements</h2>

<ul>
  <li>JDK 11 or higher installed</li>
  <li>Accessible relational database (MySQL, PostgreSQL, SQLite, etc.)</li>
  <li>Build tool (Maven/Gradle) if using dependency management</li>
  <li>Environment variables or <code>config.properties</code> file with credentials</li>
</ul>

<hr>

<h2>Database Configuration (Example)</h2>

<p>Example SQL script (<code>sql/schema.sql</code>) that creates a users table:</p>

<pre><code>CREATE TABLE IF NOT EXISTS users (
  id SERIAL PRIMARY KEY,
  username VARCHAR(100) NOT NULL UNIQUE,
  full_name VARCHAR(200),
  email VARCHAR(200),
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
</code></pre>

<hr>

<h2>Example Configuration File</h2>

<p><code>src/main/resources/config.properties</code> (example):</p>

<pre><code>db.driver=org.postgresql.Driver
db.url=jdbc:postgresql://localhost:5432/registrydb
db.username=registry_user
db.password=registry_password
db.poolSize=10
</code></pre>

<hr>

<h2>Quick Start — Build and Run</h2>

<h3>Using Maven</h3>

<p>Create a <code>pom.xml</code> with the JDBC driver dependency. Minimal example (excerpt):</p>

<pre><code>&lt;dependencies&gt;
  &lt;!-- Example for PostgreSQL --&gt;
  &lt;dependency&gt;
    &lt;groupId&gt;org.postgresql&lt;/groupId&gt;
    &lt;artifactId&gt;postgresql&lt;/artifactId&gt;
    &lt;version&gt;42.5.0&lt;/version&gt;
  &lt;/dependency&gt;

  &lt;!-- JUnit for testing --&gt;
  &lt;dependency&gt;
    &lt;groupId&gt;org.junit.jupiter&lt;/groupId&gt;
    &lt;artifactId&gt;junit-jupiter&lt;/artifactId&gt;
    &lt;version&gt;5.10.0&lt;/version&gt;
    &lt;scope&gt;test&lt;/scope&gt;
  &lt;/dependency&gt;
&lt;/dependencies&gt;
</code></pre>

<p>Build:</p>

<pre><code>mvn clean package</code></pre>

<p>Run (assuming <code>App</code> has a main method):</p>

<pre><code>java -jar target/vanilla-registry-1.0.jar</code></pre>

<hr>

<h3>Without Maven (javac/java)</h3>

<p>Compile:</p>

<pre><code>javac -d out $(find src/main/java -name "*.java")</code></pre>

<p>Run:</p>

<pre><code>java -cp out com.example.registry.App</code></pre>

<hr>

<h2>Example Implementation (Skeleton)</h2>

<h3>DBConfig.java (Simplified Example)</h3>

<pre><code>package com.example.registry.config;

import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.SQLException;

public class DBConfig {
    private static final String URL = System.getenv()
        .getOrDefault("DB_URL", "jdbc:postgresql://localhost:5432/registrydb");
    private static final String USER = System.getenv()
        .getOrDefault("DB_USER", "registry_user");
    private static final String PASS = System.getenv()
        .getOrDefault("DB_PASS", "registry_password");

    public static Connection getConnection() throws SQLException {
        return DriverManager.getConnection(URL, USER, PASS);
    }
}
</code></pre>

<h3>UserDAO.java (Skeleton)</h3>

<pre><code>package com.example.registry.dao;

import com.example.registry.model.User;
import com.example.registry.config.DBConfig;

import java.sql.*;
import java.util.Optional;

public class UserDAO {

    public Optional&lt;User&gt; findById(long id) {
        String sql = "SELECT id, username, full_name, email, created_at FROM users WHERE id = ?";

        try (Connection conn = DBConfig.getConnection();
             PreparedStatement ps = conn.prepareStatement(sql)) {

            ps.setLong(1, id);
            ResultSet rs = ps.executeQuery();

            if (rs.next()) {
                User u = new User(
                    rs.getLong("id"),
                    rs.getString("username"),
                    rs.getString("full_name"),
                    rs.getString("email"),
                    rs.getTimestamp("created_at").toInstant()
                );
                return Optional.of(u);
            }

        } catch (SQLException e) {
            // handle exception appropriately
            e.printStackTrace();
        }

        return Optional.empty();
    }

    // create/update/delete/list methods...
}
</code></pre>

<hr>

<h2>Testing</h2>

<ul>
  <li>Add unit tests using JUnit 5 for service-layer logic (mock the DAO).</li>
  <li>For database integration tests, use a test database (Docker) or in-memory SQLite with setup/teardown scripts.</li>
</ul>

<hr>

<h2>Best Practices and Security</h2>

<ul>
  <li>Do not commit credentials to the repository.</li>
  <li>Use TLS/SSL for database connections in production.</li>
  <li>Use connection pooling (e.g., HikariCP) if high load is expected.</li>
  <li>Handle and log exceptions without exposing secrets.</li>
  <li>Validate inputs to prevent SQL injection (always use <code>PreparedStatement</code>).</li>
</ul>

<hr>

<h2>Contribution</h2>

<ol>
  <li>Open an issue describing the proposal.</li>
  <li>Fork the repository and create a branch for your feature/bugfix.</li>
  <li>Submit a Pull Request documenting the changes and including tests.</li>
</ol>

<hr>

<h2>License</h2>

<p>Add your chosen license (e.g., MIT, Apache-2.0). Create a <code>LICENSE</code> file with the full license text.</p>
