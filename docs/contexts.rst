.. _contexts:

.. currentmodule:: flask_sqlalchemy

Introduction into Contexts
==========================

If you are planning on using only one application you can largely skip
this chapter.  Just pass your application to the :class:`SQLAlchemy`
constructor and you're usually set.  However if you want to use more than
one application or create the application dynamically in a function you
want to read on.

If you define your application in a function, but the :class:`SQLAlchemy`
object globally, how does the latter learn about the former?  The answer
is the :meth:`~SQLAlchemy.init_app` function::

    from flask import Flask
    from flask_sqlalchemy import SQLAlchemy

    db = SQLAlchemy()

    def create_app():
        app = Flask(__name__)
        db.init_app(app)
        return app


What it does is prepare the application to work with
:class:`SQLAlchemy`.  However that does not now bind the
:class:`SQLAlchemy` object to your application.  Why doesn't it do that?
Because there might be more than one application created.

So how does :class:`SQLAlchemy` come to know about your application?
You will have to setup an application context.  If you are working inside
a Flask view function or a CLI command, that automatically happens. However,
if you are working inside the interactive shell, you will have to do that
yourself (see `Creating an Application Context
<http://flask.pocoo.org/docs/1.0/appcontext/#manually-push-a-context>`_).

If you try to perform database operations outside an application context, you
will see the following error:

    No application found. Either work inside a view function or push an
    application context.

In a nutshell, do something like this:

>>> from yourapp import create_app
>>> app = create_app()
>>> app.app_context().push()

Alternatively, use the with-statement to take care of setup and teardown::

    def my_function():
        with app.app_context():
            user = db.User(...)
            db.session.add(user)
            db.session.commit()

Some functions inside Flask-SQLAlchemy also accept optionally the
application to operate on:

>>> from yourapp import db, create_app
>>> db.create_all(app=create_app())
