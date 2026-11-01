# FortiMon setup guide

This takes you from nothing to a dashboard showing your first FortiGate. Back to the [README](../README.md).

FortiMon connects to each FortiGate's REST API with a **read-only API key**, so it can look but not change anything.

## Requirements

- A **Windows** server (64-bit) or a **Linux** server with systemd and Python 3.9 or newer (Ubuntu, Debian, RHEL, Rocky
  and similar). The Windows installer includes everything it needs.
- A web browser. FortiMon serves its dashboard over HTTPS, on port 8443 by default.

## Install

Download the newest files from the [latest release](https://github.com/RowShack/FortiMon/releases/latest).

### Windows

1. Download `FortiMon-<version>.msi` and `SHA256SUMS.txt`.
2. [Check the download](#check-the-download).
3. Double-click the MSI and answer the questions:

   | Question | Choices |
   |---|---|
   | **Run FortiMon** | **As a Windows service** (recommended): starts with Windows, keeps monitoring when nobody is signed in and restarts itself if it stops. Or **as a program** that runs only while it is open. |
   | **Who can open the web interface** | **Other computers on the network** (adds a Windows Firewall rule) or **only this computer**. |
   | **Port** | 8443 by default (HTTPS). |

   At the end it opens FortiMon in your browser, where you create the first admin account.

Silent install (Intune, GPO, an RMM tool):

```
msiexec /i FortiMon-<version>.msi /qn MODE=service ACCESS=all PORT=8443
```

`MODE` is `service` or `app`, `ACCESS` is `all` or `local`. An upgrade keeps the earlier choices and all data: install the
newer MSI over the old one.

The service is called **FortiMon** in `services.msc`. Its data (settings, users, history, certificates, logs) is in
`C:\ProgramData\FortiMon`. Uninstalling keeps that folder; delete it yourself to remove everything.

**Windows may warn you.** The installer is not signed with a paid code-signing certificate yet, so SmartScreen can show
"Windows protected your PC". Check the SHA-256 below first. If it matches, choose **More info**, then **Run anyway**.

### Linux

```
sudo sh fortimon-<version>-linux.run
```

It asks whether to install a systemd service (recommended), the port, whether other computers may connect and, if ufw or
firewalld is active, whether to open the port. Run a newer `.run` over the top to upgrade; data is kept.

| | |
|---|---|
| Program | `/opt/fortimon` |
| Data | `/var/lib/fortimon` |
| Service | `systemctl status fortimon`; logs with `journalctl -u fortimon -f` |
| Command | `sudo fortimon --help` runs FortiMon's commands against the service's data |
| Uninstall | `sudo /opt/fortimon/uninstall.sh` (add `--purge` to delete the data too) |

For no questions: `sudo sh fortimon-<version>-linux.run --unattended --service --port 8443`. `--local-only`,
`--no-firewall` and `--no-service` also work; `--help` lists everything.

### Check the download

Each release lists `SHA256SUMS.txt`. The checksum of your file must match its line there.

```
# Windows (PowerShell)
Get-FileHash .\FortiMon-<version>.msi -Algorithm SHA256

# Linux
sha256sum -c SHA256SUMS.txt --ignore-missing
```

The in-app updater goes further: it only accepts releases signed with MonStack's own key, which is built into the app.

## First sign-in

Open `https://<server>:8443/`. The first time, FortiMon asks you to **create the first admin account**. After that
everyone signs in with their own account.

- **Certificate warning.** FortiMon makes its own (self-signed) HTTPS certificate, so browsers warn the first time. Continue
  past the warning, or put your company's certificate in under **Settings > Web server & HTTPS**.
- **Setup code.** If you create the first account from another computer rather than on the server itself, FortiMon asks for
  a setup code. It is in `setup-code.txt` in the data folder and in the log. This stops someone else on the network from
  claiming a fresh install first.

## Make a read-only API user on the FortiGate

1. *System > Admin Profiles*: create a profile with **Read** access to System, Network, WiFi & Switch Controller and
   Log & Report.
2. *System > Administrators > Create New > REST API Admin*: assign that profile and add the FortiMon server's IP under
   **Trusted Hosts**. Copy the API key it shows; it is only displayed once.

## Add the FortiGate

In FortiMon: **Settings > Sites > + Add FortiGate**. Enter the name, address, admin HTTPS port and the API key, click
**Test connection**, then **Add site**. Data appears within seconds.

Tokens are stored encrypted and never shown again. Changing a site's address means entering its token again.

**Test connection** runs the full API test and shows which data sets the FortiGate answered. A data set the FortiGate does not
offer is "not offered", not an error. A data set that is slow to answer (switch health with many FortiSwitches) is a warning:
FortiMon gives it more time automatically.

If a site shows as unreachable or is missing data, the same test runs from the command line, which is also useful before you
install: `FortiMon.exe test-api --host <FortiGate address>` on Windows, `sudo fortimon test-api --host <FortiGate address>`
on Linux. It asks for the token and does not save it; `test-api --help` lists the options.

## Try it without a FortiGate

`fortimon --demo` (or `python fortimon.py --demo` from source) runs three simulated sites with faults built in: HQ (an AP
outage traced to its switch port, a cable throwing CRC errors, a Wi-Fi coverage hole, a full DHCP pool, a rogue AP and WAN
packet loss), Cape Town (both internet links fail at the same moment, reported as one shared cause) and Durban (healthy).
Demo mode keeps its own settings and users.

## Next steps

- Create users and decide who needs two-step sign-in, and set up alerts and reports: see the
  [configuration reference](configuration.md).
- Read what each tab shows in the [user guide](user-guide.md).
- Put FortiMon's address under **Settings > Web server & HTTPS** so links in alerts and emails open the right page.
