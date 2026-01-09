# Project Issues and Resolutions

This document tracks the technical challenges encountered during the setup and configuration of the Frappe Helpdesk POC and the applied resolutions.

## 1. Environment & Setup Issues

### Issue: Python Version Incompatibility

- **Problem**: The default `latest` Docker image for Frappe utilized Python 3.14, which caused syntax errors and compatibility issues with the current codebase (expecting Python 3.10).
- **Resolution**: Switched to the `ghcr.io/frappe/helpdesk:stable` image tag to ensure a stable runtime environment with compatible Python versions.

### Issue: Database ConnectionRefused / Access Denied

- **Problem**: The Frappe application container could not connect to the MariaDB container due to strict host permissions (`'root'@'localhost'`).
- **Resolution**:
  - Created a dedicated `frappe_bridge` network in Docker Compose to isolate and facilitate container communication.
  - Updated the `mariadb` configuration to allow connections from any host (`%`) to permit container-to-container access.

## 2. Configuration & Logic Issues

### Issue: Email Notifications Muted

- **Problem**: By default, the development environment (Bench) mutes all outgoing emails to prevent accidental spam during testing.
- **Resolution**: Modified the `init.sh` script to explicitly unmute emails (`bench enable-email`) and enable the background scheduler (`bench enable-scheduler`) during initialization.

### Issue: SMTP Authentication Failure (Gmail)

- **Problem**: Configuring Gmail with the standard user password resulted in "Daily Limit Exceeded" or generic authentication errors due to Google's security policies for "Less Secure Apps".
- **Resolution**:
  - Enabled 2-Step Verification on the source Google Account.
  - Generated an **App Password** specifically for the "Frappe" application.
  - Used this 16-character App Password in the Frappe Email Account settings.

### Issue: No Emails Sending Despite Configuration

- **Problem**: Even with valid SMTP settings and unmuted emails, no notifications were being generated when tickets were created.
- **Resolution**: Identified missing business logic triggers.
  - Created a specific **Notification Rule** for Document Type `HD Ticket`.
  - Trigger Condition: `New`.
  - Configured recipients to map to `contact_email` or fixed recipients for testing.
