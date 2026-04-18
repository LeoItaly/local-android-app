<div align="center">

<img src="./android-pipeline.png" alt="Local Android app pipeline — workflow banner" width="100%" style="border-radius:16px" />

<br/>

# Local Android App

**An LLM-orchestrated workflow for building, installing, and updating your own Android apps — skill files as agent memory, checkpoint validation at every step.**

<br/>

![Platform](https://img.shields.io/badge/platform-Windows-0078D6?style=flat-square&logo=windows)
![Android](https://img.shields.io/badge/target-Android-3DDC84?style=flat-square&logo=android)
![Framework](https://img.shields.io/badge/framework-Expo%20%2F%20React%20Native-000020?style=flat-square&logo=expo)
![AI](https://img.shields.io/badge/agent-Claude%20Code-CC785C?style=flat-square)

</div>

---

## Why this exists

Every time I want to push a code change to my own Android phone from a React Native side project, it's a 6-step brittle ritual: build, sign, uninstall old version, install new, grant permissions, relaunch. Each step can fail silently — signature mismatch, `INSTALL_FAILED_UPDATE_INCOMPATIBLE`, version code collision, stale keystore path. When something breaks, you trace backwards through the logs to find which step actually failed. On a good day it takes 15 minutes. On a bad day it eats an afternoon.

The commands themselves aren't hard. They're deterministic. `gradlew assembleRelease`, `adb install -r`, `keytool -genkey` — no AI needed to produce them. **The hard part is the orchestration layer**: reading messy build output, deciding which step actually broke, knowing when to retry vs when to stop and ask. That's where an LLM earns its keep.

This repo encodes the full workflow as structured markdown skill files that Claude Code (or another chat-enabled IDE) reads step-by-step. Each step has an explicit test gate. The agent runs the command, validates the expected output, continues only on pass. On failure, it consults troubleshooting references, applies the smallest safe fix, and retests. No free-wheeling, no invented commands, no "looks good" on silently broken steps.

---

## What's in here

```text
local-android-app/
├── README.md
├── first-prompt.md                          ← paste this into your IDE chat to start
├── requirements-for-local-android-skills.md ← prereq checklist
├── expo-android-local-first-app/            ← skill: first-ever local install
│   └── SKILL.md
└── android-local-app-update/                ← skill: push updates to the installed app
    └── SKILL.md
```

Two skills, one playbook:

| Skill | When to use |
|------|-------------|
| `expo-android-local-first-app` | First time — create the Expo project, build a dev build, generate a keystore, build the signed standalone APK, install on device |
| `android-local-app-update` | Every time after — change code, bump version, rebuild signed APK, reinstall over the existing app |

---

## How it works

Each skill is a gated checklist. Every step has the same shape:

1. State what to change or run
2. Run it
3. Validate against an explicit test (file exists, expected output line, version code bumped, install ends with `Success`)
4. On failure, consult `references/troubleshooting.md`, apply the smallest matching fix, retest
5. Continue only when the test passes

Skill files are written as **agent memory**, not prompts. They give the agent a consistent shape to execute against — goal → prerequisites → command → expected output → common failure modes → recovery steps. The consistent shape is what makes the orchestration reliable. One generic prompt hallucinates Gradle flags and lies about success; a structured playbook with checkpoints doesn't.

The iteration that got it there:

- **v1** — one prompt that tried to do everything. Invented commands, reported "success" on steps that had silently failed.
- **v2** — split into per-step skill files. Stopped inventing commands.
- **v3** — added explicit test gates after each step. Stopped lying about success.

v3 is what's in the repo.

---

## Quick start

1. **Check prerequisites** → [`requirements-for-local-android-skills.md`](./requirements-for-local-android-skills.md). Don't skip.
2. **Clone and open** the repo in Claude Code (or another chat-enabled IDE that can read repo files and run commands).
3. **Paste** the contents of [`first-prompt.md`](./first-prompt.md) into the chat. This tells the agent to find the requirements, verify them, and enter the `expo-android-local-first-app` skill.
4. **Run the first-app skill** end-to-end. Don't let the agent skip a failed test — the skills are written to pause, fix, retest, then continue.
5. **For later updates**, run the `android-local-app-update` skill instead. Same pattern, shorter sequence.

---

## Important notes

> - This is a **local-only** workflow. Not Play Store publishing.
> - Standalone updates require the **same Android package name and same signing keystore** used for the first install. Lose the keystore and the update path breaks.
> - Keep the keystore at `android/keystores/release.keystore` so the Gradle relative path stays stable.
> - A development build (from `npx expo run:android`) is not the same thing as the standalone APK. Editing source files updates the dev build live; updating the standalone app requires rebuilding and reinstalling the signed APK.

---

## Troubleshooting

Each skill has its own `references/troubleshooting.md` covering the common failure modes — unauthorized ADB device, keystore path errors, `INSTALL_FAILED_UPDATE_INCOMPATIBLE`, PowerShell quoting issues. When a step fails, the agent is instructed to consult those references, apply the smallest matching fix, and retest. If a failure isn't covered there, the skill is designed to stop and surface the error rather than hide it.

---

## Status

Working on Windows 11 + Android physical device, Expo SDK, Node.js. Used across multiple React Native side projects (`panini-app`, others).

Known limitations: brittle to Gradle version drift (skills may need manual refresh if the build toolchain updates), tested on one phone + Windows setup. Next step is having Claude auto-validate the skills themselves against a minimal test project and flag steps that have gone stale.
