# BitKeeper version 7.3.3 released Dec 29 2018

A bugfix release, but fixes build issues on current systems

- This release fixes some problems where bk fails to build correctly
  with newer versions of glibc.  (conflict with symbol FILE) And it
  needed fixes to work with gperf v3

- 'bk relink' could assert in a certain corner case
  `bk: remap.c:119: remap_lstat: Assertion sccs failed.`

- Fix a bug where 'bk cp' didn't work correctly when run from outside
  of a the repository containing the file being copied.

-  Fix 'bk changes --html' to handle quoting in comments.
   Also fix html tag order

-  difftool would use the wrong timestamp when extracting data from bk
   history. This used to work in the past but was broken with a
   different bug was fixed.

- Fixed a bug where 'bk collapse' could corrupt a repository by
  removing some required tags.

***

# BitKeeper version 7.3.2 released Sep 23 2017

A small collection of bugfixes

- Improve the error message when bk fails to allocate a temp file
  https://users.bitkeeper.org/t/read-only-file-system-causes-suboptimal-error-message/478

- Clear BK_CONFIG before running regressions
  https://users.bitkeeper.org/t/bk-config-checkout-get-makes-bk-fail-tests/563

- Relax the consistency tests for tags to allow strange constructs
  created by 15 year old versions of bk. These will still be rejected
  on a newer repository.

- In `bk changes URL`, fix problem with extremely long output lines
  not getting transmitted correctly.
  https://users.bitkeeper.org/t/bk-changes-path-stops-when-it-hits-a-maximally-long-comment/542

- Document 'bk export -a' that includes files from the `BitKeeper/`
  subdirectory and fix a problem where it didn't work with the
  `-tpatch` option.
  https://users.bitkeeper.org/t/bk-export-omits-changes-to-bitkeeper-etc-ignore/344

- Minor tweaks to fix testcases to continue working correctly

- bkd commands are now able to rotate logfiles to prevent
  BitKeeper/log from growing too large

- Testcase tweak to avoid occasional regression failures
  https://users.bitkeeper.org/t/occasional-regression-failure-upgrade-running-in-deleted-dir/577/2

- More documentation for 'bk changes --oneline/--short'

***


# BitKeeper version 7.3.1ce released Sep 29 2016

A smaller collection of bugfixes and new features. This is mainly
being released because of a bug introducted in bk-7.3 that caused some
older repositories to stop working.

## New features

- Add a simple mechinism for users to add new commands to BitKeeper.
  If 'bk-foo' exists on the PATH, then 'bk foo' will call that command.
  (git already does something similar)
  If you want 'bk help foo' to work, you do that by making 'bk-foo --help'
  produce whatever help you want.  Your command must handle --help even
  if it just does nothing.  The reason: if your command is potentially
  destructive and it doesn't handle --help, it just ignores it, that
  could lead to a mess.

- The dspec expression `$first(:KEY:)` was introduced in bk-7.0 but with a
  different less useful syntax.  Now `$first()` has changed and is documented.

- The `bk send` and `bk receive` commands will now correctly package
  csets from a nested repositories that impact multiple
  components. With this release nested csets can be transferred like
  this:
  ```
bk send -rRANGE - | <send to new machine> | bk receive -
```

- The 'changes' command has new --short and --oneline command line
  options. See 'bk help changes' for details.

## Changes

- In the past, when running 'bk foo' where 'foo' is not a builtin
  BitKeeper command, the bk executable would try to execute 'foo' from
  the user's PATH. This could cause confusion and so this feature has
  been removed. Now and commands that are not understood by BitKeeper,
  or match the new bk-foo extention above, will print a command not
  found error.

- The dspec language used to extract metadata with 'bk changes' and
  'bk log' is now better documented in the 'bk help log' man page.

## Bugfixes

- In bk-7.3ce, a feature was removed where imports might put the user
  running the import after the hostname like this:
  `someone@company.com[me]`
  This change introduced a bug where old csets created with that
  syntax were not read correctly if the repository was also in the OLD
  bk-5.0 syntax.

- Fix a problem with incremental fast-export on nested repositories

