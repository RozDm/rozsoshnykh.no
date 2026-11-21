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

This is a reminder of principles that are easy to stop thinking about after years at work, and that are rarely thought about at home.

## The model: independent failures

Everything that can destroy data has a reach. A deleted file affects one file, a dead disk one disk, a fire a whole room. A copy protects you only if it sits outside the reach of the failure, so independence is always relative to a threat. Two copies that can be lost to the same cause are one copy against that cause. The original and a copy in the same room count as one copy against a fire and as two against a dead disk.

The question to put to any setup is this: **for each event I must survive, how many copies are left afterwards?** None means no protection against that event, and one means no margin. Decide which events need margin.

Two numbers decide how much of this you need. RPO (Recovery Point Objective) is how much data you can afford to lose, measured in time, and RTO (Recovery Time Objective) is how long you can afford to be down. They decide how often you copy and which media and locations you use, so write them down before choosing a solution. Sort the data too, since irreplaceable data needs the full rule, reproducible data needs only the code and configuration that rebuild it, and temporary data needs nothing.

## The rule, part by part

| Part | The threat it answers | Without it |
| --- | --- | --- |
| 3 copies of the data, the original included | A second failure while you recover from the first | A failure during the restore leaves you with nothing |
| 2 different kinds of storage | Failures that hit a whole type: the same manufacturing fault, the same wear, the same software bug | One systematic fault takes all the copies at once |
| 1 copy at another site | Events that take a whole place: fire, water, theft, lightning, a power surge | All the copies are within reach of the same event |

**Why three.** With three copies, a single loss (a dead disk, a deleted file) still leaves two, one to restore from and one in reserve if the restore fails. The reserve matters because a restore reads everything, under stress, for the first time in a long while, and that is when latent faults show. Against the loss of a site or a fault in one kind of storage, 3-2-1 guarantees only that one copy survives, with no margin. That is the minimum the rule buys, and data that needs margin there too needs more copies, kinds or sites.

**Why two kinds.** Two disks of the same type, bought and put into service together, wear alike and may share a hidden defect, and so do two copies managed by the same software. What matters is whether the failure modes differ, whatever the labels say. With three copies on two kinds, two of them share a kind, and if a fault of that kind strikes you are left with one copy and no margin.

**Why another site.** How far away depends on the event you are guarding against. The copy has to lie outside the reach of whatever can hit the original, so the same room, the same building or the same power circuit all mean the same fate.

## What is not a backup

Three things look like a backup and are useful, but they answer a different question.

- **RAID and replication** keep the service running when a disk or a machine drops out, and in their usual form they pass every change on within moments, so a deletion or a corrupted file is copied as quickly as anything else.
- **Sync** keeps two places identical, so deleting a file in one deletes it in the other. A cloud photo library on a phone is sync with one provider under one account, so if you delete a photo or lose the account, it is gone everywhere.
- **Snapshots on the same system** are good for a quick undo, but they share hardware, file system and administrators with what they protect, so they become a copy only once they have been copied out.

A simple test is whether one command or one mistake in production can remove the copy. If it can, the copy is not a backup. Passing the test is necessary, not sufficient, because the copy must also lie outside the hardware and location domain.

## Independence is the real rule

Hardware and location are easy to see, and the administrative domain is the one that gets forgotten. If every copy can be reached with the same account, the same credentials or from the same admin machine, one fault can delete them all, whether it is a compromised machine or a clean-up script with the wrong path. The copies may be physically apart and still be one copy administratively.

- **Separate access.** The backups have their own accounts and credentials, and administering production does not make you an administrator of the copies. Production can write new copies but cannot delete or overwrite old ones, and better still, the backup system pulls the data itself with read-only access.
- **Another provider, contract or account.** Copies with the same provider, under the same contract, legal entity or payment can share a fate even with separate accounts and immutability. An account can be closed by mistake or for a breach of terms, data can be deleted after an unpaid bill, and a provider can stop the service. At least one copy should be outside that domain.
- **Versions in a chain.** Versions in an incremental or deduplicated chain depend on each other, so one damaged base block or repository index can take every later version. Check the repository regularly, and make at least one copy directly from the source into a separate repository, since replicating from the same repository copies the damage.

## Data rots quietly

Stored data degrades even when nobody touches it. Disks develop sectors that can no longer be read, flash memory loses its charge, faster when it is worn or left unpowered for a long time, and optical discs and tapes age. Errors also slip in on the way, in transfer, in memory or in firmware. Most of it is silent, and the file still opens, or seems to, until the day it does not. Storage systems at work often check for this. A drive in a drawer at home, or a memory stick with the only copy of the photos, gets no such care.

The defence is the same everywhere. Store checksums and verify every copy regularly, so that a rotten copy is found while a good one still exists, and replace it from that one. Move data to new media before the old ones age out. And keep enough history, because rot that is copied into every backup and outlives the retention takes the last good version with it.

## 3-2-1-1-0 and retention

The extended form adds two parts for failures the simple rule does not cover.

**+1: a copy that is offline or immutable.** It need not be a fourth copy and can be one of the three, often the off-site one, but it adds no margin by itself. An offline copy is physically disconnected except while it is being written, so nothing running on the network can reach it. An immutable copy can be written only once and is kept for a fixed time that nobody can shorten, not even an administrator. It gives that guarantee only if the storage itself enforces the lock, not if it is a setting the same credentials can change.

