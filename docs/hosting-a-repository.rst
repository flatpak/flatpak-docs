Hosting a repository
====================

.. note::

  Flathub uses flat-manager to host its Flatpak repository. See
  https://github.com/flatpak/flat-manager

The section on :doc:`flatpak-builder` describes how to generate
repositories. The resulting repository can be hosted on a web server for
consumption by users.

Important details
-----------------

Flatpak repositories use archive-z2, meaning that they contain a single file
for each file in the application. This means that pull operations involve
a lot of HTTP requests. Since new requests can be slow, it is important to
enable HTTP keep-alive on the web server that is hosting your repository.

Flatpak supports something called static deltas. These are single files that
contain all the data needed to go between two revisions (or from nothing to
a revision). Creating such deltas will take up more space on the server,
but will make downloads much faster. This can be done with the ``flatpak
build-update-repo --generate-static-deltas`` option.

.flatpakrepo files
------------------

``.flatpakrepo`` files are a convenient way to let users add a
repository. These are simple description files which contain information
about the repository. For example, the Flathub repo file looks like::

  [Flatpak Repo]
  Title=Flathub
  Url=https://dl.flathub.org/repo/
  Homepage=https://flathub.org/
  Comment=Central repository of Flatpak applications
  Description=Central repository of Flatpak applications
  Icon=https://dl.flathub.org/repo/logo.svg
  GPGKey=mQINBFlD2sABEADsiUZUO...

Here you can see that the repo file contains descriptive metadata, such as
the repository name, description, icon and website. The file also contains
information that is needed to add the repository, including a download URL
and the repository's GPG key.

``.flatpakrepo`` files can be used to add a repository from the command
line. For example, the command to add Flathub using its repo file is::

  $ flatpak remote-add --if-not-exists flathub https://dl.flathub.org/repo/flathub.flatpakrepo

The command line isn't the only way to add a repository using a
``.flatpakrepo`` file - on desktops that support Flatpak, it is just a matter
of clicking the repository file or a download link that points to it.

.. note::

  ``.flatpakrepo`` files should include the base64-encoded version of the
  GPG key that was used to sign the repository. This can be obtained with
  the following command::

  $ base64 --wrap=0 < key.gpg

Hosting a repository on Gitlab/Github pages
-------------------------------------------

A Flatpak repository can be easily hosted through Gitlab or Github pages
and distributed to users.

.. note::
  Github or Gitlab may have pipeline quotas, storage and bandwidth
  limits. Please consult their documentation on this.

On Gitlab
^^^^^^^^^

The instructions will use Gitlab.com.

1. Create a new blank repository on Gitlab

2. Clone the repository locally

.. code-block:: bash

  git clone git@gitlab.com:your_user_name/repo_name.git && cd repo_name

3. Create a ``.gitlab-ci.yml`` with the following contents.

.. note::
  A worflow that builds for aarch64 and x86_64 is provided
  below.

