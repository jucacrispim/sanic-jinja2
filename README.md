# sanic-jinja2
Jinja2 support for sanic

![Example](https://github.com/lixxu/sanic-jinja2/blob/master/example/example.png)

## Installation

`pip install sanic-jinja2`


## Usage

```
NOTICE:
If you want to use flash and get_flashed_messages, you need setup session first

Currently, app and request are hooked into jinja templates, thus you can use them in template directly.

And, from version 0.3.0 enable_async is default to True.
If you need sync functions, use jinja.render_sync, jinja.render_string_sync

BUG: request should not be set to global environment, so you need use request['flash'] instead of jinja.flash and need pass request to render to use get_flashed_messages.
```


```python

    #!/usr/bin/env python
    # -*- coding: utf-8 -*-

    from sanic import Sanic
    from sanic_session import InMemorySessionInterface
    from sanic_jinja2 import SanicJinja2

    app = Sanic()

    jinja = SanicJinja2(app)
    # or setup later
    # jinja = SanicJinja2()
    # jinja.init_app(app)

    session = InMemorySessionInterface(cookie_name=app.name, prefix=app.name)


    @app.middleware('request')
    async def add_session_to_request(request):
        # before each request initialize a session
        # using the client's request
        await session.open(request)


    @app.middleware('response')
    async def save_session(request, response):
        # after each request save the session,
        # pass the response to set client cookies
        await session.save(request, response)


    @app.route('/')
    async def index(request):
        request['flash']('success message', 'success')
        request['flash']('info message', 'info')
        request['flash']('warning message', 'warning')
        request['flash']('error message', 'error')
        return await jinja.render('index.html', greetings='Hello, sanic!',
                                  request=request)


    if __name__ == '__main__':
        app.run(host='0.0.0.0', port=8000, debug=True)
```
