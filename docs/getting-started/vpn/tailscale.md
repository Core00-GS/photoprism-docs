# Tailscale VPN

!!! tldr ""
    Tailscale is a third-party mesh VPN. If you run into issues with their service or client, contact the [Tailscale community](https://tailscale.com/community/) because we cannot debug vendor-specific problems.

Tailscale builds a private WireGuard® network between your devices, so you can reach PhotoPrism over an encrypted tunnel without exposing port 2342 to the public internet. The steps below cover the most common home-lab scenario (Linux server + mobile/desktop clients) and highlight optional ACL rules that keep untrusted nodes isolated.

## 1. Create a Tailnet and Install the Client

1. Visit [tailscale.com](https://tailscale.com/) and click **Use Tailscale**.
2. Sign up with Google, Microsoft, GitHub, Apple, or an email address. This creates a *tailnet* tied to your identity or organization.
3. Install the client on every device that should access PhotoPrism:
   - **Linux server (PhotoPrism host)**
     ```bash
     curl -fsSL https://tailscale.com/install.sh | sh
     sudo systemctl enable --now tailscaled
     sudo tailscale up --accept-dns=true --authkey tskey-auth-XXXXXXXXXXXXXXXX
     ```
     Generate one-time auth keys in the [Admin Console](https://login.tailscale.com/admin/settings/keys) or authenticate interactively with `sudo tailscale up` and a browser login.
   - **Mobile / desktop clients**: follow the platform downloads for [Android](https://tailscale.com/download/android), [iOS](https://tailscale.com/download/ios), [macOS](https://tailscale.com/download/macos), [Windows](https://tailscale.com/download/windows), etc. Sign in with the same account and approve each device when prompted.

!!! tip
    Enable [MagicDNS](https://tailscale.com/kb/1081/magicdns/) in the Admin Console so every node gets a friendly name like `photoprism.your-tailnet.ts.net`. This avoids memorizing 100.x.y.z IP addresses.

## 2. Verify Connectivity

Once the PhotoPrism host and your client devices are connected, they appear in the [Machines](https://login.tailscale.com/admin/machines) list:

![](img/tailscale-4.png){ class="shadow" }

Each device receives an auto-assigned 100.x.x.x address (sometimes shown as `100.64.0.0/10`). Use that IP or MagicDNS name plus PhotoPrism’s port (default `2342`) to reach the UI:

```
http://photoprism-host.ts.net:2342/
# or
http://100.120.34.10:2342/
```

If PhotoPrism runs behind a reverse proxy, continue to access it through the proxy port (for example `https://photos.ts.net/`).

!!! warning
    Make sure your firewall allows inbound connections on interface `tailscale0` (Linux) or the Tailscale adapter (Windows) for the PhotoPrism port. Otherwise the VPN will come up but requests to port 2342 will fail.

## 3. Optional: Share Devices or Use Funnel

- **Device sharing** lets you invite individual Tailscale users to a single machine without adding them to your whole tailnet. Use the **••• > Share** button next to a device in the Admin Console.
- **Tailscale Funnel** exposes a service to the public internet via Tailscale’s relay. Only enable Funnel if you understand the implications; at that point PhotoPrism is publicly reachable and you still need HTTPS plus authentication. For private family use, stick with the default private tailnet model.

## 4. Restrict Access with ACL Tags

If you host PhotoPrism in the cloud but only want outbound access *from* your desktop (not the other way around), create Access Control Lists (ACLs) that tag and isolate nodes. The example below creates two tags (`lan` and `cloud`) and prevents the cloud server from initiating connections to your LAN machines.

1. Open the [ACL editor](https://login.tailscale.com/admin/acls/file) and add tags + rules:
   ```json
   {
     "tagOwners": {
       "tag:lan": ["autogroup:admin"],
       "tag:cloud": ["autogroup:admin"]
     },
     "acls": [
       {
         "action": "accept",
         "src": ["tag:lan"],
         "dst": ["*"]
       }
     ]
   }
   ```
2. Go to the [Machines](https://login.tailscale.com/admin/machines) page, open the **ACL tags** dialog for each device, and assign `tag:lan` to desktops/laptops and `tag:cloud` to the cloud VM.
   ![](img/tailscale-7.png){ class="shadow" }
   ![](img/tailscale-8.png){ class="shadow" }
3. Test the policy:
     - Access PhotoPrism on the cloud VM from your tagged `lan` desktop (create an album, add labels, etc.).
     - SSH from the desktop into the cloud VM — should work.
     - Try to ping or SSH from the cloud VM back to the desktop — it should fail because no ACL rule allows it.

!!! note
    You can stack additional rules to permit maintenance traffic (for example, allow the cloud VM to reach your monitoring node) while still blocking everything else.

!!! example ""
    **Help improve these docs!** You can contribute by clicking :material-file-edit-outline: to send a pull request with your changes.
