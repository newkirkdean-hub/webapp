# KIDSCLUB Migration Context & Working Notes

Version: 0.1  
Last updated: 2026-02-16 20:49:52
Author: Dean <newkirkdean@gmail.com> (GitHub: newkirkdean-hub)

Change log:
- 2026-02-16 — initial v0.1 — added ALL_CAPS convention and clarified Git branches

## Purpose / top-level overview
- "KIDSCLUB" is a small, local network Java/FX program that reads/writes a Microsoft Access DB for managing customer records, invoices, time clock, inventory, sales, maintenance, and simple reporting.[...]

## Overall goal 
- To migrate "KIDSCLUB" to a local network web service + MS Access while keeping user functionality and appearance as close to the same as is current. Migrate corp2 into same source files as the rest[[...]
- Target cutover June, 2026
- there should not be any down time during cutover as testing will continue until we have all the bugs fixed.
- browser support list (IE, Google Chrome, Brave).


## Names
- Top-level program will be referred to as KIDSCLUB; this is the all-inclusive program of the two branches COUNTERFX and CORP2.
- COUNTERFX and CORP2 will be referred to as branches (Git branches); I renamed two branches under the master Git branch. For reference, when I ask you to look at a file in COUNTERFX, that means look [...]
- Other folders under the COUNTERFX branch will be referred to as sub-folder. 
Example, "please read the file Games.FXML in sub-folder GAMES.
- Most of this Naming is for the owners needs so he can communicate to you. 
- KIDSCLUBWEB; this is the all-inclusive program of the current progress of the migration, this refers to all new work being done.

## Where the Access DB lives (non-secret pointer)
- "Microsoft Access Driver (*.accdb)"
- Access Database Engine 2016
- File path (local/dev): /**-NET-DRIVE/clubdb
- Shared location on local computer in our office, backup copies are on owners home computer and owner will provide table descriptions in text format.

## How to run the existing app (short)
- JDK version: "JDK 25.0.1"
- Build command: `./ANT`
- Run command:
    • java -jar Local-Jar-Files/corp2/dist/corp2.jar
    • java -jar Local-Jar-Files/counterFX/dist/counterFX.jar
- Testing, Owner has complete version at home for design and testing. This is how any future testing will be done.


## Important source files & entry points
- Main class: COUNTERFX and CORP2 (two programs; COUNTERFX is mostly dependent on JavaFX and CORP2 uses Swing)
- Data access: src/.../AccessDataRepository.java (reads .mdb via UCanAccess)
- UI / forms: There are two main forms in COUNTERFX called "counter" and "Main". CORP2's main form is "Corpform_Main2".

## High-level data model
- Number of tables: 38
- Most important tables: Members, Inventory, Timeclock, Sales, Emails, Member Mail.
- Tables themselves do not maintain referential integrety, The code will keep this. The only referenatial integrety is in the
    • Members Table
    • MembersDetail Table
- Referenatial integrity will be explored more when we get to that issue.

## Table-level notes (short per-table summary)
- Table name: Members
  • Rows (Current): 20,000 (100 new Members per Month avg.)
  • Usage: read-heavy; few writes/day
  • Sensitive columns: Member Number, Member ID, 
  • Migration note: preserve autonumber customer_id; emails have some NULLs; create unique index on email after cleanup.

- Table name: MembersDetail
  • Rows (est): 20,000 - 50,000 in 4 month period, we clean the detail records once they exceed 40,000+
  • Usage: read-heavy; Most writes/day of all tables
  • Sensitive columns: Member ID (Keyed To Members), 
  • Tables MEMBERS and MEMBERSDETAIL both must balance. MEMBERS has a single field "Balance" which is the total added and subtracted of all the MEMBERSDETAIL. These are the only tables that must sta[...]  
(Future link to MEMBERS and MEMBERSDETAIL balance page)

(For full data dictionary use docs/DATA_DICTIONARY.md)

## Current business logic implemented outside DB
- All Queries are run from code none are stored or run from the MSACCESS Database.
- Reports: "Reports will be the last thing tackled".

## Known bugs / oddities
- Currently the only consistent issue is dropping the connection to the tables. (Network issue) Solved with Database and code stored in same location.
- Because we clean the tables after 40,000+ records in the Detail and through them into a history table that only get used maybe once a month we have stopped the tables from breaking and having to rep[...]  

## Security & compliance (summary)
- Because we run only on a local network security is very simple at this time. We only use a valid user number that they type in and if valid in the database then they can use the system. Some users a[...]  
- see EmployeePasswordSecurityImprovments.md
- https://github.com/newkirkdean-hub/KIDSCLUB/blob/fd30676a7ebf0fdb11c8b905b4c83c74ec2f3193/EmployeePasswordSecurityImprovments.md

## Migration priorities and constraints
- Priority 1: Keep feel and look and functionality very similar. (CSS files)
- Priority 2: Migrate in sections (This is one large program with many sub folders / programs)
- Cutover estimate is June, 2026
- Data volume estimate: 2 MB total (Confirmed) Current in use DB.
- Owner has full working system at home to test changes before implementing them in the Office Server. 
- There is no urgency to this and Owner is not a full time programmer, he has a 40 hour a week Job.

## Local Office Network
- Topology is Peir to Peir connected thru Cat Cabling 
- 1 Windows stores all files
- 13 Windows User, 1 Mac User, 1 Windows Tablet.

## Contacts
- Owner: Dean <dean@example.com> (github: newkirkdean-hub)
- Other contributors: NONE

## How to use this file with the assistant
- When you want me to review/update context, either:
  • Paste the changed sections into chat; or
  • Tell me to "Please read KIDSCLUB Migration & Working Notes.md in the repo" and include a permalink to the file, or let me fetch it if you ask me to inspect the repository.
- Keep "Last updated" current so I know which version to use.

## Next steps (example)
1. Fill docs/DATA_DICTIONARY.md with full column lists and row counts.
2. Create docs/MIGRATION_PLAN.md with per-table migration approach.
3. Add docs/SECURITY.md with roles and encryption/backups policy.

## Schema files location and notes:
- All database schema files, migration SQL, and related README or import scripts will be stored under the repository path `db/schema/`.
- Each subsystem gets its own subfolder, for example:
  - `db/schema/copr/` — COPR (current COPR Postgres schema and README)
  - `db/schema/counterfx/` — COUNTERFX schema files
  - `db/schema/members/`, `db/schema/sales/`, etc. for other domains
- File naming convention: use `<subsystem>_schema.sql` for DDL, and `README.md` alongside it to document mapping notes and import guidance.
- Files are committed to the `master` branch for easy retrieval. If you prefer them in a feature branch during development, mention the branch name.
- Current files present:
  - `db/schema/copr/copr_schema.sql`
  - `db/schema/copr/README.md`
 
## Current Progress

### 1. Server Setup
- **Tomcat Installation and Configuration**
  - Installed Tomcat and configured it for the local testing and development environment.
  - Enabled connection pooling for the Microsoft Access database.

### 2. Database Connectivity
- **Database Access and Setup**
  - Configured database servlet:
    ```java
    @WebServlet(name = "DatabaseServlet", urlPatterns = {"/dbServlet"})
    @WebServlet("/initializeDatabase")
    @WebListener
    ```
  - Updated `web.xml` with the following resource reference:
    ```xml
    <res-ref-name>jdbc/dbServlet</res-ref-name>
    ```
  - Updated `context.xml` with resource:
    ```xml
    <Resource name="jdbc/dbServlet" url="jdbc:ucanaccess:///C:/Jar_Files_Local/member_tcat.accdb" />
    ```

### 3. UI Development
- **Initial Index Page**
  - Designed `index.html` to act as a screen chooser for navigation.
  - Functionality includes:
    - Checking `localStorage.getItem("lastScreen")` to determine the previous screen.
    - Buttons for database troubleshooting:
      - **DBConnectionPooling**: Validates that the database connection pool is active.
      - **ValidatePathToDatabase**: Confirms that the database is reachable.
      - **StartDatabase**: Links to a JSP that initializes the database.

### 4. Feature Progress
- **Completed Screens**
  - `tip_Calculator` (popup)
  - `passwordModal_include` (a reusable HTML file for securing sections with employee passwords)
- **In-Progress Screens**
  - `Bridge`: Includes `timeclockservlet`, and `vouchers`
  - `Cafe`: Includes `vouchers`, and `tip_Calculator`
  - `Vouchers`: Includes a popup interface for voucher management.

- **New Features**
  - Employee password validator (`EmployeePasswordValidationServlet`)
    - Authenticates employee numbers from the "employee" database table and constructs an `Employee` object for session handling.
  - Automated time clock updates:
    - `BridgeTimeClockServlet` checks changes in the time clock every 2 minutes and updates the bridge screen.
  - Folder Location for DB and XML files:
    - all files now stored on C drive in Kidsclub_Web/
    - first folder is /xml (useing docBase in server.xml, connector is listening on Port 8085 )
    - Context docBase="C:/Kidsclub_Web/XML" path="/xml" crossContext="true"