.. code-block:: yaml

  variables:
    # Application id of the app, should be same as id used in flatpak manifest and MetaInfo
    APP_ID: tld.vendor.app_name
    # Location of the flatpak manifest, root of git repository
    MANIFEST_PATH: $CI_PROJECT_DIR/${APP_ID}.yaml
    # Name of flatpak bundle
    BUNDLE: "${APP_ID}.flatpak"
    # Docker image to use
    DOCKER_IMAGE: "ghcr.io/flathub-infra/flatpak-github-actions:freedesktop-25.08"
    SCHEDULE_TASK: default

  stages:
    - setup
    - build
    - deploy

  # This will check for updates using external data checker and send PRs to the repo
  update-sources:
    stage: setup
    image:
      # https://github.com/flathub-infra/flatpak-external-data-checker
      name: ghcr.io/flathub/flatpak-external-data-checker
      # Open shell rather than the bin
      entrypoint: [""]
    before_script:
      - git config --global user.name "${GITLAB_USER_LOGIN}"
      - git config --global user.email "${GITLAB_USER_EMAIL}"
    script:
      - /app/flatpak-external-data-checker --update --commit-only $MANIFEST_PATH

      # Creates a merge request targetting the default repo branch and sets up auto merge when pipeline succeeds
      - git push -o merge_request.create -o merge_request.target=${CI_DEFAULT_BRANCH} -o merge_request.merge_when_pipeline_succeeds
        "https://${GITLAB_USER_NAME}:${CI_GIT_TOKEN}@${CI_REPOSITORY_URL#*@}" || true
    artifacts:
      paths:
        - $MANIFEST_PATH
      expire_in: 1 week
    rules:
      # Set up a pipeline schedule for this https://docs.gitlab.com/ee/ci/pipelines/schedules.html
      - if: $CI_PIPELINE_SOURCE == "schedule" || $CI_PIPELINE_SOURCE == "trigger"
        when: always
      - when: never

  flatpak:
    stage: build
    image: ${DOCKER_IMAGE}
    variables:
      # Stable Flathub repo
      RUNTIME_REPO: "https://flathub.org/repo/flathub.flatpakrepo"
    script:
      # Set up an user as the docker image used here comes with none
      - |
        cat <<EOF > /etc/passwd
        root:x:0:0:root:/root:/bin/bash
        EOF

        cat <<EOF > /etc/group
        root:x:0:
        EOF

      # Sets up the stable Flathub repository for dependencies
      - flatpak remote-add --user --if-not-exists flathub ${RUNTIME_REPO}
      # Sets up GPG signing
      # Initialise GPG
      - gpg --list-keys --with-keygrip
      - echo "allow-preset-passphrase" >> ~/.gnupg/gpg-agent.conf
      - gpg-connect-agent reloadagent /bye
      - cat $GPG_PASSPHRASE | /usr/libexec/gpg-preset-passphrase --preset $GPG_KEY_GREP
      - gpg --import --batch ${GPG_PRIVATE_KEY}
      # Build & install build dependencies, branch can be unset too then it will default to master see man flatpak-manifest > branch
      - flatpak-builder build --user --install-deps-from=flathub --gpg-sign=${GPG_KEY_ID} --disable-rofiles-fuse --disable-updates --force-clean --repo=repo ${BRANCH:+--default-branch=$BRANCH} ${MANIFEST_PATH}
      # Generate a Flatpak bundle for testing in MRs
      - flatpak build-bundle --gpg-sign=${GPG_KEY_ID} repo ${BUNDLE} --runtime-repo=${RUNTIME_REPO} ${APP_ID} ${BRANCH}
      # flatpak-builder exports contents to repo folder after building
      # This generates summary, appstream refs and prunes the folder to keep the latest commit only in preparation for publishing
      - flatpak build-update-repo --gpg-sign=${GPG_KEY_ID} --generate-static-deltas --prune repo/
    artifacts:
      paths:
        - $BUNDLE
        # This artifact will be used when publishing to the static site
        - repo
      expire_in: 1 week
    rules:
      - if: $CI_PIPELINE_SOURCE == "schedule"
        when: never
      - if: $CI_COMMIT_BRANCH == $CI_DEFAULT_BRANCH
        when: on_success
      - if: $CI_COMMIT_BRANCH != $CI_DEFAULT_BRANCH
        when: manual

  # Deploys the generated package to Gitlab pages name.gitlab.io/repo_name
  pages:
    variables:
      BUILD_OUTPUT_PATH: ${CI_PROJECT_DIR}/repo
    stage: deploy
    image: alpine:latest
    before_script:
      - apk add rsync
      # replace html assets relative path with pages absolute path
      - find $BUILD_OUTPUT_PATH \( -type d -name .git -prune \) -o -type f -print0 | xargs -0 sed -i -e "s#href=\"\/#href=\"$CI_PAGES_URL/#g" -e "s#src=\"\/#src=\"$CI_PAGES_URL/#g"
    script:
      - mkdir public || true
      - rsync -av --exclude='public' --exclude='.git' $BUILD_OUTPUT_PATH/ public
    artifacts:
      paths:
        - public
      expire_in: 1 week
    rules:
      - if: $CI_PIPELINE_SOURCE == "schedule"
        when: never
      - if: $CI_COMMIT_BRANCH == $CI_DEFAULT_BRANCH
        when: on_success

4. `Create <https://www.gnupg.org/gph/en/manual/c14.html>`_ a new GPG key
   locally, to sign the repository.

5. Go to ``https://gitlab.com/-/profile/personal_access_tokens`` and create
   a token for ``$CI_GIT_TOKEN``. Note that the token is valid for a
   maximum of one year and you should renew it before it expires.

