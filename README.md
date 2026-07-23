# PiP New Desktop Demo (UiPath 2024.10.9, Windows)

An **attended** UiPath automation that demonstrates the benefit of **Picture in Picture (PiP)
"New Desktop" mode**: the process runs in an isolated desktop **inside your current Windows
session**, so it needs **no additional authentication** and **does not stop you from using
your machine** for other work while it runs.

## Why "New Desktop" needs no extra sign-in

PiP can isolate an attended run in two ways:

| PiP mechanism | How it isolates | Authentication |
|---|---|---|
| **Child Session** | Starts a **second Windows logon session** for your user | Needs your **Windows credentials** (or auto-login), because it is a real second logon |
| **New Desktop** (this demo) | Creates a new **desktop object inside your *existing* logon session** | **None** — you are already logged in, so no password prompt |

Because New Desktop reuses the session you are already signed into, there is no second logon and
therefore no extra authentication. Your primary desktop keeps full control of the keyboard and
mouse, so you can keep working while the robot works in its own desktop.

## What the automation does

[Main.xaml](Main.xaml):

1. Logs a start message.
2. Opens **Notepad**.
3. Types an explanatory message into it (Simulate input — works in the isolated desktop).
4. Loops for ~20 seconds, **repeatedly activating Notepad to bring it to the foreground**.
5. Logs a completion message.

The repeated **Activate** (bring-to-foreground) in step 4 is the real differentiator — see below.

The benefit of PiP New Desktop is **not** about typing text — a background input method like
Simulate types the same whether the desktop is visible or isolated, so typing alone shows no
difference. The benefit is about anything that **takes over your screen and steals focus**. PiP
gives the robot its own desktop to do that on, leaving your real desktop free.

This automation demonstrates it with **repeated window activation**: in the loop it keeps
bringing Notepad to the foreground (`Activate`).

- **Without PiP** — Notepad keeps jumping in front of whatever you are working on and stealing
  focus. You cannot use your machine while the automation runs.
- **With PiP New Desktop** — the exact same activation happens inside the **isolated desktop**,
  so your real desktop is never touched. You keep working (or lock your screen) the whole time.

Activation works in PiP because it is a **window-message** operation, not hardware input.

> **Why the typing itself doesn't differ:** the Type Into step uses **Simulate**
> (`SimulateType=True`), which pushes text straight into the control regardless of foreground —
> so it behaves identically with and without PiP. (Hardware keystrokes *would* differ, but they
> never reach the isolated desktop, producing an empty Notepad — so they aren't usable here.)

Same automation, run two ways — see [Demonstrating the difference](#demonstrating-the-difference).

## Requirements

- Windows
- **UiPath Studio 2024.10.9** (project targets Windows / .NET compatibility)
- **UiPath Assistant / Robot 2024.10** (PiP New Desktop is a Robot feature)
- Dependencies (auto-restored on open):
  - `UiPath.System.Activities` `24.10.5`
  - `UiPath.UIAutomation.Activities` `24.10.6`

> If your local feed has slightly different patch versions, let Studio resolve them via
> **Manage Packages** — the automation only uses core activities (Log Message, Start Process,
> Delay, Type Into, While).

## How to run the demo

### Option A — from UiPath Assistant (recommended for the demo)

1. Publish this project to your Assistant/Orchestrator, or add the local folder as a process.
2. In **UiPath Assistant**, find the process and open its **settings** (the "…" menu).
3. Set the run mode to **Picture in Picture**, and choose **New Desktop** as the PiP type.
   (Assistant setting — *not* stored in the project. It applies per-process at run time.)
4. Click **Run**.
5. A PiP window opens showing the isolated desktop. **Immediately start typing in your own
   apps** — you will see both desktops work at the same time, and you were never asked for a
   password.

### Option B — from Studio (for development/debug)

1. Open the folder in **Studio 2024.10.9** (open `project.json`).
2. Let dependencies restore.
3. **Debug** ▸ enable **Picture in Picture**, then **Run**.

> The project is marked PiP-ready in `project.json`:
> ```json
> "runtimeOptions": {
>   "isAttended": true,
>   "readyForPiP": true,
>   "startsInPiP": true
> }
> ```
> The **New Desktop vs Child Session** choice itself is made in **Assistant/Robot at run time**,
> not in the project file.

## Demonstrating the difference

Run the **same** process twice. During each run, try to keep working in another app (a browser,
a document — anything you type into):

1. **Without PiP** — run normally from Assistant/Studio. Every few seconds Notepad jumps in front
   of your work and takes focus. You cannot get anything done while it runs.
2. **With PiP New Desktop** — enable PiP New Desktop (Option A above) and run again. The identical
   activation loop runs in the **isolated desktop**; your real desktop is never disturbed and no
   password was required. Keep typing in your other app — nothing interrupts you.

### Making it even more dramatic (optional, in Studio)

Add a **Maximize Window** activity (`UiPath.Core.Activities.MaximizeWindow`) targeting Notepad
at the top of the loop so the window covers the whole screen each time it activates. It needs a
Window input, so wrap it in **Use Application/Browser** (or **Attach Window**) pointing at
Notepad. This is easiest to add in the Studio designer; I left it out of the shipped XAML because
its Window-object wiring is fiddly to hand-edit. `Activate` alone already delivers the
focus-stealing effect.

## Files

- `project.json` — attended, PiP-ready process definition (Windows target).
- `Main.xaml` — the demo workflow.
