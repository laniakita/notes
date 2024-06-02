# Creating an Arch Linux container with Distrobox + Podman on NixOS

## Setup

Make sure podman is [installed](https://nixos.wiki/wiki/Podman) (seems to be rootless by default on NixOS), and distrobox is too (you probably want it as a [system package](https://search.nixos.org/options?channel=unstable&show=environment.systemPackages&from=0&size=50&sort=relevance&type=packages&query=environment.systemPackages)).

## Create/Enter

Then just create the box!

```zsh
~ 
❯ distrobox create --image archlinux:latest --name archlinux --home /home/$USER/containers/archlinux --volume /etc/static/profiles/per-user:/etc/profiles/per-user:ro --additional-packages "systemd"  --pre-init-hooks "pacman -S -y -y; pacman-key --init; pacman-key --populate archlinux; pacman -S --noconfirm openssl dbus" --verbose --init
```
### Notes:
- You don't necessarily need a different home folder for the container, but it's my preference (cleaner).
- Attaching your user profile seems to allow the container access to some of the host's binaries installed with home manager.
- Installing openssl was quite important for today (06/01/2024), but it likely won't be necessary into the future (it will probably be a different dep, which is bound to happen considering Arch is usually bleeding edge).
- You might not need the --verbose flag, but I find it helpful.
- I've not read deep enough into the official init script to understand what risk (if any) exists in performing the partial update above (force refresh + install pkg xyz). If things go wrong, it might be helpful to tweak the install command to "pacman -S -y -y -u" instead, but I haven't tried that. 

And enter!

```zsh
~ 
❯ distrobox enter archlinux --verbose
```

### Notes:
- The verbose flag is especially useful when entering the container for the first time, since if something goes wrong, you won't necessarily need to hunt down the podman logs.
- In addition to the above, what might go wrong on first enter will (again) likely be a random dependency failing to match a required spec. Just add it to the pre-init hook above, and that should take care of it.

## More

There was a [helpful thread in the Distrobox discussions](https://github.com/89luca89/distrobox/discussions/1294) with even more information on doing exactly this, and formed the basis for the commands I ran. You might find it useful too!
