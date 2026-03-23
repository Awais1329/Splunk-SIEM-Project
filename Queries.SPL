Splunk Search Queries (SPL)

These are all the SPL queries I ran during this project. SPL — Splunk Processing Language — is what you type into the search bar to pull specific data out of your indexed logs. I've included a short explanation with each one so it's clear what it does and when you'd actually want to use it.

All of these can be pasted directly into the Splunk search bar. By default, Splunk searches the last 24 hours, but you can adjust the time range using the picker on the left side of the search bar.


Basic Searches:
View the most recent incoming logs
spl
index=main | head 20

This is the first query I ran after setting up the connection. It pulls the 20 most recent events from the main index — basically just confirming that logs are actually arriving from the forwarder. If you see results and the `host` field shows your forwarder machine's hostname, the connection is working.

index=main host="<forwarder-hostname>"

Once you have multiple machines sending logs to the same Splunk server, this becomes useful. Replace `<forwarder-hostname>` with the actual hostname of your forwarder machine. It filters results so you only see events that came from that specific host, ignoring everything else.

index=main "error"

A simple keyword search. Splunk will return every event that contains the word "error" anywhere in the raw log text. You can swap in any word — "failed", "denied", "warning" — depending on what you're looking for.

## Counting and Grouping

index=main | stats count by source

This one gives you a breakdown of how many log entries came from each file being monitored. If you're monitoring `/var/log/syslog` and `/var/log/auth.log`, you'll see a count for each. It's a quick way to see which log files are the most active.

### Count events grouped by host
index=main | stats count by host

Similar to the above, but grouped by machine instead of file. If you ever add more forwarders, this query tells you exactly how many log events each machine has sent. Useful for comparing activity levels across different machines.

### Count events grouped by log level or event type

index=main | stats count by sourcetype

This groups events by their sourcetype — Splunk's way of categorizing what kind of log data it's receiving. You might see sourcetypes like `syslog`, `linux_secure`, etc. depending on what you've configured.

## Time-Based Searches

index=main earliest=-1h latest=now

Limits results to only events that arrived within the past hour. You can adjust this — `earliest=-6h` for the last six hours, `earliest=-24h` for the last day, and so on. This is one of the most commonly used filters once you're dealing with high log volumes.

### Show logs from a specific date range

index=main earliest="03/01/2025:00:00:00" latest="03/23/2025:23:59:59"

Use this when you need to investigate something that happened on a specific date. The format is `MM/DD/YYYY:HH:MM:SS`. Adjust the dates to whatever range you need.

## Filtering and Table Views

index=main ("error" OR "warning") | table _time, host, source, _raw

This searches for events containing either "error" or "warning", then displays the results as a clean table with four columns — the timestamp, the machine the log came from, the source file, and the raw log text. Much easier to read than the default view when you're scanning for problems.

### Find failed login attempts
index=main "Failed password" | table _time, host, _raw

This looks specifically for SSH failed login attempts in the auth log. If your forwarder machine is monitoring `/var/log/auth.log`, you'll see every failed SSH login attempt here. A useful query for spotting brute force activity.

 Find successful logins
index=main "Accepted password" | table _time, host, _raw

The opposite of the above — shows successful logins. Comparing these two queries gives you a picture of login activity on the monitored machine.

## Notes on Running These Queries

Time range matters a lot. If a query returns nothing, the first thing to check is whether the time range picker (top-right of the search bar) covers the period when your logs were actually collected.
The "| table command" at the end of a query formats results as a clean table with only the columns you specify. Without it, Splunk shows the full raw event view.
Saving a query as a report after running any search, click Save As → Report in the top-right corner. You can then schedule it to run automatically or add it to a dashboard.
The "stats command" is one of the most powerful parts of SPL. Once you're comfortable with the basics here, it's worth reading more about what `stats` can do — it can calculate averages, minimums, maximums, and much more across your log data.
