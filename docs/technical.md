### **Technical Documentation: Automated Attendance Management System**

**Version:** 2
**Date:** 02 July 2025

#### **1. Introduction**

This document provides a detailed technical overview of the Automated Attendance Management System. It is intended for software developers and system administrators who will build, maintain, or extend the system.

The project is designed to solve the inefficiencies of manual attendance tracking by providing a secure, scalable, and low-cost digital solution. The system comprises three main components:

1.  **A Flutter Mobile Application:** For employees to perform clock-ins, submit leave requests, and view their attendance history.
2.  **A React Web Dashboard:** For administrators to manage employees, oversee attendance records, approve leaves, and configure system settings.
3.  **A Firebase Backend:** A serverless platform providing authentication, database, and business logic execution.

This document details the system's architecture, the technology stack, core logic, database schema, security measures, and deployment procedures.

#### **2. System Architecture**

The system is built on a modern client-server architecture, with Firebase acting as the serverless backend-as-a-service (BaaS). The architecture is designed to be data-driven, with core business rules (e.g., time windows, penalty rules) being configurable in the database rather than hardcoded in the application logic.

**2.1. High-Level Architecture Diagram**

The following diagram illustrates the primary components and their interactions. Clients (Mobile App, Web Dashboard) communicate securely with Firebase services, which handle all business logic and data persistence.

```mermaid
graph TB
    subgraph CLIENT ["🎯 Client Layer"]
        direction TB
        A["📱 Flutter Mobile App<br/><br/>👤 Employee Portal<br/>━━━━━━━━━━━━━<br/>✓ Clock In/Out<br/>✓ Submit Leave Requests<br/>✓ View Attendance History<br/>✓ Real-time GPS Tracking"]
        B["💻 React Web Dashboard<br/><br/>👨‍💼 Admin Portal<br/>━━━━━━━━━━━━━<br/>✓ Employee Management<br/>✓ Attendance Oversight<br/>✓ Leave Approvals<br/>✓ System Configuration<br/>✓ Analytics & Reports"]
    end

    subgraph FIREBASE ["🔥 Firebase Backend"]
        direction TB
        C["🔐 Authentication<br/><br/>User Security Layer<br/>━━━━━━━━━━━━━<br/>• JWT Token Management<br/>• Role-Based Access<br/>• Session Control<br/>• Multi-factor Auth"]

        D["📊 Cloud Firestore<br/><br/>NoSQL Database<br/>━━━━━━━━━━━━━<br/>📋 users<br/>📅 attendance_records<br/>🏖️ leave_requests<br/>💰 penalties<br/>⚙️ company_settings"]

        E["⚡ Cloud Functions<br/><br/>Serverless Logic<br/>━━━━━━━━━━━━━<br/>🎯 handleClockIn()<br/>📍 Geofence Validation<br/>📊 Status Calculation<br/>💸 Penalty Processing<br/>📧 Notifications"]

        F["📁 Cloud Storage<br/><br/>File Repository<br/>━━━━━━━━━━━━━<br/>🏥 Medical Certificates<br/>📄 Leave Documents<br/>📸 Profile Pictures"]
    end

    subgraph LOGIC ["🧠 Business Intelligence"]
        direction TB
        G["⏰ Three-Check System<br/><br/>Time Management<br/>━━━━━━━━━━━━━<br/>🌅 Check 1: 08:30-09:15<br/>🍽️ Check 2: 13:00-14:00<br/>🌇 Check 3: 16:45-17:30<br/>⏱️ 30min Grace Period"]

        H["📍 Geofencing Engine<br/><br/>Location Intelligence<br/>━━━━━━━━━━━━━<br/>🛰️ GPS Coordinates<br/>📏 Distance Calculation<br/>🎯 Workplace Radius<br/>🚫 Anti-Spoofing"]

        I["💰 Penalty Calculator<br/><br/>Violation Management<br/>━━━━━━━━━━━━━<br/>🔴 4th Violation Rule<br/>💸 20 USD - Absent<br/>💸 15 USD - Half Day<br/>💸 10 USD - Late/Early"]
    end

    subgraph FLOW ["🔄 Data Pipeline"]
        direction LR
        J["📲 Clock Request<br/>Location + Auth"]
        K["✅ Validation<br/>Security + Location"]
        L["💾 Data Update<br/>Record Storage"]
        M["📊 Status Sync<br/>Final Calculation"]
    end

    A -.->|Secure Auth| C
    A ==>|API Calls| E
    A -.->|File Upload| F

    B -.->|Admin Auth| C
    B ==>|Direct Query| D
    B ==>|Admin Functions| E

    C ==>|Token Verify| E
    E ==>|Read/Write| D
    E -.->|File Ops| F

    E ==>|Time Logic| G
    E ==>|Location Logic| H
    E ==>|Penalty Logic| I

    J ==> K
    K ==> L
    L ==> M
    M -.-> J

    A -.-> J
    E -.-> K

    classDef clientStyle fill:#e8f4fd,stroke:#1565c0,stroke-width:3px,color:#0d47a1
    classDef firebaseStyle fill:#fff8e1,stroke:#ef6c00,stroke-width:3px,color:#bf360c
    classDef logicStyle fill:#f3e5f5,stroke:#7b1fa2,stroke-width:3px,color:#4a148c
    classDef flowStyle fill:#e8f5e8,stroke:#2e7d32,stroke-width:3px,color:#1b5e20
    classDef subgraphStyle fill:#fafafa,stroke:#424242,stroke-width:2px

    class A,B clientStyle
    class C,D,E,F firebaseStyle
    class G,H,I logicStyle
    class J,K,L,M flowStyle
    class CLIENT,FIREBASE,LOGIC,FLOW subgraphStyle
```

