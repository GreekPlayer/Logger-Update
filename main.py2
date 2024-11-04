import discord
from discord.ext import commands
import re
from datetime import datetime, timedelta

# Replace 'YOUR_BOT_TOKEN' with your bot's token
TOKEN = 'Your Bot Token'
# Replace 'YOUR_CHANNEL_ID' with the channel ID where you want to log messages
LOG_CHANNEL_ID = 123456789012345678  # Example: replace with actual channel ID

intents = discord.Intents.default()
intents.members = True
intents.message_content = True

# Add channel IDs where links are allowed
WHITELISTED_CHANNELS = {1302821011766378527, 1302817542095765585, 1302823547474804776, 1302818024541388850, 1302816950354710579}  # Example channel IDs where links are allowed
# Add role IDs that are allowed to post links
WHITELISTED_ROLES = {1302819626190766164, 1302819745183043586, 1302819323055706224}  # Example role IDs allowed to post links

# Regular expression to detect links
link_regex = re.compile(r"(https?://[^\s]+)")

intents = discord.Intents.default()
intents.members = True               # Required for member join/leave events
intents.message_content = True        # Required for logging messages
intents.guilds = True
intents.bans = True                   # Required for ban/unban events
intents.guild_messages = True         # Required for message events
intents.voice_states = True           # Required for voice activity

bot = commands.Bot(command_prefix="!", intents=intents)

@bot.event
async def on_ready():
    print(f'Logged in as {bot.user.name}')
    print(f'Bot ID: {bot.user.id}')
    print('------')

@bot.event
async def on_member_join(member):
    # Log join message in a specific channel
    channel = bot.get_channel(1302834184817213521)
    if channel:
        await channel.send(f'🚪 **Join:** {member.mention} has joined the server.')

@bot.event
async def on_member_remove(member):
    # Log leave message in a specific channel
    channel = bot.get_channel(1302834234628636683)
    if channel:
        await channel.send(f'🚶 **Leave:** {member.mention} has left the server.')

@bot.event
async def on_message(message):
    if message.author == bot.user:
        return
    # Log message in a specific channel
    channel = bot.get_channel(1302840245359874058)
    if channel:
        await channel.send(f'💬 **Message:** {message.author} in {message.channel}: {message.content}')

@bot.event
async def on_message_delete(message):
    # Log message deletion
    channel = bot.get_channel(1302840245359874058)
    if channel:
        await channel.send(f'🗑️ **Deleted Message:** {message.author} in {message.channel}: {message.content}')

@bot.event
async def on_message_edit(before, after):
    # Log message edits
    channel = bot.get_channel(1302840245359874058)
    if channel:
        await channel.send(
            f'✏️ **Edited Message:** {before.author} in {before.channel}\n'
            f'**Before:** {before.content}\n'
            f'**After:** {after.content}'
        )

@bot.event
async def on_voice_state_update(member, before, after):
    # Log voice channel join/leave/move
    channel = bot.get_channel(1302840646935117844)
    if not channel:
        return
    
    if before.channel is None and after.channel is not None:
        await channel.send(f'🔊 **Voice Join:** {member.mention} joined {after.channel.name}')
    elif before.channel is not None and after.channel is None:
        await channel.send(f'🔇 **Voice Leave:** {member.mention} left {before.channel.name}')
    elif before.channel != after.channel:
        await channel.send(f'🔄 **Voice Move:** {member.mention} moved from {before.channel.name} to {after.channel.name}')

@bot.event
async def on_member_ban(guild, user):
    # Log ban events
    channel = bot.get_channel(1302840881304567830)
    if channel:
        await channel.send(f'⛔ **Ban:** {user.mention} has been banned from the server.')

@bot.event
async def on_member_unban(guild, user):
    # Log unban events
    channel = bot.get_channel(1302841066516648008)
    if channel:
        await channel.send(f'✅ **Unban:** {user.mention} has been unbanned.')

@bot.event
async def on_member_remove(member):
    # Log kick event - usually inferred from a leave if permissions allow
    # (Discord doesn't have a direct on_member_kick event)
    channel = bot.get_channel(1302834234628636683)
    if channel:
        await channel.send(f'👢 **Kick (or Leave):** {member.mention} has left the server.')

@bot.event
async def on_member_join(member):
    log_channel = bot.get_channel(1302844248990679060)
    
    # Step 1: Check account age
    account_age = datetime.utcnow() - member.created_at
    if account_age < timedelta(days=ALT_ACCOUNT_DAYS_LIMIT):
        # Notify the user and take action if account age is suspicious
        try:
            await member.send("Your account is too new to join this server. Please try again later.")
        except discord.Forbidden:
            # Couldn't send a DM to the user; possibly DMs are disabled
            pass
        await member.kick(reason="Detected as an alt account due to recent account creation.")
        
        if log_channel:
            await log_channel.send(f'⚠️ **Anti-Alt:** {member.mention} was kicked for having an account age of {account_age.days} days, below the limit of {ALT_ACCOUNT_DAYS_LIMIT} days.')
        return

    # Step 2: Check for suspicious usernames
    if suspicious_username_regex.match(member.name):
        try:
            await member.send("Your username appears suspicious. Please verify with the admins.")
        except discord.Forbidden:
            pass
        if log_channel:
            await log_channel.send(f'⚠️ **Suspicious Username:** {member.mention} joined with a potentially suspicious username "{member.name}". Manual verification may be required.')


@bot.event
async def on_message(message):
    # Ignore messages from the bot itself
    if message.author == bot.user:
        return

    # Check if the message is in a whitelisted channel
    if message.channel.id in WHITELISTED_CHANNELS:
        await bot.process_commands(message)
        return

    # Check if the author has a whitelisted role
    if any(role.id in WHITELISTED_ROLES for role in message.author.roles):
        await bot.process_commands(message)
        return

    # Anti-Link Detection: If message contains a link
    if link_regex.search(message.content):
        # Delete the message
        await message.delete()

        # Notify the user
        await message.channel.send(f'{message.author.mention}, posting links is not allowed in this server.')

        # Log the attempt in the logging channel
        log_channel = bot.get_channel(1302846198637658237)
        if log_channel:
            await log_channel.send(
                f'🚫 **Anti-Link:** {message.author.mention} tried to post a link in {message.channel.mention}.\n'
                f'**Content:** "{message.content}"'
            )

    # Process other commands if present
    await bot.process_commands(message)

bot.run('Your Bot Token')
