# socks5 proxy: How It Actually Works, Where It Breaks, and How to Run Residential SOCKS5 IPs in Chrome, Python, and cURL

Most people searching for a socks5 proxy want one of three things: to log into a tool that only accepts SOCKS5 credentials, to scrape something without getting blocked, or to understand why the proxy they already bought keeps timing out. All three are answerable, but the answer to the third one usually has nothing to do with the proxy itself.

Start with what the protocol is actually for, then look at the places it silently fails, then get to picking and configuring one.

## What a SOCKS5 proxy does (and doesn't do)

SOCKS5 is defined in RFC 1928. A client says "connect me to this destination," the proxy opens the connection and relays bytes both ways. It sits below HTTP, which is the whole point: it doesn't parse requests, doesn't rewrite headers, doesn't care whether the payload is a web page, an SSH session, a torrent tracker handshake, or a game server packet. You can run it with `curl -x socks5://127.0.0.1:1080 http://example.com` and confirm that in about ten seconds.

Two things people get wrong:

- Logging in through a proxy hides your IP from the destination. It does not encrypt your traffic. Anyone sitting on the path between you and the proxy can still read what you send. If you need confidentiality on the wire, the proxy layer isn't where you get it.
- SOCKS5 supports UDP (UDP ASSOCIATE), unlike SOCKS4. That's what makes it usable for things like qBittorrent traffic or real-time app traffic. Support still varies by server and by client, so verify before you build around it.

SOCKS5 also adds username/password authentication, which SOCKS4 has no concept of. That matters more than it sounds, because a lot of software implements the protocol but not the auth handshake.

## SOCKS5 vs HTTP proxy vs VPN

|  | SOCKS5 | HTTP proxy | VPN |
| --- | --- | --- | --- |
| Layer | Transport (TCP/UDP) | Application (HTTP) | Network |
| Reads/modifies request content | No | Yes | No |
| Handles non-HTTP traffic | Yes | Usually no | Yes |
| Built-in encryption | No | No | Yes |
| Username/password auth | Yes | Yes | Certificates/keys |
| Typical use | Automation, multi-app routing, P2P | Browser/API traffic | Whole-device privacy |

The practical split: white-label HTTP proxies are fine when you're only moving HTTP requests through a script. When your workflow involves an antidetect browser, a desktop app that writes its own packets, or a mix of tools on one machine, SOCKS5 is the layer that every one of them can agree on.

## Where SOCKS5 quietly fails in real setups

This is the part most guides skip, and it's the reason people end up convinced their provider is broken.

Chrome has no SOCKS5 authentication support. From Chromium's own networking documentation: name resolution for SOCKSv5 is always done proxy-side in Chrome, and no authentication methods are supported for SOCKSv5. If your provider hands you `user:pass@host:port`, Chrome's built-in settings can't use it. You need FoxyProxy or a system-level router like Proxifier.

Chrome also only proxies TCP through SOCKS5, so anything relying on UDP is out in that browser.

Firefox is the better-behaved option: built-in SOCKS5 support, credential fields, and `network.proxy.socks_remote_dns` to control where hostnames get resolved.

DNS is resolved locally by default in most tooling. With `socks5://`, cURL resolves the target hostname on your machine; with `socks5h://`, the proxy resolves it. If the destination only makes sense from the proxy's network, or your local resolver is the thing handing back wrong regional results, `socks5h://` is the correct scheme and the plain one is not equivalent.

iOS can't do it. According to 9Proxy's own integration documentation, iOS only supports HTTP proxies over Wi-Fi — HTTPS and SOCKS5 are not supported, and proxies can't be used on cellular networks at all. If your workflow is phone-based, that's a real constraint you need to know before paying.

Playwright's proxy option doesn't accept SOCKS5 credentials. Teams writing browser automation in Python or Node typically route automation through an HTTP endpoint and reserve the SOCKS5 endpoint for the desktop tools that need it.

Apps lie about their proxy support. Outlook has no proxy configuration at all. npm doesn't speak SOCKS. Git takes a SOCKS5 URL in its config while many GUI tools around it don't. The fix is Proxifier (31-day trial, and it deliberately tunnels a named `.exe`), not more time spent debugging credentials that were correct the whole time.

