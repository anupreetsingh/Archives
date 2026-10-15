Prepping for the OS Software Engineer, Evaluation role on Apple's AIML Experimentation team (Role 200688625-0836, Cupertino).

## Screen: Tuesday, Oct 20, 2026

| | |
|---|---|
| **When** | Tuesday, Oct 20, 2026, **10:10–11:00 AM PDT** (1:10–2:00 PM EDT) |
| **Interviewer** | **Rajan Ranga**, Engineering Manager (the hiring manager) |
| **Format** | Webex, 50 min, one interviewer. CoderPad for the design and coding part |
| **Recruiter** | Josh Richardson (AIML Evaluation, AFM), <joshua_richardson@apple.com> |
| **Coordinator** | Mari (sent the confirmation; contact her to reschedule or if no one joins) |

Day-of rules:

- Join **no earlier than 10 minutes** before the start (10:00 AM PDT / 1:00 PM EDT).
- If Webex fails, the interviewer will call my phone number on file.
- If no one joins within **5 minutes** of the start, contact Mari.
- Dress: whatever presents my best self in a work setting.
- No AI tools during the interview unless the interviewer says otherwise.

## Job Description

**OS Software Engineer, Evaluation** · Cupertino, California · Software and Services · Posted Oct 12, 2026

For the engineer that obsesses on how software can enable OS developers to evaluate and improve their features, there is no better place than Apple's Experimentation team. The Experimentation team builds an end-to-end platform spanning the OS, data, and server for all the Apple OSes, operating at a scale of 2.5+ billion devices. We are addressing key challenges to accelerate the adoption of AI and an experimentation culture mindset across all Apple products/services. We are seeking engineers that can push the boundaries of low-level operating system technologies while maintaining a customer-centric focus on the developer and user experience.

The mission of Evaluation is to measure, understand, and proactively improve all Apple features. We are looking for an engineer to build and extend the experimentation system for Apple to use across all our operating systems. You will be responsible for the on-device daemon and framework, that is at the heart of the experimentation platform. You will partner with feature teams across Apple to enable them to make a positive impact for the customers. You will ship code that runs on devices you use every day and powers a variety of Apple products. You will obsess over memory, power, performance in addition to the developer experience.

RESPONSIBILITIES

- Design and implement new features and APIs for the on-device Experimentation service that is deployed across multiple Apple operating systems
- Develop and maintain high-performance libraries that are easy to use by developers
- Participate in all phases of software development cycle: from problem definition and design, through implementation, testing, and debugging
- Diagnose and resolve issues across different layers of the software stack
- Write unit and integration tests for new and existing code
- Work with cross-functional teams to enable developer and product features
- An appreciation how an OS design impacts end-user experience

MINIMUM QUALIFICATIONS

- Working proficiency in Objective-C, Swift, C/C++, Kotlin, or similar languages
- Strong academic foundation in operating systems concepts, including concurrency, multi-threading, synchronization primitives (mutexes, locks, semaphores), inter-process communication (IPC), and memory management.
- Hands-on experience developing low-level or systems software through internships, academic capstones, coursework labs, or substantial open-source/personal projects (e.g., building a shell, multi-threaded server, user-space service, or operating system project).
- Excellent problem-solving, debugging, and communication skills, with a track record of collaborating effectively in team or group-project settings.
- Bachelor's degree in Computer Science, Computer Engineering, or a related technical discipline (or equivalent practical experience).

PREFERRED QUALIFICATIONS

- Prior software engineering internship experience in systems software, embedded platforms, or platform engineering.
- Exposure to Unix/POSIX APIs, event loops/run loops (libdispatch/GCD, kqueue, epoll), or system daemons/services (launchd, systemd).
- Experience in iOS, macOS, or similar operating systems
- An appreciation of how a platform design impacts end-user experience
- Experience in experimentation or A/B testing and AI
- Master's degree in Computer Science, Computer Engineering, or a related technical discipline (or equivalent practical experience).

PAY

Base pay range: **$129,300 – $225,300**, plus RSUs, ESPP (discounted stock), medical/dental, retirement, tuition reimbursement, and possibly a discretionary bonus and relocation.

**Logistics and To-Do (from the recruiter email)**

