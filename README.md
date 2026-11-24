# GL.iNet Security Incident (20 Nov 2026): Check & Block Commands

Copy-and-paste commands to check GL.iNet routers and KVMs for the malware from GL.iNet's security notice of 20 November 2026, and to block the IP range reported by the community.

> **Command labels**
> 🟢 **Read-only**: only looks, changes nothing
> 🟠 **Changes things**: modifies your device's config

---

## 📌 Links

| What | Link |
| --- | --- |
| Best Steps for recovery | https://github.com/RemoteToHome-io/gl-security-incident-2026 |
| Official GL.iNet security notice (updated 23 Nov) | https://www.gl-inet.com/en-us/blogs/blog/glinet-security-notice-2026-11-20 |
| My blog post (full write-up) | https://blog.kltechvideos.co.uk/2026/11/22/gl-inet-security-incident-how-to-check-your-routers-and-kvms-for-malware/ |
| Router firmware download centre | https://dl.gl-inet.com/ |
| KVM firmware download centre | https://dl.gl-inet.com/kvm |
| U-Boot / debrick guide (advanced, see warning below) | https://docs.gl-inet.com/router/en/3/tutorials/debrick/ |
| ⭐ Community security check (linked by GL.iNet; independently maintained) | https://gl-i.net/security/csa-2026-01/fix/ |
| Community check source code | https://github.com/admonstrator/glinet-csa-2026-01 |

---

> 🆕 **Updated 24 Nov 2026:** GL.iNet updated its notice. The quick `ps | grep tmp` check has **limited coverage** (it mainly targets the KVM malware), so **a clean result does not mean a device is clean**. Run the community check below first, then install the security firmware.

## ⭐ Start here: community security check (recommended)

GL.iNet's updated notice links to this independently maintained check. It covers the gl-helper malware, the remote shell, cloud-command traces, running programs, network and firewall changes, replaced tools and autostart entries, and the older backdoor.

> ⏱️ **Run it BEFORE you update, restart or reset.** A restart removes the remote shell, and the new firmware deletes known traces on first boot.

**Check-only mode** 🟢 Read-only on your device (downloads a script). SSH in as root, on a router or KVM:

```sh
curl -fsSL https://app.gl-i.net/csa-2026-01-fix -o gl-helper-triage.sh ; sh gl-helper-triage.sh
```

| Verdict | Meaning |
| --- | --- |
| **INFECTED** | Malware files, a remote shell, GL.iNet's indicator or the older backdoor is present |
| **ATTACKED** | A cloud command ran as root but nothing was left behind. Treat the device as hit |
| **No gl-helper indicators found** | None of the known traces. Not proof nothing happened |

