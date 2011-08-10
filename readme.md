# Assembles and manages pieces of Your Application

Micon is infrastructural component, invisible to user and it's main goal is to simplify development. It reduces complex monolithic application to set of simple low coupled components. 

Concentrate on business logic and interfaces and Micon will provide automatic configuration, life cycle management and dependency resolving.

Technically it's [IoC][ioc] like framework with components, callbacks, scopes and bijections, inspired by Spring and JBoss Seam.

Is it usefull, is there any real-life application? - I'm using it as a heart of my [web framework][rad_core], this site http://ruby-lang.info for example powered with it.

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

	# callbacks, we need to parse routes right after environment is initialized
	app.after :environment do
		app[:router].parse '/config/routes.rb'
	end

	# dynamic components, will be created and destroyed for every request
	class Request
		register_as :request, scope: :request
	end

	class Application
		# injecting components into attributes
		inject request: :request, logger: :logger

		def do_business
			# now we can use injected component
			do_something_with request
			logger.info 'done'
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

## Note

Current wersion isn't thread-safe, instead it supported evented IO (EventMachine).
Actually I implemented first wersion as thread-safe, but because there's no actual multithreading in
Ruby, the only thing it does - adds complexity and performance losses, so I removed it.
But if you need it it can be done very easy.
	
## Installation

	$ sudo gem install micon
	
## License

Copyright (c) Alexey Petrushin http://petrush.in, released under the MIT license.

[ioc]: http://en.wikipedia.org/wiki/Inversion_of_control
[rad_core]: https://github.com/alexeypetrushin/rad_core