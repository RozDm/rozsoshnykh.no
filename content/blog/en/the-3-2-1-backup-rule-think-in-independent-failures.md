---
title: "The 3-2-1 backup rule: think in independent failures"
description: "A backup is judged only by the restore. 3-2-1 is a model of independent failures: each part answers a threat, and you see why the rule looks the way it does."
date: "2026-11-21"
tags: ["Sikkerhet"]
translationOf: "3-2-1-regelen-for-sikkerhetskopiering-tenk-i-uavhengige-feil"
draft: true
---

A backup is judged on one thing: whether you get the data back when you need it. Everything before that is preparation, and anything untested is a claim.

The 3-2-1 rule, commonly attributed to the photographer Peter Krogh, is usually remembered as three numbers. The numbers are shorthand for a model: your copies must not be lost to the same cause. Once you have the model, you can see why the rule looks the way it does, and when it is not enough.

## The model: independent failures

Everything that can destroy data has a reach. A deleted file affects one file, a dead disk one disk, a fire a whole room. A copy protects you only if it sits outside the reach of the failure, so independence is always relative to a threat. Two copies that can be lost to the same cause are one copy against that cause. The original and a copy in the same room count as one copy against a fire and as two against a dead disk.

The question to put to any setup is this: **for each event I must survive, how many copies are left afterwards?** None means no protection against that event, and one means no margin. Decide which events need margin.

## The rule, part by part

| Part | The threat it answers | Without it |
| --- | --- | --- |
| 3 copies of the data, the original included | A second failure while you recover from the first | A failure during the restore leaves you with nothing |
| 2 different kinds of storage | Failures that hit a whole type: the same manufacturing fault, the same wear, the same software bug | One systematic fault takes all the copies at once |
| 1 copy at another site | Events that take a whole place: fire, water, theft, lightning, a power surge | All the copies are within reach of the same event |

**Why three.** With three copies, a single loss (a dead disk, a deleted file) still leaves two, one to restore from and one in reserve if the restore fails. The reserve matters because a restore reads everything, under stress, for the first time in a long while, and that is when latent faults show. Against the loss of a site or a fault in one kind of storage, 3-2-1 guarantees only that one copy survives, with no margin. That is the minimum the rule buys, and data that needs margin there too needs more copies, kinds or sites.

**Why two kinds.** Two disks of the same type, bought and put into service together, wear alike and may share a hidden defect, and so do two copies managed by the same software. What matters is whether the failure modes differ, whatever the labels say. With three copies on two kinds, two of them share a kind, and if a fault of that kind strikes you are left with one copy and no margin.

**Why another site.** How far away depends on the event you are guarding against. The copy has to lie outside the reach of whatever can hit the original, so the same room, the same building or the same power circuit all mean the same fate.

## Decide first what you must survive

Two numbers drive everything else, and both are decisions that come before any solution. RPO (Recovery Point Objective) is how much data you can afford to lose, measured in time. RTO (Recovery Time Objective) is how long you can afford to be down. Write them down, and measure against them later.

RPO sets how often you copy, and a copy every night means you can lose up to a day. A low RPO rarely comes from copying everything often. It comes from a base copy plus a continuous archive of the transaction log, so that the RPO depends on the log and not on how often the full copies run. RTO sets the media and the location, because bringing a large volume back over a slow link takes time. A copy that needs a week to come back protects against disasters and does nothing for a mistake on a Tuesday. Many setups therefore keep two tiers, a near, fast copy for everyday errors and a distant one for whatever takes the whole site.

Sort the data too, because not everything deserves the full rule and a long history. Irreplaceable data gets both. Reproducible data needs only what reproduces it, meaning code and configuration, which you do back up. Temporary data such as caches, build output and rebuildable indexes needs no copy. Without this sorting, cost and RTO both explode.

## What is not a backup

Three things look like a backup and are useful, but they answer a different question.

- **RAID and replication** give availability, so the service carries on when a disk or a machine drops out. In their usual form they pass every change on within moments, which means a deletion, a corrupted file or a wrong overwrite is copied as quickly as anything else. Delayed replication gives a window to stop the error spreading and to recover from the lagging replica, but it still belongs to the same administrative domain.
- **Sync** keeps two places identical, so deleting a file in one deletes it in the other. A sync service's version history or trash helps against some mistakes, but it remains in the same administrative domain.
- **Snapshots on the same system** are good for a quick undo, but they share hardware, file system and administrators with what they protect. If the system goes, the snapshots go with it. They become a copy once they have been copied out.

A simple test is whether one command or one mistake in production can remove the copy. If it can, the copy is not a backup. Passing the test is necessary, not sufficient, because the copy must also lie outside the hardware and location domain. Snapshots that production cannot delete still share the hardware.

## Consistency

