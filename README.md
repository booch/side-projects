My Side Projects
================

This is a place to keep my ideas for side projects that don't (yet) have their own repo.


Servers
-------

### Migrate Everything to DIgital Ocean

* Install PHP
    * `sudo apt-get install php7.0-common php7.0-common php7.0-fpm`
* Install DokuWiki
* Move DokuWiki sites to Digital Ocean
* Inatall WordPress
* Move WordPress sites to Digital Ocean
* Shut down old server
* Move DokuWiki sites to Hugo
* Remove DokuWiki
* Move WordPress sites to Hugo
* Remove WordPress
* Remove PHP

### BoochTek and Personal Sites

* Move SSH port
    * Studies show 1000x decrease in login attempts


### Nginx Configs

* Re-write my Lua code to do short-URL redirects more easily
    * Use a flat-file to store all the short URLs and the long URLs they expand to
* Add rate limiting
    ~~~ nginx
    # We're in the `http` section here.
    # Limit to 10 reqs/s on average (over 60 seconds), with bursts up to 100 requests/second, and 5 MB cache.
    limit_req_zone $binary_remote_addr zone=rate_limit:5m rate=10r/s;
    limit_req zone=rate_limit burst=10; # This can also go in server or location sections.
    ~~~
* Add a snippet for reverse proxying
    ~~~ nginx
    # We're in the `http` section here.
    upstream my_app {
        server http://localhost:3000;
        #server unix:///var/run/my_app.sock;
    }
    server {
        ...
        set $app http://my_app; # Can also use http://localhost:3000 if you don't want an `upstream` section
        location / {
            proxy_pass $app;
            proxy_redirect off;
            proxy_set_header Host $http_host;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        }
    }
    ~~~


Open Source Contributions
-------------------------

### Ruby

* Allow Ranges to have a `nil` lower bound
    * We already allow a `nil` upper bound
    * Would be really useful for translating `<` to SQL in ActiveRecord
    * Have to worry about completely infinite ranges (nil..nil)
        * Have to worry about trying to iterate through one of those
	    * Just throw a `RangeError`, like `entries` already does for semi-infinite ranges
* Fix Array and String (and maybe Hash) before to allow them to be subclassed
    * Biggest fix would be using `self.class.new` whenever the internals need to create a new Array or String
    * Will take a lot of work, and will likely have to be done in C
