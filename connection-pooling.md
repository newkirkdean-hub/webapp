# Database Connection Pooling Implementation

## Overview
This document describes the database connection pooling implementation for the webapp using Apache Tomcat's JNDI DataSource and a ServletContextListener for initialization.

---

## Purpose
Connection pooling improves application performance by:
- Reusing existing database connections instead of creating new ones
- Reducing connection overhead and latency
- Managing connection lifecycle automatically
- Providing connection validation and recovery
- Limiting maximum concurrent connections to the database

---

## Implementation

### AppContextListener.java
**Package:** `com.database.init`  
**Type:** ServletContextListener  
**Annotation:** `@WebListener`

**Purpose:**
- Initializes the connection pool when the application starts
- Tests the DataSource configuration at startup
- Provides early error detection for database connectivity issues
- Logs initialization status

---

## Java Code

```java
package com.database.init;

import javax.naming.InitialContext;
import javax.naming.NamingException;
import javax.sql.DataSource;
import javax.servlet.ServletContextEvent;
import javax.servlet.ServletContextListener;
import javax.servlet.annotation.WebListener;
import java.sql.Connection;

@WebListener
public class AppContextListener implements ServletContextListener {
    @Override
    public void contextInitialized(ServletContextEvent sce) {
        System.out.println("Initializing connection pool at application startup...");
        try {
            // Lookup the JNDI DataSource defined in context.xml
            InitialContext ctx = new InitialContext();
            DataSource dataSource = (DataSource) ctx.lookup("java:comp/env/jdbc/dbServlet");

            // Test the connection to ensure the pool is initialized
            try (Connection connection = dataSource.getConnection()) {
                System.out.println("Connection pool initialized successfully!");
            }
        } catch (NamingException e) {
            System.err.println("JNDI lookup failed for the DataSource: " + e.getMessage());
            throw new RuntimeException(e);
        } catch (Exception e) {
            System.err.println("Error during pool initialization: " + e.getMessage());
            throw new RuntimeException(e);
        }
    }

    @Override
    public void contextDestroyed(ServletContextEvent sce) {
        System.out.println("Shutting down application.");
        // You can release global resources here (if needed)
    }
}
```

---

## How It Works

### 1. Application Startup (`contextInitialized`)
When the web application starts, this listener:

1. **Creates InitialContext** - Establishes JNDI naming context
2. **Looks up DataSource** - Retrieves the configured connection pool from `java:comp/env/jdbc/dbServlet`
3. **Tests Connection** - Opens and immediately closes a test connection
4. **Logs Success/Failure** - Provides immediate feedback on pool status
5. **Throws Runtime Exception** - Prevents application startup if pool initialization fails

### 2. Application Shutdown (`contextDestroyed`)
When the web application stops:
- Logs shutdown message
- Placeholder for releasing global resources
- Tomcat automatically closes pooled connections

---

## Configuration Requirements

### Tomcat context.xml
The DataSource must be configured in your Tomcat configuration file.

**Location:** 
- `META-INF/context.xml` (in your WAR file)
- OR `$CATALINA_BASE/conf/context.xml` (server-wide)
- OR `$CATALINA_BASE/conf/Catalina/localhost/[app-name].xml` (app-specific)

**Example Configuration:**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<Context>
    <!-- Database Connection Pool Configuration -->
    <Resource name="jdbc/dbServlet"
              auth="Container"
              type="javax.sql.DataSource"
              factory="org.apache.tomcat.jdbc.pool.DataSourceFactory"
              
              <!-- Connection Properties -->
              driverClassName="com.mysql.cj.jdbc.Driver"
              url="jdbc:mysql://localhost:3306/your_database?useSSL=false&amp;serverTimezone=UTC"
              username="your_db_user"
              password="your_db_password"
              
              <!-- Pool Configuration -->
              maxTotal="100"
              maxIdle="30"
              minIdle="10"
              maxWaitMillis="10000"
              
              <!-- Connection Validation -->
              testOnBorrow="true"
              testOnReturn="false"
              testWhileIdle="true"
              validationQuery="SELECT 1"
              validationQueryTimeout="5"
              timeBetweenEvictionRunsMillis="30000"
              
              <!-- Connection Lifecycle -->
              removeAbandonedOnBorrow="true"
              removeAbandonedOnMaintenance="true"
              removeAbandonedTimeout="60"
              logAbandoned="true"
              
              <!-- Performance Tuning -->
              initialSize="10"
              minEvictableIdleTimeMillis="60000"
              jmxEnabled="true"
              jdbcInterceptors="org.apache.tomcat.jdbc.pool.interceptor.ConnectionState;
                                org.apache.tomcat.jdbc.pool.interceptor.StatementFinalizer"/>
</Context>
```

---

## Configuration Parameters Explained

### Connection Properties
| Parameter | Value | Description |
|-----------|-------|-------------|
| `name` | jdbc/dbServlet | JNDI name for lookup |
| `driverClassName` | com.mysql.cj.jdbc.Driver | JDBC driver class |
| `url` | jdbc:mysql://... | Database connection URL |
| `username` | your_db_user | Database username |
| `password` | your_db_password | Database password |

### Pool Size Configuration
| Parameter | Default | Description |
|-----------|---------|-------------|
| `maxTotal` | 100 | Maximum number of connections in pool |
| `maxIdle` | 30 | Maximum idle connections maintained |
| `minIdle` | 10 | Minimum idle connections maintained |
| `initialSize` | 10 | Connections created at startup |

### Connection Validation
| Parameter | Value | Description |
|-----------|-------|-------------|
| `testOnBorrow` | true | Validate connection before giving to app |
| `testWhileIdle` | true | Validate idle connections periodically |
| `validationQuery` | SELECT 1 | SQL query to test connection |
| `validationQueryTimeout` | 5 | Timeout for validation query (seconds) |

### Abandoned Connection Handling
| Parameter | Value | Description |
|-----------|-------|-------------|
| `removeAbandonedOnBorrow` | true | Remove abandoned connections when pool is full |
| `removeAbandonedTimeout` | 60 | Time before connection is considered abandoned (seconds) |
| `logAbandoned` | true | Log stack trace of abandoned connections |

---

## Usage in Servlets

Once the connection pool is initialized, use it in your servlets like this:

```java
package com.yourcompany.servlet;

