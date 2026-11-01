# FortiMon FAQ

Back to the [README](../README.md).

## Does FortiMon change anything on my FortiGate?

No. It uses a REST API admin with a read-only profile (System, Network, WiFi & Switch Controller and Log & Report set to
Read) and only reads.

## Where is my API key kept?

Encrypted in `config.json`, with a key stored beside it (`fortimon.key`; protected by Windows DPAPI on Windows, readable
only by the service user on Linux). It is never sent back to the browser, and a saved key is never sent to a different
FortiGate address unless an admin enters it again.

## A site shows as unreachable or is missing data

Open **Settings > Sites**, edit the site and run **Test connection**. Check that the FortiMon server's IP is in the API
user's **Trusted Hosts**, that the admin HTTPS port is right, and that the API user's profile has the read access listed in
the [setup guide](setup.md). The command-line `test-api` runs the same checks.

## The browser warns about the certificate

FortiMon makes a self-signed HTTPS certificate on first start. Continue past the warning once, upload your own certificate
under **Settings > Web server & HTTPS**, or trust FortiMon's certificate on the computers that use it.

## Why does FortiMon say "not offered" for something?

The FortiGate returned "not found" for that data set (for example a feature your model or firmware does not have). That is
not an error and it is not polled again until the firmware changes.

## Can I try it without a FortiGate?

Yes: run it with `--demo`. See the [setup guide](setup.md#try-it-without-a-fortigate).

## I forgot the admin password

On the server itself, run `--reset-password admin` (see [configuration](configuration.md#users-and-roles)). If the user does
not exist it is created as an admin.

## How do I update?

**Settings > Updates** checks once a day. A service install can use **Update now**, which backs up your data and goes back
to the old version by itself if the new one does not start properly. Otherwise run the newer installer over the old one: your
data and settings are kept. Servers on a version before 1.11.1 looked at an older download page, so install 1.11.1 or newer
once by hand.

## Is there a SiteMon connection?

Yes. Make a **card** key under **Settings > API keys** and add FortiMon in [SiteMon](https://github.com/RowShack/SiteMon), or
use **Send to SiteMon** on the same page if SiteMon cannot reach this server.

## What does it cost, and can I copy it?

It is free to use on as many computers as you like. It is closed source: read the [licence](../LICENSE) for what is not
allowed.
