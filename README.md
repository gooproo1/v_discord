# `v-discord` (Ultra-Simple Edition) Documentation

Welcome to `v-discord` (Ultra-Simple Edition)! This Python module is designed for extreme ease of use, offering a direct, script-like approach to common Discord bot tasks. It wraps `discord.py` to simplify its usage significantly.

## 1. Setup

1.  **Install `discord.py`**:
    ```bash
    pip install discord.py
    ```
2.  **Get `v_discord_ultra_simple.py`**:
    Save the `v_discord_ultra_simple.py` file (provided in the previous response) into your project directory, alongside your main bot script.

## 2. Core Concepts

*   **`VDBot` Object**: The main object you interact with. Initialize it with your bot token.
*   **Prefix**: Commands start with a prefix (default `!`).
*   **Command Handlers**: Functions you define with `@bot.command()` to react to specific commands.
*   **Message Recording**: The bot needs `bot.record_messages(True)` to be active to listen for and process commands or custom message events.
*   **Asynchronous**: Many operations (like sending messages) are `async`, so your command handlers and some other parts of your script will use `async` and `await`.

## 3. Basic Bot Script

This script demonstrates the fundamental structure.

```python
# my_simple_bot.py
import asyncio
import discord # Useful for type hints like discord.Message
from v_discord_ultra_simple import VDBot # Import the class

# --- 1. Initialize Bot with Token ---
# Replace "YOUR_DISCORD_BOT_TOKEN_HERE" with your actual token
BOT_TOKEN = "YOUR_DISCORD_BOT_TOKEN_HERE"
bot = VDBot(BOT_TOKEN)

# --- 2. Configure (Optional but Recommended) ---
bot.set_prefix("mybot ") # Example: "mybot ping"

MY_GUILD_ID = 123456789012345678  # Replace with your Server ID
GENERAL_CHANNEL_ID = 123456789012345679 # Replace with your Channel ID
bot.set_defaults(
    guild_id=MY_GUILD_ID,
    general_chat_id=GENERAL_CHANNEL_ID
)

# --- 3. Define Command Handlers ---
@bot.command(name="ping") # Command: "mybot ping"
async def handle_ping_command(message: discord.Message, args: list[str]):
    """Replies with Pong! when 'mybot ping' is typed."""
    await bot.reply_to_message("Pong!", message)
    print(f"Replied to ping from {message.author.name}")

@bot.command(name="greet") # Command: "mybot greet" or "mybot greet @User"
async def handle_greet_command(message: discord.Message, args: list[str]):
    """Greets the user or a mentioned user."""
    if args: # If there are arguments (e.g., a mention)
        target = " ".join(args)
        await bot.send_message(f"Hello {target}!", channel_id=message.channel.id)
    else:
        await bot.reply_to_message(f"Hello {message.author.mention}!", message)

# --- 4. Start "Recording" Messages ---
# This enables command processing and custom message listeners.
bot.record_messages(enable=True)

# --- 5. Run the Bot ---
# This connects to Discord and starts listening for events.
# It's a blocking call, so it should be the last part of your script.
async def main():
    print("Bot starting up...")
    bot.run() # This function will run until the bot is stopped (e.g., Ctrl+C)

if __name__ == "__main__":
    try:
        asyncio.run(main())
    except KeyboardInterrupt:
        print("\nBot shutting down by user request.")
    # Note: The bot.client.close() is handled internally by client.run() on KeyboardInterrupt
```

## 4. API Reference & Examples

### Initialization & Configuration

#### `VDBot(token: str)`
Initializes the bot.
*   `token`: Your Discord bot token (string). **Required.**

```python
from v_discord_ultra_simple import VDBot
bot = VDBot("YOUR_TOKEN_HERE")
```

#### `bot.set_prefix(prefix: str)`
Sets the command prefix. Default is `!`.
```python
bot.set_prefix("bot>") # Commands will be like "bot>ping"
```

