# docker4mac intf

As of the time of writing Docker for Mac can't access containers via IP from
the host. Let's fix that.

It worth remembering that this appears to be a commonly requested feature, so
it might be [worth checking][docker-for-mac-networking] to see if it's been
fixed in recent versions. The current version now is ce-17.03.

[docker-for-mac-networking]: https://docs.docker.com/docker-for-mac/networking/

## Approach

Add an additional network interface (provided by `tuntaposx`) to `moby` (the VM
containing the Linux kernel and Docker daemon) that's accessible to the `host`.
Use the `macvlan` docker network type to attach containers directly to the new
interface with direct conectivity to the `host`.

## Guide

This is a quick overview of the steps involved in making containers accessible
to the host. Keep scrolling for a script to automate the process!

1. Install [`tuntap` OSX][tto] driver
2. Make the local user own `/dev/tap1` - prevents needing to run Docker as root
3. Move `com.docker.hyperkit` to `com.docker.hyperkit.real` in the Docker app
4. Install `com.docker.hyperkit` [shim][shim] to manipulate arguments
5. Restart Docker
6. Create a `macvlan` network with `eth1` as the parent
7. Register the host of the `tap` interface

**WARNING:** Unfortunately step 7 must currently be performed after every
restart of Docker. This is because the `tap` interface only persists while
Docker is running. Hopefully this can be improved upon.

[tto]: http://tuntaposx.sourceforge.net/
[shim]: /install.sh#L38-L57

## Install

A script to perform most of the steps above can be found [here][script].
Unfortunately the warning regarding step seven still applies.

[script]: /install.sh

## Uninstall

There's no dedicated uninstaller, but the process is pretty simple:

1. Move `com.docker.hyperkit.real` back to `com.docker.hyperkit`
2. Reboot Docker
3. Change the owner of the chosen `tap` device to `root`, or
4. Removal instructions for tuntaposx can be found in [their FAQ][ttofaq].

[ttofaw]: http://tuntaposx.sourceforge.net/faq.xhtml

## Known Limitations

- Ignored port mappings - due to usage of a `macvlan` network

## Thanks

- **Michael Henkel**
  Without these [forum][mhenkel1] [posts][mhenkel2] this wouldn't exist.
- **tuntaposx.sourceforge.net**

[mhenkel1]: https://forums.docker.com/t/support-tap-interface-for-direct-container-access-incl-multi-host/17835/2
[mhenkel2]: https://forums.docker.com/t/support-tap-interface-for-direct-container-access-incl-multi-host/17835/3
