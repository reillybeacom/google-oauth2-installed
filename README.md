# Google Oauth2 Installed

Configure and authenticate to Google with OAuth2 as an installed application.

This is for when your application needs to authenticate to Google services, as
opposed to your application's users.

Extracted from applications that use the DFP and Analytics APIs,
`google-oauth2-installed` helps with configuration (from `ENV`).
It also helps with setup by providing an easy command to generate your OAuth tokens.

For more information about Installed Apps: [https://developers.google.com/accounts/docs/OAuth2InstalledApp](https://developers.google.com/accounts/docs/OAuth2InstalledApp)


## Why

Google provides three ways to authenticate with OAuth2:

1. **Web application**. This is what you use when your application needs access
   as the user of your application.
2. **Service account**. This is really what you want, but can only be used by
   applications authenticating with a google apps account. If you have a google
   apps account, do not use GoogleOAuth2Installed, use service accounts instead.
3. **Installed application**. This allows you to authenticate as an application,
   but still requires an `access_token`, which requires user interaction through
   a browser. `google-oauth2-installed` tries to make this process as simple as
   possible. The bad news is that `access_token`s expire. The good news is that
   the `access_token` comes with a `refresh_token` that does not expire. So,
   once you've aquired your tokens, you can store them for later use and forget
   about it.


## Installation

Add this line to your application's Gemfile:

    gem 'google-oauth2-installed'

Or install it yourself as:

    $ gem install google-oauth2-installed

## Setup

### Create an application in Google Cloud Console

First, you need to create an application identifier in Google Cloud Console. Please follow
[these instructions](https://github.com/googleads/google-api-ads-ruby/wiki/OAuth2#creating-an-application-identifier)
lovingly copied (and only slightly altered) from the
[google-api-ads-ruby library](https://github.com/googleads/google-api-ads-ruby).

> Visit [Google Cloud Console](https://cloud.google.com/console) and:
>
> 1. Click **CREATE PROJECT** to create a new project.
> 1. Enter the Project Name (and optionally, choose your own Project ID), and click **Create**.
> 1. The newly created project should automatically open. Click **APIs & auth** to expand the menu, and then click **Credentials**.
> 1. Click **CREATE NEW CLIENT ID** to create a new client identifier and client secret.
> 1. Choose **Installed application**, and **Other* for the "Installed application type".
> 1. Click **CREATE CLIENT ID** to complete the registration. Client ID and client secret will be created and displayed.
>
> ![API Access Page](https://developers.google.com/adwords/api/images/oauth2-client-id-secret.png)
>
> The Client ID and secret values are the parameters you will need in the next step.

### Set up your credentials in order to retrieve an access token

Define the following environment variables. I recommend using the
[`dotenv`](https://github.com/bkeepers/dotenv) gem (`dotenv-rails` in rails projects)
and adding these variables to a `.env` file.

```
OAUTH2_CLIENT_ID="..."
OAUTH2_CLIENT_SECRET="..."
OAUTH2_SCOPE="..."
```

`OAUTH2_SCOPE` is a space delimited list of the scopes your application will
need access to. For example, for readonly access to analytics, and write access to
DFP, use:

```
OAUTH2_SCOPE="https://www.googleapis.com/auth/analytics.readonly https://www.google.com/apis/ads/publisher"
```

### Get your access token

Once you have these environment variables defined, run this rake task and
authenticate to google with the user you need access as.

```
rake googleoauth2installed:get_access_token
```

This rake task will give you a url to load up in the browser. You will need to log in
to Google, allow access to the requested scopes, and copy the provided code. Paste
this code back in to the rake task that is waiting for you. It will then output the
rest of the environment variables you need to authenticate.

(If the URL shows an error, you may need to supply a Product Name under the Consent Screen section of your Google Developers Console.)

If you are using `.env`, it should now look something like:

```shell
OAUTH2_CLIENT_ID="..."
OAUTH2_CLIENT_SECRET="..."
OAUTH2_SCOPE="..."

OAUTH2_ACCESS_TOKEN="..."
OAUTH2_REFRESH_TOKEN="..."
OAUTH2_EXPIRES_AT="..."
```

### Not in Rails?

You might need to reference our rake task from your `Rakefile`. Try something like this:

```ruby
require 'rubygems'
require 'bundler/setup'
load 'tasks/get_access_token.rake'
```

## Usage

Once you have all of your environment variables set up, just ask `GoogleOauth2Installed`
for an access token. `GoogleOauth2Installed` will handle refreshing it if needed.

Example usage with `Legato` for Analytics:

```ruby
Legato::User.new GoogleOauth2Installed.access_token
```

If you just need the details (and not a refreshed access token), use
`GoogleOauth2Installed.credentials`.

Example usage with `DfpApi`:

```ruby
dfp_authentication = GoogleOauth2Installed.credentials.merge(
  application_name: ENV['DFP_APPLICATION_NAME'],
  network_code: ENV['DFP_NETWORK_CODE'],
)

::DfpApi::Api.new({
  authentication: dfp_authentication,
  service: { environment: 'PRODUCTION' },
})
```

## Contributing

1. Fork it ([http://github.com/carnesmedia/google-oauth2-installed/fork](http://github.com/carnesmedia/google-oauth2-installed/fork))
2. Create your feature branch (`git checkout -b my-new-feature`)
3. Commit your changes (`git commit -am 'Add some feature'`)
4. Push to the branch (`git push origin my-new-feature`)
5. Create new Pull Request