#### `bot.set_defaults(...)`
Sets default IDs for Guild, general chat, announcements, and moderator role. Simplifies later calls.
*   `guild_id: Optional[int]`
*   `general_chat_id: Optional[int]`
*   `announcement_chat_id: Optional[int]`
*   `moderator_role_id: Optional[Union[int, str]]` (Role ID as int or Role Name as string)

```python
MY_SERVER_ID = 111
GENERAL_CHAT = 222
ANNOUNCEMENT_CHAT = 333
MOD_ROLE = "Moderators" # Or an ID like 444

bot.set_defaults(
    guild_id=MY_SERVER_ID,
    general_chat_id=GENERAL_CHAT,
    announcement_chat_id=ANNOUNCEMENT_CHAT,
    moderator_role_id=MOD_ROLE
)
```

### Message Handling & Commands

#### `bot.record_messages(enable: bool = True)`
Starts or stops the bot from processing messages for commands and custom listeners.
```python
bot.record_messages(True) # Start listening
# ... later ...
# bot.record_messages(False) # Stop listening
```

#### `@bot.command(name: Optional[str] = None)` (Decorator)
Defines a command handler. The decorated function **must** be `async` and accept `(message: discord.Message, args: list[str])`.
*   `name`: The command name (without prefix). If `None`, the function's name is used.
*   `message`: The `discord.Message` object that triggered the command.
*   `args`: A list of strings, representing words after the command name.

```python
@bot.command(name="info")
async def show_info(message: discord.Message, args: list[str]):
    user_info = f"You are {message.author.name} (ID: {message.author.id})"
    channel_info = f"This channel is '{message.channel.name}' (ID: {message.channel.id})"
    await bot.reply_to_message(f"{user_info}\n{channel_info}", message)

@bot.command() # Name will be "add"
async def add(message: discord.Message, args: list[str]):
    if len(args) < 2:
        await bot.reply_to_message("Please provide two numbers to add. Usage: !add <num1> <num2>", message)
        return
    try:
        num1 = float(args[0])
        num2 = float(args[1])
        result = num1 + num2
        await bot.reply_to_message(f"The sum of {num1} and {num2} is {result}.", message)
    except ValueError:
        await bot.reply_to_message("Invalid numbers provided.", message)
```

#### `bot.get_triggered_command_message() -> Optional[discord.Message]`
Returns the `discord.Message` object that triggered the most recently processed `@bot.command` handler. This is useful for a specific style of replying where you don't pass the message object directly.

```python
# (Inside a command handler, or if you know a command just ran)
triggered_msg = bot.get_triggered_command_message()
if triggered_msg:
    await bot.reply_to_message("Responding to the command you just sent!", triggered_msg)
```
**Note:** Your earlier example `bot.replyToMessage("Pong!", bot.exec_command(name="ping"))` is not how this works. `get_triggered_command_message()` provides the *source* message.

#### `@bot.on_message_received` (Decorator)
Defines a custom listener for *any* message the bot sees (if `record_messages` is enabled). Processed *after* checking for commands. The function **must** be `async` and accept `(message: discord.Message)`.

```python
@bot.on_message_received
async def keyword_listener(message: discord.Message):
    if message.author.bot: # Ignore other bots
        return
    if "python" in message.content.lower():
        await bot.reply_to_message("I see you mentioned Python! It's a great language.", message)
```

### Sending & Manipulating Messages

#### `async bot.send_message(...)`
Sends a message.
*   `message_text: str`: Content of the message.
*   `guild_id: Optional[int]`: Target Guild ID. Uses default if `None`.
*   `channel_id: Optional[int]`: Target Channel ID. Uses default `general_chat_id` if `None`.
*   `embed: Optional[discord.Embed]`: A `discord.Embed` object.
*   Returns: `Optional[discord.Message]` (the sent message) or `None`.

```python
# Send to default general chat in default guild
await bot.send_message("Hello everyone!")

# Send to a specific channel in default guild
SPECIFIC_CHANNEL_ID = 987654321
await bot.send_message("Important update!", channel_id=SPECIFIC_CHANNEL_ID)

# Send to specific channel in specific guild
OTHER_GUILD_ID = 1122334455
OTHER_CHANNEL_ID = 6677889900
await bot.send_message("Message for another server.", guild_id=OTHER_GUILD_ID, channel_id=OTHER_CHANNEL_ID)
```

