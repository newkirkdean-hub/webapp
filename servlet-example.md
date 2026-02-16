# Voucher Viewer Servlet

## Overview
A Java servlet that retrieves voucher records from a database and returns them as JSON. This servlet handles voucher data display functionality for the web application.

---

## Servlet Information

| Property | Value |
|----------|-------|
| **Package** | `com.database.getlists` |
| **Class Name** | `VoucherViewer` |
| **URL Pattern** | `/voucherviewer` |
| **HTTP Method** | GET |
| **Response Type** | JSON |

---

## Java Code

```java
package com.database.getlists;

import javax.naming.InitialContext;
import javax.naming.NamingException;
import javax.sql.DataSource;
import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.sql.Connection;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

@WebServlet(name = "VoucherViewer", urlPatterns = {"/voucherviewer"})
public class VoucherViewer extends HttpServlet {
    private DataSource dataSource;

    @Override
    public void init() throws ServletException {
        try {
            // Lookup the JNDI DataSource
            InitialContext ctx = new InitialContext();
            dataSource = (DataSource) ctx.lookup("java:comp/env/jdbc/dbServlet");
        } catch (NamingException e) {
            throw new ServletException("Failed to lookup JNDI DataSource", e);
        }
    }

    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws IOException {
        response.setContentType("application/json");
        response.setCharacterEncoding("UTF-8");

        // SQL query to retrieve voucher data
        String sql = "SELECT Employee_ID, Date, Day, Emp_Name, Voucher_Amount, Tclock_ID, Reason FROM Voucher";

        // Prepare the response list
        List<Map<String, Object>> vouchers = new ArrayList<>();

        try (Connection conn = dataSource.getConnection();
             PreparedStatement stmt = conn.prepareStatement(sql);
             ResultSet rs = stmt.executeQuery()) {

            // Parse the database result set
            while (rs.next()) {
                Map<String, Object> voucher = new HashMap<>();
                voucher.put("Employee_ID", rs.getLong("Employee_ID"));
                voucher.put("Date", rs.getString("Date"));
                voucher.put("Day", rs.getString("Day"));
                voucher.put("Emp_Name", rs.getString("Emp_Name"));
                voucher.put("Voucher_Amount", rs.getDouble("Voucher_Amount"));
                voucher.put("Tclock_ID", rs.getString("Tclock_ID"));
                voucher.put("Reason", rs.getString("Reason"));
                vouchers.add(voucher);
            }

            // Convert list to JSON and write the response
            response.getWriter().write(mapListToJson(vouchers));

        } catch (SQLException e) {
            e.printStackTrace();
            response.setStatus(HttpServletResponse.SC_INTERNAL_SERVER_ERROR);
            response.getWriter().write("{\"error\":\"Database query failed.\"}");
        }
    }

    
    private String mapListToJson(List<Map<String, Object>> data) {
    StringBuilder json = new StringBuilder("[");
    for (int i = 0; i < data.size(); i++) {
        Map<String, Object> map = data.get(i);
        json.append("{");
        json.append("\"Employee_ID\"").append(":\"" + escapeJson(map.get("Employee_ID").toString()) + "\", ");
        json.append("\"Date\"").append(":\"" + escapeJson(map.get("Date").toString()) + "\", ");
        json.append("\"Day\"").append(":\"" + escapeJson(map.get("Day").toString()) + "\", ");
        json.append("\"Emp_Name\"").append(":\"" + escapeJson(map.get("Emp_Name").toString()) + "\", ");
        json.append("\"Voucher_Amount\"").append(":\"" + map.get("Voucher_Amount") + "\", "); // Numbers don't need escaping
        json.append("\"Tclock_ID\"").append(":\"" + escapeJson(map.get("Tclock_ID").toString()) + "\", ");
        json.append("\"Reason\"").append(":\"" + escapeJson(map.get("Reason").toString()) + "");
        json.append("}");
        if (i < data.size() - 1) {
            json.append(",");
        }
    }
    json.append("]");
    return json.toString();
}

private String escapeJson(String input) {
    if (input == null) {
        return ""; // null-safe handling
    }
    // Replace problematic characters with their escaped forms
    return input.replace("\\", "\\\\") // Escape backslashes
                .replace("\"", "\\\"") // Escape quotes
                .replace("\n", "\\n") // Escape newline
                .replace("\r", "\\r") // Escape carriage return
                .replace("\t", "\\t"); // Escape tabs
}
}
}
```

---

## Methods Documentation

### `init()`
**Purpose**: Initialize the servlet and establish database connection pool

**Process**:
1. Creates an InitialContext
2. Looks up the JNDI DataSource: `java:comp/env/jdbc/dbServlet`
3. Throws ServletException if lookup fails

