# GameScope Flatpak extension for the Freedesktop runtime

With this Flatpak packaging of [GameScope](https://github.com/Plagman/gamescope) it's possible to run an existing Flatpak
application through the GameScope compositor without needing to bundle GameScope with application or make any change to
the existing packaging.

## Why

This extension was mainly created in preparation for me getting a HiDPI monitor.  
I've been avoiding HiDPI displays like the plague, so I won't have to deal with problems in the so many apps that still
use outdated widget toolkits that don't support fractional scalling, but it does look like I would be forced to get such
display.  
Hopefully, this extension will help scale these applications to fit HiDPI displays.  

Another reason is sandboxing X11 applications from each other, while introducing less issues than `Xpra` or similar tools.

## How this works

The Freedesktop runtime defines an extension mount point that bind-mounts any extension with the
`org.freedesktop.Platform.VulkanLayer` prefix onto a subdirectory of `/usr/lib/extensions/vulkan`.  
This GameScope extension takes advantage of the above, and adds GameScope and it supporting files to the runtime under
`/usr/lib/extensions/vulkan/GameScope`.

Yes, this is a hack, as this specific extension mount point was likely not intended to be used with this kind of tool,
and submission of this extension to Flathub might not be approved due to this fact, but that's good enough for my needs.

The extension definition doesn't includes the `add-ld-path` key, and there's no extension key to handle append a
directory to `PATH`, so we will need to take care of both of these in a different way.

It should be noted that this extension adds a couple of shared libraries that can possibly conflict with those provided
by the application.  
This includes `libevdev`, `libfontenc`, `libinput`, `libliftoff`, `libmtdev`, `libseat`, `libwlroots`, `libxcvt`, `libXfont2`, `libXRes`.  
As I haven't made too much use of this extension, this doesn't concern me currently, but I might look into statically
linking against these libraries.  
Please file a bug report if you hit an issue due to a wrong shared lib resolution/loading.

While this should be obvious, please be aware that the extension branch/version must match the runtime's one.

## How to use

First, build and install the extension. See [Flatpak Builder](https://github.com/flatpak/flatpak-builder)'s
documentation for more details.

Next, run the application with `gamescope` as the given command, while setting the `PATH` and `LD_LIBRARY_PATH`
environment variables to include the directories of our GameScope extension.  
Make sure to allow Wayland socket access, and append to the end gamescope's option and of course the application's
executable name.

Here it should be noted that the application's packaging might also set these variables manually, so if the application
doesn't work as expected, then check its packaging.

For example, let's run [ImHex](https://imhex.werwolv.net/) through GameScope.
```
$ flatpak run \
  --command=gamescope \
  --env=LD_LIBRARY_PATH=/usr/lib/extensions/vulkan/GameScope/lib \
  --env=PATH=/app/bin:/usr/bin:/usr/lib/extensions/vulkan/GameScope/bin \
  --socket=wayland \
  net.werwolv.ImHex \
  -w 1920 -h 1080 -- \
  imhex
```

ImHex is actually a good example, as this is an X11 application without Wayland support, so we can block X11 session
access and see that it still runs correctly.
```
$ flatpak run \
  --command=gamescope \
  --env=LD_LIBRARY_PATH=/usr/lib/extensions/vulkan/GameScope/lib \
  --env=PATH=/app/bin:/usr/bin:/usr/lib/extensions/vulkan/GameScope/bin \
  --nosocket=x11 \
  --socket=wayland \
  net.werwolv.ImHex \
  -w 1920 -h 1080 -- \
  imhex
```

## Limitations

GameScope is not perfect (yet?), and there are a couple of limitations to be aware of.  
The main one is the [missing clipboard sync support](https://github.com/Plagman/gamescope/issues/303).  
Possibly others are [keyboard modifiers forwarding](https://github.com/Plagman/gamescope/issues/266), and maybe
[zero-copy](https://github.com/Plagman/gamescope/issues/64).

## Possible Flathub submission

I don't plan to submit this, and if fact, I'm very much against it due to the current limitations of runtime extensions,
and the existence of shared libraries in the extension.

If GameScope is actually useful for regular non-games applications, then in my opinion it should be added to the
runtime, and applications that need a newer version than the runtime's one can bundle their own copy.
