# Local Software Setup

??? warning "Local OS"

    All of the software and installation processes were only tested on a
    **Windows 11** computer. For Mac and Linux users some software may be very
    similar to Windows (e.g. RPi imager, VS Code), but some steps (e.g. X11
    forwarding) will be different to what is shown in these instructions.
    
    If you run into any problems with the software setup, please
    [post an issue](https://github.com/maxsitt/insect-detect-docs/issues){target=_blank}
    at the GitHub repo.

## Raspberry Pi Imager

We will use the official Raspberry Pi Imager to download and install Raspberry
Pi OS Lite to the micro SD card and directly configure some important options.
[Download Raspberry Pi Imager](https://www.raspberrypi.com/software/){target=_blank}
and install it to your computer. [In the next section](pisetup.md){target=_blank},
we are going through all necessary steps to install and configure your
Raspberry Pi OS.

---

## Visual Studio Code

We are going to use Visual Studio Code to connect to the Raspberry Pi Zero 2 W
via [SSH](https://en.wikipedia.org/wiki/Secure_Shell){target=_blank}.
[Download VS Code](https://code.visualstudio.com/){target=_blank} and install it
to your computer. More information on how to set up VS Code can be found in the
[official Docs](https://code.visualstudio.com/Docs/setup/setup-overview){target=_blank}.

After installing and starting VS Code, open the Extensions view by clicking on
the Extensions icon in the left side bar and install the
[Remote - SSH](https://bit.ly/3dw6tSI){target=_blank} extension and the
[Remote X11](https://bit.ly/3A6BNz1){target=_blank} extension.

<figure markdown>
  ![VS Code Extensions](assets/images/vscode_extensions.png){ width="400" }
  <figcaption>VS Code Extensions icon in the left side bar</figcaption>
</figure>

- The Remote - SSH extension will enable us to use the Raspberry Pi Zero 2 W as
  a remote development environment which will make testing and programming a
  lot easier.
- The Remote X11 extension in combination with a X window server (e.g. VcXsrv)
  is necessary to forward the OAK-1 camera stream received from the Raspberry
  Pi to your local machine via a
  [X11 SSH tunnel](https://en.wikipedia.org/wiki/X_Window_System){target=_blank}
  and show it on your local PC.

??? info "Recommended Extensions"

    - [Python](https://bit.ly/2Zm3Ypq){target=_blank} and
      [Pylance](https://bit.ly/3s4qKTF){target=_blank} extensions if you are
      working with Python scripts.
    - [Excel Viewer](https://bit.ly/3DNAsjM){target=_blank} extension for a
      more readable preview of the .csv metadata and log files.
    - [Save Commands](https://bit.ly/3NntHsb){target=_blank} extension if you
      want to save some often used terminal commands for direct execution.

??? question "Raspberry Pi Zero W (v1)"

    The Remote - SSH extension will not work with the Raspberry Pi Zero W, as
    the armv6l architecture is not supported. Instead, you can connect to the
    RPi Zero W via SSH directly in the VS Code Terminal by following these steps:

    - [Generate a SSH key](pisetup.md#ssh-key-based-authentication){target=_blank}.
    - [Install Raspberry Pi OS](pisetup.md#raspberry-pi-os-installation){target=_blank} to your microSD card.
    - Set the `DISPLAY` environment variable in Windows by
      [opening](https://www.howtogeek.com/235101/10-ways-to-open-the-command-prompt-in-windows-10/){target=_blank}
      the Command Prompt (cmd) and running:

        ``` powershell
        setx DISPLAY "localhost:0.0"
        ```

        You may have to reboot your computer for the changes to take effect.

    - Connect to your RPi Zero via SSH and trusted X11 forwarding (`-Y`)
      in the VS Code Terminal by running:

        ``` powershell
        ssh -Y pi@raspberrypi
        ```

        If you set a different hostname than `raspberrypi` during the
        [RPi OS installation](pisetup.md#raspberry-pi-os-installation){target=_blank},
        adapt it accordingly. When you are asked if you are sure you
        want to continue connecting, type in `yes` and hit ++enter++.

    - Use **right-click** to paste commands to the Pi's SSH Terminal.
    - You can check if X11 forwarding works correctly by running:

        ``` bash
        echo $DISPLAY
        ```

        ...which should give you the following output:

        ``` bash
        localhost:10.0
        ```

    - For a similar remote explorer experience as with the Remote - SSH extension, install the
      [SSH FS](https://marketplace.visualstudio.com/items?itemName=Kelvin.vscode-sshfs){target=_blank}
      extension.
    - Open the SSH FS extension by clicking on the new icon in the left side bar.
    - Create a new SSH FS configuration (Name: e.g. `rpi_zero`) with the following fields
      (**insert your correct Windows username**):

        - Host: `raspberrypi`
        - Port: `22`
        - Root: `~/`
        - Username: `pi`
        - Private key: `c:\Users\<username>\.ssh\id_rsa`

    - Leave the other fields blank and save the configuration with the
      **Save** button at the bottom.
    - In the SSH FS extension, click on the first symbol to the right of your
      configuration: `Add as Workspace folder`. Retry if it does not work
      immediately. This will open the `/home/pi` folder in your VS Code explorer.
      You can now view and edit files (e.g. [Python scripts](programming.md){target=_blank}
      and images) directly in VS Code and drag & drop any files or folders
      (e.g. `insect-detect`) from your PC to the RPi Zero.
    - Follow the steps for [RPi configuration](pisetup.md#rpi-configuration){target=_blank},
      [PiJuice Zero configuration](pisetup.md#pijuice-zero-configuration){target=_blank}
      and [OAK-1 configuration](pisetup.md#oak-1-configuration){target=_blank}.
    - Skip the last step to configure X11 forwarding, as this should
      already work by connecting via SSH and trusted X11 forwarding (`ssh -Y`).

---

## VcXsrv Windows X Server

To be able to see the OAK-1 camera stream that is sent to the Raspberry Pi on
your local computer (e.g. for testing or adjusting the camera trap), you will
need to install a X server if you are communicating with the RPi via SSH.
[Download VcXsrv](https://sourceforge.net/projects/vcxsrv/){target=_blank} and
install it to your computer. Start the X server with the `XLaunch.exe` and keep
all the default settings. After that you will see a tray icon in the taskbar
and you can test if the connection to the Raspberry Pi is active in
[VS Code](pisetup.md#configure-x11-forwarding){target=_blank}.

---

## DiskInternals LinuxReader

We will use DiskInternals LinuxReader to save data from the Raspberry Pi's SD
card to your local computer. This is only necessary if you are not using a
Linux-based OS, as the Linux partition format is not compatible with Windows.
[Download DiskInternals LinuxReader](https://www.diskinternals.com/linux-reader/){target=_blank}
and install it to your computer.

- Insert the Raspberry Pi's SD card in your card reader and open the program.
  You will see two partitions on the SD card: `boot` and `rootfs`.
- Double-click on the `rootfs` partition and navigate to
  `/home/pi/insect-detect/data` to see all log files and recorded images.
- Select everything you want to download to your PC and click the **Save**
  button in the upper menu bar.
- Keep the **Save Files** option, choose your output folder and select both
  options **Save directory structure** and **Extract file date from metadata**.
  After that, the data you selected will be copied to your computer.
