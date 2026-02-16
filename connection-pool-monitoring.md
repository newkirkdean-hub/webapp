# Connection Pool Monitoring and Best Practices

## Overview
This document provides guidance on monitoring database connection pools in Tomcat and best practices for maintaining optimal performance and reliability.

---

## Table of Contents
1. [JMX Monitoring Setup](#jmx-monitoring-setup)
2. [Monitoring Tools](#monitoring-tools)
3. [Key Metrics to Monitor](#key-metrics-to-monitor)
4. [Best Practices](#best-practices)
5. [Troubleshooting Guide](#troubleshooting-guide)
6. [Performance Tuning](#performance-tuning)

---

## JMX Monitoring Setup

### Enable JMX in Tomcat

JMX (Java Management Extensions) allows you to monitor and manage your connection pool in real-time.

#### For Linux/Mac (catalina.sh)

Add the following to your Tomcat startup script before starting Tomcat:
```bash
# Enable JMX Remote Monitoring
export CATALINA_OPTS="$CATALINA_OPTS -Dcom.sun.management.jmxremote"
export CATALINA_OPTS="$CATALINA_OPTS -Dcom.sun.management.jmxremote.port=9090"
export CATALINA_OPTS="$CATALINA_OPTS -Dcom.sun.management.jmxremote.authenticate=false"
export CATALINA_OPTS="$CATALINA_OPTS -Dcom.sun.management.jmxremote.ssl=false"
```

**Location**: Add to `$CATALINA_HOME/bin/setenv.sh` (create if it doesn't exist)
```bash
#!/bin/bash
# JMX Configuration for Connection Pool Monitoring
CATALINA_OPTS="$CATALINA_OPTS -Dcom.sun.management.jmxremote"
CATALINA_OPTS="$CATALINA_OPTS -Dcom.sun.management.jmxremote.port=9090"
CATALINA_OPTS="$CATALINA_OPTS -Dcom.sun.management.jmxremote.authenticate=false"
CATALINA_OPTS="$CATALINA_OPTS -Dcom.sun.management.jmxremote.ssl=false"
CATALINA_OPTS="$CATALINA_OPTS -Dcom.sun.management.jmxremote.local.only=false"
CATALINA_OPTS="$CATALINA_OPTS -Djava.rmi.server.hostname=localhost"
```

#### For Windows (catalina.bat)

Create or edit `%CATALINA_HOME%\bin\setenv.bat`:
```batch
@echo off
rem JMX Configuration for Connection Pool Monitoring
set CATALINA_OPTS=%CATALINA_OPTS% -Dcom.sun.management.jmxremote
set CATALINA_OPTS=%CATALINA_OPTS% -Dcom.sun.management.jmxremote.port=9090
set CATALINA_OPTS=%CATALINA_OPTS% -Dcom.sun.management.jmxremote.authenticate=false
set CATALINA_OPTS=%CATALINA_OPTS% -Dcom.sun.management.jmxremote.ssl=false
set CATALINA_OPTS=%CATALINA_OPTS% -Dcom.sun.management.jmxremote.local.only=false
set CATALINA_OPTS=%CATALINA_OPTS% -Djava.rmi.server.hostname=localhost
```

### JMX Configuration Parameters

| Parameter | Value | Description |
|-----------|-------|-------------|
| `jmxremote` | (flag) | Enables JMX remote monitoring |
| `jmxremote.port` | 9090 | Port for JMX connections |
| `jmxremote.authenticate` | false | Disables authentication (dev only) |
| `jmxremote.ssl` | false | Disables SSL (dev only) |
| `jmxremote.local.only` | false | Allows remote connections |
| `rmi.server.hostname` | localhost | RMI server hostname |

⚠️ **Security Warning**: The configuration above is for **development only**. For production, enable authentication and SSL.

### Production JMX Configuration

For production environments, use authentication and SSL:
```bash
# Production JMX Configuration
CATALINA_OPTS="$CATALINA_OPTS -Dcom.sun.management.jmxremote"
CATALINA_OPTS="$CATALINA_OPTS -Dcom.sun.management.jmxremote.port=9090"
CATALINA_OPTS="$CATALINA_OPTS -Dcom.sun.management.jmxremote.authenticate=true"
CATALINA_OPTS="$CATALINA_OPTS -Dcom.sun.management.jmxremote.ssl=true"
CATALINA_OPTS="$CATALINA_OPTS -Dcom.sun.management.jmxremote.password.file=$CATALINA_HOME/conf/jmxremote.password"
CATALINA_OPTS="$CATALINA_OPTS -Dcom.sun.management.jmxremote.access.file=$CATALINA_HOME/conf/jmxremote.access"
```

---

## Monitoring Tools

### 1. JConsole (Built-in)

**Location**: Included with JDK

**How to Connect**:
```bash
# Start JConsole
jconsole

# Connect to:
# Remote Process: localhost:9090
```

**Steps**:
1. Start JConsole from your JDK bin directory
2. Select "Remote Process"
3. Enter: `localhost:9090`
4. Click "Connect"
5. Navigate to: **MBeans → Catalina → DataSource → jdbc/dbServlet**

**Available Metrics**:
- `numActive` - Active connections in use
- `numIdle` - Idle connections available
- `maxTotal` - Maximum pool size
- `maxIdle` - Maximum idle connections
- `numTestsPerEvictionRun` - Validation tests per run

---

### 2. VisualVM (Recommended)

**Download**: https://visualvm.github.io/

**Advantages**:
- Better UI than JConsole
- More detailed graphs
- Plugin support
- Performance profiling

**How to Connect**:
1. Start VisualVM
2. Right-click "Local" → "Add JMX Connection"
3. Connection: `localhost:9090`
4. Click "OK"
5. Double-click the connection to open

**Features**:
- Real-time connection pool graphs
- Thread monitoring
- Memory heap analysis
- CPU profiling
- MBean browser

---

### 3. Java Mission Control (Advanced)

**Location**: Included with Oracle JDK or download separately

**Best For**:
- Production monitoring
- Performance tuning
- Flight recorder analysis

---

### 4. Tomcat Manager (Basic)

**Access**: `http://localhost:8080/manager/status`

**Shows**:
- Basic connection pool stats
- Active/idle connections
- Max connections configured

**Configuration**: Requires manager-gui role in `tomcat-users.xml`

---

## Key Metrics to Monitor

### Connection Pool Metrics

| Metric | What to Monitor | Healthy Range | Action Needed |
|--------|-----------------|---------------|---------------|
| **Active Connections** | Currently in use | 0-70% of maxTotal | If >80%, increase pool size |
| **Idle Connections** | Available for use | 10-30% of maxTotal | If 0, increase minIdle |
| **Connection Wait Time** | Time waiting for connection | <100ms | If >500ms, increase maxTotal |
| **Failed Connection Attempts** | Connection failures | 0 | Investigate database/network |
| **Abandoned Connections** | Not properly closed | 0 | Fix code leaks |
| **Validation Failures** | Stale connection tests | <1% | Check database stability |

---

### Monitoring Dashboard Example

**What to Watch**:
```
✅ Healthy Pool:
- Active: 15/100 (15%)
- Idle: 25/30 (83%)
- Wait Time: 5ms average
- Failed Attempts: 0
- Abandoned: 0

⚠️ Warning Signs:
- Active: 85/100 (85%) → Need more connections
- Idle: 0/30 (0%) → Pool exhausted
- Wait Time: 850ms → Requests queuing
- Failed Attempts: 12 → Database issues
- Abandoned: 5 → Code leaks

🔴 Critical Issues:
- Active: 100/100 (100%) → Pool maxed out
- Wait Time: 5000ms+ → Timeout errors imminent
- Failed Attempts: 50+ → Database down
- Abandoned: 20+ → Severe connection leaks
```

---

## Best Practices

### 1. Always Use Try-With-Resources

**❌ Bad Practice**:
```java
Connection conn = dataSource.getConnection();
PreparedStatement stmt = conn.prepareStatement(sql);
ResultSet rs = stmt.executeQuery();
// Connection might not be closed if exception occurs
conn.close();
```

**✅ Best Practice**:
```java
try (Connection conn = dataSource.getConnection();
     PreparedStatement stmt = conn.prepareStatement(sql);
     ResultSet rs = stmt.executeQuery()) {
    
    // Process results
    // All resources automatically closed, even on exception
}
```

**Why**: Ensures connections always return to pool, even on exceptions.

---

### 2. Set Appropriate Pool Sizes

**Based on Expected Load**:

| Application Type | Recommended maxTotal | minIdle | maxIdle |
|------------------|---------------------|---------|---------|
| Small app (<100 users) | 20-50 | 5 | 10 |
| Medium app (100-1000 users) | 50-100 | 10 | 30 |
| Large app (1000-10000 users) | 100-300 | 20 | 50 |
| High-traffic (>10000 users) | 300-500 | 50 | 100 |

**Formula**:
```
maxTotal = (Expected Concurrent Requests × Average Query Time) / 1000ms
minIdle = maxTotal × 0.2
maxIdle = maxTotal × 0.3
```

**Example**:
- 500 concurrent requests
- 200ms average query time
- maxTotal = (500 × 200) / 1000 = 100
- minIdle = 100 × 0.2 = 20
- maxIdle = 100 × 0.3 = 30

---

### 3. Enable Connection Validation

**Configuration**:
```xml
<Resource name="jdbc/dbServlet"
          testOnBorrow="true"
          testWhileIdle="true"
          validationQuery="SELECT 1"
          validationQueryTimeout="5"
          timeBetweenEvictionRunsMillis="30000"/>
```

**Why**: Detects and removes stale connections before they cause errors.

**Validation Query by Database**:
- **MySQL**: `SELECT 1`
- **PostgreSQL**: `SELECT 1`
- **Oracle**: `SELECT 1 FROM DUAL`
- **SQL Server**: `SELECT 1`
- **H2**: `SELECT 1`

---

### 4. Monitor Abandoned Connections

**Configuration**:
```xml
<Resource name="jdbc/dbServlet"
          removeAbandonedOnBorrow="true"
          removeAbandonedTimeout="60"
          logAbandoned="true"/>
```

**Benefits**:
- Detects connection leaks in code
- Logs stack traces showing where leak originated
- Automatically reclaims leaked connections

**Check Logs For**:
```
SEVERE: Connection was not closed, stack trace:
    at com.yourapp.SomeServlet.doGet(SomeServlet.java:45)
```

---

### 5. Use PreparedStatements

**❌ Bad Practice** (SQL Injection risk):
```java
String sql = "SELECT * FROM users WHERE username = '" + username + "'";
Statement stmt = conn.createStatement();
ResultSet rs = stmt.executeQuery(sql);
```

**✅ Best Practice**:
```java
String sql = "SELECT * FROM users WHERE username = ?";
try (PreparedStatement stmt = conn.prepareStatement(sql)) {
    stmt.setString(1, username);
    try (ResultSet rs = stmt.executeQuery()) {
        // Process results
    }
}
```

**Benefits**:
- Prevents SQL injection
- Better performance (query plan caching)
- Proper type handling

---

### 6. Test Configuration Changes in Development

**Process**:
1. Make pool config changes in dev environment
2. Run load tests with tools like JMeter or Gatling
3. Monitor pool metrics during tests
4. Adjust based on results
5. Only deploy to production after validation

**Load Testing Example** (Apache JMeter):
```
Test Plan:
- 100 concurrent users
- 1000 requests each
- Ramp-up: 30 seconds
- Monitor: Active connections, wait time, errors
```

---

### 7. Document Your Pool Settings

**Example Documentation**:
```markdown
## Production Pool Configuration
- **maxTotal**: 150
  - Reasoning: Peak load of 120 concurrent users + 25% buffer
- **minIdle**: 30
  - Keeps 20% of connections warm for quick response
- **maxIdle**: 45
  - Allows 30% idle during off-peak hours
- **removeAbandonedTimeout**: 60
  - Max query time in app is 45 seconds
- **Last Updated**: 2026-02-16
- **Updated By**: Dean Newkirk
- **Reason**: Increased traffic after new feature launch
```

---

### 8. Regular Monitoring of Pool Metrics

**Daily Checks**:
- [ ] Review connection pool graphs
- [ ] Check for abandoned connections
- [ ] Verify no failed connection attempts
- [ ] Monitor average wait times

**Weekly Checks**:
- [ ] Analyze peak usage patterns
- [ ] Review application logs for connection errors
- [ ] Check database slow query logs
- [ ] Verify pool settings still appropriate

**Monthly Checks**:
- [ ] Performance trending analysis
- [ ] Capacity planning review
- [ ] Update documentation
- [ ] Review and tune configuration

---

## Troubleshooting Guide

### Problem: High Active Connections (>80%)

**Symptoms**:
- Slow application response
- Users experiencing delays
- High CPU usage

**Investigation**:
```bash
# Check current pool status in JConsole
MBeans → Catalina → DataSource → jdbc/dbServlet → numActive

# Check application logs
grep "connection" catalina.out
```

**Solutions**:
1. Increase `maxTotal` in context.xml
2. Optimize slow queries (check database logs)
3. Look for connection leaks (enable logAbandoned)
4. Add read replicas if read-heavy

---

### Problem: Connection Wait Timeouts

**Symptoms**:
- Errors: "Timeout: Pool empty"
- Users getting 500 errors
- Long response times

**Solutions**:
1. Increase `maxTotal`
2. Reduce `maxWaitMillis` to fail faster
3. Optimize database queries
4. Check for connection leaks

---

### Problem: Abandoned Connections

**Symptoms**:
- Log shows: "Connection was not closed"
- Pool gradually exhausted
- Memory leaks

**Find the Leak**:
```xml
<Resource removeAbandonedOnBorrow="true"
          logAbandoned="true"/>
```

Check logs for stack trace showing leak location.

**Fix**:
Always use try-with-resources in the offending code.

---

## Performance Tuning

### Optimize for Read-Heavy Applications

```xml
<Resource maxTotal="200"
          maxIdle="100"
          minIdle="50"
          testOnBorrow="false"
          testWhileIdle="true"/>
```

**Reasoning**:
- More idle connections for burst traffic
- Skip validation on borrow (faster)
- Validate while idle instead

---

### Optimize for Write-Heavy Applications

```xml
<Resource maxTotal="100"
          maxIdle="20"
          minIdle="10"
          testOnBorrow="true"
          validationQuery="SELECT 1"/>
```

**Reasoning**:
- Fewer idle connections (writes less concurrent)
- Validate on borrow (ensure connection quality)
- Smaller pool (database write locks limit concurrency)

---

### Optimize for Long-Running Queries

```xml
<Resource maxTotal="50"
          maxWaitMillis="30000"
          removeAbandonedTimeout="300"
          validationQueryTimeout="10"/>
```

**Reasoning**:
- Smaller pool (queries hold connections longer)
- Longer wait time
- Longer abandoned timeout
- Longer validation timeout

---

## Related Files

- `connection-pooling.md` - Main connection pooling documentation
- `servlet-example.md` - Example servlet using the pool

---

## Repository Information
- **Repository**: newkirkdean-hub/webapp
- **Branch**: main
- **Created**: 2026-02-16
- **Purpose**: Connection pool monitoring and best practices guide