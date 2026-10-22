## First Impressions of Willa Project

`willa-astro/` was the marketing site, built with **Astro**, but is now scrapped. Astro is a static site generator in the same space as Next.js, except it isn't built entirely on top of React like Next.js is. Astro ships plain HTML/CSS by default and only loads React, Vue, or other such UI framework components ("islands") where a page actually needs interactivity.

`Slate/` was meant to be the company's own CMS(Content Management System) — an in-house alternative to WordPress — but it's now scrapped as well. The only integration it ever shipped was for Astro sites (a package called `astro-slate`), with `willa-astro` as its sole consumer.

`willa-wordpress/` is the currently live marketing site.
**Wordpress** splits two jobs:

1. Overarching Theme set by the developers
2. Content Editing done by the editor in the editor UI.

`willa/` is the actual application, still in development, and not yet wired into the marketing site — there's no sign-up or "sign in" path from `willachat.com` to the app yet. That link is meant to be added once `willa/` is ready to ship as a live webapp.

## Verifying the Microsoft App

1. Create the Microsoft Partner Account for PBS-LLC.
2. Get the account verified.
3. Repackage the desktop app from NSIS to MSIX/APPX(Electron can do it, not a big deal)

## General Knowledge about the App

Guardian Agent AI Classifier in a general network request from a device that has WillaChat installed.

```text
Child's browser
      ↓
OS network filter
      ↓
Local TLS proxy
      ↓
AI classifier
      ↓
Safety pipeline
      ↓
Allow  |  Rewrite  |  Block  |  Alert
      ↓                    ↓
Traffic continues     Child sees a block page
      ↓
Audit event
      ↓
Parent dashboard
```

**Tamper resistance** — a child must not be able to stop the service, uninstall it, or delete the root CA without the parent's password. That requirement is what drives the entire packaging discussion.

## ToDos

1. Client side agent for blocking inappropriate requests on the network on that device.
2.

## Microsoft Installer

General Overview of the Release pipeline:

```txt
┌────────────┐   ┌────────────┐   ┌────────────┐   ┌────────────┐   ┌────────────┐   ┌────────────┐
│    BUILD   │ → │   PACKAGE  │ → │    SIGN    │ → │ DISTRIBUTE │ → │   INSTALL  │ → │   UPDATE   │
└────────────┘   └────────────┘   └────────────┘   └────────────┘   └────────────┘   └────────────┘
 electron-        EXE · MSI ·      CA cert +        download ·       user's           electron-
 builder          MSIX             timestamp        Store · MDM      machine          updater
```

### Build

Electron helps you build your app: `electron-builder` takes `main.cjs`, the Vite `dist/`, the Python sidecar, and the packaged Electron runtime, and produces a folder of files. Same folder regardless of format.

### Package