**2.2. Workflow Sequence Diagram: Clock-In Process**

This diagram details the sequence of events for a single, successful clock-in attempt by an employee. The critical takeaway is that the Flutter App never decides if a clock-in is valid; it only reports its location to the Cloud Function, which acts as the secure validator.

```mermaid
sequenceDiagram
    participant E as 📱 Employee<br/>Flutter App
    participant A as 🔐 Firebase<br/>Authentication
    participant F as ⚡ Cloud<br/>Functions
    participant D as 🗄️ Firestore<br/>Database
    participant G as 🗺️ Geofencing<br/>Engine
    participant T as ⏰ Time<br/>Validator
    participant P as 💰 Penalty<br/>Calculator
    participant N as 🔔 Notification<br/>Service

    Note over E,N: 🌅 MORNING CHECK-IN WORKFLOW

    E->>+A: 🚀 Initiate Login Request
    A-->>-E: ✅ Authentication Token

    E->>E: 📍 Capture GPS Location
    E->>+F: 🎯 handleClockIn(location, token, timestamp)

    Note over F: 🔍 SECURITY VALIDATION PHASE
    F->>+A: 🎫 Validate Auth Token
    A-->>-F: ✅ Token Valid + User Info

    Note over F: 📊 DATA RETRIEVAL PHASE
    F->>+D: 📋 Fetch User Profile
    D-->>-F: 👤 User Details + Role

    F->>+D: ⚙️ Get Company Settings
    D-->>-F: 🏢 Geofence Config + Time Windows

    F->>+D: 📅 Get Today's Attendance Record
    D-->>-F: 📊 Current Check Status

    Note over F: ⏰ TIME WINDOW VALIDATION
    F->>+T: 🕐 Validate Current Time Window
    T->>T: 🎯 Check Against:<br/>Morning: 08:30-09:15<br/>Lunch: 13:00-14:00<br/>Evening: 16:45-17:30
    T-->>-F: ✅ Window Valid (Check 1 - On Time)

    Note over F: 📍 GEOFENCE VALIDATION
    F->>+G: 🗺️ Validate Location
    G->>G: 📐 Calculate Distance<br/>from Workplace Center
    G->>G: 🎯 Check Against Radius<br/>(e.g., 100 meters)
    G-->>-F: ✅ Location Valid (Within Geofence)

    Note over F: 💾 ATTENDANCE RECORD UPDATE
    F->>+D: 📝 Update Attendance Record
    Note over D: 🔄 Set check1_timestamp<br/>🔄 Set check1_status: "on_time"<br/>🔄 Update daily_status
    D-->>-F: ✅ Record Updated Successfully

    Note over F: 📊 STATUS CALCULATION
    F->>F: 🧮 Count Completed Checks<br/>(1 of 3 completed)
    F->>F: 📊 Calculate Daily Status<br/>(In Progress)

    F-->>-E: 🎉 SUCCESS: Check-1 Completed!<br/>Status: On Time

    Note over E: 🔔 USER FEEDBACK
    E->>E: 📱 Display Success Message<br/>🎯 Update UI Status<br/>📊 Refresh Dashboard

    Note over E,N: 🍽️ LUNCH RETURN CHECK-IN

    E->>+F: 🎯 handleClockIn(location, token, timestamp)
    F->>+T: 🕐 Validate Time Window
    T-->>-F: ⚠️ Late Check (13:45 - 15min late)
    F->>+G: 🗺️ Validate Location
    G-->>-F: ✅ Location Valid
    F->>+D: 📝 Update Record
    Note over D: 🔄 Set check2_timestamp<br/>🔄 Set check2_status: "late"
    D-->>-F: ✅ Updated
    F-->>-E: ⚠️ SUCCESS: Check-2 Late<br/>Warning: Late arrival noted

    Note over E,N: 🌇 END OF DAY CHECK-OUT

    E->>+F: 🎯 handleClockIn(location, token, timestamp)
    F->>+T: 🕐 Validate Time Window
    T-->>-F: ✅ On Time (17:00)
    F->>+G: 🗺️ Validate Location
    G-->>-F: ✅ Location Valid
    F->>+D: 📝 Update Final Record
    Note over D: 🔄 Set check3_timestamp<br/>🔄 Set check3_status: "on_time"<br/>🔄 Set daily_status: "present"
    D-->>-F: ✅ Day Complete

    Note over F: 💰 PENALTY ASSESSMENT
    F->>+P: 📊 Assess Monthly Violations
    P->>+D: 📈 Count Month Violations
    D-->>-P: 📊 Violation Count: 3
    P->>P: 🎯 Check Penalty Rule<br/>(4th violation triggers penalty)
    P-->>-F: ✅ No Penalty (Under threshold)

    F-->>-E: 🎉 DAY COMPLETED!<br/>Status: Present<br/>All checks successful

    Note over E,N: 🔔 NOTIFICATION WORKFLOW
    F->>+N: 📧 Trigger Success Notification
    N->>E: 🔔 Push: "Daily attendance complete!"
    N->>+D: 📋 Log Notification
    D-->>-N: ✅ Logged
    N-->>-F: 📧 Notification Sent

    Note over E,N: ❌ ERROR SCENARIO EXAMPLE

    rect rgb(255, 245, 245)
        Note over E,N: 🚨 GEOFENCE VIOLATION SCENARIO
        E->>+F: 🎯 handleClockIn(remote_location, token)
        F->>+G: 🗺️ Validate Location
        G->>G: 📐 Distance: 2.5km from office<br/>🚫 Outside 100m radius
        G-->>-F: ❌ GEOFENCE VIOLATION
        F-->>-E: 🚫 FAILED: Outside workplace area<br/>Distance: 2.5km from office
        E->>E: 📱 Show Error Message<br/>🗺️ Display Map with location
    end

    rect rgb(255, 248, 245)
        Note over E,N: ⏰ TIME WINDOW VIOLATION
        E->>+F: 🎯 handleClockIn(location, token, 10:00)
        F->>+T: 🕐 Validate Time (10:00 AM)
        T->>T: ❌ All windows closed<br/>Morning: 08:30-09:15 (MISSED)<br/>Grace: 09:15-09:45 (EXPIRED)
        T-->>-F: ❌ NO VALID TIME WINDOW
        F-->>-E: 🚫 FAILED: No active check window<br/>Next window: 13:00-14:00
    end

    Note over E,N: 📊 END OF DAY ANALYTICS UPDATE
    F->>+D: 📈 Update Daily Analytics
    F->>+D: 📊 Update Monthly Stats
    D-->>-F: ✅ Analytics Updated
```