- Lots of fixes to the Makefile's to support parallel builds.

- Pathnames in RCS and SCCS keywords (see `bk help keywords`) are now
  correctly expanded using historical pathnames when fetch old
  versions of files that have been renamed. This was preventing
  `bk export -tplain -rREV DIR` for generating a perfect replicate
  of older releases that contain keywords.

- In BK/Web when looking at files changes, binary files are not
  reported correctly.

---

# Release notes for BitKeeper version 7.3ce release July 13 2016

This release contains changes aimed at decoupling built-in packages
from BitKeeper to make it easier to include to system packages.  Since
some functionality changed, but nothing major was added the minor
version number was incremented.

## New features

- The 'bk fast-import' command was added to quickly import git
  repositories into bk. This is the initial release of a work in
  progress. The following features are not yet implemented
    - incremental import
    - git repos with submodules
    - octopus merges
    - 'git fast-export -M' support

- Add 'bk untag' command to delete tags and extend 'bk changes' output
  to show better information about tags.  BK has always version
  controlled tags and it is possible to have the same tag at multiple
  places in the history. This release adds the ability to delete tags
  (previously you could move them but there was no way to delete them).
  bk changes -t shows only the currently tagged changesets;
  bk changes -tt shows all changesets even if the tag is moved/deleted
  and annotates them with that information:
    ```
   ChangeSet@1.2678.1.218, 2015-07-15 15:09:24-07:00, wscott@work.bitkeeper.com
     Fix problem with t.status in tagged release
     TAG: bk-7.0 (currently on 1.2678.1.220)
```

## Changes

- Removed the out-of-date version of GNU diff from our tree.  The few
  places where BK assumed a GNU diff, it will now use the one on the
  system.  There is an exception for Windows. If no diff is found BK
  will use a 2.8.7 binary that we still ship (upgraded from 2.7).

  This is part of a larger effort to unbundle all of the OSS packages
  that are likely to already be installed (and more up-to-date).

- Rename the old "bk diffs" command to "bk diff".  "bk diffs" remains
  as an alias.  Scripts expecting on the "bk diff" to run a GNU diff
  should be updated to remove the "bk " prefix.

- Removed the out-of-date verison of GNU patch from our tree.
  The code will now use whatever version of patch the user has
  installed.
  The 'bk patch' code will also call the system's patch(1) but will
  first scan the patch and runs 'bk edit' on all pathnames found in
  the patch.
  As a side-effect of this change we removed the 'bk import -temail'.
  It was not used and was too dependent on the exact version of patch
  used.

- The libraries zlib, pcre, lz4, tomcrypt & tommath will now link to
  the system's copy instead of the using our included copies if a
  simplistic compile test works.  The src/mkconf.sh script can be
  tweaked if these libraries are found in non-standard locations on
  your system. When building from a tar.gz these libraries must be
  pre-installed.

- Update the documentation to describe 'bk get' instead of 'bk co'
  and 'bk delta' instead of 'bk ci'.  The get/delta verbs have become
  the names for these operations and the documentation should reflect
  that.   These commands were aliased before and after this release
  so no functionality has changed and no scripts should break.

- When overriding a csets username with BK_USER, bk would record the
  new user and system's username together as "user/realuser".  The
  code was changed to only display "user" in most cases. Visible
  changes:
    - annotations will only be username not user/realuser
    - changes -uUSER will now match just the user name
    - limiting by user in bkweb works
    - findkey can now search for user in the user/realuser case

- As part of the tag delete changes a number of dspecs were changed
  - The $each() function was normalized to strictly walk each line
    (separated by newlines) of a dspec for processing.  Previously
    `:DSPEC:` and `$each(:DSPEC:)` were separate and could do different
    things.
  - So the :C: macro was changed when not used in $each to return the
    comments with newlines.  Previous it would join multiple lines
    without adding spaces which isn't very useful.
  - The tag releated dspecs changed:
    (these are lists separated by newlines)

    | Keyword  | Meaning |
	| -------- | ------- |
    |:TAGGED:  |all tags on delta that are still active|
    |:TAGS:    |all tags on delta and inactive ones are annotated|
    |          |if they are deleted or moved.|
	|:ALL_TAGS:|Show all tags on delete active or not (old :TAG:)|

    Some aliases were added for compability:
	```
      :TAG: -> :TAGGED:
      :SYMBOL: -> :ALL_TAGS:
```
- Rework some of the build dependancies in our Makefile so it no
  longer assumes that 'bk' is installed and tries to use the installed
  bk when building BitKeeper.  Now the 'bk' binary is built first and
  that binary is used for constructing the manpages and GUIs.

