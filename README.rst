=======================================================
Renesas IoT Sandbox Data Monitoring API Client - Dweepy
=======================================================

This is a version of Dweepy, a simple Python client for, `renesas.dweet.io <https://renesas.dweet.io>`_. 
This platform is known as the 
Renesas IoT Sandbox Data Monitoring. Extensive information on the
Renesas IoT Sandbox is available at http://learn.iotcommunity.io/

Get the Code
------------

Clone the public repository fork::

    $ git clone https://github.com/iotcommunity/dweepy

Installation
------------

Once you have a copy of the source, you can embed it in your Python package, 
or install it into your site-packages easily::

    $ python setup.py install


S5D9 Preparation
----------------

You'll need to get your thing name. Connect the S5D9
IoT Fast Prototyping Kit board to your computer with a USB
cable. It will mount as a USB drive. Your thing name will be in
ThingName.txt

.. image:: https://codetricity.github.io/s5d9-hacking/monitoring/img/thingname.png

Your S5D9 board must be connected to USB and power.

.. image:: https://codetricity.github.io/s5d9-hacking/monitoring/img/usb-ethernet.jpg


Your dashboard should be receiving sensor data. The dashboard is available 
at http://renesas.dweet.io/landing.html

.. image:: https://codetricity.github.io/s5d9-hacking/monitoring/img/dash2.png

Overview of board layout with onboard sensors.

.. image:: https://codetricity.github.io/s5d9-hacking/monitoring/img/S5D9schematic.jpg

API Usage
---------

Dweepy aims to provide a simple, pythonic interface to cover 
the renesas.dweet.io API entirely.

First you'll need to import dweepy.::

    import dweepy

dweepy example for S5D9 IoT Fast Prototyping Kit
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

::

    import dweepy
    import pprint

    response = dweepy.get_latest_dweet_for('S5D9-63de')
    pprint.pprint(response)

The response.

::

    $ python renesasiot.py
    [{u'content': {u'globals': {u'application': u'DweetS5D9Client',
                                u'dweet-count': 1065},
                u'sensors': {u'AccelX': 0.02,
                                u'AccelY': 0.31,
                                u'AccelZ': 0.97,
                                u'Humidity': 38,
                                u'LED': 0,
                                u'MagX': -77,
                                u'MagY': 34,
                                u'MagZ': -179,
                                u'Pressure': 1016,
                                u'SoundLevel': 2,
                                u'TemperatureC': 29,
                                u'TemperatureF': 84}},
    u'created': u'2017-10-18T20:18:00.162Z',
    u'thing': u'S5D9-63de'}]

Dweeting
~~~~~~~~

You can send a dweet from a thing with a specified name.::

    >>> dweepy.dweet_for('this_is_a_thing', {'some_key': 'some_value'})
    {
        u'content': {u'some_key': u'some_value'},
        u'created': u'2014-03-19T10:38:46.010Z',
        u'thing': u'this_is_a_thing'
    }


Getting Dweets
~~~~~~~~~~~~~~

To read the latest dweet for a thing, you can call::

    >>> dweepy.get_latest_dweet_for('this_is_a_thing')
    [
        {
            u'content': {u'some_key': u'some_value'},
            u'created': u'2014-03-19T10:38:46.010Z',
            u'thing': u'this_is_a_thing'
        }
    ]


Note that dweet.io only holds on to the last 500 dweets over a 24 hour period. If the thing hasn't dweeted in the last 24 hours, its history will be removed.

Or to read all the dweets for a thing, you can call::

    >>> dweepy.get_dweets_for('this_is_a_thing')
    [
        {
            u'content': {u'some_key': u'some_value'},
            u'created': u'2014-03-19T10:42:31.316Z',
            u'thing': u'this_is_a_thing'
        },
        {
            u'content': {u'some_key': u'some_value'},
            u'created': u'2014-03-19T10:38:46.010Z',
            u'thing': u'this_is_a_thing'
        }
    ]


Alerts
~~~~~~

Set an alert::

    >>> dweepy.set_alert(
    ...     'this_is_a_thing',
    ...     ['test@example.com', 'anothertest@example.com'],
    ...     "if(dweet.alertValue > 10) return 'TEST: Greater than 10'; if(dweet.alertValue < 10) return 'TEST: Less than 10';",
    ...     'this-is-a-key',
    ... )
    {
        u'thing': u'this_is_a_thing',
        u'condition': u"if(dweet.alertValue > 10) return 'TEST: Greater than 10'; if(dweet.alertValue < 10) return 'TEST: Less than 10';",
        u'is_demo': False,
        u'recipients': [
            {
                u'type': u'email',
                u'address': u'test@example.com',
            },
            {
                u'type': u'email',
                u'address': u'anothertest@example.com',
            }
        ]
    }


