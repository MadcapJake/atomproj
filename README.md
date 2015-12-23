# atomproj

This is a hackish attempt at giving projects more of a first-class feel (at least in Gnome Shell).

I tried passing a command-line argument to atom, using an environment variable, and using a small config file, but really nothing seemed that good until I just decided to stick with a full-on bash solution :fist:.

## What it does

* It will use `xdotool` to execute a series of keyboard commands on an open instance of Atom to open a project.
* It can create desktop files and add them to an `Atom Projects` menu directory that will allow you to access them from Gnome's menu.

Type in `atom` or `project` and you will see all of the projects that you've added in this way!

## Caution

To get the full effect, you'll need `devilspie2`, `xdotool`, `xdg-desktop-menu`, and Gnome Shell.

## Reason

Originally I wanted to add projects to a quicklist but the only way to update those (that I could find) was by restarting Gnome Shell.  So I at least wanted a quick way to find my projects and open them without having to open atom and type in `alt+shift+p` followed by the project name.

## Setup

First I am using `devilspie2` and this lua script:

```lua
-- $HOME/.config/devilspie2/atom.lua
debug_print("Window Name: " .. get_window_name());
debug_print("Application Name: " .. get_application_name());

if (string.find(get_window_name(), "Atom")) then
  if (string.find(get_window_name(), "$HOME %-")) then
        set_window_workspace(get_workspace_count());
  end
  debug_print("xid: " .. get_window_xid());
  local xid = get_window_xid();
  while string.find(get_window_name(), "untitled") do
    os.execute("sleep " .. 1);
    local xprop_res = io.popen("xprop -id " .. xid .. " WM_NAME");
    local wm_name = string.match(xprop_res:read('*l'), 'WM_NAME%(UTF8_STRING%) = "(.+)"')
    if (string.find(wm_name, "%- $HOME %-")) then
      set_window_workspace(get_workspace_count());
    end

  end
  debug_print("New Window Name: " .. wm_name);
```

This allows me to open Atom in my home folder and move to another desktop OOSOOM. I originally tried using `xvfb` or `xpra` but these solutions proved ineffective.  I couldn't find a way to get one of Atom's zygote instances to run in my `:1` display.

So now I've added `atom $HOME` to my startup applications and thus it automatically shuffles an atom instance to another desktop.  This has the added bonus of being an already-open instance for editing one-off files, config files, rc files and the like.

You'll also need to copy the directory file into `~/.local/share/desktop-directories/` and the svg folder icon (or use your own to match your theme) into somewhere that Gnome looks for icons (`/home/jrusso/.icons/`).

Now I've linked the `atomproj` bash script into my `~/.local/bin` folder and I can run the script from the command line!  You can just use the script that way or use the `-c` arg to create desktop files and update the menu (only available in systems with `xdg-desktop-menu`.

## Future

Maybe I'll automate some of this into an install script.  I was also thinking of adding a `-a` arg for adding all projects as desktop files.  If it's possible, it'd be neat to also add support for other icon themes (my icon is just a vertex folder icon with an atom symbol and gradient applied).
