.. _client:


API Documentation
=================
This is the primary API client to make API calls. It deals with constructing
and executing XML-RPC calls against the SoftLayer API. Below are some links
that will help to use the SoftLayer API.


* `SoftLayer API Documentation <http://sldn.softlayer.com/reference/softlayerapi>`_
* `Source on GitHub <https://github.com/softlayer/softlayer-python>`_

::

	>>> import SoftLayer
	>>> client = SoftLayer.Client(username="username", api_key="api_key")
	>>> resp = client['Account'].getObject()
	>>> resp['companyName']
	'Your Company'


Getting Started
---------------
You can pass in your username and api_key when creating a SoftLayer client
instance. However, you can also set these in the environmental variables
'SL_USERNAME' and 'SL_API_KEY'.

Creating a client instance by passing in the username/api_key:
::

    import SoftLayer
    client = SoftLayer.Client(username='YOUR_USERNAME', api_key='YOUR_API_KEY')

Creating a client instance with environmental variables set:
::

    $ export SL_USERNAME=YOUR_USERNAME
    $ export SL_API_KEY=YOUR_API_KEY
    $ python
    >>> import SoftLayer
    >>> client = SoftLayer.Client()

Below is an example of creating a client instance with more options. This will
create a client with the private API endpoint (only accessible from the
SoftLayer private network) and a timeout of 4 minutes.
::

    client = SoftLayer.Client(username='YOUR_USERNAME',
                              api_key='YOUR_API_KEY'
                              endpoint_url=SoftLayer.API_PRIVATE_ENDPOINT,
                              timeout=240)

Managers
--------
For day-to-day operation, most users will find the managers to be the most
convenient means for interacting with the API. Managers abstract a lot of the
complexities of using the API into classes that provide a simpler interface to
various services. These are higher-level interfaces to the SoftLayer API.
::

	from SoftLayer import VSManager, Client
	client = Client(...)
	vs = VSManager(client)
	vs.list_instances()
	[...]

**Available managers**:

