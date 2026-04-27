# Product Requirements Document (PRD) & MVP Scope

**Project Name:** Asyut Lawyers Club & Resort Management System  
**Version:** 1.0  
**Status:** Final Draft  
**Date:** October 2023  

---

## 1. Executive Summary
The Asyut Lawyers Club & Resort Management System is a locally hosted web application designed to digitize operations for a resort with 15 rooms and a private lawyers' club. The system handles room reservations, membership management, housekeeping, and financial reporting.

The application utilizes a Client-Server architecture where a central Windows server hosts the database and backend, while staff members access the interface via standard web browsers on the Local Area Network (LAN). The system features Role-Based Access Control (RBAC), robust ID encryption for guest privacy, and PWA (Progressive Web App) capabilities for a native-app-like experience.

---

## 2. System Architecture & Deployment

### 2.1 Architecture
*   **Model:** Client-Server (Local Area Network).
*   **Server:** A Windows machine running the Backend API and Database.
*   **Clients:** Any PC/Mobile connected to the local network router, accessing the app via a static LAN IP (e.g., `http://192.168.1.50:3000`).

### 2.2 Technology Stack
*   **Frontend:** React.js + Vite.
*   **Styling:** Tailwind CSS or Material UI.
*   **Backend:** Node.js (Express) or Python (FastAPI).
*   **Database:** SQLite (recommended for ease of local setup) or PostgreSQL.
*   **Printing:** Browser-native print API.

### 2.3 Security
*   **Data Encryption:** AES-256 encryption for all stored ID images.
*   **Authentication:** JWT (JSON Web Tokens) for session management.
*   **Access Control:** Middleware route protection based on User Roles.

### 2.4 Installation & Access
*   **Installer:** A Windows Executable (`.exe`) package that installs the database, backend service, and serves the frontend build.
*   **PWA (Progressive Web App):** The application will be installable on client machines. Officers can click an "Install" button in the browser to add a desktop shortcut, opening the app in a standalone window (no URL bars/tabs).

---

## 3. User Roles & Permissions (RBAC)

### A. Administrator
*   **Profile:** General Manager / System Owner.
*   **Access:** Full access.
*   **Permissions:**
    *   Manage User Accounts (Create/Disable Officers).
    *   Configure Global Settings (Member Discount %, Receipt Template).
    *   Manage Room Inventory (Edit Prices, Block Rooms).
    *   View Analytics & Financial Reports.
    *   View Audit Logs.

### B. Membership Officer
*   **Profile:** Club Secretary.
*   **Access:** Restricted to Membership Module.
*   **Permissions:**
    *   Register new members (Create).
    *   Update member details (Update).
    *   Search/View member list (Read).
    *   Renew memberships.

### C. Reservation Officer
*   **Profile:** Front Desk Receptionist.
*   **Access:** Reservations, Housekeeping, and Room Status.
*   **Permissions:**
    *   Create/Modify/Cancel Reservations.
    *   Check-in / Check-out guests.
    *   Process payments and print receipts.
    *   Upload ID documents.
    *   Update Housekeeping status (Clean/Dirty).

---

## 4. Functional Requirements

### 4.1 Membership Module
*   **Registration Form:** 
    *   Fields: Name, Phone, Address.
    *   **Mandatory Field:** Lawyers Syndicate Member Number (Unique constraint).
*   **Membership Status:** Active vs. Expired flags.
*   **Integration:** Members linked to reservations receive automatic discounts.

### 4.2 Room Management Module
*   **Inventory:** Fixed list of 15 rooms (14 Standard, 1 Suite).
*   **Room Attributes:**
    *   Room Number, Type, View Type (Garden/Pool).
    *   **Price Per Night:** Editable by Admin.
    *   Status: Available, Occupied, Reserved, Maintenance.
*   **Plan View:**
    *   Visual map of the resort layout.
    *   Color-coded status indicators.
    *   Click-to-select functionality for reservations.

### 4.3 Reservation & Check-in Module
**Workflow:**
1.  **Room Selection:** Officer selects dates and picks a room from the Plan View.
2.  **Guest Details:**
    *   Toggle "Is Member?" -> Auto-applies Admin-defined discount.
    *   Input: Reserver Name, Phone, Address.
    *   **ID Capture:** Upload image of Reserver’s ID (Encrypted).
