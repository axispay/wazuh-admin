# A turotial in how to configure any rule file in Wazuh

## Is the rule file a duplicate for a default rule in ossec/ruleset ?
### If no skip step 1 and 2

### 1. On the Wazuh server exclude the default rule in 'ossec.conf'

```bash
sudo nano /var/ossec/etc/ossec.conf
sudo vi /var/ossec/etc/ossec.conf
```

### 2. Search for 'ruleset' and add this line into the <ruleset> XML block
### **Note**: Change the '####-rule.xml' into the default rule name.

```xml
    <rule_exclude>####-default_rule.xml</rule_exclude>
```

### Then restart the server to apply the changes

```bash
sudo systemctl restart wazuh.manager.service
```

### 3. On the Wazuh server make a rule file

```bash
sudo nano /var/ossec/etc/rules/custom-filename_rules.xml
```

### 4. To test it go to

```bash
sudo /var/ossec/bin/wazuh-logtest
```

### and paste any log related to the custom rule.
### If no error or warning are present do not restart the server until you fix them. 