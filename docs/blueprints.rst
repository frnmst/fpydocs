Blueprints
==========

post-receive git hook for AUR packages
--------------------------------------

Generate a bare repository on an Arch Linux instance.

Example for fpyutils


::

    #!/usr/bin/bash -l
    #
    # The MIT License (MIT)
    #
    # Copyright (c) 2008-present Tom Preston-Werner and Jekyll contributors
    #               2021 Franco Masotti
    #
    # Permission is hereby granted, free of charge, to any person obtaining a copy
    # of this software and associated documentation files (the "Software"), to deal
    # in the Software without restriction, including without limitation the rights
    # to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
    # copies of the Software, and to permit persons to whom the Software is
    # furnished to do so, subject to the following conditions:
    #
    # The above copyright notice and this permission notice shall be included in all
    # copies or substantial portions of the Software.
    #
    # THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
    # IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
    # FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
    # AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
    # LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
    # OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
    # SOFTWARE.

    set -euo pipefail

    # Edit these variables.
    TEST_COMMAND="python3 -c 'import fpyutils'"
    BASE_URL='https://blog.franco.net.eu.org/software'
    PROJECT='fpyutils'
    ARCHLINUX_PACKAGE='python-fpyutils'
    ORIGIN="user@domain:gituser/fpyutils.git"
    TAG="$(git tag | sort --human-numeric-sort --ignore-leading-blanks | tail --lines=1)"
    SHA512SUM="$(/usr/bin/curl ${BASE_URL}/${PROJECT}-${TAG}.tar.gz.SHA512SUM.txt | awk '{ print $1 }')"
    TMP_GIT_CLONE=""${HOME}"/tmp/${PROJECT}"

    # Abort if the tag is not present
    if [ -z "$(git tag | grep "${TAG}")" ]; then
    exit 0
    fi

    git clone "${GIT_DIR}" "${TMP_GIT_CLONE}"
    unset GIT_DIR

    # Create a disposable directory because we need to clean the temporary repository later.
    mkdir /dev/shm/"${PROJECT}"

    pushd "${TMP_GIT_CLONE}"/packages/aur

    cp -aR PKGBUILD /dev/shm/"${PROJECT}"

    pushd /dev/shm/"${PROJECT}"

    # Replace SKIP with SHA512SUM.
    sed -i "s/sha512sums=('SKIP' 'SKIP'/sha512sums=('SKIP' '${SHA512SUM}'/" PKGBUILD

    # Build the package.
    makepkg -rsi --noconfirm

    # Run a command to test the package.
    pushd ~
    eval ${TEST_COMMAND}
    popd

    # Print package information and then remove all traces of it.
    pacman -Qi "${ARCHLINUX_PACKAGE}"
    sudo pacman -Rnus --noconfirm "${ARCHLINUX_PACKAGE}"
    rm -rf pkg src *.tar.*

    makepkg --printsrcinfo > .SRCINFO

    popd
    popd

    pushd "${TMP_GIT_CLONE}"

    # Push updated pkgbuild to an empty disposable git branch.
    # Put PKGBUILD and .SRCINFO
    git checkout --orphan packages-aur
    git rm -rf .
    mv /dev/shm/"${PROJECT}"/{PKGBUILD,.SRCINFO} .
    git add PKGBUILD .SRCINFO
    git commit -m "Updated PKGBUILD and .SRCINFO."

    # Update remotes.
    git remote add repo "${ORIGIN}"

    # Remove remote branch.
    git push repo --delete packages-aur || echo OK

    # Push the files.
    git push --set-upstream repo packages-aur

    popd

    rm --recursive --force "${TMP_GIT_CLONE}" /dev/shm/"${PROJECT}"


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
