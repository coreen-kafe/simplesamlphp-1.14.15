Information cards module for simpleSAMLphp
==========================================

**WARNING: THIS IS NOT mature software, it's released with testing, educational, developing purposes. It's on a very
early version, so don't rely the life of anybody on it.**


Introduction
-------------

This is a SimpleSAMLphp module that works with Information Cards technologies and provides two basic functionalities:

* **RP**: acting as a _Relying Party_, you can accept user authentication through InfoCards consuming tokens sent by
an STS.

* **STS**: acting as a _Secure Token Service_ you can provide information to a RP generating tokens. Currently, only
user password and self issued credentials are supported.

* **InfoCard Generator**: your users could request their InfoCard filling a form with their username and password.


Very important
--------------

This document is not a strict guide, I mean it might have some errors or missed information. I've tried to comment
almost every trick I've used to make the system work and make your life easier. So, if at any point of the installation
you feel lost, breathe twice, think for ten minutes in what you are trying to do, read again the documentation and use
your common sense. It'll be useful when you'll face again the configuration file.


Installation
------------

Once you have installed SimpleSAMLphp, installing this module is very simple. Just execute the following
command in the root of your SimpleSAMLphp installation:

```
composer.phar require simplesamlphp/simplesamlphp-module-infocard:dev-master
```

where `dev-master` instructs Composer to install the `master` branch from the Git repository. See the
[releases](https://github.com/simplesamlphp/simplesamlphp-module-infocard/releases) available if you
want to use a stable version of the module.

When you have successfully installed your module, you need to proceed with the following steps:

1. Copy the _InfoCard_ folder in your modules directory in your SimpleSAMLphp installation directory.
2. Copy (or move) the file `modules/InfoCard/extra/config-login-infocard.php` to the config directory in your
SimpleSAMLphp installation directory.
3. Edit the `config/config-login-infocard.php` file, you should configure some values like: `help_desk_email_URL`,
`contact_info_URL`, `server_key`, `server_crt`, `sts_crt`, `requiredClaims` and `optionalClaims` to fit your needs.
4. Edit the `config/authsources.php` file, add the following text before the last `);`:

```php
        'InfoCard' => array(
            'InfoCard:ICAuth',
        ),
```

5. That's all.


Adding an Infocard Generator
----------------------------

1. Go into the `modules/InfoCard` folder.
2. Copy `extra/getinfocard.php` to `www/getinfocard.php`
3. Edit the `config/config-login-infocard.php` file and uncomment this line:

```php
    // 'CardGenerator' => 'getinfocard.php', (delete the heading slashes "//").
```

4. Following the previous example, uncomment this `values:certificates`, `sts_key`, `tokenserviceurl` and `mexurl`.
6. Check the previous values and modify them if you need.
5. Read the **User functions** section.


Adding the STS functionality
----------------------------

1. Go into the `modules/InfoCard` folder.
2. Copy `extra/mex.php` and `extra/tokenservice.php` to the `www` folder.
3. Edit the `config/config-login-infocard.php` file and uncomment the values: `certificates` and `sts_key`.
4. Read the **User functions** section.


User functions
--------------

Because there are many authentication issues I cannot guess for you, you'll have to code a little bit to fit this
module to your authentication system.

We we'll work with the file `UserFunctions.php` located in `modules/InfoCard/lib/`.

* `validateUser`: it receives two strings, username and password, you do the validation (against your database?) and
return true if you want to validate the user or false instead.

* `fillClaims`: it's used by the _tokenservice_ to give information about the user to the relying party. It receives
the username, the configured required and optional claims and the claims requested by the RP. It works filling the
_claimValues_ array and your job is the ensure the _value_ variable (`$claimValues[$claim]['value']=` ) of the array
gets the value you want. Understand that requested values and your configured ones could not match.

* `fillICdata`: it's used by the card generator to retrieve needed information. It receives an authenticated username
and returns an array containing information such as the card name, the card image, the expiring time, etc.


Certificates and hosts
----------------------

The architecture is composed by three independent elements:

* **User**: Identity Selector
* **IDP**: Relying Party
* **STS**: IC and token generation.

That's because you should configure two hosts (with two x509 certificates) if you want two have the IDP and STS
functionalities in the same machine.