`electron-builder` produces a folder: `WillaChat.exe` (Electron's own binary, renamed),
`resources/app.asar` holding the Vite `dist/` and `main.cjs`, the Python sidecar, and the
Chromium DLLs. Double-click it and Willa runs — but it's a couple hundred loose files with
no shortcut, no uninstaller, and no entry in "Add or remove programs." Packaging is what
wraps that folder into something you can hand to a parent.

Electron itself has no installer story; it stops at the folder. `electron-builder` fills
the gap by handing that folder to a packaging backend — and which backend runs is the
choice between the three formats below.

1. **EXE** (Executable) — built by **NSIS** (Nullsoft Scriptable Install System), a
   compiler that turns an install script plus a folder of files into one self-extracting
   `.exe`. `electron-builder` writes the script and invokes NSIS for you; it even
   downloads NSIS into a cache, so nothing is installed on the build machine. Maximum
   freedom, zero enforced structure — an EXE installer is just an ordinary program that
   happens to copy files. Windows doesn't know it's an installer and can't undo what it
   did. Uninstall works only because the script wrote an `Uninstall.exe`. **This is what
   Willa ships today.**

2. **MSI** (Microsoft Software Installer) — not a program at all, but a *database* of what
   the installed state should look like, which the Windows Installer service reads and
   applies. Windows knows what's inside, so it can roll back a failed install, repair a
   damaged one, and uninstall cleanly without cooperation from you. Less freedom, far more
   accountability.

3. **MSIX** — the successor to APPX (the old Store/UWP container format), extended to
   handle traditional desktop apps. A signed archive plus a manifest, installed into a
   sandbox. Guaranteed clean uninstall, because the sandbox contains everything the app
   ever wrote.

The app inside is byte-for-byte identical in all three. Format is a late, reversible
choice — in `desktop/package.json` it's one string in `build.win.target`.

### Sign

When a parent downloads the app, Windows checks whether the file is signed. A **signature** does two things: proves the file came from you, and proves nobody modified it in transit.

To sign, you buy a code-signing certificate from a **CA** (certificate authority) like DigiCert or Sectigo. Two grades exist:

- **OV (Organization Validation)** — the CA confirms your company legally exists. Cheaper, lighter paperwork.
- **EV (Extended Validation)** — a much stricter identity audit, and the private key must live on a hardware token or cloud HSM (Hardware Security Module), so it can't be copied off a laptop. EV certificates cost roughly 600 $ / year.

Then SmartScreen (Microsoft's reputation service) decides what the parent sees:

- Unsigned: a red-tinted "Windows protected your PC" wall. The "Run anyway" button is hidden behind "More info." Most parents stop here.
- OV-signed, new: still warned, because the certificate has no download history yet. The warning fades as installs accumulate — which can take weeks or months you don't have.
- EV-signed: historically gets immediate or near-immediate SmartScreen trust, skipping the reputation-building period. That's the entire reason to pay for EV.

### Distribute

The considerations for putting them on store are:

```text
|                   | **MSI / EXE on the Store**          | **MSIX on the Store**                |
| ----------------- | ----------------------------------- | ------------------------------------ |
| **Install**       | Your own installer runs             | The Store installs the package       |
| **Updates**       | You ship them (`electron-updater`)  | The Store handles them               |
| **Signing**       | Your own OV/EV certificate          | Microsoft signs it — no cert needed  |
| **System access** | Full — services, drivers, root CAs  | Sandboxed                            |
| **Uninstall**     | However you wrote it                | Guaranteed clean                     |
| **Guardian**      | Works                               | Cannot work                          |
```

**Imp:** Guardian Agent on the Client side because of MSIX's sandbox — the thing that makes it clean and safe — is precisely what prevents installing a system-wide network filter service and a trusted root CA.

**Decision**: Because of that we are going to go with EXE / MSI,  get an EV Certificate and sign the app so it doesn't give a warning at install.

## Guardian

You can block the request at two levels:

### Part A: DNS lookup, OS network stack when asked for opening a socket or first step of TCP handshake

DNS:  Make the computer's "phone book" refuse to look up AI vendors.  On macOS you ship a configuration profile ; a `.mobileconfig` file — the same mechanism schools and companies use to configure Macs

### Part B: HTTPS traffic actually flowing as prompts to the AI vendors

The intended flow:

```txt
Browser builds the HTTP request (plaintext, inside the browser)
  → browser's own TLS code encrypts it for "claude.ai"
  → leaves the browser as ciphertext
  → OS filter catches the connection and redirects it to 127.0.0.1 (GuardianProxy)

  TLS leg 1: browser ⇄ GuardianProxy
  → proxy completes the TLS handshake pretending to be claude.ai,
    showing a claude.ai certificate it generated, signed by the per-device CA
  → browser accepts it, because that CA was installed in the OS trust store
  → so the proxy holds the session keys and can decrypt → plaintext HTTP request
  → classifier reads it → allow or block

  TLS leg 2: GuardianProxy ⇄ real claude.ai
  → proxy opens its own TLS connection to the real server, re-encrypts, forwards
  → the response comes back through both legs in reverse
```
