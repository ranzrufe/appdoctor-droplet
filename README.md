# AppDoctor

A [Droplet](https://getdroppy.app/docs/droppykit) for [Droppy](https://getdroppy.app), the Dynamic
Island and shelf for Mac.

Drop an app on it and AppDoctor tells you whether a newer version exists, installs it, or removes
the app. Left alone it keeps checking the apps you dropped on it, and says something on the notch
when one of them is behind.

AppDoctor is a community droplet. It is free, and it is MIT licensed.

## What it does

- **Drop an app on the shelf widget**, or on the manager that opens from it, and the app is read,
  tracked, and checked for updates straight away.
- **Three update sources, in this order.**
  - A **Homebrew cask**, upgraded with `brew upgrade --cask`, which is the tool that installed it
    and therefore knows what it would install.
  - The **Mac App Store**, for an app the store sold. The receipt in the bundle is how AppDoctor
    knows, and the store is what has to install the update.
  - The **app's own Sparkle feed**, `SUFeedURL` out of its `Info.plist`. The newest release is
    downloaded, checked against the developer's signature, and put in place.
- **Progress while an update runs.** The download is a bar, the steps after it are named, and the
  percentage Homebrew prints is read as it arrives.
- **Automatic checks** on a timer, six hours by default, plus one when an app is dropped, one when
  the shelf opens after a gap, and a Check now button on the widget, in the manager and in
  Settings.
- **Removal to the Trash**, with the app quit first, and a confirmation unless you turn that off.
  Nothing is deleted outright.
- **The files an app leaves behind go with it.** Its settings, its caches, its saved window state
  and its container are found and moved to the Trash as well, so a reinstall starts from a clean
  copy. The confirmation says how many there are before anything moves.
- **A button on every row that takes the app off AppDoctor's list**, without touching the app
  itself.
- **A settings page** for all of it: the interval, what it may do when it finds an update, and the
  list of watched apps.

## What it will not do

The refusals are as much a part of it as the actions.

- Nothing under `/System` is ever removed or replaced. An app macOS ships is read, shown, and left
  alone.
- An update is only installed when the app **inside the download** is newer than the one installed.
  A feed that mislabels its version cannot downgrade anything.
- An update signed by a **different developer** is refused. The code signature is read on both
  copies and compared, and when both are signed and the teams differ, nothing happens.
- When an app ships an Ed25519 signing key (`SUPublicEDKey`), the download is **verified against
  it** before anything is mounted or run.
- An app with none of the three sources says so: *No update source*. AppDoctor does not guess a
  vendor's download page.
- A **group container** is never removed, because it is shared with other apps. Neither is a
  symbolic link, which is somebody else's file wearing this app's name.
- When something does not work, AppDoctor says so in a sentence you can act on:
  *AdBlock is still open. Quit the app, then try again.* Every message stays in the manager until
  the work behind it succeeds, or you dismiss it.

## The shelf

AppDoctor asks Droppy's shelf to stay open, so the shelf does not close on its own while you are
working with it. You close it: the floating close button below the shelf, a swipe up, the shelf
shortcut and a click outside all still work. There is nothing to turn on and nothing to turn off.

## Installing it

**Droppy.** Open Droppy, then Settings, Store, Local droplets, and drop the bundle in. Droppy asks
once before it runs that build, and again each time it opens, unless *Don't ask again for local
droplets* is on.

**Droppy Playground**, the free app, takes the same bundle without the questions:

```bash
mkdir -p ~/Library/Application\ Support/Droppy\ Playground/Droplets/appdoctor
cp ~/Documents/AppDoctor.droplet ~/Library/Application\ Support/Droppy\ Playground/Droplets/appdoctor/
```

Then read its Store row: the subtitle is the loader's verdict.

## Using it

1. Open the shelf and drop an app on the AppDoctor card, or on the manager that opens from it.
2. The app is checked straight away. A row shows what was found: the version you have, the version
   there is, and where the update comes from.
3. **Update** installs it. **Remove** moves the app to the Trash, and the files it left behind with
   it. The **X** takes the app off AppDoctor's list and leaves the app alone.
4. The manager has every watched app, its status, and Update, Remove and Stop watching for each.
   Right click a row for Reveal in Finder, Check again and Open.

The widget shows the apps it watches and what it found for each. The button at the top of the card
opens the manager, which has everything.

## What it asks for

| Capability | For |
| --- | --- |
| `hud` | The strip on the notch when a check finds something, or when something fails |
| `expanded-surface` | The manager, which is where the list and the actions live |
| `network-client` | Reading an app's update feed, and downloading a release |
| `shelf-write` | Keeping the shelf open while you work with it |

That is the whole list. It asks for no file, accessibility, Apple events or automation capability,
and it runs no helper tool.

## The App Store

An app the Mac App Store sold is updated by the store, with `mas`:

```bash
brew install mas
```

Without it, AppDoctor still knows the app belongs to the store and says so, and it says what to
install to do better.

## Building it from source

Needs the DroppyKit SDK checkout on your `PATH` for the `droppykit` command, and Xcode 26 or
later.

```bash
export PATH="$HOME/Documents/.droplet/droppykit/Scripts:$PATH"
cd ~/Documents/.droplet/appdoctor

droppykit build      # .build/AppDoctor.droplet, universal
droppykit validate   # the checks the Store's intake runs
droppykit run        # the harness: Droppy's own Settings panel, running this
Tests/run-tests.sh   # this droplet's own tests
```

The sources are in `~/Documents/.droplet/appdoctor`. The developer README there has the layout, the
test suites and the notes.

## Licence

MIT. The text is in `LICENSE`, in the same folder as the sources.
