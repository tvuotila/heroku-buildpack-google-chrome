## WARNING: `heroku-buildpack-google-chrome` is deprecated

This buildpack is deprecated and is no longer being maintained. Use (https://github.com/heroku/heroku-buildpack-chrome-for-testing.

# heroku-buildpack-google-chrome

This buildpack downloads and installs (headless) Google Chrome for Testing. Requires [Heroku APT Buildpack](https://github.com/heroku/heroku-buildpack-apt.git) after this buildpack in order to install dependencies.

## Shims and Command Line Flags

This buildpack installs shims that always add `--headless`, `--disable-gpu`, 
`--no-sandbox`, and `--remote-debugging-port=9222` to any `google-chrome` 
command as you'll have trouble running Chrome on a Heroku dyno otherwise.

You'll have two of these shims on your path: `google-chrome` and
`google-chrome-$GOOGLE_CHROME_CHANNEL`. They both point to the binary of
the selected channel.

## Selenium

To use Selenium with this buildpack, you'll also need Chrome's webdriver.
This buildpack does not install chromedriver, but there is a
[chromedriver buildpack](https://github.com/tvuotila/heroku-buildpack-chromedriver)
also available.

Additionally, chromedriver expects Chrome to be installed at `/usr/bin/google-chrome`,
but that's a read-only filesystem in a Heroku slug. You'll need to tell Selenium/chromedriver
that the chrome binary is at `/app/.chrome/bin/chrome` instead.

To make that easier, this buildpack makes `$GOOGLE_CHROME_BIN`, and
`$GOOGLE_CHROME_SHIM` available as environment variables. With them, you can 
use the standard location locally and the custom location on Heroku. An example 
configuration for Ruby's Capybara:

```
chrome_bin = ENV.fetch('GOOGLE_CHROME_SHIM', nil)

chrome_opts = chrome_bin ? { "chromeOptions" => { "binary" => chrome_bin } } : {}

Capybara.register_driver :chrome do |app|
  Capybara::Selenium::Driver.new(
     app,
     browser: :chrome,
     desired_capabilities: Selenium::WebDriver::Remote::Capabilities.chrome(chrome_opts)
  )
end

Capybara.javascript_driver = :chrome
```

## Releasing a new version

Make sure you publish this buildpack in the buildpack registry

`heroku buildpacks:publish heroku/google-chrome master`
