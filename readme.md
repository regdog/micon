# Micro Container - assembles and manages pieces of your application

Micon is infrastructural component, invisible to user and it's main goal is to simplify development. It reduces complex monolithic application to set of simple low coupled components. 

Concentrate on business logic and interfaces and Micon will provide automatic configuration, life cycle management and dependency resolving.

Technically it's like [IoC][ioc] container managing components, callbacks, scopes and bijections, inspired by Spring and JBoss Seam.

## Usage
	
Let's suppose you are building the Ruby on Rails clone, there are lots of modules let's try to deal with them

	require 'micon'
	def app; Micon end

	# static (singleton) components
	class Environment
		register_as :environment
	end

	class Logger
		register_as :logger
		
		def info msg; end
	end	

	class Router
		register_as :router
		
		def parse rote_filename
		  # do something
		end
	end

	# callbacks, we need to parse routes right after environment will be initialized
	app.after :environment do
		app[:router].parse '/config/routes.rb'
	end

	# dynamic	components, will be created and destroyed for every request
	class Request
		register_as :request, :scope => :request
	end

	class Application
		# injecting components into attributes
		inject :request => :request, :logger => :logger

		def do_business
			# now we can use injected component
			logger.info "routes parsed"
			do_something_with request
			logger.info 'well done'
		end
		
		def do_something_with request; end
	end

	# Web Server / Rack Adapter
	class RackAdapter
		def call env		
			# activating new request scope, the session component will be created and destroyed automatically
			app.activate :request, {} do
				Application.new.do_business
			end
		end
	end    
	
	RackAdapter.new.call({})
	
For actual code go to spec/overview_spec.rb
	
## Installation

	$ sudo gem install micon

Copyright (c) 2009 Alexey Petrushin [http://bos-tec.com](http://bos-tec.com), released under the MIT license.

[ioc]: http://en.wikipedia.org/wiki/Inversion_of_control