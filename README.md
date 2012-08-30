Rack::Insight
=========

Rack::Insight began life as an fork of Logical::Insight by LRDesign.  I started a
fork because LogicalInsight was namespaced as "Insight", which is causing namespace
collisions everywhere I have an Insight model.  I had to re-namespace all the code.
I also needed to build a few extension gems with additional panels, and added the
Config class to allow for custom panel load paths, and other future extensions
that don't work in the *use Middleware* declaration.  It should be *even* easier
to extend than LogicalInsight was, because extension gems can access the Config class
and truly bolt-on cleanly.

Having made really significant architectural changes, I'll be keeping Rack::Insight
a separate project for the forseeable future.

* Forked From: [logical-insight](http://github.com/LRDesign/logical-insight), [documentation](http://lrdesign.github.com/rack-insight/)
* Which Was Forked From: [rack-bug](http://github.com/brynary/rack-bug)

Description
-----------

Rack::Insight adds a diagnostics toolbar to Rack apps. When enabled, it injects a floating div
allowing exploration of logging, database queries, template rendering times, etc.   Rack::Insight
stores debugging info over many requests, incuding AJAX requests.

Features
--------

* Password-based security
* IP-based security
* Rack::Insight instrumentation/reporting is broken up into panels.
    * Panels in default configuration:
        * Rails Info
        * Timer
        * Request Variables
        * SQL
        * Active Record
        * Cache
        * Templates
        * Log
        * Memory
    * Other bundled panels:
        * Redis
        * Speedtracer
    * Retired panels - if needed they could come back quickly:
        * Sphinx
        * Mongo
    * The API for adding your own panels is simple and very powerful
        * Consistent interface to instrument application code
        * Consistent timing across panels
        * Easy to add sub-applications for more detailed reports (c.f. SQLPanel)
        * The documentation is scarce, so there's a feeling of adventure :/

Rails quick start
---------------------------

Add this to your Gemfile
    gem "rack-insight"

In config/environments/development.rb, add:

    config.middleware.use "Rack::Insight::App",
      :secret_key => "someverylongandveryhardtoguesspreferablyrandomstring"

Any environment with Rack::Insight loaded will have a link to "Rack::Insight" added to as
the last child of BODY to normal responses.  Clicking that link will load the
toolbar.  It's set with an id of "rack-insight-enabler", so it can be styled
to go somewhere more noticeable.  E.g. "position: absolute; top: 0; left: 0"

Using with non-Rails Rack apps
------------------------------

Just 'use Rack::Insight' as any other middleware.  See the SampleApp in the
spec/fixtures folder for an example Sinatra app.

If you wish to use the logger panel define the LOGGER constant that is a ruby
Logger or ActiveSupport::BufferedLogger

Configuring custom panels
-------------------------

Specify the set of panels you want, in the order you want them to appear:

    require "rack-insight"

    ActionController::Dispatcher.middleware.use "Rack::Insight::App",
      :secret_key => "someverylongandveryhardtoguesspreferablyrandomstring",
      :panel_files => %w[
        timer_panel
        request_variables_panel
        redis_panel
        templates_panel
        log_panel
        memory_panel
      ]

By default panel files are looked up by prepending "rack/insight/panels/" and requiring them.
Subclasses of Rack::Insight::Panel are loaded and added to the toolbar.  This makes
it easier to work with the configuration and extend Rack::Insight with plugin gems.

If you need to customize the load paths where Rack::Insight will look for panels you can configure the load paths in an
initializer.  Example config/initializers/rack_insight.rb:

    Rack::Insight::Config.configure do |config|

      # Note: The parent directory of the 'special' directory must already be in Ruby's load path.
      config[:panel_load_paths] = File.join('special','path')

      # Example 1: Do not load any of the regular Rack::Insight panels:
      config[:panel_load_paths] = File.join('my','custom','panel','directory')

      # Example 2: Add your custom path to the existing load paths, to have your panels join the party!
      config[:panel_load_paths] << 'custom/panels'

    end

Running Rack::Insight in staging or production
------------------------------------------

We have have found that Rack::Insight is fast enough to run in production for specific troubleshooting efforts.

### Configuration ####

Add the middleware configuration to an initializer or the appropriate
environment files, taking the rest of this section into consideration.

### Security ####

Restrict access to particular IP addresses:

    require "ipaddr"

    ActionController::Dispatcher.middleware.use "Rack::Insight::App"
      :secret_key => "someverylongandveryhardtoguesspreferablyrandomstring",
      :ip_masks   => [IPAddr.new("2.2.2.2/0")]

Restrict access using a password:

    ActionController::Dispatcher.middleware.use "Rack::Insight::App",
      :secret_key => "someverylongandveryhardtoguesspreferablyrandomstring",
      :password   => "yourpassword"

#### custom file path for the logging database ####

Logical Rack::Insight uses SQLite to store data from requests, and outputs a database
file in the root directory. If you need the file to be created at another
location (i.e. Heroku), you can pass a custom file path.

    ActionController::Dispatcher.middleware.use "Rack::Insight::App"
      :secret_key => "someverylongandveryhardtoguesspreferablyrandomstring",
      :database_path => "tmp/my_insight_db.sqlite"

Authors
-------

- Maintained by [Judson Lester](mailto:judson@lrdesign.com)
- Contributions from Luke Melia, Joey Aghion, Tim Connor, and more
- Based on Rack::Bug by Bryan Helmkamp

Thanks
------
Rack::Insight owes a lot to both LogicalInsight and Rack::Bug, as the basis projects.  There's a lot of smart
in there.  Many thanks to Evan, Judson, and Bryan for building them.

Inspiration for Rack::Bug is primarily from the Django debug toolbar.
Additional ideas from Rails footnotes, Rack's ShowException middleware, Oink,
and Rack::Cache

License
-------

See LICENSE.txt in this directory.

Notes
-----

Legacy files: would like to re-include them, but they need work

    lib/insight/views/panels/mongo.html.erb
    lib/insight/panels/mongo_panel/mongo_extension.rb
    lib/insight/panels/mongo_panel/stats.rb
    lib/insight/panels/mongo_panel.rb

    lib/insight/views/panels/sphinx.html.erb
    lib/insight/panels/sphinx_panel/stats.rb
    lib/insight/panels/sphinx_panel.rb

This one is mostly just a curiosity
    lib/insight/panels/speedtracer_panel/profiling.rb
