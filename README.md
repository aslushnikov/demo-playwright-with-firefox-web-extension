# Running Firefox WebExtensions with Playwright

Demo on running Firefox WebExtension with Playwright.
See https://github.com/microsoft/playwright/issues/2644 for details.

- We load a simple web extension (lives at `//webextension`)
- We use playwright to connect to the browser with webextension.

```js
const {firefox} = require('playwright');
const webExt = require('web-ext').default;

(async () => {
  // Enable verbose logging and start capturing logs.
  webExt.util.logger.consoleStream.makeVerbose();
  webExt.util.logger.consoleStream.startCapturing();

  // Launch firefox
  const runner = await webExt.cmd.run({
    sourceDir: path.join(__dirname, 'webextension'),
    firefox: firefox.executablePath(),
    args: [`-juggler=1234`],
  }, {
    shouldExitProgram: false,
  });

  // Parse firefox logs and extract juggler endpoint.
  const JUGGLER_MESSAGE = `Juggler listening on`;
  const message = webExt.util.logger.consoleStream.capturedMessages.find(msg => msg.includes(JUGGLER_MESSAGE));
  const wsEndpoint = message.split(JUGGLER_MESSAGE).pop();

  // Connect playwright and start driving browser.
  const browser = await firefox.connect({ wsEndpoint });
  const page = await browser.newPage();
  await page.goto('https://mozilla.org');
  // .... go on driving ....
})();
```
