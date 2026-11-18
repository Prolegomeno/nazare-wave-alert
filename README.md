# Nazare Wave Alert

An n8n automation that tells you when giant waves are forecast at Nazaré, Portugal. It checks the swell forecast every morning and sends you one Telegram message if the conditions look right for giant waves. The rest of the time it stays silent.

## How it works

1. **Start Every Morning** runs the workflow at 08:00. **Start Manually** does the same for tests.
2. **Get Swell Forecast** asks Open-Meteo for the next seven days of swell at a point in the open sea in front of Nazaré.
3. **Split Forecast Hours** turns the forecast into one item per hour.
4. **Keep Giant Wave Hours** keeps only the hours that meet the rule below.
5. **Keep First Giant Hour** keeps the earliest of them.
6. **Get Telegram Chat Id** reads your chat number from a Data Table.
7. **Combine Hour and Chat Id** joins the hour and the chat number into one item.
8. **Send Wave Alert** sends the message.

If no hour meets the rule, the workflow ends at step 4 and nothing is sent.

### The rule

An hour counts as a giant wave hour when all three of these are true:

- swell height is 6 m or more,
- swell period is 15 s or more,
- swell direction is between 270 and 340 degrees (west to north-west).

These numbers are an estimate based on public descriptions of how the Nazaré canyon amplifies the swell. They are not an official rule. Change them in **Keep Giant Wave Hours**.

## Security rules

- **No secrets in the workflow.** The bot token lives in an n8n credential. Your chat number lives in a Data Table. Exporting the workflow exports neither.
- **Fail closed.** If the forecast request fails, the workflow stops and sends nothing.
- **A missing chat number is loud, not silent.** **Get Telegram Chat Id** is set to always output an item. If the row is missing, the Telegram step fails with an error instead of the run ending quietly.
- **No pinned data.** Test data is pinned only while you test. Remove it before you publish.
- **One direction only.** The workflow reads a public forecast and writes to one chat. It accepts no incoming messages.

## What you need

- n8n 2.x with Data Tables.
- A Telegram account and a bot made with BotFather.
- Nothing for the forecast. Open-Meteo needs no key for non-commercial use. Credit Open-Meteo wherever you share the data.
- Optional: an error workflow, such as `Utility - Notify Error` from [living-portfolio](https://github.com/Prolegomeno/living-portfolio).

## Setup

1. In Telegram, ask BotFather for a new bot with `/newbot`. Keep the token to yourself.
2. In n8n, create a Telegram credential and paste the token there.
3. Send your new bot any message. A bot cannot write to you first.
4. Find your chat number. Open `https://api.telegram.org/botTOKEN/getUpdates` in a private browser window, replacing `TOKEN` with your bot token. The number is the `id` inside `chat`. Close the window afterwards.
5. Create a Data Table with two text columns, `name` and `value`. Add one row: `name` = `telegram_Id`, `value` = your chat number.
6. Import `workflows/nazare-wave-alert.json`.
7. Open **Get Telegram Chat Id** and choose your table. Open **Send Wave Alert** and choose your credential.
8. Test it. Pin `docs/test-forecast.json` as the output of **Get Swell Forecast** and run **Start Manually**. You should get one message about the hour `2026-11-20T05:00`. Then remove the pin.
9. Optional: in the workflow settings, choose your error workflow and your time zone.
10. Publish the workflow. n8n 2.x runs the published version, so publish again after every edit.

## Good to know

- **It is a forecast, not a certainty.** The rule uses the swell in the open sea. The canyon makes the wave at the beach much bigger than that.
- **It repeats.** While a giant swell stays inside the seven-day forecast, you get the message every morning. Each one is an update.
- **Keep the longitude negative.** Nazaré is west of Greenwich. A positive longitude lands in the Mediterranean, where the rule never fires and the workflow stays silent.
- **Time zone.** The hours in the message follow the `timezone` query parameter (`Europe/Madrid`). Change it in **Get Swell Forecast**.
- **Credit.** Forecast data from Open-Meteo.

## Licence

MIT. See `LICENSE`.
