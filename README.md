# 🛡️ GL.iNet Security Incident (Nov 2026): Check, Fix & Block

Copy-and-paste commands to check GL.iNet routers and KVMs for the malware in GL.iNet's security notice of 20 November 2026, install the fixed firmware, and block the IP range reported by the community.

| Label | Meaning |
| --- | --- |
| 🟢 **Read-only** | Only looks. Changes nothing on your device |
| 🟠 **Changes things** | Modifies your device's config |

---

## 🆕 Updates

| Date | What changed |
| --- | --- |
| **27 Nov 2026** | GL.iNet's COO has posted a [statement on the incident](https://www.reddit.com/r/GlInet/comments/1y77edg/a_message_from_glinet_coo/) on r/GlInet. |
| **24 Nov 2026** | GL.iNet updated its [security notice](https://www.gl-inet.com/en-us/blogs/blog/glinet-security-notice-2026-11-20). The quick `ps \| grep tmp` check has **limited coverage**, so a clean result does **not** mean a device is clean. The community check is now step 1. Firmware versions and recovery advice updated. |
| **22 Nov 2026** | First version. |

---

## ⚡ The short version

1. **[Run the community check](#-step-1-run-the-community-check)** before you update, restart or reset anything.
2. **[Install the security firmware](#-step-2-install-the-security-firmware)** for your exact model.
3. **[Act on your result](#-step-3-act-on-your-result)**: recover if anything was flagged, and change passwords and keys if the device was ever on GoodCloud.

---

## 📌 Links

| What | Link |
| --- | --- |
| Official GL.iNet security notice (updated 23 Nov) | https://www.gl-inet.com/en-us/blogs/blog/glinet-security-notice-2026-11-20 |
| GL.iNet COO statement (27 Nov) | https://www.reddit.com/r/GlInet/comments/1y77edg/a_message_from_glinet_coo/ |
| ⭐ Community security check | https://gl-i.net/security/csa-2026-01/fix/ |
| Community check source code | https://github.com/admonstrator/glinet-csa-2026-01 |
| Best steps for recovery (RemoteToHome) | https://github.com/RemoteToHome-io/gl-security-incident-2026 |
| Router firmware downloads | https://dl.gl-inet.com/ |
| KVM firmware downloads | https://dl.gl-inet.com/kvm |
| My full blog write-up | https://blog.kltechvideos.co.uk/2026/11/22/gl-inet-security-incident-how-to-check-your-routers-and-kvms-for-malware/ |

---

## 📋 What happened, in brief

An unauthorised party abused GL.iNet's cloud remote management (GoodCloud) to put malware on a subset of GL.iNet routers and KVMs. GL.iNet's updated notice names three issues:

- **GoodCloud message-authorisation flaw.** Fixed on GL.iNet's servers on 20 November.
- **AstroWarp / SD-WAN command injection on routers.** Applies **even with AstroWarp switched off**, so update anyway.
- **RTTY configuration and certificate-checking issue on KVMs.** RTTY is the remote terminal feature.

GL.iNet says devices that never connected to GoodCloud are **not likely exposed**. CVE numbers have been requested.

---

## ⭐ Step 1: run the community check

GL.iNet's notice links to this independently maintained check. It covers the gl-helper malware, the remote shell, cloud-command traces, running programs, network and firewall changes, replaced tools, autostart entries and the older backdoor.

> ⏱️ **Run it BEFORE you update, restart or reset.** A restart removes the remote shell, and the new firmware deletes known traces on first boot.

🟢 **Read-only** (downloads a script, then only reads). SSH in as root on a router or KVM:

```sh
curl -fsSL https://app.gl-i.net/csa-2026-01-fix -o gl-helper-triage.sh ; sh gl-helper-triage.sh
```

| Verdict | Meaning |
| --- | --- |
| **INFECTED** | Malware files, a remote shell, GL.iNet's indicator or the older backdoor is present |
| **ATTACKED** | A cloud command ran as root but left nothing behind. Treat the device as hit |
| **No gl-helper indicators found** | None of the known traces. Not proof nothing happened |

- A flagged item isn't automatically proof of infection. If anything shows up, contact **support@gl-inet.com**.
- `--archive` saves evidence to `/tmp` (copy it off before restarting). It can contain Wi-Fi and VPN keys, so **never post it**.
- `--fix` removes what it found. Read the [check's page](https://gl-i.net/security/csa-2026-01/fix/) before using either option.
- Not run or endorsed by GL.iNet. It runs as root, so give it a read first.

---

## 📦 Step 2: install the security firmware

Minimum versions with the fixes, from GL.iNet's notice. This lists models with security firmware available, **not** models confirmed as compromised.

| Type | Models | Minimum version |
| --- | --- | --- |
| Router | GL-MG1300, GL-MT3000, GL-MT6000 | **4.11.1** |
| Router | GL-E5800 | **4.10.3** |
| Router | GL-BE3600, GL-BE9300 | **4.10.2** |
| Router | GL-BE14000 | **4.9.3** |
| Router | GL-BE6500, GL-MT3600BE, GL-MT5000 | **4.9.1** |
| Router | GL-BE10000 | **4.8.7** |
| Router | GL-AX1800, GL-AXT1800, GL-SFT1200, GL-X3000, GL-XE3000 | **4.8.4** |
| Router | GL-MT2500 / MT2500A, GL-X2000 | **4.7.14** |
| KVM | GL-RM1, RM1V2, RM10, RM10RC, RM1PE, RM4PE | **1.10.3** |
| KVM | GL-RMQ1 (Comet Q) | **1.8.7** |

Install the latest stable release for **your exact model** only.

---

## ✅ Step 3: act on your result

**Never connected to GoodCloud and nothing flagged?**
Install the security firmware as routine maintenance. No reset or password change needed.

**Connected to GoodCloud, even with a clean check?**
Install the firmware, then change passwords and replace keys anyway (GL.iNet precaution).

**Known or suspected compromise:**

1. **Contain first.** If access looks ongoing, disconnect the device if safe and contact support@gl-inet.com.
2. **Reinstall the security firmware with settings OFF** (router: untick "Keep Settings"; KVM: clear "Save current configuration" where offered). Set it up again by hand and **don't restore a backup** from the device.
3. **A factory reset alone does not install the fixes.** Keeping settings is allowed but gives less assurance: in community testing a startup script and SSH key survived a settings-preserving upgrade on a test RM1.
4. **Rotate credentials:** admin password, VPN / WireGuard / SSH keys, DDNS and cloud passwords, and anything reused elsewhere. Delete VPN profiles at the other end and create new ones.
5. **KVM users:** check the connected computer for unknown accounts, software, browser extensions, startup items or settings changes, then run a full scan.

> 🛑 **U-Boot reflashing is for advanced users only. You can brick your device.** Only follow GL.iNet's [official debrick guide](https://docs.gl-inet.com/router/en/3/tutorials/debrick/). I can't help with this process.

---

## 🔒 Harden it: the real fix

- **Turn off GoodCloud / KVM cloud access** on anything that doesn't strictly need it. The attack came in through the cloud.
- **Use your own remote access** (WireGuard / Tailscale) instead of vendor cloud management.
- **Keep KVMs off the internet** and reach them over your VPN.
- **Keep firmware current.** The AstroWarp bug applies even with AstroWarp switched off.

---

## 🧰 Extra checks (optional)

These are GL.iNet's original quick checks plus my own. They look for fewer indicators than the community check, so use them **as well as** step 1, not instead of it.

<details>
<summary><strong>🔍 What the original indicator looks like</strong></summary>

One process: **`crond -i /tmp/.log`**

- If it's running, treat the device as **infected**.
- A line that only shows the `grep` command itself is **not** a match.
- `crond -f -c /tmp/gl_crontabs ...` is GL.iNet's normal scheduler, **not** the malware.

</details>

<details>
<summary><strong>📶 Quick check: routers</strong></summary>

> ⚠️ **Limited coverage.** A clean result does not mean the router is clean.

**Official command** 🟢 Read-only

```sh
ps | grep tmp
```

**My enhanced command** 🟢 Read-only. Shows the hostname, can't match itself, gives a plain verdict.

```sh
echo "== $(uci get system.@system[0].hostname 2>/dev/null) =="; ps | grep '[c]rond -i /tmp/.log' && echo "ALERT: INDICATOR FOUND - INFECTED" || echo "OK: indicator not found"; ls -la /tmp/.log 2>/dev/null
```

**No SSH? Use LuCI**

1. Admin Panel → **System → Advanced Settings** → sign in to **LuCI**
2. **Status → Processes**
3. Press **Ctrl+F** (Cmd+F on Mac) and search for `tmp/.log`
4. `crond -i /tmp/.log` = infected. No match = indicator not found.

</details>

<details>
<summary><strong>🖥️ Quick check: KVMs</strong></summary>

> ⚠️ GL.iNet calls this **optional, with limited coverage**. Install the security firmware regardless.

Open the KVM web interface: **Toolbox → Terminal → Access**. Run these **in the KVM's terminal**, not on the computer it's connected to.

**Official command: Comet Q (GL-RMQ1)** 🟢 Read-only

```sh
ps -a | grep tmp
```

**Official command: all other KVMs** 🟢 Read-only

```sh
ps -aux | grep tmp
```

**My enhanced command** 🟢 Read-only. On a Comet Q, change `ps -aux` to `ps -a`.

```sh
ps -aux 2>/dev/null | grep '[c]rond -i /tmp/.log' && echo "ALERT: INDICATOR FOUND - INFECTED" || echo "OK: indicator not found"
```

</details>

<details>
<summary><strong>🧪 Advanced: full indicator check (<code>glcheck</code>)</strong></summary>

🟢 **Read-only.** Works on routers and KVMs. Paste the whole block.

Checks for suspicious processes, RAM-only leftovers, implant marker files, a replaced `netstat`, persistence entries, connections to the reported IP range, and the leftover firewall rule.

> ⚠️ **Point-in-time check.** The remote shell and the `/tmp/f` pipe disappear after a reboot, so a clean result after a reboot proves less than one before.

```sh
glcheck() {
  hits=0
  flag() { echo "ALERT: $*"; hits=$((hits+1)); }
  echo "== $(cat /proc/sys/kernel/hostname) =="

  # 1. Processes: GL.iNet's indicator plus the reported remote shell
  for P in "ps -aux" "ps -a" "ps w" "ps"; do $P >/dev/null 2>&1 && break; done
  $P 2>/dev/null | grep -E '[c]rond -i /tmp/\.log|[m]kfifo|(^|[ /])[n]c [0-9]|/bin/[s]h -i|[s]ocat|[n]etcat|170\.39\.226' \
    && flag "suspicious process(es) listed above"

  # 2. RAM-only leftovers (gone after a reboot)
  [ -p /tmp/f ] && flag "/tmp/f named pipe exists - remote shell has run since boot"
  [ -e /tmp/.log ] && flag "/tmp/.log exists"

  # 3. Implant marker files
  for f in /etc/glinet/.cache/gl-helper /etc/glinet/.cache/gb /etc/config/.gl/gl-helper /etc/config/.gl/gb; do
    [ -e "$f" ] && flag "marker file: $f"
  done

  # 4. Replaced netstat - OpenWrt routers only (checks the file, never runs it)
  if [ -e /etc/openwrt_release ]; then
    for f in /bin/netstat /usr/bin/netstat; do
      [ -f "$f" ] && [ ! -L "$f" ] && flag "$f is a real file, not a busybox link"
    done
  fi

  # 5. Persistence: autostart, cron, init scripts, keep-on-upgrade lists
  r=$(grep -lE 'gl-helper|/etc/config/\.gl|glinet/\.cache|/tmp/\.log|170\.39\.226' \
      /etc/rc.local /etc/crontabs/* /etc/sysupgrade.conf /lib/upgrade/keep.d/* \
      /etc/init.d/* /etc/uci-defaults/* 2>/dev/null)
  [ -n "$r" ] && flag "implant reference in: $r"

  # 6. Connections to the reported range, read from kernel tables
  grep -q '170\.39\.226\.' /proc/net/nf_conntrack 2>/dev/null && flag "conntrack entry to 170.39.226.0/24"
  grep -qiE ' ([0-9A-F]{2}E227AA|AA27E2[0-9A-F]{2}):' /proc/net/tcp 2>/dev/null \
    && flag "TCP socket to 170.39.226.0/24"

  # 7. Firewall rule left behind by the remote-shell command (iptables or nftables)
  { iptables-save 2>/dev/null | grep SDWAN_IP_MAPPING | grep -q '127\.0\.0\.1'; } || \
  nft list ruleset 2>/dev/null | awk '/chain SDWAN_IP_MAPPING/{f=1;next} f&&/^[[:space:]]*}/{f=0} f&&/127\.0\.0\.1/{x=1} END{exit !x}' \
    && flag "SDWAN_IP_MAPPING rule pointing at 127.0.0.1"

  [ "$hits" -eq 0 ] && echo "OK: no indicators found (point-in-time check)" \
    || echo "ALERT: $hits indicator(s) - treat this device as compromised"
}
glcheck
```

</details>

<details>
<summary><strong>⚠️ KVM terminal says <code>bash: !!: event not found</code>?</strong></summary>

The KVM's Toolbox terminal runs **bash**, which treats `!!` as history expansion. Older copies of these commands used `!!!` in their alerts, so the command fails before it runs. That error is **not** a sign of infection.

The commands on this page no longer use `!!!`, so copy them again. If you're pasting an older copy, run this first in bash (not on the BusyBox RM1 shell, which rejects it):

```sh
set +H
```

</details>

---

## 🚫 The suspect IP range: `170.39.226.0/24`

GL.iNet **hasn't** published any IP addresses or domains to block. Users on the GL.iNet forum reported infected routers opening a remote shell back to **170.39.226.219**, and the community check watches the whole **/24** range.

> This comes from **user reports** and **hasn't been confirmed by GL.iNet**. You've got no legitimate reason to talk to that range, so blocking it costs you nothing. But an attacker can change IPs in minutes, so this isn't the real fix.

### Has anything talked to it?

🟢 **Read-only.** Run on a GL.iNet router over SSH.

```sh
grep '170\.39\.226\.' /proc/net/nf_conntrack 2>/dev/null && echo "CONNECTION TO SUSPECT RANGE FOUND" || echo "OK: no current connections to 170.39.226.0/24"
```

This reads the kernel's connection-tracking table instead of `netstat`, because the malware can replace `netstat` with a fake.

**Best check:** search your separate firewall's flow history (e.g. Firewalla) for `170.39.226`. An infected device can lie to you; your firewall won't.

### Block it

**Firewalla (in the app)**

1. Create a new **Block** rule.
2. Target: IP range `170.39.226.0/24`.
3. Apply to **all devices**, **both directions**.

**GL.iNet router** 🟠 **Changes things** (adds 2 firewall rules, then prints them back)

```sh
uci add firewall rule; uci set firewall.@rule[-1].name='Block-GL-IOC-router'; uci set firewall.@rule[-1].dest='wan'; uci set firewall.@rule[-1].dest_ip='170.39.226.0/24'; uci set firewall.@rule[-1].proto='all'; uci set firewall.@rule[-1].target='REJECT'; uci add firewall rule; uci set firewall.@rule[-1].name='Block-GL-IOC-lan'; uci set firewall.@rule[-1].src='*'; uci set firewall.@rule[-1].dest='wan'; uci set firewall.@rule[-1].dest_ip='170.39.226.0/24'; uci set firewall.@rule[-1].proto='all'; uci set firewall.@rule[-1].target='REJECT'; uci commit firewall; /etc/init.d/firewall reload; uci show firewall | grep -A6 'Block-GL-IOC'
```

The first rule stops the router itself reaching the range; the second stops devices behind it. Prefer clicking? Add the same rules in LuCI under **Network → Firewall → Traffic Rules**.

**Any other firewall:** block the range `170.39.226.0/24`.

---

*Community check: https://gl-i.net/security/csa-2026-01/fix/ (source: https://github.com/admonstrator/glinet-csa-2026-01). Independently maintained, linked from GL.iNet's notice, not run or endorsed by GL.iNet.*

**KL Tech Videos** · [YouTube](https://www.youtube.com/@kltechvideos) · [Blog](https://blog.kltechvideos.co.uk) · [X](https://x.com/kltechvideos) · [Ko-fi](https://www.ko-fi.com/kltechvideos)
