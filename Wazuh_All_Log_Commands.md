# Linux Command Logging with Rsyslog and Wazuh

## Linux configuration

### 1. Update bash configuration

```bash
echo 'export PROMPT_COMMAND='\''PREV_CMD=$(history 1); RETRN_VAL=$?; if [ "$PREV_CMD" != "$LAST_CMD" ]; then logger -p local6.debug -t Bash_History "$(srcuser) : $(dstuser) : $(pwd) - $(echo "$PREV_CMD")"; fi; LAST_CMD="$PREV_CMD"'\''' | sudo tee -a /etc/bash.bashrc > /dev/null
```

### 2. Create rsyslog configuration file and add the following line to specify the log destination

```bash
sudo sh -c 'echo "local6.* /var/log/commands.log" > /etc/rsyslog.d/bash.conf'
```
### 3. Restart Rsyslog

```bash
sudo systemctl restart rsyslog
```
### 4. Make a log rotation process to archive the archived logs (Edit it as you wish)

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
### 5. Configure Wazuh Agent ossec.conf

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

### 6. Configure the decoder at the Wazuh Manager

```bash
sudo nano /var/ossec/etc/decoders/local_decoder.xml
```
### Add the following XML block into an empty section:

```xml
<decoder name="Bash_History">
  <program_name>^Bash_History</program_name>
</decoder>

<decoder name="Bash_History">
  <parent>Bash_History</parent>
  <regex>^\s*(\S+)\s*:</regex>
  <order>logname</order>
</decoder>

<decoder name="Bash_History">
  <parent>Bash_History</parent>
  <regex>:\s(\S+)\s:</regex>
  <order>whoami</order>
</decoder>

<decoder name="Bash_History">
  <parent>Bash_History</parent>
  <regex>:\s+(/\.+)\s+-</regex>
  <order>pwd</order>
</decoder>

 <decoder name="Bash_History">
  <parent>Bash_History</parent>
  <regex>-\s*(\.+)$</regex>
  <order>command</order>
</decoder>
```
### 7. Configure the rule for the Wazuh Manager:

```bash
sudo nano /var/ossec/etc/rules/local_rules.xml
```

### Add the following XML block into an empty section:

```xml
<group name="bash">
  <rule id="100010" level="3">
    <program_name>^Bash_History</program_name>
    <description>Bash History Logging</description>
  </rule>
</group>
```

### 8. Test the rule with wazuh-logset

### Copy a raw log from the Wazuh Agent to test it:

```bash
sudo tail /var/log/commands.log
```
### The output of the logs should return like this:

```
Jan 12 21:14:49 suricata Bash_History: sukuna : root : /var/lib/suricata/rules -   270  cd rules/
Jan 12 21:14:50 suricata Bash_History: sukuna : root : /var/lib/suricata/rules -   271  ls
Jan 13 00:56:58 suricata Bash_History: sukuna : root : /var/lib/suricata/rules -   272  nano suricata.rules
Jan 13 00:57:13 suricata Bash_History: sukuna : root : /var/lib/suricata/rules -   273  tail /var/log/commands.log
Jan 13 00:59:16 suricata Bash_History: sukuna : root : /var/lib/suricata/rules -   274  nano /etc/bash.bashrc
Jan 13 01:00:46 suricata Bash_History: sukuna : root : /var/lib/suricata/rules -   275  systemctl start suricata.service
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
Jan 12 21:14:34 suricata Bash_History: sukuna : root : /var/lib/suricata -   265  cd /var/lib/suricata/

**Phase 1: Completed pre-decoding.
        full event: 'Jan 12 21:14:34 suricata Bash_History: sukuna : root : /var/lib/suricata -   265  cd /var/lib/suricata/'
        timestamp: 'Jan 12 21:14:34'
        hostname: 'suricata'
        program_name: 'Bash_History'

**Phase 2: Completed decoding.
        name: 'Bash_History'
        command: '   265  cd /var/lib/suricata/'
        logname: 'sukuna'
        pwd: '/var/lib/suricata'
        whoami: 'root'

**Phase 3: Completed filtering (rules).
        id: '100010'
        level: '3'
        description: 'Bash History Logging'
        groups: '['bash']'
        firedtimes: '1'
        mail: 'False'
**Alert to be generated.
```
### 9. Restart the wazuh-manager

```bash
sudo systemctl restart wazuh-manager
```

### Now check the Wazuh Dashboard discover logs and it should come out with every normal command:
![Alt text](./assets/image.png)

## Feel free to adjust the instructions based on your specific requirements or preferences. Thank you.

