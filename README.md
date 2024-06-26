# 🌱 herbe
> Daemon-less notifications without D-Bus. Minimal and lightweight.

<p align="center">
  <img src="https://user-images.githubusercontent.com/24730635/90975811-cd62fd00-e537-11ea-9169-92e68a71d0a0.gif" />
</p>

## Features
* Under 200 lines of code
* Doesn't run in the background, just displays the notification and exits
* No external dependencies except Xlib and Xft
* Configurable through `config.h` or Xresources ([using this patch](https://github.com/dudik/herbe/pull/11))
* [Actions support](#actions)
* Extensible through [patches](https://github.com/dudik/herbe/pulls?q=is%3Aopen+is%3Apr+label%3Apatch)

### Dismiss a notification
A notification can be dismissed either by clicking on it with `DISMISS_BUTTON` (set in config.h, defaults to left mouse button) or sending a `SIGUSR1` signal to it:
```shell
$ pkill -SIGUSR1 herbe
```
Dismissed notifications return exit code 2.

### Actions
Action is a piece of shell code that runs when a notification gets accepted. Accepting a notification is the same as dismissing it, but you have to use either `ACTION_BUTTON` (defaults to right mouse button) or the `SIGUSR2` signal.
An accepted notification always returns exit code 0. To specify an action:
```shell
$ herbe "Notification body" && echo "This is an action"
```
Where everything after `&&` is the action and will get executed after the notification gets accepted.

### Newlines
Every command line argument gets printed on a separate line by default e.g.:
```shell
$ herbe "First line" "Second line" "Third line" ...
```
You can also use `\n` e.g. in `bash`:
```shell
$ herbe $'First line\nSecond line\nThird line'
```
But by default `herbe` prints `\n` literally:
```shell
$ herbe "First line\nStill the first line"
```
Output of other programs will get printed correctly, just make sure to escape it (so you don't end up with every word on a separate line):
```shell
$ herbe "$(ps axch -o cmd:15,%cpu --sort=-%cpu | head)"
```

### Multiple notifications
Notifications are put in a queue and shown one after another in order of creation (first in, first out). They don't overlap and each one is shown for its entire duration.

### Notifications don't show up
Most likely a running notification got terminated forcefully (SIGKILL or any uncaught signal) which caused the semaphore not getting unlocked. First, kill any `herbe` instance that is stuck:
```shell
$ pkill -SIGKILL herbe
```
Then just call `herbe` without any arguments:
```shell
$ herbe
```
Notifications should now show up as expected.

Don't ever send any signals to `herbe` except these:
```shell
# same as pkill -SIGTERM herbe, terminates every running herbe process
$ pkill herbe

$ pkill -SIGUSR1 herbe
$ pkill -SIGUSR2 herbe
```
And you should be fine. That's all you really need to interact with `herbe`.

### Dependencies
* X11 (Xlib)
* Xft

### Build
```shell
doas make install
```
`make install` requires root privileges because it copies the resulting binary to `/usr/local/bin`. This makes `herbe` accessible globally.

You can also use `make clean` to remove the binary from the build folder, `sudo make uninstall` to remove the binary from `/usr/local/bin` or just `make` to build the binary locally.

## Configuration
My configuration just consists of using Fira Mono Nerd Font and the Tokyonight color pallete. I suggest checking out the original repository if you are looking to make any changes/your own config.