6. Go to ``https://gitlab.com/your_user_name/repo_name/-/settings/ci_cd``.
   Expand `General` and disable public pipeline. Click Save.
   Expand `variables`. Add the following
   `variables <https://docs.gitlab.com/ci/variables/#define-a-cicd-variable-in-the-gitlab-ciyml-file>`_
   necessary for the pipeline to run:

.. list-table::
   :widths: 15 20 25 15 15
   :header-rows: 1

   * - Type
     - Key
     - Value
     - Protected
     - Masked
   * - Variable
     - GPG_KEY_GREP
     - Keygrip of GPG key
     - Yes
     - Optional
   * - Variable
     - GPG_KEY_ID
     - Keyid of GPG key
     - Yes
     - Optional
   * - File
     - GPG_PASSPHRASE
     - Passphrase of GPG Key
     - Yes
     - Optional
   * - File
     - GPG_PRIVATE_KEY
     - ASCII armoured private key
     - Yes
     - Optional
   * - Variable
     - CI_GIT_TOKEN
     - Token
     - Yes
     - Optional

To get the keygrip of the GPG key generated in step 4, run the
following in your terminal and look at the ``Keygrip`` section:

.. code-block:: bash

  gpg --list-secret-keys --with-keygrip

To find the keyid of the GPG key run the following in the terminal. The
keyid should be in the first line starting with ``sec`` and
``algorithm/id``. The ``id`` part is the required keyid.

.. code-block:: bash

  gpg --list-secret-keys --keyid-format=long

The following will generate an ASCII armoured private key. Then paste
the contents of that file in the CI variable settings.

.. code-block:: bash

  gpg --output private.pgp --armor --export-secret-key <keyid or email>

7. Create a ``app_name.flatpakref`` in the root of the git repo with
   the following contents.

.. parsed-literal::

  [Flatpak Ref]
  Title=<A pretty application or repo name>
  Name=<Application id in tld.vendor.app_name format>
  Branch=< branch of generated ostree refs, defaults to master>
  Url=<Url of Gitlab page>
  SuggestRemoteName=<A name for the flatpak remote>
  Homepage=<URL of the homepage>
  Icon=<Direct link to an icon>
  RuntimeRepo=< Link to repo where runtime and other dependencies are eg. https://dl.flathub.org/repo/flathub.flatpakrepo>
  IsRuntime=false
  GPGKey=<base64 encoded GPG key>

You can find the Gitlab page in
``https://gitlab.com/your_user_name/repo_name/pages``. Disable
`Use unique domain` there and hit save. To generate the base64
encoded ``GPGKey``, run the following and paste the string:

.. code-block:: bash

  gpg --export <keyid> > example.gpg
  base64 example.gpg | tr -d '\n'

8. The root of the repository should contain the following
   files: ``.gitlab-ci.yml``, ``app_name.flatpakref``, the flatpak manifest
   ``tld.vendor.app_name.yaml`` and any other files/folders referenced
   in the manifest. ``git add`` these files, ``git commit`` and
   ``git push``.

9. If everything was set up correctly, the push will trigger the
   pipeline to build and deploy your application with flatpak.

10. To install the build, you can run:

.. code-block:: bash

  flatpak install --user https://gitlab.com/your_user_name/repo_name/-/raw/branch/app_name.flatpakref

This will set up a flatpak remote userwide, install the dependencies and
the application. Updates will be fetched when running ``flatpak update``
if they are available.

11. You can set up a `pipeline schedule <https://docs.gitlab.com/ci/pipelines/schedules/>`_,
    optionally to automatically check for updates using
    `flatpak-x-checker <https://github.com/flathub-infra/flatpak-external-data-checker>`_
    and send PRs to the repo.


Multi-architecture workflow
^^^^^^^^^^^^^^^^^^^^^^^^^^^

This uses Gitlab.com's `hosted aarch64 runners <https://docs.gitlab.com/ci/runners/hosted_runners/linux/#machine-types-available-for-linux---arm64>`_ for building on aarch64.

