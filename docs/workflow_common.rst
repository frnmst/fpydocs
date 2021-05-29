Workflow [common]
=================

Rules
-----

- Assume that the root of the repository is ``./``.
- 3.5 <= Python version < 4
- Variables are marked with braces and the dollar sign, e.g: ``${variable}``.

Variable reference
``````````````````

+--------------------------------+---------------------------------------------------------------------------------------------------------------+
| Variable name                  | Description                                                                                                   |
+================================+===============================================================================================================+
| ``project_name``               | the name of the project                                                                                       |
+--------------------------------+---------------------------------------------------------------------------------------------------------------+
| ``project_directory``          | usually equal to ``project_name``                                                                             |
+--------------------------------+---------------------------------------------------------------------------------------------------------------+
| ``projects_aur_git_directory`` | the directory of the corresponding `AUR <https://wiki.archlinux.org/index.php/Arch_User_Repository>`_ package |
+--------------------------------+---------------------------------------------------------------------------------------------------------------+
| ``gpg_signing_key``            | a valid PGP key used to sign commits                                                                          |
+--------------------------------+---------------------------------------------------------------------------------------------------------------+
| ``dev_branch``                 | development git branch                                                                                        |
+--------------------------------+---------------------------------------------------------------------------------------------------------------+
| ``version_id``                 | the version identifier used in the git tag                                                                    |
+--------------------------------+---------------------------------------------------------------------------------------------------------------+
| ``${MAJOR}``                   | a variable of the  `Semantic Versioning <https://semver.org/#summary>`_ document                              |
+--------------------------------+---------------------------------------------------------------------------------------------------------------+
| ``${MINOR}``                   | a variable of the  `Semantic Versioning <https://semver.org/#summary>`_ document                              |
+--------------------------------+---------------------------------------------------------------------------------------------------------------+
| ``${PATCH}``                   | a variable of the  `Semantic Versioning <https://semver.org/#summary>`_ document                              |
+--------------------------------+---------------------------------------------------------------------------------------------------------------+

Sequence
--------

Follow these instructions in sequential order.

1. preliminary configuration
````````````````````````````

- IF not enabled previously setup git signing

 -

    ::

        git config commit.gpgsign true

 -

    ::

        git config user.signingkey ${gpg_signing_key}


- IF not enabled previously setup a bare git repository serving
  static pages. `Follow this blog post <https://blog.franco.net.eu.org/notes/an-alternative-to-github-pages.html>`_

- IF `AUR <https://wiki.archlinux.org/index.php/Arch_User_Repository>`_ setup a ``post-receive``
  git hook in a bare repository. See `this <https://blog.franco.net.eu.org/notes/an-alternative-to-github-pages.html>`_
  and use the provided "post-receive git hook for AUR packages" blueprint:

 -

    ::

        git init --bare ${project_name}


 - copy the blueprint into repo/hooks and set it as executable

 - add the ``pacman`` command to the ones not requiring password in the sudoers file, using ``visudo``:


   ::


        build-user ALL=NOPASSWD: /bin/pacman


2. finish working on the development branch
```````````````````````````````````````````

-

 ::

     cd ${project_directory}

- check that the current branch is not ``master``

- IF needed, create a new `asciinema <https://asciinema.org/>`_ demo file and upload it

 -

    ::

        cd ./asciinema

 - IF there have significant been changes from the previous version

  - modify the ``${project_name}_asciinema_${MAJOR}_${MINOR}_${PATCH}_demo.sh`` file accordingly

  -

    ::

        pipenv run asciinema rec --command=./${project_name}_asciinema_${MAJOR}_${MINOR}_${PATCH}_demo.sh ${project_name}_asciinema_${MAJOR}_${MINOR}_${PATCH}.json

  -

    ::

        pipenv run asciinema play ${project_name}_asciinema_${MAJOR}_${MINOR}_${PATCH}.json

  -

    ::

        pipenv run asciinema upload ${project_name}_asciinema_${MAJOR}_${MINOR}_${PATCH}.json

  - edit the ``./README.rst`` file with the new asciinema link

 - ELSE

  -

    ::

        ln -s ${project_name}_asciinema_${MAJOR}_${OLD_MINOR}_${OLD_PATCH}_demo.sh ${project_name}_asciinema_${MAJOR}_${MINOR}_${PATCH}_demo.sh

  -

    ::

        ln -s ${project_name}_asciinema_${MAJOR}_${OLD_MINOR}_${OLD_PATCH}.json ${project_name}_asciinema_${MAJOR}_${MINOR}_${PATCH}.json


-

  ::

      git add -A

-

  ::

      git commit -m "An interesing message."

-

  ::

      git push

3. update version numbers, requirements, etc...
```````````````````````````````````````````````

-  FOREACH file update version numbers:

 - ``./setup.py``

 - ``./docs/conf.py``

 - all downstream distribution packages (see the ``./packages`` directory)

- IF there have been dependencies updates update the ``./Pipfile``

-

  ::

      make install-dev

-

  ::

      make doc

-

  ::

      make test

-

  ::

      make install

-

  ::

      cd ~ && python3 -c 'import ${package_name}' && cd ${OLDPWD}

-

  ::

      make uninstall

-

  ::

      make clean

- FOREACH changed file update copyright years, emails and contributors:

 - ``./README.rst``

 - ``./docs/conf.py``

 - ``./docs/copyright_license.rst``

 - all Python source files

 - all downstream distribution packages (see the ``./packages`` directory)

-

  ::

      git add -A

-

  ::

      git commit -m "Preparing for new release."

-

  ::

      git push

4. update the documentation
```````````````````````````

-

  ::

      make clean && make doc

-

  ::

      rm -rf ~/html && cp -aR docs/_build/html ~

-

  ::

      git branch --delete --force docs

-

  ::

      git checkout --orphan docs

-

  ::

      git rm --cached -r .

-

  ::

      git clean -f && git clean -f -d

-

  ::

      mv ~/html/{*,.nojekyll,.buildinfo} .

-

  ::

      git add -A

-

  ::

      git commit --no-verify -m "New release."

-

  ::

      git push --force deploy-docs docs


5. merge the branches and tag the release
`````````````````````````````````````````

-

  ::

      git checkout master

-

  ::

      git merge ${dev_branch}

-

  ::

      git tag -s -a ${version_id} -m "Some sensible comments highlighting relevant changes from the previous release."

-

  ::

      git push

-

  ::

      git push origin ${version_id}


6. upload the package to PyPI
`````````````````````````````

- IF the package is present on PyPI:

 -

   ::

       make clean

 -

    ::

       make dist

 -

    ::

       make upload


7. upload the package on the software page
``````````````````````````````````````````

- follow the instructions reported `here <https://blog.franco.net.eu.org/software/#upload>`_

8. update downstream distribution packages
``````````````````````````````````````````

- IF `AUR <https://wiki.archlinux.org/index.php/Arch_User_Repository>`_:

 -

    ::

        git push packages-aur

 -

    ::

        git push --tags packages-aur

 -

    ::

        make clean && git fetch --all && git checkout packages-aur

 -

    ::

        cp PKGBUILD .SRCINFO ${projects_aur_git_directory}

 -

    ::

        git checkout master

 -

    ::

        cd ~/${projects_aur_git_directory}

-

    ::

        git add PKGBUILD .SRCINFO

 -

    ::

        git commit -m "New release."

 -

    ::

        git push


 - update the `frnmst-aur-packages-mirror <https://github.com/frnmst/frnmst-aur-packages-mirror>`_ repository

9. other
````````

- IF needed update the entry on the `Free Software Directory <https://directory.fsf.org/wiki/Main_Page>`_
