# Kidsclub Web App Printing System Documentation

**Date Created:** 2026-02-08  
**Author:** newkirkdean-hub  
**Purpose:** Reference documentation for the client-side printing architecture using escpos-coffee.jar

---

## Table of Contents
1. [System Architecture Overview](#system-architecture-overview)
2. [Technology Stack](#technology-stack)
3. [Component Breakdown](#component-breakdown)
4. [Data Flow](#data-flow)
5. [File Structure](#file-structure)
6. [Deployment Guide](#deployment-guide)
7. [Configuration](#configuration)
8. [Troubleshooting](#troubleshooting)

---

## System Architecture Overview

The Kidsclub printing system uses a **client-side printing architecture** where each workstation runs a local Java print agent. This approach is ideal for local network environments.


---

---

## Technology Stack

### Server-Side (Tomcat)
- **Apache Tomcat** - Web server
- **JSP (JavaServer Pages)** - User interface
- **Servlet API** - Request handling
- **JDBC** - Database connectivity (optional)
- **Gson** - JSON serialization

### Client-Side (Workstation)
- **Java SE 11+** - Runtime environment
- **Java AWT/Swing** - System tray UI
- **escpos-coffee.jar** - ESC/POS printer formatting
- **com.sun.net.httpserver** - Built-in HTTP server
- **Gson** - JSON parsing
- **Java Print Service API** - Printer communication

---

## Component Breakdown

### 1. printVoucher.jsp
**Location:** `/webapp/printVoucher.jsp` (Tomcat Server)

**Purpose:** 
- User interface for entering or selecting voucher data
- Sends print request to servlet (optional)
- Communicates with local print agent via JavaScript

**Key Features:**
- HTML form for voucher data entry
- JavaScript fetch API for HTTP communication
- Error handling and user feedback
- AJAX calls to both servlet and print agent

**Called By:** User in web browser

**Calls:** 
- `VoucherServlet.java` (to get data)
- `http://localhost:9100/printVoucher` (to print)

---

### 2. VoucherServlet.java
**Location:** `/src/main/java/com/yourcompany/servlet/VoucherServlet.java` (Tomcat Server)

**Purpose:**
- Receives voucher request from JSP
- Queries database for employee/voucher details
- Enriches data (company info, calculations)
- Returns complete JSON to JSP

**Key Features:**
- Database connectivity
- Data validation
- JSON response generation
- Error handling

**Called By:** `printVoucher.jsp`

**Calls:** Database layer

**Note:** This component is **optional** if voucher data is entered manually in JSP without database lookup.

---

### 3. PrintServiceTrayApp.java
**Location:** Client Workstation (System Tray Application)

**Purpose:**
- **Main application** that runs in system tray
- HTTP server listening on `localhost:9100`
- Receives JSON voucher data
- Routes to printing method
- Provides user interface for monitoring and testing

**Key Features:**
- System tray icon with right-click menu
- Visual status indicator (running/stopped/error)
- Built-in HTTP server for receiving print requests
- JSON parsing
- User-friendly status checking
- Test print functionality
- Auto-start capability
- CORS headers for local development
- Error handling and logging
- Notification messages for print status

**User Interface:**
**Called By:** 
- Started automatically on Windows login (when configured)
- JavaScript in `printVoucher.jsp` sends HTTP requests

**Calls:** `ReceiptRouter.route()`

**Runs As:** 
- Background system tray application
- Auto-starts with Windows (optional)
- Always visible in system tray for easy access

**Dependencies:**
```xml
<dependency>
    <groupId>com.github.anastaciocintra</groupId>
    <artifactId>escpos-coffee</artifactId>
    <version>4.1.0</version>
</dependency>
<dependency>
    <groupId>com.google.code.gson</groupId>
    <artifactId>gson</artifactId>
    <version>2.10.1</version>
</dependency>
```

---

## Repository Information
- **Original File**: Kidsclub Web App Printing (from newkirkdean-hub/KIDSCLUB)
- **Copied To**: newkirkdean-hub/webapp
- **Date Copied**: 2026-02-16