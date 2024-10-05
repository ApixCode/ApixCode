import json
import asyncio
import aiohttp
import nextcord
from nextcord.ext import commands
import re
import time

TOKEN = 'MTI5MTMxMzY4MzY3ODM2Nzc0NA.GAeIw7.Lq6ODcHDedyypPwOzcZc57DYPQZLqPAK8cG3PQ'
AUTOBYPASS_DATABASE_FILE = 'autobypass-database.json' ## make new file and give name autobypass-database.json
ZANERU_API_URL = 'https://zaneru-official.vercel.app/api/bypass/delta?link={}&api_key=zaneru-official'  
ZANE_API_URL = 'https://zaneru-official.vercel.app/api/bypass/fluxus?link={}&api_key=zaneru-official'
ZANE2_API_URL = 'https://zaneru-official.vercel.app/api/bypass/relzhub?link={}&api_key=zaneru-official'

AUTHORIZED_USER_IDS = [1043344853595062342]

intents = nextcord.Intents.default()
intents.message_content = True
intents.guilds = True
bot = commands.Bot(command_prefix="-", intents=intents)

try:
    with open(AUTOBYPASS_DATABASE_FILE, 'r') as f:
        autobypass_channels = json.load(f)
except FileNotFoundError:
    autobypass_channels = []

def set_autobypass(channel_id):
    if channel_id not in autobypass_channels:
        autobypass_channels.append(channel_id)
        with open(AUTOBYPASS_DATABASE_FILE, 'w') as f:
            json.dump(autobypass_channels, f, indent=2)

def remove_autobypass(channel_id):
    if channel_id in autobypass_channels:
        autobypass_channels.remove(channel_id)
        with open(AUTOBYPASS_DATABASE_FILE, 'w') as f:
            json.dump(autobypass_channels, f, indent=2)

def is_auto_bypass(channel_id):
    return channel_id in autobypass_channels

async def call_zaneru_api(link):
    async with aiohttp.ClientSession() as session:
        try:
            async with session.get(ZANERU_API_URL.format(link)) as response:
                if response.status == 200:
                    return await response.json()
                else:
                    return {"error": f"{response.status}"}
        except Exception as error:
            return {"error": f"{str(error)}"}

async def call_zane_api(link):
    async with aiohttp.ClientSession() as session:
        try:
            async with session.get(ZANE_API_URL.format(link)) as response:
                if response.status == 200:
                    return await response.json()
                else:
                    return {"error": f"Error {response.status}", "message": response.reason}
        except Exception as error:
            return {"error": f"Exception occurred: {str(error)}"}

async def call_zane2_api(link):
    async with aiohttp.ClientSession() as session:
        try:
            async with session.get(ZANE2_API_URL.format(link)) as response:
                if response.status == 200:
                    return await response.json()
                else:
                    return {"error": f"Error {response.status}", "message": response.reason}
        except Exception as error:
            return {"error": f"Exception occurred: {str(error)}"}

def create_support_button():
    invite_link = "https://discord.gg/gJskAM1" # Replace with your discord invite link
    bot_link = "https://example.com/botinvite"  # Replace with your bot invite link or website bypass link or etc

    button_view = nextcord.ui.View()
    button_view.add_item(nextcord.ui.Button(label="ðŸ”— Support Server", style=nextcord.ButtonStyle.link, url=invite_link))
    button_view.add_item(nextcord.ui.Button(label="ðŸ¤– Invite Me", style=nextcord.ButtonStyle.link, url=bot_link))
    return button_view

def is_valid_link(link):
    valid_links = [
        "https://getkey.relzscript.xyz/redirect.php?hwid=",
        "https://gateway.platoboost.com/a/8?id=",
        "https://flux.li/android/external/start.php?HWID="
    ]
    return any(link.startswith(valid) for valid in valid_links)

@bot.slash_command(name='set-auto-bypass', description='Set an auto-bypass for this channel')
async def set_auto_bypass(interaction: nextcord.Interaction, channel: nextcord.TextChannel):
    if interaction.guild.owner_id != interaction.user.id and interaction.user.id not in AUTHORIZED_USER_IDS:
        await interaction.response.send_message('You don\'t have permission to use this command.', ephemeral=True)
        return
    set_autobypass(channel.id)
    await interaction.response.send_message(f'Auto-Bypass set for this channel: "<#{channel.id}>"', ephemeral=True)