.. toctree::
   :maxdepth: 1
   :glob:

   managers/*

If you need more power or functionality than the managers provide, you can
make direct API calls as well.


Making API Calls
----------------
For full control over your account and services, you can directly call the
SoftLayer API. The SoftLayer API client for python leverages SoftLayer's
XML-RPC API. It supports authentication, object masks, object filters, limits,
offsets, and retrieving objects by id. The following section assumes you have
an initialized client named 'client'.

The best way to test our setup is to call the
`getObject <http://sldn.softlayer.com/reference/services/SoftLayer_Account/getObject>`_
method on the
`SoftLayer_Account <http://sldn.softlayer.com/reference/services/SoftLayer_Account>`_
service.
::

    client['Account'].getObject()

For a more complex example we'll retrieve a support ticket with id 123456 along
with the ticket's updates, the user it's assigned to, the servers attached to
it, and the datacenter those servers are in. To retrieve our extra information
using an `object mask <http://sldn.softlayer.com/article/Extended-Object-Masks>`_.

Retrieve a ticket using object masks.
::

    ticket = client['Ticket'].getObject(
        id=123456, mask="updates, assignedUser, attachedHardware.datacenter")


Now add an update to the ticket with
`Ticket.addUpdate <http://sldn.softlayer.com/reference/services/SoftLayer_Ticket/addUpdate>`_.
This uses a parameter, which translate to positional arguments in the order
that they appear in the API docs.
::

    update = client['Ticket'].addUpdate({'entry' : 'Hello!'}, id=123456)

Let's get a listing of virtual guests using the domain example.com
::

    client['Account'].getVirtualGuests(
        filter={'virtualGuests': {'domain': {'operation': 'example.com'}}})

This call gets tickets created between the beginning of March 1, 2013 and
March 15, 2013.
::

    client['Account'].getTickets(
        filter={
            'tickets': {
                'createDate': {
                    'operation': 'betweenDate',
                    'options': [
                        {'name': 'startDate', 'value': ['03/01/2013 0:0:0']},
                        {'name': 'endDate', 'value': ['03/15/2013 23:59:59']}
                    ]
                }
            }
        }
    )

SoftLayer's XML-RPC API also allows for pagination.
::

    client['Account'].getVirtualGuests(limit=10, offset=0)  # Page 1
    client['Account'].getVirtualGuests(limit=10, offset=10)  # Page 2

Here's how to create a new Cloud Compute Instance using
`SoftLayer_Virtual_Guest.createObject <http://sldn.softlayer.com/reference/services/SoftLayer_Virtual_Guest/createObject>`_.
Be warned, this call actually creates an hourly virtual server so this will
have billing implications.
::

    client['Virtual_Guest'].createObject({
            'hostname': 'myhostname',
            'domain': 'example.com',
            'startCpus': 1,
            'maxMemory': 1024,
            'hourlyBillingFlag': 'true',
            'operatingSystemReferenceCode': 'UBUNTU_LATEST',
            'localDiskFlag': 'false'
        })


API Reference
-------------
.. autoclass:: SoftLayer.Client
   :members:
   :undoc-members:

   .. automethod:: SoftLayer.Client.__getitem__

.. autoclass:: SoftLayer.API.Service
   :members:
   :undoc-members:

   .. automethod:: SoftLayer.API.Service.__call__


.. automodule:: SoftLayer.exceptions
   :members:
   :undoc-members:


Backwards Compatibility
-----------------------
As of version 3.0 of the API bindings, the old API methods and parameters no
longer work. Below are examples of converting old API calls to the new ones.

**Get the IP address for an account**
::

	# Old
	import SoftLayer.API
	client = SoftLayer.API.Client('SoftLayer_Account', None, 'username', 'api_key')
	client.set_object_mask({'ipAddresses' : None})
	client.set_result_limit(10, offset=10)
	client.getObject()

	# New
	import SoftLayer
	client = SoftLayer.Client(username='username', api_key='api_key')
	client['Account'].getObject(mask="ipAddresses", limit=10, offset=0)

**Importing the module**
::

	# Old
	import SoftLayer.API

	# New
	import SoftLayer

**Creating a client instance**
::

	# Old
	client = SoftLayer.API.Client('SoftLayer_Account', None, 'username', 'api_key')

	# New
	client = SoftLayer.Client(username='username', api_key='api_key')
	service = client['Account']

**Making an API call**
::

	# Old
	client = SoftLayer.API.Client('SoftLayer_Account', None, 'username', 'api_key')
	client.getObject()

	# New
	client = SoftLayer.Client(username='username', api_key='api_key')
	client['Account'].getObject()

	# Optionally
	service = client['Account']
	service.getObject()

**Setting Object Mask**
::

	# Old
	client.set_object_mask({'ipAddresses' : None})

	# New
	client['Account'].getObject(mask="ipAddresses")

**Using Init Parameter**
::

	# Old
	client.set_init_parameter(1234)

	# New
	client['Account'].getObject(id=1234)

**Setting Result Limit and Offset**
::

	# Old
	client.set_result_limit(10, offset=10)

	# New
	client['Account'].getObject(limit=10, offset=10)

**Adding Additional Headers**
::

	# Old
	# These headers are persisted accross API calls
	client.add_header('header', 'value')

	# New
	# These headers are NOT persisted accross API calls
	client['Account'].getObject(headers={'header': 'value'})

**Removing Additional Headers**
::

	# Old
	client.remove_header('header')

	# New
	client['Account'].getObject()

**Adding Additional HTTP Headers**
::

	# Old
	client.add_raw_header('header', 'value')

	# New
	client['Account'].getObject(raw_headers={'header': 'value'})

**Changing Authentication Credentials**
::

	# Old
	client.set_authentication('username', 'api_key')

	# New
	client.username = 'username'
	client.api_key = 'api_key'
