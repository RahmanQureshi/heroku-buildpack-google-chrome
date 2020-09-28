# heroku-buildpack-google-chrome-83

Forked and hacked from https://github.com/heroku/heroku-buildpack-google-chrome.

This buildpack downloads and installs (headless) Google Chrome stable version 83 from slimjet. Ignore the things below about being able to select channels.

## Channels

You can choose your release channel by specifying `GOOGLE_CHROME_CHANNEL` as
a config var for your app, in your app.json (for Heroku CI and Review Apps),
or in your pipeline settings (for Heroku CI).

Valid values are `stable`, `beta`, and `unstable`. If unspecified, the `stable`
channel will be used.

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
[chromedriver buildpack](https://github.com/heroku/heroku-buildpack-chromedriver)
also available.

Additionally, chromedriver expects Chrome to be installed at `/usr/bin/google-chrome`,
but that's a read-only filesystem in a Heroku slug. You'll need to tell Selenium/chromedriver
that the chrome binary is at `/app/.apt/usr/bin/google-chrome` instead.

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

## Running chrome inside the docker yourself

## Running chrome inside the docker yourself

`docker build --no-cache  --build-arg STACK=heroku-18 --build-arg GOOGLE_CHROME_CHANNEL=stable --build-arg BASE_IMAGE=heroku/heroku:18 .`

`docker run -it -p 9222:9222 <image id>`


`chmod +x .profile.d/*.sh`

`source .profile.d/000_apt.sh`

`source .profile.d/010_google-chrome.sh`

`$HOME/.apt/usr/bin/google-chrome-stable --headless --no-sandbox --disable-gpu --remote-debugging-address=0.0.0.0 --remote-debugging-port=9222 --no-sandbox --enable-logging --v=1`
