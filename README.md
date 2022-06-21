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

## Thunderbird
### Adding a Google Calendar
**Description:** When adding a Google Mail account to Thunderbird does detect the associated calendar, but doesn't actually import it. If you Google the issue the first result that comes up suggests installing an addon, but it's incredibly outdated as the max. Thunderbird version it supports is 78.x whereas the latest one is 91.x (at the moment of writing).

**Solution:** Add the calendar manually.

Switch to the Calendars tab on Thunderbird, and on Calendars toolbar on the left, click on the plus sign. In the popup window, select "On the Network", and in the next form, enter your e-mail address to the `User name` field, and `https://apidata.googleusercontent.com/caldav/v2/` to the `Location` field.

Now, when you hit the `Find Calendars` button, another popup will open and allow you to select the calendars to sync.

If you have multiple Google Mail accounts on Thunderbird, you can add them individually, but once you've added them, make sure that they're associated with the right e-mail address. To do that, right click on a calendar in the Calendar toolbar, go into Properties, and select the correct address in the `E-mail` field.

## Docker on Windows
### docker and docker-compose Commands Fail on WSL 1
**Description:** When attempting to run the `docker` and `docker-compose` commands on WSL 1 they fail with a message that starts with the line `The command 'docker' could not be found in this WSL 1 distro.`

**Explanation:** Microsoft has drastically improved the Linux integration in WSL v2, so much so that Docker for Windows uses a WSL2-based Linux instance to run the containers on. As a result of them working actively working on WSL2, however, the WSL1 side of things are neglected.

As long as you enable the `Expose daemon on tcp://localhost:2375 without TLS` setting on Docker Desktop, you can just run `docker.exe` or `docker-compose.exe` on a WSL1 terminal instead of `docker` or `docker-compose` and they work perfectly fine. The reason the latter two don't run is that they're actually pointed towards shell scripts that simply fail without a real reason as soon as they realize the user is running WSL1.

Which is nothing but absurd because it's still perfectly reasonable that users prefer WSL1 over WSL2 because the filesystem performance between the host and the guest systems on WSL2 is a nightmare to the point of being unusable.

**Solution:** Remove the absurd gatekeeping code from the shell scripts at `C:\Program Files\Docker\Docker\resources\bin\docker` and `C:\Program Files\Docker\Docker\resources\bin\docker-compose`.

If you're paranoid like me you can also back those files up, but in the end, both shell scripts can contain something like this:

```bash
#!/usr/bin/env sh
#
# Copyright (c) Docker Inc.

binary=$(basename "$0")
"$binary.exe" "$@"
```

...which basically runs `docker.exe` or `docker-compose.exe` whenever you run `docker` or `docker-compose`. Since we only update the contents of those files we don't need to restart our terminals either, they'll just work.
