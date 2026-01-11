# CheckMK Raw Edition: Custom Host Attributes in Notifications

## The Problem

**CheckMK Raw Edition** uses **Nagios core**, which does not automatically pass custom host attributes to notification scripts.

### Symptoms:
- You created custom attributes with `add_custom_macro: True` ✅
- Notification script doesn't receive your variables ❌
- `NOTIFY_HOST_*` variables are empty in the script ❌

### Root Cause:

The `check_mk_templates.cfg` file is missing the lines needed to pass your custom variables to the environment in the `check-mk-notify` command.

---

## Solution

### 1. Edit the template:

```bash
nano ~/etc/nagios/conf.d/check_mk_templates.cfg
```

### 2. Find the `check-mk-notify` command and add your variables:

```bash
define command {
    command_name    check-mk-notify
    command_line    # ... existing variables ... \
                    NOTIFY_HOST_TELEGRAM_CHAT_ID='$_HOSTTELEGRAM_CHAT_ID$' \
                    NOTIFY_HOST_TELEGRAM_THREAD_ID='$_HOSTTELEGRAM_THREAD_ID$' \
                    NOTIFY_HOST_ISP_NAME='$_HOSTISP_NAME$' \
                    NOTIFY_HOST_ISP_CONTRACT='$_HOSTISP_CONTRACT$' \
                    NOTIFY_HOST_ISP_PHONE='$_HOSTISP_PHONE$' \
                    NOTIFY_HOST_LOCATION='$_HOSTLOCATION$' \
                    # ... rest of the command
}
```

### 3. Apply changes:

```bash
cmk -O
# or
omd restart nagios
```

---

## Variable Naming Convention

| Custom Attribute in CheckMK | Nagios Macro                 | Environment Variable              |
|-----------------------------|------------------------------|-----------------------------------|
| `telegram_chat_id`          | `$_HOSTTELEGRAM_CHAT_ID$`    | `NOTIFY_HOST_TELEGRAM_CHAT_ID`    |
| `isp_name`                  | `$_HOSTISP_NAME$`            | `NOTIFY_HOST_ISP_NAME`            |
| `location`                  | `$_HOSTLOCATION$`            | `NOTIFY_HOST_LOCATION`            |

**Pattern:** 

```
attribute_name → $_HOSTATTRIBUTE_NAME$ → NOTIFY_HOST_ATTRIBUTE_NAME
```

---

## Why Does This Happen?

| Edition | Behavior |
|---------|----------|
| **CheckMK Enterprise Edition (CMC)** | ✅ Automatically passes all custom attributes with `add_custom_macro: True` |
| **CheckMK Raw Edition (Nagios core)** | ❌ Requires **manual addition** of each attribute to `check_mk_templates.cfg` |

This is documented but **easy to miss** and not obvious.

---

## Debugging

Add this to your notification script to see all available variables:

```bash
env | grep "^NOTIFY_" | sort >> /tmp/notify_debug.log
```

If your `NOTIFY_HOST_*` variables are empty — the problem is in `check_mk_templates.cfg`.

---

## References

- [CheckMK Custom Host Attributes](https://docs.checkmk.com/latest/en/wato_hosts.html#custom_attributes)
- [Nagios Macros](https://assets.nagios.com/downloads/nagioscore/docs/nagioscore/3/en/macrolist.html)

---

**Tested on:** CheckMK Raw 2.1.x, 2.2.x, 2.3.x with Nagios Core 3.5.x

## License

MIT