#### **3. Technology Stack & Libraries**

The project utilizes a specific set of modern technologies chosen for their efficiency, scalability, and suitability for a solo developer.

**3.1. Backend**

- **Firebase:** The core serverless backend.
  - **Cloud Firestore:** A NoSQL, document-based database for all application data.
  - **Firebase Authentication:** Handles secure user registration, login, and session management.
  - **Cloud Functions for Firebase:** Executes all secure, server-side business logic written in TypeScript.
  - **Cloud Storage:** Used for storing files, such as medical certificates uploaded for leave requests.
- **Node.js:** The runtime environment for executing Cloud Functions.
- **TypeScript:** Provides static typing for robust and maintainable Cloud Functions code.

**3.2. Mobile App (Flutter)**

- **Dart:** The programming language for Flutter.
- **Framework Packages:**
  - `firebase_core`, `firebase_auth`, `cloud_firestore`, `cloud_functions`: The FlutterFire suite for connecting to Firebase.
  - `geolocator`: To get the device's current GPS coordinates.
  - `table_calendar`: For creating a highly customizable calendar view for attendance and leave history.
  - `dynamic_color`: To implement Material 3's dynamic theming based on the user's wallpaper (Android).
  - `intl`: For robust date and time formatting.

**3.3. Admin Dashboard (React)**

