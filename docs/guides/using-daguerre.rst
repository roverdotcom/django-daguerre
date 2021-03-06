Using daguerre
==============

.. module:: daguerre.templatetags.daguerre

.. templatetag:: {% adjust %}

{% adjust %}
++++++++++++

The easiest way to use Daguerre is through the :ttag:`{% adjust %}`
template tag:

.. code-block:: html+django

    {% load daguerre %}
    <img src="{% adjust my_model.image width=128 height=256 %}" />

:mod:`daguerre` works directly with any ImageField (or storage path).
There is no magic. You don't need to change your models. It Just Works.

Let's be lazy
-------------

So the :ttag:`{% adjust %}` tag renders as a URL, which gets an
adjusted image, right? Well, yes, but in a very lazy fashion. It
actually renders a URL to an adjustment view, which runs the
adjustment (if necessary), and then redirects the user to the actual
adjusted image's URL.

The upshot is that no matter how many :ttag:`{% adjust %}`
tags you have on a page, it will render as quickly when the
thumbnails already exist as it will when the thumbnails still need
to be created. The thumbnails will then be filled in as the user
starts to request them.

.. note::

    The adjustment view has some light security in place to
    make sure that users can't run arbitrary image resizes on your
    servers.

Different adjustments
---------------------

The :ttag:`{% adjust %}` tag currently supports three different
adjustments: **fit, fill, and crop.** These can be passed in as an
additional parameter to the tag:

.. code-block:: html+django

    <img src="{% adjust my_model.image width=128 height=256 adjustment="fit" %}" />

Take this picture:

.. figure:: /_static/lenna.png

    Full size: 512x512

Let's use :ttag:`{% adjust %}` with width 128 (25%) and height 256
(50%), with each of the three adjustments.

+-----------------------------------+------------------------------------+------------------------------------+
| "fit"                             | "fill" (default)                   | "crop"                             |
+===================================+====================================+====================================+
| .. image:: /_static/lenna_fit.png | .. image:: /_static/lenna_fill.png | .. image:: /_static/lenna_crop.png |
+-----------------------------------+------------------------------------+------------------------------------+
| Fits the entire image into the    | Fills the entire space given by    | Crops the image to the given       |
| given dimensions without          | the dimensions by cropping to the  | dimensions without any resizing.   |
| distorting it.                    | same width/height ratio and then   |                                    |
|                                   | scaling.                           |                                    |
+-----------------------------------+------------------------------------+------------------------------------+

.. note::

    If you have defined :class:`.Area`\ s for an image in the admin,
    those will be protected as much as possible (according to their
    priority) when using the crop or fill adjustments. Otherwise,
    any cropping will be done evenly from opposing sides.

Getting adjusted width and height
---------------------------------

.. code-block:: html+django

    {% load daguerre %}
    {% adjust my_model.image width=128 height=128 adjustment="fit" as image %}
    <img src="{{ image }}" width={{ image.width }} height={{ image.height }} />

The object being set to the ``image`` context variable is an
:class:`.AdjustmentInfoDict` instance. In addition to rendering as
the URL for an image, this object provides access to some other
useful pieces of information—in particular, the width and height
that the adjusted image *will have*, based on the width and height
of the original image and the parameters given to the tag. This can
help you avoid changes to page flow as adjusted images load.

Named crops (advanced)
----------------------

If you are defining :class:`.Area`\ s in the admin, you can refer to
these by name to pre-crop images **before** applying the adjustment
you've selected. For example:

.. code-block:: html+django

    {% load daguerre %}
    <img src="{% adjust my_model.image width=128 height=128 adjustment="fit" crop="face" %}" />

This would first crop the image to the "face" :class:`.Area` (if available)
and then fit that cropped image into a 128x128 box.

.. note::

    If a named crop is being used, :class:`.Area`\ s will be
    ignored even if you're using a fill or crop adjustment. (This may
    change in the future.)


.. templatetag:: {% adjust_bulk %}

{% adjust_bulk %}
+++++++++++++++++

If you are using a large number of similar adjustments in one
template - say, looping over a queryset and adjusting the same
attribute each time - you can save yourself queries by using
:ttag:`{% adjust_bulk %}`.

.. code-block:: html+django

    {% load daguerre %}
    {% adjust_bulk my_queryset "method.image" width=200 height=400 as adjusted_list %}
    {% for my_model, image in adjusted_list %}
      <img src="{{ image }}" />
    {% endfor %}

The syntax is similar to :ttag:`{% adjust %}`, except that:

* ``as <varname>`` is required.
* an iterable (``my_queryset``) and an lookup to be performed on each
  item in the iterable (``"method.image"``) are provided in place
  of an image file or storage path.
* :ttag:`{% adjust_bulk %}` **doesn't support named crops**.

Editing Areas
+++++++++++++

Daguerre provides a widget which can be used with any
:class:`ImageField` to edit :class:`Areas <.Area>` for that image file.
Using this widget with a :class:`ModelAdmin` is as simple as defining
appropriate `formfield_overrides`_.

.. code-block:: python

    from daguerre.widgets import AreaWidget

    class YourModelAdmin(admin.ModelAdmin):
        formfield_overrides = {
            models.ImageField: {'widget': AreaWidget},
        }
        ...

.. _formfield_overrides: https://docs.djangoproject.com/en/dev/ref/contrib/admin/#django.contrib.admin.ModelAdmin.formfield_overrides