#### `async bot.reply_to_message(...)`
Replies to a given `discord.Message` object.
*   `reply_text: str`: Content of the reply.
*   `message_to_reply_to: discord.Message`: The message object you are replying to.
*   `embed: Optional[discord.Embed]`
*   Returns: `Optional[discord.Message]` or `None`.

```python
# (Inside a command handler where 'message' is the triggering message)
await bot.reply_to_message("Got your command!", message)
```

#### `bot.create_embed(...) -> discord.Embed`
Helper to create `discord.Embed` objects.
*   `title: Optional[str]`
*   `description: Optional[str]`
*   `color_hex: Optional[int]` (e.g., `0x00FF00` for green)
*   `author_name: Optional[str]`, `author_icon_url: Optional[str]`
*   `footer_text: Optional[str]`, `footer_icon_url: Optional[str]`
*   `image_url: Optional[str]`, `thumbnail_url: Optional[str]`
*   Returns: A `discord.Embed` object. You can then use `my_embed.add_field(name="Field Name", value="Field Value", inline=False)`.

```python
my_embed = bot.create_embed(
    title="Server Rules",
    description="Please follow these rules:",
    color_hex=0xFF0000, # Red
    footer_text="Thanks for reading!",
    thumbnail_url="https://example.com/server_logo.png" # Replace with actual URL
)
my_embed.add_field(name="1. Be Respectful", value="Treat everyone with kindness.")
my_embed.add_field(name="2. No Spam", value="Keep channels clean.")

# Send the embed (e.g., in a command handler)
# await bot.send_message(message_text=None, embed=my_embed, channel_id=message.channel.id)
# Or reply with it:
# await bot.reply_to_message(reply_text=None, message_to_reply_to=message, embed=my_embed)
```

#### `async bot.edit_message(...)`
Edits a message previously sent by the bot.
*   `message_to_edit: discord.Message`
*   `new_content: Optional[str]`
*   `new_embed: Optional[discord.Embed]`
*   Returns: `Optional[discord.Message]` (the edited message) or `None`.

```python
# (Inside a command handler)
# First, send a message and store it
status_message = await bot.send_message("Processing your request...", channel_id=message.channel.id)

# ... do some work ...

# Then edit it
if status_message:
    await bot.edit_message(status_message, new_content="Request complete!", new_embed=None)
```

#### `async bot.delete_message(...)`
Deletes a message. The bot needs "Manage Messages" permission if it's not its own message.
*   `message_to_delete: discord.Message`
*   `delay_seconds: Optional[float]` (wait this long before deleting)
*   Returns: `bool` (True if successful).

```python
# (Inside a command handler, 'message' is the triggering message)
# Delete the command message itself (if bot has perms)
await bot.delete_message(message, delay_seconds=5)
print(f"Scheduled deletion of message from {message.author.name}")

# Delete a message the bot sent
bot_msg_to_delete = await bot.send_message("This message will self-destruct.", channel_id=message.channel.id)
if bot_msg_to_delete:
    await bot.delete_message(bot_msg_to_delete, delay_seconds=10)
```

### Moderation (Bot needs appropriate permissions)

#### `async bot.kick_member(...)`
Kicks a member from the guild.
*   `user_id_or_mention: Union[int, str]` (User ID, mention string like `<@123...>`, or `"Username#1234"`)
*   `reason: Optional[str]`
*   `guild_id: Optional[int]` (Uses default guild if `None`)
*   Returns: `bool` (True if successful).

```python
# (Inside a command handler, e.g., !kick <user> [reason])
# args[0] would be the user_id_or_mention, args[1:] the reason
MEMBER_TO_KICK = 123456789012345678 # Replace with actual ID or get from args
kick_reason = "Violated server rules."
success = await bot.kick_member(MEMBER_TO_KICK, reason=kick_reason)
if success:
    await bot.send_message(f"Successfully kicked user {MEMBER_TO_KICK}.", channel_id=message.channel.id)
else:
    await bot.send_message(f"Failed to kick user {MEMBER_TO_KICK}.", channel_id=message.channel.id)
```