- **Next.js:** A React framework providing Server-Side Rendering (SSR) and a powerful development experience.
- **TypeScript:** Ensures type safety across the entire web application.
- **Tailwind CSS:** A utility-first CSS framework for rapid UI development.
- **Firebase (Web SDK v9):** For connecting the React app to Firebase services.
- **Component Libraries & UI:**
  - **Shadcn/ui:** A collection of beautifully designed, accessible, and customizable components used for `Card`, `Table`, `Dialog`, `Badge`, and `Calendar`.
  - **React-Leaflet:** A library for creating interactive map views to allow the admin to set the workplace geofence.

#### **4. Detailed Geofencing & Core Logic Implementation**

**4.1. The Role of Firebase Cloud Functions**
All critical business logic is executed within Firebase Cloud Functions to ensure security and data integrity. The primary function `handleClockIn` is an HTTPS Callable Function.

**4.2. Core Logic: The Configurable Three-Check Attendance System**
The system's logic is highly flexible, driven by rules stored in the `COMPANY_SETTINGS` collection in Firestore. This allows administrators to adjust business rules without requiring a code deployment.

- **Time Windows & Grace Periods:** The `handleClockIn` function dynamically fetches the `timeWindows` and `gracePeriods` objects from `COMPANY_SETTINGS` before validating the timestamp of a check-in request.
- **Attendance Status Logic:** The final status for the day is calculated based on the number of valid checks recorded:
  - **Present:** All **three** checks are successfully completed.
  - **Half-Day Absent:** Exactly **two** checks are completed.
  - **Absent:** **One or zero** checks are completed.
- **Late Arrival & Early Leave Logic:** The definitions for late/early checks are configured in the `gracePeriods` object in settings. A check outside the configured grace period is considered a missed check and is not recorded.

**4.3. Penalty & Leave System**

- **Violation Tracking:** Every time a negative status (`late`, `early_leave`) is recorded in an attendance document, or a day is finalized as `absent` or `half_day_absent`, a corresponding document is created in the `VIOLATION_HISTORY` collection.
- **Penalty Incurrence:** A separate, scheduled Cloud Function (`calculateMonthlyPenalties`) can be run at the end of each month. This function reads the `VIOLATION_HISTORY` for each user, applies the `penaltyRules` from `COMPANY_SETTINGS`, and creates documents in the `PENALTIES` collection if the violation count exceeds the configured threshold.
- **Leave Logic:** When an admin approves a leave request via the dashboard, a Cloud Function (`handleLeaveApproval`) is triggered. This function validates the request, deducts the balance from the user's profile, updates the leave request status, and can retroactively create `ATTENDANCE_RECORDS` with the status "on_leave" for the approved dates.

