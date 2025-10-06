# Keylogger-Creation-for-Educational-Use
Lab setup (preparation)

Create VMs

Create Target VM (where simulator.py will run) and Analysis VM (where detection scripts run).

Set network to host-only or an isolated lab network to prevent internet exfil.

Take snapshots of both VMs before you begin:

Example: “Snapshot — clean_state_01”.

Choose working folder

On each VM create a project folder for the lab (example):

Linux/macOS:

mkdir -p ~/safe_keylogger_lab
cd ~/safe_keylogger_lab


Windows (PowerShell):

New-Item -ItemType Directory -Path $env:USERPROFILE\safe_keylogger_lab
Set-Location $env:USERPROFILE\safe_keylogger_lab


Install Python & packages

Ensure Python 3.8+ is installed on both VMs.

Install required packages:

Linux/macOS / Windows (PowerShell):

python -m pip install --upgrade pip
python -m pip install psutil watchdog


Verify installation:

python -c "import psutil, watchdog; print('OK', psutil.__version__)"

Files to place in the project folder

Create or paste the safe scripts (you can copy from the earlier chat):

consent_logger.py (or consent_logger_robust.py)

simulator.py

file_activity_scanner.py

process_enumerator.py

Step-by-step execution plan
Phase A — Baseline & verification

Baseline process snapshot (Analysis VM)

Run:

python process_enumerator.py


Expected: A printed table of running processes and a file process_snapshot.json created in your folder.

Save process_snapshot.json as the baseline (copy to baseline_process_snapshot.json).

Start file watcher (Analysis VM)

Run:

python file_activity_scanner.py


Expected: Terminal displays: Watching sim_logs for new files. Ctrl-C to stop. (directory created if missing).

(Optional) Start consent logger (Analysis VM) — to demonstrate legitimate logging on analysis side:

Run:

python consent_logger.py


When prompted, type yes to consent, then type hello test and EXIT.

Expected: Terminal shows [logged @ ...]. File consent_log.txt (or temp fallback) contains the typed lines.

Phase B — Simulated “attacker” activity

Start simulator (Target VM)

Run:

python simulator.py


Expected: Simulator prints Created sim_logs/sim_log_001.txt every 5 seconds (files created in sim_logs folder).

Observe detection (Analysis VM)

The file watcher terminal should report:

[DETECT] New simulated log: sim_log_001.txt (size=... bytes)


If Analysis and Target are separate VMs, ensure the sim_logs folder is on a shared folder or run the simulator on the same VM as the watcher for simplicity.

Capture process snapshot while simulator is running (Analysis VM)

Run:

python process_enumerator.py


Expected: process_snapshot.json shows the simulator process (or python process) and flags suspicious paths if the executable is under a temp directory.

Collect artifacts

Copy these items into a collected_artifacts/ folder:

sim_logs/* files (simulated logs).

process_snapshot.json (after simulation).

file_activity_scanner console output (save terminal log / screenshot).

consent_log.txt (if used).

Example copy command:

mkdir -p collected_artifacts
cp -r sim_logs collected_artifacts/
cp process_snapshot.json collected_artifacts/

Phase C — Forensic checks & optional network simulation

Optional: simulate exfil within lab

Move files from sim_logs to a shared analysis folder (simulate exfil):

mv sim_logs/*.txt /path/to/shared/analysis/folder/


Monitor network traffic with Wireshark on the isolated network if you want to show transfer patterns.

Memory / strings checks (advanced & optional)

If you have Volatility and know how to capture a memory image from VM, you can search for strings from the simulator logs in the memory dump as a defensive exercise. (Do this only on lab VMs and sanitized images.)

What to record for the report

For each run include:

Lab environment: OS, Python version, VM snapshot name.

Exact commands you ran (copy & paste).

Screenshots of:

file_activity_scanner detecting new files.

simulator creating files.

process_enumerator output table and process_snapshot.json sample.

consent_logger showing logged entries and consent_log.txt.

Collected artifacts (packaged in collected_artifacts.zip).

Observations: file creation frequency, file sizes, suspicious process paths, and detection timestamps.

Expected outputs & how to verify them

sim_logs/ folder with files:

sim_logs/sim_log_001.txt
sim_logs/sim_log_002.txt
...


Each file contains a line: simulated entry X at 2025-10-05T...

file_activity_scanner.py output (real-time):

Watching sim_logs for new files. Ctrl-C to stop.
[DETECT] New simulated log: sim_log_001.txt (size=63 bytes)


process_snapshot.json (created by process_enumerator.py) containing a JSON array of process entries (pid, name, exe, cmdline, create_time). Compare to your baseline to see what changed.

consent_log.txt (if consent_logger.py succeeded) containing lines:

2025-10-05T12:00:00.000000 -- hello test

Cleanup & revert

Stop all running scripts (Ctrl-C).

Remove created files or revert snapshots:

If you used VMs, revert to the snapshot you took earlier (clean_state_01) — this is the safest cleanup.

If not using snapshots, delete produced folders:

rm -rf sim_logs collected_artifacts process_snapshot.json consent_log.txt sim_log_*.txt


Sanitize and redact any logs if you must store or share them (remove any accidental real data).

Troubleshooting tips

PermissionError when writing files:

Check write permissions:

import os
print(os.getcwd(), os.access(os.getcwd(), os.W_OK))


Use consent_logger_robust.py which falls back to the OS temp dir or console.

watchdog errors:

Make sure watchdog installed (pip install watchdog). On Linux you may need kernel headers for some optional features but basic watching usually works.

psutil errors:

Install via pip install psutil. On Windows run PowerShell as Administrator if permission errors arise when enumerating certain system processes.

