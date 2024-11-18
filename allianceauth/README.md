# AllianceAuth - first deployment guide

This deployment assumes that there's an existing docker network `web` with a running instance of traefik.

## Prerequisites

### Subdomain

Prepare a new subdomain to host AllianceAuth on beforehand. `aa` could be a good choice.

### ESI application

Visit [EVE Developer portal](https://developers.eveonline.com/) and create an application. Set callback URL to `https://subdomain.example.com/sso/callback` substituting your domain and subdomain. Choose "Authentication & API Access" connection type and select the following scopes:

* `publicData`
* `esi-universe.read_structures.v1`
* `esi-corporations.read_structures.v1`
* `esi-characters.read_notifications.v1`
* `esi-assets.read_corporation_assets.v1`
* `esi-corporations.read_starbases.v1`
* `esi-planets.read_customs_offices.v1`

Keep in mind: installing new modules in the future might require adding more scopes.

### Email address

When using ESI, CCP requires to provide an email address in UserAgent for contact in case of issues. You will need to provide one during AllianceAuth deployment.

### Discord application

Visit [Discord Developer portal](https://discord.com/developers/applications) and create an application.

In "Installation tab", uncheck "User Install".

Go to "Bot" tab and click "Reset Token". Save the generated token. 

Go to "OAuth2" tab and add a redirect to `https://subdomain.example.com/discord/callback` substituting your domain and subdomain. On the same page, save "Client ID", click "Reset Secret" and save it as well.

At the bottom on the page, use the link generator to create an invite link with a single scope `bot`. Use this link to invite the bot of the created application to the server where AllianceAuth will be operating.

### Discord server ID

First, make sure Developer mode is enabled in discord: go to settings (gear at the botton left near you name), Advanced, the first switch should be enabled.

Then, in the left pane with server icons, click on the server AllianceAuth should operate in with right mouse button to bring up the context menu, and click "Copy Server ID" at the bottom. Save that ID.

## Deployment

Run `./prepare-env.sh` and follow instructions

Then, edit generated `.env` file to fill in Discord-related fields at the bottom of the file:

* `DISCORD_APP_ID` - app ID from Discord app
* `DISCORD_APP_SECRET` - oauth2 app secret from discord app
* `DISCORD_BOT_TOKEN` - bot token from discord app
* `DISCORD_GUILD_ID` - discord server ID

Then, start all the containers:

```
docker compose --env-file=.env up -d
```

After that, shell into the running gunicord container:

```
docker compose exec allianceauth_gunicorn bash
```

And run this (the last command will require you to input `Y`):

```
auth migrate
auth collectstatic
auth structures_load_eve
```

Lastly, create superuser account:

```
auth createsuperuser
```

Now you can login with the superuser account at https://subdomain.example.com/admin