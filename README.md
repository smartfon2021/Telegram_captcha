# Telegram_captcha

<p align="left">
    <img width="100%" height="100%" src="https://github.com/user-attachments/assets/eec4ee8b-2734-4d86-985f-53f5f4e752f2">
    <img width="35%" height="35%" src="https://github.com/user-attachments/assets/191d9a7b-5f28-46a4-9597-01bd3d7d1436">
</p>

## Overview

TLG_JoinCaptchaBot is a Telegram Bot designed to verify that new members joining a group are humans by presenting a [CAPTCHA challenge](https://en.wikipedia.org/wiki/CAPTCHA).

The bot:

- Automatically sends a CAPTCHA when a new user joins
- Removes users who fail to solve the CAPTCHA within a specified time limit
- Deletes messages containing URLs sent by users who haven't completed the CAPTCHA (anti-spam)
- Block new users to send media messages before the captcha is not solved
- Provides different captcha modes, like visual animated video captchas, image captchas, custom poll captchas, etc
- Has multilanguage support with near to ~30 languages
- Allows custom configuration per group (time for solving the captcha, captcha mode, difficulty, language, welcome message, etc)
- Keeps the groups clean from Bot and captcha messages by automatically self-removing them after a while.

## Table of Contents

- [Requirements](#requirements)
- [Installation](#installation)
- [Configuration](#configuration)
- [Usage](#usage)
- [Deployment](#deployment)
  - [Systemd Service](#systemd-service)
  - [Docker](#docker)
- [Advanced Features](#advanced-features)
  - [Bot Owner Commands](#bot-owner)
  - [Private Mode](#make-bot-private)
  - [Scalability (Polling vs Webhook)](#scalability-polling-or-webhook)
- [Localization](#adding-a-new-language)


## Requirements

- Python 3.12+
- Manim requirements
- Pillow requirements
- Telegram Bot Token (from [@BotFather](https://t.me/BotFather))

## Installation

sudo apt-get install build-essential make python3 python3-dev python3-pip
sudo apt-get install libcairo2-dev libpango1.0-dev

### 1. Install Python3 and tools

```bash
sudo apt-get install python3 python3-pip python3-venv
```

### 2. Install Pillow & Manim prerequisites

```bash
sudo apt update
sudo apt install -y build-essential make libcairo2-dev libpango1.0-dev
sudo apt install -y libtiff5-dev libjpeg62-turbo-dev zlib1g-dev libfreetype6-dev liblcms2-dev libwebp-dev tcl8.6-dev tk8.6-dev python3-tk
```

### 3. Get and setup the project

```bash
https://github.com/smartfon2021/Telegram_captcha.git
cd Telegram_captcha
make setup
```

### 4. Configure your Telegram Bot token

Edit the `src/settings.py` file:

```python
'TOKEN' : 'XXXXXXXXX:XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX'
```

## Configuration

Configuration is handled through the `src/settings.json` file. Advanced users can also use environment variables, which is useful for deployment scenarios with [virtual environments](https://docs.python.org/3/tutorial/venv.html) or [Docker](https://docs.docker.com/get-started/).

## Usage

A `Makefile` is provided for convenient operation:

```bash
# View available commands
make

# Start the bot
make start

# Check bot status
make status

# Stop the bot
make stop
```

## Deployment

### Systemd Service

To run the bot as a daemon service on systemd-based systems:

1. Create a service file:

```bash
sudo nano /etc/systemd/system/Telegram_captcha.service
```

2. Add the following content (adjust paths as needed):

```
[Unit]
Description=Telegram captcha Bot Daemon
Wants=network-online.target
After=network-online.target

[Service]
Type=forking
WorkingDirectory=/path/to/Telegram_captcha/src/
ExecStart=/path/to/Telegram_captcha/tools/start
ExecReload=/path/to/Telegram_captcha/tools/kill

[Install]
WantedBy=multi-user.target
```

3. Enable and start the service:

```bash
sudo systemctl enable --now Telegram_captcha.service
sudo systemctl start Telegram_captcha.service
```

4. Check service status:

```bash
sudo systemctl status Telegram_captcha.service
```

### Docker

Docker support is available for easy deployment and server migration. See the [Docker specific documentation](docker/README.md) for details on creating a Docker container for the bot.

## Advanced Features

### Bot Owner

The bot owner can execute special commands:
- `/allowgroup`: Add groups to the allowed list (when bot is private)
- `/allowuserlist`: Exempt specific users from CAPTCHA verification (useful for accessibility needs)

Set a bot owner in `settings.py`:

```python
"BOT_OWNER": "@YourUsername",
```

### Make Bot Private

By default, anyone can add the bot to any group. To restrict usage to specific groups:

1. Set the bot to private mode in `settings.py`:

```python
"BOT_PRIVATE": True,
```

2. Use the `/allow_group` command to specify permitted groups.

**Note:** If a public group becomes a supergroup, the chat ID may change, requiring re-authorization.

### Scalability (Polling or Webhook)

#### Polling (Default)
The bot periodically checks for updates from Telegram servers. This is suitable for small to medium deployments.

#### Webhook (For larger deployments)
The bot receives updates directly from Telegram servers, which improves performance for high-traffic bots.

To configure webhook:

1. Generate SSL certificate:

```bash
openssl req -newkey rsa:2048 -sha256 -nodes -keyout private.key -x509 -days 3650 -out cert.pem
```

2. Configure webhook settings in `settings.py`:

```python
"WEBHOOK_IP": "0.0.0.0",
"WEBHOOK_PORT": 8443,
"WEBHOOK_PATH": "/Telegram_captcha",
"WEBHOOK_CERT": SCRIPT_PATH + "/cert.pem",
"WEBHOOK_CERT_PRIV_KEY": SCRIPT_PATH + "/private.key",
```

3. For reverse proxy setups (optional):

```python
"WEBHOOK_URL": "https://example.com:8443/Telegram_captcha"
```

4. Enable webhook mode:

```python
"CAPTCHABOT_USE_WEBHOOK": True,
```

## Adding a New Language

The bot supports multiple languages through external JSON files. To add a new language:

1. Fork the repository and create a new branch (e.g., `language-support-xx`)
2. Copy an existing language file from the [language directory](https://github.com/smartfon2021/Telegram_captcha/tree/master/src/language)
3. Rename the file to the ISO code of your target language
4. Translate all text values while maintaining:
   - The JSON structure and key names
   - Command names in English (`START`, `HELP`, etc.)
   - Special characters (`{}`, `"`, `'`, `\n`, etc.)
5. Submit a pull request with your translation


