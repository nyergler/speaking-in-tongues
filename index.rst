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

Agenda
======

* What changed?
* Key decision points
* Localizing Eventbrite
* Lessons learned / time machine dreams

Aside: Locale vs Language
=========================

* Locales, not Languages
* Date & number formatting
* Dialects
* ``fr-CA`` and ``fr-FR`` are locales

What We Did
===========

* Fully localized experience

  * Translated user interface
  * Locally relevant "featured content"
  * Localized interface: formatting, currency, etc.

* Served on local TLDs
* High degree of organizer control

Key Decisions
=============

* What locale do anonymous users use?
* How do locales and TLDs interact
* How does authentication work
* What is the canonical URL for a page?

Locales & TLDs
==============

.. rst-class:: build

* Are all locales available on all TLDs?
* What's the default on a TLD?

  * Content negotiation
  * "Best guess"

Locales & TLDs
==============

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

  * Geo IP
  * HTTP-Language-Accept
  * TLD

* Scribbles on ``request``
* Remember middleware is long-lived!

Setting the Locale
==================

Last thing middleware does is activate the locale.

.. code-block:: python

   from django.utils import translation

   class LocaleMiddleware(object):

       def process_request(self, request):

           locale = ... # black magic

           translation.activate(locale)

This loads the translations from disk or cache.

Links Don't Break
=================

* Locale-enabled views redirect if needed
* Critically important for user-controlled content

Marking Translations
====================

The ``_`` function traditionally marks translations.

.. code-block:: python

   from i18n.translation import ugettext as _

   _("Hello, world")

.. nextslide:: Marking Translations (Mako)

.. code-block:: none

   ${ _('Hello, world') }

.. nextslide:: Marking Translations (Javascript)

.. code-block:: javascript

   window.gettext('Hello, world')

.. nextslide:: Marking Translation (Handlebars)

.. code-block:: none

   {{_ "Hello, world" }}

.. code-block:: javascript

    Handlebars.registerHelper('_',
        function(text, options){

          // ...
        }
    );

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


Event Pages
===========

* Events have a configurable locale
* Used for all event-related communication
* So we switch locales mid-stream for event emails, etc

Switch-And-Restore
==================

* This means another place call to ``activate``

.. code-block:: python

   from functools import wraps
   from common.utils.i18n import set_language, get_language

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


Issues
======

* Subtle bugs with switching domains (SSO)
* Need to be careful with changing locales midstream
* Retrofitting best practices is hard (ie, concatenation)

Lessons Learned
===============

* Do you really need other TLDs?
* Translators don't get the visual context
* Start early

Thanks!
=======

* http://github.com/nyergler/speaking-tongues
* nathan@eventbrite.com
* `@nyergler`_

.. _`@nyergler`: http://twitter.com/nyergler
