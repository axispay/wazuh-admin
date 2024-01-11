# Linux Command Logging with Rsyslog and Wazuh

## Linux Configuration

### 1. Update Bash Configuration

```bash
echo 'export PROMPT_COMMAND="RETRN_VAL=$?;logger -t LinuxCommandsWazuh -p local6.debug \"User \$(whoami) [$$]: \$(history 1 | sed \"s/^[ ]*[0–9]\+[ ]*//\" )\""' | sudo tee -a /etc/bash.bashrc
```

### 2. Create Rsyslog Configuration File

```bash
sudo sh -c 'echo "local6.* /var/log/commands.log" > /etc/rsyslog.d/bash.conf'
```
### 3. Modify Rsyslog Default Configuration

```bash
sudo sed -i 's/\*.*;auth,authpriv.none -\/var\/log\/syslog/\*.*;auth,authpriv.none,local6.none -\/var\/log\/syslog/' /etc/rsyslog.d/50-default.conf
```

### 4. Restart Rsyslog

```bash
sudo systemctl restart rsyslog
```
### 5. Log Rotation

```bash
    sudo sh -c 'echo "/var/log/commands.log {
    daily
    rotate 7
    compress
    delaycompress
    missingok
    notifempty
}" > /etc/logrotate.d/rsyslog'
```
### 6. Configure Wazuh Agent ossec.conf

```bash
sudo nano /var/ossec/etc/ossec.conf
```
### Add the following XML block within the <ossec_config> section:

```xml
  <localfile>
     <location>/var/log/commands.log</location>
     <log_format>syslog</log_format>
  </localfile>
```

### Restart the wazuh-agent:

```bash
sudo systemctl restart wazuh-agent
```

### 7. Configure the decoder at the Wazuh Manager

```bash
sudo nano /var/ossec/etc/decoders/local_decoder.xml
```
### Add the following XML block into an empty section:

```xml
<decoder name="Linux-commands">
    <program_name>^LinuxCommandsWazuh</program_name>
</decoder>

<decoder name="Linux-commands1">
    <parent>Linux-commands</parent>
    <regex>User (\w+) [\d+]: (\.+)</regex>
    <order>User, Command</order>
</decoder>
```
### 8. Configure the rule for the Wazuh Manager:

```bash
sudo nano /var/ossec/etc/rules/local_rules.xml
```

### Add the following XML block into an empty section:

```xml
<rule>
    <group name="Linux-commands">
        <rule id="100002" level="3">
            <program_name>LinuxCommandsWazuh</program_name>
            <description>Command: "$(Command)" executed by $(User) in $(hostname)</description>
            <group>syslog, local</group>
        </rule>
    </group>
</rule>
```

### 9. Test the rule with wazuh-logset

### Copy a raw log from the Wazuh Agent to test it:

```bash
sudo tail /var/log/commands.log
```
### The output of the logs should resturn like this:

```
Jan 11 16:07:15 suricata LinuxCommandsWazuh: User root [4404]:   117  sudo sh -c 'echo "/var/log/commands.log {#012    daily#012    rotate 7#012    compress#012    delaycompress#012    missingok#012    notifempty#012}" > /etc/logrotate.d/rsyslog'
Jan 11 16:10:42 suricata LinuxCommandsWazuh: User root [4404]:   118  nano /var/ossec/etc/ossec.conf
Jan 11 16:10:53 suricata LinuxCommandsWazuh: User root [4404]:   119  systemctl restart wazuh-agent #Copy this one for example
Jan 11 16:11:18 suricata LinuxCommandsWazuh: User root [4404]:   120  journalctl -xeu wazuh-agent
```

### Run `wazuh-logtest` to simulate a log entry and check if the decoder and rule are working as expected:

```bash
sudo bash /var/ossec/bin/wazuh-logtest
```
### Paste the log you have entered to intiate the log test:

```
Jan 11 16:10:53 suricata LinuxCommandsWazuh: User root [4404]:   119  systemctl restart wazuh-agent
```

### The result should come out like this:

```
**Phase 1: Completed pre-decoding.
        full event: 'Jan 11 16:14:19 suricata LinuxCommandsWazuh: User root [4404]:   122  systemctl restart wazuh-agent'
        timestamp: 'Jan 11 16:14:19'
        hostname: 'suricata'
        program_name: 'LinuxCommandsWazuh'

**Phase 2: Completed decoding.
        name: 'Linux-commands'
        Command: '  122  systemctl restart wazuh-agent'
        User: 'root'

**Phase 3: Completed filtering (rules).
        id: '100002'
        level: '3'
        description: 'Command: "  122  systemctl restart wazuh-agent" executed by root in suricata'
        groups: '['Linux-commandssyslog', ' local']'
        firedtimes: '1'
        mail: 'False'
**Alert to be generated.
```
### 10. Restart the wazuh-manager:

```bash
sudo systemctl restart wazuh-manager
```

### Now check the Wazuh Dashboard discover logs and it should come out with every normal command:
![Alt text](.image.png)

## Feel free to adjust the instructions based on your specific requirements or preferences. Thank you.

