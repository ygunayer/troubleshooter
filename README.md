# Memory Lane
Basically a TIL repo that contains a number of small tips and tricks for various topics.

Entries are vaguely categorized under subsections, and each entry has a subsection under its category. The said section is titled with a short description of the corresponding problem, and the details about the issue, the reason or explanation behind it, and its solutions are listed under.

## Git
### Git commands behave like `less` or `vim`
**Description:** When using a git command like `git branch -vv`, the output is in `vim` or `less` format where the entire screen is replaced by the said app, as opposed to simply being output in a `cat`-like fashion.

**Explanation:** Git has the concept of *pager*s, which aim to solve the problem of displaying long lines without the ability to navigate through them easily, exactly the same problem that Unix pagers such as `more`, `less` and even `vim` might solve. While this sounds reasonable for a command such as `git log`, commands with reasonably smaller output such as `git branch` also seems to be using a pager now. AFAIK this was not the case before.

**Solution:** Disable the pager for specific commands by setting the `pager.<command name>` to `false`. For instance:
```bash
$ git config --global pager.branch false
```

That way, if you ever need the opposite behavior, you can simply use the good ol' `|` operator to pipe the output to your favorite pager.
```bash
$ git branch -vv | less
```