3.  **Companions Logic:**
    *   "Add Companion" button creates a list entry.
    *   **Fields:** Name, Relation (Dropdown).
    *   **Conditional ID Upload:**
        *   If Relation == "Family" (Spouse/Child/Parent): ID Upload is hidden/optional.
        *   If Relation != "Family" (Friend/Colleague/Other): ID Upload Drop Zone appears and is **Mandatory**.
4.  **Payment:**
    *   Calculate Total (Nights × Rate - Discount).
    *   Fields: Amount Paid, Payment Method (Cash/Card/Transfer).
5.  **Finalize:** 
    *   Save Reservation.
    *   Trigger Receipt Printing.
    *   Update Room Status to "Occupied".

### 4.4 Housekeeping Module
*   **Dashboard:** Simple list view of rooms.
*   **Status Toggle:** "Dirty" (Red) -> "Clean" (Green).
*   **Logic:** Warning displayed if trying to check-in to a "Dirty" room.

### 4.5 Admin Tools & Analytics
*   **Price Management:** Form to update `Price Per Night` for any room.
*   **Receipt Editor:** Rich text editor to customize Receipt Header, Footer, and Terms.
*   **Analytics Dashboard:**
    *   Total Revenue (Daily/Monthly).
    *   Occupancy Rate %.
    *   New Members Count.
*   **Audit Log:** Table tracking User, Timestamp, Action (e.g., "Officer X deleted Reservation #123").

### 4.6 Data Migration
*   **Historical Data:** A utility script to import existing Excel sheets (Members/Reservations) into the database during setup.

---

## 5. Data Schema (Simplified)

### Users
| Field | Type | Notes |
| :--- | :--- | :--- |
| id | UUID | PK |
| username | String | Unique |
| password_hash | String | Bcrypt hashed |
| role | Enum | 'Admin', 'MembershipOfficer', 'ReservationOfficer' |

### Rooms
| Field | Type | Notes |
| :--- | :--- | :--- |
| id | UUID | PK |
| room_number | String | e.g., "101" |
| type | Enum | 'Standard', 'Suite' |
| price_per_night | Decimal | Editable by Admin |
| status | Enum | 'Available', 'Occupied', 'Maintenance' |
| housekeeping_status | Enum | 'Clean', 'Dirty' |

### Members
| Field | Type | Notes |
| :--- | :--- | :--- |
| id | UUID | PK |
| name | String | |
| syndicate_number | String | **Mandatory, Unique** |
| phone | String | |

### Reservations
| Field | Type | Notes |
| :--- | :--- | :--- |
| id | UUID | PK |
| room_id | FK | Links to Rooms |
| member_id | FK | Nullable |
| check_in | Date | |
| check_out | Date | |
| total_price | Decimal | |
| payment_method | Enum | 'Cash', 'Card', 'Transfer' |
| id_image_url | String | Encrypted path |

### Companions
| Field | Type | Notes |
| :--- | :--- | :--- |
| id | UUID | PK |
| reservation_id | FK | Links to Reservation |
| name | String | |
| relation | String | 'Family', 'Friend', etc. |
| id_image_url | String | Encrypted path (Mandatory if non-family) |

---

## 6. MVP Scope (Minimum Viable Product)

### Phase 1: Infrastructure & Core Setup
1.  **Project Scaffold:** React Vite setup, Express backend, Database connection.
2.  **Installer:** Basic Windows service wrapper to start server on boot.
3.  **Authentication:** Login screen and JWT implementation.
4.  **Room Seed:** Populate database with 15 rooms.

### Phase 2: Reservation "Happy Path"
1.  **Reservation Form:** Reserver details input + ID Upload.
2.  **Companion Logic:** Add companion button + Relation conditional logic.
3.  **Price Calculation:** Basic math (Nights * Rate).
4.  **Payment:** Record payment details.
5.  **Printing:** Hardcoded receipt template (PDF/Browser Print).

### Phase 3: Management Features
1.  **Membership:** Add/Edit Member forms.
2.  **Admin Room Prices:** Interface to update prices.
3.  **Housekeeping:** Toggle room status.
4.  **Plan View:** Visual UI for room selection.

### Phase 4: Polish & Security
1.  **Encryption:** Implement AES-256 for ID images.
2.  **PWA Manifest:** Enable "Install App" feature.
3.  **Analytics:** Charts for Admin Dashboard.
4.  **Receipt Editor:** Allow custom receipt text.

