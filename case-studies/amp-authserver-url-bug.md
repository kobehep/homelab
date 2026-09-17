# Case Study: A Management Platform's Default Config Pointed at the Wrong Host

## Situation

After installing CubeCoders AMP to manage the Valheim server, the "Manage Instance" dashboard flow broke immediately after login.

## Investigation

Both the base ADS instance and the Valheim sub-instance had shipped with `Login.AuthServerURL` set to `http://localhost:8080/` by default. That value resolves correctly only from the machine running the browser — not from the LXC actually hosting AMP — so any client trying to follow that URL to reach the instance was being pointed at itself instead of the server.

## Root cause

A default configuration value assumed the management UI would always be accessed from the same host it was installed on, which doesn't hold once AMP is managed remotely over the network — the normal way to use it.

## Corrective action

Reconfigured the value to the LXC's real LAN IP using AMP's own CLI tool: `ampinstmgr reconfigure <instance> +Core.Login.AuthServerURL 'http://<lxc-ip>:8080/'`. This required stopping the instance/ADS first — `ampinstmgr` refuses live config changes while the ADS is running.

## Lessons

- A "localhost" default in any config that's meant to be reached from other machines is a strong hint it needs to be corrected as part of setup, not left as-is because it looks harmless.
- Worth checking for this same pattern on any future AMP instance created on this box, rather than re-discovering it once per install.
