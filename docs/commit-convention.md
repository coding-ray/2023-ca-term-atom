## Commit Convention

Primary reference:

- [Conventional Commit Messages](https://gist.github.com/qoomon/5dfcdf8eec66a051ecd85625518cfd13)
- [Clean Conventional Commits - DEV Community](https://dev.to/sublimegeek/clean-conventional-commits-40l8)

<a id="rules"></a>
<a id="atomic-commits"></a>

## Basic Rules for Commits

1. Commit **frequently** to have more flexibility to revert changes, like having multiple save-points in a game.
1. Commit **atomically** to have the same benefits of committing frequently.
1. Commit **descriptively** to allow everyone know what's going on in a few seconds. It is related to a commit title.
1. Have decentralized code base, such as pushing commits to GitHub or backing up code in an USB, to prevent data loss when the computer is corrupted someday.
1. Make commits immutable. That is, replace force pushing with reverting commits. Otherwise, the users who clone the repo with orphan commits will be stumbled with conflicts in the same brach.
1. For examples, please check the references in the top of this document.

## Commit Format

A conventional commit is in the following format, which includes title, body and footer(s). `[...]`'s are optional `<...>` need to be replaced with proper contexts.

```
<type>[(scope)][!]: <description>

[body]

[footer(s)]
```

<a id="title"></a>

### Commit Title and Breaking Changes (!)

1. A commit title is all lowercased, except for the words that commonly written in capital, such as Docker, RISC-V, C#, USA, etc.
1. Add `!` before the first colon for breaking changes. That is, after the scope if the scope is present, or after type. Breaking changes are the modification that breaks the API, compatibilities of "components, versions, or platforms". Remember to add the breaking change [footer](#footer).

### Commit Type

1. A commit type in the title can be one of the following types. Choose the best matched one.
1. If two or more types matched, please break into multiple commits to have only one matched type.

| Type     | Content                                                     |
| -------- | ----------------------------------------------------------- |
| build    | Building components like CI pipeline, dependencies, etc.    |
| ci       | CI pipeline changes                                         |
| chore    | Changes that don’t alter the source, but necessary.         |
| docs     | Documentation changes.                                      |
| feat     | New features.                                               |
| fix      | Fixing defects or bugs.                                     |
| misc     | The same as "chore".                                        |
| perf     | Performance enhancements.                                   |
| refactor | Refactors; altering the code with the same functionalities. |
| revert   | Reverting changes.                                          |
| style    | Cosmetic changes irrelevant to functionalities.             |
| test     | Unit tests.                                                 |

### Commit Scope

Commit scope may be one of the following values, or the other words best matching your commits. With scopes, everyone knows the impacted code.
1. api
1. compiler
1. Docker dependencies
1. release

### Commit Description

1. Use the imperative, present tense, such as "change", instead of "changed" or "changes". Think of `This commit will <description>` or `This commit should <description>`.
1. Don't capitalize the first letter.
1. Don't dot (.) at the end.

### Commit Body

1. The body should include the motivation for the change and contrast this with previous behavior.
1. Use the imperative, present tense, such as "change", instead of "changed" or "changes"
1. This is the place to mention issue identifiers and their relations.

<a id="footer"></a>

### Commit Footer

1. The footer should contain any information about Breaking Changes and is also the place to reference Issues that this commit refers to.
1. Optionally reference an issue by its ID.
1. Breaking changes should start with the word `BREAKING CHANGES:` followed by a space or two newlines. The rest of the commit message is then used for this.