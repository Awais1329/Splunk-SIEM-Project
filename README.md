### Splunk Log Monitoring — Server \& Forwarder Setup

I built this project to get hands-on experience with centralized log management. The idea was simple: set up a Splunk server on one machine and then configure a second machine to forward its logs to that server in real time. What sounds straightforward on paper involves quite a few moving parts — ports, configurations, receiver settings — so I documented everything here so anyone can follow along or reproduce it from scratch.

Objective:

Most systems generate logs constantly — login attempts, service errors, network activity, application events. The problem is those logs are stuck on whichever machine produced them. Splunk solves this by pulling all those logs into one place where you can search, filter, and build reports across everything at once.

In this project, I have two machines talking to each other:

* Machine 1 (Splunk Server): This is where all the logs land. Splunk Enterprise runs here, and this is where I do all my searching and analysis through the web interface.
* Machine 2 (Forwarder) This machine has Splunk Universal Forwarder installed. It monitors specific log files and ships them over the network to the server continuously.

Once both were connected and the logs started flowing in, I ran several SPL (Splunk Processing Language) queries to filter, analyze, and report on the data.

Why I Made This:

Setting up a SIEM-like environment is one of those skills that comes up constantly in system administration and cybersecurity roles. I wanted to actually build one rather than just read about it. Going through the process of configuring a forwarder, troubleshooting a connection that wasn't working, and then finally seeing live logs appear in the Splunk dashboard — that's the kind of experience you can't get from a video.

This project also gave me a good foundation for understanding how enterprise monitoring tools work at a basic level.

 Project Structure
 
Splunk-SIEM-project/

│

├── README.md              ← You are here

├── SETUP.md               ← Full installation and configuration walkthrough

├── queries.md             ← All SPL queries I used, with explanations

└── screenshots/           ← Visual proof of each step working



SETUP.md:walks through the entire process from installing Splunk Enterprise on the server to configuring the Universal Forwarder on the second machine and verifying that logs are actually being received. If you want to replicate this project, start there.

queries.md: contains every SPL search query I used during this project. Each query has a short explanation of what it does and when you'd use it. These range from basic "show me everything" searches to more specific filters like finding errors or grouping events by host.

screenshots: It holds all the screenshots I captured at each stage — the login page, the receiver configuration, the forwarder connection, and the final search results. If you want a visual sense of what everything looks like before attempting it yourself, browse through that folder first.

Tools and Technologies:

* Splunk Enterprise the main server that indexes and displays logs
* Splunk Universal Forwarder lightweight agent installed on the second machine to send logs
* Window OS both Devices ran Window during this setup
* SPL (Splunk Processing Language) the query language used to search through indexed log data

Things I Learned Along the Way:

Getting the forwarder to actually connect was the trickiest part. The server needs to have receiving explicitly enabled on port 9997 — it doesn't happen automatically just by installing Splunk. I also had to make sure that port wasn't being blocked by a firewall before the connection would go through. Once that was sorted, logs started appearing almost immediately, which was a satisfying moment.

For the full setup guide, go to  [View Setup](./configs/setup.md).  

For the search queries, go to [View Queries](./queries/detections.spl)

For screenshots, check the [View Screenshots](./screenshots/).

