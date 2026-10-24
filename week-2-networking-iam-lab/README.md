# AWS Student Builders Group - Week 2 Lab
## Networking & IAM Fundamentals

**Track:** Cloud & DevOps  
**Name:** Lakin Awel 
**Operating System:** Windows 10  
**Date Completed:** 24 October 2026

## Task 1 - Identify Your Network Information

**Command used:** `ipconfig`

- **Operating system:** Windows 10
- **Active network adapter:** Wi-Fi
- **Local IPv4 address:** Private IPv4 address (exact value omitted for privacy)
- **Default gateway/router:** Present (exact value omitted for privacy)
- **Address type:** Private

**Explanation:**

The `ipconfig` command displays the computer's network configuration. I used the Wi-Fi adapter because it represents my wireless network connection. My IPv4 address is private because it belongs to the private range `10.0.0.0` to `10.255.255.255`. A default gateway allows my computer to communicate with destinations outside its local network.

## Task 2 - Practice Subnetting

**Starting network:** `10.0.0.0/24`

I divided the network into four equal-sized subnets.

### Calculations

- An IPv4 address contains 32 bits.
- The `/24` prefix leaves 8 host bits.
- Four subnets require 2 borrowed bits because \(2^2 = 4\).
- The new prefix is `/26` because \(24 + 2 = 26\).
- Remaining host bits: \(32 - 26 = 6\).
- Addresses per subnet: \(2^6 = 64\).
- Conventional usable addresses per subnet: \(64 - 2 = 62\), excluding the network and broadcast addresses.
- AWS usable IPv4 addresses per subnet: \(64 - 5 = 59\), because AWS reserves five IPv4 addresses in each subnet.

### Subnet Results

| Subnet | CIDR notation | Full address range | Conventional usable host range | Broadcast address | AWS usable IPv4 addresses |
|---|---|---|---|---|---:|
| 1 | `10.0.0.0/26` | `10.0.0.0–10.0.0.63` | `10.0.0.1–10.0.0.62` | `10.0.0.63` | 59 |
| 2 | `10.0.0.64/26` | `10.0.0.64–10.0.0.127` | `10.0.0.65–10.0.0.126` | `10.0.0.127` | 59 |
| 3 | `10.0.0.128/26` | `10.0.0.128–10.0.0.191` | `10.0.0.129–10.0.0.190` | `10.0.0.191` | 59 |
| 4 | `10.0.0.192/26` | `10.0.0.192–10.0.0.255` | `10.0.0.193–10.0.0.254` | `10.0.0.255` | 59 |

**Conclusion:** The original `/24` network is divided into four equal `/26` subnets, each containing 64 total addresses.

## Task 3 - Test DNS Resolution

**Command used:**

```cmd
nslookup example.com
```

- **Domain tested:** `example.com`
- **DNS resolution:** Successful
- **IPv4 addresses returned:** `104.20.23.154`, `172.66.147.243`
- **IPv6 addresses returned:** `2606:4700:10::6814:179a`, `2606:4700:10::ac42:93f3`

**Explanation:**

DNS (Domain Name System) translates domain names into IP addresses so computers can locate services on a network. The lookup succeeded because IP addresses were returned for `example.com`. The result was marked non-authoritative, meaning the responding DNS server was not the authoritative source for the domain's DNS records.

## Task 4 - Inspect an HTTPS Website

**Website tested:** `https://example.com`

**Command used:**

```cmd
curl.exe -I https://example.com
```

- **HTTPS request:** Successful
- **HTTP status code:** `200 OK`
- **Server reported:** Cloudflare

**Explanation:**

The server returned `HTTP/1.1 200 OK`, indicating that the HTTP request was successful. HTTPS adds TLS encryption to HTTP, helping protect data in transit and verify the server's identity through its certificate. It also provides protection against undetected modification of data in transit.

DNS resolution alone does not guarantee that a website will load. The network connection must work, the server must be reachable, and the TLS handshake and HTTP request must succeed.

## Task 5 - Interpret HTTP Status Codes

| Status code | Meaning | Example situation |
|---|---|---|
| `200` | OK - request succeeded | A webpage is returned successfully. |
| `301` | Moved Permanently | A page has permanently moved to another URL. |
| `403` | Forbidden | The server refuses access to a protected resource. |
| `404` | Not Found | The requested page or resource does not exist at that URL. |
| `500` | Internal Server Error | The server encounters an unexpected problem processing the request. |

**Difference between DNS failure, connection timeout, TLS certificate error, and HTTP error response:**

- **DNS failure:** The domain name cannot be resolved to an IP address.
- **Connection timeout:** The connection does not complete within the allowed time, possibly because of network problems, filtering, or an unavailable service.
- **TLS certificate error:** The secure connection cannot be trusted, for example because the certificate is invalid, expired, or does not match the hostname.
- **HTTP error response:** The server responds with an HTTP status such as `403`, `404`, or `500`. This means an HTTP response was received, even though the requested operation may have failed.

## Task 6 - Write a Least-Privilege IAM Policy

The file `s3-read-policy.json` contains a policy allowing only the `s3:GetObject` action on objects inside `example-bucket`.

**Answers:**

1. **Which field specifies the permitted action?** The `Action` field.
2. **Which field specifies the resources?** The `Resource` field.
3. **Does `s3:GetObject` allow listing all objects in a bucket?** No. Listing objects requires the separate `s3:ListBucket` permission.
4. **Why is a narrow permission safer than granting `s3:*`?** It grants only the access needed for the task and reduces the risk of accidental or unauthorized actions.
5. **Why should AWS access keys never be committed to GitHub?** Anyone who obtains exposed credentials may misuse the account or create unexpected charges. Credentials should be kept private and managed securely.

The policy is for practice only and has not been attached to a live AWS identity.

## Task 7 - Troubleshooting Practice

### Scenario 1: A domain name does not resolve to an IP address

**Layer:** DNS

**Reason:** The domain cannot be translated into an IP address. I would first check the domain spelling, DNS server settings, and DNS resolution.

### Scenario 2: A domain resolves, but the browser connection times out

**Layer:** Network connectivity

**Reason:** DNS has returned an IP address, but the connection is not completing. I would investigate network connectivity, routing, firewalls, security rules, and whether the service is available.

### Scenario 3: The browser displays a TLS certificate warning

**Layer:** TLS

**Reason:** The browser cannot establish that the website's secure connection is trustworthy. I would check the certificate's validity, expiry, hostname, and certificate chain.

### Scenario 4: A website responds with HTTP `404`

**Layer:** HTTP/application

**Reason:** The server responded, but the requested resource could not be found at that URL. I would check the URL and whether the resource exists.

### Scenario 5: An EC2 application can reach Amazon S3 but receives `AccessDenied` when reading an object

**Layer:** IAM/S3 authorization

**Reason:** Network connectivity is working, but access is denied. I would check the EC2 IAM role, the `s3:GetObject` permission, the object ARN, the bucket policy, and any explicit denies or encryption-key permissions that may apply.

## Conclusion

This lab helped me practice IP addressing, subnetting, DNS resolution, HTTPS inspection, HTTP status codes, IAM policy structure, and troubleshooting across different network and authorization layers.

## Evidence

Relevant command-output screenshots are stored in the `evidence/` folder where available.
