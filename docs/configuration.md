# FortiMon configuration reference

Everything is set in the web interface under **Settings** (admins only), and applies immediately. Back to the
[README](../README.md).

## FortiMon's own sections

| Section | What admins manage |
|---|---|
| **Sites** | Add, edit, pause or delete FortiGates: name, address, admin HTTPS port, API token, VDOM, event-log location and certificate checking. **Test connection** runs the full API test before you save. FortiGates usually have a self-signed certificate, so certificate checking is off unless you tick **Require a trusted certificate** for a site. |
| **Monitoring** | The poll interval (10 seconds by default), how long history is kept, the weekly and monthly saved reports (day and time), and the user-experience probes: which site this server is at, and the probe list. |
| **Firmware standards** | The firmware version each model should run (all sites, or per site), and fetching Fortinet's known issues and security advisories (on or off, optional proxy, **Refresh now**). |
| **Wallboard links** | Keys for screens nobody signs in to (shown under **API keys**, see below). |

### History

History is stored in `fortimon.db`, a SQLite file in the data folder, at three resolutions so the file stays small:

| Resolution | Kept for (default) |
|---|---|
| Every poll | 6 hours |
| 1-minute averages | 3 days |
| 15-minute averages | 90 days |

All three are set under **Settings > Monitoring**. The **History** picker at the top right switches every chart between 15
minutes and 90 days. The issue log, each metric's learned baseline and which ports, APs and links were recently up are saved
too, so diagnostics stay smart after a restart. To start fresh, delete `fortimon.db`.

### Monitoring FortiMon itself

`/api/health` returns 200 while every site has polled in the last 2 minutes and 503 otherwise, so external monitoring can
watch FortiMon. Without signing in it returns only the overall status. Logs: `C:\ProgramData\FortiMon\logs\fortimon.log` on a
Windows service, `journalctl -u fortimon` on Linux, or the console when run by hand.

### Older settings files

Older `config.json` files keep working: a plain `token`, or a `token_env` environment variable, is still read for each
FortiGate.

## Settings shared by every MonStack app

These sections are the same in FortiMon, UniMon, MerakiMon and SiteMon. Open **Settings** with the gear button at the top
right. Changes apply immediately; nothing here needs a restart.

### Users and roles

| Role | Can |
|---|---|
| **Admin** | Everything: the settings, users, API keys, backup and restore, the audit log, acknowledging issues and saving reports. |
| **Operator** | Everything a view-only user can, plus acknowledging and muting issues and saving reports. Cannot change settings or users. |
| **View only** | Every dashboard and report. Cannot change anything. |

Operators and view-only users can be limited to some sites; they then see only those everywhere, including the wallboard
and the API. No sites ticked means every site, including ones added later. Admins always see every site.

Admins add users under **Settings > Users** with a temporary password, and can change roles, reset a password (which
signs the person out everywhere), and disable or delete accounts. There is always at least one active admin. Everyone
can change their own password under **My account**.

- **Lockout:** after 5 wrong passwords in 15 minutes from one address, or for one username, sign-in is locked for a while.
- **Session expiry:** a session ends after 24 hours without use, and 14 days after signing in at the latest.
- **Two-step sign-in:** anyone can add authenticator-app codes (with 10 one-time backup codes) under **My account**. Admins
  can require it for **Nobody**, **Admins** or **Everyone** under **Settings > Users**.

Forgotten password with no other admin, or a lost phone with no backup codes? Run this on the server itself:

```
"C:\Program Files\FortiMon\FortiMon.exe" --reset-password admin     (Windows, as Administrator)
sudo fortimon --reset-password admin                                 (Linux)
```

`--reset-mfa USER` turns off two-step sign-in for a user, who sets it up again at the next sign-in.

### API keys

