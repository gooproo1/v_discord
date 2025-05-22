# `v-discord` (Super Easy Edition) - Documentation

`v-discord` (Super Easy Edition) is a Python module designed to simplify Discord bot creation by providing a more synchronous-looking, "script-style" interface while managing the underlying asynchronous `discord.py` client in background threads. It aims to make common bot tasks feel as straightforward as using libraries like PyAutoGUI.

**Goal:** Make bot development accessible by hiding much of the `asyncio` complexity for common use cases.

## Table of Contents

1.  [Features](#features)
2.  [Setup](#setup)
3.  [Quickstart Example](#quickstart-example)
4.  [API Reference](#api-reference)
    *   [Bot Lifecycle](#bot-lifecycle)
    *   [Command Handling](#command-handling)
    *   [Messaging](#messaging)
    *   [Embeds](#embeds)
    *   [Message Manipulation](#message-manipulation)
    *   [Moderation Tools](#moderation-tools)
    *   [Information Getters](#information-getters)
    *   [Utility](#utility)
5.  [Full Example Script](#full-example-script)
6.  [Important Considerations](#important-considerations)

## 1. Features

*   **Simplified Bot Startup:** Start your bot with a single function call that blocks until ready.
*   **Synchronous-Style Functions:** Send messages, reply, kick, ban, etc., using functions that feel synchronous, even though Discord is async.
*   **Easy Command Handling:** Define command handlers as regular Python functions using a simple decorator.
*   **Embed Helper:** Create rich embeds with a straightforward dictionary-based helper.
*   **Background Operation:** The core `discord.py` client runs in background threads, managed by the module.
*   **Common Tools Included:** Basic moderation, user/server info retrieval, and message management.

## 2. Setup

1.  **Install `discord.py`**:
    If you haven't already, install the `discord.py` library:
    ```bash
    pip install -U discord.py
    ```

2.  **Get `v_discord_super_easy.py`**:
    Download or copy the `v_discord_super_easy.py` module file (provided in previous responses or from its source) and place it in your project directory, alongside your main bot script.

## 3. Quickstart Example

```python
# my_bot.py
from v_discord_super_easy import * # Import all for simplicity
import time

# --- Configuration ---
BOT_TOKEN = "YOUR_DISCORD_BOT_TOKEN_HERE"  # !!! REPLACE THIS !!!
SERVER_ID = 123456789012345678      # Optional: Your Server ID
CHANNEL_ID = 123456789012345679     # Optional: Your default Channel ID

# --- Define Command Handlers ---
@on_command("ping") # Bot will respond to "!ping" (if default prefix)
def handle_ping(message_obj): # Receives the discord.Message object
    reply_to_message(message_obj, "Pong!")
    print(f"Responded to ping from {get_message_author_name(message_obj)}")

@on_command("echo") # Example: !echo Hello there
def handle_echo(message_obj):
    content_to_echo = get_message_content(message_obj).split(" ", 1)
    if len(content_to_echo) > 1:
        send_message(content_to_echo[1], channel_id=message_obj.channel.id)
    else:
        reply_to_message(message_obj, "Usage: !echo <your message>")

# --- Main Bot Logic ---
if __name__ == "__main__":
    print("Starting bot...")

    # Start the bot. This call blocks until the bot is connected and ready.
    start_bot(
        token=BOT_TOKEN,
        prefix="!",  # Default command prefix
        default_server_id=SERVER_ID, # Optional
        default_channel_id=CHANNEL_ID  # Optional
    )

    if is_bot_ready():
        print("Bot is online and ready!")
        # Send a startup message to the default channel (if configured)
        send_message("Hello! Super Easy Bot is now online. Prefix is '!'")
        
        # Keep the main script alive so the bot (in background threads) can run
        keep_alive() # This function includes a loop with Ctrl+C handling
    else:
        print("Bot failed to start.")
    
    print("Bot script finished.")
```

## 4. API Reference

All functions (unless otherwise specified) are designed to be called from your main synchronous script.

### Bot Lifecycle

#### `start_bot(token: str, prefix: str = "!", default_server_id: Optional[int] = None, default_channel_id: Optional[int] = None)`
Initializes and connects the bot to Discord. This function **blocks** until the bot is ready or fails (timeout: 30s).
*   `token`: Your Discord bot token (string). **Required.**
*   `prefix`: The command prefix (string, default: `!`).
*   `default_server_id`: (Optional int) ID of the server for default operations.
*   `default_channel_id`: (Optional int) ID of the channel for default message sending.

#### `stop_bot()`
Stops the bot, disconnects from Discord, and cleans up background threads. Call this when your script is exiting.

#### `is_bot_ready() -> bool`
Returns `True` if the bot is connected to Discord and ready to operate, `False` otherwise.

### Command Handling

#### `@on_command(command_name: str)` (Decorator)
Registers a function to handle a specific command.
*   `command_name`: The name of the command (e.g., `"ping"` for `!ping`).
*   The decorated function **must** accept one argument: `message_obj` (which will be a `discord.Message` object representing the command invocation).
*   The decorated function itself should be a standard synchronous Python function.

Example:
```python
@on_command("greet")
def my_greeting_handler(message_obj):
    user_name = get_message_author_name(message_obj)
    reply_to_message(message_obj, f"Hello, {user_name}!")
```

### Messaging

#### `send_message(message_content: Optional[str] = None, channel_id: Optional[int] = None, embed_dict: Optional[Dict] = None) -> Optional[discord.Message]`
Sends a message. Uses default channel if `channel_id` is not specified and a default is configured.
*   `message_content`: (Optional string) The text content of the message.
*   `channel_id`: (Optional int) The ID of the channel to send the message to.
*   `embed_dict`: (Optional dict) An embed dictionary created by `create_embed()`.
*   **Note:** Either `message_content` or `embed_dict` (or both) must be provided.
*   Returns the sent `discord.Message` object on success, `None` on failure.

#### `reply_to_message(original_message_obj: discord.Message, reply_text: Optional[str] = None, embed_dict: Optional[Dict] = None) -> Optional[discord.Message]`
Replies to a given `discord.Message` object.
*   `original_message_obj`: The `discord.Message` object to reply to.
*   `reply_text`: (Optional string) The text content of the reply.
*   `embed_dict`: (Optional dict) An embed dictionary.
*   **Note:** Either `reply_text` or `embed_dict` (or both) must be provided.
*   Returns the sent `discord.Message` object on success, `None` on failure.

### Embeds

#### `create_embed(...) -> Dict`
A helper function to create embed dictionaries compatible with `send_message` and `reply_to_message`.
*   `title: Optional[str]`
*   `description: Optional[str]`
*   `color_hex: Optional[int]` (e.g., `0xFF0000` for red)
*   `author_name: Optional[str]`, `author_icon_url: Optional[str]`
*   `footer_text: Optional[str]`, `footer_icon_url: Optional[str]`
*   `image_url: Optional[str]`
*   `thumbnail_url: Optional[str]`
*   `fields: Optional[List[Dict[str, Union[str, bool]]]]`
    *   `fields` should be a list of dictionaries. Each field dictionary should have:
        *   `"name": str`
        *   `"value": str`
        *   `"inline": bool` (optional, defaults to `False`)
*   Returns: A dictionary representing the embed.

Example:
```python
my_embed = create_embed(
    title="Important Info",
    description="This is a test embed.",
    color_hex=0x3498DB, # Blue
    fields=[
        {"name": "Section 1", "value": "Content for section 1.", "inline": True},
        {"name": "Section 2", "value": "Content for section 2.", "inline": True}
    ],
    footer_text="Bot Footer"
)
send_message(embed_dict=my_embed, channel_id=SOME_CHANNEL_ID)
```

### Message Manipulation

#### `edit_message(message_obj_to_edit: discord.Message, new_content: Optional[str] = None, new_embed_dict: Optional[Dict] = None) -> bool`
Edits a message previously sent **by the bot**.
*   Returns `True` on success, `False` on failure.

#### `delete_message(message_obj_to_delete: discord.Message, delay_seconds: Optional[float] = None) -> bool`
Deletes any message (bot needs "Manage Messages" permission if not its own message).
*   `delay_seconds`: (Optional float) Wait this long before deleting.
*   Returns `True` on success, `False` on failure.

### Moderation Tools

(Bot requires appropriate server permissions for these actions.)

#### `kick_member(user_id_or_name: Union[int, str], reason: Optional[str] = None, server_id: Optional[int] = None) -> bool`
Kicks a member from the server.
*   `user_id_or_name`: User ID (int), mention string (e.g., `<@123...>`), or `"Username#1234"`.
*   `server_id`: (Optional int) Uses default server if `None`.
*   Returns `True` on success, `False` on failure.

#### `ban_member(user_id_or_name: Union[int, str], reason: Optional[str] = None, delete_message_days: int = 0, server_id: Optional[int] = None) -> bool`
Bans a user/member from the server.
*   `delete_message_days`: (int, 0-7) Number of days of messages from the user to delete.
*   Returns `True` on success, `False` on failure.

### Information Getters

These functions retrieve information about Discord entities.

#### `get_message_author_name(message_obj: discord.Message) -> Optional[str]`
Returns the display name (nickname or username) of the message author.

#### `get_message_content(message_obj: discord.Message) -> Optional[str]`
Returns the text content of the message.

#### `get_user_info(user_id_or_name: Union[int, str], server_id: Optional[int] = None) -> Optional[Dict]`
Gets information about a user or member.
*   Returns a dictionary containing keys like `id`, `name`, `discriminator`, `display_name`, `is_bot`. If a member in the specified server, also includes `roles` (list of role names) and `joined_at` (ISO string). Returns `None` if user/member not found.

#### `get_server_info(server_id: Optional[int] = None) -> Optional[Dict]`
Gets information about a server (guild).
*   Uses default server if `server_id` is `None`.
*   Returns a dictionary with keys like `id`, `name`, `owner_id`, `member_count`, `icon_url`, `text_channel_count`, `voice_channel_count`. Returns `None` if server not found.

### Utility

#### `keep_alive()`
A helper function that enters an infinite loop (`time.sleep(1)`), effectively keeping your main script alive so the background bot threads can continue to operate. It handles `KeyboardInterrupt` (Ctrl+C) to call `stop_bot()` gracefully.
Call this as the last step in your main script if it doesn't have its own indefinite loop.

## 5. Full Example Script

(Refer to the `my_advanced_easy_bot.py` example from the previous Gist/response for a more comprehensive script demonstrating many of these features in action, including embeds and moderation commands.)

Here's a slightly more structured version of the Quickstart:

```python
# main_bot_script.py
from v_discord_super_easy import *
import time

# --- CONFIGURATION ---
BOT_TOKEN = "YOUR_BOT_TOKEN"
# Optional:
DEFAULT_GUILD_ID = 123456789012345678
DEFAULT_CHANNEL_ID = 123456789012345679
MOD_LOG_CHANNEL_ID = 123456789012345600

# --- COMMAND HANDLERS ---
@on_command("hello")
def cmd_hello(msg): # msg is a discord.Message object
    author = get_message_author_name(msg)
    reply_to_message(msg, f"Greetings, {author}!")

@on_command("stats")
def cmd_stats(msg):
    server_info = get_server_info(msg.guild.id if msg.guild else None)
    if server_info:
        embed = create_embed(
            title=f"Stats for {server_info['name']}",
            fields=[
                {"name": "Members", "value": str(server_info['member_count']), "inline": True},
                {"name": "Owner ID", "value": str(server_info['owner_id']), "inline": True}
            ]
        )
        reply_to_message(msg, embed_dict=embed)
    else:
        reply_to_message(msg, "Couldn't get server stats (are we in a DM?).")

# --- MAIN EXECUTION ---
def run_my_bot():
    print("Initializing bot...")
    start_bot(
        token=BOT_TOKEN,
        prefix=">", # Custom prefix
        default_server_id=DEFAULT_GUILD_ID,
        default_channel_id=DEFAULT_CHANNEL_ID
    )

    if is_bot_ready():
        print(f"Bot is online! Prefix: '>'")
        send_message(
            message_content="Bot has started successfully!",
            # Uses default_channel_id if set, otherwise needs to be specified
            # channel_id=DEFAULT_CHANNEL_ID # Explicitly if needed
        )
        keep_alive() # Keeps the bot running
    else:
        print("Failed to start the bot. Please check your token and network.")

if __name__ == "__main__":
    run_my_bot()
```

## 6. Important Considerations

*   **Error Handling:** While the module tries to print errors, complex error handling from the background threads back to your main synchronous code can be limited. Check the console for error messages.
*   **Performance:** For very high-traffic bots, the overhead of bridging synchronous and asynchronous code via threads might become a factor. For most common use cases, it should be acceptable.
*   **Rate Limits:** Discord imposes API rate limits. Sending too many messages or performing too many actions too quickly will result in temporary blocks. This module doesn't inherently manage complex rate limiting beyond what `discord.py` handles.
*   **Background Threads:** The bot's core logic runs in separate threads. Be mindful of this if you are doing complex state management that needs to be thread-safe (though for simple command handlers, this is usually not an issue).
*   **Simplicity vs. Power:** This module prioritizes ease of use. For highly advanced or customized bot behavior, directly using `discord.py` with `async/await` will offer more control and flexibility.

This module provides a gentle entry point into Discord bot development. Happy botting!
```
v_discord v1.1.105
```
