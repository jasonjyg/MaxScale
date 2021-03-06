# Release process

## Pre-release Checklist

* Make sure all bugs that have been fixed are also closed on Jira and have the
  correct fixVersion

For major releases:

* Create new release notes and add all fixed bugs, use a previous one as a
  template

For bug fix releases:

* Run the `Documentation/Release-Notes/generate_release_notes.sh` script to
  auto-generate release notes

Finally:

* Add link to release notes and document major changes in Changelog.md

## 1. Tag

Release builds are always made using a tag and a separate branch. However, the
used tag is a _tentative_ tag, to ensure that there never is a need to _move_
any tag, should the release have to be modified after it has been tagged. All
that is needed is to create a new tentative tag.

The source for release `x.y.z` is tagged with `maxscale-x.y.z-ttN` where `N` is
1 for the first attempt and incremented in case the `x.y.z` source still needs
to be modified and the release rebuilt.

The final tag `maxscale-x.y.z` is created _after_ the packages have been
published and we are certain they will not be rebuilt, without also the version
number being changed.

To create the tag and the branch from the main _x.y_ branch:

```
git checkout x.y
git checkout -b x.y.z
git tag maxscale-x.y.z-ttN
git push -u origin x.y.z
git push origin refs/tags/maxscale-x.y.z-ttN
```

**A note on fixing bugs while doing a release:**

A separate branch is used to guarantee that no commits are added once the
release proceedings have started. If any fixes to code or documentation need to
be done, do them on the _x.y.z_ branch. If a fix has been made, create a new tag
by incrementing the `-ttN` suffix by one and push both the branch and the new
tag to the repo.

**NOTE** The tentative suffix - `-ttN` - is **only** used when
specifying the tag for the build, it must **not** be present in
any other names or paths.

## 2. Build and upgrade test

The BuildBot [build_for_release](https://maxscale-ci.mariadb.com/#/builders/38)
job should be used for building the packages. Use your GitHub account to log in
to actually see the job. Click the blue _Build for release_ button in the top
right corner to start it.

### Parameters to define

#### `branch`

This is the tag that is used to build the release.

```
refs/tags/maxscale-x.y.z-ttN
```

#### `The version number of this release in x.y.z format`

The version number of this release in x.y.z format. This will create two packages; maxscale-x.y.z-release and maxscale-x.y.z-debug.

```
x.y.z
```

#### `Old target`

The previous released version, used by upgrade tests. Set it to the previous
release e.g. for 2.2.19 set it to 2.2.18. For GA releases, set it to the latest
release of the previous major release e.g. for 2.3.0 set it to 2.2.19.

```
x.y.z
```

## 3. Copying to code.mariadb.com

ssh to `code.mariadb.com` with your LDAP credentials.

Create directories and copy repositories files. Replace `x.y.z` with the
correct version.

```bash
cd  /home/mariadb-repos/mariadb-maxscale/
mkdir x.y.z
mkdir x.y.z-debug
cd x.y.z
rsync -avz  --progress --delete -e ssh vagrant@max-tst-01.mariadb.com:/home/vagrant/repository/maxscale-x.y.z-release/mariadb-maxscale/ .
cd ../x.y.z-debug
rsync -avz  --progress --delete -e ssh vagrant@max-tst-01.mariadb.com:/home/vagrant/repository/maxscale-x.y.z-debug/mariadb-maxscale/ .
```

Once the code has been uploaded, update the symlink for the current major
release.

```bash
cd  /home/mariadb-repos/mariadb-maxscale/
rm x.y
ln -s x.y.z x.y
```

If this is the GA release of a new major version, update the `latest` symlink to
point to `x.y`.

## 4. Email webops-requests@mariadb.com

Email example:

Subject: `MaxScale x.y.z release`

```
Hi,

Please publish Maxscale x.y.z binaries on web page.

Repos are on code.mariadb.com at /home/mariadb-repos/mariadb-maxscale/x.y.z

Br,
  YOUR NAME HERE
```

Replace `x.y.z` with the correct version.

**NOTE** Sometimes - especially at _big_ releases when the exact release
date is fixed in advance - the following steps 5, 6 and 7 are done right
after the packages have been uploaded, to ensure that the steps 4 and 8
can be done at the same time.

## 5. Create the final tag

Once the packages have been made available for download, create
the final tag

```bash
git checkout maxscale-x.y.z-ttN
git tag -a -m "Tag for MaxScale x.y.z" maxscale-x.y.z
git push origin refs/tags/maxscale-x.y.z
```

and remove the tentative tag(s)

```bash
git tag -d maxscale-x.y.z-ttN
git push origin :refs/tags/maxscale-x.y.z-ttN
```

## 7. Update the release date

Once the branch `x.y.z` has been created and the actual release
date of the release is known, update the release date in the
release notes.

```bash
git checkout x.y.z
# Update release date in .../MaxScale-x.y.z-Release-Notes.md
git add .../MaxScale-x.y.z-Release-Notes.md
git commit -m "Update release date"
git push origin x.y.z
```

**NOTE** The `maxscale-x.y.z` tag is **not** moved. That is, the
release date is _not_ available in the documentation marked with
the tag `maxscale-x.y.z` but _is_ available in the branch marked
with `x.y.z`.

Merge `x.y.z` to `x.y`.

```
git checkout x.y
git merge x.y.z
```

At this point, the last commits on branches `x.y` and `x.y.z`
should be the same and the commit should be the one containing the
update of the release date. Further, that commit should be the only
difference between the branches and the tag `maxscale-x.y.z`.
**Check that indeed is the case**.

## 8. Update documentation

Email webops-requests@mariadb.com with a mail containing the following. Replace
`x.y.z` with the correct version, also in the links.

Subject: `Please update MaxScale x.y knowledge base`

```
Hi,

Please update https://mariadb.com/kb/en/mariadb-enterprise/mariadb-maxscale-XY/

from https://github.com/mariadb-corporation/MaxScale/tree/x.y.z/Documentation

Br,
  YOUR NAME HERE
```

## 9. Send release email to mailing list

After the KB has been updated and the binaries are visible on the downloads
page, email maxscale@googlegroups.com with a mail containing the
following. Replace `x.y.z` with the correct version.

Subject: `MariaDB MaxScale x.y.z available for download`

```
Hi,

We are happy to announce that MariaDB MaxScale x.y.z GA is now available for download. This is a bugfix release.

The Jira list of fixed issues can be found here(ADD LINK HERE).

* [MXS-XYZ] BUG FIX DESCRIPTION HERE

Binaries:
https://mariadb.com/downloads/mariadb-tx/maxscale

Documentation:
https://mariadb.com/kb/en/mariadb-enterprise/maxscale/

Release notes:
KB LINK TO RELEASE NOTES HERE

Source code:
https://github.com/mariadb-corporation/MaxScale/releases/tag/maxscale-x.y.z

Please report any issues on Jira:
https://jira.mariadb.org/projects/MXS/issues

On behalf of the entire MaxScale team,

YOUR NAME HERE
```