## Why residential SOCKS5 instead of datacenter

Datacenter IPs come from cloud ranges. They're fast, cheap, and immediately recognizable as belonging to a hosting provider, which is why they get challenged on retail, social, and travel sites.

Residential traffic exits through real consumer connections. You're slower and more expensive, but the IP looks like a person. For scraping, multi-account management, SERP tracking, and price monitoring, that distinction is the difference between a 40% success rate and a 90% one.

## Picking a residential SOCKS5 provider

Ask these before you buy, roughly in this order:

1. Does it actually expose SOCKS5 endpoints, or only HTTP? Plenty of "SOCKS5" landing pages mean "our tool supports SOCKS5 clients."
2. How big is the pool, and how many countries? A 20M+ pool spread across 90+ countries is workable; a 500K pool is fine until you need city-level targeting.
3. Which targeting levels exist — country, state, city, ZIP, ISP?
4. Is authentication username/password, IP whitelist, or both?
5. How is it billed: per IP, per GB, or bundled? This changes the math more than the sticker price does.
6. Is there a desktop app requirement, and does it run on your OS?
7. Can you test before committing real budget?

That middle question about billing is where most people choose wrong, so it's worth being concrete about how the two models behave in practice.

## Where 9Proxy fits

9Proxy is a residential proxy provider with a pool of 20M+ IPs across 90+ locations, offering HTTP/HTTPS and SOCKS5, geo-targeting down to country, city, ZIP code, and ISP, and pay-as-you-go pricing rather than a subscription. The company advertises 99.95% uptime. Entry pricing runs from around $0.018 per IP and $0.68 per GB at the high-volume tiers.

It splits into two models, and they behave very differently:

**Residential by IPs** — you buy a fixed number of IPs and pay per IP rather than per gigabyte. Bandwidth through an active IP is unlimited. Each IP stays alive for somewhere between a few hours and about 24 hours of natural residential uptime, and unused IPs never expire. This model requires the 9Proxy desktop app, which handles local port forwarding plus optional proxy authentication. Best fit: session-stable work like account logins, cart sessions, and long scraping runs where per-request bandwidth is unpredictable.

**Residential by GB** — you buy traffic, and the system generates unlimited endpoints from it. GB packages carry a 180-day validity, except Enterprise tiers where the balance doesn't expire. This works straight from the dashboard with username/password or IP whitelist authentication, no desktop app. Best fit: high-rotation work where each request moves very little data — ad verification, geo-checking, lightweight crawling, API polling.

The provider did raise prices on IP-based and bundle packages effective June 1, 2026, its first adjustment since launch; GB-based pricing was left unchanged in that update. If you're comparing against an older review, the IP-tier numbers you find on smaller blogs are probably stale.

