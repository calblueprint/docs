Rails
====
Articles that need to be written:

- How to use bluebase (https://github.com/calblueprint/bluebase)
- How to configure Postgres without crying
- How to use Gon to push data to javascript
- How to set up staging and production environments for Heroku
- How to use I18n for page text
- How to use phrasing for some CMS capability
- Cool development gems
- How to set up poltergeist for JS Capybara tests
- How to set up emailing on Heroku using Mandrill
- Cheatsheet for RSpec / Capybara

##Table of Contents
1. [Environment Variables with Figaro](#figaro)
2. [Staging and Production Environment for Heroku](#heroku)
3. [Using Guard with Rspec, Rubocop, and Livereload](#guard)

<a name="figaro"></a>
Figaro
----
See [this blog post](http://tech.calblueprint.org/using-figaro/) for background.

1. In Gemfile:

		gem "figaro"

2. Run

		$ bundle install

3. Run

		$ figaro install

4. Add environment variables to the generated `config/application.yml`
5. Make a `config/application.yml.sample` file with dummy variables (this is checked into git).
	- If using a CI system like Travis, make sure this file is copied over into an actual `config/application.yml` before running tests.
6. Add required keys to config/initializers/figaro.rb (keys that the app needs to function properly in all environments)
7. Write environment variables to Heroku with

		figaro heroku:set --remote=<your git remote> -e <your environment>

<a name="heroku"></a>
Staging and Production Environments for Heroku
----
This post is aimed to help you better understand the development workflow and provide some insight on best practices for configuring staging and production environments. Before beginning, it’s important to understand the differences between each respective environment: development, staging and production.

The **development environment** is the working code copy that is updated rapidly. Changes made by developers are deployed here so integration and features can be tested.

The **staging environment** is the release candidate, which normally mirrors the production environment. This environment contains the “next” version of the application to be released and is used for final stress testing and client/manager approvals before going live.

The **production environment** is the currently released version of the application, accessible to the client/end users. This environment changes during scheduled releases of the application.

An example workflow might look like this:

- Developers work on a new feature in separate branches. Minor updates can be pushed directly into master.
- Once the features are implemented, they are merged into master and pushed to a staging Heroku application.
- The staging environment is mainly to hammer out some final bugs that may show up on Heroku but not in the development environment. So after ensuring the staging application is bug-free, the code can be pushed to production.

To configure a proper staging and production environment for Rails applications, be sure to have a `development.rb`, `staging.rb`, and `production.rb` file in `/config/environments`.

An example `development.rb` file might look something like this:

```ruby
Rails.application.configure do
	# Settings specified here will take precedence over those in config/application.rb.
	# In the development environment your application's code is reloaded on
	# every request. This slows down response time but is perfect for development
	# since you don't have to restart the web server when you make code changes.
	config.cache_classes = false

	# Do not eager load code on boot.
	config.eager_load = false

	 # Show full error reports and disable caching.
	config.consider_all_requests_local = true
	config.action_controller.perform_caching = false
	# Don't care if the mailer can't send.
	config.action_mailer.raise_delivery_errors = true
	# Don't send emails in development
	config.action_mailer.perform_deliveries = false
	# Print deprecation notices to the Rails logger.
	config.active_support.deprecation = :log
	# Raise an error on page load if there are pending migrations.
	config.active_record.migration_error = :page_load

	# Debug mode disables concatenation and preprocessing of assets.
	# This option may cause significant delays in view rendering with a large
	# number of complex assets.
	config.assets.debug = true

	# Adds additional error checking when serving assets at runtime.
	# Checks for improperly declared sprockets dependencies.
	# Raises helpful error messages.
	config.assets.raise_runtime_errors = true

	# Raises error for missing translations
	config.action_view.raise_on_missing_translations = true

	config.action_mailer.default_url_options = { host: 'localhost:3000' }
end
```

An example `staging.rb` file might look something like this:

```ruby
require_relative "production"

Mail.register_interceptor(
 RecipientInterceptor.new(ENV.fetch("EMAIL_RECIPIENTS"), subject_prefix:
"[STAGING]"))
Rails.application.configure do
 config.action_mailer.default_url_options = { host: "staging.example-app.com" }
end
```

An example production.rb file might look something like this:

```ruby
Rails.application.configure do
# Settings specified here will take precedence over those in config/application.rb.

	# SMTP SETTINGS
	SMTP_SETTINGS = {
		address: ENV.fetch("SMTP_ADDRESS"), # example: "smtp.sendgrid.net"
		authentication: :plain,
		domain: ENV.fetch("SMTP_DOMAIN"), # example: "this-app.com"
		enable_starttls_auto: true,
		password: ENV.fetch("SMTP_PASSWORD"),
		port: "587",
		user_name: ENV.fetch("SMTP_USERNAME")
	}

	# Code is not reloaded between requests.
	config.cache_classes = true

	# Eager load code on boot. This eager loads most of Rails and
	# your application in memory, allowing both threaded web servers
	# and those relying on copy on write to perform better.
	# Rake tasks automatically ignore this option for performance.
	config.eager_load = true

	# Full error reports are disabled and caching is turned on.
	config.consider_all_requests_local       = false
	config.action_controller.perform_caching = true

	# Disable Rails’s static asset server (Apache or nginx will already do this).
	config.serve_static_assets = false

	 # Enable deflate / gzip compression of controller-generated responses
	config.middleware.use Rack::Deflater

	 # Compress JavaScripts and CSS.
	config.assets.js_compressor = :uglifier

	# Do not fallback to assets pipeline if a precompiled asset is missed.
	config.assets.compile = false

	# Generate digests for assets URLs.
	config.assets.digest = true

	# `config.assets.precompile` and `config.assets.version` have moved to
	# config/initializers/assets.rb

	# Specifies the header that your server uses for sending files.
	# config.action_dispatch.x_sendfile_header = "X-Sendfile" # for apache
	# config.action_dispatch.x_sendfile_header = 'X-Accel-Redirect' # for nginx

	# Force all access to the app over SSL, use Strict-Transport-Security, and
	# use secure cookies.
	# config.force_ssl = true

	# Set to :debug to see everything in the log.
	config.log_level = :info

	# Ignore bad email addresses and do not raise email delivery errors.
	# Set this to true and configure the email server for immediate delivery
	# to raise delivery errors.
	config.action_mailer.delivery_method = :smtp
	config.action_mailer.smtp_settings = SMTP_SETTINGS

	# Enable locale fallbacks for I18n (makes lookups for any locale fall-back to
	# the I18n.default_locale when a translation cannot be found).
	config.i18n.fallbacks = true

	# Send deprecation notices to registered listeners.
	config.active_support.deprecation = :notify

	# Use default logging formatter so that PID and timestamp are not suppressed.
	config.log_formatter = ::Logger::Formatter.new

	# Do not dump schema after migrations.
	config.active_record.dump_schema_after_migration = false

	# Set host to production heroku app
	config.action_mailer.default_url_options = { host: "ros-production.herokuapp.com" }
end
```

These files are all from Blueprint’s Roots of Success Rails application (included in the list of helpful links below!)

After, you’ll need to set up your remotes for staging and production. Here are some helpful how-tos to get you started!

**This is assuming you already have an application on your local machine and you are ready to push to Heroku**

To create the remote staging environment, run:

	$ heroku create --remote staging

To push code to staging, run:

	$ git push staging master
	…
	$ heroku ps --remote staging

Once your staging app is up and running properly, you can create your production app by running:

	$ heroku create --remote production
	…
	$ git push production master
	…
	$ heroku ps --remote production

And with that, you’ve got the same codebase running as two separate Heroku apps - one staging and one production!

Happy coding! :-)

Helpful links:

- https://github.com/calblueprint/roots-of-success/tree/master/config/environments
- http://guides.beanstalkapp.com/deployments/best-practices.html
- https://devcenter.heroku.com/articles/multiple-environments

<a name="guard"></a>
Using Guard with Rspec, Rubocop, and Livereload
----
### Guard
Guard is a command line tool that detects file system changes. We use this with other plugins to make development faster and and with less hassle.

Add Guard to your Gemfile in your project's root:

    gem 'guard'

Then install it by running Bundler:

    $ bundle

Generate an empty Guardfile with:

    $ bundle exec guard init

### RSpec
We use the RSpec plugin for Guard so that whenever files are changed and saved, RSpec tests will run against relevant changed code.

Add the following to your Gemfile inside the development group:

    gem 'guard-rspec', require: false

Add guard definition to your Guardfile by running this command:

    $ guard init rspec

### RuboCop
Like the RSpec plugin, the RuboCop plugin for Guard runs the RuboCop style checker against code that was recently changed.

Setting up the RuboCop plugin is nearly identical to setting up the RSpec plugin.

Add the following to your Gemfile inside the development group:

    gem 'guard-rubocop'

Add guard definition to your Guardfile by running this command:

    $ guard init rubocop

### LiveReload
The LiveReload plugin reloads your browser when you change 'view' files so your browser will stay update when developing.

Add the following to your Gemfile inside the development group:

    gem 'guard-livereload', '~> 2.4', require: false

Add guard definition to your Guardfile by running this command:

    $ guard init livereload

### Usage
Keeping Guard running while developing allows RSpec, RuboCop, and LiveReload to be used most effectively. After following the setup instructions above, you can run Guard by running the following in your project's root directory in a new terminal window and keeping it open.

    $ bundle exec guard

Guard will automatically run whenever you save files. With the installed plugins, Guard will run intelligently by on checking the style and running specs on the changed files. Pressing 'return' will run Guard and all plugins on all files.

For reference, [this](https://github.com/calblueprint/roots-of-success/blob/master/Guardfile) is the Guardfile for Roots of Success.
