---
layout: post
title:  "Installing Proxmox VE 9 on Intel MacBook Pros"
date:   2025-08-29 12:00:00 -0700
tags:   laptops proxmox linux macbook-pro apple
---

As I get a new laptop, I usually keep the old one around for a while and try to find a new use for them instead of selling or donating them. That is how I have two different MacBook Pro laptops that have been sitting around and collecting dust, including a 15-inch Late 2016 MacBook Pro (2016 MBP) and a 16-inch 2019 MacBook Pro (2019 MBP). Apple had stopped supporting the 2016 MBP when it came to new Mac OS X/macOS versions a couple of years ago and the 2019 MBP will get its last major macOS version when macOS 26 is released.

Over the past couple of months, I have been toying with installing a Linux distro on the 2016 MBP and actually got the just-released [Debian 13 (Trixie)](https://www.debian.org/News/2025/20250809) up and running on it ([Mastodon thread with progress and roadblocks](https://linh.social/@qlp/115074226275506158)). Unfortunately and as expected, not everything worked out of the box and there are things that I still have not been able to get working, including the Touch Bar and the Broadcom wireless controller. Both of those mean that using the laptop as a portable computer and use while roaming around wouldn't work out very well.

That's when I had the idea of turning the 2016 MBP (and the 2019 MBP) into a server, specifically, as a virtualization host using [Proxmox VE](https://www.proxmox.com/en/products/proxmox-virtual-environment/overview) 9.0. I'm already using Proxmox VE on a pair of desktop computers turned servers and I wanted a system to try out the new version. I've documented my experience with installing Proxmox VE on both MBP laptops and issues that I ran across in doing so.

### Late 2016 MacBook Pro

The [15-inch Late 2016 MacBook Pro](https://support.apple.com/en-us/111975) (identifiers: [MacBookPro13,3 / A1707](https://everymac.com/systems/apple/macbook_pro/specs/macbook-pro-core-i7-2.7-15-late-2016-retina-display-touch-bar-specs.html)) has a quad-core [Intel Core i7-6820HQ processor](https://www.intel.com/content/www/us/en/products/sku/88970/intel-core-i76820hq-processor-8m-cache-up-to-3-60-ghz/specifications.html), 16 GB of RAM, 1 TB of storage, a Radeon Pro 455 dedicated GPU, the infamous butterfly switch keyboard, and a Touch Bar. I previously used this laptop as my main laptop for development, editing photos and images, and other usual tasks before replacing it with the 2019 MBP.

<div class="row">
    <div class="col col-6">
        <figure class="figure">
            <a target="_blank" href="/assets/images/mbp-proxmox/2016-mbp-proxmox-summary.png">
            <img src="/assets/images/mbp-proxmox/2016-mbp-proxmox-summary.png" class="img-fluid border" alt="Proxmox VE Host Summary Page for a 2016 Apple MacBook Pro">
            </a>
            <figcaption class="figure-caption text-center">
                Proxmox VE Node Summary for a 2016 Apple MacBook Pro
            </figcaption>
        </figure>
    </div>
    <div class="col col-6">
        <figure class="figure">
            <a target="_blank" href="/assets/images/mbp-proxmox/2016-mbp-proxmox-fastfetch.png">
            <img src="/assets/images/mbp-proxmox/2016-mbp-proxmox-fastfetch.png" class="img-fluid border" alt="Output of the fastfetch command running under Proxmox VE 9 on a 2016 Apple MacBook Pro">
            </a>
            <figcaption class="figure-caption text-center">
                Obligatory fastfetch Output for the 2016 MacBook Pro
            </figcaption>
        </figure>
    </div>
</div>

Before I installed Proxmox VE on the laptop, I made sure that I backed up everything important from the laptop, shut down the laptop, plugged in an [IODD ST400](http://en.iodd.kr/wiki/index.php/Iodd-ST400) drive into an Anker PowerShare 6-in-1 USB Type-C hub (there is a [newer version of the hub](https://www.anker.com/products/a8365) that I don't have and have not tested with Linux) with a Gigabit Ethernet adapter and powered on the laptop while holding the `Option` key so I could boot off of the Proxmox VE 9 installer ISO mounted on the IODD drive. At the boot device selector, I selected the boot device labeled "GRUB" and pressed the `Enter` key to continue the boot process.

When I got to the Proxmox VE installer boot menu, I noticed that I couldn't use the laptop keyboard to change the menu selection and continnue booting. I pulled out a spare USB keyboard, plugged it into the USB hub and continued on with the text-based installer.

Once in the installer, the internal storage device was detected and I was able to configure the network interface with a static IP address, then continued with the install. After rebooting the laptop and being greeted with the GRUB menu, I noticed that the laptop keyboard was working and successfully logged into the console. From my main laptop, I was able to get to the Proxmox VE web interface and didn't see any issues or errors appear in the logging panel.

Before proceeding with the configuration and testing of Proxmox VE, I switched from the default Proxmox VE Enterprise repositories to the no-subscription repositories following the instructions provided by Proxmox's wiki page, "[Package Repositories](https://pve.proxmox.com/wiki/Package_Repositories)".

By default, the laptop will go into sleep or suspend mode when I close the lid. To disable that behavior, I edited the `/etc/systemd/logind.conf` configuration file and set all three of the following configuration keys to `ignore`:

- `HandleLidSwitch`
- `HandleLidSwitchDocked`
- `HandleLidSwitchExternalPower`

After editing the file, I ran `systemd restart systemd-logind` for the changes to take effect. I ran a continuous ping on the laptop's IP address, closed the laptop's lid, and was still able to ping and connect to the laptop. Later on, I needed to restart the laptop after installing some updates, I noticed that the laptop seemed to shutdown rather than restarting. Knowing that MacBook Pros require having at least an external display connected to the laptop to operate in clamshell mode when using macOS, I plugged in a [fit-headless HDMI display emulator](https://edge.compulab.com/store/fit-headless/) to the USB hub, booted up the laptop, closed the lid and remotely restarted the laptop. Thankfully, the laptop restarted normally and I didn't have to manually boot the laptop up after each major update or system-wide configuration change.

<div class="row">
    <div class="col col-6">
        <figure class="figure">
            <a target="_blank" href="/assets/images/mbp-proxmox/2016-mbp-proxmox-debian-vm-fastfetch.png">
            <img src="/assets/images/mbp-proxmox/2016-mbp-proxmox-debian-vm-fastfetch.png" class="img-fluid border" alt="Debian 13 virtual machine running under Proxmox VE 9 on a 2016 Apple MacBook Pro">
            </a>
            <figcaption class="figure-caption text-center">
                Debian 13 Proxmox Virtual Machine Running on the 2016 MacBook Pro
            </figcaption>
        </figure>
    </div>
    <div class="col col-6">
        <figure class="figure">
            <a target="_blank" href="/assets/images/mbp-proxmox/2016-mbp-proxmox-grafana-cpu-temp.png">
            <img src="/assets/images/mbp-proxmox/2016-mbp-proxmox-grafana-cpu-temp.png" class="img-fluid border" alt="Processor temperature charts for the 2016 Apple MacBook Pro from Grafana">
            </a>
            <figcaption class="figure-caption text-center">
                Hardware Temperature Monitor Grafana Graph for the 2016 MacBook Pro
            </figcaption>
        </figure>
    </div>
</div>

In order to reduce overall power consumption and temperatures, I used the "[Proxmox VE CPU Scaling Governor](https://community-scripts.github.io/ProxmoxVE/scripts?id=scaling-governor)" helper script to change the CPU scaling governor from `performance` to `powersave`. The script adds the following line to the root user's crontab that automatically sets the correct value on reboot.

```bash
@reboot (sleep 60 && echo "powersave" | tee /sys/devices/system/cpu/cpu*/cpufreq/scaling_governor)
```

To set the scaling governor back to `performance` on each reboot, run `crontab -e` and change `powersave` to `performance`.

For grins and giggles, I tried to passthrough the dedicated AMD Radeon Pro GPU through to a Debian 13 virtual machine using the "[PCI Passthrough](https://pve.proxmox.com/wiki/PCI_Passthrough)" article in Proxmox's wiki and a [forum post with instructions](https://forum.proxmox.com/threads/pci-gpu-passthrough-on-proxmox-ve-8-installation-and-configuration.130218/) for Proxmox VE 8 (though they would also apply to version 9). Unfortunately, even with IOMMU and drivers blacklisted, I encountered the AMD GPU reset bug and it caused the laptop to crash and lock up. Thankfully, the laptop booted back up just fine. Since it's not something that I will need, I decided to undo the changes.

The 2016 MBP was *not* going to break any performance records and would not perform nearly as well as my other two Proxmox VE servers, in part to the older generation Intel Core processor and in part to Apple's decision to make the laptop thin at the great expense to thermals. At idle, the core temperatures will sit around 60 degrees C and spiking up to around 70-75 degrees C under lighter loads with the CPU scaling governor set to `powersave`. When running a stress test in the Debian 13 virtual machine, most of the core temperatures spiked to 90-93 degrees C no matter if the governor was set to `powersave` or `performance`.

### 2019 MacBook Pro

Even with the successful installation and use of Proxmox VE 9 on the 2016 MBP, I was quite pessimistic when it came to the [16-inch 2019 MacBook Pro](https://support.apple.com/en-us/111932) (identifiers: [MacBookPro16,1 / A2141](https://everymac.com/systems/apple/macbook_pro/specs/macbook-pro-core-i9-2.3-eight-core-16-2019-scissor-specs.html)). I knew going in that the Apple computers with the [Apple T2](https://en.wikipedia.org/wiki/Apple_T2) security chip could cause additional headaches.

The 2019 MBP has an [Intel Core i9-9880H processor](https://www.intel.com/content/www/us/en/products/sku/192987/intel-core-i99880h-processor-16m-cache-up-to-4-80-ghz/specifications.html), 32 GB of RAM, 2 TB of storage, an AMD Radeon Pro 5500M, a Touch Bar, and most importantly and the reason why I ordered the laptop as soon as it was orderable, a scissor-switch keyboard. While the keyboard was so much better than the awful butterfly-switch keyboard Apple inflicted on their MacBook laptops for far too long, thermals was still a major issue, especially with a Core i9 processor. As with the 2016 MBP, I primarily used the laptop for development, editing photos and images, amongst the usual stuff.

<div class="row">
    <div class="col col-6">
        <figure class="figure">
            <a target="_blank" href="/assets/images/mbp-proxmox/2019-mbp-proxmox-summary.png">
            <img src="/assets/images/mbp-proxmox/2019-mbp-proxmox-summary.png" class="img-fluid border" alt="Proxmox VE Host Summary Page for a 2019 Apple MacBook Pro">
            </a>
            <figcaption class="figure-caption text-center">
                Proxmox VE Node Summary for a 2019 Apple MacBook Pro
            </figcaption>
        </figure>
    </div>
    <div class="col col-6">
        <figure class="figure">
            <a target="_blank" href="/assets/images/mbp-proxmox/2019-mbp-proxmox-fastfetch.png">
            <img src="/assets/images/mbp-proxmox/2019-mbp-proxmox-fastfetch.png" class="img-fluid border" alt="Output of the fastfetch command running under Proxmox VE 9 on a 2019 Apple MacBook Pro">
            </a>
            <figcaption class="figure-caption text-center">
                Obligatory fastfetch Output for the 2019 MacBook Pro
            </figcaption>
        </figure>
    </div>
</div>

I used an Anker PowerShare 5-in-1 USB Type-C hub (there is a [newer version of the hub](https://www.anker.com/products/a8338) that I don't have and have not tested with Linux) and the IODD drive to try to boot the Proxmox VE 9 installer on the 2019 MBP while holding down the `Option` key, but choosing the "GRUB" option would eventually boot the laptop into recovery mode (while macOS was still installed) and stated that an update needed to be installed in order to boot from the installer ISO. I chose to run the update, which took a few minutes and a reboot of the laptop, I ran into the same issue. I tried booting using the `EFI Boot` boot device selection, but the laptop just rebooted. At that point, I remembered that the laptop had the Secure Boot setting defaulted to "Full Security" in the "Startup Security Utility". I used the instructions found in Apple's "[About Startup Security Utility on a Mac with the Apple T2 Security Chip](https://support.apple.com/en-us/102522)" support document to set the "Secure Boot" setting to "No Security". Upon rebooting, I was finally able to boot into the Proxmox installer using the `EFI Boot` boot device selection.

I'm not certain that the Secure Boot setting would be an issue for all of the 2019-2020 MacBook Pro laptops, as it may depend on the firmware version on each laptop and other variables that I may or may not know about.

While the laptop keyboard did work at the GRUB boot menu, it would not work in the text-based installer. So, out came the USB keyboard and I was able to finish the installation. While booting into the installation environment, there was around a 30 second pause with the following messages printed to the console:

```text
Starting systemd-udevd version 257.7-1
Synthesizing the initial hotplug events (subsystem)
Synthesizing the initial hotplug events (devices)
Waiting for /dev to be fully populated
```

No other error messages appeared between the pause and the installer starting up. As with the 2016 MBP, the installer found the USB Gigabit Ethernet controller and was able to get a DHCP address. Unfortunately, that's when things started to go a bit wrong.

With the installation complete and the laptop rebooting, I noticed that there was a longer pause when Proxmox was trying to stand up a network connection and the `vmbr0` bridge interface. There were a couple of error messages that were written out to the console:

```text
[FAILED] Failed to start ifupdown2-pre.service - Helper to synchronize boot up for ifupdown.
[DEPEND] Dependency failed for networking.service - Network initialization
[FAILED] Failed to start systemd-udev-settle.service - Wait for udev To Complete Device Initialization.
[DEPEND] Dependency failed for zfs-import-cache.service - Import ZFS pools by cache file.
```

After the pause, the login prompt would appear with the static IP address I had configured during the installation process. So, I tried pinging the IP address, but no response. I logged in through the console and ran `ip link` and saw that the corresponding network interface shown as `DOWN` and `vmbr0` was not listed. Running `ifup` and `ifdown` did not help, but I was able to get the interface and `vmbr0` to come up after running:

```bash
systemctl restart ifupdown2-pre.service
systemctl restart networking.service
```

Unfortunately, the network interfaces would not come back up after a reboot, necessitating running the commands each and every time the laptop was booted. When the network interfaces were up, I was able to get to the Proxmox VE web interface and test things out. Since I wanted to be able to run the laptop headless, having to go through the whole rigmarole of restarting services after each boot would be too much work.

<div class="row">
    <div class="col col-6">
        <figure class="figure">
            <a target="_blank" href="/assets/images/mbp-proxmox/2019-mbp-almalinux-10-fastfetch.png">
            <img src="/assets/images/mbp-proxmox/2019-mbp-almalinux-10-fastfetch.png" class="img-fluid border" alt="Fastfetch output from an AlmaLinux 10 virtual machine running under Proxmox VE 9 on a 2019 Apple MacBook Pro">
            </a>
            <figcaption class="figure-caption text-center">
                Obligatory fastfetch Output from an AlmaLinux 10 Proxmox Virtual Machine Running on a 2019 MacBook Pro
            </figcaption>
        </figure>
    </div>
    <div class="col col-6">
        <figure class="figure">
            <a target="_blank" href="/assets/images/mbp-proxmox/2019-mbp-almalinux-10-cockpit.png">
            <img src="/assets/images/mbp-proxmox/2019-mbp-almalinux-10-cockpit.png" class="img-fluid border" alt="Cockpit web interface for an AlmaLinux 10 virtual machine running under Proxmox VE 9 on a 2019 Apple MacBook Pro">
            </a>
            <figcaption class="figure-caption text-center">
                Cockpit Web Interface for an AlmaLinux 10 Proxmox Virtual Machine Running on a 2019 MacBook Pro
            </figcaption>
        </figure>
    </div>
</div>

I didn't have a lot of spare time to figure out why the network interface would not automatically come up on reboot and what was causing that pause during the boot process.

### Extending Battery Life

In order to extend the life of the battery while running Proxmox VE, I will need to look at ways to cap the battery charge level by way of `tlp` or `upower`. Without that, the battery will charge up to and stay charged at around 100% of the usable capacity. That is not a good thing.

I found a GitHub repo, [applesmc-next](https://github.com/c---/applesmc-next), that provides the required kernel module that exposes the required control, specifically `/sys/class/power_supply/BAT0/charge_control_end_threshold`. Instead of installing the standard `linux-headers` metapackage in Proxmox VE 9, I had to install the `pve-headers` metapackage before I could build and install the module via DKMS.

To set `/sys/class/power_supply/BAT0/charge_control_end_threshold` to 75 when Proxmox VE starts up, I added the following entry to root's crontab:

```bash
@reboot (sleep 60 && echo 75 | tee /sys/class/power_supply/BAT0/charge_control_end_threshold)
```

I still need to tests to make sure that everything is working properly.

### Next Steps

Although I would love to be able to get both the 2016 and 2019 MacBook Pro laptops up and running with Proxmox VE, I have run out of spare time to troubleshoot the 2019 MBP. Since I have a spare USB hub with an integrated Gigabit Ethernet controller, I don't have to take it offline in order to work on the 2019 MBP. I'll publish an update if I am able to resolve the network interface issues when Proxmox VE boots up.

I am also entertaining the idea of installing [AlmaLinux](https://almalinux.org/) or [Rocky Linux](https://rockylinux.org/) on the 2019 MBP to see how usable it would be with the [Cockpit](https://cockpit-project.org/) web-based management interface. And, if I'm really feeling a little spicy, I might give [Arch Linux](https://archlinux.org/) or the new Arch-based hotness, [CachyOS](https://cachyos.org/), a try.

As for the 2016 MBP, I would like get a USB 2.5 Gigabit Ethernet controller that is well supported on Linux and either a USB 3.0 or a Thunderbolt 3 enclosure for an M.2 NVMe SSD. I have a spare 2 TB WD Black M.2 NVMe drive that I want to use as storage for virtual machines and containers. This will reduce the number of write cycles on the internal, soldered-on SSD storage. What will probably be the limiting factor for the 2016 MBP is the sixth-gen Core i7 processor along with the MBP's constrained thermal design. I'm just glad that the laptop isn't just collecting dust and, hopefully, extend its useful lifespan.

### Additional Information

If you are interested in seeing what PCI and USB devices show up under Proxmox VE, I have posted the output of `lspci` and `lsusb` for both MacBook Pro laptops. Both laptops had a different Anker USB hub connected in order to have a network connection.

- 2016 MacBook Pro (MacBookPro13,3)
  - [lspci](/assets/text/mbp-proxmox/2016-mbp-lspci.txt)
  - [lsusb](/assets/text/mbp-proxmox/2016-mbp-lsusb.txt)
- 2019 MacBook Pro (MacBookPro16,1)
  - [lspci](/assets/text/mbp-proxmox/2019-mbp-lspci.txt)
  - [lsusb](/assets/text/mbp-proxmox/2019-mbp-lsusb.txt)
