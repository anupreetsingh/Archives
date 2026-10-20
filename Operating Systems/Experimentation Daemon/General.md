# Experimentation Daemon

A feature can ship in an OS update without being turned on for everyone. The experimentation daemon decides who gets it: which devices, when, and how fast it ramps up or gets rolled back, all through server config instead of another update. Apple's experimentation daemon is `triald`. The Foundations/ notes use `expd`, a stand-in based on my on-device experimentation project mentioned in the resume, as a running example to explain OS mechanisms.

The most common use is an A/B experiment: test a change on a slice of users before shipping it to everyone. The control group (A) keeps the current behavior, the treatment group (B) gets the change, and a metric is compared between the two groups. The example below tries a new search ranking on 1% of devices.

**Experimentation** is running the A/B experiment: devices are split into groups, and each group gets a different **variant** of the behavior, here control or treatment. **Evaluation** comes after: the logged data is used to judge whether the treatment group actually did better than the control group. An ML model is also evaluated offline on a fixed test set before it ships; the A/B experiment is its online evaluation.

## The flow

1. Define: a team writes the experiment on the server as config, not code: experiment name, variants name, targeting rules, and split, e.g. `new_search_ranking` with control/treatment at 99/1, only on macOS 26 and later.
2. Fetch: the daemon downloads configs in the background when it's cheap (Wi-Fi, charging, idle), at a time the dasd daemon picks. Large payloads like ML models download separately.
3. Assign: the device checks targeting, then buckets itself by hashing a stable ID with the experiment name, e.g. `hash(id + "new_search_ranking") % 100 < 1` → treatment. The same device lands in the same bucket every time, so the server keeps no per-device list. The experiment name acts as a **salt** (a value mixed into the hash), so different experiments bucket independently.

   The stable ID is one per machine, and the daemon owns it. On first run it generates a random one, e.g. with `uuid_generate_random()`, so nothing comes from the hardware or the account, and stores it in its protected data directory, so it survives reboots. Apps get only the variant name over XPC, never the ID or the bucket, and the server never needs it because bucketing happens on the device. It changes only if that file is deleted, like when the device is erased, and then the device lands in new buckets in every experiment.
4. Ask: at the branch point, the app asks the experimentation daemon over XPC for its variant, which the daemon picked from the device's bucket and the config, then branches on it. If the device isn't enrolled or anything fails, it gets the default, control:

   ```c
   if (strcmp(variant, "treatment") == 0) use_new_ranking();
   else use_old_ranking();
   ```

   The app doesn't know in advance whether it's in an experiment. The developer shipped the branch point in the OS, so the apps code asks on every device, and the daemon's answer says whether this one is enrolled. That's why experiments start, ramp, and stop with only a server config change. Real systems often ask for a parameter like `search.ranking_model` instead of an experiment name, and the daemon maps the running experiment onto it, so a follow-up experiment needs no code change at all.

5. Log: the app records exposure (the user actually hit the changed code) plus outcome metrics like clicks on results, tagged with the variant. These go back to the server privately.

   The server is the experimentation platform's backend, the same one that served the config in step 1. For an Apple feature like Finder, that's Apple's server. Events upload only if the user turned on analytics sharing, arrive without the Apple Account attached, and get aggregated per variant. The team that owns the feature sees only those aggregates in step 6. Third-party apps can't use Trial, so they send data to their own company's experimentation service.
6. Decide (evaluation): the server compares the two groups with a statistical test to check that treatment beat control by more than chance, then ramps the winner to 100% or turns it off with a config change.

## Why a daemon owns it

Each app could fetch configs and hash for itself, but one daemon gives:

- One fetch for all apps, which saves network and battery.
- One assignment shared by every process, so an app and its frameworks can't disagree.
- Opt-outs and kill switches (turning an experiment off everywhere at once) enforced in one place.

## Pitfalls

- Counting assignment instead of exposure **dilutes** the result with users who never saw the change.
- An **unstable** ID flips users between variants and mixes the data.
- Stopping as soon as B looks good (**peeking**) inflates false positives.

## Where OS updates still matter

Experiments can only switch between code that already shipped. A new feature ships dark (present but off) in an OS release, gets ramped through experiments, then becomes the default.

## triald: Apple's version

Trial is Apple's experimentation system, and `triald` is its daemon. It also delivers ML models and other assets without an OS update. Its code is private, so everything here comes from its launchd plists, read with `plutil -p /System/Library/LaunchAgents/com.apple.triald.plist`.

It runs as two processes:

- `/usr/libexec/triald`: a per-user agent, from `/System/Library/LaunchAgents/com.apple.triald.plist`.
- `/usr/libexec/triald_system`: a system daemon, from `/System/Library/LaunchDaemons/com.apple.triald.system.plist`. One of its Mach services is `com.apple.triald.system.from-agent`, so the agent talks to it over XPC.

How it maps onto the flow above:

- Fetch (step 2): the plist declares XPC activities under `LaunchEvents` → `com.apple.xpc.activity`, which dasd schedules. `com.apple.triald.fetch-experiments` repeats daily (`Interval` 86400), only on Wi-Fi (`RequireInexpensiveNetworkConnectivity`), never on battery (`AllowBattery` false), after a random delay of up to an hour (`RandomInitialDelay` 3600) so devices don't all hit the server at once. A cellular fetch runs every 3 days, and one-off fetches run after an OS upgrade (`PostInstall`) and after setup (with a 2 GB `NetworkDownloadSize` budget).
- Push: the `com.apple.aps.triald` Mach service is its Apple Push Notification channel, so the server can wake it for an urgent change, like a kill switch, instead of waiting for the daily fetch.
- Assign (step 3): it re-checks targeting when an input changes. Its `LaunchEvents` wake it on Darwin notifications (system-wide broadcasts through notifyd), like `com.apple.language.changed`, `com.apple.GeoServices.countryCodeChanged`, and an Apple Account being added or removed.
- Ask (step 4): apps don't send raw XPC dictionaries. They link the private `Trial.framework` in `/System/Library/PrivateFrameworks`, which talks to triald.
- Log (step 5): it wakes on `com.apple.crashreporter.auto_submit_preference_changed`, which fires when the user turns analytics sharing on or off.
- Idle exit: `EnablePressuredExit` and `EnableTransactions` let the system kill it when it has no open requests, and launchd starts it again on the next event.
- Data: kept in `~/Library/Trial`, which even your own user can't list ("Operation not permitted"). `RequiresClassC` on every activity means it runs only after the first unlock since boot, when files with data protection class C become readable.

## expd: the study version

`expd` isn't a real daemon. It's the example the Foundations notes build to explain each mechanism, named `com.example.expd` to mark it as hypothetical:

It stands in for the daemon in my On-Device Experimentation Service project and the design answer in [Apple.md](../../Interview%20prep/Apple/Apple.md). It isn't called triald because triald's internals are private. expd's code is something I can explain line by line; for triald I only state what its plists show.

| | expd | triald |
|---|---|---|
| Runs as | one system daemon in `/Library/LaunchDaemons` | a per-user agent plus a system daemon |
| Mach services | `com.example.expd` | five (agent) and six (system), like `com.apple.triald.system.from-agent` |
| Clients talk through | raw XPC dictionaries, or a Unix domain socket | the private `Trial.framework` |
| Fetch schedule | one `xpc_activity` | daily Wi-Fi, 3-day cellular, post-upgrade, setup, and push |
