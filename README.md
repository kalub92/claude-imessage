# Claude iMessage

Turn Claude Code into an autonomous iMessage assistant that monitors your messages and responds automatically using the full power of Claude Code.

## What is this?

This plugin adds iMessage capabilities to Claude Code, allowing you to:

1. **Read and send iMessages** via command-line tools
2. **Run a daemon** that monitors incoming messages from a specific contact
3. **Trigger autonomous agent sessions** where Claude can work on tasks, check for follow-up messages, and send multiple replies
4. **Use the full Claude Code ecosystem** - the agent has access to all your skills, tools, and resources

The daemon creates a continuous conversation thread with Claude Code over iMessage, making it accessible from iPhone, iPad, Mac, or Apple Watch.

## Features

### iMessage Skill

The plugin includes a comprehensive iMessage skill with tools for:

- **Reading messages**: Access message history and conversation context via SQLite database
- **Sending messages**: Send to contacts, phone numbers, or group chats
- **Checking new messages**: Monitor for incoming messages
- **File attachments**: Send and receive files, images, documents
- **Conversation management**: List conversations, get chat identifiers

See [`skills/imessage/SKILL.md`](skills/imessage/SKILL.md) for complete documentation.

### Auto-Reply Daemon

The daemon script (`daemon/imessage-auto-reply-daemon.sh`) monitors iMessages and automatically:

1. Detects new messages from a configured contact
2. Starts an autonomous Claude Code agent session
3. Sends multiple messages as it works on tasks
4. Checks for new messages every 30-60 seconds
5. Maintains conversation continuity across sessions

The agent has access to all Claude Code capabilities including other skills, tools, and can perform complex multi-step tasks autonomously.

## Installation

### Prerequisites