**Settings > API keys** makes keys for other programs. A **wallboard** key opens the full-screen wallboard on a screen nobody
signs in to. A **card** key lets SiteMon read this app's summary (`GET /api/card/v1`) and nothing else. A key is shown once,
stored only as a hash, can be limited to some sites, and can be revoked. The same page has **Send to SiteMon**, which
pushes the card to a SiteMon that cannot reach this server (see the [SiteMon guide](https://github.com/RowShack/SiteMon)).

### Appearance & branding

Dark, light or follow the device, the accent colour, and your own product name and logo.

### Web server & HTTPS

Who can connect (this computer or the network), the port, HTTPS on or off, the address used in links in email and alerts,
and the HTTPS certificate.

On first start the app makes a self-signed certificate for the server's names and addresses and renews it before it
expires, so browsers warn the first time. Either continue past the warning, upload your company's certificate (PEM, with
its key) here, or trust the app's own certificate on the computers that use it. Turn HTTPS off only behind a reverse proxy
that does HTTPS (`--http` on the command line does the same).

### Email reports

Enter your SMTP server (Microsoft 365, Google Workspace and internal relays all work) and use **Send test email**. Add
recipients, who do not need an account, and choose for each weekly and/or monthly reports and which sites. The weekly
report goes out on Monday at 06:00 by default (the day and hour are set under **Monitoring** in FortiMon, UniMon and
MerakiMon) and the monthly one on the 1st.
**Send now** sends this week's report straight away.

### Alerts

**Settings > Alerts** sends issues to people as they happen. Add destinations: **Microsoft Teams** (a Workflows webhook),
a **webhook** (JSON to any HTTPS address) or **email** (using the SMTP server above). Each destination has its own filters
for sites, areas, minimum severity and problem types, and can also send when an issue clears.

- Issues raised close together are grouped into one message a minute, and each destination is capped per hour.
- Muted and score-excluded issues never alert; an acknowledged issue alerts again only if it gets worse.
- A failed delivery is retried a few times. Each destination keeps a delivery log, and admins see a banner if one has been
  failing for over an hour. **Send test** sends a clearly marked sample.

To make a Teams URL: in the channel choose **... > Workflows**, pick **Post to a channel when a webhook request is
received**, name it, choose the team and channel, copy the URL it shows and paste it into the destination. Use a Workflows
URL: Microsoft is retiring the old Office 365 Incoming Webhook connectors.

### Updates

**Settings > Updates** checks this app's GitHub releases once a day (it is a public page, so no account or token is
needed) and shows what is new. For a **service** install, **Update now** downloads the release, checks its signature and
checksums, backs up your data, installs it, and checks that the new version answers. If it does not, the helper puts the
old version back by itself. A server without internet access can use the **offline update** file from the release.
The channel is **Stable**, or **Test releases too** for a test server.

### Audit log, backup and restore

The audit log records every sign-in (including failed ones), settings and user change, acknowledgement and report email,
with who, when and from which address, for 400 days.

**Backup & restore** downloads all settings and users as one file and restores it, for example on a new server. Secrets in
it (API keys, the SMTP password) are encrypted with a backup password you choose. The history database is not in the
backup: copy `fortimon.db` from the data folder if you need it. Two-step sign-in is not in a backup, so users set it up
again after a restore.

### The data folder

| File | What it is |
|---|---|
| `config.json` | Settings, written by FortiMon when admins change them (the previous version is kept as `config.json.bak`) |
| `fortimon.key` | The key that encrypts stored secrets. On Windows it is itself protected by Windows (DPAPI); on Linux only the service user can read it. Without it the keys and passwords have to be entered again. |
| `admin.db` | Users (password hashes), sessions and the audit log |
| `fortimon.db` | History |
| `tls/` | The HTTPS certificate and key |
| `reports/`, `logs/` | Saved reports and the service log |

`C:\ProgramData\FortiMon` on a Windows service, `/var/lib/fortimon` on a Linux service.

### Command line

| Option | Meaning |
|---|---|
| `--data-dir DIR` | Where settings, users and history are kept |
| `--demo` | Simulated data, no gear needed (it keeps its own settings and users) |
| `--port 8443` / `--bind 0.0.0.0` | Web interface port / address (saved to the settings) |
| `--http` | Turn HTTPS off, for example behind a reverse proxy (saved) |
| `--interval N` | Seconds between reads (saved) |
| `--reset-password USER` | Set a new password (creates the user as an admin if missing; `--make-admin` also makes an existing user admin) |
| `--reset-mfa USER` | Turn off two-step sign-in for a user |
| `--open` / `--no-browser` | Open the running app in the browser / do not open it on start |
| `--log FILE` | Also write the output to a file |
| `--version` | Print the version |

## Command line, FortiMon only

| Option | Meaning |
|---|---|
| `--host IP --token T [--fgt-port 8443] [--vdom root]` | Monitor just this one FortiGate (not saved) |
| `test-api ...` | The API tester (`test-api --help`) |
