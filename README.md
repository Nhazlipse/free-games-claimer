# free-games-claimer
Claims free games on
- [Epic Games Store](https://www.epicgames.com/store/free-games)
- [Amazon Prime Gaming](https://gaming.amazon.com)
- PRs welcome :)

## Setup
... should be the same on Windows/macOS/Linux:

1. [Install Node.js](https://nodejs.org/en/download)
2. Clone/download this repository and `cd` into it in a terminal
3. Run `npm install && npx playwright install`

This downloads {chromium, firefox, webkit} (742 MB) to a cache in home ([doc](https://playwright.dev/docs/browsers#managing-browser-binaries)).

## Usage
<!-- Use `npm run login` which opens a browser where you can login. When closing the browser, it writes a file `auth.json` containing cookies that should keep you logged in for some time (`expires` in a month?). -->

Both scripts start an automated Chrome instance. It will first check if you are logged in, and if not wait for you to do so. After login, you can also restart the script if it does not redirect back.

If something goes wrong, use `PWDEBUG=1 node epic-games` to [inspect](https://playwright.dev/docs/inspector).

Ideally, claiming would run in *headless mode* (without browser GUI - comment out `headless: false` to test), and on a Raspberry Pi:
- Epic Games Store detects running in headless mode (despite stealth plugin) and gets stuck with a captcha challenge ([issue](https://github.com/vogler/free-games-claimer/issues/2)). Did not test it yet for Prime Gaming.
- Playwright seems to not run on (headless) RPi? See [issue](https://github.com/vogler/free-games-claimer/issues/3).

### Epic Games Store
Run `node epic-games`

Login: Instead of redirecting back, the website seems to just reload the login URL. Go to https://www.epicgames.com/store/en-US/free-games manually, or restart the script.

### Amazon Prime Gaming
Run `node prime-gaming` 

Claiming the Amazon Games works, external Epic Games also work if the account is linked.
Origin needs testing - it shows a key, which should be printed to the console, but the selector may be wrong.
Other stores not tested.

### Run periodically
Epic Games releases one (sometimes more) free game *every week*, but around christmas every day.
Prime Gaming has new games *every month*.

It is save to run both scripts every day. Since they are not running headless, it makes sense to run them at a time or on a machine that you are not actively using at that point. You could run them in a virtual machine, on a server, or you wake your PC at night to do it.

- Linux/macOS: `crontab -e`
- macOS: [launchd](https://stackoverflow.com/questions/132955/how-do-i-set-a-task-to-run-every-so-often)
- Windows: [task scheduler](https://active-directory-wp.com/docs/Usage/How_to_add_a_cron_job_on_Windows/Scheduled_tasks_and_cron_jobs_on_Windows/index.html), [other options](https://stackoverflow.com/questions/132971/what-is-the-windows-version-of-cron)

## History/DevLog
<details>
  <summary>Click to expand</summary>

Tried [epicgames-freebies-claimer](https://github.com/Revadike/epicgames-freebies-claimer), but does not work anymore since epicgames introduced hcaptcha (see [issue](https://github.com/Revadike/epicgames-freebies-claimer/issues/172)).

Played around with puppeteer before, now trying newer https://playwright.dev which is pretty similar.
Playwright Inspector and `codegen` to generate scripts are nice, but failed to generate the right code for clicking a button in an iframe.

Added [main.spec.ts](https://github.com/vogler/epicgames-claimer/commit/e5ce7916ab6329cfc7134677c4d89c2b3fa3ba97#diff-d18d03e9c407a20e05fbf03cbd6f9299857740544fb6b50d6a70b9c6fbc35831) which was the test script generated by `npx playwright codegen` with manual fix for clicking buttons in the created iframe. Can be executed by `npx playwright test`. The test runner has options `--debug` and `--timeout` and can execute typescript which is nice. However, this only worked up to the button 'I Agree', and then showed an hcaptcha.

Added [main.captcha.js](https://github.com/vogler/epicgames-claimer/commit/e5ce7916ab6329cfc7134677c4d89c2b3fa3ba97#diff-d18d03e9c407a20e05fbf03cbd6f9299857740544fb6b50d6a70b9c6fbc35831) which uses beta of `playwright-extra@next` and `@extra/recaptcha@next` (from [comment on puppeteer-extra](https://github.com/berstend/puppeteer-extra/pull/303#issuecomment-775277480)).
However, `playwright-extra` seems to be old and missing `:has-text` selector (fixed [here](https://github.com/vogler/epicgames-claimer/commit/ba97a0e840b65f4476cca18e28d8461b0c703420)) and `page.frameLocator`, so the script did not run without adjustments.
Also, solving via [2captcha](https://2captcha.com?from=13225256) is a paid service which takes time and may be unreliable.
<!-- Alternative: https://anti-captcha.com -->

Added [main.stealth.js](https://github.com/vogler/epicgames-claimer/commit/64d0ba8ce71baec3947d1b64acd567befcb39340#diff-f70d3bd29df4a343f11062a97063953173491ce30fe34f69a0fc52517adbf342) which uses the stealth plugin without `playwright-extra` wrapper but up-to-date `playwright` (from [comment](https://github.com/berstend/puppeteer-extra/issues/454#issuecomment-917437212)).
The listed evasions are enough to not show an hcaptcha. Script claimed game successfully in non-headless mode.

Removed `main.captcha.js`.
Using Playwright Test (`main.spec.ts`) instead of Library (`main.stealth.js`) has the advantage of free CLI like `--debug` and `--timeout`.
<!-- TODO: check if stealth plugin can be setup with `contextOptions` ([doc](https://playwright.dev/docs/test-configuration#more-browser-and-context-options)). -->

Button selectors should preferably use text in order to be more stable against changes in the DOM.

Renamed repository from epicgames-claimer to free-games-claimer since a script for Amazon Prime Gaming was also added. Removed all old scripts in favor of just `epic-games.js` and `prime-gaming.js`.

epic games: `headless` mode gets hcaptcha challenge. More details/references in [issue](https://github.com/vogler/free-games-claimer/issues/2).

</details>
