# Vanilla Registry System with SQL Connection

## Visão geral

**Vanilla Registry System with SQL Connection** é um projeto em Java que implementa um sistema de registro simples (registry) com persistência em banco de dados relacional via conexão SQL. O objetivo é servir como uma base limpa e "vanilla" (sem frameworks pesados) para demonstrar padrões básicos de CRUD, gestão de conexões JDBC, organização mínima de camadas (DAO/service) e exemplo de configuração de banco de dados.

---

## Principais funcionalidades (esperadas / recomendadas)

- Operações CRUD (Create, Read, Update, Delete) para entidades de registro (ex.: `User`, `Customer`, `Reservation`).
- Abstração de acesso a dados via DAO (Data Access Object) usando JDBC puro.
- Gerenciamento de conexão com banco SQL (MySQL, PostgreSQL, SQLite ou SQL Server) via `DataSource` ou `DriverManager`.
- Arquivo de configuração externo para credenciais e parâmetros de conexão (`config.properties` ou variáveis de ambiente).
- Scripts SQL de criação de esquema e dados de exemplo.
- Execução via Maven/Gradle ou `javac/java` (simples).

---

## Arquitetura sugerida

Estrutura mínima do projeto (sugestão):
Vanilla-RegistrySystemWithSqlConnection/
- ├─ README.md
- ├─ pom.xml (ou build.gradle)
- ├─ src/
- │ ├─ main/
- │ │ ├─ java/
- │ │ │ ├─ com.example.registry/
- │ │ │ │ ├─ App.java
- │ │ │ │ ├─ config/
- │ │ │ │ │ └─ DBConfig.java
- │ │ │ │ ├─ dao/
- │ │ │ │ │ ├─ BaseDAO.java
- │ │ │ │ │ └─ UserDAO.java
- │ │ │ │ ├─ model/
- │ │ │ │ │ └─ User.java
- │ │ │ │ └─ service/
- │ │ │ │ └─ UserService.java
- │ ├─ resources/
- │ │ ├─ config.properties
- │ │ └─ schema.sql
- └─ sql/
- └─ schema.sql


---

## Tecnologias e dependências

- Java 11+ (recomenda-se Java 17 LTS)
- JDBC (driver do banco: MySQL Connector/J, PostgreSQL JDBC, etc.)
- Build: Maven ou Gradle (incluir dependência do driver JDBC no `pom.xml` ou `build.gradle`)
- (Opcional) HikariCP para pool de conexões em produção
- (Opcional) JUnit 5 para testes unitários

---

## Requisitos

- JDK 11 ou superior instalado
- Banco de dados relacional acessível (MySQL, PostgreSQL, SQLite, etc.)
- Ferramenta de build (Maven/Gradle) se optar por usar gerenciamento de dependências
- Variáveis de ambiente ou arquivo `config.properties` com credenciais

---

## Configuração do banco (exemplo)

Exemplo de script SQL (`sql/schema.sql`) que cria uma tabela de usuários:

```sql
CREATE TABLE IF NOT EXISTS users (
  id SERIAL PRIMARY KEY,
  username VARCHAR(100) NOT NULL UNIQUE,
  full_name VARCHAR(200),
  email VARCHAR(200),
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

## Exemplo de arquivo de configuração

- src/main/resources/config.properties (exemplo):
- db.driver=org.postgresql.Driver
- db.url=jdbc:postgresql://localhost:5432/registrydb
- db.username=registry_user
- db.password=registry_password
- db.poolSize=10

<h2>Guia rápido — compilação e execução</h2>

<h3>Usando Maven</h3>

<p>Criar <code>pom.xml</code> com dependência do driver JDBC. Exemplo mínimo (trecho):</p>

<pre><code>&lt;dependencies&gt;
  &lt;!-- Exemplo para PostgreSQL --&gt;
  &lt;dependency&gt;
    &lt;groupId&gt;org.postgresql&lt;/groupId&gt;
    &lt;artifactId&gt;postgresql&lt;/artifactId&gt;
    &lt;version&gt;42.5.0&lt;/version&gt;
  &lt;/dependency&gt;

  &lt;!-- JUnit para testes --&gt;
  &lt;dependency&gt;
    &lt;groupId&gt;org.junit.jupiter&lt;/groupId&gt;
    &lt;artifactId&gt;junit-jupiter&lt;/artifactId&gt;
    &lt;version&gt;5.10.0&lt;/version&gt;
    &lt;scope&gt;test&lt;/scope&gt;
  &lt;/dependency&gt;
&lt;/dependencies&gt;
</code></pre>

<p>Compilar:</p>

<pre><code>mvn clean package</code></pre>

<p>Executar (assumindo App com método main):</p>

<pre><code>java -jar target/vanilla-registry-1.0.jar</code></pre>

<hr>

<h3>Sem Maven (javac/java)</h3>

<p>Compilar:</p>

<pre><code>javac -d out $(find src/main/java -name "*.java")</code></pre>

<p>Executar:</p>

<pre><code>java -cp out com.example.registry.App</code></pre>

<hr>

<h2>Exemplo de implementação (esqueleto)</h2>

<h3>DBConfig.java (exemplo simplificado)</h3>

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

<h3>UserDAO.java (esqueleto)</h3>

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
            // tratar exceção apropriadamente
            e.printStackTrace();
        }

        return Optional.empty();
    }

    // métodos create/update/delete/listar...
}
</code></pre>

<hr>

<h2>Testes</h2>

<ul>
  <li>Adicione testes unitários com JUnit 5 para a lógica de serviço (mock do DAO).</li>
  <li>Para testes de integração com DB, utilize um banco de teste (Docker) ou SQLite em memória e scripts de setup/teardown.</li>
</ul>

<hr>

<h2>Boas práticas e segurança</h2>

<ul>
  <li>Não comitar credenciais no repositório.</li>
  <li>Use TLS/SSL para conexões ao banco em produção.</li>
  <li>Utilize connection pooling (HikariCP) se espera alta carga.</li>
  <li>Trate e registre exceções sem expor segredos.</li>
  <li>Implemente validação de entrada para evitar SQL injection (use PreparedStatement).</li>
</ul>

<hr>

<h2>Contribuição</h2>

<ol>
  <li>Abra uma issue descrevendo a proposta.</li>
  <li>Faça um fork e crie um branch com a feature/bugfix.</li>
  <li>Envie um Pull Request documentando as mudanças e incluindo testes.</li>
</ol>

<hr>

<h2>Licença</h2>

<p>Adicione a licença desejada (ex.: MIT, Apache-2.0). Crie um arquivo <code>LICENSE</code> com o texto da licença escolhida.</p>
