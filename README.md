# GL.iNet Security Incident (20 Nov 2026): Check & Block Commands

Copy-and-paste commands to check GL.iNet routers and KVMs for the malware from GL.iNet's security notice of 20 November 2026, and to block the IP range reported by the community.

> **Command labels**
> 🟢 **Read-only**: only looks, changes nothing
> 🟠 **Changes things**: modifies your device's config

---

## 📌 Links

| What | Link |
| --- | --- |
| Official GL.iNet security notice | https://www.gl-inet.com/en-gb/blogs/blog/glinet-security-notice-2026-11-20 |
| My blog post (full write-up) | https://blog.kltechvideos.co.uk/2026/11/22/gl-inet-security-incident-how-to-check-your-routers-and-kvms-for-malware/ |
| Router firmware download centre | https://dl.gl-inet.com/ |
| KVM firmware download centre | https://dl.gl-inet.com/kvm |
| U-Boot / debrick guide (advanced, see warning below) | https://docs.gl-inet.com/router/en/3/tutorials/debrick/ |
| 3rd-party malware checker (not endorsed by me or GL.iNet) | https://github.com/admonstrator/glinet-csa-2026-01 |

---

## 🔍 What you're looking for

One process: **`crond -i /tmp/.log`**

- If it's running, treat the device as **infected**.
- A line that only shows the `grep` command itself is **not** a match.
- `crond -f -c /tmp/gl_crontabs ...` is GL.iNet's normal scheduler. It's **not** the malware.

---

## 📶 GL.iNet router check

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

## 🖥️ GL.iNet KVM check

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
ps -aux 2>/dev/null | grep '[c]rond -i /tmp/.log' && echo "!!! INDICATOR FOUND - INFECTED" || echo "OK: indicator not found"
```

---

## 🧪 Advanced: full indicator check (`glcheck`)

🟢 **Read-only.** Works on routers and KVMs. Paste the whole block over SSH.

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
grep '170\.39\.226\.' /proc/net/nf_conntrack 2>/dev/null && echo "!!! CONNECTION TO SUSPECT RANGE FOUND" || echo "OK: no current connections to 170.39.226.0/24"
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
- **Install the stable security firmware** for your model from the download centres above.
- **Use your own remote access** (WireGuard / Tailscale) instead of vendor cloud management.
- **Keep KVMs off the internet** and reach them over your VPN.

---

## ⚠️ If you're infected

- **GL.iNet's official advice:** install the security beta, or factory reset, then install the stable security firmware.
- If you update instead of resetting, **don't keep settings**. Community analysis found the malware hides in `/etc/config`.
- **Rotate every secret** the device held: Wi-Fi and admin passwords, VPN / WireGuard keys, DDNS credentials, SSH keys.
- **KVM users:** also check the connected computer for unknown programs, browser extensions, accounts, startup items or scheduled tasks, then run a full security scan.
- **Not sure?** Email support@gl-inet.com.

> 🛑 **U-Boot reflashing is for advanced users only. You can brick your router or KVM.** Only follow GL.iNet's official guides. I can't assist with this process.
> https://docs.gl-inet.com/router/en/3/tutorials/debrick/

---

*Third-party checker: https://github.com/admonstrator/glinet-csa-2026-01. It's community work, not endorsed by me or GL.iNet, and it runs as root, so read it before running it.*

**KL Tech Videos** · [YouTube](https://www.youtube.com/@kltechvideos) · [Blog](https://blog.kltechvideos.co.uk) · [X](https://x.com/kltechvideos) · [Ko-fi](https://www.ko-fi.com/kltechvideos)
