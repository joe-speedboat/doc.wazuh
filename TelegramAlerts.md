# HowTo get Telegram Alerts into group
* Create Group and api-bot. See `telegram_bot.md`
* copy `files/custom-telegram` to `/var/ossec/integrations/custom-telegram`
  `chown root:wazuh /var/ossec/integrations/custom-telegram`
  `chmod 750 /var/ossec/integrations/custom-telegram`
* Update `CHAT_ID` in `/var/ossec/integrations/custom-telegram`
* Add integration to wazuh config
```
<integration>
    <level>10</level>
    <name>custom-telegram</name>
    <hook_url>https://api.telegram.org/bot<INSERT-TOKEN-HERE>/sendMessage</hook_url>
    <alert_format>json</alert_format>
</integration>
```
* Restart manager: `systemctl restart wazuh-manager`

