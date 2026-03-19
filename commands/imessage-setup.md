---
description: "Guided setup for the iMessage assistant. Checks prerequisites, configures your contact, and starts the daemon."
---

# iMessage Assistant Setup

Guide the user through a complete, interactive setup of the iMessage auto-reply assistant.

## Your Task

Walk through each step below in order. Be conversational and helpful. After each step that requires user input, wait for confirmation before proceeding.

---

## Step 1: Welcome

Greet the user and explain what the setup will do:

> "Welcome to the iMessage Assistant setup! I'll walk you through everything needed to get your iMessage auto-reply daemon running. This should only take a couple of minutes."
>
> "Here's what we'll do:
> 1. Check prerequisites
> 2. Configure the contact you want the assistant to respond to
> 3. Save your configuration
> 4. Start the daemon"

Ask them to confirm they're ready to proceed.

---

## Step 2: Check Prerequisites

Check each prerequisite and report status:

### 2a. macOS Check
Run: `sw_vers -productName`
- ✅ If "macOS" — good
- ❌ Otherwise — tell user this only works on macOS and stop

### 2b. Messages App Check
Run: `ls ~/Library/Messages/chat.db 2>/dev/null && echo "found" || echo "not found"`
- ✅ If found — good
- ⚠️ If not found — warn the user: "The Messages database wasn't found. Make sure you're signed in to iMessage in the Messages app. Open Messages > Settings > iMessage and sign in, then re-run setup."

### 2c. Full Disk Access Check
Run: `sqlite3 ~/Library/Messages/chat.db "SELECT count(*) FROM message LIMIT 1;" 2>/dev/null && echo "access_ok" || echo "access_denied"`
- ✅ If `access_ok` — good
- ❌ If `access_denied` — tell the user:
  > "Terminal needs Full Disk Access to read your Messages database. Please:
  > 1. Open **System Settings → Privacy & Security → Full Disk Access**
  > 2. Enable access for **Terminal** (or your terminal app)
  > 3. Re-run `/imessage-setup`"

  Then stop.

Report results to the user before continuing.

---

## Step 3: Configure Your Contact

Explain:
> "Now let's set up the contact whose messages the assistant will respond to. This should be the phone number or email you'll use to message your Mac."

Ask for the following one at a time:

### 3a. Phone Number
Ask: "What's the phone number (digits only, e.g. `4155551234`)?"
- Validate: must be digits only, 10-11 digits
- If invalid, ask again with a clear example

### 3b. Contact Name
Ask: "What's the name for this contact (as it appears in Messages, e.g. `John Doe`)?"

### 3c. iMessage Email (optional)
Ask: "Do you also use an email address for iMessage with this contact? (Press Enter to skip)"
- Accept blank/empty as "no"

### 3d. Check Interval (optional)
Ask: "How often should the daemon check for new messages? (in seconds, default: 1)"
- Accept blank as default (1)
- Validate: must be a positive integer

---

## Step 4: Save Configuration

Write the config file:

```bash
cat > ~/.claude-imessage.env << 'ENVEOF'
export IMESSAGE_CONTACT_PHONE="<phone>"
export IMESSAGE_CONTACT_NAME="<name>"
export IMESSAGE_CONTACT_EMAIL="<email_or_empty>"
export IMESSAGE_CHECK_INTERVAL="<interval>"
export IMESSAGE_TMP_DIR="$HOME/tmp/imessage"
ENVEOF
```

Replace placeholders with the collected values. Omit `IMESSAGE_CONTACT_EMAIL` line entirely if the user skipped it.

After writing, confirm:
> "Configuration saved to `~/.claude-imessage.env`."

Show a summary of what was saved (mask the phone number partially, e.g. `***1234`).

---

## Step 5: Create Log Directory

Run:
```bash
mkdir -p ~/tmp/imessage
```

---

## Step 6: Start the Daemon

Ask: "Everything looks good! Ready to start the daemon now?"

If yes:
1. Source the config: `source ~/.claude-imessage.env`
2. Find the daemon script path — it's installed with the plugin at:
   `~/.claude/skills/imessage/daemon/imessage-auto-reply-daemon.sh`
3. Make it executable: `chmod +x ~/.claude/skills/imessage/daemon/imessage-auto-reply-daemon.sh`
4. Start it: `nohup ~/.claude/skills/imessage/daemon/imessage-auto-reply-daemon.sh > ~/tmp/imessage/imessage-auto-reply.log 2>&1 &`
5. Wait 2 seconds, then check it's running: `ps aux | grep imessage-auto-reply-daemon | grep -v grep`

If running:
> "✅ The iMessage daemon is running! Send a message from your configured contact and Claude will respond automatically.
>
> Useful commands:
> - `/imessage-daemon status` — Check status and view logs
> - `/imessage-daemon stop` — Stop the daemon
> - `/imessage-daemon start` — Restart the daemon"

If not running:
> "⚠️ The daemon didn't start. Let's check the log:"

Run: `tail -20 ~/tmp/imessage/imessage-auto-reply.log`

Show the log output and help the user diagnose the issue.

---

## Important Notes

- Be friendly and patient throughout — not all users are technical
- If any step fails, explain clearly what went wrong and how to fix it
- Never proceed past a failed prerequisite check
- Always confirm before writing files or starting processes