**Called**: Once when servlet is first loaded

---

### `doGet(HttpServletRequest, HttpServletResponse)`
**Purpose**: Handle GET requests to retrieve voucher data

**Parameters**:
- `request`: HTTP request object
- `response`: HTTP response object

**Process**:
1. Sets response type to JSON with UTF-8 encoding
2. Executes SQL query to retrieve all vouchers
3. Parses ResultSet into a List of Maps
4. Converts data to JSON format
5. Writes JSON to response
6. Returns error JSON on SQLException

**Returns**: JSON array of voucher objects or error object

---

### `mapListToJson(List<Map<String, Object>>)`
**Purpose**: Convert list of maps to JSON string

**Parameters**:
- `data`: List of voucher records as Map objects

**Process**:
1. Manually constructs JSON array string
2. Iterates through each voucher record
3. Escapes special characters in string values
4. Builds properly formatted JSON

**Returns**: JSON string representation

**Note**: Manual JSON construction (consider using Jackson or Gson library)

---

### `escapeJson(String)`
**Purpose**: Escape special characters for safe JSON output

**Parameters**:
- `input`: Raw string value

**Escapes**:
- Backslashes: `\` → `\\`
- Quotes: `"` → `\"`
- Newlines: `\n` → `\\n`
- Carriage returns: `\r` → `\\r`
- Tabs: `\t` → `\\t`

**Returns**: Escaped string safe for JSON

**Null Safety**: Returns empty string for null input

---

## Database Schema

### Voucher Table Columns

| Column Name | Data Type | Java Type | Description |
|-------------|-----------|-----------|-------------|
| `Employee_ID` | BIGINT | Long | Employee identifier |
| `Date` | VARCHAR/DATE | String | Voucher date |
| `Day` | VARCHAR | String | Day of week |
| `Emp_Name` | VARCHAR | String | Employee name |
| `Voucher_Amount` | DECIMAL | Double | Voucher value |
| `Tclock_ID` | VARCHAR | String | Time clock identifier |
| `Reason` | VARCHAR | String | Voucher reason/notes |

---

## JSON Response Format

### Success Response
```json
[
  {
    "Employee_ID": "12345",
    "Date": "2026-02-16",
    "Day": "Monday",
    "Emp_Name": "John Doe",
    "Voucher_Amount": "25.50",
    "Tclock_ID": "TC001",
    "Reason": "Overtime meal voucher"
  },
  {
    "Employee_ID": "67890",
    "Date": "2026-02-15",
    "Day": "Sunday",
    "Emp_Name": "Jane Smith",
    "Voucher_Amount": "30.00",
    "Tclock_ID": "TC002",
    "Reason": "Holiday shift meal"
  }
]
```

### Error Response
```json
{
  "error": "Database query failed."
}
```

---

## Configuration Requirements

### JNDI DataSource Setup
The servlet requires a JNDI DataSource configured in your application server:

**Resource Name**: `java:comp/env/jdbc/dbServlet`

**Example Configuration** (Tomcat context.xml):
```xml
<Resource name="jdbc/dbServlet"
          auth="Container"
          type="javax.sql.DataSource"
          maxTotal="100"
          maxIdle="30"
          maxWaitMillis="10000"
          username="your_db_user"
          password="your_db_password"
          driverClassName="com.mysql.cj.jdbc.Driver"
          url="jdbc:mysql://localhost:3306/your_database"/>
```

---

## Security Considerations

✅ **Good Practices**:
- Uses PreparedStatement (prevents SQL injection)
- Escapes JSON output (prevents XSS)
- Null-safe JSON escaping
- Try-with-resources for proper connection management

⚠️ **Potential Improvements**:
- Add authentication/authorization checks
- Implement pagination for large datasets
- Add input validation for potential query parameters
- Consider using JSON library (Jackson, Gson) instead of manual construction
- Add logging framework instead of printStackTrace
- Implement connection pooling optimization
- Add request throttling/rate limiting

---

## Usage Example

### Frontend Fetch Request
```javascript
async function loadVouchers() {
    try {
        const response = await fetch('/voucherviewer');
        if (!response.ok) {
            throw new Error('Failed to fetch vouchers');
        }
        const vouchers = await response.json();
        console.log('Vouchers:', vouchers);
        // Process voucher data...
    } catch (error) {
        console.error('Error loading vouchers:', error);
    }
}
```

---

## Repository Information
- **File**: servlet example.txt → servlet-example.md
- **Repository**: newkirkdean-hub/webapp
- **Branch**: main
- **Package**: com.database.getlists