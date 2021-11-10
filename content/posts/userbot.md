---
title: "Setting Up a Telegram Userbot"
date: 2020-11-30T00:00:00+00:00
draft: false
---

## Why do I need one?

A Telegram userbot helps you manage groups, sticker packs, your inbox; do quick
web searches, download music, mirror files to your Google Drive, and hundreds
of other commands. It's not an actual bot that has its own username like Miss
Rose bot. Userbots run as a session of your own account, it responds to
commands you give out, as you. For example, if I send `.currency 100 inr usd`
in any group/PM, the userbot will detect it, convert the currency from INR to
USD, and will edit that message to give you the output in that message itself.

## Is it safe to use one?

The short answer is yes. The long answer is that it depends on your usage of
the bot. There are certain commands that you must use with extreme caution,
such as `.term`, `.exec` and `.eval`. These commands allow you to run snippets
of code from the userbot's host. There is some basic protection applied to
these commands but you'll have to check the code and make sure it's safe before
using it. There are a few commands that spam Telegram's servers such as
`.spam`, `.scam` and anything else that sends out messages/edit in rapid
succession. The bot or its devs are not responsible for your misuse of these
commands, use them at your own discretion.

## Where do I host it?

The easiest way to host the userbot is on [Heroku](https://heroku.com). They
give you 500 hours per month for free. If you add your billing info (no need to
pay anything, just giving them your address and credit card numbers), they give
you an additional 500 hours for free.

## Which bot do I use?

I maintain a userbot called [KensurBot](https://github.com/KenHV/KensurBot),
which has a few exclusive modules that I wrote, and is overall sane. As of
writing, it supports 210 commands. I keep it up-to-date and I fix bugs
relatively fast. You can use other userbots too, the procedure is mostly the
same but I'll use my bot for this guide.

## What do I need?

First of all you'll need Telegram's API. To get your own API ID and API Hash,
do this:

- Log in to your Telegram core: https://my.telegram.org. Depending on your
  country, you might need a VPN.
- Go to 'API development tools' and fill out the form for creating an app (you
  can fill in anything, it doesn't matter).
- You will get basic addresses as well as the `api_id` and `api_hash`
  parameters required for user authorization.

Next up, you'll need to generate a session string. This will allow your bot to
login as _you_. Think of it as a separate device that's logged in to your
account. I've simplified this process, do this:

- Go to [http://sessiongen.kenhv.repl.run](http://sessiongen.kenhv.repl.run)
  and click on `Run` at the top.
- Enter the `API_ID` and `API_HASH` that you got from the previous steps.
- Enter your phone number with country code (for eg, +919876543210) and follow
  the instructions.
- Check the _Saved Messages_ section in your Telegram, your session string will
  be sent there.
- This value is your `SESSION_STRING`.

Do _NOT_ share this session string anywhere, as this is essentially a key that
allows someone (or in our case, the userbot) to log in to your account.

That's pretty much everything you need for getting the userbot up and running,
there are other stuff that you might want in the future such as GDrive
management, LastFM integration, and these will need their own credentials and
API keys but that's out of the scope of this guide.

You'll need to make a separate group in your Telegram for the userbot to store
its logs and other info.

- Go to Telegram and create a group.
- Add `@MissRose_bot` to your group.
- Send `/id` in the group, Rose will give you the Chat ID of your group.
- The Chat ID will have a `-` sign at the start, don't leave out the `-` sign.
- This value is your `BOTLOG_CHATID`.

Make a Heroku account, add in your billing info if you want the additional 500
hours of uptime. Google will help you with this part. Head to
https://dashboard.heroku.com/account and get your API key.

## How do I deploy the bot in Heroku?

Now that the hard part is done, the rest is pretty straightforward.

- Head to [KensurBot's GitHub repo](https://github.com/kenhv/kensurbot) and
  click on _Deploy to Heroku_ under the _Setting Up_ section. Or just click
  [here](https://heroku.com/deploy?template=https://github.com/KenHV/KensurBot/tree/sql-extended).
  This will take you to Heroku's deploy page.
- Fill in the app name, you can put anything you want here. As for the region,
  choose whichever one is closest to your physical location.
- Fill in the `API_KEY` (API_ID), `API_HASH`, `SESSION_STRING`, `BOTLOG_CHATID`
  and `HEROKU_API_KEY` you got from the previous section.
- Copy the name of your app that you entered at the start to `HEROKU_APP_NAME`
  _as it is_.
- Set `ALIVE_NAME` to the name you want the userbot to display when you send
  `.alive` (don't worry you can change this all later at any time).
- Read through the rest of the config options and add/change them if you need
  (like the above step you can change these at any time).
- Leave the logging options as they are (KensurBot has logging enabled by
  default).
- Click on _Deploy App_ at the bottom of the page.
- If all goes well, you should see _Your app was successfully deployed_.
- Click on the _Manage App_ button.
- Go to _Resources_ tab, and make sure the _worker_ is on. If not, click on the
  pencil icon and turn it on and click _Confirm_.
- Click on the _More_ button (above the tabs, and right next to _Open App_
  button), and select _View logs_.
- If all went well, you should see something like this at the bottom:
  `Congratulations, the bot is up and running! Send .help in any chat for more info.`
- Like it says, send `.help` in any chat for info on how to use the commands.

## Updates

Send `.update` and the bot will check for updates. If there are any, you can
send `.update deploy` and the bot will update itself.

## Support group

In case you face errors during deploying the bot, face any crashes, have
questions, or just want to hang out in a dumb off-topic group, head to my
[Telegram group](https://t.me/KensurOT). Check out my [Telegram
channel](https://t.me/KenVerse), I post all my projects there and share some
stuff/resources.

Peace out!
