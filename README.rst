django-hostproof-auth
======================

.. image:: https://travis-ci.org/jpintado/django-hostproof-auth.png?branch=master
    :target: https://travis-ci.org/jpintado/django-hostproof-auth

.. image:: https://coveralls.io/repos/jpintado/django-hostproof-auth/badge.svg?branch=master
  :target: https://coveralls.io/r/jpintado/django-hostproof-auth?branch=master

.. image:: https://pypip.in/v/django_hostproof_auth/badge.png
    :target: https://crate.io/packages/django_hostproof_auth/
    :alt: Latest PyPI version

.. image:: https://pypip.in/d/django_hostproof_auth/badge.png
    :target: https://crate.io/packages/django_hostproof_auth/
    :alt: Number of PyPI downloads

Secure Host-Proof authentication backend for Django-powered sites.

The password is never transmitted to the server. The server is limited to persisting and retrieving whatever encrypted data is sent to it, and never actually accesses the sensitive data in its plain form.


Requirements
========

- Python 2.6, 2.7, 3.2 or 3.3

- Django (1.6+)

- rsa


Installation
============

The easiest way to install is with pip_::

    pip install django_hostproof_auth
    
or clone from github_:

- Clone the repository::

    git clone https://github.com/jpintado/django-hostproof-auth.git

- Install the package::

    python setup.py install

You could require root permissions to execute the previous commands.
    

Configuration
=============

- In **settings.py**:

  - Add ``hostproof_auth`` to ``INSTALLED_APPS``.

  - Add the authentication backend to your application::

      AUTH_USER_MODEL = 'hostproof_auth.User'

      AUTHENTICATION_BACKENDS = (
          'hostproof_auth.auth.ModelBackend',
      )

- Include *hostproof_auth* in your **urls.py** with some prefix::

      urlpatterns = patterns('',
          # ... snip ...
          url(r'^auth/', include('hostproof_auth.urls')),
          # ... snip ...
      )

Usage
=====

*django-hostproof-auth* provides a JavaScript client to register and login users in your django application. 
You can easily access to this client in your templates by including the following::

  {% load staticfiles %}

  <script type="text/javascript" src="{% static "hostproof_auth/hostproof_auth.js" %}"></script>

The JavaScript client uses the SJCL library and Jquery_, so in case you don't have already in your project you can use the version included in the package (Jquery 2.0.3):

.. _Jquery: http://www.jquery.com/

::

  <script type="text/javascript" src="{% static "hostproof_auth/sjcl.js" %}"></script>
  <script type="text/javascript" src="{% static "hostproof_auth/jquery-2.0.3.min.js" %}"></script>
  
These are examples about the use of this client that you can directly include in your login/registration templates:

Registration
------------

::

    {% load staticfiles %}

    <script type="text/javascript" src="{% static "hostproof_auth/sjcl.js" %}"></script>
    <script type="text/javascript" src="{% static "hostproof_auth/jquery-2.0.3.min.js" %}"></script>
    <script type="text/javascript" src="{% static "hostproof_auth/hostproof_auth.js" %}"></script>
    <script>
        function doRegistration() {
            username = $("#id_username").val();
            email = $("#id_email").val();
            password = $("#id_password").val();
            $.when(
                register("{% url 'hostproof_auth_register' %}", username, email, password)
            ).done(function(d) {
                console.log(d);
                //Registration completed. Redirect to desired page.
            }).fail(function(d) {
                console.log(d);
                //Registration completed. Show desired message.
            });
        }
    </script>

    <input id="id_username" type="text" name="username" maxlength="100" />
    <input id="id_email" type="text" name="email" maxlength="100" />
    <input id="id_password" type="password" name="password" maxlength="100" /></p>
    <button onclick="doRegistration()">Register</button>


Login
-----

::  

    {% load staticfiles %}

    <script type="text/javascript" src="{% static "hostproof_auth/sjcl.js" %}"></script>
    <script type="text/javascript" src="{% static "hostproof_auth/jquery-2.0.3.min.js" %}"></script>
    <script type="text/javascript" src="{% static "hostproof_auth/hostproof_auth.js" %}"></script>
    <script>
        function doLogin() {
            username = $("#id_username").val();
            password = $("#id_password").val();
            $.when(
                login("{% url 'hostproof_auth_challenge' %}", username, password)
            ).done(function(d) {
                console.log(d);
                //Login completed. Redirect to desired page.
            }).fail(function(d){
                console.log(d);
                //Login Failed. Show desired message.
            });
        }
    </script>

    <input id="id_username" type="text" name="username" />
    <input id="id_password" type="password" name="password" />
    <button onclick="doLogin()">Login</button>

Advanced Usage
==============

You may create your own JavaScript client, or create a client in any other language. In that case, you will need to make the necessary requests to register and login users. Below is the documentation for these API requests:

Registration
------------

- POST request to the ``hostproof_auth_register`` URL (typically something like */auth/register/*) with the parameters:

  - username
  - email
  - encrypted_challenge
  - challenge
  
  The client application needs to generate a random string as challenge, and encrypt that string using a secure algorith (for example, AES-256) with the user password to generate the encrypted challenge.

  Example::
  
    username=foobar&email=foobar@domain.com&challenge=randomstring&encrypted_challenge=U2FsdGVkX19ED2i2M8uE3AySNJyKzw8SXtru9JQbNmo=

Login
-----

- GET request to the ``challenge`` URL (typically something like */auth/challenge/*) with the parameters.

  - username
  - format (OPTIONAL): specifies the response format. Supported "text" and "json". The default value is "text".

  Example::
  
    /challenge/?username=foobar&format=json

  Response::
  
    {
      "encrypted_challenge" : "U2FsdGVkX19ED2i2M8uE3AySNJyKzw8SXtru9JQbNmo="
    }
    
- POST request to the ``challenge`` URL with the parameters:

  - username
  - challenge: the challenge after the decryption with the user password.
  - format (OPTIONAL): specifies the response format. Supported "text" and "json". The default value is "text".

  The client application needs to decrypt the encrypted_challenge using the password, and send the original challenge as response to be authenticated.
  
  Example::

    username=foobar&challenge=randomstring&format=json

  Response::
    
    {
        "rsa_public": "-----BEGIN RSA PUBLIC KEY-----\nMEgCQQC6ZV2lMzO50HoJhznNat7pB+cVwY91Qpn58iIC8X4QleNatgyqJfZzu3RdwQQJDr2uUv+sXdEm+wYGBXg0gqZjAgMBAAE=\n-----END RSA PUBLIC KEY-----\n"
    }
 
 
.. _pip: https://pypi.python.org/pypi/django_hostproof_auth
.. _github: https://github.com/jpintado/django-hostproof-auth