Copied files are not the same as a consistent state. A database copied at file level while it runs can end up unusable, and related systems captured at different times can disagree, for instance the database at 02:00 and the file store it refers to at 02:40. Use the database's own consistent dump or snapshot. If related systems keep changing, no order makes them consistent, and the order only decides which way they diverge. Capture the system that refers first and the one it refers to afterwards, the database before the file store. Then every object the database names exists and the extras are harmless, as long as an object is deleted only after a delay longer than the gap between the two captures, and objects are not overwritten in place.

## Independence is the real rule

Hardware and location are easy to see, and the administrative domain is the one that gets forgotten. If every copy can be reached with the same account, the same credentials or from the same admin machine, one fault can delete them all, whether it is a compromised machine or a clean-up script with the wrong path. The copies may be physically apart and still be one copy administratively.

- **Separate access.** The backups have their own accounts and credentials, and administering production does not make you an administrator of the copies.
- **Least privilege.** The production side can write new copies but cannot delete or overwrite old ones. Better still, the backup system pulls the data itself with read-only access, so a compromised production side holds no key to the copies. Retention rules are set and enforced by the backup system as well.
- **Few common dependencies.** The same password manager or management tool is a shared point of failure, however many copies stand behind it.
- **The organisational and contractual domain.** Copies with the same provider, under the same contract, legal entity or payment can share a fate even with separate accounts and immutability. An account can be closed by mistake or for a breach of terms, data can be deleted after an unpaid bill, and a provider can stop the service. At least one copy should be outside that domain.
- **Independence in time.** Versions in an incremental or deduplicated chain depend on each other. One damaged base block or a damaged repository index can take every later version, and a log chain is the extreme case, because one missing log segment ends the restore at that point even if the base copy is intact. Check the integrity of the repository itself regularly, take independent full copies at intervals, and make at least one copy directly from the source into a separate repository, since replicating from the same repository copies the damage.

## 3-2-1-1-0

The extended form adds two parts for failures the simple rule does not cover.

**+1: a copy that is offline or immutable.** It need not be a fourth copy and can be one of the three, often the off-site one, but it adds no margin by itself. An offline copy is physically disconnected except while it is being written, so nothing running on the network can reach it. An immutable copy can be written only once and is kept for a fixed time that nobody can shorten, not even an administrator. It gives that guarantee only if the storage itself enforces the lock, not if it is a setting the same credentials can change. This covers deletion and encryption carried out with legitimate access, and faults in the automation. You cannot clear the copy away until the time is up, and the reverse can happen too. Whoever can write can fill immutable storage with junk that nobody can delete, so the quota fills and new copies stop, or the bill grows until the account is closed. Limit the write rights, and cap how much one run, or one period, may add, so that a run far beyond normal stops and raises an alarm before the storage fills. A quota alone only protects the budget. The same cap catches a pulled copy of junk or ciphertext from a compromised source.

**0: zero errors in verified restores.** A job that reports success proves little. What counts is a restore that was read back and compared, and that produced zero errors.

## Versions and retention

A backup that overwrites yesterday's copy with today's is a mirror with a delay, because corrupted data is copied on the next run. You need history, usually in tiers that thin out with age: daily copies for the last few weeks, weekly for the months, monthly further back.

Compare that history with the time it takes you to notice a problem. If silent corruption starts in March, you notice in May and you keep six weeks, no intact version exists. **If the time to detect a problem is longer than the retention, the data is lost.** Retention must therefore exceed the longest detection time you can realistically expect.

Retention also has an upper limit, since some data must be deleted after a time and the deletion must sooner or later reach the copies too. Immutability and required deletion have to be reconciled in the design. Either the immutability window is no longer than the time within which deletion must happen, or you encrypt data sets with separate keys and delete the key (cryptographic erasure), including the escrowed copies of it. That works only at coarse granularity (per customer, data set or period), and it moves the problem to the key store, which needs its own design. It must survive as well as the data does, yet still allow keys to be deleted.

Run the model's question over a typical setup, with the original and a local copy at site 1 and an immutable copy at site 2, and this is what it gives.

![A grid showing how many copies survive each event. The columns are the original (site 1, kind A), a local copy (site 1, kind B), an off-site copy (site 2, kind A, immutable) and the number of copies left. A deleted file or a dead disk: the original is lost, the other two are kept, 2 copies left. A fault in kind A: the original and the off-site copy are lost, the local copy is kept, 1 left. Fire at site 1: the original and the local copy are lost, the off-site copy is kept, 1 left. A compromised shared admin account: the original and the local copy are lost, the off-site copy is kept, 1 left. Corruption older than the retention: all three are lost, 0 left. Each result is also labelled with a word: margin for 2, no margin for 1 and data lost for 0.](/images/3-2-1-regelen-for-sikkerhetskopiering-tenk-i-uavhengige-feil/copies-left.svg)

## Encryption

Encrypt every copy. For copies that leave your control, whether at another site, with another party or on media that are moved, encryption is not optional, and it is done on your side, before the data leaves you, with keys you hold yourself. Encryption at rest with the provider's keys protects neither against a compromised account nor against the provider. The channel should be encrypted in transit as well. Every copy is also a copy for a thief, so more copies widen the attack surface, and a backup protects availability and integrity but does nothing against stolen data.