- macOS with Messages app signed in to iMessage
- Claude Code installed ([get Claude Code](https://code.claude.com))
- Full Disk Access permission for Terminal (System Preferences > Security & Privacy > Privacy > Full Disk Access)

### Install as Claude Code Plugin

1. **Install the plugin from GitHub:**

```bash
# In any Claude Code session
/plugin marketplace add dvdsgl/claude-imessage
/plugin install imessage@dvdsgl
```

2. **Restart Claude Code** to load the plugin

3. **Verify installation:**

```bash
# Check that the imessage skill is available
claude -p "Use the imessage skill to list my recent conversations"
```

The iMessage skill is now available in all your Claude Code sessions!

### Alternative: Manual Installation

If you prefer to install manually or want to modify the code:

1. **Clone the repository:**

```bash
git clone https://github.com/dvdsgl/claude-imessage.git ~/Developer/claude-imessage
```

2. **Add as a local skill** in your Claude Code project:

```bash
# In your project's .claude/settings.json, add:
{
  "skills": [
    {
      "path": "~/Developer/claude-imessage/skills/imessage"
    }
  ]
}
```

## Quick Start

### Using the iMessage Skill

Once installed, you can use iMessage tools in any Claude Code conversation:

```bash
# Read recent messages from a contact
claude -p "Use the imessage skill to read my recent messages from 4155551234"

# Send a message
claude -p "Use the imessage skill to send a message to 4155551234 saying 'Hello from Claude!'"

# Check for new messages
claude -p "Use the imessage skill to check if I have any new messages"
```

### Running the Auto-Reply Daemon

The daemon monitors iMessages and creates autonomous agent sessions. This is perfect for having Claude as an always-available assistant via iMessage.

#### 1. Configure the Daemon

Create a `.env` file with your configuration:

```bash
# Copy the example
cp ~/Developer/claude-imessage/examples/.env.example ~/.claude-imessage.env

# Edit with your details
nano ~/.claude-imessage.env
```

Required environment variables:

```bash
# Contact to monitor (required)
IMESSAGE_CONTACT_PHONE="4155551234"  # Phone number without + or dashes
IMESSAGE_CONTACT_NAME="John Doe"     # Display name

# Optional settings
IMESSAGE_CONTACT_EMAIL="john@example.com"  # If they use email for iMessage
IMESSAGE_CHECK_INTERVAL="1"                 # Check every N seconds (default: 1)
IMESSAGE_TMP_DIR="$HOME/tmp/imessage"      # Where to store logs and state
```

#### 2. Start the Daemon

```bash
# Load your configuration
source ~/.claude-imessage.env

# Start the daemon
nohup ~/Developer/claude-imessage/daemon/imessage-auto-reply-daemon.sh > /dev/null 2>&1 &
```

#### 3. Monitor the Daemon

```bash
# Check if daemon is running
ps aux | grep imessage-auto-reply-daemon

# View logs
tail -f ~/tmp/imessage/imessage-auto-reply.log
tail -f ~/tmp/imessage/imessage-agent.log

# Stop the daemon
pkill -f imessage-auto-reply-daemon
```

#### 4. Send a Message

Send an iMessage to the phone number configured in the daemon from your other phone, and watch Claude respond autonomously!

## How It Works

### Agent Workflow

When the daemon receives a message, it:

1. **Starts a Claude Code agent** with your message as the prompt
2. **Provides full context**: Recent conversation history, available skills/tools
3. **Runs autonomously**: The agent can:
   - Send multiple messages as it works
   - Check for new messages from you
   - Use any Claude Code skills (calendar, files, APIs, etc.)
   - Break down complex tasks into steps
   - Ask for clarification when needed
4. **Maintains continuity**: Uses conversation resume to maintain context across sessions

### Conversation Continuity

The daemon uses Claude Code's conversation resume feature to maintain a continuous thread:

- First message starts a new conversation
- Subsequent messages resume the same conversation ID
- Full history is available across all agent sessions
- Save the conversation ID to `~/tmp/imessage/imessage_claude_conversation_id.txt` to persist across daemon restarts

### Example Interaction

```
You: "What's on my calendar today?"
Claude: "Let me check your calendar..."
Claude: "You have 3 events today:
        - 9am: Team standup
        - 2pm: Product review
        - 4pm: 1:1 with Sarah"

You: "Remind me to prepare slides before the product review"
Claude: "I'll create a reminder..."
Claude: "Done! I've added a reminder for 1:30pm to prepare slides for your product review."
```

## Configuration

### Environment Variables

All configuration is via environment variables in your `.env` file:

| Variable | Required | Description |
|----------|----------|-------------|
| `IMESSAGE_CONTACT_PHONE` | Yes | Phone number to monitor (digits only) |
| `IMESSAGE_CONTACT_NAME` | Yes | Display name of contact |
| `IMESSAGE_CONTACT_EMAIL` | No | Email if they use iMessage email |
| `IMESSAGE_CHECK_INTERVAL` | No | Check interval in seconds (default: 1) |
| `IMESSAGE_TMP_DIR` | No | Directory for logs and state (default: ./tmp) |

### Customizing Agent Behavior

The daemon script (`daemon/imessage-auto-reply-daemon.sh`) contains the agent prompt that defines Claude's behavior. You can customize:

- Communication style and tone
- Available skills and tools
- Task handling approach
- Progress update frequency
- Error handling behavior

Edit the `agent_prompt` variable in the daemon script to customize.

## Architecture

### Directory Structure

```
claude-imessage/
├── .claude-plugin/
│   └── plugin.json          # Plugin metadata for Claude Code
├── skills/
│   └── imessage/
│       ├── SKILL.md         # Skill documentation
│       ├── send-message.sh  # Send messages via AppleScript
│       ├── send-to-chat.sh  # Send to group chats
│       ├── read-messages-db.sh      # Read message history from DB
│       ├── check-new-messages-db.sh # Check for new messages
│       ├── list-conversations.sh    # List all conversations
│       ├── send-file.sh     # Send file attachments
│       └── get-message-attachments.sh  # Retrieve attachments
├── daemon/
│   ├── imessage-auto-reply-daemon.sh  # Auto-reply daemon
│   └── README.md            # Daemon documentation
├── examples/
│   └── .env.example         # Example configuration
└── README.md
```

### How Messages Work

The skill uses two approaches:

1. **SQLite Database** (reading): Directly reads from `~/Library/Messages/chat.db`
   - Fast and reliable
   - No permission issues
   - Access to full message history
   - Extracts text from both incoming and outgoing messages

2. **AppleScript** (sending): Uses Messages app automation
   - Send to contacts or phone numbers
   - Send to group chats by chat identifier
   - Support for file attachments
   - Works with both iMessage and SMS

## Troubleshooting

### Permission Issues

If you get permission errors:

1. Grant Full Disk Access to Terminal:
   - System Preferences > Security & Privacy > Privacy > Full Disk Access
   - Add Terminal.app (or your terminal emulator)
   - Restart your terminal

2. Messages app permissions:
   - System Preferences > Security & Privacy > Privacy > Automation
   - Ensure Terminal can control Messages

### Daemon Not Responding

If the daemon isn't responding to messages:

1. Check if it's running: `ps aux | grep imessage-auto-reply-daemon`
2. Check logs: `tail -f ~/tmp/imessage/imessage-auto-reply.log`
3. Verify environment variables are set: `echo $IMESSAGE_CONTACT_PHONE`
4. Test the skill manually: `./skills/imessage/check-new-messages-db.sh`

### Messages Not Sending

If Claude can't send messages:

1. Verify Messages app is signed in to iMessage
2. Test sending manually: `./skills/imessage/send-message.sh "YOUR_PHONE" "test"`
3. Check AppleScript permissions (see Permission Issues above)
4. Try using the phone number instead of contact name

## Advanced Usage

### Multiple Contacts

To monitor multiple contacts, run multiple daemon instances with different configurations:

```bash
# Terminal 1
export IMESSAGE_CONTACT_PHONE="4155551234"
export IMESSAGE_CONTACT_NAME="Alice"
~/Developer/claude-imessage/daemon/imessage-auto-reply-daemon.sh

# Terminal 2
export IMESSAGE_CONTACT_PHONE="4155555678"
export IMESSAGE_CONTACT_NAME="Bob"
~/Developer/claude-imessage/daemon/imessage-auto-reply-daemon.sh
```

### Integrating with Other Skills

The iMessage agent has access to all your Claude Code skills. You can:

- Check calendars via iMessage
- Create tasks and notes
- Query APIs and databases
- Run file operations
- Send emails via other skills
- Any other Claude Code capability

Just make sure those skills are available in your Claude Code environment.

### Conversation Persistence

To maintain conversation history across daemon restarts:

1. After the first message, check the logs for the conversation ID
2. Save it to a file: `echo "conv_abc123" > ~/tmp/imessage/imessage_claude_conversation_id.txt`
3. The daemon will automatically resume this conversation for all future messages

## Security Considerations

- The daemon has full access to your iMessage database and can read all messages
- Claude Code can send messages on your behalf
- Environment variables may contain sensitive phone numbers
- Logs may contain message content
- Grant permissions carefully and only run the daemon in trusted environments

## Contributing

Contributions are welcome! Please feel free to submit issues, feature requests, or pull requests.

### Development Setup

1. Fork and clone the repository
2. Make changes to the skill or daemon
3. Test locally using the manual installation method
4. Submit a pull request

## License

MIT License - see LICENSE file for details

## Credits

Created by David Siegel ([@dvdsgl](https://github.com/dvdsgl))

Powered by [Claude Code](https://code.claude.com) from Anthropic

## Support

- GitHub Issues: [dvdsgl/claude-imessage/issues](https://github.com/dvdsgl/claude-imessage/issues)
- Claude Code Docs: [code.claude.com/docs](https://code.claude.com/docs)

---

**Note**: This plugin requires macOS and is designed for personal use. Be mindful of privacy when using automated message responses.
