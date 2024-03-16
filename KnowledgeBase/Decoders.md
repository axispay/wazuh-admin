# A turotial in how to configure any decoder file in Wazuh

## Is the decoder file a duplicate for a default decoder in ossec/ruleset ?
### If no skip step 1 and 2

### 1. On the Wazuh server exclude the default decoder in 'ossec.conf'

```bash
sudo nano /var/ossec/etc/ossec.conf
sudo vi /var/ossec/etc/ossec.conf
```

### 2. Search for 'ruleset' and add this line into the <ruleset> XML block
### **Note**: Change the '####-default_decoders.xml' into the default decoder name.

```xml
    <decoder_exclude>####-default_decoders.xml</decoder_exclude>
```

### Then restart the server to apply the changes

```bash
sudo systemctl restart wazuh.manager.service
```

### 3. On the Wazuh server make a decoder file

```bash
sudo nano /var/ossec/etc/decoders/custom-filename_decoders.xml
```

### 4. To test it go to

```bash
sudo /var/ossec/bin/wazuh-logtest
```

### and paste any log related to the custom decoder.
### If no error or warning are present do not restart the server until you fix them. 