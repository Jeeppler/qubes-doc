---
layout: doc
title: Updating software in dom0
permalink: /doc/software-update-dom0/
redirect_from:
- /en/doc/software-update-dom0/
- /doc/SoftwareUpdateDom0/
- /wiki/SoftwareUpdateDom0/
---

Updating software in dom0
=========================

Why would one want to update software in dom0?
----------------------------------------------

Normally, there should be few reasons for updating software in dom0. This is because there is no networking in dom0, which means that even if some bugs are discovered e.g. in the dom0 Desktop Manager, this really is not a problem for Qubes, because none of the 3rd party software running in dom0 is accessible from VMs or the network in any way. Some exceptions to this include: Qubes GUI daemon, Xen store daemon, and disk back-ends. (We plan move the disk backends to an untrusted domain in Qubes 2.0.) Of course, we believe this software is reasonably secure, and we hope it will not need patching.

However, we anticipate some other situations in which updating dom0 software might be necessary or desirable:

-   Updating drivers/libs for new hardware support
-   Correcting non-security related bugs (e.g. new buttons for qubes manager)
-   Adding new features (e.g. GUI backup tool)

How is software updated securely in dom0?
-----------------------------------------

The update process is split into two phases: "resolve and download" and "verify and install." The "resolve and download" phase is handled by the "UpdateVM." (The role of UpdateVM can be assigned to any VM in the Qubes VM Manager, and there are no significant security implications in this choice. By default, this role is assigned to the firewallvm.) After the UpdateVM has successfully downloaded new packages, they are sent to dom0, where they are verified and installed. This separation of duties significantly reduces the attack surface, since all of the network and metadata processing code is removed from the TCB.

Although this update scheme is far more secure than directly downloading updates in dom0, it is not invulnerable. For example, there is nothing that the Qubes project can feasibly do to prevent a malicious RPM from exploiting a hypothetical bug in GPG's `--verify` operation. At best, we could switch to a different distro or package manager, but any of them could be vulnerable to the same (or a similar) attack. While we could, in theory, write a custom solution, it would only be effective if Qubes repos included all of the regular TemplateVM distro's updates, and this would be far too costly for us to maintain.

How to update software in dom0
------------------------------

As of Qubes R2 Beta 3, the main update functions have been integrated into the Qubes VM Manager GUI: Simply select dom0 in the VM list, then click the **Update VM system** button (the blue, downward-pointing arrow). In addition, updating dom0 has been made more convenient: You will be prompted on the desktop whenever new dom0 updates are available and given the choice to run the update with a single click.

Of course, command line tools are still available for accomplishing various update-related tasks (some of which are not available via Qubes VM Manager). In order to update dom0 from the command line, start a console in dom0 and then run one of the following commands:

1.  To check and install updates for dom0 software:

    ~~~
    $ sudo qubes-dom0-update
    ~~~

1.  To install additional packages in dom0 (usually not recommended):

    ~~~
    $ sudo qubes-dom0-update anti-evil-maid
    ~~~

    You may also pass the `--enablerepo=` option in order to enable optional repositories (see yum configuration in dom0). However, this is only for advanced users who really understand what they are doing.

### How to downgrade a specific package

1.  Download an older version of the package:

    ~~~
    sudo qubes-dom0-update package-version
    ~~~

    Yum will say that there is no update, but the package will nonetheless be downloaded to dom0.

2.  Downgrade the package:

    ~~~
    sudo yum downgrade package-version
    ~~~

### How to re-install a package

You can re-install in a similar fashion to downgrading.

1.  Download the package:

    ~~~
    sudo qubes-dom0-update package
    ~~~

    Yum will say that there is no update, but the package will nonetheless be downloaded to dom0.

2.  Re-install the package:

    ~~~
    sudo yum reinstall package
    ~~~

    Note that yum will only re-install if the installed and downloaded versions match. You can ensure they match by either updating the package to the latest version, or specifying the package version in the first step using the form `package-version`.

### How to uninstall a package

If you've installed a package such as anti-evil-maid, you can remove it with the following command:

    sudo yum remove anti-evil-maid
    
### Testing repositories

There are three Qubes dom0 testing repoistories:

* `qubes-dom0-current-testing` -- testing packages that will eventually land in the stable
  (`current`) repository
* `qubes-dom0-security-testing` -- a subset of `qubes-dom0-current-testing` that contains packages
  that qualify as security fixes
* `qubes-dom0-unstable` -- packages that are not intended to land in the stable (`qubes-dom0-current`)
  repository; mostly experimental debugging packages

To temporarily enable any of these repos, use the `--enablerepo=<repo-name>`
option. Example commands:

~~~
sudo qubes-dom0-update --enablerepo=qubes-dom0-current-testing
sudo qubes-dom0-update --enablerepo=qubes-dom0-security-testing
sudo qubes-dom0-update --enablerepo=qubes-dom0-unstable
~~~

To enable or disable any of these repos permanently, change the corresponding boolean in
`/etc/yum.repos.d/qubes-dom0.repo`.

### Kernel Upgrade ###

Install newer kernel. The following example installs kernel 3.19 and was tested on Qubes R3 RC1.

~~~
sudo qubes-dom0-update kernel-3.19*
~~~

Rebuild grub config.

~~~
sudo grub2-mkconfig -o /boot/grub2/grub.cfg
~~~

Reboot required.

### Upgrading over Tor ###

Requires installed [Whonix](/doc/privacy/whonix/).

Go to Qubes VM Manager -> System -> Global Settings. See the UpdateVM setting. Choose your desired Whonix-Gateway ProxyVM from the list. For example: sys-whonix.

    Qubes VM Manager -> System -> Global Settings -> UpdateVM -> sys-whonix
