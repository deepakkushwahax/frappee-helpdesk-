# How We Got Frappe Helpdesk Running (and Fixed the Email Issues)

Hey! So, you asked for a rundown of how we set this all up, the roadblocks we hit (and smashed), and specifically how to get those email notifications actually showing up in your inbox. Here is the full story and the guide.

## 1. The "Just Code" Setup (How to Run It)

We decided to keep it simple. Instead of compiling everything from scratch (which gave us headaches), we switched to the official pre-built Docker image.

**To start fresh:**

1.  Open your terminal in the `docker` folder.
2.  Run `docker compose up -d`.
3.  That's it.

The script (`init.sh`) does the heavy lifting: it sets up the database, creates the site `helpdesk.localhost`, and installs the app. Just wait a few minutes until you see "bench start" in the logs (`docker compose logs -f frappe`).

**Access it here:** [http://localhost:8000/helpdesk](http://localhost:8000/helpdesk)
**Login:** `Administrator` / `admin`

---

## 2. Issues We Faced (The Bumpy Road)

It wasn't smooth sailing initially. Here are the gremlins we had to fight:

- **Python 3.14 Crash**: The default Frappe image updated to Python 3.14, which broke _everything_ (syntax errors in the code).
  - _Fix_: We switched to the stable pre-built image `ghcr.io/frappe/helpdesk:stable`. This saved us from dependency hell.
- **Database "Access Denied"**: The app couldn't talk to the database because of strict permissions.
  - _Fix_: I updated the setup script to allow the database user to login from any container (`%` wildcard).
- **Emails Were "Muted"**: By default, the development setup acts like a mime—it doesn't send emails.
  - _Fix_: I added a line to `init.sh` to unmute emails and enable the background scheduler automatically.

---

## 3. The Email Configuration Saga (Important!)

This was the trickiest part. Even with the settings correct, emails weren't sending. Here is exactly what you need to do to make it work.

### Part A: The Google "Security" Check

If you use Gmail, your normal password **will not work**. Google will block the connection (we saw the "Daily Limit Exceeded" authentication error in the logs).

1.  Go to your Google Account > Security.
2.  Enable **2-Step Verification**.
3.  Search for **"App Passwords"**.
4.  Create one (name it "Frappe"). It gives you a 16-character code.
5.  **Use this code** as the password in Frappe's _Email Account_ settings.

### Part B: The Missing Rule (Why nothing happened)

Even with the connection working, the system stayed silent. Why? Because we hadn't told it _when_ to speak.

**You must create a Notification Rule:**

1.  Search for **"Notification List"** in the app.
2.  Create a **New** notification.
3.  **Subject**: `New Ticket Created`
4.  **Document Type**: `HD Ticket` (This is the crucial one! Don't select generic "Ticket").
5.  **Send Alert On**: `New`.
6.  **Recipients**:
    - To test quickly: Just add your own email in the recipients table.
    - To do it properly: Scroll to "Send to Custom Field" and type `contact_email` (so the customer gets it) or `owner` (so the agent gets it).
7.  **Save**.

Now, when you create a ticket, the system knows: "Aha! A new HD Ticket! I should email this person."

---

## 4. Troubleshooting Cheat Sheet

If emails stop working:

- **Check the Queue**: Search for "Email Queue".
  - _Status "Sending"?_ -> Probably a network or scheduler issue.
  - _Status "Error"?_ -> Open it to see the message. If it says "SMTP data error", Google is blocking you (check your App Password).
- **Force Send**: If things are stuck, run this in the terminal to kick the queue:
  ```bash
  docker compose exec -w /home/frappe/frappe-bench frappe bench --site helpdesk.localhost execute frappe.email.queue.flush
  ```

That's pretty much it! You have a stable, containerized Helpdesk with working email alerts.
