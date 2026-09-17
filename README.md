# cloud computer storage: NVMe vs SSD vs HDD vs S3 Explained, with Real Monthly Prices and No Bill Shock

Type "cloud computer storage" into a search bar and you get a strange mix of results: some pages explain cloud storage, some explain cloud computing, and some just assume you want a Dropbox-style file sync app. The phrase itself blends two things, which is probably why you're here. This article untangles them, explains the storage types you actually run into when computing in the cloud, and puts real numbers on what each one costs, using rate cards published openly by Sharktech, an infrastructure provider that has been running its own network since 2003.

By the end you should know which storage type fits your workload, roughly what a terabyte per month costs at each performance tier, and where the surprise charges on cloud bills tend to hide.

## First, What Are You Actually Looking For?

"Cloud computer storage" sits between two distinct concepts, so let's split them cleanly.

**Cloud computing** is renting processing power — virtual machines, CPUs, RAM — that run somewhere else's data center. Storage in that context means the disks attached to your virtual machines: the drive your database sits on, the volume your operating system boots from.

**Cloud storage** in the consumer sense means services that hold your files and sync them across devices. But in the infrastructure world, it means the same underlying thing: disks in a data center, just billed and accessed differently.

If you searched this phrase, you most likely fall into one of two camps. Either you're deploying something that computes — an app, a database, a game server, a CI pipeline — and you need to figure out where its data lives and what that costs. Or you're comparing storage options in general and keep hitting jargon like "block storage," "object storage," and "NVMe tiers" without a plain explanation.

Both camps end up needing the same knowledge. So let's build it from the ground up.

## The Three Flavors of Storage You'll Meet in the Cloud

Almost every cloud platform, from the hyperscalers down to smaller infrastructure providers, offers storage in three broad shapes. Understanding the split does most of the work of choosing correctly.

**Block storage** is the one that looks familiar. It's a volume that attaches to a virtual machine and behaves like a hard drive. Your OS boots from it, your database writes to it, your application treats it as a disk. It comes in performance tiers — usually HDD, SSD, and NVMe — and you pay per gigabyte, typically per hour or per month.

**Object storage** is different. There's no drive letter, no file system. You put data into buckets as objects — a photo, a video, a backup archive, a build artifact — and retrieve them over an HTTP API. The S3 API has become the de facto standard, which means almost every backup tool, DevOps platform, and scripting library speaks it natively. Object storage is slower for random reads and writes but extremely cheap at scale, and it's the right home for anything you write once and read occasionally: backups, archives, media files, logs.

**File storage** is the third shape — a shared network filesystem multiple machines can mount at once. Useful for legacy enterprise apps, less common in modern cloud deployments. Many smaller providers skip it entirely; you can approximate it with block storage or object storage depending on your needs.

Here's the practical rule of thumb: if your data is being *actively computed on* — a database, application state, anything where latency matters — you want block storage, and you want it on the fastest tier your budget allows. If your data is being *kept* — backups, archives, user uploads, release artifacts — object storage wins on price almost every time.

## Performance Tiers: What NVMe, SSD, and HDD Actually Deliver

Within block storage, the tier you pick changes both speed and cost dramatically. Sharktech's cloud platform, which runs on OpenStack across data centers in Los Angeles, Las Vegas, Denver, Chicago, and Amsterdam, publishes all three tiers with estimated performance figures:

| Storage tier | Estimated throughput | Estimated IOPS | Hourly rate per GB |
| --- | --- | --- | --- |
| NVMe | 1.2 GB/s | ~18,000 | $0.00009 |
| SSD | 350 MB/s | ~6,000 | $0.00006 |
| HDD | 120 MB/s | ~3,000 | $0.00002 |

(The provider notes results vary with technical factors — treat these as ballparks, not guarantees.)

The gap between tiers is enormous. NVMe runs at roughly four times the IOPS of HDD. In HostAdvice's independent benchmarking of the platform, a VM rebuilt on the NVMe layer hit around 5,020 MB/s sequential reads — numbers their reviewer described as "hyperscaler territory" — while the default SSD tier landed in traditional-SSD territory: fine for websites and small databases, limiting for heavy video processing or large-scale databases.

