# Proof of Concept (POC): Frappe Helpdesk on Docker

## 1. Objective

The goal of this POC was to validate the feasibility of running **Frappe Helpdesk** in a containerized environment (Docker) and ensuring critical features—specifically **Email Notifications** for ticket updates—function correctly.

## 2. Environment Setup

- **Application**: Frappe Helpdesk (Stable Image)
- **Infrastructure**: Docker & Docker Compose
- **Database**: MariaDB 10.8 (On custom `frappe_bridge` network)
- **Caching/Queue**: Redis Alpine
- **OS**: Linux

## 3. Implementation Details

### A. Docker Architecture

We moved away from manual building to use the **pre-built Docker image** (`ghcr.io/frappe/helpdesk:stable`). This solved early Python version incompatibility issues (Python 3.14 vs 3.10) and ensured a stable runtime environment.

- **Network**: Created a dedicated `frappe_bridge` network to isolate traffic and fix database connection capability.
- **Persistence**: configured `init.sh` to handle site creation idempotently (it checks if the site exists before trying to create it).

### B. Email Notification Logic

To prove the system is "production-ready" for communication, we configured:

1.  **SMTP Relay**: Connected to Gmail using App Password authentication (bypassing the "Daily Limit" errors by validating credentials).
2.  **Background Scheduler**: Enabled the Frappe background worker (`enable-scheduler`) to process the `Email Queue` asynchronously.
3.  **Business Logic**: Created a specific **Notification Rule** (`New Ticket Created` -> `HD Ticket`) to trigger emails automatically.

## 4. Verification & Testing

| Feature             | Test Case                     | Status   | Notes                                             |
| :------------------ | :---------------------------- | :------- | :------------------------------------------------ |
| **Deployment**      | Run `docker compose up -d`    | **Pass** | Containers (frappe, redis, mariadb) healthy.      |
| **Access**          | Open `localhot:8000/helpdesk` | **Pass** | UI loads, Admin login successful.                 |
| **DB Connection**   | App connects to MariaDB       | **Pass** | Fixed initial "Access Denied" via `%` host scope. |
| **Ticket Creation** | Create ticket via Portal      | **Pass** | Record created in `HD Ticket` table.              |
| **Email Alert**     | Auto-email on new ticket      | **Pass** | **Success**: Email received in inbox.             |

## 5. Conclusion

This POC confirms that **Frappe Helpdesk can be successfully self-hosted using Docker**. The critical requirement of **automated email notifications** has been satisfied and verified.

The current setup in `/home/deepak/Desktop/helpdesk/docker` is stable and ready for use as a development or staging environment.