#### **5. Database Schema (Firestore Data Model)**

The database is structured into several collections to normalize data and ensure scalability. Below is a detailed description of each collection based on the project's ERD.

- **`USERS`**: Stores all user-related information.

  - `userId` (PK), `email` (UK), `fullName`, `role` ("employee", "admin"), `department`, `position`, `isActive`, `fullLeaveBalance`, `halfLeaveBalance`, `medicalLeaveBalance`, `maternityLeaveBalance`, `phoneNumber`, `profileImageUrl`, `createdAt`, `updatedAt`.

- **`ATTENDANCE_RECORDS`**: A record for each employee for each working day.

  - `recordId` (PK: `{userId}_{YYYY-MM-DD}`), `userId` (FK), `attendanceDate`, `status` ("present", "absent", etc.), `check1_timestamp`, `check1_status`, `check1_location`, `check2_timestamp`, `check2_status`, `check2_location`, `check3_timestamp`, `check3_status`, `check3_location`, `totalWorkingHours`, `notes`, `isManualEntry`, `approvedBy` (FK).

- **`LEAVE_REQUESTS`**: Manages all employee leave submissions.

  - `requestId` (PK), `userId` (FK), `leaveType`, `startDate`, `endDate`, `totalDays`, `reason`, `status` ("pending", "approved", etc.), `approvedBy` (FK), `documentUrl`.

- **`PENALTIES`**: Contains records of all financial penalties applied to users.

  - `penaltyId` (PK), `userId` (FK), `dateIncurred`, `violationType`, `amount`, `status` ("active", "waived", etc.), `approvedBy` (FK).

- **`VIOLATION_HISTORY`**: A log of every individual attendance violation.

  - `historyId` (PK), `userId` (FK), `violationDate`, `violationType`, `monthlyCount`, `penaltyTriggered`, `penaltyId` (FK).

- **`COMPANY_SETTINGS`**: A singleton document (ID: "main") that configures the system's behavior.

  - `companyName`, `workplace_center` (GeoPoint), `workplace_radius` (number), `timeWindows` (object), `gracePeriods` (object), `penaltyRules` (object), `leavePolicy` (object), `timezone`, `workingDays`, `holidays`.

- **`NOTIFICATIONS`**: Stores user-facing notifications.

  - `notificationId` (PK), `userId` (FK), `title`, `message`, `category`, `isRead`, `relatedId`.

- **`AUDIT_LOGS`**: Tracks significant actions taken within the system for security and compliance.
  - `logId` (PK), `userId` (FK), `action`, `resource`, `resourceId`, `oldValues`, `newValues`, `timestamp`, `status`.

#### **6. Security Considerations**

1.  **Authentication:** All access is gated by Firebase Authentication.
2.  **Authorization:** Firestore Security Rules must be written to reflect the complex model and roles ("admin", "employee").
    - Employees can read their own documents across all relevant collections.
    - Admins have read/write access to most collections for management purposes.
    - **Crucially, no client can directly write to `AUDIT_LOGS`, `PENALTIES`, or `VIOLATION_HISTORY`. These collections are written to exclusively by trusted server-side Cloud Functions.**
3.  **Data Integrity:** All business logic is executed in Cloud Functions, providing a secure, server-authoritative environment.

#### **7. Environment Setup & Deployment**

1.  **Prerequisites:** Install Node.js, Flutter SDK, Firebase CLI, and Vercel CLI.
2.  **Project Setup:** Clone repositories and install dependencies (`npm install`, `flutter pub get`).
3.  **Firebase Configuration:** Create a Firebase project and store configuration keys in `.env` files. These files must not be committed to version control.
4.  **Deployment:**
    - **Cloud Functions:** `firebase deploy --only functions`
    - **Firestore Rules:** `firebase deploy --only firestore:rules`
    - **React Admin Dashboard:** `vercel deploy --prod`
    - **Flutter Mobile App:** Build and deploy through the Google Play Store and Apple App Store.