What does this mean in monthly terms? Using the published hourly rates and a 730-hour average month, a full terabyte of block storage costs approximately:

- **NVMe:** $0.00009 × 1,000 GB × 730 hrs ≈ **$65.70/month per TB**
- **SSD:** $0.00006 × 1,000 GB × 730 hrs ≈ **$43.80/month per TB**
- **HDD:** $0.00002 × 1,000 GB × 730 hrs ≈ **$14.60/month per TB**

That's my arithmetic from their rate card, not an official price sheet — but it's the math the rate card implies, and it illustrates the real decision. NVMe costs about 4.5 times what HDD does per gigabyte. If your workload is latency-sensitive, that premium is cheap. If you're storing archives on NVMe because the ordering form defaulted to it, you're burning money every hour.

The same rate card prices compute alongside storage — vCPUs at $0.0025/hour (about $1.83 per core per month) and RAM at $0.0035/hour per GB (about $2.56 per GB per month) — which matters because in reality you rarely buy storage in isolation. You buy a pool of CPU, RAM, and disk, then slice it into as many virtual machines as the pool allows. On this platform, at least, the number of VMs is unlimited; your pool is the limit.

👉 [See the full cloud rate card and deploy a resource pool](https://bit.ly/SharKTech)

## Object Storage: The Flat $4.90/TB Option

For data that isn't being constantly rewritten, Sharktech also offers S3-compatible object storage, and its pricing model is about as simple as storage pricing gets: **$4.90 per TB per month for storage, with bandwidth at $0.00 per TB**. The invoice has exactly two line items — storage and bandwidth — and one of them is zero.

Compare that to the ~$14.60/TB you'd pay for even the cheapest HDD block storage tier, and the gap widens further against SSD or NVMe. For backups, media libraries, compliance archives, and CI/CD artifacts, object storage at this price point is hard to argue with. There's no minimum commitment — the provider specifically states you get the same rate whether you start small or grow big, which sidesteps the volume-commitment contracts larger providers often require for their best rates.

The use cases are the standard S3 playbook. DevOps teams store build artifacts and deployment packages in buckets, integrating with tools like Jenkins, GitLab, and Terraform through the API. Web apps serve user-uploaded images and video directly from storage. Companies park regulatory and legal archives there because retrieval stays possible without paying for fast disks year-round.

One honest caveat: object storage is not fast for random access. It's built for data written once and read occasionally. Don't put a live database on it — that's what the NVMe tier above is for.

👉 [Check current S3 object storage pricing and deploy a bucket](https://bit.ly/SharKTech)

## The Full Plan Lineup: Every Tier on the Official Page

Sharktech's cloud comes in two billing models on the same infrastructure. **Public Cloud** is pay-as-you-go: each plan includes a base resource commit, and if you exceed it, you pay hourly for the overage. **Dedicated Cloud** is prepaid: you order a fixed pool and get exactly that, flat-billed monthly, with no overage charges possible. Their own summary of the difference: "If you pay for 8 cores, you get 8 cores. No more, no less."

Here is every tier currently shown on their official cloud page:

**Dedicated Cloud (prepaid, flat monthly billing):**

| Tier | What it is | Price | Billing cycle | Purchase |
| --- | --- | --- | --- | --- |
| Tiny | Entry-level fixed resource pool | $7.95/mo | Monthly, flat | [Deploy the entry tier](https://bit.ly/SharKTech) |
| Small | Larger fixed pool, same flat model | Set via on-site calculator | Monthly, flat | [Configure a Small pool](https://bit.ly/SharKTech) |
| Medium | Mid-size pool for multi-VM setups | Set via on-site calculator | Monthly, flat | [Configure a Medium pool](https://bit.ly/SharKTech) |
| Large | Full multi-tier app territory | Set via on-site calculator | Monthly, flat | [Configure a Large pool](https://bit.ly/SharKTech) |
| Huge | Heavy workloads, larger teams | Set via on-site calculator | Monthly, flat | [Configure a Huge pool](https://bit.ly/SharKTech) |
| Giant | Beyond Huge, same predictable billing | Set via on-site calculator | Monthly, flat | [Configure a Giant pool](https://bit.ly/SharKTech) |
| Colossal | Top of the prepaid lineup | Set via on-site calculator | Monthly, flat | [Configure a Colossal pool](https://bit.ly/SharKTech) |

**Public Cloud (pay-as-you-go with a base commit):**

| Tier | What it is | Price | Billing cycle | Purchase |
| --- | --- | --- | --- | --- |
| Small | Base commit + hourly overage | Set via on-site calculator | Monthly/hourly | [Deploy Public Cloud](https://bit.ly/SharKTech) |
| Medium | Larger base commit | Set via on-site calculator | Monthly/hourly | [Deploy Public Cloud](https://bit.ly/SharKTech) |
| Large | Larger still, resource-capped | Set via on-site calculator | Monthly/hourly | [Deploy Public Cloud](https://bit.ly/SharKTech) |
| Enterprise | 64 vCPUs, 128 GB RAM, 5,000 GB SSD, 20 TB bandwidth (per HostAdvice's checkout walkthrough) | $499/mo | Monthly/hourly | [Deploy the Enterprise tier](https://bit.ly/SharKTech) |
| Custom | Tailored compute/storage/network allocation | Contact sales | Quoted | [Get a custom quote](https://bit.ly/SharKTech) |

A note on the "calculator" entries: the official page presents most tier prices through an interactive calculator that adjusts with your location, resource mix, and billing cycle, rather than fixed printed figures. I'm not going to invent numbers the page doesn't state. The two prices above — the $7.95 entry tier and the $499 Enterprise tier — are the ones visible on the page and confirmed in HostAdvice's hands-on review respectively. For everything in between, the calculator on the order page gives you the real figure in about thirty seconds.

Also worth knowing: Public Cloud plans (except Enterprise and Custom) carry a **maximum resource cap** so a runaway script at 3 AM can't rack up an unbounded bill. It's a small policy detail that says a lot about who this platform is built for.

Every tier runs on the same OpenStack foundation with unlimited VMs from your pool, private networking, load balancers, firewall security groups, IPv6 support, and a 99.999% uptime guarantee.

## Where Cloud Storage Bills Actually Hide

Ask anyone who has migrated off a big cloud platform why they left, and the answer usually isn't compute pricing. It's egress — the fee for moving your own data back out.

Data transfer is where cloud bills get weird. Ingress is often free (data coming in), but egress — serving your website, delivering your files, backing up to another region, or just leaving the platform — gets billed per gigabyte at rates that quietly become the largest line item for anything bandwidth-heavy. The effect also works as soft lock-in: the more data you store, the more it costs to walk away.

Sharktech's policy here is unusually simple, and it's published plainly on the cloud page:

- **Incoming traffic: unlimited, free.** No ingress charges, period.
- **Outgoing traffic: 5,000 GB included** with cloud services, then **$0.002 per GB** beyond that.
- **First public IPv4 address: free**; additional ones are $1.50/month each.

Two dollars per terabyte of overage egress is a fraction of what hyperscalers charge for outbound data transfer. Sharktech itself claims customers save 50–80% on cloud costs compared to the big platforms — that's the vendor's own marketing figure, so discount it accordingly, but the structural ingredients behind it (free ingress, near-zero egress rates, no proprietary lock-in) are verifiable on the pricing page, and the direction of the savings for bandwidth-heavy workloads is hard to dispute.

There's also the exit-cost question, which people forget until it's too late. Platforms built on proprietary tooling can make leaving painful. This one is OpenStack-based, and the provider states you can download your own disk images at any time — for backup, migration, or simply taking your workloads elsewhere. No export fees, no hostage situation. If you've ever priced out moving 10 TB between cloud providers, you know why this matters.

## What Independent Testing and Reviews Show

Vendor pages tell you what a company wants you to believe, so here's the third-party picture.

HostAdvice's expert review of the public cloud, which included actual benchmark runs, scored the platform 9.4/10 overall. Their testing found CPU performance stable under load (about 13,000 sysbench events/second with sub-millisecond latency), memory throughput around 45 GB/sec, ~10 Gbps network speeds inside the infrastructure with 0.17 ms idle latency, and full stability in a 60-second stress test. Their reviewer's support ticket — sent at 1 AM — got a reply in 39 minutes. Their criticisms: SSD-tier disk speeds are only decent (NVMe is the fix), ticket answers can be general for advanced tuning questions, and the number of global regions is limited compared to hyperscalers.

Trustpilot tells a more mixed story: around 3.5 out of 5, but across a small number of reviews — too small a sample to treat as definitive. The recurring positive themes across review platforms are consistent: network reliability, DDoS protection that's built into the network rather than bolted on, and support staffed by humans reachable by phone 24/7/365, which is genuinely uncommon at this price tier.

The most important caveat, and one you should weigh seriously: **there is no refund policy.** Per HostAdvice's review, all payments are non-refundable, with the only exception being a billing dispute raised within 30 days that Sharktech upholds — which yields account credit, not cash back. There's also no free trial. Payment options include credit cards, PayPal, wire transfers, Western Union, and Alipay.

> Practical implication: test with the $7.95 entry tier before committing to a larger pool or the Enterprise tier. The hourly billing on Public Cloud means a short experiment can cost pocket change, but whatever you prepay, you should assume you're keeping.

## How to Actually Choose

Strip away everything above and the decision tree is short:

1. **Is the data being computed on right now?** Database, application state, active working set → block storage. Latency-sensitive (production database, AI workloads) → NVMe. General-purpose (websites, most apps) → SSD is enough. Archival on a budget → HDD.
2. **Is the data being kept rather than computed?** Backups, media, archives, build artifacts → S3 object storage at $4.90/TB with free bandwidth.
3. **Is your workload predictable or spiky?** Predictable → Dedicated Cloud, flat monthly bill, no overage anxiety. Spiky or experimental → Public Cloud, hourly metering with a resource cap so it can't spiral.
4. **Do you move a lot of data in and out?** If yes, ingress and egress pricing should be a first-class decision factor, not an afterthought. Free ingress and $0.002/GB egress change the math for anything bandwidth-heavy.

And a hybrid pattern worth knowing: many setups put the hot path on NVMe block storage and everything else in object storage buckets. A 100 GB NVMe volume plus a few TB of S3 costs a fraction of keeping the whole dataset on fast disks, and the performance where it matters stays intact.

## Before You Click Deploy: A Short Checklist

A few closing facts that don't fit neatly anywhere else but genuinely affect the experience:

- **Five locations to choose from:** Los Angeles, Las Vegas, Denver, Chicago, and Amsterdam. Picking the one closest to your users is the cheapest latency optimization available.
- **DDoS protection is built into the network**, not sold as an add-on — relevant if you're running game servers or anything that attracts attention.
- **Both Linux and Windows VMs are supported**, deployed from official cloud images updated weekly, with support for custom ISOs and your own disk images.
- **No setup fees on cloud services**, and resources scale up or down from the management panel without redeploying.
- **Plan for the no-refund policy.** Start at $7.95, verify performance matches your workload, then scale.

Cloud computer storage, once you break the phrase apart, isn't mysterious. It's a small set of storage types with published rates, a billing model you can do arithmetic on, and a handful of policy details — egress fees, exit rights, refund terms — that determine whether the bill matches what you expected. The providers that publish their rate cards openly and price egress sanely make that arithmetic possible. The ones that don't, make you find out the hard way.

If you want to run the numbers yourself, the order page's calculator will price any configuration — tier mix, location, VM count — before you commit to anything.

👉 [Open the calculator and price your configuration](https://bit.ly/SharKTech)
