# Local Setup

??? warning "Local OS"

    The following instructions and software were only tested on a **Windows 11**
    computer. For macOS or Linux users, some steps and software versions might
    be different. If you run into any problems with the software setup, please
    [post an issue](https://github.com/maxsitt/insect-detect-docs/issues){target=_blank}
    to the GitHub repo.

## Raspberry Pi Imager

Download [Raspberry Pi Imager](https://www.raspberrypi.com/software/){target=_blank}
and install it to your computer. We will use it to set up the microSD card with
Raspberry Pi OS and do some initial [configuration](pisetup.md#install-raspberry-pi-os){target=_blank}.

---

## Visual Studio Code

Download [Visual Studio Code](https://code.visualstudio.com/){target=_blank}
and install it to your computer. Install the following
[extensions](https://code.visualstudio.com/docs/editor/extension-marketplace){target=_blank}:

- [Python](https://marketplace.visualstudio.com/items?itemName=ms-python.python){target=_blank}
  to work with Python scripts.
- [SSH FS](https://marketplace.visualstudio.com/items?itemName=Kelvin.vscode-sshfs){target=_blank}
  to access the Raspberry Pi file explorer.
- [Rainbow CSV](https://marketplace.visualstudio.com/items?itemName=mechatroner.rainbow-csv){target=_blank} and/or
  [Excel Viewer](https://marketplace.visualstudio.com/items?itemName=GrapeCity.gc-excelviewer){target=_blank}
  for a more readable preview of the generated .csv files.

---

## Python

Download [Python 3.12.10](https://www.python.org/downloads/release/python-31210/){target=_blank}
and install it to your computer (3.13 is not yet supported by some packages). Select
[Install Now](https://docs.python.org/3.12/using/windows.html#the-full-installer){target=_blank}
with the checkbox `Add Python to PATH` activated. Run `Disable path length limit`
at the end of the installation to remove the
[MAX_PATH limitation](https://docs.python.org/3.12/using/windows.html#removing-the-max-path-limitation){target=_blank}.

You will need Python to run the scripts for
[classification](../deployment/classification.md){target=_blank} of the captured
insect images and subsequent metadata [post-processing](../deployment/post-processing.md){target=_blank}.

We are going to use the
[Python Launcher](https://docs.python.org/3.12/using/windows.html#python-launcher-for-windows){target=_blank}
for Windows to run Python scripts with the `py` command. You can check if the
Python Launcher works correctly and display the installed Python version(s),
by opening a Terminal (e.g. Windows PowerShell) and running:

``` powershell
py --list
```

After installing the [Python](https://marketplace.visualstudio.com/items?itemName=ms-python.python){target=_blank}
extension, activate Python in VS Code by opening the Command Palette with
++ctrl+shift+p++ and running the `Python: Select Interpreter` command. You can
find more information including a tutorial at
[Python in Visual Studio Code](https://code.visualstudio.com/docs/languages/python){target=_blank}.

---

## DiskInternals Linux Reader

Download [DiskInternals Linux Reader](https://www.diskinternals.com/linux-reader/){target=_blank}
and install it to your computer. You can use it to save data from the Raspberry
Pi's microSD card to your local computer, as the Linux partition format is not
compatible with Windows and you will only see the boot partition in your file explorer.

Follow these steps to save the captured data to your computer:

- Insert the Raspberry Pi's microSD card into your card reader and start DiskInternals
  Linux Reader. You will see two partitions on the microSD card: `rootfs` and `bootfs`.
- Open the `rootfs` partition and navigate to the `/home/pi` directory.
- Select the `insect-detect` directory and hit the **Save** button in the upper menu bar.
- Keep the **Save Files** option, choose your output folder and select both options:

    :fontawesome-regular-square-check: Save directory structure

    :fontawesome-regular-square-check: Extract file date from metadata
