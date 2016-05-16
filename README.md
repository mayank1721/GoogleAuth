[![Build Status](https://travis-ci.org/wstrange/GoogleAuth.svg?branch=develop)](https://travis-ci.org/wstrange/GoogleAuth)
[![License](https://img.shields.io/badge/license-BSD-blue.svg?style=flat)](https://github.com/wstrange/GoogleAuth/blob/master/LICENSE)

README
======

GoogleAuth is a Java server library that implements the _Time-based One-time
Password_ (TOTP) algorithm specified in [RFC 6238][RFC6238].

This implementation borrows from [Google Authenticator][gauth], whose C code has
served as a reference, and was created upon code published in [this blog
post][tgb] by Enrico M. Crisostomo.


Whom Is This Library For
------------------------

Any developer who wants to add TOTP multi-factor authentication to a Java
application and needs the server-side code to create TOTP shared secrets
and verify TOTP passwords.

Users may use TOTP-compliant token devices (such as those you get from your bank),
or a software-based token application (such as Google Authenticator).

Requirements
------------

The minimum Java version required to build and use this library is Java 7.

Installing
----------

Add a dependency to your build environment.

If you are using Maven:

    <dependency>
      <groupId>com.warrenstrange</groupId>
      <artifactId>googleauth</artifactId>
      <version>0.5.0</version>
    </dependency>

If you are using Gradle:

     compile 'com.warrenstrange:googleauth:0.5.0'

The required libraries will be automatically pulled into your project:

  * Apache Commons Codec.
  * Apache HTTP client.

Client Applications
-------------------

Both the Google Authenticator client applications (available for iOS, Android
and BlackBerry) and its PAM module can be used to generate codes to be validated
by this library.

However, this library can also be used to build custom client applications if
Google Authenticator is not available on your platform or if it cannot be used.

Library Documentation
---------------------

This library includes full JavaDoc documentation and a JUnit test suite that can
be used as example code for most of the library purposes.

Texinfo documentation sources are also included and a PDF manual can be
generated by an Autotools-generated `Makefile`:

  * To bootstrap the Autotools, the included `autogen.sh` script can be used.

        $ ./autogen.sh

  * Configure and build the documentation:

        $ ./configure
        $ make pdf

Since typical users will not have a TeX distribution installed in their
computers, the PDF manuals for every version of GoogleAuth are hosted at
[this address][pdfdoc].

Usage
-----

The following code creates a new set of credentials for a user.  No user name is
provided to the API and it is a responsibility of the caller to save it for
later use during the authorisation phase.

    GoogleAuthenticator gAuth = new GoogleAuthenticator();
    final GoogleAuthenticatorKey key = gAuth.createCredentials();

The user should be given the value of the _shared secret_, returned by

    key.getKey()

so that the new account can be configured into its token device.  A convenience
method is provided to easily encode the secret key and the account information
into a QRcode.

When a user wishes to log in, he will provide the TOTP password generated by his
device.  By default, a TOTP password is a 6 digit integer that changes every 30
seconds.  Both the password length and its validity can be changed.  However,
many token devices such as Google Authenticator use the default values specified
by the TOTP standard and they do not allow for any customization.

The following code checks the validity of the specified `password` against the
provided Base32-encoded `secretKey`:

    GoogleAuthenticator gAuth = new GoogleAuthenticator();
    boolean isCodeValid = gAuth.authorize(secretKey, password);

Since TOTP passwords are time-based, it is essential that the clock of both the
server and the client are synchronised within the tolerance used by the
library.  The tolerance is set by default to a window of size 3 and can be
overridden when configuring a `GoogleAuthenticator` instance.

Scratch codes
-------------

By default 5 scratch codes are generated together with a new shared secret.
Scratch codes are meant to be a safety net in case a user loses access to their
token device.  Scratch nodes are _not_ a functionality required by the TOTP
standard and it is up to the developer to decide whether they should be used in
his application.

Storing User Credentials
------------------------

The library can assist with fetching and storing user credentials and a hook is
provided to users who want to integrate this functionality.  The
*ICredentialRepository* interface defines the contract between a credential
repository and this library.

The credential repository can be set in multiple ways:

  * The credential repository can be set on a per-instance basis, using the
  `credentialRepository` property of the `IGoogleAuthenticator` interface.

  * The library looks for instances of this interface using the
    [Java ServiceLoader API][serviceLoader] (introduced in Java 6), that is,
    scanning the `META-INF/services` package looking for a file named
    `com.warrenstrange.googleauth.ICredentialRepository` and, if found, loading
    the provider classes listed therein.

Two methods needs to be implemented in the *ICredentialRepository* interface.

  * `String getSecretKey(String userName)`.
  * `void saveUserCredentials(String userName, ...)`.

The credentials repository establishes the relationship between a user _name_
and its credentials.  This way, API methods receiving only a user name instead
of credentials can be used.

The following code creates a new set of credentials for the user `Bob` and
stores them on the configured `ICredentialRepository` instance:

    GoogleAuthenticator gAuth = new GoogleAuthenticator();
    final GoogleAuthenticatorKey key = gAuth.createCredentials("Bob");


The following code checks the validity of the specified `code` against the
secret key of the user `Bob` returned by the configured
`ICredentialRepository` instance:

    GoogleAuthenticator gAuth = new GoogleAuthenticator();
    boolean isCodeValid = gAuth.authorizeUser("Bob", code);


If an attempt is made to use such methods when no credential repository is
configured, an exception is thrown:

    java.lang.UnsupportedOperationException: An instance of the
      com.warrenstrange.googleauth.ICredentialRepository service must be
      configured in order to use this feature.

Bug Reports
-----------

Please, [read the manual][pdfdoc] before opening a ticket.  If you have read the
manual and you still think the behaviour you are observing is a bug, then open a
ticket on [github][githubIssues].

----

Copyright (c) 2013 Warren Strange

Copyright (c) 2014-2015 Enrico M. Crisostomo

All rights reserved.

Redistribution and use in source and binary forms, with or without
modification, are permitted provided that the following conditions are met:

* Redistributions of source code must retain the above copyright notice, this
  list of conditions and the following disclaimer.

* Redistributions in binary form must reproduce the above copyright notice,
  this list of conditions and the following disclaimer in the documentation
  and/or other materials provided with the distribution.

* Neither the name of the author nor the names of its
  contributors may be used to endorse or promote products derived from
  this software without specific prior written permission.

THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS"
AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE
IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE
DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE LIABLE
FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL
DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR
SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER
CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY,
OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE
OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.

[RFC6238]: https://tools.ietf.org/html/rfc6238
[gauth]: https://code.google.com/p/google-authenticator/
[tgb]: http://thegreyblog.blogspot.com/2011/12/google-authenticator-using-it-in-your.html?q=google+authenticator
[serviceLoader]: http://docs.oracle.com/javase/6/docs/api/java/util/ServiceLoader.html
[SecureRandom]: http://docs.oracle.com/javase/8/docs/api/java/security/SecureRandom.html
[sr-algorithms]: http://docs.oracle.com/javase/8/docs/technotes/guides/security/StandardNames.html#SecureRandom
[githubIssues]: https://github.com/wstrange/GoogleAuth/issues
[pdfdoc]: https://drive.google.com/folderview?id=0BxZtP9CHH-Q6TzRSaWtkQ0pEYk0&usp=sharing