👉 [Check 9Proxy's current SOCKS5 residential plans](https://bit.ly/9-Proxy)

## Full package comparison

All currently published tiers, including the bonus-IP offers and the Enterprise bandwidth packages:

| Category | Package | Unit price | Total | Billing | Buy |
| --- | --- | --- | --- | --- | --- |
| IP-based | 100 IPs | $0.24/IP | $24 | One-off, balance-based | [Get 100 IPs](https://bit.ly/9-Proxy) |
| IP-based | 500 IPs | $0.144/IP | $72 | One-off, balance-based | [Get 500 IPs](https://bit.ly/9-Proxy) |
| IP-based | 1,000 + 500 bonus IPs | $0.084/IP | $126 | One-off, balance-based | [Get the 1,500 IP package](https://bit.ly/9-Proxy) |
| IP-based | 2,500 IPs | $0.084/IP | $210 | One-off, balance-based | [Get 2,500 IPs](https://bit.ly/9-Proxy) |
| IP-based | 5,000 IPs | $0.072/IP | $360 | One-off, balance-based | [Get 5,000 IPs](https://bit.ly/9-Proxy) |
| IP-based | 15,000 IPs | $0.048/IP | $720 | One-off, balance-based | [Get 15,000 IPs](https://bit.ly/9-Proxy) |
| IP-based | 25,000 IPs | $0.035/IP | $863 | One-off, balance-based | [Get 25,000 IPs](https://bit.ly/9-Proxy) |
| IP-based | 50,000 IPs | $0.029/IP | $1,438 | One-off, balance-based | [Get 50,000 IPs](https://bit.ly/9-Proxy) |
| Business IP | 100,000 IPs | $0.023/IP | $2,300 | One-off, balance-based | [Get the 100,000 IP tier](https://bit.ly/9-Proxy) |
| Business IP | 200,000 IPs | $0.021/IP | $4,140 | One-off, balance-based | [Get the 200,000 IP tier](https://bit.ly/9-Proxy) |
| Business IP | 500,000 IPs | $0.018/IP | $8,625 | One-off, balance-based | [Get the 500,000 IP tier](https://bit.ly/9-Proxy) |
| GB-based | 5 GB | $3.00/GB | $15 | One-off, 180-day validity | [Get 5 GB](https://bit.ly/9-Proxy) |
| GB-based | 50 + 5 bonus GB | $2.10/GB | $105 | One-off, 180-day validity | [Get 55 GB](https://bit.ly/9-Proxy) |
| GB-based | 100 GB | $1.50/GB | $150 | One-off, 180-day validity | [Get 100 GB](https://bit.ly/9-Proxy) |
| GB-based | 200 GB | $1.00/GB | $200 | One-off, 180-day validity | [Get 200 GB](https://bit.ly/9-Proxy) |
| GB-based | 1,000 GB | $0.80/GB | $800 | One-off, 180-day validity | [Get 1,000 GB](https://bit.ly/9-Proxy) |
| GB-based | 2,000 GB | $0.75/GB | $1,500 | One-off, 180-day validity | [Get 2,000 GB](https://bit.ly/9-Proxy) |
| Enterprise GB | 3,000 GB | $0.72/GB | $2,160 | One-off, no expiry | [Get the 3,000 GB tier](https://bit.ly/9-Proxy) |
| Enterprise GB | 6,000 GB | $0.70/GB | $4,200 | One-off, no expiry | [Get the 6,000 GB tier](https://bit.ly/9-Proxy) |
| Enterprise GB | 10,000 GB | $0.68/GB | $6,800 | One-off, no expiry | [Get the 10,000 GB tier](https://bit.ly/9-Proxy) |
| Bundle | Starter: 100 IPs + 5 GB | — | $30 | One-off, balance-based | [Get the Starter bundle](https://bit.ly/9-Proxy) |
| Bundle | Popular: 1,500 IPs + 50 GB | — | $180 | One-off, balance-based | [Get the Popular bundle](https://bit.ly/9-Proxy) |
| Bundle | Pro: 5,000 IPs + 500 GB | — | $720 | One-off, balance-based | [Get the Pro bundle](https://bit.ly/9-Proxy) |

> Prices and tiers reflect 9Proxy's published pricing after its June 1, 2026 adjustment and can change. Confirm the current numbers on the sign-up page before you top up.

## Which tier you actually need

Two questions decide it.

**Does your job need the same IP twice?** Logging into the same account, keeping a cart alive, or maintaining a session across requests means you need an IP-based package. The unit cost climbs the lower you go, but 100 IPs at $24 with unlimited bandwidth is genuinely cheap for that kind of work — you're not watching a GB meter while a session stays open.

**Does your job need a different IP every few requests?** Then you're buying GB. Everyone over-buys GB on their first attempt, so start at 5 GB for $15 and measure. A high-rotation scraping job that looks enormous on paper often burns a few gigabytes a day; rendering-heavy work burns far more.

Bundles exist because plenty of teams need both, and the Pro tier's 5,000 IPs plus 500 GB at $720 undercuts buying the two separately at list price. If you can't yet predict the split, buying a bundle and watching which half runs out first is a faster way to learn your own usage pattern than guessing.

One thing worth flagging: the 180-day validity on GB packages is a real deadline. If your volume is seasonal or your project stalls, that balance is on a clock. Enterprise tiers remove it, but you're committing to thousands of gigabytes to get there.

## Setting it up

**1. Create the account and add a sub-user.** The sub-user is what the proxy credentials hang off. If you're running multiple campaigns or clients, separate sub-users keep the accounting legible.

**2. Generate a proxy from the dashboard.** You'll get a host, a port, a username, and a password.

**3. Understand the username format.** 9Proxy structures targeting settings inside the username itself:


<subaccount>-country-<country_code>-st-<state_code>-city-<city_code>-isp-<isp_code>-ssid-<session_id>-sst-<session_time>


Real example from their docs: `useruser123-country-US-ssid-rhdN1907ma`. You don't need every field — country plus a session ID covers most jobs. If you want to stay on the same exit IP across requests, keep the `ssid` value identical; change it to rotate.

**4. Route traffic.**

- **Chrome** — install FoxyProxy, add a proxy profile, set type to SOCKS5, paste host/port, and add the credentials. Built-in Chrome settings won't take them.
- **Firefox** — Settings → General → Network Settings → Manual proxy configuration, enter the IP and port under SOCKS, select SOCKS5, and fill in the credential fields.
- **Desktop apps without proxy support** — Proxifier, Profile → Proxy Servers → Add, type SOCKS5. Then create a Proxification Rule naming the specific `.exe`, so only that application gets tunneled.
- **cURL** — `curl -x socks5h://user:pass@host:port https://example.com`. Use the `h` variant unless you specifically want local DNS resolution.
- **Python** — `requests` needs the SOCKS extra installed, then `proxies={"http": "socks5h://user:pass@host:port", "https": "socks5h://user:pass@host:port"}`. Quote special characters in the password with `urllib.parse.quote`.

## Verifying it, and diagnosing it when it breaks

Check your IP without the proxy first, then with it. If the address changed and matches the proxy's location, routing works. Don't stop there — an exit-IP change with the wrong locale or an empty page response is not a working setup.

| Symptom | Most likely cause |
| --- | --- |
| 407 Proxy Authentication Required | Credentials rejected, or the username's targeting string is malformed. This is a proxy error; a 401 is the target site's problem and unrelated. |
| Empty list of alive IPs | Check plan balance first — allocated IPs may already be consumed. |
| Could not resolve host | You're on `socks5://` and expecting proxy-side DNS. Switch to `socks5h://`. |
| Connection refused | Test the endpoint outside your software with cURL. If it fails there too, it isn't your antidetect browser. |
| Slow, then worse | Try a different location. Distance and node load both show up as latency. |
| One app leaks traffic | It ignores system proxy settings. Give it a Proxifier rule instead of a system-wide setting. |

A DNS leak test and a WebRTC leak test are worth running once on any new setup; both catch cases where the IP changed but resolution didn't.

## FAQ

**Is a free SOCKS5 proxy list ever a good idea?**
Public lists are unauthenticated, unstable, and sometimes deliberately logged. Throughput is throttled because the operator is carrying strangers' traffic for free. For anything you wouldn't do on a stranger's Wi-Fi, don't.

**Does SOCKS5 make me anonymous?**
No. It changes your network path. Cookies, browser fingerprints, logged-in sessions, and application headers still identify you. SOCKS5 is one layer of a setup, not the setup.

**What's the difference between `socks5://` and `socks5h://`?**
With `socks5://`, your machine resolves the hostname. With `socks5h://`, the proxy does. If you need the destination name to stay off your local resolver, use the `h` form.

**Can I use SOCKS5 on my phone?**
On iOS, no — per 9Proxy's documentation, iOS only supports HTTP proxies, and only over Wi-Fi, not cellular. Android tooling varies by app.

**IP-based or GB-based for a first purchase?**
If you can't say how many gigabytes a day you'll burn, you probably can't. Start on the smaller GB tier, measure for a week, then move to IP-based if you find yourself pinning sessions.

The short version: SOCKS5 is a routing protocol, not a security product. Most "the proxy is broken" tickets are Chrome's missing auth support, a missing `h` in the scheme, or an app that quietly ignored your settings. Fix those three and the protocol does exactly what it says on the tin.

👉 [Start with a 9Proxy residential SOCKS5 package](https://bit.ly/9-Proxy)
