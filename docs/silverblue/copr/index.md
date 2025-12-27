# COPR

Provides RPM-builder and repositories to the community.

See the offical [documentation](https://docs.pagure.org/copr.copr/user_documentation.html)

!!! Note

    This works fine. But only makes sense for packages with multiple clients.
    My custom iot packages are only used by me, so I prefer to just keep the RPMs locally installed.  
    I still build with Tito, but just the [Tagged Build](#tagged-build).

## Setup

Use toolbox `fedora`.

Create `~/.config/copr` with content from [here](https://copr.fedorainfracloud.org/api/).

Followed [guide](https://docs.pagure.org/copr.copr/screenshots_tutorial.html#screenshots-tutorial).

## Build Own Packages

Use Tito to build RPMs in a way that can result in an official COPR.

### Prep

In the repository, the file `.tito/tito.props` contains the Tito configuration:

```
[buildconfig]
builder = tito.builder.Builder
tagger = tito.tagger.VersionTagger
changelog_do_not_remove_cherrypick = 0
changelog_format = %s (%ae)
```

### Test Build

To build a local RPM, run something like this:

```console
$ distrobox enter fedora
$ cd ROOT OF REPOSITORY
$ # maybe validation spec: rpmlint PROJECT_NAME.spec
$ rm -rf /tmp/tito/x86_64/ ; tito build --rpm --test
```

This will build to `/tmp/tito/x86_x64` using an incremental version number (but based on the version in the spec-file).

### Tagged Build

In order to build an official version, tito needs some prepping:

Update spec version (--keep-version to keep manually maintained version in spec file):

```console
$ export EDITOR=vi
$ mkdir .tito/packages
$ tito tag --keep-version
```

This will result in a tag on the git repository, named 'PROJECT_NAME-SPECVERSION'.
And an updated file `.tito/packages/PROJECT_NAME`

*This tag* needs to be pushed to GitHub, for the next step to work.

Note that without this above, the build command will fail with the message:

```text
ERROR: Unable to lookup latest package info.
Perhaps you need to tag first?
```

You can now build a correctly tagged RPM package:

```console
$ rm -rf /tmp/tito/x86_64/ ; tito build --rpm
```

### COPR Build

Create new project, fill in:

 * Project Name
 * fedora-43-x86_64

Setup project build (from `fedora` container):

```console
$ copr-cli buildscm --clone-url https://github.com/jskov/REPOPATH.git --method tito jskov/PROJECT_NAME
```

This will add a package-entry and start a build (from the tag created above).


The result is an official repository for the RPM file which can be added to the box.