@bot.slash_command(name='remove-auto-bypass', description='Remove an auto-bypass for this channel')
async def remove_auto_bypass(interaction: nextcord.Interaction, channel: nextcord.TextChannel):
    if interaction.guild.owner_id != interaction.user.id and interaction.user.id not in AUTHORIZED_USER_IDS:
        await interaction.response.send_message('You don\'t have permission to use this command.', ephemeral=True)
        return
    remove_autobypass(channel.id)
    await interaction.response.send_message(f'Auto-Bypass removed for this channel: "<#{channel.id}>"', ephemeral=True)

@bot.event
async def on_message(message):
    if message.author.bot:
        return

    if is_auto_bypass(message.channel.id) and is_valid_link(message.content):
        await handle_auto_bypass(message, message.author)

    await bot.process_commands(message)

async def handle_auto_bypass(message, author):
    username = author.mention  
    original_author = author

    try:
        loading_message = None
        start_time = time.time()
        avatar_url = message.author.display_avatar.url  
        link = message.content.strip()  

        if link.startswith("https://gateway.platoboost.com/a/8?id="):
            embed = nextcord.Embed(color=0x04ABEE)
            embed.add_field(name="Bypassing Delta:", value="```Waiting for result...```", inline=False)
            embed.add_field(name="Execution Time:", value="```Proccessing on result```", inline=False)
            embed.set_thumbnail(url=avatar_url)  
            embed.set_footer(text=f"Requested by {original_author.name}", icon_url=avatar_url)
            loading_message = await message.reply(content=f"**Bypassing Delta For** {username}", embed=embed)

            response = await call_zaneru_api(link)
            time_taken = time.time() - start_time  

            if "error" in response:
                error_embed = nextcord.Embed(color=nextcord.Color.red())
                error_embed.timestamp = nextcord.utils.utcnow()
                error_embed.add_field(name="Failed Message:", value=f"```{response['error']}```", inline=False)
                error_embed.add_field(name="Execution Time:", value=f"```{time_taken:.2f} seconds```", inline=False)  
                error_embed.set_thumbnail(url=avatar_url)  
                error_embed.set_footer(text=f"Requested by {original_author.name}", icon_url=avatar_url)
                await loading_message.edit(content=f"**Bypass Delta Failed** {username}.", embed=error_embed, view=create_support_button())
            else:
                bypass_key = response.get("key")
                time_left = response.get("time_left", "N/A") 

                result_embed = nextcord.Embed(color=nextcord.Color.green())
                result_embed.timestamp = nextcord.utils.utcnow()
                result_embed.add_field(name="Delta Key:", value=f"```{bypass_key}```", inline=False)  
                result_embed.add_field(name="Time Left:", value=f"```{time_left}```", inline=False) 
                result_embed.add_field(name="Execution Time:", value=f"```{time_taken:.2f} seconds```", inline=False)  
                result_embed.set_thumbnail(url=avatar_url)  
                result_embed.set_footer(text=f"Requested by {original_author.name}", icon_url=avatar_url)
                await loading_message.edit(content=f"**Bypass Delta Completed For** {username}.", embed=result_embed, view=create_support_button())

        elif link.startswith("https://flux.li/android/external/start.php?HWID="):
            embed = nextcord.Embed(color=0x04ABEE)
            embed.add_field(name="Bypassing Fluxus:", value="```Waiting for result...```", inline=False)
            embed.add_field(name="Execution Time:", value="```Proccessing on result```", inline=False)
            embed.set_thumbnail(url=avatar_url)  
            embed.set_footer(text=f"Requested by {original_author.name}", icon_url=avatar_url)
            loading_message = await message.reply(content=f"**Bypassing Fluxus For** {username}", embed=embed)

            response = await call_zane_api(link)
            time_taken = time.time() - start_time  

            if "error" in response:
                error_embed = nextcord.Embed(color=nextcord.Color.red())
                error_embed.timestamp = nextcord.utils.utcnow()
                error_embed.add_field(name="Failed Message:", value=f"```{response['error']}```", inline=False)
                error_embed.add_field(name="Execution Time:", value=f"```{time_taken:.2f} seconds```", inline=False)  
                error_embed.set_thumbnail(url=avatar_url)  
                error_embed.set_footer(text=f"Requested by {original_author.name}", icon_url=avatar_url)
                await loading_message.edit(content=f"**Bypass Fluxus Failed** {username}.", embed=error_embed, view=create_support_button())
            else:
                bypass_fluxus = response.get("key")

                result_embed = nextcord.Embed(color=nextcord.Color.green())
                result_embed.timestamp = nextcord.utils.utcnow()
                result_embed.add_field(name="Fluxus Key:", value=f"```{bypass_fluxus}```", inline=False)  
                result_embed.add_field(name="Execution Time:", value=f"```{time_taken:.2f} seconds```", inline=False)  
                result_embed.set_thumbnail(url=avatar_url)  
                result_embed.set_footer(text=f"Requested by {original_author.name}", icon_url=avatar_url)
                await loading_message.edit(content=f"**Bypass Fluxus Completed For** {username}.", embed=result_embed, view=create_support_button())

        elif link.startswith("https://getkey.relzscript.xyz/redirect.php?hwid="):
            embed = nextcord.Embed(color=0x04ABEE)
            embed.add_field(name="Bypassing Relz Hub:", value="```Waiting for result...```", inline=False)
            embed.add_field(name="Execution Time:", value="```Proccessing on result```", inline=False)
            embed.set_thumbnail(url=avatar_url)  
            embed.set_footer(text=f"Requested by {original_author.name}", icon_url=avatar_url)
            loading_message = await message.reply(content=f"**Bypassing Relz Hub For** {username}", embed=embed)

            response = await call_zane2_api(link)
            time_taken = time.time() - start_time  

            if "error" in response:
                error_embed = nextcord.Embed(color=nextcord.Color.red())
                error_embed.timestamp = nextcord.utils.utcnow()
                error_embed.add_field(name="Failed Message:", value=f"```{response['error']}```", inline=False)
                error_embed.add_field(name="Execution Time:", value=f"```{time_taken:.2f} seconds```", inline=False)  
                error_embed.set_thumbnail(url=avatar_url)  
                error_embed.set_footer(text=f"Requested by {original_author.name}", icon_url=avatar_url)
                await loading_message.edit(content=f"**Bypass Relz Hub Failed** {username}.", embed=error_embed, view=create_support_button())
            else:
                bypass_relz = response.get("key")

                result_embed = nextcord.Embed(color=nextcord.Color.green())
                result_embed.timestamp = nextcord.utils.utcnow()
                result_embed.add_field(name="Relz Key:", value=f"```{bypass_relz}```", inline=False)   
                result_embed.add_field(name="Execution Time:", value=f"```{time_taken:.2f} seconds```", inline=False)  
                result_embed.set_thumbnail(url=avatar_url)  
                result_embed.set_footer(text=f"Requested by {original_author.name}", icon_url=avatar_url)
                await loading_message.edit(content=f"**Bypass Relz Hub Completed For** {username}.", embed=result_embed, view=create_support_button())

        else:
            await message.reply(content="**Invalid link provided. Only support delta, fluxus, and relz hub bypass**", view=create_support_button())

    except Exception as e:
        embed = nextcord.Embed(color=nextcord.Color.red())
        embed.timestamp = nextcord.utils.utcnow()
        embed.add_field(name="Error Message:", value=f"```{str(e)}```", inline=False)
        embed.set_thumbnail(url=avatar_url)  
        embed.set_footer(text=f"Requested by {original_author.name}", icon_url=avatar_url)

        if loading_message: 
            await loading_message.edit(embed=embed, view=create_support_button())
        else:
            await message.channel.send(embed=embed, view=create_support_button())

        print(f"An error occurred: {str(e)}")

