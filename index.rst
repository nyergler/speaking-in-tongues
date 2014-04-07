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

* What we did
* Key Decisions
* Execution
* Issues
* Lessons Learned

What We Did
===========

* Fully localized experience

  * Translated content
  * Local formmating
  * Local defaults

* Served on local TLDs
* High degree of organizer control

Key Decisions
=============

* What language do anonymous users see?
* How do locales and TLDs interact
* How does authentication work
* What is the canonical URL for a page?

Execution
=========

* Default locale for TLDs
* "SSO" for handling authentication cross-sites
* Organizers' preference respected for event pages
* Links are never broken
* Localized Python, Javascript, handlebars, HTML from single catalog
* Event pages have their own locale

i18n code
=========

Marking
-------

Extracting
----------

Localization
------------

Issues
======

* Lots of subtle bugs with switching domains
* Need to be careful with swapping in event page locale
* Retrofitting best practices is hard (ie, concatenation)

Lessons Learned
===============

* Do you really need other TLDs?
* Translators don't get the visual context

Thanks!
=======

* http://github.com/nyergler/speaking-tonuges
* nathan@eventbrite.com
* `@nyergler`_

.. _`@nyergler`: http://twitter.com/nyergler
