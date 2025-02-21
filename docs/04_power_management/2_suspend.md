## Suspend

### Power States

The `/sys/power/state` file in Linux is an interface that allows you to manage the system's power states. It is part of the `sysfs` virtual filesystem and provides options for putting the system into various power-saving modes such as suspend or hibernate.

The available power states can vary depending on your hardware and system configuration, but generally include:

1.  `freeze`: The system's processes are frozen, and the CPU goes into a low-power state, but no state is lost. This is similar to a very low-power idle state.
2.  `standby`: The system enters a low-power state, similar to freeze, but with deeper power savings. This mode usually affects more hardware components, and waking up from it is faster than suspend.
3.  `mem`: This state corresponds to suspend-to-RAM. In this state, the system's state is saved in RAM, and most hardware components are powered down, but the system can wake up quickly. This is commonly known as "sleep" or "suspend."
4.  `disk`: This is suspend-to-disk (hibernate). The system's state is saved to the disk, and the machine can be powered off. When the system is powered back on, the state is restored from the disk.
5.  `on` (if supported): A fully operational state where the system is not in any power-saving mode.

You can view the available power states by reading the contents of `/sys/power/state`:

```bash
cat /sys/power/state
```

Mine is:

```bash
freeze mem disk
```

To set one of the power states you can write to `/sys/power/state`:

```bash
echo mem | sudo tee /sys/power/state
```

But some of them don't work as expected and it is better to run them through `systemd`.

### Suspend to RAM

To check the supported sleep states run:

```bash
cat /sys/power/mem_sleep
```

In this case, `s2idle` corresponds to S0ix, while `deep` refers to S3 (the traditional standby state).
The output shows the selection. You should get something like `s2idle [deep]` which shows `deep` is selected.
`deep` is more energy efficient rather than `s2idle` and the only active component is RAM.

To activate `deep` run:

```bash
echo deep | sudo tee /sys/power/mem_sleep
```

`deep` represents suspend-to-RAM.

you can make the change permanent through the `MemorySleepMode`:

```bash
sudo mkdir /etc/systemd/sleep.conf.d/
sudo nano /etc/systemd/sleep.conf.d/mem-deep.conf
```

Add:

```conf
[Sleep]
MemorySleepMode=deep
```

If you want to set `s2idle` just for test, there is a chance that replacing `deep` with `s2idle` doesn't work for you (like me). In this case you should try _kernel parameter_ `mem_sleep_default`. To set a kernel parameter in `systemd-boot`:

```bash
sudo nano /boot/loader/entries/<YOUR-KERNEL>.conf
```

Find the options line and add `mem_sleep_default=s2idle` at the end of the line separating it with a space:

```bash
options ... mem_sleep_default=s2idle
```

Finally:

```bash
sudo reboot
```

And check the parameter:

```bash
cat /sys/power/mem_sleep
```

My default suspend mode is `deep` and there is no need to do any of this for me!

**References:**

- <https://wiki.archlinux.org/title/Power_management/Suspend_and_hibernate>
- <https://docs.kernel.org/admin-guide/pm/sleep-states.html#basic-sysfs-interfaces-for-system-suspend-and-hibernation>
- <https://wiki.archlinux.org/title/Kernel_parameters>
- <https://web.archive.org/web/20230614200816/https://01.org/blogs/qwang59/2018/how-achieve-s0ix-states-linux>

### Changing the power button behavior in systemd

1. Create "drop-ins" in the `/etc/systemd/logind.conf.d/` directory:

   ```bash
   sudo mkdir -p /etc/systemd/logind.conf.d/
   sudo nano /etc/systemd/logind.conf.d/override.conf
   ```

   Then add:

   ```conf
   [Login]
   HandlePowerKey=suspend
   ```

2. Restart `systemd-logind`:
   ```bash
   sudo systemctl restart systemd-logind
   ```

**References:**

- https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/9/html/administering_the_system_using_the_gnome_desktop_environment/changing-the-power-button-behavior_administering-the-system-using-the-gnome-desktop-environment#changing-the-power-button-behavior-in-systemd_changing-the-power-button-behavior
