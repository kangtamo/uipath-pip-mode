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
2. Opens **Notepad** in the isolated desktop.
3. Types an explanatory message into it.
4. Loops for ~20 seconds logging progress (simulated background work).
5. Logs a completion message.

The Type Into step is deliberately configured to use **hardware input** (`SimulateType=False`)
with **`ActivateBefore=True`**. This forces Notepad to the **foreground** and sends **real
keyboard events** — which is what makes the with-PiP vs without-PiP difference obvious:

- **Without PiP** — the automation grabs your foreground window and keyboard mid-run. If you
  are typing in another app, the keystrokes land in Notepad instead. Clearly disruptive.
- **With PiP New Desktop** — the exact same activation and keystrokes happen inside the
  isolated desktop, so your real desktop keeps its focus and keyboard. You work uninterrupted.

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

Run the **same** process twice and try to keep working (e.g. type into another app) during each run:

1. **Without PiP** — run normally from Assistant/Studio. Notepad jumps to the foreground and the
   robot types with real keystrokes, hijacking your keyboard. Your own typing gets interrupted.
2. **With PiP New Desktop** — enable PiP New Desktop (Option A above) and run again. The identical
   activation and typing happen in the isolated desktop; your foreground app and keyboard are
   untouched, and no password was required.

## Files

- `project.json` — attended, PiP-ready process definition (Windows target).
- `Main.xaml` — the demo workflow.