Get an alert (with status)::

    >>> dweepy.get_alert('this_is_a_thing', 'this-is-a-key')
    {
        u'status': {
            u'message': u'',
            u'since': None,
            u'open': False,
            u'alerts_sent_today': 0,
            u'alerts_allowed_today': 100,
        },
        u'thing': u'this_is_a_thing',
        u'condition': u"if(dweet.alertValue > 10) return 'TEST: Greater than 10'; if(dweet.alertValue < 10) return 'TEST: Less than 10';",
        u'is_demo': False,
        u'recipients': [
            {
                u'type': u'email',
                u'address': u'test@example.com'
            },
            {
                u'type': u'email',
                u'address': u'anothertest@example.com'
            }
        ]
    }


Remove an alert::

    >>> dweepy.remove_alert('this_is_a_thing', 'this-is-a-key')
    {
        u'thing': u'this_is_a_thing'
    }


Subscriptions & Notifications
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~


You can create a real-time subscription to dweets using a "chunked" HTTP response.::

    >>> for dweet in dweepy.listen_for_dweets_from('this_is_a_thing'):
    >>>     print dweet
    {u'content': {u'some_key': u'some_value'}, u'thing': u'this_is_a_thing', u'created': u'2014-03-19T10:45:28.934Z'}
    {u'content': {u'some_key': u'some_value'}, u'thing': u'this_is_a_thing', u'created': u'2014-03-19T10:45:31.574Z'}

The server will keep the connection alive and send you dweets as they arrive.


Locking & Security
~~~~~~~~~~~~~~~~~~

By default, all things are publicly accessible if you know the 
name of the thing. You can also lock things so that they are 
only accessible to users with valid security credentials. 
To purchase locks, visit 
`https://renesas.dweet.io/locks <https://renesas.dweet.io/locks>`_. 
The locks will be emailed to you.

This is the pricing information from the renesas.dweet.io/locks
site.

    *Locks are just $1.99 a month and will be emailed to you as 
    soon as you check out. Locks come with 30 day storage for 
    each locked dweet. That's up to 2.5 million dweets a month! 
    If you need more storage, just get in touch.*


To lock a thing::

    >>> dweepy.lock("my-thing", "my-lock", "my-key")


To unlock a thing::

    >>> dweepy.unlock("my-thing", "my-key")
    "my-thing"


To remove a lock no matter what it's attached to::

    >>> dweepy.remove_lock("my-lock", "my-key")
    "my-lock"


Once a thing has been locked, you must pass the key to the lock with any call you make to other functions in this client library. The key will be passed as an optional keyword argument. For example::

    >>> dweepy.dweet_for("my-locked-thing", {"some":"data"}, "my-key")
    >>> dweepy.get_latest_dweet_for("my-locked-thing", "my-key")
    >>> dweepy.get_dweets_for("my-locked-thing", "my-key")
    >>> dweepy.listen_for_dweets_from("my-locked-thing", "my-key")

Failure to pass a key or passing an incorrect key for a locked thing will result in an exception being raised.


Error Handling
~~~~~~~~~~~~~~

When dweepy encounters an error a ``DweepyError`` exception is raised. This can happen either when a HTTP request to the dweet.io API fails with an invalid status code, or if the HTTP request succeeds but the request fails for some reason (invalid key, malformed request data, invalid action etc.).


Request Sessions
~~~~~~~~~~~~~~~~

Each API call allows a request ``Session`` to be optionally set to persist certain parameters across dweepy calls. Sessions can be used for:

* reusing the the underlying TCP connection if you're making several requests to the same host
* configuring HTTP Proxies
* enabling timeouts for HTTP requests

Further information of requests session can be found in 
`Request Session Advanced Usage <http://docs.python-requests.org/en/master/user/advanced/>`_.

To enable a session (in this case with a 5 second timeout)::

    >>> import requests
    >>> session_with_timeout = requests.session(timeout=5.0)


The session may be used in all dweepy API calls::

    >>> dweepy.dweet({'some_key': 'some_value'}, session=session_with_timeout)
    >>> dweepy.dweet_for('this_is_a_thing', {'some_key': 'some_value'}, session=session_with_timeout)


Additional information
----------------------

`S5D9 Hacking Guide <https://codetricity.github.io/s5d9-hacking/>`_


Copyright & License
-------------------

| Original Copyright (c) 2014 Patrick Carey (https://github.com/paddycarey)
| Licensed under the **MIT** license.
| Documentation updated by iotcommunity.io in 2017 to help dweepy work
| with the Renesas IoT Sandbox