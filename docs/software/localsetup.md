# Local Software Setup

??? warning "Local OS"

    All of the software and installation processes were only tested on a
    **Windows 11** computer. For Mac and Linux users some software may be very
    similar to Windows (e.g. RPi imager, VS Code), but some steps (e.g. X11
    forwarding) might be different to what is shown in these instructions.
    
    If you run into any problems with the software setup, please
    [post an issue](https://github.com/maxsitt/insect-detect-docs/issues){target=_blank}
    at the GitHub repo.

---

## Raspberry Pi Imager

[Download RPi Imager](https://www.raspberrypi.com/software/){target=_blank}
and install it to your computer. We will use the official Raspberry Pi Imager to
download and install Raspberry Pi OS Lite to the microSD card and configure some
important options, which are shown in the [next section](pisetup.md){target=_blank}.

---

## Visual Studio Code

[Download VS Code](https://code.visualstudio.com/){target=_blank} and install it
to your computer. More information on how to set up VS Code can be found in the
[official Docs](https://code.visualstudio.com/Docs/setup/setup-overview){target=_blank}.
Using Visual Studio Code to connect to your Raspberry Pi via
[SSH](https://en.wikipedia.org/wiki/Secure_Shell){target=_blank} is highly
recommended. For casual users, install the extensions listed under `Basic`.

For experienced users, connecting and working exclusively via the Terminal
(e.g. Windows PowerShell) is also possible. If you want to use the Raspberry
Pi Zero 2 W as a remote development environment, install the VS Code extensions
listed under `Advanced`.

???+ info "Recommended Extensions"

    === "Basic"

        - [SSH FS](https://marketplace.visualstudio.com/items?itemName=Kelvin.vscode-sshfs){target=_blank}
          extension to mount the `/home/pi` folder as local workspace folder in VS Code.
        - [Excel Viewer](https://bit.ly/3DNAsjM){target=_blank} extension
          for a more readable preview of the .csv metadata and log files.

    === "Advanced"

        - [Remote - SSH](https://bit.ly/3dw6tSI){target=_blank} extension to
          use the Raspberry Pi Zero 2 W as a remote development environment
          with more functions, including Pylint support. **Not supported by
          RPi Zero W (v1)!**
        - [Remote X11](https://bit.ly/3A6BNz1){target=_blank} extension to
          forward the OAK-1 camera stream received from the Raspberry Pi via
          X11 and show it on your local PC. Required if you are using the
          Remote - SSH extension instead of connecting via the Terminal.
        - [Python](https://bit.ly/2Zm3Ypq){target=_blank} and
          [Pylance](https://bit.ly/3s4qKTF){target=_blank} extensions if you are
          working with Python scripts.
        - [Save Commands](https://bit.ly/3NntHsb){target=_blank} extension if you
          want to save some often used terminal commands for direct execution.

---

## VcXsrv Windows X Server

[Download VcXsrv](https://sourceforge.net/projects/vcxsrv/){target=_blank} and
install it to your computer. To be able to see the OAK-1 camera stream that is
sent to the Raspberry Pi on your local computer, you will need to install a X
server if you are communicating with the RPi via SSH. Start the X server with
the `XLaunch.exe` and keep all the default settings:

1.  **Select display settings**:

    - :fontawesome-regular-square-check: Multiple windows
    - Display number: `-1`

2.  **Select how to start clients**:

    - :fontawesome-regular-square-check: Start no client

3.  **Extra settings**:

    - :fontawesome-regular-square-check: Clipboard
        - :fontawesome-regular-square-check: Primary Selection
    - :fontawesome-regular-square-check: Native opengl

After that you will see the VcXsrv tray icon in the taskbar and running e.g.
one of the [preview](programming.md#oak-camera-preview){target=_blank} scripts
will open the OAK-1 camera stream in a new VcXsrv window on your computer.

---

## DiskInternals LinuxReader

[Download DiskInternals LinuxReader](https://www.diskinternals.com/linux-reader/){target=_blank}
and install it to your computer. You can use DiskInternals LinuxReader to
save data from the Raspberry Pi's SD card to your local computer. This is
only necessary if you are not using a Linux-based OS, as the Linux
partition format is not compatible with Windows.

After collecting the microSD card from your camera trap, follow these steps to
save the captured images:

- Insert the Raspberry Pi's microSD card in your card reader and open the program.
  You will see two partitions on the SD card: `boot` and `rootfs`.
- Double-click on the `rootfs` partition and navigate to
  `/home/pi/insect-detect/data` to see all log files and recorded images.
- Select everything you want to download to your PC and click the **Save**
  button in the upper menu bar.
- Keep the **Save Files** option, choose your output folder and select both
  options **Save directory structure** and **Extract file date from metadata**.
  After that, the data you selected will be copied to your computer.

---

## Python

[Download Python 3.11.6](https://www.python.org/downloads/release/python-3116/){target=_blank}
and install it to your computer. The latest version 3.12 is not yet supported
by some packages. You will need Python to run the scripts for
[classification](../deployment/classification.md){target=_blank} of the captured
insect images and subsequent [analysis](../deployment/analysis.md){target=_blank}
and post-processing of the metadata. If you have the [Python](https://bit.ly/2Zm3Ypq){target=_blank}
extension installed in VS Code, you can activate Python in VS Code by pressing
++f1++ to open the Command Palette and running the `Python: Select Interpreter` command.
You can find a
[Getting started tutorial](https://code.visualstudio.com/docs/python/python-tutorial){target=_blank}
and more information on
[Python in VS Code](https://code.visualstudio.com/docs/languages/python){target=_blank}
at the official VS Code Docs.