#### `async bot.ban_member(...)`
Bans a member from the guild.
*   `user_id_or_mention: Union[int, str]`
*   `reason: Optional[str]`
*   `delete_message_days: int` (0-7 days of messages from the user to delete)
*   `guild_id: Optional[int]` (Uses default guild if `None`)
*   Returns: `bool` (True if successful).

```python
# (Inside a command handler, e.g., !ban <user> [reason])
MEMBER_TO_BAN = "OffendingUser#1234" # Replace with actual tag or ID
ban_reason = "Repeated violations."
success = await bot.ban_member(MEMBER_TO_BAN, reason=ban_reason, delete_message_days=1)
if success:
    await bot.send_message(f"Successfully banned {MEMBER_TO_BAN}.", channel_id=message.channel.id)
else:
    await bot.send_message(f"Failed to ban {MEMBER_TO_BAN}.", channel_id=message.channel.id)
```

### Utility Getters

These help you retrieve common Discord objects if you have their IDs.

*   `bot.get_guild(guild_id: Optional[int] = None) -> Optional[discord.Guild]`
    *   Returns the `discord.Guild` object. Uses default guild if `guild_id` is `None`.
*   `bot.get_channel(channel_id: int, guild_id: Optional[int] = None) -> Optional[discord.abc.GuildChannel]`
    *   Returns a channel object from a specific guild (or default guild if `guild_id` is `None`).
*   `bot.get_user(user_id: int) -> Optional[discord.User]`
    *   Returns a `discord.User` object.
*   `async bot.fetch_member_from_guild(user_id: int, guild_id: Optional[int] = None) -> Optional[discord.Member]`
    *   Fetches a `discord.Member` object from a guild (tries cache first, then API).

```python
# (Inside a command handler or other async function)
target_guild = bot.get_guild() # Gets default guild
if target_guild:
    print(f"Operating in guild: {target_guild.name}")

some_channel = bot.get_channel(channel_id=123456789) # Channel from default guild
if some_channel:
    await bot.send_message("Hello from utility getter!", channel_id=some_channel.id)

a_user = bot.get_user(user_id=987654321)
if a_user:
    print(f"Found user: {a_user.name}")

member_object = await bot.fetch_member_from_guild(user_id=1122334455)
if member_object:
    print(f"Fetched member: {member_object.display_name}, Roles: {member_object.roles}")
```

### Running the Bot

#### `bot.run()`
**Crucial:** Starts the bot, connects to Discord, and begins the event loop. This is a **blocking call** and should usually be the last action in your script's main execution path (often within an `asyncio.run(main_function())` block).

```python
# At the end of your bot script:
async def main_async_loop():
    # Any other async setup can go here
    print("Bot starting...")
    bot.run() # This takes over until Ctrl+C or an unhandled critical error

if __name__ == "__main__":
    try:
        asyncio.run(main_async_loop())
    except KeyboardInterrupt:
        print("\nBot manually shut down.")
    # The underlying discord.py client handles closing the connection on KeyboardInterrupt.
```

### Advanced: Accessing Raw `discord.py` Client

#### `bot.raw_discord_client -> discord.Client` (Property)
For features not directly exposed by `v-discord` (Ultra-Simple), you can access the underlying `discord.Client` object from `discord.py`.

```python
# (Inside an async function)
raw_client = bot.raw_discord_client
print(f"Bot Latency via raw client: {raw_client.latency * 1000:.2f} ms")
# You can now use raw_client as you would in standard discord.py for advanced tasks.
# e.g., raw_client.wait_for('reaction_add', ...), etc.
```

This documentation should provide a good starting point for creating simple Discord bots with `v-discord` (Ultra-Simple Edition). Remember to replace placeholder IDs and tokens with your actual values. Happy botting!
```
