=====================
 Speaking in Tongues
=====================

.. ifslides::

   .. figure:: /_static/type.jpg
      :class: fill

      CC BY-NC https://www.flickr.com/photos/catalinr/103875147/

.. slide::

   .. figure:: /_static/before.png
      :class: fill

.. slide::

   .. figure:: /_static/after.png
      :class: fill

Aside: Locale vs Language
=========================

* Locales, not Languages
* Date & number formatting
* Local dialects
* ``en`` and ``fr`` are languages
* ``en-US``, ``fr-CA``, and ``fr-FR`` are locales

What We Did
===========

* Fully localized experience

  * Translated user interface
  * Locally relevant "featured content"
  * Localized interface: formatting, currency, etc.

* Served on local TLDs
* High degree of organizer control

.. note::

   There are several key decisions we had to make as we started this
   process. The first of them was how locales and TLDs interact.


Locales & TLDs
==============

.. rst-class:: build

* Are all locales available on all TLDs?
* What's the default on a TLD?

  * Content negotiation
  * TLD/rule based

Locales & TLDs at Eventbrite
============================

.. rst-class:: build

* Eventbrite maps your locale to a TLD
* Switching locales *may* switch your TLD
* Anonymous users get the TLD default
* Unless you're in Canada

Handling Authentication
=======================

* When you log in, we redirect to your preferred TLD/locale
* Switching TLDs means your cookies "disappear"
* Built "SSO" mechanism for this

  * Invalidates old cookies
  * Issues new ones

Canonical URLs
==============

* SEO penalty for duplicate content
* Which TLD does an event page "belong" under?
* We use the organizer's preferred TLD to determine this

.. slide:: Execution
   :level: 1

Default Locales
===============

* Custom middleware handles determining locale

  * Cookie
  * Geo IP
  * HTTP-Language-Accept
  * TLD

* Scribbles on ``request``
* Middleware is long-lived!

Setting the Locale
==================

Last thing middleware does is activate the locale.

.. code-block:: python

   from django.utils import translation

   class LocaleMiddleware(object):

       def process_request(self, request):

           locale = ... # Eventbrite rules: cookie, geoip, etc.

           translation.activate(locale)

This loads the translations from disk or cache.

Marking Translations
====================

The ``_`` function traditionally marks translations.

.. code-block:: python

   from django.utils.translation import ugettext as _

   _("Hello, world")

.. code-block:: python

   _("Hello, %(name)s") % ("Nathan",)


Marking Translations (Mako)
---------------------------

.. code-block:: none

   ${ _('Hello, world') }

.. code-block:: none

   ${ _("Hello, %(name)s") % ("Nathan",) }


Marking Translations (Django templates)
---------------------------------------

.. code-block:: none

   {% trans "Hello, world" %}

.. code-block:: none

   {% blocktrans %}Hello, {{ name }}.{% endblocktrans %}


Marking Translations (Javascript)
---------------------------------

Django's `Javascript catalog`_ provides a ``gettext`` and
``interpolate`` function.

.. _`Javascript catalog`: https://docs.djangoproject.com/en/1.6/topics/i18n/translation/#using-the-javascript-translation-catalog

.. code-block:: javascript

   window.gettext('Hello, world')

.. code-block:: javascript

   window.interpolate(
        window.gettext('Hello, %s'), ['Nathan']
     );

``gettext`` is easily extendable to take an object, simplifying the
syntax.

.. code-block:: javascript

    window.gettext('Hello, %(name)s', {
        name: 'Nathan'
    });


Marking Translation (Handlebars)
--------------------------------

.. code-block:: none

   {{_ "Hello, world" }}

.. code-block:: none

   {{_ 'Hello, %(name)s' name='Nathan' }}


Extracting Translations
=======================

* Many sources of content to localize

  * Python
  * Templates
  * Javascript
  * Handlebars

* Want a single tools for managing translations

pybabel
-------

* PyBabel_ supports pluggable extractors

.. code-block:: none

   [extractors]
   handlebars = i18n.extract_handlebars:extract_handlebars

   [handlebars: js/templates/src/**.handlebars]

   [javascript: js/src/**.js]

   [python: **.py]

   [mako: **/templates/**.html]
   input_encoding = utf-8

.. nextslide:: Custom Extractors

.. code-block:: python

   def extract_handlebars(file_object, keywords=None,
                           comment_tags=None, options={}):

       """Extract messages from handlebars template files.

       :param file_object: the file-like object the messages should be
                   extracted from
       :param keywords: not used, here to maintain compatibiliy with babel
       :param comment_tags: not used, here to maintain compatibiliy
                            with babel
       :param options: a dictionary of additional options (optional)
       :return: an iterator over ``(line_number, funcname, message,
                comments)`` tuples, funcname and comments do not apply
                for the handlebars extraction """

Using Translations
==================

* Django expects everything will be in the ``django`` or ``djangojs``
  domain
* Translations are stored in files with a specific path format:

  ``<locale>/LC_MESSAGES/django.po``

Where to Store Translations
---------------------------

* Django looks for these paths in:

  * A ``locale`` directory in each application
  * A list of paths specified in ``settings.LOCALE_PATHS``
  * A ``django/conf/locale/`` directory on ``sys.path``

Compiling Translations
----------------------

* You'll want to compile to binary ``.mo`` files

::

  $ pybabel compile -D django -d django/conf/locale/ -f

::

  $ django-admin.py compilemessages --locale=pt_BR


Localization
============

* Context processor to make helper funcs available
* Babel ships with full library of formatting data

* ``format_date``
* ``format_number``
* ``format_currency``

Bytes v. Strings
----------------

* Legacy code expects UTF-8 bytes
* Django returns Unicode objects
* This complicates helpers
* ``EncodedString`` smooths over these bumps

``EncodedString``
-----------------

.. code-block:: python

   EncodedString.new(
       format_date(event.start_date),
   )

* Stores encoding with the data, defaulting to UTF-8
* Subclasses ``str`` to avoid implicit up-cast when ``join``\ ing
* Overrides ``decode`` to use stored encoding
* https://gist.github.com/nyergler/10394248

Switching Locales
=================

* Events have a configurable locale
* Used for all event-related communication
* So we switch locales mid-stream for event emails, etc

Switch-And-Restore
==================

* This means another place call to ``activate``

.. code-block:: python

   from functools import wraps
   from django.utils.translation import get_language

   def preserve_request_locale(f):
       """Preserves the request locale across a function.

       Some functions make calls to activate as a one-off (sending email, etc)
       This decorator preserves the request locale.
       """

       def wrapper(*args, **kwargs):
           lang = translation.get_language()
           func = f(*args, **kwargs)
           translation.activate(lang)
           return func

       return wraps(f)(wrapper)

Lessons Learned
===============

* Multi-TLD support adds a lot of complexity
* Translators don't get the visual context
* Need to be careful with changing locales midstream
* Retrofitting best practices is hard (ie, concatenation)

Thanks!
=======

* http://github.com/nyergler/speaking-in-tongues
* nathan@eventbrite.com
* `@nyergler`_

.. _`@nyergler`: http://twitter.com/nyergler
