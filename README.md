# GNX Dockur interactive compatibility lane

This local GitHub harness orchestrates the pinned `dockurr/windows` image; it does not modify Dockur. The lane is consumer-style compatibility evidence, not a substitute for the physical three-host GNX acceptance lab.

It fails closed unless the hosted Linux runner supplies KVM API 12, nested virtualization, TUN, at least 15 GiB RAM, and 36 GiB free disk. The frozen setup asset is downloaded with read-only repository contents permission and SHA-256 verified on Linux before Windows starts.

## Interactive sequence

Dispatch one deterministic logical slot at a time, in order: `controller`, `member-1`, then `member-2`. These are independent ephemeral compatibility sessions and cannot prove quorum, Corosync, direct paths, or cluster-network acceptance.

1. Open the authenticated HTTPS noVNC URL in the job summary. It is a Tailscale Serve endpoint to Dockur's loopback-bound web viewer on port 8006; no Funnel is used.
2. In Windows, open `C:\OEM` and double-click `01-Install-GNX.cmd`. It starts the frozen `Quetzalcoatl-0.3.0-preview-setup-x64.exe` through the normal UAC flow. The `90-Repair-GNX.cmd` and `99-Uninstall-GNX.cmd` helpers reuse that exact frozen bundle.
3. Allow every real Windows reboot the installer requires; do not treat viewer availability as product convergence.
4. Double-click `02-Configure-GNX.cmd`. It runs the real `gnx configure` in an interactive Windows console; secret entry is deliberately interactive and must not be piped or automated.
5. Double-click `03-Collect-GNX-Evidence.cmd`. It writes redacted evidence to Dockur's documented shared `Z:` drive, including the installer hash, detected GNX service state/identity, installed binary hashes, and the exact `gnx status --json` output when GNX is installed.
6. Interpret `gnx status --json` as the runtime evidence. If GNX is absent or unconfigured, the collector says so honestly; the workflow does not claim convergence.

Official Dockur `/oem/install.bat` is intentionally not used. GNX configuration needs a real interactive Windows console, so this lane exercises the consumer flow rather than an unattended surrogate.

The noVNC session is explicitly limited to 60, 120, or 180 minutes and is then cleaned up. Optional loopback RDP is intentionally secondary; no credentials, passwords, or auth keys are placed in the job summary. Evidence artifacts are redacted and retained for one day.

The workflow reuses the existing `TS_RDP_AUTH_KEY`/`tag:github-rdp` edge for authenticated noVNC as well as optional RDP; it does not introduce a second tailnet key.

DERP or RTT of 5 ms or more remains a physical/GNX cluster acceptance failure when actual member evidence exists. This hosted lane neither fabricates a cluster probe nor derives network acceptance from the runner edge. Historical native Windows workflows remain manual/non-required coverage.
