## Chapter 22. Contributing to Buildroot

There are many ways in which you can contribute to Buildroot: analyzing and fixing bugs, analyzing and fixing package build failures detected by the autobuilders, testing and reviewing patches sent by other developers, working on the items in our TODO list and sending your own improvements to Buildroot or its manual. The following sections give a little more detail on each of these items.

If you are interested in contributing to Buildroot, the first thing you should do is to subscribe to the Buildroot mailing list. This list is the main way of interacting with other Buildroot developers and to send contributions to. If you aren’t subscribed yet, then refer to [Chapter 5, *Community resources*](https://buildroot.org/downloads/manual/manual.html#community-resources) for the subscription link.

If you are going to touch the code, it is highly recommended to use a git repository of Buildroot, rather than starting from an extracted source code tarball. Git is the easiest way to develop from and directly send your patches to the mailing list. Refer to [Chapter 3, *Getting Buildroot*](https://buildroot.org/downloads/manual/manual.html#getting-buildroot) for more information on obtaining a Buildroot git tree.

## 22.1. Reproducing, analyzing and fixing bugs

A first way of contributing is to have a look at the open bug reports in the [Buildroot bug tracker](https://gitlab.com/buildroot.org/buildroot/-/issues). As we strive to keep the bug count as small as possible, all help in reproducing, analyzing and fixing reported bugs is more than welcome. Don’t hesitate to add a comment to bug reports reporting your findings, even if you don’t yet see the full picture.

## 22.2. Analyzing and fixing autobuild failures

The Buildroot autobuilders are a set of build machines that continuously run Buildroot builds based on random configurations. This is done for all architectures supported by Buildroot, with various toolchains, and with a random selection of packages. With the large commit activity on Buildroot, these autobuilders are a great help in detecting problems very early after commit.

All build results are available at [http://autobuild.buildroot.org](http://autobuild.buildroot.org/), statistics are at http://autobuild.buildroot.org/stats.php. Every day, an overview of all failed packages is sent to the mailing list.

Detecting problems is great, but obviously these problems have to be fixed as well. Your contribution is very welcome here! There are basically two things that can be done:

- Analyzing the problems. The daily summary mails do not contain details about the actual failures: in order to see what’s going on you have to open the build log and check the last output. Having someone doing this for all packages in the mail is very useful for other developers, as they can make a quick initial analysis based on this output alone.
- Fixing a problem. When fixing autobuild failures, you should follow these steps:
  1. Check if you can reproduce the problem by building with the same configuration. You can do this manually, or use the [br-reproduce-build](http://git.buildroot.org/buildroot-test/tree/utils/br-reproduce-build) script that will automatically clone a Buildroot git repository, checkout the correct revision, download and set the right configuration, and start the build.
  2. Analyze the problem and create a fix.
  3. Verify that the problem is really fixed by starting from a clean Buildroot tree and only applying your fix.
  4. Send the fix to the Buildroot mailing list (see [Section 22.5, “Submitting patches”](https://buildroot.org/downloads/manual/manual.html#submitting-patches)). In case you created a patch against the package sources, you should also send the patch upstream so that the problem will be fixed in a later release, and the patch in Buildroot can be removed. In the commit message of a patch fixing an autobuild failure, add a reference to the build result directory, as follows:

```
Fixes: http://autobuild.buildroot.org/results/51000a9d4656afe9e0ea6f07b9f8ed374c2e4069
```

## 22.3. Reviewing and testing patches

With the amount of patches sent to the mailing list each day, the maintainer has a very hard job to judge which patches are ready to apply and which ones aren’t. Contributors can greatly help here by reviewing and testing these patches.

In the review process, do not hesitate to respond to patch submissions for remarks, suggestions or anything that will help everyone to understand the patches and make them better. Please use internet style replies in plain text emails when responding to patch submissions.

To indicate approval of a patch, there are three formal tags that keep track of this approval. To add your tag to a patch, reply to it with the approval tag below the original author’s Signed-off-by line. These tags will be picked up automatically by patchwork (see [Section 22.3.1, “Applying Patches from Patchwork”](https://buildroot.org/downloads/manual/manual.html#apply-patches-patchwork)) and will be part of the commit log when the patch is accepted.

- Tested-by

  Indicates that the patch has been tested successfully. You are encouraged to specify what kind of testing you performed (compile-test on architecture X and Y, runtime test on target A, …). This additional information helps other testers and the maintainer.

- Reviewed-by

  Indicates that you code-reviewed the patch and did your best in spotting problems, but you are not sufficiently familiar with the area touched to provide an Acked-by tag. This means that there may be remaining problems in the patch that would be spotted by someone with more experience in that area. Should such problems be detected, your Reviewed-by tag remains appropriate and you cannot be blamed.

- Acked-by

  Indicates that you code-reviewed the patch and you are familiar enough with the area touched to feel that the patch can be committed as-is (no additional changes required). In case it later turns out that something is wrong with the patch, your Acked-by could be considered inappropriate. The difference between Acked-by and Reviewed-by is thus mainly that you are prepared to take the blame on Acked patches, but not on Reviewed ones.

If you reviewed a patch and have comments on it, you should simply reply to the patch stating these comments, without providing a Reviewed-by or Acked-by tag. These tags should only be provided if you judge the patch to be good as it is.

It is important to note that neither Reviewed-by nor Acked-by imply that testing has been performed. To indicate that you both reviewed and tested the patch, provide two separate tags (Reviewed/Acked-by and Tested-by).

Note also that *any developer* can provide Tested/Reviewed/Acked-by tags, without exception, and we encourage everyone to do this. Buildroot does not have a defined group of *core* developers, it just so happens that some developers are more active than others. The maintainer will value tags according to the track record of their submitter. Tags provided by a regular contributor will naturally be trusted more than tags provided by a newcomer. As you provide tags more regularly, your *trustworthiness* (in the eyes of the maintainer) will go up, but *any* tag provided is valuable.

Buildroot’s Patchwork website can be used to pull in patches for testing purposes. Please see [Section 22.3.1, “Applying Patches from Patchwork”](https://buildroot.org/downloads/manual/manual.html#apply-patches-patchwork) for more information on using Buildroot’s Patchwork website to apply patches.

### 22.3.1. Applying Patches from Patchwork

The main use of Buildroot’s Patchwork website for a developer is for pulling in patches into their local git repository for testing purposes.

When browsing patches in the patchwork management interface, an `mbox` link is provided at the top of the page. Copy this link address and run the following commands:

```
$ git checkout -b <test-branch-name>
$ wget -O - <mbox-url> | git am
```

Another option for applying patches is to create a bundle. A bundle is a set of patches that you can group together using the patchwork interface. Once the bundle is created and the bundle is made public, you can copy the `mbox` link for the bundle and apply the bundle using the above commands.

## 22.4. Work on items from the TODO list

If you want to contribute to Buildroot but don’t know where to start, and you don’t like any of the above topics, you can always work on items from the [Buildroot TODO list](http://elinux.org/Buildroot#Todo_list). Don’t hesitate to discuss an item first on the mailing list or on IRC. Do edit the wiki to indicate when you start working on an item, so we avoid duplicate efforts.

## 22.5. Submitting patches

### Note

*Please, do not attach patches to bugs, send them to the mailing list instead*.

If you made some changes to Buildroot and you would like to contribute them to the Buildroot project, proceed as follows.

### 22.5.1. The formatting of a patch

We expect patches to be formatted in a specific way. This is necessary to make it easy to review patches, to be able to apply them easily to the git repository, to make it easy to find back in the history how and why things have changed, and to make it possible to use `git bisect` to locate the origin of a problem.

First of all, it is essential that the patch has a good commit message. The commit message should start with a separate line with a brief summary of the change, prefixed by the area touched by the patch. A few examples of good commit titles:

- `package/linuxptp: bump version to 2.0`
- `configs/imx23evk: bump Linux version to 4.19`
- `package/pkg-generic: postpone evaluation of dependency conditions`
- `boot/uboot: needs host-{flex,bison}`
- `support/testing: add python-ubjson tests`

The description that follows the prefix should start with a lower case letter (i.e "bump", "needs", "postpone", "add" in the above examples).

Second, the body of the commit message should describe *why* this change is needed, and if necessary also give details about *how* it was done. When writing the commit message, think of how the reviewers will read it, but also think about how you will read it when you look at this change again a few years down the line.

Third, the patch itself should do only one change, but do it completely. Two unrelated or weakly related changes should usually be done in two separate patches. This usually means that a patch affects only a single package. If several changes are related, it is often still possible to split them up in small patches and apply them in a specific order. Small patches make it easier to review, and often make it easier to understand afterwards why a change was done. However, each patch must be complete. It is not allowed that the build is broken when only the first but not the second patch is applied. This is necessary to be able to use `git bisect` afterwards.

Of course, while you’re doing your development, you’re probably going back and forth between packages, and certainly not committing things immediately in a way that is clean enough for submission. So most developers rewrite the history of commits to produce a clean set of commits that is appropriate for submission. To do this, you need to use *interactive rebasing*. You can learn about it [in the Pro Git book](https://git-scm.com/book/en/v2/Git-Tools-Rewriting-History). Sometimes, it is even easier to discard you history with `git reset --soft origin/master` and select individual changes with `git add -i` or `git add -p`.

Finally, the patch should be signed off. This is done by adding `Signed-off-by: Your Real Name <your@email.address>` at the end of the commit message. `git commit -s` does that for you, if configured properly. The `Signed-off-by` tag means that you publish the patch under the Buildroot license (i.e. GPL-2.0+, except for package patches, which have the upstream license), and that you are allowed to do so. See [the Developer Certificate of Origin](http://developercertificate.org/) for details.

To give credits to who sponsored the creation of a patch or the process of upstreaming it, you may use [email subaddressing](https://datatracker.ietf.org/doc/html/rfc5233) for your git identity (i.e. what is used as commit author and email `From:` field, as well as your Signed-off-by tag); add suffix to the local part, separated from it by a plus `+` sign. E.g.:

- for a company which sponsored the submitted work, use the company name as the detail (suffix) part:

  `Your-Name Your-Surname <your-name.your-surname+companyname@mail.com>`

- for an individual who sponsored the submitted work, use their name and surname:

  `Your-Name Your-Surname <your-name.your-surname+their-name.their-surname@mail.com>`

Alternatively, especially if your email server does not support subaddressing, you can include the sponsor in your author name in parentheses, e.g. "Your Name (Sponsor Name)".

When adding new packages, you should submit every package in a separate patch. This patch should have the update to `package/Config.in`, the package `Config.in` file, the `.mk` file, the `.hash` file, any init script, and all package patches. If the package has many sub-options, these are sometimes better added as separate follow-up patches. The summary line should be something like `<packagename>: new package`. The body of the commit message can be empty for simple packages, or it can contain the description of the package (like the Config.in help text). If anything special has to be done to build the package, this should also be explained explicitly in the commit message body.

When you bump a package to a new version, you should also submit a separate patch for each package. Don’t forget to update the `.hash` file, or add it if it doesn’t exist yet. Also don’t forget to check if the `_LICENSE` and `_LICENSE_FILES` are still valid. The summary line should be something like `<packagename>: bump to version <new version>`. If the new version only contains security updates compared to the existing one, the summary should be `<packagename>: security bump to version <new version>` and the commit message body should show the CVE numbers that are fixed. If some package patches can be removed in the new version, it should be explained explicitly why they can be removed, preferably with the upstream commit ID. Also any other required changes should be explained explicitly, like configure options that no longer exist or are no longer needed.

If you are interested in getting notified of build failures and of further changes in the packages you added or modified, please add yourself to the DEVELOPERS file. This should be done in the same patch creating or modifying the package. See [the DEVELOPERS file](https://buildroot.org/downloads/manual/manual.html#DEVELOPERS) for more information.

Buildroot provides a handy tool to check for common coding style mistakes on files you created or modified, called `check-package` (see [Section 18.25.2, “How to check the coding style”](https://buildroot.org/downloads/manual/manual.html#check-package) for more information).

