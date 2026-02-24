# Lab 4 — Operating Systems & Networking

> **Environment:** macOS (commands adapted for launchd/BSD instead of systemd)

---

## Task 1 — Operating System Analysis

### 1.1 Boot Performance Analysis

#### Last Reboot
```
reboot time                                Tue Feb 10 16:25
shutdown time                              Tue Feb 10 16:25
reboot time                                Wed Dec  3 15:08
shutdown time                              Wed Dec  3 15:08
reboot time                                Wed Oct 29 21:03
```

#### System Uptime
```
9:47  up 13 days, 17:22, 2 users, load averages: 1.78 2.07 2.31
```

#### Logged Users
```
USER       TTY      FROM    LOGIN@  IDLE WHAT
shiyanovn  console  -      10Feb26 13days -
shiyanovn  s004     -      15Feb26 4days -zsh
```

**Observations:**
- System uptime: 13 days 17 hours
- Load average: 1.78 (1 min), 2.07 (5 min), 2.31 (15 min) — moderate load
- Logged users: 2 sessions (console + terminal)
- Last reboot: February 10, 2026

---

### 1.2 Process Forensics

#### Top Memory-Consuming Processes
```
  PID  PPID COMM             %MEM  %CPU
 1044  1028 /Applications/Cu  4.1   0.1
 3372     1 /Applications/Ya  2.2   0.2
10859 44260 /Applications/Vi  2.0   1.1
10669  3372 /Applications/Ya  1.6   0.0
  572     1 /System/Library/  1.5   1.2
```

#### Top CPU-Consuming Processes
```
  PID  PPID COMM             %MEM  %CPU
  367     1 /System/Library/  0.8  17.9
  542     1 /Library/Applica  0.2  13.6
96208     1 /System/Library/  1.2  10.5
10847 44260 /Applications/Vi  1.3   6.8
36112 36109 /Applications/Я   1.1   5.1
```