- [x] Reply with **8 timeslots across multiple dates**, 8am–6pm **Pacific** (convert from Eastern: 11am–9pm ET). Sent Oct 13.
- [x] Confirm the Oct 20 slot with the coordinator (Mari). Done Oct 14.
- [ ] Install and test **Webex** (audio + video), and get familiar with **CoderPad** (run a practice problem in it in C++ and Swift).
- [x] Make sure the **Apple ID** is up to date.
- [x] Read and approve the **Privacy Consent** (panel can't be scheduled without it).
- [x] Confirm the application was submitted directly on jobs.apple.com. Submitted Oct 12.
- **No AI tools during the interview** unless the interviewer says otherwise.

## Interview Process

### Step 1: Hiring Manager & System Design Screen (Webex, 50 min)

- **Part 1: Hiring manager conversation (Rajan Ranga).** Learn about the team, and they learn about my background. Example questions: *where/when have you done your best work*, *what capabilities have you picked up in the past*. No coding, but **expect in-depth technical detail about past work**, so every resume line has to survive three "why"s (see [Resume Deep Dive](#resume-deep-dive)).
- **Part 2: System design.** "Design/implement a system that provides the opportunity to apply your device and operating system expertise." Communicate throughout: think out loud, ask clarifying questions.

What the recruiter confirmed about Part 2 (Oct 14):

| My question | Answer |
|---|---|
| High-level architecture only, or also lower-level component and interface design? | **Both** |
| Will they give a prompt, or should I bring a design of my own? | **They give the prompt** ("he'll ask you to build XYZ system") |
| In CoderPad: APIs, data flow, and pseudocode, or also executable code? | **Both** |
| Device/OS-oriented, rather than a traditional distributed backend? | **Yes, device/OS-oriented** |

So: start at the architecture, then go down to component interfaces, and be ready to write real, runnable code for a piece of it.

### Step 2: Virtual Panel (4 × Webex, 45 min each)

| # | Session | What they said | What it means for prep |
|---|---|---|---|
| 1 | Coding: OS systems, APIs, memory/storage | User-space services, IPC, scheduling priorities, reducing CPU and memory across the OS | Daemons, IPC options, QoS/priority inversion, virtual memory, page cache, mmap, jetsam/OOM |
| 2 | Coding: DSA | Technically complex, real-world problem. Code structure, edge cases, failure conditions | LRU/TTL cache, rate limiter, bounded thread-safe queue, timer scheduler, interval merging, bucketing |
| 3 | System design: scalable system | Builds on the screen. Services used by other developers, works on devices with varying resources, resource tradeoffs, uptime and reliability | Same playbook as the screen, but with more depth on server fan-out, rollout/rollback, and device tiers |
| 4 | Collaboration & communication | Work across an org, balance simplicity vs reliability, surface issues early, keep stakeholders informed, take feedback from all levels | CARL stories (see [Probable Behavioural Questions](#probable-behavioural-questions)) |

### The 6 Technical Tips, as a Checklist for Every Answer

1. **Principle first, platform second.** Explain the portable OS concept, then how macOS/Linux implement it. "Focus on the OS you're most familiar with" means macOS + Linux for me.
2. **Name the resource cost of every choice:** CPU, memory, persistent storage. Also power and network, since this is on-device.
3. **APIs are the developer's UX:** easy to understand, hard to misuse, cheap to call.
4. **Scale across devices:** Watch → iPhone → Mac → Apple TV/HomePod → Vision Pro. Say what changes per tier.
5. **Follow-ups are collaborative, not attacks.** Answer, then check: "does that address what you were getting at?"
6. **They grade approach, communication, and depth, not a perfect answer.** State assumptions, trade-offs, and what I'd measure.

## Company Research

### The Team

- **Org:** AIML (Apple's AI/ML org), Experimentation team, under the **Evaluation** mission: *measure, understand, and proactively improve all Apple features*.
- **Platform:** end-to-end, spanning **OS, data, and server**, for all Apple OSes, **2.5B+ active devices**.
- **This role:** owns the **on-device daemon and framework**, the piece every feature team links against. It delivers configurations and treatment assignments, persists state, collects measurements, and supports fast rollout and rollback across the fleet.
- **Experimentation vs Evaluation:** *experimentation* is enabling controlled variants (A/B tests, feature configurations, algorithms, ML models). *Evaluation* is measuring whether a variant improves the target outcome **without regressing** latency, crashes, CPU, memory, storage, battery, or reliability (the [guardrail metrics](#experimentation-vocabulary)).
- **Who the users are:** mostly other Apple engineers, so API ergonomics, safety, low runtime overhead, and being hard to misuse matter as much as the daemon's internals.
- **Why AI is in the pitch:** AI features (models, prompts, thresholds) need to be shipped as variants, measured, and rolled back safely. That makes an experimentation platform the delivery and measurement pipeline for AI features.
- **Sibling postings:** the same team has senior roles open ("AIML - Sr OS Software Engineer, Evaluation", "Sr Lead Engineer, Evaluation", "Sr Data Engineer, Measurement"), so the team is actively growing. One senior posting describes a service that can **roll out / roll back experiments to billions of devices in a few hours**. That number sets the bar for rollback speed in a design answer.
- **Where this team sits:** the wider Evaluation org also covers backend/measurement and AI evaluation. The OS group is one layer of a larger platform, so in a design answer the server and analysis sides are other teams' systems with interfaces to mine.

### The Hiring Manager: Rajan Ranga

- **Engineering Manager**, joined in **March 2026** (per his LinkedIn), so about 7 months into leading the team by the screen.
- **Background:** mobile/embedded OS platforms and low-level systems.
- **Hiring sequence:**
  1. Apple posted a **Sr OS Engineering Manager, Evaluation** role to lead the on-device daemon/framework effort. He appears to be the person hired into it (March 2026).
  2. He posted the **Sr OS Software Engineer, Evaluation** role (200658427-0836) on May 28, 2026. It still looked open on Oct 15, so the two roles are probably being filled in parallel.
  3. He posted this **OS Software Engineer** role on Oct 12.

  He's building out the on-device team from scratch, and this may be one of his first hires.
- What that likely means for the screen:
  - His background is low-level OS work, so expect follow-ups to go deep (memory, IPC, scheduling, resource limits on small devices) rather than stay at the boxes-and-arrows level.
  - He's still setting the team's direction, so he'll want someone who ramps up fast and can own pieces of the daemon and framework without much hand-holding.
  - The senior role probably owns the overall architecture, and this role builds features and APIs alongside it. Pitch myself as someone who takes a well-scoped piece end to end and learns quickly from a senior engineer.
  - A new manager is also judging fit for the team he's building, so Part 1 (best work, capabilities picked up) carries real weight.
- Don't lead with any of this. Let it come out through questions (see [Followup Questions](#followup-questions), Q7 and Q8).

### triald: Apple's Experimentation Daemon, as Seen on My Mac

On macOS 26.4 there is a shipping daemon that matches the posting's "on-device daemon and framework" description:

- Binaries: `/usr/libexec/triald` (per-user **LaunchAgent**) and `/usr/libexec/triald_system` (system-wide **LaunchDaemon**).
- Frameworks (private): `Trial.framework` (client side), `TrialServer.framework` (daemon logic), `TrialProto.framework` (serialization, probably protobuf).
- Public reporting (Eclectic Light, 2022) describes Trial as running "experiments" and delivering **ML models/assets** for Siri, Photos analysis, Visual Look Up, and Live Text without an OS update. Its data lives in `~/Library/Trial` and can take substantial disk space.

Reading its launchd plists (`plutil -p /System/Library/LaunchAgents/com.apple.triald.plist`) shows the design principles this team cares about:

| What the plist says | What it means | Design lesson |
|---|---|---|
| `MachServices` (`com.apple.trial.status`, `...namespace-management`, `...cache-delete`, `com.apple.aps.triald`) | Launched **on demand** when a client connects over XPC; not running otherwise | Zero idle cost. `aps` is likely Apple Push (wake on new experiment); `cache-delete` is likely the OS asking it to purge assets when disk is low |
| `EnableTransactions` + `EnablePressuredExit` | launchd may kill it when it has no open XPC transactions, and **under memory pressure** while idle | Daemon must be killable at any moment, so state lives on disk, not in memory |
| `ProcessType = Adaptive` | Runs as Background, gets boosted to Interactive when a client is waiting on it over XPC | Priority follows the caller (priority donation), no hard-coded priority |
| `xpc.activity` `fetch-experiments`: `Interval 86400`, `RequireInexpensiveNetworkConnectivity`, `AllowBattery false`, `Priority Maintenance`, `RandomInitialDelay 3600` | Fetch once a day, only on Wi-Fi, only on power, at low priority, with up to 1 h of random jitter | **Jitter prevents a thundering herd across 2.5B devices**; the system scheduler (DAS) picks the cheapest moment |
| `...cellular`: `Interval 259200` | Cellular fetch only every 3 days | Tiered network policy |
| `setup-assistant-fetch`: `NetworkDownloadSize 2 GB`, `DataBudgeting` | Big asset download during device setup, within a data budget | Large ML assets are budgeted, not free |
| `post-upgrade` activities (`PostInstall`) | Re-runs after an OS update | Re-evaluate everything when the OS version changes |
| `RequiresClassC` on every activity | My reading: only runs after first unlock, when Class C (default) data-protection files are readable | A daemon can start before the user unlocks, so it must cope with unreadable files |
| `LaunchEvents` → `notifyd` matching: `language.changed`, `countryCodeChanged`, Apple account added/deleted, keyboard/Siri prefs, MDM profile changes, `crashreporter.auto_submit_preference_changed`, `com.apple.triald.new-experiment` | Woken whenever a **targeting input** changes, including the **analytics-sharing consent toggle** | Re-evaluate eligibility when inputs change, and honor consent immediately |

**How to use this in the interview:** say "I was curious how Apple's own daemon is set up, so I read triald's launchd plist on my Mac." It shows curiosity. Don't claim to know its internals or that this team owns it; ask instead (see [Followup Questions](#followup-questions)).

### Summary

**Mental model for the role:** build infrastructure that lets Apple teams experiment safely on billions of very different devices while using almost no noticeable CPU, memory, storage, battery, or network.

Apple's Experimentation team runs the system that lets any Apple feature ship variants (including AI models) to a slice of 2.5B devices, measure the effect, and roll forward or back safely. This role owns the on-device half: a daemon that fetches and evaluates experiment configs under strict memory, power, network, and privacy limits, and the framework feature teams call to read their values. My On-Device Experimentation Service project is a small version of exactly that, and my Turfco A/B testing and Platinum RAG evaluation work cover the "why measure" side.

## Concepts to Speak Fluently

### Experimentation Vocabulary

- **Experiment vs rollout vs holdout.** An experiment compares arms (control vs treatments). A rollout ramps one change to 100% (staged %, with kill switch). A holdout keeps a small group on the old behavior long-term to measure cumulative impact.
- **Randomization unit.** Device, user/account, or session. Must match how the effect is felt. A per-request split lets one person see both prices; pick a unit that keeps each person in one arm.
- **Deterministic bucketing.** `bucket = hash(salt ‖ experiment_id ‖ unit_id) mod 10000`. Same inputs → same arm every time, with no server call and no stored assignment.
  - **Per-experiment salt** keeps assignments independent across experiments. Reusing a salt means the same devices always land in treatment together.
  - **Ramping** 1% → 10% → 50% by widening the bucket range keeps already-treated devices in treatment.
  - Use a **stable, specified hash** (e.g., MurmurHash3, xxHash, SHA-256 truncated), never `std::hash`, which differs across standard library implementations.
- **Layers / mutual exclusion.** Experiments touching the same surface share a layer so a device is in at most one of them.
- **Targeting / eligibility.** OS version, device class, region, locale, hardware capability (e.g., Apple Intelligence-capable), opted-in analytics. Re-evaluate when these change.
- **Exposure (trigger) logging.** Log when the feature *actually reads* the value, not at assignment. Counting everyone assigned dilutes the effect with people who never saw the feature.
- **Sample Ratio Mismatch (SRM).** If a 50/50 split reports 52/48, something is broken, usually logging that fails more often in one arm (e.g., the treatment crashes before its exposure is flushed). Check with a chi-square test.
- **Guardrail metrics.** Crash rate, hangs, launch time, memory footprint, battery drain. A winning metric doesn't ship if a guardrail regresses.
- **Stats basics.** Significance (α = 0.05), power (80%), MDE, sample size ≈ 16σ²/δ² per arm. Peeking inflates false positives, which is why sequential testing exists. Novelty effects mean early lifts fade.
- **AI-specific evaluation.** Model/prompt/threshold variants delivered as assets; offline eval set first (like my 150-control set), then online A/B; quality signals (task success, user corrections), latency, model load time, memory, and thermal impact.
- **Privacy (Apple's version).** Compute assignment **on device** so the server never needs a per-device ID. Upload **aggregated** counts rather than raw event streams. Respect the analytics-sharing consent. Consider rotating identifiers and differential privacy.

### Apple OS Vocabulary (with the Linux Equivalent)

| Concept | Apple | Linux | One-line explanation |
|---|---|---|---|
| Service manager | **launchd**: LaunchDaemons (system) vs LaunchAgents (per-user) | **systemd**: system vs `--user` units | Starts, restarts, and on-demand-launches services |
| On-demand launch | `MachServices`, `Sockets` (+ `launch_activate_socket`), `LaunchEvents` | `.socket` units (+ `sd_listen_fds`), `.path`, `.timer` | Service runs only when someone needs it, so idle cost is zero |
| Idle exit | `EnableTransactions`, `EnablePressuredExit`, `xpc_transaction_begin/end` | Exit-on-idle + socket activation | The OS may kill an idle daemon to reclaim memory |
| IPC | **XPC** (`xpc_connection`, `NSXPCConnection`), Mach ports, UDS, pipes, shared memory | D-Bus, UDS, pipes, shared memory | XPC adds launchd lifecycle, typed messages, and **peer entitlement checks via audit token** |
| Broadcast signal | Darwin notifications (`notify_post`, `notify_register_dispatch`), no payload | D-Bus signals, `inotify` on a file | Cheap "something changed, go re-read" |
| Event loop | **GCD** queues + dispatch sources, CFRunLoop, **kqueue** | **epoll**, `timerfd`, `signalfd`, `eventfd` | kqueue watches fds, timers, signals, processes, and file changes in one API; epoll needs an fd for each |
| Priorities | **QoS classes**: user-interactive, user-initiated, default, utility, background | nice, cgroups cpu weight, `SCHED_*` | Scheduler + I/O + timer coalescing all follow QoS |
| Priority inversion | `os_unfair_lock`, `pthread_mutex`, and XPC replies **donate priority**; `DispatchSemaphore` **does not** | PI mutexes (`PTHREAD_PRIO_INHERIT`) | A semaphore doesn't know who will signal it, so a high-QoS waiter can be starved by a low-QoS worker |
| Deferred background work | **DAS** (`dasd`) via `xpc_activity`, `BGTaskScheduler`, `NSBackgroundActivityScheduler` | systemd timers, cron | The OS chooses *when* to run, based on power, network, thermal state, and usage |
| Memory limits | **jetsam** (iOS has no swap; kills by priority band at a per-process limit), memory compression | **OOM killer**, cgroup `memory.max`, swap | On iOS, exceeding the limit means being killed, so design for a fixed budget |
| Memory accounting | **footprint** = dirty + compressed. **Clean** (file-backed, unmodified) pages don't count; the OS can drop and reload them | RSS, PSS, cgroup memory includes page cache | mmap'd read-only data is clean memory and shared across processes, so it's cheap |
| File protection | Data Protection classes A/B/**C (default: available after first unlock)**/D | (none built in) | A daemon running before first unlock can't read Class C files |
| Atomic file swap | write temp → `fsync` → `rename()` (same volume) | same | Readers see the old or the new file, never half of one |
| Low-disk purge | CacheDelete (purgeable content) | (app-specific) | OS asks services to free space under storage pressure |
| Observability | `os_log` / unified logging, `os_signpost` + **Instruments**, `sysdiagnose`, `footprint`, `vmmap`, `leaks`, `powermetrics`, `spindump` | journald, `perf`, `bpftrace`, `/proc`, `valgrind` | Measure before optimizing; signposts show intervals in Instruments |

## Prep Areas and Sources

### JD Phrase → My Evidence

| JD asks for | My evidence |
|---|---|
| On-device daemon + framework | **On-Device Experimentation Service**: C++ daemon + C library + Swift wrapper |
| launchd / systemd | Socket-activated on both; Role Radar's launchd agent with restart and SIGTERM drain |
| kqueue / epoll / GCD | Single non-blocking kqueue/epoll loop; Role Radar reads pipes on GCD queues |
| Concurrency, mutexes, semaphores | Mutex-guarded ring buffer + writer thread; global and per-source semaphores; atomic-counter thread pool; GA assignments on mutexes, semaphores, and lock-free CAS |
| IPC | Unix domain sockets, mmap'd shared snapshot, pipes (JSON CLI), GA assignments on sockets and pipes |
| Memory management | Page-cache OOM fix with `posix_fadvise`; mmap snapshot; under 2 MB RSS; GA page-table and page-fault assignments |
| Memory, power, performance obsession | 90 ns vs 30 µs lookups; 12× fewer wakeups via a coalesced timer |
| APIs that are easy to use | One-call `variant(for:)`; C ABI + Swift wrapper |
| Diagnose across layers of the stack | Platinum OOM traced across app → container → OS |
| Unit and integration tests | GoogleTest + ThreadSanitizer; 380+ tests (Role Radar); 200+ tests (GA) |
| Experimentation / A/B and AI | Turfco hashed 50/50 price-rule A/B tests; Platinum 150-control eval set, 64% → 92% |
| Collaboration | Compliance analysts (Platinum), business owners (Turfco), 10 student Scrum teams (GA) |

### Gaps to Close (be honest, show a plan)

- **Objective-C:** not on the resume, and many Apple daemons and frameworks are ObjC. Learn the basics: message sending, ARC, blocks, protocols, `NSXPCConnection` with an `@protocol` interface. Be able to *read* ObjC fluently.
- **XPC:** my project uses Unix domain sockets for portability. Know how the same design looks with XPC (see Resume Deep Dive).
- **Instruments / os_signpost:** be able to say how I'd profile memory (Allocations, Leaks, VM Tracker), CPU (Time Profiler), and power (Energy Log, `powermetrics`).
- **Production Apple-platform experience:** I have personal projects only. Lean on depth of understanding plus the triald plist reading.

### Screen Priority List (in order)

1. Elevator pitch out loud, under 90 seconds, then 2 minutes per resume project (directional flow: where it starts → what it passes through → where it ends).
2. Resume Deep Dive traps below. Especially the **clock**, **page cache OOM**, **snapshot reclamation**, and **exposures with a lock-free fast path**.
3. One full mock of the [System Design Playbook](#system-design-playbook-screen-part-2) example, timed to 25 minutes, talking out loud.
4. Experimentation vocabulary until it's natural (exposure, SRM, salt, ramp, holdout, guardrails).
5. Apple vocabulary table above.
6. 3 followup questions picked and memorized.

### Panel Topics (after the screen)

- **Session 1 (OS coding):** process vs thread; scheduling and QoS; priority inversion and donation; IPC options and when to use each; virtual memory, paging, copy-on-write, page cache, `mmap`; dirty vs clean memory; jetsam / OOM killer; `fsync` and atomic rename; SQLite WAL; flash wear and write amplification; reducing CPU and memory (lazy init, on-demand launch, coalescing, batching, no polling, idle exit). Existing notes: [Mutexes and Locks](<../../DSA/C++/Concurrency/Mutexes and Locks.md>), [Concurrency Models](<../../Software Engineering/Concurrency Models.md>), [Stack and Heap](<../../DSA/C++/Memory/Stack and Heap.md>), [Computer Architecture](<../../Software Engineering/Computer Architecture.md>), [Undefined Behavior and Sanitizers](<../../DSA/C++/Memory/Undefined Behavior and Sanitizers.md>). Still to write: condition variables, semaphores, atomics and memory ordering, IPC, virtual memory.
- **Session 2 (DSA, real-world):** LRU cache (+ TTL), token-bucket rate limiter, thread-safe bounded queue / ring buffer, timer scheduler (min-heap), merge intervals (e.g., coalescing wakeups), top-K, trie, topological sort (service dependency order), percentage bucketing. Code in **C++** (or Swift). Always cover: empty input, overflow, capacity full, concurrent callers, failure paths. Notes: [Heuristics](<../../DSA/Heuristics.md>), [STL Containers](<../../DSA/C++/Standard Library/STL Containers.md>).
- **Session 3 (scalable design):** the playbook below, plus server fan-out (CDN for configs), staged rollout and rollback in hours, device tiers, SLOs. Notes: [System Design Basics](<../../System Design/System Design Basics.md>), [Caching](<../../System Design/Caching.md>), [API Design](<../../Software Engineering/API Design.md>), [Observability and Monitoring](<../../Software Engineering/Observability and Monitoring.md>).
- **Session 4 (collaboration):** story bank below, CARL format from [General Pointers](<../General Pointers.md>).

## System Design Playbook (Screen Part 2)

### How to Run an On-Device Design (~25 min)

1. **Clarify (3 min).** Who calls it (feature teams? which processes)? Which devices? Read-heavy or write-heavy? Latency budget for the caller? Freshness needed? Privacy constraints? What happens offline?
2. **API first (3 min).** What does the calling developer write? This is the "user experience" they explicitly grade.
3. **Directional flow (5 min).** Draw the path from source to device to caller and back. Name every hop.
4. **Components in flow order (8 min).** For each one: process boundary, threading, storage, IPC.
5. **Resource budget (2 min).** CPU, memory, storage, power (wakeups), network. Give rough numbers.
6. **Failure modes (3 min).** Crash, corrupt data, bad config, no network, low disk, memory pressure, clock skew, OS upgrade.
7. **Scale across devices (1 min).** What changes on Watch vs Mac vs Apple TV.

### Worked Example: Design an On-Device Experimentation Service

The most likely prompt, given the team. Even if the prompt is different (feature flags, model delivery, telemetry), most of this answer still applies.

**Clarifying questions to ask:** How many experiments active per device (tens? thousands)? Do values change mid-session, or stay stable until next launch? Are assets (ML models) involved, or only small values? Is assignment per device or per account? Which devices? Is there a consent requirement for logging?

**The flow, start → end:**

```text
Feature team defines experiment on server (targeting, arms, %, salt)
        │
        ▼
Server publishes ONE signed config blob for everyone ──► CDN
        │                                      (no per-device state on the server)
        ▼
Device: push (APNs) or daily DAS fetch wakes daemon (Wi-Fi + power + jitter)
        │
        ▼
Daemon verifies signature → evaluates targeting → computes arm locally
   bucket = hash(salt ‖ exp_id ‖ device_random_id) mod 10000
        │
        ▼
Daemon writes snapshot: temp file → fsync → rename()  ──► notify_post("updated")
        │
        ▼
Feature process: framework mmaps snapshot (read-only, lock-free)
   value(for:default:) → typed value, never blocks, never throws
        │
        ▼
On first real use → exposure recorded (buffered in-process, batched to daemon)
        │
        ▼
Daemon aggregates counts per experiment/arm/day → uploads when cheap (consent-gated)
        │
        ▼
Server analysis: metrics, SRM check, guardrails → ship / ramp / roll back
        │
        └──► rollback = new config (kill switch) pushed → devices converge in hours
```

**Components, in flow order:**

1. **Config source (server, brief).** Since assignment happens on device, every device gets the *same* signed blob. It's cacheable on a CDN, and the server never learns who is in which arm. Rough load: 2.5B devices fetching once a day ≈ 29k req/s average. **Random jitter** keeps it near the average instead of spiking at midnight.
2. **Daemon lifecycle.** Launched on demand by launchd (XPC `MachServices` + `LaunchEvents`), never polling. Exits when idle. It must tolerate being killed at any time, so all state is on disk. Separate per-user agent and system daemon on multi-user Macs, the same way triald / triald_system are split.
3. **Fetch.** Scheduled through DAS/`xpc_activity`: Wi-Fi + power for big assets; small configs and **kill switches allowed on cellular**. A push (APNs) can trigger an urgent fetch for rollbacks. Conditional GET (ETag) so unchanged configs cost almost nothing.
4. **Evaluate.** Verify signature → check schema version (ignore unknown fields so older OS versions keep working) → evaluate targeting (OS version, device class, locale, region, consent) → hash to bucket → resolve values. Do the work **once per config change**, not on every read. Re-evaluate on targeting-input changes (locale, country, account, consent), the way triald's `LaunchEvents` subscribe to them.
5. **Publish snapshot.** A flat, versioned, read-only file: header (version, checksum) + a sorted table or perfect hash of `namespace.key → typed value`. Atomic `rename()` swap. A Darwin notification tells clients to remap. Keep the **last-known-good** copy.
6. **Client framework (in the feature's process).** mmap the snapshot read-only, so it's clean memory shared across all processes. Lookup = hash probe, ~100 ns, no IPC. **Pin a snapshot per session** so a feature doesn't flip arms mid-use. Swap snapshots through an atomic pointer, and only unmap the old one at a safe point.
7. **Exposure logging.** The framework records "experiment X, arm B, exposed" once per session per experiment. It buffers in a small fixed-size ring and sends batches to the daemon asynchronously. Bounded memory, with a counted drop policy when full.
8. **Upload.** Daemon aggregates to counts (not raw events), stores in a capped on-disk queue, and uploads in a deferred, discretionary background task. Only with analytics consent.
9. **Analysis + decision (server, brief).** SRM check, metric deltas, guardrails. A bad experiment is stopped by publishing a new config.

**API sketch** (what a feature team writes):

```swift
// Typed keys generated from the experiment definition, so typos fail at compile time.
let ranking = Experiments.photosSearch.rankingModel.value(default: .v1)
// - Never blocks, never throws, never does IPC on the hot path.
// - The default is required at the call site: no snapshot, corrupt file, or ineligible
//   device all return it, so the feature always works.
// - Exposure is logged automatically on first read per session (opt-out for prefetching).
```

```c
// C ABI for C/C++/ObjC daemons that can't link Swift; same semantics.
double exp_get_double(const char *ns, const char *key, double default_value);
```

**Hard to misuse:** required defaults, typed values, stable per-session values, no async on the hot path, no mutable global state exposed. **Cheap to call:** hash lookup in shared clean memory.

**Resource budget:**

| Resource | Cost | Why |
|---|---|---|
| Memory | Snapshot ~10s of KB to a few MB, **clean and shared** across processes; per-client overhead a few KB; daemon 0 MB when idle | mmap read-only; daemon exits when idle |
| CPU | Hashing and targeting once per config change; lookups ~100 ns | Precompute in daemon, not at read time |
| Power | ≤ 1 scheduled wakeup/day + rare pushes; exposures batched | DAS chooses when; coalesced timers; no polling |
| Storage | Configs small; ML assets capped and purgeable on low disk | CacheDelete-style purge; last-known-good only |
| Network | Configs: KB, any network. Assets: MB–GB, Wi-Fi + power only | Tiered policy, data budgeting |

**Failure modes:**

- **Corrupt or partial snapshot** → checksum fails → use last-known-good → else compiled-in defaults.
- **Daemon crashes** → clients keep reading the existing mmap; launchd relaunches on next demand.
- **Bad experiment crashes the feature** → server kill switch via push. On device, a **crash-loop guard** reverts to defaults if the process crashes N times right after a new config.
- **No network for weeks** → keep the last config. Experiments carry an end date, so stale ones expire to defaults.
- **Device clock wrong** → don't trust the local clock for start/end alone; use server-signed timestamps plus tolerance.
- **Thundering herd** → random initial delay, staged % ramp.
- **OS upgrade or downgrade** → schema versioning, re-evaluate post-upgrade, unknown fields ignored.
- **Before first unlock** → config files can't be read under Class C → use defaults until unlock, or keep a non-sensitive copy in a less protected class.
- **Exposure loss that correlates with an arm** (e.g., the treatment crashes before flushing) → biased results → caught by SRM checks.

**Scaling across devices:**

- **Apple Watch:** tight memory and battery, jetsam limits. Smaller snapshot (only experiments targeting watchOS); fetch through the paired iPhone instead of its own radio; fewer exposures, aggregated harder.
- **iPhone:** the baseline. No swap, jetsam, cellular costs.
- **Mac:** multiple users → per-user agent + system daemon; more memory but the same principles.
- **Apple TV / HomePod:** always on power and Wi-Fi → can fetch more often. Few or no user-facing processes.
- **Vision Pro:** thermal limits; heavy ML assets.
- The same daemon code with **per-platform policy** (budgets, intervals, asset caps), not separate forks.

**Privacy:**

- Random per-install ID for bucketing (not hardware UUID or account ID); never leaves the device.
- Uploads are aggregated counts tied to consent.
- Re-check consent on the notification (triald listens for the crash-reporter auto-submit preference change).

### Other Likely Prompts (and the angle each tests)

- **On-device telemetry / logging daemon:** batching, bounded buffers, backpressure, drop policy, consent, upload scheduling, privacy aggregation.
- **ML model / asset delivery service:** large downloads (resume, integrity check, delta updates), disk caps and purge, version compatibility with OS, atomic activation, rollback.
- **Feature flag / kill switch:** a subset of the experiment design. Focus on propagation speed and safe defaults.
- **Background job scheduler (like DAS):** priority queue keyed on deadline and conditions; power, network, and thermal gating; coalescing.
- **Rate limiter for a system service:** token bucket per client, identify client by audit token, fairness, priority.
- **Shared config cache across processes:** mmap + atomic swap, invalidation through notifications, consistency.
- **Cross-device sync (iPhone ↔ Watch):** conflict resolution, battery, intermittent connectivity.

## Elevator Pitch

*(Rehearse out loud. Target 60–90 seconds.)*

Hi, I'm Anupreet. I recently finished my master's in computer science at UMBC, where I was also a graduate assistant for Operating Systems and designed the C++ assignments on threads, synchronization, IPC, and page tables. Systems work is where I do my best work. At Platinum Business Services, I traced out-of-memory failures in our model-loading service through the app, container, and OS layers down to multi-gigabyte model files sitting in the page cache, and fixed them with targeted cache eviction. On the same team, I built the evaluation set and a multi-model fallback for a RAG compliance agent, which took its accuracy from 64 to 92 percent, so I've seen firsthand how much a good evaluation loop changes what you ship. Before that, at Turfco, I ran A/B tests on pricing rules from the server side. To see what assignment takes on the device itself, I built an on-device experimentation daemon and client library: socket-activated under launchd and systemd, one kqueue/epoll event loop, and an mmap'd snapshot that answers lookups in about 90 nanoseconds without a round trip. That's why this role stands out to me. It's the same problem at 2.5 billion devices, where every byte and every wakeup matters.

> Only keep the "to see what assignment takes on the device" bridge if that's genuinely why the project was built. If not, replace it with the real reason. They will ask.

## Followup Questions

*(Pick 3 for the screen. Save the rest for the panel.)*

1. The posting says this role owns the on-device daemon and framework. How is ownership split between the on-device side, the server/config side, and the data/analysis side, and where do the hand-offs between them tend to cause the most friction?

2. I was curious how Apple's own daemon is set up, so I read triald's launchd plist on my Mac. It runs one daily Wi-Fi fetch with random jitter, a separate per-user agent and system daemon, and it wakes when the locale or analytics consent changes. Is that the system this team works on? If so, what's the hardest constraint today: memory on smaller devices, propagation speed for rollbacks, or something else?

3. The posting mentions accelerating the adoption of AI. Are AI features evaluated differently from classic A/B tests? For example, are you delivering model variants as assets and measuring quality on device? And how does Apple's privacy stance shape what can be measured?

4. How does the daemon's behavior differ across device classes? Does an Apple Watch run the same daemon with a tighter budget, or does it lean on the paired iPhone?

5. What's the most common way feature teams misuse the current framework API, and how does the team find out about it?

6. For someone joining in this role, what would you expect them to own in the first 6 months, and what would success look like?

7. You joined the team earlier this year. What's changed since then, and what are you focused on building over the next year?

8. I saw the team is also hiring a senior OS engineer. How do you see the two roles dividing the daemon and framework work?

## Resume Deep Dive

For each item: explain it as a **flow** first (start → middle → end), then expect the probes. Items marked **⚠ Trap** are the follow-ups most likely to catch me off guard. Know my actual answer before the screen.

### On-Device Experimentation Service

**The flow:** client process calls `variant(for:)` → C library checks the mmap'd snapshot (fast path, ~90 ns) → if the snapshot is missing, falls back to the socket path: launchd/systemd starts the daemon on the first connection → daemon's kqueue/epoll loop accepts on the Unix domain socket → computes the arm from `hash(salt, experiment, device_id)` → replies. Config pushes rewrite the snapshot via temp file + `rename`. Exposures go into a mutex-guarded ring buffer that a writer thread flushes on a coalesced 60 s timer.

- **Why socket activation?** The daemon costs nothing until the first client connects. launchd/systemd own the listening socket, so connections made while the daemon restarts queue up instead of failing. APIs: `launch_activate_socket()` (macOS), `sd_listen_fds()` (Linux).
- **Why Unix domain sockets and not XPC?** Portability across macOS and Linux. On Apple-only, **I'd use XPC**: launchd lifecycle built in, typed messages, automatic priority donation, and **peer entitlement checks through the audit token** (UDS only gives uid/gid via `getpeereid` / `SO_PEERCRED`).
- **Why one event-loop thread?** Each request is a tiny CPU-bound lookup, so one thread avoids locks and keeps the footprint small. Anything blocking (disk flush) moves to another thread, which is exactly what the ring-buffer bullet is about.
- **kqueue vs epoll:** kqueue registers changes and waits in one syscall, and watches timers, signals, processes, and file changes natively (`EVFILT_TIMER`, `EVFILT_SIGNAL`, `EVFILT_VNODE`). epoll only watches fds, so timers and signals need `timerfd` / `signalfd`. Know whether I used level- or edge-triggered (`EPOLLET` / `EV_CLEAR`) and why.
- **Salted hash:** the salt keeps arms independent across experiments. **⚠ Trap:** which hash did I use? If `std::hash`, admit it isn't stable across standard libraries and say I'd switch to a specified hash (Murmur/xxHash/SHA-256). **Where does the device ID come from** (`/etc/machine-id`, IOPlatformUUID, a generated UUID on disk)? At Apple, a hardware ID would be a privacy problem; a random per-install ID is better.
- **Atomic snapshot swap:** write temp in the same directory → `fsync` → `rename()` over the old path. Same filesystem → atomic. Processes that already mapped the old file keep the old inode alive, so they see a consistent old snapshot.
  - **⚠ Trap:** *How does a client learn a new snapshot exists?* (Notification? `stat()` checking `st_ino`? A generation counter?) Know exactly what my code does and its cost per lookup.
  - **⚠ Trap:** *When is it safe to `munmap` the old snapshot if another thread may still be reading it?* Options: never unmap during a session (pin per session), reference counting, or epoch-based reclamation. "Lock-free" also needs an `std::atomic` pointer for the current mapping.
- **⚠ Trap: exposures with a lock-free fast path.** If lookups never touch the daemon, how do exposures reach it? Know the answer (async fire-and-forget message? batched per process?).
- **90 ns vs 30 µs:** how measured (benchmark loop, warm cache, median vs p99)? The 30 µs is two syscalls + two context switches + waking the daemon. First lookup after mapping is slower (page fault).
- **Ring buffer + writer thread:** why a mutex and not a lock-free SPSC queue? Simpler, provably correct, uncontended in practice, tiny critical section. What happens when the buffer is full? Drop oldest or newest, and **count drops**. What if the device dies before the flush? Up to 60 s lost. That's fine if loss is random across arms, and harmful if one arm crashes more (SRM).
- **12× fewer wakeups / 2 MB RSS:** from what to what, and measured how (`powermetrics`, `top -stats idlew`, `/proc/<pid>/status` context switches)? Coalescing = timer leeway (`DispatchSource` leeway, timer slack on Linux), which lets the OS batch wakeups across processes.
- **ThreadSanitizer:** did it catch a real race? Have one specific example ready.
- **Why a C library with a Swift wrapper?** A stable C ABI is callable from C, C++, ObjC, and Swift. The Swift layer adds types and ergonomics. Apple's own frameworks follow a similar pattern (C/ObjC core + Swift overlay).

### Role Radar

**The flow:** SwiftUI app → installs and supervises the checker as a launchd agent → checker runs checks, bounded by semaphores → app reads checker state by running its JSON CLI over pipes on GCD queues → when the Mac sleeps, a Lambda takes over through a DynamoDB lease.

- **launchd supervision:** `KeepAlive` restarts it; on stop, launchd sends SIGTERM, then SIGKILL after `ExitTimeOut` (I set 60 s). Handle SIGTERM with a dispatch source / kqueue `EVFILT_SIGNAL`, not work inside a signal handler (only async-signal-safe calls are allowed there).
- **⚠ Trap: pipes.** If the app waits for the child to exit before reading stdout, and the output exceeds the pipe buffer (~64 KB), both block forever. Reading stdout and stderr concurrently on GCD queues avoids that. Be ready to say this is why.
- **flock PID file:** the kernel releases the lock on *any* exit, including crashes, so there's no stale PID file to clean up. `flock` locks belong to the open file description; `fcntl` locks are per-process and drop when *any* fd to the file closes, which is a classic bug. Windows: `LockFileEx` on a byte.
- **Semaphores:** global cap + per-source cap = overall concurrency limit + politeness per site. **⚠ Apple trap:** `DispatchSemaphore` doesn't donate priority, so it can cause priority inversion. Don't block Swift concurrency's cooperative threads with semaphores either.
- **DynamoDB lease + fencing token:** acquire with a conditional write (`expiresAt < now OR holder = me`), incrementing `epoch` each time it changes hands. Every save is conditioned on `epoch = myEpoch`, so a stale holder that wakes from sleep can't overwrite the new owner's data.
- **⚠ Trap: the clock wording.** The resume says "macOS's monotonic clock stops during sleep." On macOS, **`CLOCK_MONOTONIC` keeps counting during sleep** (verified in `man clock_gettime`, macOS 26.4). What stops is `mach_absolute_time()` / `CLOCK_UPTIME_RAW`, which `DispatchTime` and Swift's `SuspendingClock` use. `mach_continuous_time()` and `ContinuousClock` keep counting. Say it precisely: *"the uptime clock that DispatchTime uses stops during sleep."* The stronger point: a lease checked by **two machines** has to use wall-clock time anyway, because monotonic clocks are per-machine and per-boot. Clock skew between machines is exactly what the fencing token protects against. Consider fixing the resume line.
- **8 min failover:** where does it come from (lease TTL + Lambda schedule interval)? Know the breakdown.

### Rendering Pipelines from Scratch

- **Thread pool claiming row chunks from an atomic counter** = dynamic load balancing. Rows vary in cost (reflective objects), so static splits leave threads idle. Watch for **false sharing** if per-thread counters sit on one cache line.
- **BVH median split:** per-ray cost goes from O(n) to roughly O(log n).
- **⚠ Trap:** decompose the **759×**: how much from BVH, how much from threads (how many cores?), how much from compiler flags (`-O2` vs debug)?

### Data AI Engineer, Platinum Business Services

**The flow:** model-load request → app reads the 4–5 GB model file → container memory limit hit → OOM → I traced through app logs → container metrics → OS (`/proc/meminfo`, cgroup `memory.stat`) → found the model files occupying page cache → fix: `posix_fadvise(POSIX_FADV_DONTNEED)` on a worker thread after load.

- **⚠ Trap: "page cache is reclaimable, so why did it OOM?"** This is the first thing an OS interviewer will ask. Know which of these was true in my case:
  - In a container, **page cache is charged to the cgroup's memory limit**. Reading a model with `read()` into the heap makes **two copies**: page cache + anonymous heap (~8–10 GB charged for a 4–5 GB model).
  - Reclaim can fall behind a fast, large anonymous allocation, especially if the pages are on the active list.
  - Or the orchestrator / a health check counted cache as "used" (e.g., a Kubernetes working-set metric) and killed or refused the load before the kernel reclaimed anything.
- **Why a worker thread?** `fadvise` on gigabytes can take time; keep it off the request path.
- **What I'd also consider:** `mmap` the model so the page cache *is* the model memory (one copy, shared between workers); `O_DIRECT`. **On macOS** there is no `posix_fadvise`. The equivalent is `fcntl(fd, F_NOCACHE, 1)`.
- **RAG eval set + fallback:** this is the **Evaluation** story. Built ground truth with domain experts (150 controls), benchmarked 5 embedding models, then designed a fallback that re-runs low-citation queries on the next model's index. Know how "accuracy" was defined, and how the fallback order was chosen.

### Software Development Engineer, Turfco

- **A/B tests on price rules:** **⚠ Trap: randomization unit.** "Hashed 50/50 quote split": was it per quote or per customer? Per quote means one customer could see two prices for similar jobs. How long did it run, how many quotes, and how was significance decided? What did the owners change based on it?
- **Kafka retained topic:** why retention (replay to rebuild analytics, join to bookings later)? How was the quote → booking join done?
- **Write-invalidated Redis cache:** invalidate on rule edits rather than a TTL, so prices are never stale. Know the race: a read that refills the cache between the DB write and the invalidation.

### Graduate Assistant, UMBC

- Be ready to **teach** any of my own assignments in 2 minutes: e.g., "implement a bounded buffer with mutex + condition variables, then with semaphores, then lock-free with CAS". This is a strong signal for Session 1.
- Mentoring 10 Scrum teams → collaboration and feedback stories.

## Probable Behavioural Questions

*(CARL: Context → Action → Results → Learning. Fill in the bracketed parts with real details.)*

Q1. Tell me about yourself.
A1. Elevator pitch above.

Q2. Where or when have you done your best work?
A2. Candidate: the **Platinum OOM investigation**. It matches "diagnose issues across layers of the stack" exactly.

- **Context:** model loads failing with OOM in production; [who was affected, how often].
- **Action:** ruled out the app → checked container limits → went down to OS memory accounting → found 4–5 GB model files lingering in page cache → fixed with `posix_fadvise` eviction on a worker thread → [how I verified it].
- **Results:** failures eliminated; [any numbers].
- **Learning:** "peel away layers of abstraction until reaching familiarity" (from my About me). The bug wasn't in the layer where the symptom appeared.

Q3. What capabilities have you picked up in the past, and how?
A3. Show a pattern of learning by building, then teaching:

- OS internals: deepened by *designing* OS assignments as a GA.
- Apple platform (launchd, GCD, SwiftUI, kqueue): learned by building Role Radar and the experimentation service.
- Evaluation methodology: learned at Platinum by building a labeled eval set with analysts before changing the system.
- Close with how I'd pick up ObjC / XPC / Instruments here.

Q4. Why Apple, and why this team?
A4. (1) It's the problem I already chose to build on my own: on-device assignment without a server round trip. (2) At 2.5B devices, memory, power, and wakeup decisions actually matter. (3) The Evaluation mission ("measure before you change") is what made the difference at Platinum (64% → 92%). (4) Apple's privacy stance makes it harder and more interesting: assignment on device, aggregated reporting.

Q5. Why did you build an on-device experimentation service?
A5. [Fill in the honest reason.] Be ready for: what I'd build next on it, and what I'd change if it had to run on an Apple Watch.

Q6. Tell me about a time you identified a problem early. (Session 4)
A6. Candidate: **Role Radar fencing token**. I foresaw that a Mac waking from sleep could still believe it held the lease while Lambda had taken over (split brain), and designed the epoch fencing token before it caused duplicate or overwritten data. [Fill in: how I noticed, how I tested it.]

Q7. How do you keep stakeholders informed? (Session 4)
A7. Candidates: **Platinum compliance analysts** (partnered on the eval set; how did I report progress and results?) or **Turfco owners** (the audit trail and A/B data let them see *why* a price was what it was). [Fill in specifics.]

Q8. Tell me about a time you incorporated feedback. (Session 4: "from all levels")
A8. Candidates: student feedback on assignments as a GA (feedback from "below"), code review or manager feedback at Platinum/Turfco (from "above"). [Fill in one real example: what feedback, what I changed, outcome.]

Q9. How do you balance simplicity with reliability? (Session 4)
A9. Candidates: **flock PID file**, which is simpler *and* more reliable than stale-PID cleanup logic because the kernel releases the lock; **mutex ring buffer over lock-free**, which is simpler, correct, and fast enough since contention is low. Theme: pick the simplest mechanism whose failure mode the OS handles for me.

Q10. Tell me about a disagreement with a teammate or manager.
A10. [Fill in.] Show: disagreed with data, listened, committed once decided.

Q11. Tell me about a mistake or failure.
A11. [Fill in.] Candidate: the imprecise clock wording itself, if it comes up ("I've since dug into exactly which macOS clocks stop during sleep").

Q12. Are you interviewing with other companies?
A12. I am actively exploring opportunities in systems and AI/ML engineering. I am in the early to mid stages of the interview process with a few companies, but I don't have any offer deadlines right now. I would prefer not to name any names while the process is still ongoing.

## Sources

- [Job posting: OS Software Engineer, Evaluation](https://jobs.apple.com/en-us/details/200688625-0836/os-software-engineer-evaluation?team=SFTWR)
- [Sibling posting: AIML - Sr OS Software Engineer, Evaluation](https://jobs.anitab.org/companies/apple/jobs/81833815-aiml-sr-os-software-engineer-evaluation)
- [Eclectic Light: triald is all about Machine Learning (2022)](https://eclecticlight.co/2022/04/06/triald-is-all-about-machine-learning/) and [the Trial tag archive](https://eclecticlight.co/tag/trial/)
- My own Mac (macOS 26.4): `plutil -p /System/Library/LaunchAgents/com.apple.triald.plist`, `man launchd.plist`, `man clock_gettime`
