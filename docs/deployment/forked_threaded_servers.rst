Forked and Threaded Servers
+++++++++++++++++++++++++++

Forked and Threaded Servers share common "gotchas" and solutions when
using pyramid and some popular packages.

Forked and Threaded servers tend to use a "copy on write" implementation detail
to optimize how they work and share memory. This can create problems when
certain actions happen before the fork or thread dispatch, such as when files or
file-descriptors are opened or random number generators are initialized.

Many servers have


Servers
++++++++++

gunicorn
========

gunicorn offers several hooks during an application lifecycle

the postfork routine is provided as a function in a configuration python script.

for example a script 'config.py' might look like:

	def post_fork(server, worker):
		log.debug("gunicorn - post_fork")

and invoking the script would look like:

	gunicorn --paste production.ini -c config.py

documentation for the `post_fork` hook appears on http://docs.gunicorn.org/en/latest/settings.html#post-fork


uwsgi
========

uwsgi offers a decorator to handle forking

your application would include code like

    from uwsgidecorators import postfork
    
    @postfork
    def my_setuo():
		log.debug("uwsgi - postfork")

documentation for the `postfork` decorator appears on https://uwsgi-docs.readthedocs.io/en/latest/PythonDecorators.html#uwsgidecorators.postfork


waitress
========

waitress is not a forking server, but it's threads can create similar issues as forking servers


Known Packages
++++++++++++++


SqlAlchemy
============

Many people use SqlAlchemy as part of their pyramid application stack.

The database connections and the connection pools in SqlAlchemy are not safe to
share across process boundaries (forks and threads). The connections and 
connection pools are lazily created on their first use, so most pyramid users 
will not encounter an issue as database interaction usually happens on a 
per-request basis.

If your pyramid application connects to a database during the application startup
however, then you must use 'Engine.dispose' to reset the connections.  It might
look something like this:

    @postfork
    def reset_sqlalchemy():
        models.engine.dispose()

Additional documentation on this topic is available from SqlAlchemy's docs:

* http://docs.sqlalchemy.org/en/latest/core/pooling.html#using-connection-pools-with-multiprocessing
* http://docs.sqlalchemy.org/en/latest/core/connections.html#engine-disposal


