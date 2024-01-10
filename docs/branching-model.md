# Branching Model

## Table of Contents

<!-- MarkdownTOC -->

1. [Preface](#preface)
   - [Environment Setup](#environment)
   - [Command Format](#format)
1. [Branches](#branches)
1. [Develop in Feature Branches](#develop)
1. [Merge to Develop with Pull Requests](#pr)
   - [Basic Principles](#pr-basic)
   - [For Developers (Pull Request Submitter)](#pr-dev)
   - [For Reviewers (Everyone)](#pr-rev)

<!-- /MarkdownTOC -->

---

<a id="preface"></a>

## Preface

<a id="environment"></a>

### Environment Setup

1. All the commands in this document work in Debian 12 (bookworm) and Ubuntu 22.04.3 (jammy). They should also work in all Debian-based distribution (distro).
1. This document assumes that the latest `git` command (version ≥ 2.39.2) is installed properly. Please install `git` by:
    ```shell
    sudo apt install -y git
    ```
1. [Login](https://github.com/login) GitHub or [create](https://github.com/signup) a new account. For more info about creating an account, refer to [this video](https://youtu.be/PxGDLdcg3oQ).
1. Follow the [instructions from GitHub](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/creating-a-personal-access-token) to create a [classic access token](https://github.com/settings/tokens) as the alternative password. Without this, you have to fork our repositories (repo), and cannot edit the collaborated repos directly.
1. Allow you to login `git` once and for all in command line. (Reference: [github - Git asks for username every time I push - Stack Overflow](https://stackoverflow.com/a/17979600))
    ```shell
    git config --global credential.helper store
    ```
1. Login to GitHub and download (clone) the repositories (repo). Replace `<access_token>` with your access token. (Reference: [git - GitHub Clone with OAuth Access Token - Stack Overflow](https://stackoverflow.com/a/66156992))
    ```shell
    ACCESS_TOKEN="<access_token>"
    git clone https://oauth2:"${ACCESS_TOKEN}"@github.com/coding-ray/riscv-atom ~/riscv-atom
    git clone https://oauth2:"${ACCESS_TOKEN}"@github.com/coding-ray/2023-ca-term-atom ~/arch-docs-main
    unset ACCESS_TOKEN
    ```
1. Set username and email for commits. You should use only the no-reply email (`<id>+<username>@users.noreply.github.com`) in your [email settings](https://github.com/settings/emails) for privacy.
    ```shell
    git config --global user.email "<no-reply-GitHub-email>"
    git config --global user.name "<the-name-you-prefer>"
    ```
    Alternatively, use the following command to get your no-reply email. Please set `<username>` to your GitHub username. (Reference: [Where can I find the GitHub ID in my account? - Stack Overflow](https://stackoverflow.com/a/17309026))
    ```shell
    USERNAME="<username>"
    curl -s https://api.github.com/users/${USERNAME} | \
        sed -n '/"id":/p' | \
        sed -e "s/[^:]*: \([0-9]*\),/\1+${USERNAME}@users.noreply.github.com/"
    unset USERNAME
    ```
1. (Optional) Set the default editor when running `git commit`, `git rebase`, etc.
    ```shell
    # vim
    git config --global core.editor "vim"
    
    # VS Code
    git config --global core.editor "code --wait"
    ```
1. (Optional) Sign your commits (Reference: [Signing commits - GitHub Docs](https://docs.github.com/en/authentication/managing-commit-signature-verification/signing-commits)):
    1. Generate an RSA-4096 SSH key pair for signing. Replace `<key-name>` with your preferred name, such as `112-1_ca-sign`; and `<comments>` such as `signing key of Ray Huang for the Computer Architecture course in 112-1`.
        ```shell
        mkdir -p ~/.ssh
        ssh-keygen -t rsa -b 4096 -f ~/.ssh/<key-name> -N "" -C "<comments>"
        ```
    1. Hide the signing key pair from others to please `git commit`.
        ```shell
        chmod 600 ~/.ssh/<key-name>*
        ```
    1. Add the public signing key `~/.ssh/<key-name>.pub` to the [GitHub keys page](https://github.com/settings/keys) by "New SSH key", adding an identifiable title, key type of "signing key", and the content of `~/.ssh/<key-name>.pub`.
    1. In a git repo (directory), require each commit and tag be signed with the SSH singing key `~/.ssh/<key-name>.pub`.  (Reference: [Signing Git Commits with SSH Keys - blog.dbrgn.ch](https://blog.dbrgn.ch/2021/11/16/git-ssh-signatures/))
        ```shell
        git config commit.gpgsign true
        git config tag.gpgsign true
        git config gpg.format ssh
        git config user.signingkey ~/.ssh/<key-name>.pub
        ```
    1. For the first time that this device signs a commit, do the following steps.
    1. Create `~/.ssh/git_allowed_signers` with your no-reply email matched your email configured previously and the public signing key. Notice the trailing space in the quoted string in the first command.
        ```shell
        echo -n "<no-reply-email> " >> ~/.ssh/git_allowed_signers
        cat ~/.ssh/<key-name>.pub >> ~/.ssh/git_allowed_signers
        ```
    1. Tell git about that file. (Reference: [Telling Git about your signing key - GitHub Docs](https://docs.github.com/en/authentication/managing-commit-signature-verification/telling-git-about-your-signing-key#telling-git-about-your-ssh-key))
        ```shell
        git config --global gpg.ssh.allowedSignersFile ~/.ssh/git_allowed_signers
        ```

<a id="format"></a>

### Command Format

1. Texts surrounded with a pair of <u>angle brackets</u> have to be replaced with proper texts. E.g., `<email address>` should be replaced and it can be `hello@gmail.com`.
1. Texts surrounded with a pair of <u>square brackets</u> are optional. E.g., `[--all]` can appear as `--all` or nothing.
1. There may be texts surrounded with both `<>` and `[]`. It means that the texts are optional. If it is used, the content should be replaced. E.g., `ls [<directory>]`.
1. All commands are run in `bash` as a normal user in `sudo` group (a `sudoer`) by default.

<a id="branches"></a>

## Branches

Primary reference in this section: [A successful Git branching model » nvie.com](https://nvie.com/posts/a-successful-git-branching-model/)

1. We have only three kinds of branches as follows in this project:
    1. `master` or `main`: A branch composed of only working and tested commits. Open a [pull request](#pr) (PR) to bring works from `develop` to `master`.
    1. `develop`: A branch in development. Open a pull request to bring works from `feat-<feature>` to `develop`.
    1. `feat-<feature>`: A branch composed of only one feature or a group of features, but a single feature is preferred to prevent messing up the code base. Features branches are prefixed `feat-`. Hot-fixes also go here.
1. As a result, our commit graph looks like the following two pictures. (Author: [Vincent Driessen](https://nvie.com/about/); source: [A successful Git branching model » nvie.com](https://nvie.com/posts/a-successful-git-branching-model/).)

<table>
   <tr>
      <td valign=top>
         <img src="res/branch-dev-master.png"
              alt="develop and master branches"
              width="60%"/>
      </td>
      <td valign=top>
         <img src="res/branch-feat-dev.png"
              alt="feature and develop branches"
              width="60%"/>
      </td>
   </tr>
</table>

<a id="develop"></a>

## Develop in Feature Branches

1. Create a new [feature branch](#branches) named `<branch-name>` for some feature, and enter into it. It is recommended to branch off `develop` to keep track of the latest development progress.
    ```shell
    git pull
    git switch develop
    git checkout -b <branch-name>
    ```
1. After making some changes, commit it [atomically](docs/commit-convention.md#atomic-commits). `-p` is to select only a part of a file to commit. `<file-or-dir-*>` means a file or a directory (folder), such as `.` for the current directory. Check [commit convention](docs/commit-convention.md) for a proper commit title and body.
    ```shell
    git add [-p] <file-or-dir-1> [<file-or-dir-2> ...]
    git commit [-m <commit-title>]
    ```
1. Push the commits to the remote repository (named `origin` by default) on GitHub. `-u` equals to `--set-upstream`.
    ```shell
    git push -u origin <branch-name>
    ```

<a id="pr"></a>

## Merge to Develop with Pull Requests

<a id="pr-basic"></a>

### Basic Principles

1. After the development of the features related to a feature branch is done, [open](../../pulls) a pull request with the following settings:
   - base: `develop`
   - compare: `feat-<feature>`
1. Everyone including the feature developer should review the pull request (PR).
1. For discussion only, go [issues](https://github.com/coding-ray/streaming-speech-classifier/issues).

<a id="pr-dev"></a>

### For Developers (Pull Request Submitter)

Primary reference:
- [Pull Request Best Practices](https://blog.codacy.com/pull-request-best-practices)
- [How do you review a Pull Request? : r/cscareerquestions](https://www.reddit.com/r/cscareerquestions/comments/za2ill/comment/iyk9xzz/)

1. Keep the PR focused and short to make code reviewing faster and better, and make developing more agile. Try to break the feature into parts as their own PRs, and separate refactoring from feature development. (Reference: [Small Pull Requests: 6 reasons why they are the best choice](https://blog.codacy.com/small-pull-requests))
1. Provide unit tests to your code.
1. Provide concise but clear title and summary about the updates.
1. Analyze your code statically and dynamically, and refine commits and pull request messages after five minutes. View your works from the reviewers' perspective.
1. Remove temporary code, such as debugging `printf` and commented code if they are irrelevant.

Introducing these tools, the code will be more robust (reliable), but we don't have them in this project:
1. Using GitHub workflow, unit tests, code coverage or continuous integration check to catch bugs on several platforms when submitting a PR. All of these approaches test against the submitted code, and test each part of code. (For more info, check [unit testing - What is Mocking? - Stack Overflow](https://stackoverflow.com/a/40244095).)
1. Variable and function naming standard, unified formats or formatters, design pattern, and affected scope related to the changes.

<a id="pr-rev"></a>

### For Reviewers (Everyone)

Primary reference: [Best Practices for Reviewing Pull Requests in GitHub - Rewind](https://rewind.com/blog/best-practices-for-reviewing-pull-requests-in-github/)

1. Respect the other people's time: Review the PRs as soon as possible (in one day) to prevent submitters forgetting their works after a while.
1. Test again before merging: Try to test some edge cases to prevent potential bugs.
1. Always provide constructive and precise feedback:
   - Comment with positive languages like "I suggest" and "You could improve ... by doing ..." instead of "Do this" and "What's wrong with you" to prevent submitters being provoked.
   - Double-check every comments you write.
   - Comments with suggestion should provide a precise direction for the improvement and come with grammarly corrected words, such as precisely pointed "it" and functionalities to improve. ([Grammarly](https://www.grammarly.com/) does well on grammar suggestion, btw.)