.. code-block:: yaml

  variables:
    MANIFEST_PATH: $CI_PROJECT_DIR/${APP_ID}.yaml
    DOCKER_IMAGE: "ghcr.io/flathub-infra/flatpak-github-actions:freedesktop-23.08"
    SCHEDULE_TASK: default

  stages:
    - setup
    - build-x86_64
    - build-aarch64
    - update-repo
    - deploy

  .setup:
    stage: setup
    image: ${DOCKER_IMAGE}
    variables:
      RUNTIME_REPO: "https://flathub.org/repo/flathub.flatpakrepo"
    before_script:
      # Set up an user as the docker image used here comes with none
      - |
        cat <<EOF > /etc/passwd
        root:x:0:0:root:/root:/bin/bash
        EOF

        cat <<EOF > /etc/group
        root:x:0:
        EOF

      # Add the flathub repository for installing build dependencies
      - flatpak remote-add --user --if-not-exists flathub ${RUNTIME_REPO}
      - gpg --list-keys --with-keygrip
      - echo "allow-preset-passphrase" >> ~/.gnupg/gpg-agent.conf
      - gpg-connect-agent reloadagent /bye
      - cat $GPG_PASSPHRASE | /usr/libexec/gpg-preset-passphrase --preset $GPG_KEY_GREP
      - gpg --import --batch ${GPG_PRIVATE_KEY}
    rules:
      - if: $CI_PIPELINE_SOURCE == "schedule"
        when: never
      - if: $CI_COMMIT_BRANCH == $CI_DEFAULT_BRANCH
        when: on_success
      - if: $CI_COMMIT_BRANCH != $CI_DEFAULT_BRANCH
        when: manual
    artifacts:
      paths:
        - repo
      expire_in: 1 week

  build-x86_64:
    variables:
      ARCH: x86_64
    extends: .setup
    script:
      # Build the app, ARCH should be host arch, BRANCH can be specified or if not it will default to master, see man flatpak-manifest > branch
      - flatpak-builder build --arch=${ARCH} --user --install-deps-from=flathub --gpg-sign=${GPG_KEY_ID} --disable-rofiles-fuse --disable-updates --force-clean --repo=repo ${BRANCH:+--default-branch=$BRANCH} ${MANIFEST_PATH}
    stage: build-x86_64

  build-aarch64:
    variables:
      ARCH: aarch64
    extends: .setup
    script:
      - flatpak-builder build --arch=${ARCH} --user --install-deps-from=flathub --gpg-sign=${GPG_KEY_ID} --disable-rofiles-fuse --disable-updates --force-clean --repo=repo ${BRANCH:+--default-branch=$BRANCH} ${MANIFEST_PATH}
    stage: build-aarch64
    # https://docs.gitlab.com/ee/ci/runners/hosted_runners/linux.html#machine-types-available-for-linux---arm64
    tags:
      - saas-linux-large-arm64
    dependencies:
      - "build-x86_64"

  update-repo:
    stage: update-repo
    image: ${DOCKER_IMAGE}
    dependencies:
      - "build-aarch64"
    extends: .setup
    script:
      # The repo folder must have contents for both arches present, so they are chained one after another through dependencies
      # prune is run to keep the latest commit only
      - flatpak build-update-repo --gpg-sign=${GPG_KEY_ID} --generate-static-deltas --prune repo

  pages:
    variables:
      BUILD_OUTPUT_PATH: ${CI_PROJECT_DIR}/repo
    stage: deploy
    image: alpine:latest
    dependencies:
      - "update-repo"
    script:
      - apk add rsync
      - find $BUILD_OUTPUT_PATH \( -type d -name .git -prune \) -o -type f -print0 | xargs -0 sed -i -e "s#href=\"\/#href=\"$CI_PAGES_URL/#g" -e "s#src=\"\/#src=\"$CI_PAGES_URL/#g"
      - mkdir public || true
      - rsync -av --exclude='public' --exclude='.git' $BUILD_OUTPUT_PATH/ public
    artifacts:
      paths:
        - public
      expire_in: 1 week
    rules:
      - if: $CI_PIPELINE_SOURCE == "schedule"
        when: never
      - if: $CI_COMMIT_BRANCH == $CI_DEFAULT_BRANCH
        when: on_success

Credits
^^^^^^^
The CI template is based on the `work <https://gitlab.com/accessable-net/gitlab-ci-templates>`_
of Flatpak community member
`proletarius101 <https://gitlab.com/proletarius101>`_.
