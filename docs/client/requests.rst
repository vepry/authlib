.. _requests_client:


OAuth for Requests
==================

.. meta::
    :description: An OAuth 1.0 and OAuth 2.0 Client implementation for Python requests,
        including support for OpenID Connect and service account, powered by Authlib.

.. module:: authlib.integrations.requests_client
    :noindex:

Requests is a very popular HTTP library for Python. Authlib enables OAuth 1.0
and OAuth 2.0 for Requests with its :class:`OAuth1Session`, :class:`OAuth2Session`
and :class:`AssertionSession`.


Requests OAuth 1.0
------------------

There are three steps in :ref:`oauth_1_session` to obtain an access token:

1. fetch a temporary credential
2. visit the authorization page
3. exchange access token with the temporary credential

It shares a common API design with :ref:`httpx_client`.

OAuth1Session
~~~~~~~~~~~~~

The requests integration follows our common guide of :ref:`oauth_1_session`.
Follow the documentation in :ref:`oauth_1_session` instead.

OAuth1Auth
~~~~~~~~~~

It is also possible to use :class:`OAuth1Auth` directly with in requests.
After we obtained access token from an OAuth 1.0 provider, we can construct
an ``auth`` instance for requests::

    auth = OAuth1Auth(
        client_id='YOUR-CLIENT-ID',
        client_secret='YOUR-CLIENT-SECRET',
        token='oauth_token',
        token_secret='oauth_token_secret',
    )
    requests.get(url, auth=auth)


Requests OAuth 2.0
------------------

In :ref:`oauth_2_session`, there are many grant types, including:

1. Authorization Code Flow
2. Implicit Flow
3. Password Flow
4. Client Credentials Flow

And also, Authlib supports non Standard OAuth 2.0 providers via Compliance Fix.

Follow the common guide of :ref:`oauth_2_session` to find out how to use
requests integration of OAuth 2.0 flow.


Using ``client_secret_jwt`` in Requests
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

There are **three default client authentication methods** defined for
``OAuth2Session``. But what if you want to use ``client_secret_jwt`` instead?
Here is how you could ``.register_client_auth_method`` it for Requests::

    from authlib.integrations.requests_client import OAuth2Session
    from authlib.oauth2.rfc7523 import ClientSecretJWT

    session = OAuth2Session(
        'your-client-id', 'your-client-secret',
        token_endpoint_auth_method='client_secret_jwt'
    )
    token_endpoint = 'https://example.com/oauth/token'
    session.register_client_auth_method(ClientSecretJWT(token_endpoint))
    session.fetch_token(token_endpoint)

The ``ClientSecretJWT`` is provided by :ref:`specs/rfc7523`.

Using ``private_key_jwt`` in Requests
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

What if you want to use ``private_key_jwt`` client authentication method,
here is the way with  ``.register_client_auth_method`` for Requests::

    from authlib.integrations.requests_client import OAuth2Session
    from authlib.oauth2.rfc7523 import PrivateKeyJWT

    with open('your-private-key.pem', 'rb') as f:
        private_key = f.read()

    session = OAuth2Session(
        'your-client-id', private_key,
        token_endpoint_auth_method='private_key_jwt',
    )
    token_endpoint = 'https://example.com/oauth/token'
    session.register_client_auth_method(PrivateKeyJWT(token_endpoint))
    session.fetch_token(token_endpoint)

The ``PrivateKeyJWT`` is provided by :ref:`specs/rfc7523`.


OAuth2Auth
~~~~~~~~~~

Already obtained access token? We can use :class:`OAuth2Auth` directly in
requests. But this OAuth2Auth can not refresh token automatically for you.
Here is how to use it in requests::

    token = {'token_type': 'bearer', 'access_token': '....', ...}
    auth = OAuth2Auth(token)
    requests.get(url, auth=auth)


Requests OpenID Connect
-----------------------

OpenID Connect is built on OAuth 2.0. It is pretty simple to communicate with
an OpenID Connect provider via Authlib. With Authlib built-in OAuth 2.0 system
and JsonWebToken (JWT), parsing OpenID Connect ``id_token`` could be very easy.

Understand how it works with :ref:`oidc_session`.


Requests Service Account
------------------------

The Assertion Framework of OAuth 2.0 Authorization Grants is also known as
service account. With the implementation of :class:`AssertionSession`, we can
easily integrate with a "assertion" service.

Checking out an example of Google Service Account with :ref:`assertion_session`.


Close Session Hint
------------------

Developers SHOULD **close** a Requests Session when the jobs are done. You
can call ``.close()`` manually, or use a ``with`` context to automatically
close the session::

    session = OAuth2Session(client_id, client_secret)
    session.get(url)
    session.close()

    with OAuth2Session(client_id, client_secret) as session:
        session.get(url)