**Observations:**
- Top by memory: application from /Applications/ (4.1% RAM)
- Top by CPU: system process /System/Library/* (17.9% CPU)
- VS Code actively uses both memory (2.0%) and CPU (6.8%)

**Answer:** The top memory-consuming process is an application from `/Applications/` using 4.1% of RAM.

---

### 1.3 Service Dependencies

#### Running Services (launchd)
```
PID     Status  Label
11204   0       com.apple.cloudphotod
693     0       com.apple.Finder
739     0       com.apple.homed
2119    0       com.apple.dataaccess.dataaccessd
760     0       com.apple.mediaremoteagent
732     0       com.apple.FontWorker
714     0       com.apple.bird
11615   0       com.apple.knowledgeconstructiond
897     0       com.apple.GameController.gamecontrolleragentd
661     0       com.apple.nsurlsessiond
721     0       com.apple.iconservices.iconservicesagent
1846    0       com.apple.diagnosticextensionsd
8452    0       com.apple.intelligenceplatformd
704     0       com.apple.wallpaper.agent
758     0       com.apple.ManagedSettingsAgent
9101    0       com.apple.photolibraryd
764     0       com.apple.bluetoothuserd
809     0       com.apple.TextInputMenuAgent
660     0       com.apple.CommCenter
692     0       com.apple.cloudd
```

**Observations:**
- macOS uses `launchd` instead of `systemd`
- Multiple Apple system services running (cloud, photos, bluetooth, etc.)
- Third-party services present: Yandex (ru.yandex-team.*), Tunnelblick VPN
- Status 0 indicates successful execution

---

### 1.4 User Sessions

#### Current Sessions
```
shiyanovn  console   Feb 10 16:26 
shiyanovn  ttys004   Feb 15 17:07 
```

#### Recent Logins
```
shiyanovn  ttys004   Sun Feb 15 17:07   still logged in
shiyanovn  console   Tue Feb 10 16:26   still logged in
reboot time          Tue Feb 10 16:25
shutdown time        Tue Feb 10 16:25
shiyanovn  ttys001   Wed Feb  4 18:38 - 18:38  (00:00)
```

**Observations:**
- Active sessions: 2 (console + terminal)
- Console session active since February 10
- Terminal session (ttys004) active since February 15
- System running continuously since reboot on February 10

---

### 1.5 Memory Analysis

#### Memory Statistics (vm_stat)
```
Mach Virtual Memory Statistics: (page size of 16384 bytes)
Pages free:                               80496.
Pages active:                            843906.
Pages inactive:                          856625.
Pages speculative:                         9604.
Pages wired down:                        266674.
Pages purgeable:                          63763.
Pages stored in compressor:              523994.
Pages occupied by compressor:            246004.
Swapins:                                 444748.
Swapouts:                                560838.
```

#### Total RAM
```
hw.memsize: 38654705664
RAM: 36 GB
```

#### Memory Summary
```
PhysMem: 34G used (3824M wired, 3844M compressor), 1398M unused.
```

**Observations:**
- Total RAM: 36 GB
- Used: 34 GB (~94%)
- Free: ~1.4 GB
- Wired (system memory): 3.8 GB
- Compressor active: 3.8 GB compressed memory
- Swap actively used (444K swapins, 560K swapouts)

---

## Task 2 — Networking Analysis

### 2.1 Network Path Tracing

#### Traceroute to github.com
```
traceroute to github.com (140.82.121.4), 64 hops max, 40 byte packets
 1  yacht-tun20854.yndx.net (10.215.205.XXX)  25.495 ms  23.924 ms  21.768 ms
 2  * * *
 3  * * *
 4  m9-ofb1-eth-trunk0-835.yndx.net (5.255.237.48)  23.686 ms  23.320 ms  23.975 ms
 5  * 81.177.108.52 (81.177.108.52)  22.699 ms  23.165 ms
 6  185.140.148.245 (185.140.148.245)  53.140 ms
    188.128.126.95 (188.128.126.95)  42.296 ms
    95.167.95.15 (95.167.95.15)  41.261 ms
 7  * * *
 8  * * *
 9  * * *
10  * * *
```

#### DNS Resolution
```
;; ANSWER SECTION:
github.com.             35      IN      A       140.82.121.3

;; Query time: 38 msec
;; SERVER: 213.180.205.1#53(213.180.205.1)
```

**Observations:**
- Traffic routed through corporate Yandex VPN (yacht-tun20854.yndx.net)
- Visible hops: 6 (others hidden by firewalls)
- GitHub IP: 140.82.121.3
- DNS server: 213.180.205.1 (Yandex DNS)
- DNS response time: 38 ms
- Many hops respond with `* * *` — ICMP blocked on routers

---

### 2.2 Packet Capture

#### DNS Traffic Capture (port 53)
```
tcpdump: data link type PKTAP
listening on any, link-type PKTAP (Apple DLT_PKTAP), snapshot length 524288 bytes
09:54:46.602213 IP 10.215.205.XXX.63772 > 213.180.205.1.53: 54409+ Type65? clarity.yandex.net. (36)
09:54:46.602759 IP 10.215.205.XXX.61321 > 213.180.205.1.53: 50667+ AAAA? clarity.yandex.net. (36)
09:54:46.603490 IP 10.215.205.XXX.63743 > 213.180.205.1.53: 3685+ A? clarity.yandex.net. (36)
09:54:46.628912 IP 213.180.205.1.53 > 10.215.205.XXX.63743: 3685 1/0/0 A 93.158.134.148 (52)
09:54:46.630574 IP 213.180.205.1.53 > 10.215.205.XXX.63772: 54409 0/1/0 (94)
5 packets captured
```

**Observations:**
- Captured 5 DNS packets
- DNS server: 213.180.205.1 (corporate Yandex)
- Query types: A (IPv4), AAAA (IPv6), Type65 (HTTPS record)
- Queried domain: clarity.yandex.net
- Response: A record with IP 93.158.134.148

**Example DNS Query Analysis:**
- Source IP: 10.215.205.XXX (sanitized, VPN client)
- Destination: 213.180.205.1:53 (Yandex DNS server)
- Query type: A record for clarity.yandex.net
- Response: 93.158.134.148 (resolved IP)

---

### 2.3 Reverse DNS Lookups

#### PTR Lookup for 8.8.4.4 (Google DNS)
```
;; QUESTION SECTION:
;4.4.8.8.in-addr.arpa.          IN      PTR

;; ANSWER SECTION:
4.4.8.8.in-addr.arpa.   75629   IN      PTR     dns.google.

;; Query time: 340 msec
;; SERVER: 213.180.205.1#53(213.180.205.1)
```

#### PTR Lookup for 1.1.2.2 (Cloudflare)
```
;; QUESTION SECTION:
;2.2.1.1.in-addr.arpa.          IN      PTR

;; AUTHORITY SECTION:
1.in-addr.arpa.         1071    IN      SOA     ns.apnic.net. read-txt-record-of-zone-first-dns-admin.apnic.net. 23597 7200 1800 604800 3600

;; Query time: 759 msec
;; SERVER: 213.180.205.1#53(213.180.205.1)
;; status: NXDOMAIN
```

**Observations:**
- **8.8.4.4** → successfully resolves to `dns.google.` (Google Public DNS)
- **1.1.2.2** → NXDOMAIN (PTR record does not exist)
  - This IP is from Cloudflare range but has no reverse record
  - Unlike 1.1.1.1 which resolves to `one.one.one.one`

**Comparison:**

| IP | PTR Record | Status |
|---|---|---|
| 8.8.4.4 | dns.google. | Found |
| 1.1.2.2 | — | NXDOMAIN |

Google configured PTR records for their public DNS servers, while Cloudflare did not configure PTR for all IPs in their 1.1.0.0/24 range.
