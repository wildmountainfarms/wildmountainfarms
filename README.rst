Wild Mountain Farms Docs
=======================================

This project contains documentation and resources for tech used on the ranch


Building
----------

To build this yourself, run these commands:

.. code-block:: shell

    cd docs/
    python3 -m venv .venv
    source .venv/bin/activate
    pip install -r source/requirements.txt
    make html
    xdg-open "file://$(pwd)/build/html/index.html"
    make latexpdf  # requires latexmk command
    make epub

