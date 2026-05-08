---
title: An x64 dev box for OMSCS on Apple Silicon
date: 2026-05-08 00:00:00 -0400
categories: [OMSCS, Infrastructure]
tags: [omscs, azure, bicep, docker, gios]
---

I use an M-series Mac. I love it. It loves me. We're a happy family. ❤️

But many OMSCS courses only support x64-based machines. Emulation via Docker and Rosetta got me through most assignments, but [GIOS][gios] (CS 6200) was the breaking point — I wanted a native x64 toolchain instead of fighting an emulator on every debug session.

So I built [`gt-omscs-infra`][repo]: a one-command Azure dev box with per-course containers that CLion connects to over SSH.

## The mental model

One Azure VM runs Docker, with a separate container per course. CLion on the Mac connects to the container's exposed SSH port. Bicep provisions the VM, networking, NSG, public IP, and an auto-shutdown schedule.

```text
CLion (local Mac)
        │ SSH
        ▼
Azure VM (Docker)
 ├─ gios-env :2222
 └─ sat-env  :2223
```

## A few decisions worth calling out

**Azure VM, not Codespaces or serverless.** I had student credits sitting around, state persists between sessions, and I'm not paying per CPU-minute during a long debug session.

**One VM, many containers.** Idle cost is shared across courses, there's one auto-shutdown schedule, one set of creds to rotate. Containers keep each course's toolchain isolated.

**`COMPOSE_PROFILES` drives both Compose and the NSG.** Disabling a course on the next deploy removes its container *and* its inbound SSH rule in the same step, so the firewall and the running services never drift.

**Auto-shutdown via `Microsoft.DevTestLab/schedules`.** The VM deallocates every night. Idle compute drops to zero; only storage keeps ticking.

## The whole flow, end to end

```bash
# one-time
brew install azure-cli yq
az login
ssh-keygen -t ed25519  # if you don't have one already

# clone and configure
git clone git@github.com:kylemart/gt-omscs-infra.git
cd gt-omscs-infra
cp .env.example .env
$EDITOR .env  # set DNS_LABEL

# deploy
./deploy.sh
```

First run is about 5 minutes (Docker install + first image build). After that, redeploys are ~30 seconds and only touch what changed.

To verify:

```bash
ssh omscs@<fqdn> 'cd containers && docker compose ps'
```

When you're done with the term, one command takes everything down:

```bash
az group delete --name rg-omscs-eastus2 --yes --no-wait
```

## Where to go next

The repo has the full walkthrough — see [`docs/DEPLOY.md`][deploy] to deploy, [`docs/CLION.md`][clion] to wire up CLion, and [`docs/ARCHITECTURE.md`][arch] for the rationale behind each piece.

If you're an OMSCS student in the same Apple-Silicon-shaped corner I was in, I hope this saves you a weekend.

[repo]: https://github.com/kylemart/gt-omscs-infra
[gios]: https://omscs.gatech.edu/cs-6200-introduction-operating-systems
[deploy]: https://github.com/kylemart/gt-omscs-infra/blob/main/docs/DEPLOY.md
[clion]: https://github.com/kylemart/gt-omscs-infra/blob/main/docs/CLION.md
[arch]: https://github.com/kylemart/gt-omscs-infra/blob/main/docs/ARCHITECTURE.md