@bot.slash_command(name='bypass', description='Bypass key/addlink')
async def bypass(interaction: nextcord.Interaction, link: str):
    original_author = interaction.user
    avatar_url = original_author.display_avatar.url 

    try:
        loading_message = None
        start_time = time.time() 

        if is_valid_link(link):
            if link.startswith("https://gateway.platoboost.com/a/8?id="):
                embed = nextcord.Embed(color=0x04ABEE)
                embed.add_field(name="Bypassing Delta:", value="```Waiting for result...```", inline=False)
                embed.add_field(name="Execution Time:", value="```Proccessing on result```", inline=False)
                embed.set_thumbnail(url=avatar_url)  
                embed.set_footer(text=f"Requested by {original_author.name}", icon_url=avatar_url)
                loading_message = await interaction.response.send_message(embed=embed)

                response = await call_zaneru_api(link)
                time_taken = time.time() - start_time  

                if "error" in response:
                    error_embed = nextcord.Embed(color=nextcord.Color.red())
                    error_embed.timestamp = nextcord.utils.utcnow()
                    error_embed.add_field(name="Failed Message:", value=f"```{response['error']}```", inline=False)
                    error_embed.add_field(name="Execution Time:", value=f"```{time_taken:.2f} seconds```", inline=False)  
                    error_embed.set_thumbnail(url=avatar_url)  
                    error_embed.set_footer(text=f"Requested by {original_author.name}", icon_url=avatar_url)
                    await loading_message.edit(embed=error_embed, view=create_support_button())
                else:
                    bypass_key = response.get("key")
                    time_left = response.get("time_left", "N/A") 

                    result_embed = nextcord.Embed(color=nextcord.Color.green())
                    result_embed.timestamp = nextcord.utils.utcnow()
                    result_embed.add_field(name="Delta Key:", value=f"```{bypass_key}```", inline=False)  
                    result_embed.add_field(name="Time Left:", value=f"```{time_left}```", inline=False) 
                    result_embed.add_field(name="Execution Time:", value=f"```{time_taken:.2f} seconds```", inline=False)  
                    result_embed.set_thumbnail(url=avatar_url)  
                    result_embed.set_footer(text=f"Requested by {original_author.name}", icon_url=avatar_url)
                    await loading_message.edit(embed=result_embed, view=create_support_button())

            elif link.startswith("https://flux.li/android/external/start.php?HWID="):
                embed = nextcord.Embed(color=0x04ABEE)
                embed.add_field(name="Bypassing Fluxus:", value="```Waiting for result...```", inline=False)
                embed.add_field(name="Execution Time:", value="```Proccessing on result```", inline=False)
                embed.set_thumbnail(url=avatar_url)  
                embed.set_footer(text=f"Requested by {original_author.name}", icon_url=avatar_url)
                loading_message = await interaction.response.send_message(embed=embed)

                response = await call_zane_api(link)
                time_taken = time.time() - start_time  

                if "error" in response:
                    error_embed = nextcord.Embed(color=nextcord.Color.red())
                    error_embed.timestamp = nextcord.utils.utcnow()
                    error_embed.add_field(name="Failed Message:", value=f"```{response['error']}```", inline=False)
                    error_embed.add_field(name="Execution Time:", value=f"```{time_taken:.2f} seconds```", inline=False)  
                    error_embed.set_thumbnail(url=avatar_url)  
                    error_embed.set_footer(text=f"Requested by {original_author.name}", icon_url=avatar_url)
                    await loading_message.edit(embed=error_embed, view=create_support_button())
                else:
                    bypass_key = response.get("key")

                    result_embed = nextcord.Embed(color=nextcord.Color.green())
                    result_embed.timestamp = nextcord.utils.utcnow()
                    result_embed.add_field(name="Fluxus Key:", value=f"```{bypass_key}```", inline=False)  
                    result_embed.add_field(name="Execution Time:", value=f"```{time_taken:.2f} seconds```", inline=False)
                    result_embed.set_thumbnail(url=avatar_url)  
                    result_embed.set_footer(text=f"Requested by {original_author.name}", icon_url=avatar_url)
                    await loading_message.edit(embed=result_embed, view=create_support_button())

            elif link.startswith("https://getkey.relzscript.xyz/redirect.php?hwid="):
                embed = nextcord.Embed(color=0x04ABEE)
                embed.add_field(name="Bypassing Relz Hub:", value="```Waiting for result...```", inline=False)
                embed.add_field(name="Execution Time:", value="```Proccessing on result```", inline=False)
                embed.set_thumbnail(url=avatar_url)  
                embed.set_footer(text=f"Requested by {original_author.name}", icon_url=avatar_url)
                loading_message = await interaction.response.send_message(embed=embed)

                response = await call_zane2_api(link)
                time_taken = time.time() - start_time  

                if "error" in response:
                    error_embed = nextcord.Embed(color=nextcord.Color.red())
                    error_embed.timestamp = nextcord.utils.utcnow()
                    error_embed.add_field(name="Failed Message:", value=f"```{response['error']}```", inline=False)
                    error_embed.add_field(name="Execution Time:", value=f"```{time_taken:.2f} seconds```", inline=False)  
                    error_embed.set_thumbnail(url=avatar_url)  
                    error_embed.set_footer(text=f"Requested by {original_author.name}", icon_url=avatar_url)
                    await loading_message.edit(embed=error_embed, view=create_support_button())
                else:
                    bypass_key = response.get("key")

                    result_embed = nextcord.Embed(color=nextcord.Color.green())
                    result_embed.timestamp = nextcord.utils.utcnow()
                    result_embed.add_field(name="Relz Key:", value=f"```{bypass_key}```", inline=False)  
                    result_embed.add_field(name="Execution Time:", value=f"```{time_taken:.2f} seconds```", inline=False)
                    result_embed.set_thumbnail(url=avatar_url)  
                    result_embed.set_footer(text=f"Requested by {original_author.name}", icon_url=avatar_url)
                    await loading_message.edit(embed=result_embed, view=create_support_button())
        else:
            await interaction.response.send_message("**Invalid link provided. Only support delta, fluxus, and relz hub bypass**", ephemeral=True)
    except Exception as e:
        embed = nextcord.Embed(color=nextcord.
