### 22.5.2. Preparing a patch series

Starting from the changes committed in your local git view, *rebase* your development branch on top of the upstream tree before generating a patch set. To do so, run:

```
$ git fetch --all --tags
$ git rebase origin/master
```

Now check the coding style for the changes you committed:

```
$ utils/docker-run make check-package
```

Now, you are ready to generate then submit your patch set.

To generate it, run:

```
$ git format-patch -M -n -s -o outgoing origin/master
```

This will generate patch files in the `outgoing` subdirectory, automatically adding the `Signed-off-by` line.

Once patch files are generated, you can review/edit the commit message before submitting them, using your favorite text editor.

Buildroot provides a handy tool to know to whom your patches should be sent, called `get-developers` (see [Chapter 23, *DEVELOPERS file and get-developers*](https://buildroot.org/downloads/manual/manual.html#DEVELOPERS) for more information). This tool reads your patches and outputs the appropriate `git send-email` command to use:

```
$ ./utils/get-developers outgoing/*
```

Use the output of `get-developers` to send your patches:

```
$ git send-email --to buildroot@buildroot.org --cc bob --cc alice outgoing/*
```

Alternatively, `get-developers -e` can be used directly with the `--cc-cmd` argument to `git send-email` to automatically CC the affected developers:

```
$ git send-email --to buildroot@buildroot.org \
      --cc-cmd './utils/get-developers -e' origin/master
```

`git` can be configured to automatically do this out of the box with:

```
$ git config sendemail.to buildroot@buildroot.org
$ git config sendemail.ccCmd "$(pwd)/utils/get-developers -e"
```

And then just do:

```
$ git send-email origin/master
```

Note that `git` should be configured to use your mail account. To configure `git`, see `man git-send-email` or https://git-send-email.io/.

If you do not use `git send-email`, make sure posted ***\*patches are not line-wrapped\****, otherwise they cannot easily be applied. In such a case, fix your e-mail client, or better yet, learn to use `git send-email`.

[https://sr.ht](https://sr.ht/) also has a light-weight UI for [preparing patchseries](https://man.sr.ht/git.sr.ht/#sending-patches-upstream) and can also send out the patches for you. There are a few drawbacks to this, as you cannot edit your patches' status in Patchwork and you currently can’t edit your display name with which the match emails are sent out but it is an option if you cannot get git send-email to work with your mail provider (i.e. O365); it shall not be considered the official way of sending patches, but just a fallback.

### 22.5.3. Cover letter

If you want to present the whole patch set in a separate mail, add `--cover-letter` to the `git format-patch` command (see `man git-format-patch` for further information). This will generate a template for an introduction e-mail to your patch series.

A *cover letter* may be useful to introduce the changes you propose in the following cases:

- large number of commits in the series;
- deep impact of the changes in the rest of the project;
- RFC [[4\]](https://buildroot.org/downloads/manual/manual.html#ftn.idm6107);
- whenever you feel it will help presenting your work, your choices, the review process, etc.

### 22.5.4. Patches for maintenance branches

When fixing bugs on a maintenance branch, bugs should be fixed on the master branch first. The commit log for such a patch may then contain a post-commit note specifying what branches are affected:

```
package/foo: fix stuff

Signed-off-by: Your Real Name <your@email.address>
---
Backport to: 2020.02.x, 2020.05.x
(2020.08.x not affected as the version was bumped)
```

Those changes will then be backported by a maintainer to the affected branches.

However, some bugs may apply only to a specific release, for example because it is using an older version of a package. In that case, patches should be based off the maintenance branch, and the patch subject prefix must include the maintenance branch name (for example "[PATCH 2020.02.x]"). This can be done with the `git format-patch` flag `--subject-prefix`:

```
$ git format-patch --subject-prefix "PATCH 2020.02.x" \
    -M -s -o outgoing origin/2020.02.x
```

Then send the patches with `git send-email`, as described above.

### 22.5.5. Patch revision changelog

When improvements are requested, the new revision of each commit should include a changelog of the modifications between each submission. Note that when your patch series is introduced by a cover letter, an overall changelog may be added to the cover letter in addition to the changelog in the individual commits. The best thing to rework a patch series is by interactive rebasing: `git rebase -i origin/master`. Consult the git manual for more information.

When added to the individual commits, this changelog is added when editing the commit message. Below the `Signed-off-by` section, add `---` and your changelog.

Although the changelog will be visible for the reviewers in the mail thread, as well as in [patchwork](https://patchwork.ozlabs.org/project/buildroot/list/), `git` will automatically ignores lines below `---` when the patch will be merged. This is the intended behavior: the changelog is not meant to be preserved forever in the `git` history of the project.

Hereafter the recommended layout:

```
Patch title: short explanation, max 72 chars

A paragraph that explains the problem, and how it manifests itself. If
the problem is complex, it is OK to add more paragraphs. All paragraphs
should be wrapped at 72 characters.

A paragraph that explains the root cause of the problem. Again, more
than one paragraph is OK.

Finally, one or more paragraphs that explain how the problem is solved.
Don't hesitate to explain complex solutions in detail.

Signed-off-by: John DOE <john.doe@example.net>

---
Changes v2 -> v3:
  - foo bar  (suggested by Jane)
  - bar buz

Changes v1 -> v2:
  - alpha bravo  (suggested by John)
  - charly delta
```

Any patch revision should include the version number. The version number is simply composed of the letter `v` followed by an `integer` greater or equal to two (i.e. "PATCH v2", "PATCH v3" …).

This can be easily handled with `git format-patch` by using the option `--subject-prefix`:

```
$ git format-patch --subject-prefix "PATCH v4" \
    -M -s -o outgoing origin/master
```

Since git version 1.8.1, you can also use `-v <n>` (where <n> is the version number):

```
$ git format-patch -v4 -M -s -o outgoing origin/master
```

When you provide a new version of a patch, please mark the old one as superseded in [patchwork](https://patchwork.ozlabs.org/project/buildroot/list/). You need to create an account on [patchwork](https://patchwork.ozlabs.org/project/buildroot/list/) to be able to modify the status of your patches. Note that you can only change the status of patches you submitted yourself, which means the email address you register in [patchwork](https://patchwork.ozlabs.org/project/buildroot/list/) should match the one you use for sending patches to the mailing list.

You can also add the `--in-reply-to=<message-id>` option when submitting a patch to the mailing list. The id of the mail to reply to can be found under the "Message Id" tag on [patchwork](https://patchwork.ozlabs.org/project/buildroot/list/). The advantage of ***\*in-reply-to\**** is that patchwork will automatically mark the previous version of the patch as superseded.

## 22.6. Reporting issues/bugs or getting help

Before reporting any issue, please check in [the mailing list archive](https://buildroot.org/downloads/manual/manual.html#community-resources) whether someone has already reported and/or fixed a similar problem.

However you choose to report bugs or get help, either by opening a bug in the [bug tracker](https://buildroot.org/downloads/manual/manual.html#community-resources) or by [sending a mail to the mailing list](https://buildroot.org/downloads/manual/manual.html#community-resources), there are a number of details to provide in order to help people reproduce and find a solution to the issue.

Try to think as if you were trying to help someone else; in that case, what would you need?

Here is a short list of details to provide in such case:

- host machine (OS/release)
- version of Buildroot
- target for which the build fails
- package(s) for which the build fails
- the command that fails and its output
- any information you think that may be relevant

Additionally, you should add the `.config` file (or if you know how, a `defconfig`; see [Section 9.3, “Storing the Buildroot configuration”](https://buildroot.org/downloads/manual/manual.html#customize-store-buildroot-config)).

If some of these details are too large, do not hesitate to use a pastebin service. Note that not all available pastebin services will preserve Unix-style line terminators when downloading raw pastes. Following pastebin services are known to work correctly: - https://gist.github.com/ - http://code.bulix.org/

