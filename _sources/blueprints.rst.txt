Blueprints
==========

Makefile
--------

::


    #!/usr/bin/env make
    #
    # Makefile
    #
    # <license>

    export PACKAGE_NAME=md_toc

    default: doc

    doc: clean
    	pipenv run $(MAKE) -C docs html

    install:
    	pip3 install . --user

    uninstall:
    	pip3 uninstall $(PACKAGE_NAME)

    install-dev:
    	pipenv install --dev
    	pipenv run pre-commit install

    uninstall-dev:
    	pipenv --rm

    demo:
    	asciinema/$(PACKAGE_NAME)_asciinema_$$(git describe --tags $$(git rev-list --tags --max-count=1) | tr '.' '_')_demo.sh

    test:
    	python -m unittest $(PACKAGE_NAME).tests.tests --failfast --locals --verbose

    dist:
    	pipenv run python setup.py sdist
    	pipenv run python setup.py bdist_wheel
    	pipenv run twine check dist/*

    upload:
    	pipenv run twine upload dist/*

    clean:
    	rm -rf build dist *.egg-info tests/benchmark-results *.md
    	pipenv run $(MAKE) -C docs clean

    .PHONY: default doc install uninstall install-dev uninstall-dev test clean demo benchmar
