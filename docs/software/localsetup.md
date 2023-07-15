# Local Software Setup

???+ warning "Local OS"

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
recommended. For casual users, install the VS Code extensions listed under `Basic`.

For experienced users, connecting and working exclusively via the Terminal
(e.g. Windows PowerShell) is also possible. If you want to use the Raspberry
Pi Zero 2 W as a remote development environment, install the VS Code extensions
listed under `Advanced` (**Not supported by RPi Zero W (v1)!**).

???+ info "Recommended Extensions"

    === "Basic"

        - [SSH FS](https://marketplace.visualstudio.com/items?itemName=Kelvin.vscode-sshfs){target=_blank}
          extension to mount the `/home/pi` folder as local workspace folder in VS Code.
        - [Excel Viewer](https://bit.ly/3DNAsjM){target=_blank} extension
          for a more readable preview of the .csv metadata and log files.

    === "Advanced"

        - [Remote - SSH](https://bit.ly/3dw6tSI){target=_blank} extension to
          use the Raspberry Pi Zero 2 W as a remote development environment
          with more functions, including Pylint support.
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
the `XLaunch.exe` and keep all the default settings. After that you will see a
tray icon in the taskbar and running e.g. one of the
[preview](programming.md#oak-camera-preview) scripts will open the OAK-1 camera
stream in a new VcXsrv window on your local computer.

---

## DiskInternals LinuxReader

[Download DiskInternals LinuxReader](https://www.diskinternals.com/linux-reader/){target=_blank}
and install it to your computer. You will use DiskInternals LinuxReader to
save data from the Raspberry Pi's SD card to your local computer. This is
only necessary if you are not using a Linux-based OS, as the Linux
partition format is not compatible with Windows.

- Insert the Raspberry Pi's microSD card in your card reader and open the program.
  You will see two partitions on the SD card: `boot` and `rootfs`.
- Double-click on the `rootfs` partition and navigate to
  `/home/pi/insect-detect/data` to see all log files and recorded images.
- Select everything you want to download to your PC and click the **Save**
  button in the upper menu bar.
- Keep the **Save Files** option, choose your output folder and select both
  options **Save directory structure** and **Extract file date from metadata**.
  After that, the data you selected will be copied to your computer.