- A flagged item isn't automatically proof of infection. If anything shows up, contact support@gl-inet.com.
- `--archive` saves evidence to `/tmp` (copy it off before restarting; it can contain Wi-Fi and VPN keys, so never post it). `--fix` removes what it found. Read the [check's page](https://gl-i.net/security/csa-2026-01/fix/) before using either.
- Not run or endorsed by GL.iNet, and it runs as root, so give it a read first.

---

## 🔍 What you're looking for

One process: **`crond -i /tmp/.log`**

- If it's running, treat the device as **infected**.
- A line that only shows the `grep` command itself is **not** a match.
- `crond -f -c /tmp/gl_crontabs ...` is GL.iNet's normal scheduler. It's **not** the malware.

---

## 📶 GL.iNet's original quick check: routers

> ⚠️ **Limited coverage.** This only looks for one indicator. A clean result does **not** mean the router is clean. Use the community check above as well.

SSH into the router and run one of these.

**Official command** 🟢 Read-only

```sh
ps | grep tmp
```

**My enhanced command** 🟢 Read-only
Shows the hostname, can't match itself, and gives a plain INFECTED / OK verdict.

```sh
echo "== $(uci get system.@system[0].hostname 2>/dev/null) =="; ps | grep '[c]rond -i /tmp/.log' && echo "!!! INDICATOR FOUND - INFECTED" || echo "OK: indicator not found"; ls -la /tmp/.log 2>/dev/null
```

**No SSH? Use LuCI instead**

1. Admin Panel → **System → Advanced Settings** → sign in to **LuCI**
2. **Status → Processes**
3. Press **Ctrl+F** (Cmd+F on Mac) and search for `tmp/.log`
4. A line showing `crond -i /tmp/.log` = infected. No match = clean.

---

## 🖥️ GL.iNet's original quick check: KVMs

> ⚠️ GL.iNet calls this KVM check **optional, with limited coverage**. A negative result does not rule out compromise. Install the security firmware regardless.

In the KVM web interface: **Toolbox → Terminal → Access**.
Run these **in the KVM's terminal**, not on the computer it's connected to.

**Official command: Comet Q (GL-RMQ1)** 🟢 Read-only

```sh
ps -a | grep tmp
```

**Official command: all other KVM models** 🟢 Read-only

```sh
ps -aux | grep tmp
```

**My enhanced command** 🟢 Read-only
On a Comet Q, change `ps -aux` to `ps -a`.

```sh
ps -aux 2>/dev/null | grep '[c]rond -i /tmp/.log' && echo "INDICATOR FOUND - INFECTED" || echo "OK: indicator not found"
```

### ⚠️ KVM Toolbox terminal: the `!!` gotcha

The KVM's Toolbox → Terminal → Access runs a **full bash shell**, unlike the BusyBox `ash` on the routers. Bash treats `!!` as history expansion, so an alert message containing `!!!` fails before the check even runs:

```
bash: !!: event not found
```

That error is **not** a sign of infection — the command just never ran. Two fixes:

**Fix A — turn off history expansion first, then run the check** 🟢 Read-only

```sh
set +H
ps -aux 2>/dev/null | grep '[c]rond -i /tmp/.log' && echo "INDICATOR FOUND - INFECTED" || echo "OK: indicator not found"
```

**Fix B — use the `!!`-free command above** 🟢 Read-only

The **My enhanced command** box above already has the `!!!` removed, so it runs in the KVM Toolbox terminal as-is. On a Comet Q, change `ps -aux` to `ps -a`.

Either way you'll get `OK: indicator not found` (clean) or the INFECTED line.

> **Note:** the router commands run in BusyBox `ash`, which has no history expansion, so the `!!!` there is harmless. This gotcha only affects the KVM's bash terminal.

---

## 🧪 Advanced: full indicator check (`glcheck`)

🟢 **Read-only.** Works on routers and KVMs. Paste the whole block over SSH.

> **KVM Toolbox terminal:** run `set +H` once before pasting this block, so bash doesn't choke on the `!!!` inside the `flag` messages.

Checks for:

1. Suspicious processes
2. RAM-only leftovers
3. Implant marker files
4. A replaced `netstat`
5. Persistence entries
6. Connections to the reported IP range
7. The leftover firewall rule

> ⚠️ This is a **point-in-time** check. Some indicators (the remote shell and the `/tmp/f` pipe) disappear after a reboot, so a clean result after a reboot proves less than one before.

```sh
glcheck() {
  hits=0
  flag() { echo "!!! $*"; hits=$((hits+1)); }
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
    || echo "!!! $hits indicator(s) - treat this device as compromised"
}
glcheck
```

---

## 🚫 The suspect IP range: 170.39.226.0/24

GL.iNet **hasn't** published any IP addresses or domains to block. One range is being passed around in the community:

- Users on the GL.iNet forum reported infected routers opening a remote shell back to **170.39.226.219**.
- The community checker watches the whole range, **170.39.226.0/24**.

> This comes from **user reports** and **hasn't been confirmed by GL.iNet**. But you've got no legitimate reason to talk to that range, so blocking it costs you nothing.

### Step 1: has anything talked to it?

🟢 **Read-only.** Run on a GL.iNet router over SSH.

```sh
grep '170\.39\.226\.' /proc/net/nf_conntrack 2>/dev/null && echo "CONNECTION TO SUSPECT RANGE FOUND" || echo "OK: no current connections to 170.39.226.0/24"
```

This searches the router's connection-tracking table, which lists every connection going through or from the router right now. It deliberately avoids `netstat`, because community analysis found the malware can replace `netstat` with a fake. The connection table comes straight from the kernel, so it's much harder to fake.

**Best check:** if you have a separate firewall (e.g. Firewalla), search its flow history for `170.39.226`. An infected device can lie to you; your firewall won't.

### Step 2: block it

**Firewalla (in the app)**

1. Create a new **Block** rule.
2. Target: IP range `170.39.226.0/24`.
3. Apply to **all devices**.
4. Block **both directions**.

**GL.iNet router** 🟠 Changes things (adds 2 firewall rules)

The first rule stops the router itself reaching the range. The second stops anything on your network reaching it through the router. The last part prints both rules back so you can check they went in.

```sh
uci add firewall rule; uci set firewall.@rule[-1].name='Block-GL-IOC-router'; uci set firewall.@rule[-1].dest='wan'; uci set firewall.@rule[-1].dest_ip='170.39.226.0/24'; uci set firewall.@rule[-1].proto='all'; uci set firewall.@rule[-1].target='REJECT'; uci add firewall rule; uci set firewall.@rule[-1].name='Block-GL-IOC-lan'; uci set firewall.@rule[-1].src='*'; uci set firewall.@rule[-1].dest='wan'; uci set firewall.@rule[-1].dest_ip='170.39.226.0/24'; uci set firewall.@rule[-1].proto='all'; uci set firewall.@rule[-1].target='REJECT'; uci commit firewall; /etc/init.d/firewall reload; uci show firewall | grep -A6 'Block-GL-IOC'
```

Prefer clicking? You can add the same rules in LuCI under **Network → Firewall → Traffic Rules**.

**Any other firewall**

Create an IP range block rule for:

```
170.39.226.0/24
```

> **Honest caveat:** blocking one IP range isn't the real fix, because an attacker can change IPs in minutes. See below.

---

## ✅ The real fix

- **Turn off GoodCloud / KVM cloud access** on any device that doesn't strictly need it. The attack came in through GL.iNet's cloud.
- **Install the security firmware** for your exact model (see the version table above). The AstroWarp bug applies even with AstroWarp switched off.
- **Use your own remote access** (WireGuard / Tailscale) instead of vendor cloud management.
- **Keep KVMs off the internet** and reach them over your VPN.

---

## ⚠️ What to do with your result

**Never connected to GoodCloud?** GL.iNet says these devices are **not likely exposed**. Install the security firmware as routine maintenance. No reset or password change needed for the update prompt alone.

**Connected to GoodCloud?** Install the firmware, then change passwords and replace keys anyway, even with a clean check (GL.iNet precaution).

**Known or suspected compromise:**

1. **Contain first.** If access looks ongoing, disconnect the device if safe and contact support@gl-inet.com.
2. **Reinstall the security firmware with settings OFF** (router: "Keep Settings" off; KVM: clear "Save current configuration" where offered). Set it up again by hand. **Don't restore a backup** from the device. A factory reset alone does **not** install the fixes.
3. Keeping settings is allowed but gives **less recovery assurance**: in community testing a startup script and SSH key survived a settings-preserving upgrade on a test RM1.
4. **Rotate credentials** after updating: admin password, VPN / WireGuard / SSH keys, DDNS and cloud passwords, and anything reused elsewhere. Delete VPN profiles at the other end and create new ones.
5. **KVM users:** check the connected computer for unknown accounts, software, browser extensions, startup items or settings changes, then run a full scan.

### 📦 Minimum security firmware versions

| Type | Models | Minimum version |
| --- | --- | --- |
| Router | GL-MG1300, GL-MT3000, GL-MT6000 | 4.11.1 |
| Router | GL-E5800 | 4.10.3 |
| Router | GL-BE3600, GL-BE9300 | 4.10.2 |
| Router | GL-BE14000 | 4.9.3 |
| Router | GL-BE6500, GL-MT3600BE, GL-MT5000 | 4.9.1 |
| Router | GL-BE10000 | 4.8.7 |
| Router | GL-AX1800, GL-AXT1800, GL-SFT1200, GL-X3000, GL-XE3000 | 4.8.4 |
| Router | GL-MT2500 / MT2500A, GL-X2000 | 4.7.14 |
| KVM | GL-RM1, RM1V2, RM10, RM10RC, RM1PE, RM4PE | 1.10.3 |
| KVM | GL-RMQ1 (Comet Q) | 1.8.7 |

Install the latest stable release for **your exact model** only. This table lists models with security firmware, not confirmed compromised models.

> 🛑 **U-Boot reflashing is for advanced users only. You can brick your router or KVM.** Only follow GL.iNet's official guides. I can't assist with this process.
> https://docs.gl-inet.com/router/en/3/tutorials/debrick/

---

*Community check: https://gl-i.net/security/csa-2026-01/fix/ (source: https://github.com/admonstrator/glinet-csa-2026-01). Independently maintained, linked by GL.iNet's notice, not run or endorsed by GL.iNet. It runs as root, so read it before running it.*

**KL Tech Videos** · [YouTube](https://www.youtube.com/@kltechvideos) · [Blog](https://blog.kltechvideos.co.uk) · [X](https://x.com/kltechvideos) · [Ko-fi](https://www.ko-fi.com/kltechvideos)