Bugfixes

- Performance fixes for 'bk fast-export'.  For large repositories the
  new code is a couple orders of magnitude faster.

- Fix a couple more places where we were using the old dir/SCCS/s.file
  pathnames in error messages

- Customer reported that pull failed to delete parallel deletes of
  symlinks.

- Add DESTDIR support to makefike.

- Fix 'bk' binary so if it has a chain of symlinks to the final
  location the code can still find the install directory.  This
  simplifies packages for NixOS.

```

============================================================================
Release notes for BitKeeper version 7.2.1ce release May 16 2016

This is a minor bugfix release

- Portability fixes for Solaris-derived machines.
  We have added an OpenIndiana/hipster machine to our build cluster
  and ship binaries built on that machine.

- Fix the BK/Web web API to return a 500 error page when a REV URL parameter
  can't be found in the current repository.

- Fix 'bk version' to not do a synchronous fetch from bitkeeper.org to
  find the latest released version.  Now this fetch is done daily in
  the background to prevent pauses on machines with limited
  connectivity.

- Add support needed for a contributed git2bk importer

============================================================================
Release notes for BitKeeper version 7.2ce release May 9 2016

This is the first open source release of BitKeeper. Many changes involve
removing the licensing code.

Changes:

  Upgraded TCL/Tk to v8.6 which improved appearance on MacOS.

  BitKeeper now uses gfile pathnames in many places where sfile
  pathnames were used in the past. The biggest change is that 'bk -r'
  returns dir/file instead of dir/SCCS/s.file.  Also many error
  messages avoid printing SCCS/s.file pathnames since those are
  confusing.

  The code now uses PCRE for regex code everywhere. This mainly
  changes 'bk changes -/regex/' and 'bk log -/regex/'.

  Removed old commands:
    bk _eula
    bk lease
    bk legal
    bk more
    bk status --compat
    bk users

  Several performance issues related to repositories containing
  abnormally large numbers of tags have been fixed.

  The BK/Web service has been given significant improvements to upgrade
  the appearance from 1998 web standards. ;-)

Changes in individual commands

  bk changes
    New --lattice && --longest options to change selections based on
    the input range.
    New --json option to format output for web apps
  
  bk delta
    if the last line has no newline at the end and has carriage
    returns, they will now be stripped.  If no data is left, then the
    line is ignored.

    The -D option is now limited to RCS diffs.  RCS diffs used to not
    recognize no newline diffs.  They are recognized now.

  bk describe
     New command to give a friendly name for the current cset like
     bk-7.1+183@0x56d9f3c0 (last tag + # of csets since @ time_t of tip)
     # Show the tip (and any other csets with that time stamp)
     bk changes -c0x56d9f3c0

  bk diffs
    -h no longer prints a blank first line.

  bk fast-export
    Add --standalone mode for exporting single components from a
    nested collection.
    Add support for incremental exports to allow a git repository to
    track a bk master.

  bk rmgone
    Rmgone was completely rewritten and removed a number of bugs and
    shortcomings.  The new code is nested aware, has a more
    understandable output, and won't incorrectly remove some files.

  bk makepatch
    Makepatch now ONLY generates the bk-5.0 fastpatch format and
    later. This breaks pull/push compatibility with bk-4.x, but old
    repositories can still be cloned and read.  bk clone --upgrade
    is your friend.

  bk upgrade
    Now downloads new versions from the www.bitkeeper.org/downloads area.
    This is also used to find the latest release as mentioned by
    'bk version'.

Bugs fixed:

  Fixed 'bk citool' support for pre-delta triggers. The old behavior
  was terrible. (sorry)

  'bk changes -R URL' with a bad URL could coredump.  Fixed.

  'bk chmod' could coredump when run outside a repository.

============================================================================
Release notes for BitKeeper version 7.1 (released Feb 24, 2016)

  See our Tips & FAQ page at:       http://www.bitkeeper.com/tips
  Follow us on Twitter:             https://twitter.com/BitKeeper

This release is identical to bk-7.0.3 with one added test to the
repository check for the problem introduced in bk-7.0 which bk-7.0.3
fixed.  Since this check could fail on existing repositories the
version was bumped to 7.1.

The most common forms of corruption from this problem will be
automatically and silently repaired by this release if the autofix
config is enabled (on by default).  Some more tricky forms are
possible, but we have never seen them in the wild. If these occur,
debugging information to send to support@bitkeeper.com will be
printed.

The recommendation is to test this release on a couple repositories
before deploying to the team at large.

============================================================================
Release notes for BitKeeper version 7.0.3 (released Feb 17, 2016)

Bugs fixed:

 - Return codes fixed to return error, such as when bk check calls itself.
 - Fix clock_skew section of config-etc man page.  BUG-ID: 2015-12-11-001
 - Fix receive and takepatch pulling old style patches into bk-7 repos.
 - Fix bug where a post-incoming trigger on a push would eat the first
   character after @ in the trigger output.
 - Do not delete patch given by bk takepatch -f patch.  Abort and resolve did.
 - Fix parallel takepatch reading old style patches created by 'bk makepatch'.
 - Occasionally force a full repository check when the ChangeSet file
   heap has grown significantly and needs garbage collection.
 - Change makepatch to send using format of receiving repository.
   This reduces conversion data from accumulating in the receiving repo.

============================================================================
Release notes for BitKeeper version 7.0.2 (released Dec 9, 2015)

Bugs fixed:

 - bk commit --ci in a nested repository could miss components
   that have modified (but not checked in) files
 - fix coredump with bk diffs -w
 - fix locking issue when running 'bk cset -xREV' in component
 - bug related to 'bk diffs -N' when given symlink not under BitKeeper
   control
 - fix problem where citool could incorrectly strip blank lines from
   file comments

GUI Enhancement:

 - Add balloon popups to revtool showing revision info
   in the following contexts:
   - mouse hover over graph nodes
   - mouse hover over links in annotation view (press c)

============================================================================
Release notes for BitKeeper version 7.0.1 (released Sep 4, 2015)

Bugs fixed:

 - bk version would report an incorrect "Latest version" when a
   licenseurl proxy was being used.
 - bk status would fail when one of the parents was not readable.
 - bk parent would not correctly normalize a Windows (DOS) path.
 - Revision controlled symlinks between repositories could cause
   'bk sfiles' to fail.
 - Reinstate the bk sendbug and support commands (they were
   temporarily removed for internal reasons).
 - Prevent the text window size tooltips from appearing and
   obscuring the comment pane in citool.

Additions:

 - Add -S option to bk cset command. This allows users to constrain
   including or excluding a cset to the current component.
   For example, to exclude the top cset in component A, as opposed to
   the whole nested collection

     cd componentA; bk cset -S -x+

OS X changes:

 - Our minimum supported MacOS version is 10.9 (Mavericks). We do
   still build on 10.6 (Snow Leopard) and can supply those images on
   request.
 - Sign both the application binary and bundle so that the firewall
   would not keep asking if it was ok to accept incoming connections.
 - Stop putting a config file inside the bundle
   (/Applications/BitKeeper.app) and put them in ~/.bk/config
   instead. Typically, this config file would only contain the license
   keys.
 - When upgrading, transfer config contents from the bundle to the new
   location.

 - Stop using bk links during install.  Traditionally, bk links has
   been used to install a /usr/bin/bk symlink that points into the
   install location. This was handy since it allowed users to use bk
   immediately without modifying their PATH. Starting in MacOS 10.11
   (el Capitan), writing into /usr is disallowed.  Now the installer
   will use the path_helper(8) infrastructure to add bk to the
   PATH. Since path_helper is run by /etc/profile, users will only see
   the updated PATH in new shells.

   Emacs users may want to add something like this to their .emacs:

    ;; get the output of path_helper (which is meant to be interpreted
    ;; by a shell), remove the MANPATH setting if any, then capture
    ;; the PATH value and stuff that into the environment
    (let ((path (shell-command-to-string "/usr/libexec/path_helper")))
      (progn
	(setq path (replace-regexp-in-string "^MANPATH=.*$\n" "" path)
	      path (replace-regexp-in-string
		    "PATH=\"\\([^;]*\\)\".*\n" "\\1" path))
	(setenv "PATH" path)))

============================================================================
Release notes for BitKeeper version 7.0 (released Jul 16, 2015)

This version includes a significant internals rewrite for performance.
The on-disk repository format has changed.  You can take advantage of these
changes by replacing your existing repositories with upgraded clones:

     bk clone --parents -sTHERE --upgrade existing-repo new-repo

Implications of upgrading

    Upgrading some but not all of your repositories can make performance
    worse for some operations between repositories of different formats.
    The operations will still work.  There is a bk clone --downgrade that
    goes back to the old ASCII format (somewhat SCCS compatible) that all
    versions of BitKeeper support.

    Example:
    * You have a bkd running bk-7, and have upgraded all the repositories
      being served by the bkd.
    * You have a user running bk-6 who does a clone.

    The bkd will downgrade the repository before sending out clone data, and
    the user's bk-6 will upgrade the repository to the new format.

    If the user upgrades to the new repository format using bk-7, then neither
    of the transformations are done and the clone returns to running quickly.

    bk-6 will not be able to access new format repositories directly, but it
    can still access them by using a version 7.0 bkd.

Performance changes
-------------------

File format
    The ChangeSet file has been reworked for more performance.  Operations
    that search the history for particular files and/or deltas are significantly
    faster.  This benefits csettool, revtool, rset, and changes.
    bk rset is the utility command used by other commands to expand a
    changeset.  In a Linux kernel repo:

	bk-6.x rset -r+ takes 1243 milliseconds
	bk-7.0 rset -r+ takes   90 milliseconds

    Commit is also faster:

	bk-6.x commit takes 2213 milliseconds
	bk-7.0 commit takes  734 milliseconds

    The size of your history will determine how much the performance changes
    help you; they are geared towards larger repositories with more
    changesets and more files.

"checkout: edit" mode
    Previously, running in this mode impacted performance of commands
    such as 'bk citool'.
    Changes were made to how BK stores metadata about editable files
    to help it go faster.

    An implication of this is that user software that relies on
    detecting or examining paths in the form "SCCS/p.file" may not
    work as expected.  Contact support@bitkeeper.com if this is a problem.

Performance improvements on NFS
    We created a realistic repository simulator and generated a repository
    with one million changesets and 230,000 files (4GB of history).  We
    created identical BitKeeper and Git repositories.  We then benchmarked
    them on a pair of Intel i7-5930K 6 core/12 thread CPUs, 64GB of ram,
    rotating disk. The client file system was NFS version 3 with ext3 as the
    backing file system on the server. The benchmark was run on machines
    (client and server) with all of the data in cache (if we had done it
    cold cache Git looks much worse, Git is optimized for all your data in
    memory.)

    The results below speak for themselves.  Your results may vary, we have
    found that on smaller repos, hot cache and/or SSD, Git is comparable to
    BK except for things that search the history (bk annotate, bk grep) or
    the integrity check.  BK is much much faster when you are looking
    through the history and is much faster for an integrity check or garbage
    collection.  If you are using BK/Nested the performance wins are even
    larger.

    Truth in advertising: the diff changed files is only that fast if you
    run in checkout:get - that mode lets us remember each file you edited
    and not scan the whole tree.  That's why we suggest checkout:get for
    really large trees.

    What                      BK              GIT         How much faster is BK?
    ----------------------------------------------------------------------------
    simulate 1M csets        7.5 hours        165 hours          22 times faster
    clone                  210 seconds      338 seconds         1.6 times faster
    clone 1 component      6.5 seconds             N.A.                      N.A
    list changed files     0.3 seconds     40.6 seconds         135 times faster
    diff changed files     0.9 seconds     34.6 seconds          38 times faster
    commit                 4.8 seconds     42.5 seconds           9 times faster
    changes -R             0.6 seconds     21.6 seconds          36 times faster
    update-only pull       7.3 seconds     42.3 seconds           6 times faster
    log of 1 file         0.01 seconds      1.6 seconds         160 times faster
    annotate/blame        0.01 seconds     32.3 seconds        3230 times faster
    search history        0.01 seconds    138.9 seconds       13890 times faster
    integrity check      206.5 seconds   6576.2 seconds          32 times faster

New features
------------

bk annotate
    Added -w option to show "who deleted" annotations. 

bk bisect
    New command to help find a changeset that introduced a bug.

bk commit --ci
    Check in edited files as part of a commit. 

bk csets
    New option --stats that prints statistics about what was pulled.

bk -j<arg> cmd
    The -j option to 'bk' has changed.  In bk-6.x and earlier it
    would list 'junk' files (extra files in the SCCS directory). This
    is files in bk's internal storage space that were not written by bk.
    This was deemed not useful and so the command line option was reused
    for an internal option.

bk config <var>
    This used to show what <var> was set to in a config file.
    If <var> wasn't in any config file, then nothing would be shown.
    This now shows the default value for <var>.

bk config -v
    This command now shows the default values for all config variables.

    The following defaults have changed:
     autofix now defaults to "on"
     parallel now defaults to "on"

bk findmerge
    New command for showing which cset was the first to merge two
    other csets.

bk fm3tool
    Added "who deleted" annotation when you turn on annotations.
    Uses the "who deleted" information to show the delta comments
    for the delta that did the delete in the comments window (formerly it
    showed the delta that added the deleted line, which is far
    less useful).

bk partition
    New option --keep-deleted that does not prune out the deleted history
    as part of the partition.

bk pull
    New option --stats that prints statistics about what was pulled.
    It can also be permanently enabled by setting the
    config variable stats_after_pull to "on".

    Fix a bug where pull was not disallowing the -R option with
    multiple parents or URLs.

bk repocheck
    Now runs in parallel automatically when used on a nested collection.
    On a fast machine this can be about a 6x speedup (hot cache) and
    13x speedup (cold cache, SSD).

bk repos
    New command to show a list of your repositories.

bk revtool
    Added "who deleted" annotation.

bk rset
    Rewritten to be faster on traditional repositories, and massively
    faster on new format repositories.

bk setup
    No longer interactive by default. Running 'bk setup' will now turn
    the current working directory into a new BitKeeper repository.

bk status
    The old 'bk status' output wasn't very useful so the command was
    rewritten to provide more useful information about the current 
    repository status.
    See 'bk help status' for details.

bk tag
    The rules for valid tags names have been restricted to more normal
    looking identifiers to avoid compatibility issues in the future.
    See 'bk help tag' for the new rules.

    Existing tags that break these rules will continue to be supported
    but new tags need to follow the new rules.

bk takepatch
    An internal command used by pull.  Enhanced to do resolve automerging
    in cases where only content merging needs to be done. Enhanced with
    parallel operation, meaning each file-patch and automerge is handled
    in a sub-process.

Bugfixes
--------

- 'bk cset -x' works in a BK/Nested product.
- 'bk cp' now correctly handles BAM binary files.
- Several more tricky merge conflicts are now handled automatically.
- 'bk collapse' now preserves file deltas that exclude other deltas.
- 'bk lock -w' in non-nested was terminating before lock was fully released.
- Fix failures caused by extremely long filenames or extremely large
  repositories.
- 'bk rset -r$MERGE_CSET' now correctly lists merge revisions of files.

Platforms
---------
Minimum Windows version supported is Windows Vista.

We continue to support Linux, Windows, MacOS, Solaris, and several
BSDs on x86.  We can add back in any other major platform if needed,
just let us know.
```
