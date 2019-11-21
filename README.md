# Troubleshooter
This is basically a reference book where I explain the reasons behind various issues and provide solutions for them.

Entries are vaguely categorized under subsections, and each entry has a subsection under its category. The said section is titled with a short description of the corresponding problem, and contain the details about the issue, the reason or explanation behind it, and its solution in the section body.

## Git
### Git commands behave like `less` or `vim`
**Description:** When using a git command like `git branch -vv`, the output is in `vim` or `less` format where the entire screen is replaced by the said app, as opposed to simply being output in a `cat`-like fashion.

**Explanation:** Git has the concept of *pager*s, which aim to solve the problem of displaying long lines without the ability to navigate through them easily, exactly the same problem that Unix pagers such as `more`, `less` and even `vim` might solve. While this sounds reasonable for a command such as `git log`, commands with reasonably smaller output such as `git branch` also seems to be using a pager now. AFAIK this was not the case before.
> *Update:* This change seems to be intended, as indicated in the release notes for Git v2.16.0 https://github.com/git/git/blob/master/Documentation/RelNotes/2.16.0.txt#L85-L88

**Solution:** Disable the pager for specific commands by setting the `pager.<command name>` to `false`. For instance:
```bash
$ git config --global pager.branch false
```

That way, if you ever need the opposite behavior, you can simply use the good ol' `|` operator to pipe the output to your favorite pager.
```bash
$ git branch -vv | less
```

### git-gui crashes on macOS
**Description:** On macOS, git-gui randomly begins to crash immediately after being launched. This issue appears to occur more frequently on multi-display environments.

**Explanation:** Written in Tcl/Tk, git-gui attempts to store its geometry information in git's config so that it can restore it later. Every now and these dimensions turn out to be invalid, and when restored (aka when git-gui is launched), they cause macOS's rendering pipeline to throw an error, crashing the app.

**Solution:** Clear the stored dimensions using the following command
```bash
$ git config --local --unset gui.geometry
```

## Erlang
### `net_kernel:start/1` Fails on Non-Distributed Nodes
**Description:** Calling `net_kernel:start/1` on a non-distributed Erlang node (aka one started without the `-name` or `-sname` options) fails with `{'EXIT', nodistribution}` error.

**Explanation:** Erlang uses EPMD (Erlang Port-Mapper Daemon) to handle distribution among Erlang nodes, and when the `-name` or `-sname` option is used EPMD is started automatically. However, when these options are omitted, the EPMD is not started, so when `net_kernel:start/1` is called, the node can't reach EPMD as it attempts to register itself.

**Solution:** Run EPMD manually in daemon mode by executing the following command:

```bash
$ epmd -daemon
```

This allows both distributed and non-distributed nodes to become distributed.

## macOS
### iTunes Hijacks Media Buttons from Other Apps
**Description:** When any media key such as Play or Pause is pressed, macOS launches iTunes, even if there's a media-related app such as Spotify running in the background.

**Explanation:** With High Sierra and Mojave, macOS began demanding explicit accessibility permissions for apps to handle user input when they're in the background, apps that can handle media keys don't receive any input unless these permissions are given. Since Apple's sandboxing principles usually don't apply to their own apps, iTunes is capable of receiving those inputs by default, via the hidden iTunes Helper process.

**Solution:** Give the necessary permissions to the app of your choice.

For instance, if you want Spotify to receive your media keys when it's in the background, open up `System Preferences`, go to `Privacy > Accessibility`, press the `+` button and add Spotify to the list of allowed apps. The result is instantaneous, so Spotify will be able receive your input immediately.
