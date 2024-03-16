# A turotial in how to configure any CDB list in Wazuh

### 1. On the Wazuh server make a file

```bash
sudo nano /var/ossec/etc/lists/list-file
sudo vi /var/ossec/etc/lists/list-file
```

### 2. Add a key and a value (optional and can be repeated) sperated by a column in each row. For example:

```txt
    key1:value1
    key2:value2
    key3:value2
```
### If you need to include the : character as part of the key, such as with MAC addresses, you must escape the complete key using quotation marks. For example:

```txt
    "a0:a0:a0:a0:a0:a0":
    "b1:b1:b1:b1:b1:b1":
```

### 3. To add this list, you must define it in the ossec.conf file

```bash
sudo nano /var/ossec/etc/ossec.conf
```

### Then add the list location inside the 'ruleset' block

```xml
<ossec_config>
  <ruleset>
    <list>etc/lists/list-file</list>
```

### 4. Restart the server

```bash
sudo systemctl restart wazuh.manager.service
```