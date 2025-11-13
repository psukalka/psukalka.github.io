+++
date = '2025-11-13T10:41:47+05:30'
draft = false
title = 'Google Workspace Setup'
+++
Setting up Google Workspace is different from setting up a personal Gmail account. Your domain is likely registered with a provider like GoDaddy or Namecheap, and email traffic must be routed to Google's servers.

**Example scenario:**
- Domain `abc.ai` registered with GoDaddy
- Cloudflare as DNS provider (optional but recommended for easier setup)
- Goal: Create Google Workspace emails like `pavan@abc.ai`

## Setup Steps

### 1. Configure DNS Provider (Optional - Cloudflare)
If using Cloudflare, update nameservers on GoDaddy domain settings to point to Cloudflare nameservers (typically `elias.ns.cloudflare.com` and `megan.ns.cloudflare.com`).

### 2. Sign Up and Pay
Choose your Google Workspace business plan and complete payment.

### 3. Verify Domain Ownership
Google verifies you own the domain. Add a `TXT` record to your DNS with the verification code provided (format: `google-site-verification=EeKPAHUX...`).

- **With Cloudflare:** Automatic via Google sign-in
- **Without Cloudflare:** Manually add TXT record (may take time to propagate)

### 4. Route Email Traffic to Google
Email uses SMTP protocol (not HTTP) and must be served by mail servers. Add `MX` records to your DNS pointing to Google's mail servers. Google provides values like: `aspmx.l.google.com`, `alt1.aspmx.l.google.com`, etc.

- **With Cloudflare:** Automatic
- **Without Cloudflare:** Manually add all 5 MX records with correct priorities

### 5. Additional Security (Recommended)
Add these DNS records:
- **SPF:** `TXT` record: `v=spf1 include:_spf.google.com ~all`
- **DKIM:** Generate in admin console, add provided TXT record
- **DMARC:** `TXT` record at `_dmarc`: `v=DMARC1; p=none; rua=mailto:admin@abc.ai`

### 6. Create Users
Go to `admin.google.com` → Users → Add users with desired email addresses.

---

**Setup time:** ~30 minutes active work + 2-4 hours DNS propagation