import javax.naming.InitialContext;
import javax.sql.DataSource;
import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.sql.Connection;
import java.sql.PreparedStatement;
import java.sql.ResultSet;

@WebServlet("/example")
public class ExampleServlet extends HttpServlet {
    private DataSource dataSource;

    @Override
    public void init() throws ServletException {
        try {
            InitialContext ctx = new InitialContext();
            dataSource = (DataSource) ctx.lookup("java:comp/env/jdbc/dbServlet");
        } catch (Exception e) {
            throw new ServletException("Failed to lookup DataSource", e);
        }
    }

    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response) {
        String sql = "SELECT * FROM your_table WHERE id = ?";
        
        try (Connection conn = dataSource.getConnection();
             PreparedStatement stmt = conn.prepareStatement(sql)) {
            
            stmt.setInt(1, 123);
            
            try (ResultSet rs = stmt.executeQuery()) {
                // Process results...
            }
            
        } catch (Exception e) {
            e.printStackTrace();
            // Handle error...
        }
    }
}
```

---

## Benefits of This Implementation

### ✅ Advantages
1. **Early Detection** - Fails fast at startup if database is unreachable
2. **Automatic Management** - Tomcat manages connection lifecycle
3. **Resource Efficiency** - Reuses connections instead of creating new ones
4. **Performance** - Eliminates connection creation overhead per request
5. **Scalability** - Handles concurrent requests efficiently
6. **Configuration Flexibility** - Tune pool settings without code changes
7. **Connection Validation** - Automatically tests and refreshes stale connections
8. **Monitoring** - JMX enabled for production monitoring

### ⚠️ Considerations
1. **Database Load** - Set `maxTotal` based on database capacity
2. **Memory Usage** - Each connection consumes memory
3. **Timeout Settings** - Balance between responsiveness and resource usage
4. **Driver Dependencies** - Ensure JDBC driver is in Tomcat's lib folder
5. **Connection Leaks** - Always use try-with-resources to prevent leaks

---

## Troubleshooting

### Problem: "JNDI lookup failed for the DataSource"
**Causes:**
- context.xml not properly configured
- Wrong JNDI name in lookup
- DataSource resource not defined

**Solution:**
- Verify context.xml exists and is properly formatted
- Confirm JNDI name matches: `java:comp/env/jdbc/dbServlet`
- Check Tomcat logs for configuration errors

---

### Problem: "Connection pool initialized" but queries fail
**Causes:**
- Wrong database URL, username, or password
- Database server not running
- Firewall blocking connection
- Wrong database driver

**Solution:**
- Test connection manually using database client
- Verify credentials and database exists
- Check database server logs
- Ensure JDBC driver JAR is in `$CATALINA_HOME/lib`

---

### Problem: "Connection pool exhausted"
**Causes:**
- Too many concurrent requests
- Connection leaks (not closing connections)
- `maxTotal` set too low

**Solution:**
- Use try-with-resources to ensure connections close
- Increase `maxTotal` parameter
- Enable `logAbandoned` to find connection leaks
- Monitor with JMX to see active connections

---

### Problem: Slow query performance
**Causes:**
- Connection validation overhead
- Too many idle connections
- Database performance issues

**Solution:**
- Tune `validationQuery` or disable `testOnBorrow`
- Adjust `maxIdle` and `minIdle` settings
- Analyze slow queries in database
- Enable query logging with jdbcInterceptors

---

## Monitoring and Maintenance

### Enable JMX Monitoring
Add to your Tomcat startup script:
```bash
CATALINA_OPTS="$CATALINA_OPTS -Dcom.sun.management.jmxremote"
CATALINA_OPTS="$CATALINA_OPTS -Dcom.sun.management.jmxremote.port=9090"
CATALINA_OPTS="$CATALINA_OPTS -Dcom.sun.management.jmxremote.authenticate=false"
CATALINA_OPTS="$CATALINA_OPTS -Dcom.sun.management.jmxremote.ssl=false"
```

### View Pool Metrics
Use JConsole or VisualVM to monitor:
- Active connections
- Idle connections
- Connection wait time
- Failed connection attempts
- Abandoned connections

---

## Best Practices

1. **Always use try-with-resources** for Connection, Statement, ResultSet
2. **Set appropriate pool sizes** based on expected load
3. **Enable connection validation** to handle stale connections
4. **Monitor abandoned connections** to detect leaks
5. **Use PreparedStatements** to prevent SQL injection and improve performance
6. **Test configuration changes** in development before production
7. **Document your pool settings** for future reference
8. **Regular monitoring** of pool metrics in production

---

## Related Files
- `servlet-example.md` - Example servlet using connection pooling
- `VoucherViewer` servlet - Production example of DataSource usage

---

## Repository Information
- **Source File:** connectionpooling.txt
- **Repository:** newkirkdean-hub/webapp
- **Branch:** main
- **Package:** com.database.init
- **Created:** 2026-02-16