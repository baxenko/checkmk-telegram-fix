# checkmk-telegram-fix
# CheckMK Raw Edition: Custom Host Attributes in Notifications

## The Problem

**CheckMK Raw Edition** uses **Nagios core**, which does not automatically pass custom host attributes to notification scripts.

### Symptoms:
- You created custom attributes with `add_to_custom_macro: True` ✅
- Notification script doesn't receive your variables ❌
- `NOTIFY_HOST_*` variables are empty in the script ❌

### Root Cause:
The `check_mk_templates.cfg` file is missing the lines needed to pass your custom variables to the environment in the `check-mk-notify` command.

---

## Solution

### Edit the template:

```bash
nano ~/etc/nagios/conf.d/check_mk_templates.cfg
