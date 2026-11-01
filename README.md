# dedicated server backup: why most servers ship without one, and how to build 3-2-1 protection on Sharktech bare metal (Acronis from $4/month)

Search "dedicated server backup" and you're usually trying to answer one of two questions: whether your server already includes backups, and if it doesn't, what a sane backup setup actually costs. Short version: most dedicated servers — including almost every unmanaged server on the market — do **not** include automatic backups by default, and a backup that lives only on the same machine barely qualifies as a backup at all. You'll want a local copy for fast restores plus an off-server copy that survives ransomware and hardware failure.

This guide covers both halves: the backup strategy itself, and the practical part — what it costs to run on Sharktech's bare-metal servers, which start at $219/month and include a 10Gbps port and 300TB of monthly bandwidth, with an Acronis Cyber Protection add-on from $4/month.

> **Quick take**
> Sharktech's services are unmanaged by default, so backups are on you — but the pieces are cheap. Bare-metal servers from **$219/month** (10Gbps port, 300TB bandwidth, 2TB NVMe M.2 included), plus **Acronis Cyber Protection from $4/month** for off-server backup. If you're setting up a server that holds anything you can't re-download, get both. 👉 [check Sharktech's bare-metal configurations and Acronis backup pricing](https://bit.ly/SharKtech)

## Do dedicated servers come with backups included?

Usually no. This is the single most common misconception in the space, and it's worth spelling out how the major providers actually handle it:

- Liquid Web/Nexcess states directly in its documentation that dedicated servers do **not** include automatic system backups by default, and that you're responsible for configuring your own — either a local secondary drive or their Acronis off-server service.
- hosting.com draws a line between managed and unmanaged: managed dedicated customers can buy its JetBackup-powered Cloud Backup as a paid add-on, while VPS and unmanaged dedicated customers "are responsible for configuring their own backups."
- Sharktech's SLA puts it bluntly: all services are considered **unmanaged** unless otherwise expressly agreed. That means no one at the provider is scheduling, verifying, or restoring your backups for you. What Sharktech *does* guarantee is the hardware underneath — more on that below.

So if your plan was "buy the server, the provider handles the rest," adjust it now. The good news is that a working 3-2-1 setup costs about as much as a coffee per month at most providers.

## The 3-2-1 rule, applied to an actual dedicated server

The 3-2-1 rule means three copies of your data, on two different types of storage, with one copy stored offsite. For a dedicated server it maps like this:

1. **Primary data** — the live production server itself.
2. **Local backup** — a separate disk or partition on the same server, not the drive your OS and applications run on.
3. **Offsite backup** — something geographically separate from the machine: an off-server backup service, an S3-compatible object store, or a second server in a different location.

Why bother with two kinds of backup instead of one? They solve different failures. The local copy is your fast lane: someone deletes a folder at 3pm, you restore it in minutes without touching the network. The offsite copy is your disaster lane — it's the only one that survives events that take out the whole machine.

Before you pick frequencies and tools, write down two numbers. The first is your **RTO** (recovery time objective): how long can this thing be offline? InMotion's guidance puts an enterprise SaaS app at roughly 30 minutes and a marketing site at a few hours. The second is your **RPO** (recovery point objective): how much data can you afford to lose? An ecommerce store doing 500 orders an hour can't lose a full day, so it might back up every 15 minutes; a dev server can happily run with a weekly backup. These two numbers decide everything else — backup interval, retention, and whether $4/month of backup capacity is enough or whether you need ten times that.

## Why a backup on the same server barely counts

Liquid Web's official documentation is unusually candid about this: local backups stored on your secondary drive are accessible by your operating system, so if the server gets infected with ransomware, your local backups will likely be encrypted along with everything else. Their docs say flat out that protecting against ransomware requires an off-server solution isolated from the server's OS.

Ransomware isn't even the only scenario. A disk controller failure can take out every drive attached to it. A fire, flood, or stolen chassis takes the machine and its "backups" out together. And in one case a backup can actively make things worse: if you back up a running MySQL database by copying files off the filesystem, you'll often capture a mid-transaction state that won't restore cleanly. Databases need their own tooling — `mysqldump --all-databases --single-transaction --routines --triggers` for MySQL/MariaDB, or Percona XtraBackup for large databases where you want a physical hot backup without locking tables. PostgreSQL gets `pg_dumpall`.

For file-level backups on Linux, Restic is the current favorite: built-in encryption and deduplication, works against S3, Backblaze B2, SFTP, or local storage, and a typical retention setup is 7 daily, 4 weekly, and 6 monthly snapshots. If you'd rather not live in a terminal, Duplicati gives you a web interface. Either way, put the job in cron and — this is the step everyone skips — actually test a restore. As ReliableSite put it years ago, a backup and restoration strategy is useless unless your backups are fully working. Recovery drills are the server equivalent of a fire alarm test: boring until the day they're not.

## What Sharktech gives you: bare metal plus a $4/month Acronis add-on

Sharktech (founded 2003) sells customizable bare-metal servers rather than fixed SKUs — you pick the chassis layout from its listing, then adjust CPU, memory, and drives during configuration. Two things come standard on every bare-metal server: a 10Gbps network port and 300TB of monthly bandwidth, with 2TB NVMe M.2 storage as the starting point. Stock systems are targeted for delivery within 24 hours; custom hardware can take 1–2 weeks depending on component availability.

The relevant piece for a dedicated server backup strategy is the **Acronis Cyber Protection** add-on, which starts at **$4/month**, with the final price set by the backup capacity and protection options you select in the customer portal at checkout. That per-option pricing matters: more protected systems, longer retention periods, and larger datasets all increase storage consumption, so a single server with 14-day retention costs meaningfully less than five servers with a year of restore points. Optional extras like File Sync & Share are priced separately, and the itemized list at checkout shows everything before you confirm.

Here's the current lineup in one table:

| Item | What you get | Price | Billing | Buy |
| --- | --- | --- | --- | --- |
| Bare-metal server (custom) | 10Gbps port, 300TB/mo bandwidth, 2TB NVMe M.2 included; CPU/RAM/storage configurable | From $219/mo | Monthly | [ order a bare-metal server](https://bit.ly/SharKtech) |
| Acronis Cyber Protection | Off-server backup for physical, virtual and cloud workloads; Windows, Linux, macOS | From $4/mo | Monthly | [ configure Acronis backup](https://bit.ly/SharKtech) |

Some regional and legacy listings still circulate — for example, a Los Angeles Dual Xeon E5-2670v2 config with 64GB RAM and 1TB NVMe at $129/month on a Gigabit port appears on Sharktech's own SLA page. Treat those as stock-clearing exceptions rather than the baseline; the current bare-metal catalog centers on the 10Gbps/300TB standard. A third-party server directory that checked the catalog in late October 2026 counted 56 bare-metal configurations ranging from $219 to $699 per month, which matches the official starting price.

If you want to compare configurations or pick a data center location before committing, 👉 [browse the current bare-metal listing](https://bit.ly/SharKtech) shows stock availability per location — Las Vegas, for instance, runs a SOC-2 compliant facility with fully redundant power and network.

## What the SLA does and doesn't cover when a drive dies

This is where most provider comparisons stop too early, and it's directly relevant to backups.

Sharktech's hardware guarantee is solid: if RAM, a processor, a hard drive, a motherboard, a network card, or similar hardware fails, it gets replaced free within **six hours** of you reporting it to the HelpDesk. The network guarantee is 99.99%, with service credits tiered by actual availability — 10% of the monthly fee at 99.0–99.99%, up to 25% at 90.0–94.9%, and 5% per lost percentage point below that. Credits land within two billing cycles and only apply to future bills.

Now the part that affects your backup math. A replaced hard drive is a working hard drive. It comes back empty. Nothing in the SLA covers data recovery, and the exclusions list explicitly includes fires, floods, viruses, hackers, and DDoS attacks — precisely the scenarios your offsite backup exists for. Also note the fine print on claiming credits: you have to report the outage through the HelpDesk within ten business days, and DDoS-driven downtime isn't compensable at all (Sharktech's position is that DDoS protection is built into its network infrastructure rather than sold as an add-on).

The practical conclusion: hardware replacement speed is a reason to like a provider, but it's never a substitute for a backup. Six hours gets your server running again; only a restore gets your data back.

## How off-server backup pricing compares

One thing the top-ranking guides on this topic have in common: they explain the 3-2-1 rule well but go vague the moment real prices come up. Their own backup add-ons are frequently priced "contact us" or buried at checkout. Since you're probably comparing total cost, here's what's actually verifiable:

| Provider | Off-server backup product | Known pricing |
| --- | --- | --- |
| Sharktech | Acronis Cyber Protection | From $4/month; final price set by capacity and options at checkout |
| Liquid Web / Nexcess | Acronis Cyber Backups (partnership) | Price not published in documentation |
| hosting.com | Cloud Backup, powered by JetBackup (managed dedicated only) | Paid add-on; price not published in documentation |
| InMotion | Backup storage via Premier Care bundle or Backup Manager add-on | 500GB included storage; price not stated in the article |
| ReliableSite | Raw FTP backup space | From $0.03 per GB |

A couple of honest observations. ReliableSite's $0.03/GB is cheap per gigabyte, but it's raw FTP space — no snapshots, no agent, no application-aware backups; you're scripting everything yourself. Sharktech's Acronis at $4/month buys you the managed platform layer: a web console where you register multiple systems, apply protection plans, set backup schedules and retention policies, and restore anything from a single file to a full machine. Acronis supports Windows, Linux, and macOS across physical, virtualized, and cloud workloads, which matters if the "one server" quietly becomes three over a year.

Where does each provider fit? If you want a fully managed experience where someone else owns your backup jobs, hosting.com's managed tier with JetBackup exists for that, and Liquid Web's managed offerings sit in the same territory. If you're comfortable being your own sysadmin — which the rest of this article assumes — an unmanaged server plus a low-cost offsite backup add-on is the cheaper path, and it's what Sharktech's model is built around.

## Setting it up: the order of operations

1. **Order the server** with storage split in mind. If you can, configure a second drive for local backups so your local copy never competes with your OS and applications for the same disk.
2. **Add Acronis Cyber Protection during checkout.** Pick backup capacity based on your RPO and retention: daily backups of a server with modest change rates need far less space than hourly ones. The portal shows the itemized price before you confirm.
3. **Register the server in the Acronis web console** and apply a protection plan — schedule, retention, what to include and exclude.
4. **Handle databases separately.** mysqldump or XtraBackup for MySQL/MariaDB, pg_dumpall for PostgreSQL, on their own schedules. File-level backups of a live database are a restore-time surprise waiting to happen.
5. **Run one restore drill now.** Restore a single file, then spot-check that the full-machine restore path works. Verify the retention policy actually pruned old snapshots instead of quietly filling your storage.

## FAQ

**Does Sharktech automatically back up my server?**
No. Sharktech's SLA defines all services as unmanaged by default, and no automatic backup is included with a bare-metal server. Backups are either the Acronis add-on (from $4/month) or whatever you script yourself.

**How much backup capacity do I actually need?**
Start from your RPO and data change rate. A practical rule: estimate how much your data changes between backups, multiply by the number of restore points you keep, then double it for overhead. Acronis's pricing scales with capacity and retention, and the portal prices your exact selection before you commit — so you can check a smaller tier first and expand if the storage consumption graphs say so.

**Is a second drive in the server enough?**
Only for convenience restores. The same ransomware that encrypts your production data will encrypt a locally mounted backup, and a chassis-level failure takes both at once. Keep the local copy for speed, keep an off-server copy for survival.

**What happens to my data when Sharktech replaces a failed drive?**
The drive gets replaced free within six hours of your HelpDesk report — and it arrives blank. Data restoration is your backup strategy's job.

## Which setup to pick

- **Holding anything irreplaceable (client data, databases, production sites):** bare-metal server plus Acronis from day one, both local and off-server copies, one tested restore. The $4/month floor makes this a rounding error next to the $219/month server.
- **Scratch servers, staging, re-downloadable workloads:** skip the add-on, script a cron backup to an offsite destination, and accept a weekly RPO.
- **Need managed backups done for you:** an unmanaged provider is the wrong shape regardless of price — you'd be looking at managed tiers elsewhere, and paying accordingly.
- **High-traffic or abuse-prone workloads:** Sharktech's included DDoS filtering and 10Gbps standard port are doing real work here beyond price; a $219 bare-metal box with 300TB of bandwidth is difficult to match from competitors that meter bandwidth.

One last check before you order: confirm your chassis has bays for a second drive if you want local backups, and pick your data center location from the stock listing rather than assuming every config exists everywhere. Then the whole stack — 👉 [bare-metal server from $219/month with Acronis backup from $4/month added at checkout](https://bit.ly/SharKtech) — is one order form away from running.
