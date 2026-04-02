# SPL Query Reference Guide
## Complete Guide to Writing Splunk Processing Language (SPL) Queries

---

## Table of Contents
1. [Query Structure](#query-structure)
2. [Basic Search](#basic-search)
3. [Filtering](#filtering)
4. [Fields & Tables](#fields--tables)
5. [Statistics & Aggregation](#statistics--aggregation)
6. [Time Bucketing](#time-bucketing)
7. [Sorting & Limiting](#sorting--limiting)
8. [Renaming Fields](#renaming-fields)
9. [Conditional Logic](#conditional-logic)
10. [Regular Expressions](#regular-expressions)
11. [Common Operators](#common-operators)
12. [Complete Examples](#complete-examples)

---

## Query Structure

### **Basic Anatomy**

```
[Search] | [Command] | [Command] | [Visualization]
```

**Example:**
```splunk
index=main source="/var/log/auth.log" status=Failed | stats count by src_ip | sort - count
```

**Breakdown:**
- `index=main source="/var/log/auth.log" status=Failed` = Search (find events)
- `| stats count by src_ip` = First command (aggregate)
- `| sort - count` = Second command (sort)

---

## Basic Search

### **Search by Index**
```splunk
index=main
```
Find all events in the "main" index.

### **Search by Source**
```splunk
source="/var/log/auth.log"
```
Find events from a specific file/source.

### **Search by Source Type**
```splunk
sourcetype=syslog
```
Find events of a specific type.

### **Search by Host**
```splunk
host=web-server-01
```
Find events from a specific host.

### **Combine Multiple Searches**
```splunk
index=main source="/var/log/auth.log" host=linux-server
```
All conditions must be true (implicit AND).

### **Search by Field Value**
```splunk
index=main status=Failed
```
Find events where the `status` field equals "Failed".

---

## Filtering

### **Equals (=)**
```splunk
status=Failed
```
Exact match.

### **Not Equals (!=)**
```splunk
status!=Accepted
```
Exclude specific value.

### **Wildcard (*)**
```splunk
user=root*
```
Matches "root", "root123", "rootadmin", etc.

### **OR Conditions**
```splunk
(status=Failed OR status="Invalid user")
```
At least one condition must be true.

### **AND Conditions**
```splunk
status=Failed AND user=root
```
All conditions must be true (same as space-separated).

### **Complex Logic**
```splunk
(status=Failed OR status="Invalid user") AND (user=root OR user=admin)
```
Status is Failed/Invalid AND user is root/admin.

### **Greater Than / Less Than**
```splunk
port > 50000
count < 100
```

### **IN Operator**
```splunk
user IN (root, admin, testuser)
```
Match any value in list.

---

## Fields & Tables

### **Select Specific Fields**
```splunk
index=main | table status, _time, user, src_ip
```
Display only these columns.

### **Keep Only Certain Fields**
```splunk
index=main | fields status, user, src_ip
```
Same as table but returns in search results format (fields are better for piping).

### **Remove Null Values**
```splunk
index=main | fields status, user, src_ip | fillnull value="Unknown"
```
Replace empty fields with "Unknown".

### **Extract New Field**
```splunk
index=main | eval new_field = old_field
```
Create a new field based on existing field.

### **Dedup (Remove Duplicates)**
```splunk
index=main | dedup user
```
Show only one entry per user (first occurrence).

### **Dedup by Multiple Fields**
```splunk
index=main | dedup src_ip, user
```
Remove duplicates based on IP and user combination.

---

## Statistics & Aggregation

### **Count Events**
```splunk
index=main | stats count
```
Total number of events: `1000`

### **Count by Field**
```splunk
index=main | stats count by status
```
Count grouped by status field.

**Result:**
```
status    | count
Failed    | 450
Accepted  | 550
```

### **Count Multiple Fields**
```splunk
index=main | stats count by src_ip, user, status
```
Count grouped by IP, user, and status.

### **Distinct Count**
```splunk
index=main | stats dc(user)
```
Count unique/distinct users.

**Result:** `23` (23 unique users)

### **Distinct Count by Field**
```splunk
index=main | stats dc(user) by src_ip
```
Count unique users per IP.

**Result:**
```
src_ip         | dc(user)
192.168.1.100  | 10
192.168.1.101  | 5
192.168.1.102  | 3
```

### **Sum**
```splunk
index=main | stats sum(bytes) by user
```
Sum of bytes transferred per user.

### **Average**
```splunk
index=main | stats avg(response_time) by status
```
Average response time per status.

### **Min / Max**
```splunk
index=main | stats min(_time), max(_time) by user
```
Earliest and latest time per user.

### **Multiple Stats**
```splunk
index=main | stats count, dc(user), sum(bytes), avg(response_time) by src_ip
```
Multiple aggregations in one command.

**Result:**
```
src_ip         | count | dc(user) | sum(bytes) | avg(response_time)
192.168.1.100  | 100   | 5        | 50000      | 250.5
```

---

## Time Bucketing

### **Bucket by Hour**
```splunk
index=main | bucket _time span=1h | stats count by _time
```
Group events into 1-hour buckets and count.

### **Bucket by Minute**
```splunk
index=main | bucket _time span=5m | stats count by _time
```
Group into 5-minute buckets.

### **Bucket by Day**
```splunk
index=main | bucket _time span=1d | stats count by _time
```
Group into 1-day buckets.

### **Common Spans**
```
span=5m     (5 minutes)
span=10m    (10 minutes)
span=1h     (1 hour)
span=1d     (1 day)
span=1w     (1 week)
span=1mon   (1 month)
```

### **Time Range Filtering**
```splunk
index=main earliest=-24h latest=now
```
Last 24 hours. (Now is default, can omit)

```splunk
index=main earliest=-7d latest=now
```
Last 7 days.

```splunk
index=main earliest=2026-03-31 latest=2026-04-03
```
Specific date range.

---

## Sorting & Limiting

### **Sort Ascending**
```splunk
index=main | stats count by src_ip | sort src_ip
```
Sort A→Z or low→high.

### **Sort Descending**
```splunk
index=main | stats count by src_ip | sort - count
```
`-` means descending. Sort high→low.

### **Limit Results**
```splunk
index=main | head 10
```
Show first 10 results.

### **Get Last N Results**
```splunk
index=main | tail 10
```
Show last 10 results.

### **Sort and Limit Combined**
```splunk
index=main | stats count by src_ip | sort - count | head 5
```
Top 5 IPs by count.

---

## Renaming Fields

### **Rename Column Header**
```splunk
index=main | table status, _time, user, src_ip
| rename status as "Login Status", _time as "Timestamp", user as "Username", src_ip as "Source IP"
```

**Result:**
```
Login Status | Timestamp               | Username  | Source IP
Failed       | 2026-03-31 17:03:58     | root      | 192.168.1.16
```

### **Rename Single Field**
```splunk
index=main | rename src_ip as "Attacker IP"
```

### **Rename in Stats Output**
```splunk
index=main | stats count as "Failed Attempts", dc(user) as "Unique Users" by src_ip
```

**Result:**
```
src_ip         | Failed Attempts | Unique Users
192.168.1.100  | 50              | 5
```

---

## Conditional Logic

### **If-Then-Else (eval with if)**
```splunk
index=main | eval severity=if(count>10, "High", "Low")
```
If count > 10, severity = "High", else "Low".

### **Case Statement (Multiple Conditions)**
```splunk
index=main 
| eval threat_level=case(count>=20, "Critical", count>=10, "High", count>=5, "Medium", 1=1, "Low")
```

**Result:**
```
count | threat_level
25    | Critical
15    | High
8     | Medium
3     | Low
```

### **Combine with Field Display**
```splunk
index=main status=Failed
| eval severity_icon=case(count>=10, "🔴 CRITICAL", count>=5, "🟠 HIGH", 1=1, "🟡 MEDIUM")
| stats count by src_ip, severity_icon
```

---

## Regular Expressions

### **Extract Field with Regex**
```splunk
index=main | rex field=_raw "user=(?<username>\w+)"
```
Extract username from raw log.

### **Match Pattern**
```splunk
index=main | where match(_raw, "Failed password")
```
Find logs matching pattern.

### **Search with Regex**
```splunk
index=main _raw="/var/log/auth.log" "Failed|Invalid"
```
Search for "Failed" OR "Invalid" in raw data.

---

## Common Operators

### **Where Clause (Post-Search Filtering)**
```splunk
index=main | stats count by src_ip | where count > 5
```
Filter results AFTER aggregation.

**Why use where?**
- Search filters BEFORE aggregation (faster)
- Where filters AFTER aggregation (on results)

**Example difference:**
```splunk
# WRONG: Filters before stats
index=main count > 5 | stats count by src_ip

# RIGHT: Filters after stats
index=main | stats count by src_ip | where count > 5
```

### **Append (Combine Two Searches)**
```splunk
index=main status=Failed 
| append [search index=web_logs status=error]
```
Combine results from two searches.

### **Join (Match Data from Two Searches)**
```splunk
index=main status=Failed 
| join src_ip [search index=threat_intel | fields src_ip, threat_level]
```
Match IPs between two indexes.

---

## Complete Examples

### **Example 1: SSH Brute Force Detection**
```splunk
index=main source="/var/log/auth.log" status=Failed
| bucket _time span=10m
| stats count, dc(user) as unique_users by src_ip, _time
| where count >= 5 AND unique_users >= 2
| rename src_ip as "Attacker IP", count as "Failed Attempts", unique_users as "Target Users", _time as "Time"
| sort - "Failed Attempts"
```

**What it does:**
1. Find failed SSH attempts
2. Group by 10-minute time buckets
3. Count failures and unique users per IP per time bucket
4. Show only IPs with 5+ failures and 2+ target users
5. Rename columns for readability
6. Sort by highest failure count first

---

### **Example 2: Top Attack Sources with Geolocation**
```splunk
index=main status=Failed
| stats count by src_ip
| where count >= 10
| lookup geoip_lookup ip as src_ip OUTPUT country, latitude, longitude
| rename src_ip as "Source IP", country as "Country", count as "Attacks", latitude as "Lat", longitude as "Lon"
| sort - Attacks
| head 20
```

**What it does:**
1. Find all failed authentication attempts
2. Count by source IP
3. Keep only IPs with 10+ attempts
4. Look up geolocation data
5. Rename for display
6. Sort by most attacks
7. Show top 20

---

### **Example 3: User Account Analysis**
```splunk
index=main 
| stats count(eval(status="Failed")) as failed_attempts, 
        count(eval(status="Accepted")) as successful_logins,
        dc(src_ip) as unique_source_ips 
        by user
| eval login_success_rate = round(successful_logins/(failed_attempts+successful_logins)*100, 2)
| where failed_attempts >= 5
| rename user as "Username", failed_attempts as "Failed", successful_logins as "Success", unique_source_ips as "Unique IPs", login_success_rate as "Success Rate %"
| sort - Failed
```

**What it does:**
1. Count failed and successful logins per user
2. Count unique IPs per user
3. Calculate success rate percentage
4. Show only users with 5+ failures
5. Rename and sort

**Result:**
```
Username    | Failed | Success | Unique IPs | Success Rate %
root        | 50     | 0       | 15         | 0.00
admin       | 30     | 5       | 10         | 14.29
testuser    | 8      | 50      | 2          | 86.21
```

---

### **Example 4: Timeline of Attacks**
```splunk
index=main source="/var/log/auth.log" status=Failed
| bucket _time span=1h
| stats count by _time, src_ip
| where count >= 5
| timechart count by src_ip limit=5
```

**What it does:**
1. Get failed SSH attempts
2. Group by hour
3. Count per IP per hour
4. Keep only hours with 5+ attempts
5. Create time-series chart showing top 5 IPs

---

### **Example 5: Successful Logins After Brute Force**
```splunk
index=main (status=Failed OR status=Accepted)
| stats count(eval(status="Failed")) as failed, 
        count(eval(status="Accepted")) as success 
        by src_ip, user
| where failed >= 5 AND success >= 1
| eval compromised_risk="High"
| rename src_ip as "Source IP", user as "User", failed as "Failed Attempts", success as "Success Count", compromised_risk as "Risk Level"
```

**What it does:**
1. Get both failed and successful logins
2. Count each by IP and user
3. Show IPs that had 5+ failures AND at least 1 success (indicates compromise!)
4. Tag as high risk
5. Rename for display

**Result:** Attackers who succeeded after brute force attempt!

---

## Quick Reference Table

| Task | Command |
|------|---------|
| Search events | `index=main` |
| Filter by field | `status=Failed` |
| Show columns | `\| table field1, field2` |
| Count events | `\| stats count` |
| Count by group | `\| stats count by field` |
| Unique count | `\| stats dc(field)` |
| Group by time | `\| bucket _time span=10m` |
| Filter results | `\| where count > 5` |
| Sort ascending | `\| sort field` |
| Sort descending | `\| sort - field` |
| Top N | `\| head 10` |
| Rename column | `\| rename old as "New"` |
| If/then logic | `\| eval new=if(condition, value1, value2)` |
| Multiple conditions | `\| eval x=case(cond1, val1, cond2, val2, 1=1, val3)` |
| Remove duplicates | `\| dedup field` |
| Extract field | `\| rex field=_raw "pattern"` |

---

## Pro Tips

### **Tip 1: Test in Stages**
```splunk
# First, see raw events
index=main status=Failed | head 20

# Then add aggregation
index=main status=Failed | stats count by src_ip

# Then add filtering
index=main status=Failed | stats count by src_ip | where count >= 5

# Then add sorting
index=main status=Failed | stats count by src_ip | where count >= 5 | sort - count
```

### **Tip 2: Use Comments (in Dashboards)**
```splunk
# Find failed SSH attempts
index=main source="/var/log/auth.log" status=Failed
# Group by 10-minute windows
| bucket _time span=10m
# Count failures per IP
| stats count by src_ip, _time
# Keep only high-count events
| where count >= 5
```

### **Tip 3: Performance - Filter Early**
```splunk
# SLOW: Processes all events then filters
index=main | where src_ip="192.168.1.100" | stats count

# FAST: Filters before processing
index=main src_ip="192.168.1.100" | stats count
```

### **Tip 4: Time Matters**
```splunk
# Always specify time range
index=main earliest=-24h latest=now status=Failed
```

### **Tip 5: Use Meaningful Field Names**
```splunk
# CONFUSING
| stats dc(user) as x, count as y by src_ip as z

# CLEAR
| stats dc(user) as unique_users, count as total_attempts by src_ip
| rename src_ip as "Source IP", unique_users as "Target Users", total_attempts as "Attempts"
```

---

## Debugging

### **Check What Fields Exist**
```splunk
index=main | fields *
```
Shows all available fields.

### **See Raw Event**
```splunk
index=main | head 1 | fields _raw
```
Shows original unprocessed log.

### **Test Field Extraction**
```splunk
index=main | table _raw, src_ip, user
```
If src_ip or user are empty, extraction failed.

### **Check Stats Output**
```splunk
index=main | stats count by src_ip, user | head 20
```
Verify aggregation is working correctly.

---

## Common Mistakes

❌ **WRONG:**
```splunk
index=main count > 5 | stats count by src_ip
```
(Tries to filter on `count` field before it exists)

✅ **RIGHT:**
```splunk
index=main | stats count by src_ip | where count > 5
```
(Creates `count` first, then filters)

---

❌ **WRONG:**
```splunk
index=main | table src_ip, dst_ip, user
```
(If `dst_ip` doesn't exist, might show nothing)

✅ **RIGHT:**
```splunk
index=main | fields src_ip, user, _raw
```
(Fields are more forgiving, will skip missing fields)

---

❌ **WRONG:**
```splunk
index=main status = Failed
```
(Space around `=`)

✅ **RIGHT:**
```splunk
index=main status=Failed
```
(No spaces around operators)

---
