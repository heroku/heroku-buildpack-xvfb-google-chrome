# heroku-buildpack-xvfb-google-chrome

This buildpack downloads and installs Xvfb and Google Chrome from your choice
of release channels.

Note that Google Chrome now has a `--headless` flag, so for many applications,
Xvfb may not be required. If so, consider [this buildpack](https://github.com/heroku/heroku-buildpack-google-chrome) instead.

## Only supported on Cedar-14

Note that this buildpack only works on the Cedar-14 stack, for which the end-of-life
window [began May 1, 2019](https://devcenter.heroku.com/changelog-items/1603).

For newer stacks such as Heroku-16 and Heroku-18 you must instead use the more modern
[Google Chrome Buildpack](https://github.com/heroku/heroku-buildpack-google-chrome),
which runs Chrome in headless mode without Xvfb. For more details see
[Browser and User Acceptance Testing](https://devcenter.heroku.com/articles/heroku-ci-browser-and-user-acceptance-testing-uat#google-chrome-via-headless).

## Channels

You can choose your Chrome release channel by specifying `GOOGLE_CHROME_CHANNEL` as
a config var for your app, in your app.json (for Heroku CI and Review Apps),
or in your pipeline settings (for Heroku CI).

Valid values are `stable`, `beta`, and `unstable`. If unspecified, the `stable`
channel will be used.

## Shims and Command Line Flags

This buildpack installs shims that always add `--disable-gpu`
and `--no-sandbox` to any `google-chrome` command as you'll have trouble
running Chrome on a Heroku dyno otherwise.

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
```

## Releasing a new version

Make sure you publish this buildpack in the buildpack registry

`heroku buildpacks:publish heroku/xvfb-google-chrome master`