The key becomes a single point of failure for every copy. It must never live only inside the backup it unlocks, since then you need the copy to open the copy, and never only on the system that can die. Keep it separately, in at least two places that do not share a fate, ideally including a sealed paper copy somewhere else. Better still, split it into shares held by different people or places, so that no single share opens the data and losing one share does not stop the restore (M of N). Test that a person who did not build the setup can get to it.

## What a restore needs

The backup itself holds the data, the configuration (how the systems were set up, so that they can be rebuilt as well as refilled) and the secrets, encrypted as described above.

Outside it sits a layer zero that comes up with nothing from the system being restored. It holds the restore runbook (step by step, written for someone who is stressed and did not build the setup), the key, the tool to read the format and the access to the backup storage, stored separately from the backup and readable without it. If reading the copy needs an agent, a licence server or a console that died with the infrastructure or no longer exists, the copy is useless.

The restore order must have no cycles. A key in a secrets manager that needs the sign-in service, which itself has to be restored, is a cycle, and so is code in a repository behind that same sign-in. The services everything rests on, such as sign-in and name services, come early in the order.

Anything deliberately left out should be on a list.

## The restore drill

Backups fail silently through the wrong path, missing files, a job that reports success after copying an empty folder, a format that needs software you no longer have or a damaged medium, and none of it shows until you restore.

- **Spot checks, often.** Restore a few random files and check the contents. It is cheap, so do it regularly.
- **A full restore, on a schedule.** Start from clean equipment with only the copy and what layer zero holds: the key, the runbook, the tool and the access to the storage. Time it and compare with the RTO. Many people find out here that the RTO was a wish.
- **Integrity.** Store checksums when the copy is made and verify them on reading and at intervals, so that silent bit rot in a stored copy is found before you need it. For a database, a restore means that it starts and passes its own consistency check.
- **A granular restore.** One table or one object from a given day, without rolling back everyone else's work. A whole-system image can pass the full-restore test and still fail this.

Layer zero rots silently too, through an expired token, a second factor tied to someone who left or a sealed key from before the last rotation. The drill finds these, and what surprised you goes into the runbook.

## After a break-in

Do not restore whole system images after a break-in, because that restores the intruder as well. Rebuild the systems from configuration and code, restore only data from a point before the intrusion, and treat it as suspect. That needs a clean version to still exist, and since intrusions are often found late, they belong among the problems retention must outlast (see above).

## Automation and monitoring

Manual backups stop happening sooner or later, through a busy week or a person who has left. Automate the schedule, the verification and the clean-up after the retention period.

Automation fails silently too, and an alert on failure is not enough, because a job that never starts sends no error. Alert on the absence of a success. Expect one successful run of each scheduled job, the copy and the verification alike, per period, and raise the alarm when one does not arrive. Also watch the age and size of the newest copy. A sudden change in either direction is a signal, since a drop can mean data is missing and growth can mean data encrypted by someone other than you, which can be neither deduplicated nor compressed, so the increments balloon. This is the pulse idea from [Who watches the watchers? Oversight in IT infrastructure](/en/blog/who-watches-the-watchers-oversight-in-it-infrastructure/). A check must not share a fate with what it watches, and silence has to count as a finding.

## Checklist

- We have written down the RPO and the RTO, and sorted the data into irreplaceable, reproducible and temporary.
- For each event we must survive, we know how many copies are left afterwards, and that number meets the margin we decided on.
- There are at least three copies, the original included, on at least two different kinds of storage, and at least one is at another site. No single provider, contract or payment account holds every copy.
- No single event (a fault, an account, a machine or a person) can delete all the copies. Production cannot delete or overwrite them, and better still, it has no access to them at all.
- At least one copy is offline or immutable, with limited write rights and a cap on how much one run, or one period, may add.
- We keep several versions, and the retention is longer than the time it takes to notice a problem.
- The repository is checked for integrity regularly, and at least one copy is made directly from the source into a separate repository, not replicated from the same repository.
- Every copy is encrypted with keys we hold, and the key is split or kept in at least two places that do not share a fate.
- The copying and the verification run automatically, and we get an alarm when a successful run of either fails to arrive.
- Configuration and secrets are in the backup. The runbook, the key, the tool to read the format and the access to the storage sit outside it, stored separately and readable without it, and the restore order has no cycles.
- The copies are consistent, with related systems captured at one point in time or in an order where nothing points to a missing object.
- We have restored from scratch, granularly too, timed it against the RTO, and repeat it on a schedule.

## A story to end with

Someone once told me about a recovery they did for a woman whose husband had died. All the photos of him, from the holidays and trips the two of them took together, were on one hard drive, and the drive failed.

Getting the data back took a year and a half. It meant finding a donor drive, an identical drive to take parts from, and working to recover the data. They knew her, so they took no money for it.

They told me she cried when she got them back. All the photos of her husband, everything, recovered.
