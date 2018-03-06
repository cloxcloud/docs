CloX Documentation
========================

This is the official repository of CloX's documentation. This
documentation is live at:
[http://clox.cloud/docs](http://clox.cloud/docs).

You are encouraged to fork this repo and send us pull requests!

Building
--------

Install dependencies: ``pip install -r requirements.txt``.

Build the documentation by running ``make html``.
[More information](http://sphinx-doc.org/).

Translations
------------

### Bootstrap a new language

    $ make gettext
    $ sphinx-intl update -c source/conf.py -p build/locale -l <lang>

### Translate

Translate your po files under ``./locale/<lang>/LC_MESSAGES/``.

### Publish the translation

    $ sphinx-intl build -c source/conf.py
    $ make -e SPHINXOPTS="-D language='<lang>'" html

### More Info

Follow [this guide](http://sphinx-doc.org/intl.html) for more information.
