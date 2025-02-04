## Running GUI applications as root

First of all I don't recommend to run GUI apps as root. Emmanuele Bassi, a GNOME developer said:

> _By running GUI applications as an admin user you are literally running millions of lines of code that have not been audited properly to run under elevated privileges; you are also running code that will touch files inside your $HOME and may change their ownership on the file system; connect, via IPC, to even more running code, etc._

Anyway if you want to do it you should first create a `polkit` action for your specific app:

```bash
sudo nano /usr/share/polkit-1/actions/org.freedesktop.pkexec.hiddify.policy
```

Attention to the file name. I used `polkit` for `hiddify`. The content of this file should be as below:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE policyconfig PUBLIC
 "-//freedesktop//DTD PolicyKit Policy Configuration 1.0//EN"
 "http://www.freedesktop.org/standards/PolicyKit/1.0/policyconfig.dtd">

<policyconfig>
    <action id="org.freedesktop.pkexec.hiddify">
        <description>Run Hiddify with elevated privileges</description>
        <message>Authentication is required to run Hiddify</message>
        <defaults>
            <allow_any>auth_admin</allow_any>
            <allow_inactive>auth_admin</allow_inactive>
            <allow_active>auth_admin</allow_active>
        </defaults>
        <annotate key="org.freedesktop.policykit.exec.path">/usr/bin/hiddify</annotate>
        <annotate key="org.freedesktop.policykit.exec.allow_gui">true</annotate>
    </action>
</policyconfig>
```

You should put the execute path key in your action. Also, allow gui using `allow_gui` set to `true` flag.

Finally you can create your new icon that uses your `polkit` configuration:

```bash
cp /usr/share/applications/hiddify.desktop ~/.local/share/applications/hiddify-sudo.desktop
```

I took the base `hiddify` icon from `/usr/share/applications`.

Edit the configuration file:

```bash
nano /home/USERNAME/.local/share/applications/hiddify-sudo.desktop
```

Change these lines as below:

```conf
[Desktop Entry]
Type=Application
Name=Hiddify (sudo)
GenericName=Hiddify (sudo)
Icon=hiddify
Exec=pkexec hiddify %U
StartupNotify=true
```

Finally update desktop database:

```bash
update-desktop-database ~/.local/share/applications
```

**References:**

- <https://wiki.archlinux.org/title/Polkit>
- <https://wiki.archlinux.org/title/Running_GUI_applications_as_root>
- <https://unix.stackexchange.com/questions/203136/how-do-i-run-gui-applications-as-root-by-using-pkexec>
- <https://askubuntu.com/questions/287845/how-to-configure-pkexec>
