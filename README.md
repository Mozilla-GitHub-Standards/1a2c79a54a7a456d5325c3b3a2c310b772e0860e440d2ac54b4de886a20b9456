templeton, a minimal web framework

Templeton is a script, module, and support files intended for rapid development
of simple web applications.  It's basically a package around web.py which
suggests a common layout and organization for web apps.

The templeton script
====================

Usage:

    templeton install <www-data-dir>

Copies support files (JS, CSS) into a "templeton" directory in <www-data-dir>.
The latter should be the root of the web site that will serve templeton
apps, since the template HTML file loads JS and CSS from /templeton.

    templeton init <appname>

Creates a directory named <appname> with "html" and "server" directories
containing templates.  You should be able to serve up your default app
by doing

    cd <appname>/server
    python server.py

Go to http://localhost:8080/ to see the result.  The next steps you'll want
to do is edit <appname>/server/handlers.py and put in your server-side
business logic and edit and create the files in <appname>/html to build up
your client-side logic.


The templeton module
====================

The templeton module has two main functions:

* set up middleware to separate static pages from dynamic REST calls.
* provide helpers for common tasks, such as handling specific request types.

middleware
----------

Include templeton.middleware patches the standard web.py development server
to reflect the standard templeton path structure and to better mirror the
deployed layout.

Paths starting with '/api' are dispatched to a handler.

Standard third-party files (JS & CSS, e.g. JQuery) are served from
'/templeton'.  Running the 'init' command of the templeton script (see above)
installs these files for deployment at the same path.

All other paths are treated as static files.  Static files are now stored in
'../html' rather than 'static'. For example, accessing
http://localhost:8080/index.html will load ../html/index.html, and 
http://localhost:8080/scripts/app.js will load ../html/scripts/app.js.


handlers
--------

templeton is geared toward client-rich, REST-based web applications.  These
typically involve a large amount of JSON.  templeton provides decorators to
simplify handler code.

@json_response is a decorator function that expects the decorated function to
return a JSON-serializable object, which it uses to construct a proper
web.py response.

The handlers module also provides helper functions.

load_urls() takes a web.py URL-handler sequence, i.e. (<path>, <class name>,
<path>, <class name>, ...), and prepends the REST API path, '/api', to each
given path.  The default server.py (created by the 'init' script command) uses
this function to load urls from handlers.py.

get_request_parms() parses the current request's search string and body as JSON
and returns the results as (args, body).

A trivial example of a JSON handler that echoes back any search-string args:

    import templeton.handlers
    
    class JsonTest(object):
    
        @templeton.handlers.json_response
        def GET(self):
            args, body = templeton.handlers.get_request_parms()
            return args
