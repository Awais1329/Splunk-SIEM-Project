# Setup Guide — Splunk Enterprise and Universal Forwarder on Windows

This guide covers everything I did to get this project running on two Windows machines, step by step. I have written it the way I wish someone had written it for me — with enough context that you understand why each step matters, not just what to click or type. Both machines in this setup run Windows, so all commands use the Windows Command Prompt and all installations use standard Windows `.msi` installers.

Before starting, make sure both machines are on and can reach each other over the network. The easiest way to test this is to open Command Prompt on one machine and ping the other using its IP address. Note down the IP address of your server machine early — you will need it during the forwarder configuration.

## Part 1 — Setting Up the Splunk Server (Machine 1)

 ###Step 1: Download Splunk Enterprise

Go to [splunk.com](https://www.splunk.com) and create a free account if you do not have one. Once logged in, go to the Downloads section and download **Splunk Enterprise** for Windows — it will be a `.msi` installer file. The file is around 500MB so it may take a few minutes depending on your connection.

### Step 2: Install Splunk Enterprise

Run the `.msi` installer you just downloaded. The installation wizard will guide you through the process:

- Accept the license agreement
- Choose the installation directory — the default `C:\Program Files\Splunk` is fine
- On the credentials screen, set an admin username and password. Write these down — you will use them every time you access the Splunk web interface
- On the final screen, make sure the option to **start Splunk as a local system service** is checked. This means Splunk will start automatically whenever Windows boots

Click Install and wait for it to finish. Windows may ask for permission to make changes — click Yes.

### Step 3: Open the Splunk Web Interface

Once installation is complete, open a web browser on the server machine and go to:

http://localhost:8000

You should see the Splunk login page. Log in with the admin username and password you set during installation. If the browser cannot connect, open **Services** in Windows (search for it in the Start menu) and confirm that the **Splunk** service is running. If it is stopped, right-click and start it.

### Step 4: Enable Receiving on Port 9997

This is the most important step on the server side and the one that is easiest to forget. By default, Splunk Enterprise does not listen for incoming forwarder connections — you have to turn this on manually.

Inside the Splunk web interface:

1. Click **Settings** in the top navigation bar
2. Under the **Data** section, click **Forwarding and Receiving**
3. Click **Configure Receiving**
4. Click **New Receiving Port** in the top right
5. Type `9997` in the port field and click Save

Port 9997 is the standard port that Splunk Universal Forwarder uses when sending data. The server and forwarder both need to use the same port number for the connection to work.

### Step 5: Open Port 9997 in Windows Firewall

Even after enabling receiving inside Splunk, Windows Firewall will block incoming connections on port 9997 by default. This is the step that silently breaks everything if you skip it — Splunk says it is listening, but the firewall never lets the connection through.

Open Command Prompt as Administrator (right-click the Start menu and choose "Command Prompt (Admin)") and run:

```cmd
netsh advfirewall firewall add rule name="Splunk Forwarder Port 9997" protocol=TCP dir=in localport=9997 action=allow
```

You should see the message `Ok.` confirming the rule was added. Now the server is genuinely ready to receive logs from a forwarder.


## Part 2 — Setting Up the Splunk Universal Forwarder (Machine 2)

### Step 6: Download the Universal Forwarder

On your **second Windows machine**, go to [splunk.com](https://www.splunk.com) and download **Splunk Universal Forwarder** for Windows. This is a separate, much smaller package compared to Splunk Enterprise — it has no web interface, just the engine that collects and forwards logs. The installer is also a `.msi` file.

### Step 7: Install the Universal Forwarder

Run the installer on the second machine:

- Accept the license agreement
- Set an admin username and password for the forwarder (these are just local credentials for managing the forwarder itself)
- On the **Deployment Server** screen, you can leave it blank for now
- On the **Receiving Indexer** screen, enter the IP address of your Splunk server machine and the port `9997`. For example: `192.168.1.10:9997`. This tells the forwarder where to send logs right from the start
- Leave the option to install as a Windows service enabled
- Click Install

If you missed the Receiving Indexer screen during installation, do not worry — you can configure it from the command line in the next step.

### Step 8: Connect the Forwarder to the Server (via Command Line)

Open Command Prompt as Administrator on the forwarder machine and navigate to the Splunk Forwarder installation directory:

```cmd
cd "C:\Program Files\SplunkUniversalForwarder\bin"
```

Run this command to point the forwarder at your server, replacing `<server-ip>` with the actual IP address of your Splunk server machine:

```cmd
splunk add forward-server <server-ip>:9997 -auth admin:yourpassword
```

A successful response looks like: `Added forwarding to: <server-ip>:9997`

If you get a connection error here, double-check that the firewall rule on the server was added correctly in Step 5.

### Step 9: Configure the Forwarder to Monitor Windows Event Logs

Now tell the forwarder which logs to collect and send. Windows Event Logs are organized into channels — the three main ones are Application, Security, and System. Run these commands to add all three:

```cmd
splunk add monitor "WinEventLog://Application" -auth admin:yourpassword
splunk add monitor "WinEventLog://Security" -auth admin:yourpassword
splunk add monitor "WinEventLog://System" -auth admin:yourpassword
```

These three channels cover the vast majority of meaningful Windows events — application crashes and errors, login attempts and security events, and system-level service and hardware events respectively.

### Step 10: Restart the Forwarder

After making configuration changes, restart the forwarder service so everything takes effect:

```cmd
splunk restart
```

Wait about 30 seconds for it to come back up fully.

---

## Part 3 — Verifying That Everything Works

### Step 11: Check That Logs Are Arriving on the Server

Go back to the Splunk web interface on the server machine. Click **Search & Reporting** from the home screen or the left navigation.

In the search bar, type:

```
index=main | head 20
```

Hit Enter. If the setup is working correctly, you should see Windows Event Log entries appearing in the results. Look at the `host` field — it should show the computer name of your forwarder machine, confirming those logs came from the second machine and not the server itself.

If nothing shows up, work through this checklist:

- Is the forwarder service running on Machine 2? Open Services in Windows and look for **SplunkForwarder** — it should show as Running
- Did you add the firewall rule on the server? Run the `netsh` command from Step 5 again to confirm
- Is the IP address in the forwarder configuration correct? Check it by running `splunk list forward-server` from the forwarder machine's command line
- Give it a minute or two — sometimes there is a short delay before the first batch of events appears

### Step 12: Confirm the Forwarder Appears in Forwarder Management

For additional confirmation, go to **Settings → Distributed Environment → Forwarder Management** in the Splunk web interface. Your second machine should appear in the list with a timestamp showing when it last connected to the server.

---

## Summary of the Complete Flow

```
Machine 2 (Forwarder - Windows)              Machine 1 (Server - Windows)
────────────────────────────────             ─────────────────────────────
Windows Event Log (Application)  ──┐
Windows Event Log (Security)     ──┼──→  SplunkForwarder  ──→  port 9997  ──→  Splunk Enterprise
Windows Event Log (System)       ──┘         service                           Web UI: port 8000
```

Once this pipeline is running, any new Windows Event Log entries on the forwarder machine appear in Splunk on the server within a few seconds. From there, everything can be searched, filtered, and turned into reports through the browser-based web interface.

