# Discord Bot

Self-hosted Discord bot with slash commands for muting / vote-muting / russian roulette, reaction roles, AMP game-server watching + control (any game AMP manages), music playback (YouTube, SoundCloud, Spotify metadata, etc) via Lavalink, and DM alerts for new SimplifyJobs off-season internship listings. Runs lean — bot itself is ~30-40 MB RAM, audio work is offloaded to a separate Lavalink container.

## Commands

### Moderation & fun
| Command | Description |
|---|---|
| `/mute <user> <duration>` | Mute a user (e.g. `30s`, `2m`). Requires Mute Members permission. |
| `/unmute <user>` | Unmute a user immediately. Requires Mute Members permission. |
| `/votemute <user> <duration>` | Start a vote to mute someone in your voice channel. |
| `/russianroulette` | Randomly mutes someone in your voice channel for 10-60s. 30s cooldown. |
| `/reactionrole <channel> <message_id> <emoji> <role>` | Set up a reaction role on a message. Requires Manage Roles. |
| `/rolepanel <title> <emoji1> <role1> ...` | Send a panel message with up to 4 emoji/role pairs. Requires Manage Roles. |
| `/removereactionrole <message_id> <emoji>` | Remove a reaction role. Requires Manage Roles. |

### Game servers (AMP)

