# ELK-IOC-Tools

IOC (Indicator of Compromise) list processing tool for ELK (Elasticsearch) threat hunting.

## Overview

`ELK_IOCListHandle.ps1` reads an IOC CSV file, extracts Domains / IPs / URLs, filters out whitelisted domains, and generates ready-to-use Elasticsearch queries for searching Proxy, Firewall, and DNS logs.

## Usage

```powershell
.\ELK_IOCListHandle.ps1 -csvPath <path_to_ioc_csv>
```

## Input

### CSV Format

The CSV file must contain the following columns:

| Column | Description | Example |
|--------|-------------|---------|
| `IOC Type` | One of: `Domain`, `IP`, `URL` | `Domain` |
| `IOC Value` | The IOC value | `evil.example.com` |

### DomainWhitelist.txt

Optional whitelist file in the same directory as the script. One domain per line. Lines starting with `#` are ignored. Whitelisted domains are excluded from the generated queries.

```
# Example whitelist
raw.githubusercontent.com
github.com
```

## Output

The script generates three Elasticsearch search queries:

### 1. Proxy Log Search

```
Index: filebeat-*
Query: observer.product: "Web Security" and destination.domain: (*domain1* or *domain2*)
```

### 2. Firewall Log Search

```
Index: filebeat-*
Query: (source.ip: (ip1 or ip2) or destination.ip: (ip1 or ip2))
       or ((tags: paloalto and Firewall.type: THREAT and Threat.name: URL-filtering)
       or (tagscls: fortinet and fortinet.firewall.type: utm
       and fortinet.firewall.subtype: (app-ctrl or webfilter))) and url.domain: (*domain*)
```

Supports both **Palo Alto** and **Fortinet** firewall log formats.

### 3. DNS Log Search

```
Index: filebeat-* and packetbeat-*
Query: dns.question.name: (*domain1* or *domain2*)
```

## How It Works

```
IOC CSV
  |
  +-- Parse Domain / IP / URL types
  |
  +-- Extract domains from URLs
  |
  +-- Filter out whitelisted domains (DomainWhitelist.txt)
  |
  +-- Generate Elasticsearch queries
       |
       +-- Proxy query (destination.domain)
       +-- Firewall query (source/destination IP + URL filtering)
       +-- DNS query (dns.question.name)
```

## Requirements

- Windows PowerShell 5.1+ or PowerShell 7+
- An IOC CSV file (exported from threat intelligence platform)
- ELK stack with `filebeat-*` / `packetbeat-*` indices

## File Structure

| File | Description |
|------|-------------|
| `ELK_IOCListHandle.ps1` | Main script |
| `DomainWhitelist.txt` | Domain whitelist (optional) |
