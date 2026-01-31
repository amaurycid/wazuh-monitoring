# Wazuh Security Monitoring Areas

Setting up an **Active Response** is the "XDR" part of Wazuh—it moves you from passive observation to active defense.

The most common use case is a **shun script**, which automatically interacts with the server's firewall (like `iptables`, `nftables`, or Windows Firewall) to drop traffic from a malicious source.

---

## Step 1: Define the Command

On your **Wazuh Manager**, you first need to tell the system _what_ action to take. Open `/var/ossec/etc/ossec.conf` and ensure the "firewall-drop" command is defined (it usually is by default).

```xml
<command>
  <name>firewall-drop</name>
  <executable>firewall-drop</executable>
  <timeout_allowed>yes</timeout_allowed>
</command>

```

## Step 2: Configure the Active Response Rule

Now, you link that command to a specific trigger. In the same file, add this block:

```xml
<active-response>
  <command>firewall-drop</command>
  <location>local</location>
  <rules_id>100005, 5712</rules_id> <timeout>1800</timeout> </active-response>

```

## Step 3: How it Works in Practice

Once you restart the Wazuh Manager, the workflow looks like this:

1. **Detection:** An attacker tries to upload a `.php` file (Rule `100005`) or fails SSH login multiple times (Rule `5712`).
2. **Trigger:** The Manager sees the alert level is high enough and matches your Active Response config.
3. **Action:** The Manager sends a message to the specific Agent.
4. **Execution:** The Agent runs the `firewall-drop` script, adding the attacker's IP to the local firewall blocklist for 30 minutes.

---

### Important Considerations

- **The "Foot-Gun" Warning:** Be careful when applying this to internal IP ranges. You don't want to accidentally lock yourself out of your own server because you mistyped a password three times!
- **Whitelisting:** You can add a `<white_list>` section in your global configuration to protect your admin workstation IPs from ever being blocked.
- **Testing:** Test this with a non-critical rule first to ensure the firewall commands are executing correctly on your specific OS.
