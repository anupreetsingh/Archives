# Experimentation Daemon

OS updates deliver the capability. The experimentation daemon controls exposure: which devices see it, when, and how fast it's ramped or rolled back, all without an update.

## The flow

1. Server: a team defines the experiment (variants, targeting rules, allocation like 1% get B). It ships as config, not code.
2. Daemon: fetches configs in the background when it's cheap (Wi-Fi, charging, idle). On macOS/iOS this is Trial (`triald`).
3. Evaluation: the device checks targeting, then buckets itself by hashing a stable ID with an experiment salt. Same device, same bucket, no server-side list.
4. Assets: large payloads like ML models download separately in the background.
5. Framework: feature code asks for its value and gets the variant, or the default if not enrolled or anything fails.
6. Measurement: metrics come back privately, then the team ramps to 100% or kills it with a config change.

## Where OS updates still matter

Experiments can only switch between code that already shipped. A new feature ships dark in an OS release, gets ramped through experiments, then becomes the default.