**0: zero errors in verified restores.** A job that reports success proves little. What counts is a restore that was read back and compared, and that produced zero errors.

A backup that overwrites yesterday's copy with today's is a mirror with a delay, because corrupted data is copied on the next run. You need history, usually in tiers that thin out with age: daily copies for the last few weeks, weekly for the months, monthly further back.

Compare that history with the time it takes you to notice a problem. If silent corruption starts in March, you notice in May and you keep six weeks, no intact version exists. **If the time to detect a problem is longer than the retention, the data is lost.** Retention must therefore exceed the longest detection time you can realistically expect.

Run the model's question over a typical setup, with the original and a local copy at site 1 and an immutable copy at site 2, and this is what it gives.

![A grid showing how many copies survive each event. The columns are the original (site 1, kind A), a local copy (site 1, kind B), an off-site copy (site 2, kind A, immutable) and the number of copies left. A deleted file or a dead disk: the original is lost, the other two are kept, 2 copies left. A fault in kind A: the original and the off-site copy are lost, the local copy is kept, 1 left. Fire at site 1: the original and the local copy are lost, the off-site copy is kept, 1 left. A compromised shared admin account: the original and the local copy are lost, the off-site copy is kept, 1 left. Corruption older than the retention: all three are lost, 0 left. Each result is also labelled with a word: margin for 2, no margin for 1 and data lost for 0.](/images/3-2-1-regelen-for-sikkerhetskopiering-tenk-i-uavhengige-feil/copies-left.svg)

## Encryption and what lives outside the backup

Encrypt every copy. For copies that leave your control, whether at another site, with another party or on media that are moved, encryption is not optional, and it is done on your side, before the data leaves you, with keys you hold yourself. Encryption at rest with the provider's keys protects neither against a compromised account nor against the provider.

The key becomes a single point of failure for every copy. It must never live only inside the backup it unlocks, since then you need the copy to open the copy, and never only on the system that can die. Keep it in at least two places that do not share a fate, ideally including a sealed paper copy somewhere else.

The backup holds the data, the configuration and the secrets. Outside it sits what must come up with nothing from the system being restored: the restore runbook, the key, the tool to read the format and the access to the backup storage, stored separately and readable without it. If reading the copy needs an agent, a licence server or a console that died with the infrastructure or no longer exists, the copy is useless. The restore order must also have no cycles, as when the key sits in a secrets manager that needs a sign-in service which itself has to be restored first.

## The restore drill

Backups fail silently through the wrong path, missing files, a job that reports success after copying an empty folder, a format that needs software you no longer have or a damaged medium, and none of it shows until you restore.

- **Spot checks, often.** Restore a few random files and check the contents. It is cheap, so do it regularly.
- **A full restore, on a schedule.** Start from clean equipment with only the copy and what is kept outside the backup, and include the loss of a whole site. Time it and compare with the RTO. Not every company tests this, and I have seen it myself.
- **Integrity.** Verify the checksums, as described under data rot. A restored database should start and pass its own consistency check, and since a running database copied at file level can end up unusable, use its own consistent dump or snapshot.

After a break-in, rebuild the systems from configuration and code and restore only data from a point before the intrusion, since restoring whole system images restores the intruder as well.

Automate the schedule, the verification and the clean-up, because manual backups stop happening sooner or later. An alert on failure is not enough, since a job that never starts sends no error, so alert on the absence of a success of both the copy and the verification, and treat silence as a finding, as in [Who watches the watchers? Oversight in IT infrastructure](/en/blog/who-watches-the-watchers-oversight-in-it-infrastructure/).

## Checklist

- We have written down the RPO and the RTO, and for each event we must survive we know how many copies are left afterwards, with the margin we decided on.
- There are at least three copies, the original included, on at least two different kinds of storage, and at least one is at another site, outside the provider and contract the others belong to.
- No single event (a fault, an account, a machine or a person) can delete all the copies. Production cannot delete or overwrite them, and better still, it has no access to them at all. At least one copy is made directly from the source into a separate repository, and the repository is checked regularly.
- At least one copy is offline or immutable, enforced by the storage itself.
- We keep several versions, and the retention is longer than the time it takes to notice a problem.
- Every copy is verified against checksums regularly, and media are renewed before they age out.
- Every copy is encrypted with keys we hold, and the key is kept in at least two places that do not share a fate.
- The runbook, the key, the tool to read the format and the access to the storage are kept outside the backup and readable without it, and the restore order has no cycles.
- The copying and the verification run automatically, and we get an alarm when a successful run of either fails to arrive.
- We have restored from clean equipment, including the loss of a whole site, timed it against the RTO, and repeat it on a schedule.

## A story to end with

Someone once told me about a recovery they did for a woman whose husband had died. All the photos of him, from the holidays and trips the two of them took together, were on one hard drive, and the drive failed.

Getting the data back took a year and a half. It meant finding a donor drive, an identical drive to take parts from, and working to recover the data. They knew her, so they took no money for it.

They told me she cried when she got them back. All the photos of her husband, everything, recovered.