Both commands talk to [AMP](https://cubecoders.com/AMP), so they work for **any game AMP manages** (Minecraft, Valheim, Rust, Terraria, etc.). The `server` option autocompletes from your ADS instance list. Requires `AMP_URL`, `AMP_USERNAME`, and `AMP_PASSWORD` to be set.

| Command | Description |
|---|---|
| `/serverwatch add <server> <channel>` | Watch an AMP server and announce online/offline + player-count changes in `<channel>`. Requires Manage Server. |
| `/serverwatch remove <server>` | Stop watching a server. Requires Manage Server. |
| `/serverwatch list` | List watched servers and their last-known state. |
| `/gameserver start\|stop\|restart <server>` | Control an AMP-managed server. `server` is **required** and autocompletes from your ADS instances. Requires a role named **minecraft** by default (configurable via `GAMESERVER_ROLE`). |
| `/gameserver status <server>` | Read-only AMP status (state + player count) for the chosen server. `server` is **required**. Open to anyone. |

> **Watching** reports the server's running state and a live **player count** (e.g. *"2 players joined — now 5 online"*) — it does not list individual player names. Player names aren't exposed uniformly across games by AMP; ask if you want name-level tracking for Minecraft specifically. A server that's stopped at the AMP panel level shows as **offline (Unavailable)**.

#### AMP environment variables

| Variable | Default | Description |
|---|---|---|
| `AMP_URL` | _(unset)_ | Base URL of your AMP / ADS controller (e.g. `https://amp.example.com`). |
| `AMP_USERNAME` / `AMP_PASSWORD` | _(unset)_ | AMP login credentials. |
| `AMP_INSTANCE` | _(unset)_ | Optional fallback instance for single-server (non-ADS) AMP setups. The `/gameserver` and `/serverwatch` commands always take an explicit `server`, so this isn't needed when using ADS. |
| `GAMESERVER_ROLE` | `minecraft` | Role name (case-insensitive) required to run `/gameserver start\|stop\|restart`. |
| `AMP_DEBUG` | _(off)_ | Set to `1` to log every AMP API request/response. Off by default to keep logs quiet (the watch polls every 15s). |

### Internships
Subscribers get a DM whenever a new listing is added to the [SimplifyJobs Off-Season Internships](https://github.com/SimplifyJobs/Summer2026-Internships/blob/dev/README-Off-Season.md) README. The bot polls every 10 minutes; the first poll seeds a baseline so existing listings don't flood your DMs.

| Command | Description |
|---|---|
| `/internships subscribe` | Get DMed when new internships are posted. The bot sends a confirmation DM, so DMs from server members must be enabled. |
| `/internships unsubscribe` | Stop receiving internship DMs. |
| `/internships status` | Show whether you're subscribed and how many listings are being tracked. |

### Music (only registered when Lavalink is configured)
| Command | Description |
|---|---|
| `/play <query>` | Play a song from a URL or search query. Joins your voice channel. |
| `/skip` | Skip the current song. |
| `/stop` | Stop playback and clear the queue. |
| `/queue` | Show the song queue. |
| `/nowplaying` | Show the currently playing track. |
| `/pause` / `/resume` | Pause or resume playback. |
| `/volume <level>` | Set playback volume (0-150). |

## Discord Bot Setup

1. Go to the [Discord Developer Portal](https://discord.com/developers/applications)
2. Create a new application and add a bot
3. Under **Bot**, enable these **Privileged Gateway Intents**:
   - Server Members Intent
   - Message Content Intent (not strictly required, but recommended)
4. Under **OAuth2 > URL Generator**, select:
   - Scopes: `bot`, `applications.commands`
   - Bot Permissions: `Manage Roles`, `Mute Members`, `Send Messages`, `Add Reactions`, `Read Message History`
5. Use the generated URL to invite the bot to your server
6. Copy the bot token from the **Bot** page -- you'll need it below

## Running Locally

Requires Node.js 18+.

```bash
npm install
BOT_TOKEN=your_token_here npm start
```

## Deploying on Unraid 7.2.3

First, copy the project to your Unraid server:

```bash
scp -r . root@<UNRAID_IP>:/mnt/user/appdata/discord-mute-bot/
```

Then SSH in and build the Docker image:

```bash
ssh root@<UNRAID_IP>
cd /mnt/user/appdata/discord-mute-bot
docker build -t discord-mute-bot .
```

### Option A: Docker UI (shows in the Docker tab)

This uses the included XML template so the container shows up in Unraid's Docker tab with a proper icon and editable settings.

1. **Install the template** by copying it to Unraid's template directory:
   ```bash
   cp /mnt/user/appdata/discord-mute-bot/unraid-template.xml \
      /boot/config/plugins/dockerMan/templates-user/my-discord-mute-bot.xml
   ```

2. In the Unraid web UI, go to **Docker** tab and click **Add Container**.

3. At the top, click **Select a template** and choose **discord-mute-bot** from the dropdown.

4. The form auto-fills. All you need to do is paste your **Bot Token** into the token field.

5. Click **Apply**. The container will start and appear in the Docker tab with a Discord icon.

6. Click the container's icon in the Docker tab to access **Start**, **Stop**, **Logs**, and **Edit** options.

### Option B: Docker Compose (also shows in the Docker tab)

Unraid 7.2.3 has built-in Compose support, and compose stacks appear in the Docker tab.

1. **Edit the compose file** to add your token:
   ```bash
   cd /mnt/user/appdata/discord-mute-bot
   nano docker-compose.yml
   ```
   Replace `your_token_here` with your actual bot token.

2. **Add the stack in the Unraid UI:**
   - Go to the **Docker** tab
   - Click **Compose** at the top
   - Click **Add New Stack**
   - Set the name to `discord-mute-bot`
   - Set the compose path to `/mnt/user/appdata/discord-mute-bot/docker-compose.yml`
   - Click **Save** and then **Compose Up**

3. The stack will appear in the Docker tab. You can start/stop/view logs from the UI.

   Alternatively, from the terminal:
   ```bash
   cd /mnt/user/appdata/discord-mute-bot
   docker compose up -d
   ```

### Verifying it works

Check the logs either from the Docker tab (click the container icon > Logs) or via terminal:

```bash
docker logs discord-mute-bot
```

You should see:
```
Logged in as YourBot#1234
Synced 6 commands.
```

### Updating the bot

After making code changes:

```bash
cd /mnt/user/appdata/discord-mute-bot
docker build -t discord-mute-bot .
docker restart discord-mute-bot
```

Or with compose: `docker compose up -d --build`

### Auto-start on boot

Both methods auto-restart on reboot. The template uses `--restart=unless-stopped` and the compose file sets `restart: unless-stopped`.

## Music (Lavalink) setup

Music commands are powered by [Lavalink v4](https://github.com/lavalink-devs/Lavalink), which runs as a separate container next to the bot. The bot only sends commands over WebSocket — all the audio decoding, Opus encoding, and voice gateway work happens inside the JVM, so it stays out of Node's event loop and YouTube anti-bot breakage is patched upstream by the [youtube-source plugin](https://github.com/lavalink-devs/youtube-source) maintainers. YouTube does still break periodically, and there is a one-time sign-in step you have to do yourself; see [YouTube playback](#youtube-playback) below.

If you used the included `docker-compose.yml`, Lavalink is already wired up. Just make sure these env vars are set on the bot container:

| Variable | Default | Description |
|---|---|---|
| `LAVALINK_HOST` | `lavalink` | Hostname of the Lavalink server (Compose service name when colocated). |
| `LAVALINK_PORT` | `2333` | Port Lavalink listens on. |
| `LAVALINK_PASSWORD` | _(unset)_ | Must match the `password` in `lavalink/application.yml`. **Music commands are only registered when this is set.** |

### YouTube playback

YouTube blocks most anonymous playback, so the bundled `lavalink/application.yml` does three things. If music stops working, check these first.

1. **Plugin version.** The config pins a `main`-branch snapshot of youtube-source (a full 40-character commit hash with `snapshot: true`) because the newest release predates YouTube's August 2026 changes. To update, pick the newest hash from the [snapshot list](https://maven.lavalink.dev/#/snapshots/dev/lavalink/youtube/youtube-plugin) (or a new release once one ships, with `snapshot: false`) and restart the Lavalink container. A `FileNotFoundException` for the plugin jar on startup means that version/`snapshot` combination doesn't exist.
2. **One-time Google sign-in (OAuth).** The `TV` client is the most reliable one and only works signed in. Use a throwaway Google account, never your main one:
   1. Start Lavalink without `YOUTUBE_REFRESH_TOKEN` set and run `docker logs -f discord-mute-bot-lavalink`.
   2. Look for `OAUTH INTEGRATION: To give youtube-source access to your account, go to https://www.google.com/device and enter code ...`. Open the link, sign in with the burner account, enter the code.
   3. The log then prints `OAUTH INTEGRATION: Token retrieved successfully ... (1//0g...)`. Put that value in `YOUTUBE_REFRESH_TOKEN` on the Lavalink container and restart it. The prompt won't appear again.
3. **Signature cipher server.** The plugin no longer deciphers stream signatures itself; it asks a [yt-cipher](https://github.com/kikkia/yt-cipher) server. The config defaults to the public instance at `https://cipher.kikkia.dev/`. Self-host it and set `YOUTUBE_CIPHER_URL` if you prefer.

The `WEB`/`WEBEMBEDDED` clients additionally need a poToken to play anything (see the plugin README); they stay in the list as a last resort and for metadata lookups.

When every client fails, Lavalink logs `AllClientsFailedException` with one line per client, e.g. `TVHTML5 failed: The page needs to be reloaded` (plugin too old), `This video requires login` (no OAuth token, or YouTube flagged the IP), `WEB failed: No supported audio streams available` (no poToken).

### Optional: Spotify links

The bundled config only resolves direct URLs from YouTube/SoundCloud/Bandcamp/Twitch/Vimeo and YouTube search queries. To play Spotify URLs (which the Spotify API doesn't allow streaming directly), add the [LavaSrc plugin](https://github.com/topi314/LavaSrc) to `lavalink/application.yml` with your Spotify client ID/secret — it resolves Spotify tracks to YouTube and plays them transparently.

## Resource Usage

The bot is configured to run lean:
- V8 heap capped at 64 MB
- Message cache limited to 50 per channel
- Presence cache disabled
- Old messages swept every 5 minutes
- Alpine-based Docker image (~50 MB)

Typical idle RAM usage for the bot: **~30-40 MB**.

When music is enabled, audio work runs inside the Lavalink container (~256 MB JVM heap by default in the included compose file). Per concurrent stream Lavalink uses roughly 30-80 MB extra and a few percent of one CPU core.