* [Changes to `dig`](https://github.com/booch/dig)
    * Make it more like `fetch`
        * Optional block parameter for `dig` to specify a default value
    * Allow it to work for things other than `Array` and `Hash`
    * Allow manipulation of the object that's found (`dig!`)
        ~~~ ruby
            class Hash
              def dig!(*args)
                fail ArgumentError.new("Must provide a block to `dig!`") unless block_given?
                yield args.inject(self){ |a,x| a[x] }
                self
              end
            end
        ~~~
        * Kind of like a lens, but not really
            * We don't really need lenses in Ruby, due to OOP
    * Created a gem to test the changes
        * Maybe that'll also be the long-term solution, instead of getting the change into Ruby itself
* Add feature to automatically set instance variables
    ~~~ ruby
    class Point
      attr_reader :x, :y
	  def initialize(@x, @y); end
    end
    ~~~
	* Like CoffeeScript and Crystal have
	* I can't see any issues with adding this feature
	* Great for simplifying initializers
* Could we add `each` to `Object` and `NilClass` and make them `Enumerable`?
    * For `NilClass`, it'd just be a no-op, same as an empty Array
    * The default for `Object` would to be to just call the block with `self`
* Change `#methods` to take a `superclass_methods` named parameter, where it now takes a Boolean
    * Allow for backwards compatibility
    * Also allow a mode where it subtracts out all methods from Object.new
    * Can make a gem to test it
        * Maybe that'll also be the long-term solution, instead of getting the change into Ruby itself
    * Make the same changes to `Class#instance_methods`
    * See http://booch.github.io/presentations/Booleans_Are_Easy/slides.html#p21
* Could we turn the JIT into an AOT compiler for Ruby?


### Rails

* Add docs about `AR::Relation#find` supporting blocks
    * Just passes it on to `super` (`Enumerable`)
    * Has been this way for *years* (2009, when `AR::Relation#find` was added)
        * https://github.com/rails/rails/commit/d92c4a84023bc0c8dd75869c9b4d5e50277f4650
    * https://github.com/rails/rails/blob/master/activerecord/lib/active_record/relation/finder_methods.rb#L68
* Refactor long `case` statements to hashes
    * See `railties/lib/rails/generators/generated_attribute.rb` for some prime examples
* Add easy `LIKE` and `ILIKE` support to ActiveRecord
    ~~~ ruby
    def where_like(key, value) # Should really take a pair, not separate key and value. TODO: What about multiple keys/values?
      where(arel_table[key.to_sym].matches("%#{sanitize_sql_like(value)}%")) # TODO: Allow regexes. Check need for leading/trailing `%`.
    end
    ~~~
    * Maybe also something to do `<`, `<=`, `>`, `>=` more easily than using ranges


### RuboCop

* Add an option to `Metrics` cops to ignore (or better yet, treat as `n` lines) hashes when counting lines.
And something similar for `Metrics/AbcSize`, which doesn't stricty count lines.
* Add a RuboCop-RSpec rule to complain about `expect(subject).to raise_error`
    * Should be `expect{ subject }.to raise_error`
    * Would be nice for RSpec to catch it as well


### Ansible

* `osx_defaults` Module
    * Allow reading values
    * Maybe replace with a completely-rewritten `macos_defaults`
    * Handle nested hashes/arrays
        * Adding to a hash
        * Adding a hash
        * Adding to an item nested within hashes
        * Need these for:
        	* Disabling Ctrl+Up/Down to allow their use in Atom


### Atom

* YAML
    * Uppercase TRUE/FALSE should be keywords
        * YAML 1.2 only supports lowercase
        * YAML 1.0 and 1.1 don't actually define any schema
        * Ansible documents allowing `yes`, `no`, `true`, and `false` in lower, upper, or title case
            * docs.ansible.com/ansible/YAMLSyntax.html
* Ruby refactoring
* Multi-preview
    * Look at file type, decide how to do a "Preview" of the file
    * Use it for "Show Preview" in my Touch Bar config


### Remark

* Add a mode that doesn't do the Markdown-to-HTML conversion
    * So I can provide my own slides already converted to HTML
* Presenter mode enhancements
    * Add current time
    * Add progress bar for number of slides down
    * Add progress bar for time elapsed (it they specify time allotted)
    * Add a cool tele-prompter mode
    * My re-arrangement of the sections
* Slide transitions


HTML Slide Show Builder
-----------------------

* Name:
    * Slide-Splitter
    * Slider
    * Slide-Shower
* Update Remark to add a mode that doesn't do the Markdown-to-HTML conversion
    * If that's not accepted upstream, crete my own fork
        * Call it Remarkable Player
        * Rip out all the JavaScript Markdown stuff
* Split on `---` lines, send each to Markdown parser
	* Can configure to split on something else
* Each slide is a `section class="slide"` by default
* All slides go inside an `article class="slideshow slides"`
	* That can be in a template
		* Template specifies where slides go: `<%= yield :slide %>`
			* Or `{{slide}}`
* Allow user to specify different Markdown add-ons
	* To interpret things that various other Markdown slideshows support
		* Start with just preset for the stuff I'm using from Remark
			* `???` for presenter notes
		*  `^` for presenter notes
    * Probably use Python Markdown parser, instead of Ruby's Kramdown
        * Supports extensions, with lots available
	* Way to specify background image
		* Fitting the image (full-height, full-width, stretch-to-fit)
		* Positioning the image
		* Effects on the image (fading, black-and-white, blurring, etc.)
	* Way to specify background color
		* Various gradient options
	* Way to specify slide configuration (classes)
	* SVG parser (from fenced code block with `svg eval`)
	* Mermaid parser (from fenced code block with `mermaid eval`)
		* And PlantUML
		* And ditaa
	* Tables
	* Footnotes / endnotes
	* Definition lists
	* Emoji substitution (Slack style)
	* Attribute lists (like Maruku and Python-Markdown extension)
	* Asides / admonitions (warning, info, etc)
		* Presenter notes should really be done using this
        * I feel like using quote syntax is the way to go
            * We'd need to specify a "type" though
	* Code "blocks" from external files
		* Allow specifying filename and optionally, line numbers
	* Code block features
		* Line numbering
		* Highlighting specific lines
		* Highlighting specific pieces of code
			* Different highlight colors, like I used for HTTP requests/responses
	* Overlaying arrows and such on top of images, code, etc
* Handle everything that all the other Markdown slide generators support
	* Headers / footers
		* Current page / total pages
		* Arbitrary text / HTML
	* Background image, text in front, with translucency masking
		* Filters on background images
* Can also output slides to PDF
	* With or without presenter notes
		* Default to landscape when not printing notes
		* Default to portrait when printing notes
			* Slide in landscape (with a border by default), notes below that


Web Dev
-------

* CSS framework
	* Start with rails-template (rails5 branch) public/stylesheets
		* Have Rails template pull it in
			* Should be a Node module
	* Variables
	* Layout
		* Flexbox
		* Grid layout
	* Components
	* Forms
		* Will flexbox give us a nice way to style?
	* Tables with pagination
	* Include some minimal JS
		* Remove noJS from HTML tag
		* Add per-browser, render engine, and OS classes to HTML tag
		* Use vanilla JS


Rails Templates
---------------

* Break into smaller pieces that I can include for various purposes
* Update and use my StandardCrudActions
* Use Formtastic or SimpleForm
* Add my SimpleTable
    * Use Kaminari for pagination
        * Actually, there's a better option out there now
* Try using Trailblazer on top of Rails
* Updates for Rails 5.1
	* Rails command will now do a `git --init` for us
		* `--yarn`
		* `--webpack elm`
	* Remove `-j` or `--javascript` 
	* Use `form_with model:` to replace `form_for`
	* Consider DatabaseRewinder over DatabaseCleaner
	* Use my ActiveRecord::Repository
	* Use my includable ActiveRecord
	* Use my Virtus::ActiveRecord
	* RSpec - https://github.com/rspec/rspec-rails/issues/1808
	* No more `render :text` or `:nothing` , or `redirect_to :back`
* Updates for Rails 5.2
	* Check http://blog.michelada.io/whats-new-in-rails-51
	* No more before/after/around filters in controllers
	* `brew install yarn`
	* `rails webpacker:install`
	* After initializing the app: `bin/rails secrets:setup`
		* Ensure `config/secrets.yml.key` is ignored by Git
		* Add a note to share the secret key with others
			* Recommendations?
			* Set it in `RAILS_MASTER_KEY` environment variable
		* In `config/production.rb`:
			* `config.read_encrypted_secrets = true`
	* `config.read_encrypted_secrets = true`
	* Replace `form_for` with `form_with`
		* Use `model` or `url` keyword
		* Replace `fields_for` with `fields` with `model` keyword
	* Set up Capybara to use ChromeDriver and FirefoxDriver
		* Both with headless options
	* Change the way we do jQuery, using Yarn
		* Probably get rid of the stuff I have in code
		* Not sure whether to even include jQuery by default any more
	* Do I need to run these manually, or do they get run with `rails s`?
		* `webpack-watcher`
		* `webpack-dev-server`
	* Move JS to `app/javascript`, with a directory for each pack
		* Call with `javascript_pack_tag`
		* Precompile assets with:
			* `rails asset:precompile`
			* `rails webpacker:compile`
* Updates for Rails 6.0


Books (to Write)
-----

* Agile book (Effective Agile Practices)
	* Context dependent
	* Actual agility
		* Perceptions of Agile being very rigid
	* Continuous improvement
		* Toolbox of techniques that might help improve
		* Context dependent
	* Started with sets of practices
		* Manifesto and values/principles came later
		* Practices are how almost all of us got started
		* Practices provide value
			* Values and principles help us understand why they work
	* Pair programming chapter


iOS Keyboard
------------

* I basically want the best of GBoard, Swype, and SwiftKey combined
* Dextr has a smart/interesting layout:
    * Mostly alphabetical, with vowels in 1 column
        * So vowels are all right next to each other
    * Slightly curved away from the thumbs
    * Discontinued
* https://github.com/wyndwarrior/iSwipe


Ruby Measurements Library
-------------------------

* Remove `aliases` - just make another unit with a factor of 1
* Include `plural` as its own option key
* Include `inexact: true` option
    * Should probably make that `exact: false`
	* Allow querying results for `exact?`
	* Might also happen with fractions


Ruby Actors
-----------

* Based on Ruby Concurrency actors
    *  `define_message :message_name do |*args|`
    * Add a `method_defined` hook to ensure no methods are added
        * Better yet, use it to transform methods into messages
    * Call with `actor.method_name!` to fire and forget
    * Call with `actor.method_name(*args)` to wait
    * Add checking to ensure we don't do what we're not supposed to

