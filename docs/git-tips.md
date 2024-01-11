<a id="commands"></a>

## Useful Commands

1. Regularly check the commit graph, especially right before you push commits to the remote. Remember `a dog`. (Reference: [git log - Pretty Git branch graphs - Stack Overflow](https://stackoverflow.com/a/35075021); reference: [explainshell.com - git log --all --oneline --graph --decorate](https://explainshell.com/explain?cmd=git+log+--all+--oneline+--graph+--decorate))
   ```shell
   git log --all --decorate --oneline --graph
   ```
   Alternatively,
   ```shell
   # do it once
   git config --global alias.adog "log --all --decorate --oneline --graph"

   # use it later when needed
   git adog
   ```
   Notes:
   * Use `q` or capital `ZZ` to quit the log; `j` or `↓` to scroll down; `k` or `↑` to scroll up in the log.
   * Enter `git log` simply to get a verbose log.
1. Synchronize to the original repo from a forked one. Take [coding-ray/riscv-atom](https://github.com/coding-ray/riscv-atom), which is a fork from [saursin/riscv-atom](https://github.com/saursin/riscv-atom), as an example. For more info about the access token, please refer to [docs/branching-model.md](docs/branching-model.md#environment).
   ```shell
   # do it once
   https://oauth2:<access-token>@github.com/coding-ray/riscv-atom ~/arch-riscv-atom
   cd ~/arch-riscv-atom
   git remote add upstream https://github.com/saursin/riscv-atom

   # do when needed
   git pull upstream
   git checkout main # in case not in main branch
   git merge upstream/main
   git push
   ```