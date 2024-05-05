# Components

!!! tip "Choose your Hardware Setup"

    Depending on your field of application, several variations of the hardware
    setup with different scopes and required components are possible. Three
    setups are presented in the following, but you can always create your own!

## Full Setup

For a total cost of about **700 &euro;**, the full hardware setup includes all
necessary components for continuous automated monitoring of insects in the
field during a whole season with the proposed flower platform as background.

With a microSD card size of 256 GB, about 30-50 million images of individual
insects (cropped detections) can be stored, depending on the resolution of the
synchronized HQ frames and the distance from camera to platform. Two batteries
with a combined capacity of ~91Wh enable a continuous monitoring of about
20 hours (e.g. 4 hours/day for 5 days), even if no sunlight is available to
charge the batteries. The weatherproof enclosure which contains all electronic
components is attached to a HPL sheet and can be mounted on a steel or wooden post
with a standard pipe clamp. The flower platform can be printed with the provided
[PDF templates](https://github.com/maxsitt/insect-detect-docs/tree/main/PDF_templates/flower_platform){target=_blank}
or with your custom design and is mounted on the same post.

---

## Minimal Setup

For a total cost of about **530 &euro;**, the minimal hardware setup includes
all necessary components for continuous automated monitoring of insects in the
field with decreased storage/battery capacity and a custom mounting option.

Due to a smaller microSD card size of 32 GB, recorded images and data have to
be collected more frequently. The single battery with a capacity of 12,000mAh
enables a continuous monitoring of about 9 hours, even if no sunlight is available
to charge it. The flower platform and components for mounting the camera trap are
not included, it is assumed that you want to try your own ideas, e.g. set up
the camera trap on a tripod and focus it on a different background.

---

## Test Setup

If you only want to test the basic hardware and software capabilities in the
lab, for about **187 &euro;** you can already get started. As there is no
PiJuice Zero pHAT and battery included in this setup, an additional power
supply will be necessary. To test the
[automated monitoring script](../software/programming.md#automated-monitoring-script){target=_blank}
without the PiJuice Zero pHAT connected to the Raspberry Pi, please use the corresponding
[script](https://github.com/maxsitt/insect-detect/blob/main/yolo_tracker_save_hqsync_nopj.py){target=_blank}
in the [`insect-detect`](https://github.com/maxsitt/insect-detect){target=_blank} GitHub repo.

---

## List of components

??? attention "Component costs"

    All prices are shown with **19% VAT** (for Germany) and **without**
    shipping costs. Depending on your country the VAT may differ and additional
    shipping costs may increase the total cost of the components.

    **The prices for specific components may change and this list is only a
    snapshot in time (October 2023).**

??? info "OAK USB cable"

    A USB 3 Type-A to Type-C cable is normally included in the OAK-1 kit and is
    therefore not listed in the components.

=== "Full Setup"

    | Component                                       | Price              | Product Link                                                                                                               |
    | ----------------------------------------------- | ------------------ | -------------------------------------------------------------------------------------------------------------------------- |
    | OAK-1 Auto-Focus (OpenCV AI Kit)                | 145.99 &euro;      | e.g. [Luxonis shop](https://shop.luxonis.com/collections/usb/products/oak-1?variant=42664380334303){target=_blank} or [Mouser](https://bit.ly/3dl8ukF){target=_blank} |
    | Raspberry Pi Zero 2 W                           | ~22.00 &euro;      | [rpilocator](https://rpilocator.com){target=_blank}                                                                        |
    | Micro SDXC card 256 GB                          | 28.99 &euro;       | e.g. [Conrad](https://bit.ly/3Z5tNsA){target=_blank}                                                                       |
    | Micro USB to USB A adapter, angled              | 5.50 &euro;        | e.g. [Reichelt](https://bit.ly/3SIgDjk){target=_blank}                                                                     |
    | USB A to micro USB cable, 20 cm, angled         | 5.60 &euro;        | e.g. [Reichelt](https://bit.ly/3VnO3VS){target=_blank}                                                                     |
    | RPi CPU Heatsink                                | 1.20 &euro;        | e.g. [Reichelt](https://bit.ly/3zTKIUA){target=_blank}                                                                     |
    | RPi Header                                      | 0.95 &euro;        | e.g. [Reichelt](https://bit.ly/3SIdSyu){target=_blank}                                                                     |
    | RPi Stacking Header                             | 1.60 &euro;        | e.g. [Reichelt](https://bit.ly/3VeXdDW){target=_blank}                                                                     |
    | RPi Spacer Bolts 10 mm                          | 3.50 &euro;        | e.g. [Reichelt](https://bit.ly/3zVuSbM){target=_blank}                                                                     |
    | RPi Spacer Bolts 20 mm                          | 2.60 &euro;        | e.g. [Reichelt](https://bit.ly/3QCdFuT){target=_blank}                                                                     |
    | **2x** *Thermal Pad (1 mm) 50x50 mm* (optional) | *4.79 &euro; x2*   | e.g. [Conrad](https://shorturl.at/BSXZ4){target=_blank}                                                                    |
    | PiJuice Zero UPS pHAT                           | 54.23 &euro;       | e.g. [Pi Supply](https://bit.ly/3rMiKXb){target=_blank} or [Distrelec](https://bit.ly/3dtoC3K){target=_blank}              |
    | PiJuice 12,000mAh LiPo Battery                  | 39.40 &euro;       | e.g. [Pi Supply](https://bit.ly/3fT0Sak){target=_blank} or [Distrelec](https://bit.ly/3vZGiKw){target=_blank}              |
    | Fibox PC 162513 Enclosure                       | 89.99 &euro;       | e.g. [Conrad](https://bit.ly/3hOlOR3){target=_blank} or [Distrelec](https://bit.ly/3JX82FC){target=_blank}                 |
    | Fibox TM 1625 Mounting Plate                    | 7.49 &euro;        | e.g. [Conrad](https://bit.ly/3BVGf5t){target=_blank} or [Distrelec](https://bit.ly/3QmVhqf){target=_blank}                 |
    | Cable Gland PG 13.5                             | 0.99 &euro;        | e.g. [Reichelt](https://bit.ly/3LDfV5q){target=_blank}                                                                     |
    | Cable Gland PG 13.5 Locknut                     | 0.27 &euro;        | e.g. [Reichelt](https://bit.ly/3HndDVp){target=_blank}                                                                     |
    | **8x** Stainless Steel Cable Tie                | 3.90 &euro;        | e.g. [Reichelt](https://bit.ly/3LhR7OS){target=_blank}                                                                     |
    | **6x** Cable Tie Mount                          | 0.99 &euro; **x6** | e.g. [Reichelt](https://bit.ly/3Nm92qw){target=_blank}                                                                     |
    | Acrylic Glas (2 mm) 40x40 mm                    | ~5.00 &euro;       | e.g. [S-Polytec](https://bit.ly/3dkMGWh){target=_blank}                                                                    |
    | EPDM Sealing Strip 20x5 mm                      | 6.25 &euro;        | e.g. [Amazon](https://amzn.to/3pcBM7J){target=_blank}                                                                      |
    | Silica Gel Pack 50 g                            | 15.99 &euro;       | e.g. [Amazon](https://amzn.to/3JR5zfV){target=_blank}                                                                      |
    | Solar Panel 6V 9W                               | 98.30 &euro;       | e.g. [Voltaic Systems](https://bit.ly/3VdIyZN){target=_blank} or [Kiwi Electronics](https://bit.ly/3QI5AVl){target=_blank} |
    | *Solar Panel Bracket, Medium* (optional)        | *11.75 &euro;*     | e.g. [Voltaic Systems](https://bit.ly/3Ths3tK){target=_blank} or [Kiwi Electronics](https://bit.ly/3v5NWCb){target=_blank} |
    | Solar Panel Extension Cable, 1 ft               | ~7.00 &euro;       | e.g. [Voltaic Systems](https://bit.ly/3CRUGZm){target=_blank} or [Funky Leisure](https://bit.ly/3CTGAqs){target=_blank}    |
    | Solar Panel Micro USB Adapter                   | 3.88 &euro;        | e.g. [Voltaic Systems](https://bit.ly/3s237et){target=_blank} or [Kiwi Electronics](https://bit.ly/3Ew9FJB){target=_blank} |
    | Voltaic 12,800mAh Li-Ion Battery                | 83.55 &euro;       | e.g. [Voltaic Systems](https://bit.ly/3MjlGDU){target=_blank} or [Kiwi Electronics](https://bit.ly/3Z4dFI0){target=_blank} |
    | **2x** *Heatsink 40x30 mm* (optional)           | *2.35 &euro; x2*   | e.g. [Reichelt](https://bit.ly/3SIrxFM){target=_blank}                                                                     |
    | HPL Sheet (4 mm) 350x250 mm                     | ~5.00 &euro;       | e.g. [HPL shop](https://bit.ly/3C5zsqK){target=_blank}                                                                     |
    | Aluminium Square Tube (23.5x23.5 mm), 1.5 m     | ~16.00 &euro;      | Local hardware store (e.g. [Bauhaus](https://bit.ly/3duFdUV){target=_blank})                                               |
    | **4x** M4 20 mm Screws (internal hexagon)       | ~2.00 &euro;       | Local hardware store                                                                                                       |
    | **7x** M4 40 mm Screws                          | ~3.00 &euro;       | Local hardware store                                                                                                       |
    | **2x** M4 60 mm Screws                          | ~2.00 &euro;       | Local hardware store                                                                                                       |
    | **21x** M4 Hex Nuts                             | ~3.00 &euro;       | Local hardware store                                                                                                       |
    | **18x** Flat Washer (e.g. M4 15 mm)             | ~3.00 &euro;       | Local hardware store (e.g. [Bauhaus](https://bit.ly/3VgD7Jn){target=_blank})                                               |
    | Pipe Clamp (60.3 mm) 70 mm + screws & nuts      | 5.09 &euro;        | e.g. [Traffic sign shop](https://bit.ly/3zVOYCK){target=_blank}                                                            |
    | Pipe Clamp (60.3 mm) 350 mm + screws & nuts     | 7.81 &euro;        | e.g. [Traffic sign shop](https://bit.ly/3zVOYCK){target=_blank}                                                            |
    | Flower Platform (e.g. 350x200 mm)               | ~17.00 &euro;      | e.g. [Lightweight foam board](https://bit.ly/3QlFJTA){target=_blank} or [Acrylic glass](https://www.wir-machen-druck.de/acrylplatte-mit-echtglasbeschichtung-in-freier-groesse-rechteckig-einseitig-40farbig-bedruckt.html){target=_blank} |
    | **Total cost**                                  | **~704 &euro;**    |                                                                                                                            |
    | *Total cost with optional components*           | *~730 &euro;*      |                                                                                                                            |

=== "Minimal Setup"

    | Component                                | Price              | Product Link                                                                                                               |
    | ---------------------------------------- | ------------------ | -------------------------------------------------------------------------------------------------------------------------- |
    | OAK-1 (OpenCV AI Kit)                    | 145.99 &euro;      | e.g. [Luxonis shop](https://bit.ly/3Ew7PbM){target=_blank} or [Mouser](https://bit.ly/3dl8ukF){target=_blank}              |
    | Raspberry Pi Zero 2 W                    | ~22.00 &euro;      | [rpilocator](https://rpilocator.com){target=_blank}                                                                        |
    | Micro SDHC Card 32 GB                    | 8.95 &euro;        | e.g. [Reichelt](https://bit.ly/3QghI06){target=_blank}                                                                     |
    | Micro USB to USB A Adapter, angled       | 5.50 &euro;        | e.g. [Reichelt](https://bit.ly/3SIgDjk){target=_blank}                                                                     |
    | RPi CPU Heatsink                         | 1.20 &euro;        | e.g. [Reichelt](https://bit.ly/3zTKIUA){target=_blank}                                                                     |
    | RPi Header                               | 0.95 &euro;        | e.g. [Reichelt](https://bit.ly/3SIdSyu){target=_blank}                                                                     |
    | RPi Stacking Header                      | 1.60 &euro;        | e.g. [Reichelt](https://bit.ly/3VeXdDW){target=_blank}                                                                     |
    | RPi Spacer Bolts 10 mm                   | 3.50 &euro;        | e.g. [Reichelt](https://bit.ly/3zVuSbM){target=_blank}                                                                     |
    | RPi Spacer Bolts 20 mm                   | 2.60 &euro;        | e.g. [Reichelt](https://bit.ly/3QCdFuT){target=_blank}                                                                     |
    | *Thermal Pad (1 mm) 50x50 mm* (optional) | *4.79 &euro;*      | e.g. [Conrad](https://shorturl.at/BSXZ4){target=_blank}                                                                    |
    | PiJuice Zero UPS pHAT                    | 54.23 &euro;       | e.g. [Pi Supply](https://bit.ly/3rMiKXb){target=_blank} or [Distrelec](https://bit.ly/3dtoC3K){target=_blank}              |
    | PiJuice 12,000mAh LiPo Battery           | 39.40 &euro;       | e.g. [Pi Supply](https://bit.ly/3fT0Sak){target=_blank} or [Distrelec](https://bit.ly/3vZGiKw){target=_blank}              |
    | Fibox PC 162513 Enclosure                | 89.99 &euro;       | e.g. [Conrad](https://bit.ly/3hOlOR3){target=_blank} or [Distrelec](https://bit.ly/3JX82FC){target=_blank}                 |
    | Fibox TM 1625 Mounting Plate             | 7.49 &euro;        | e.g. [Conrad](https://bit.ly/3BVGf5t){target=_blank} or [Distrelec](https://bit.ly/3QmVhqf){target=_blank}                 |
    | Cable Gland PG 13.5                      | 0.99 &euro;        | e.g. [Reichelt](https://bit.ly/3LDfV5q){target=_blank}                                                                     |
    | Cable Gland PG 13.5 Locknut              | 0.27 &euro;        | e.g. [Reichelt](https://bit.ly/3HndDVp){target=_blank}                                                                     |
    | **6x** Stainless Steel Cable Tie         | 3.90 &euro;        | e.g. [Reichelt](https://bit.ly/3LhR7OS){target=_blank}                                                                     |
    | **6x** Cable Tie Mount                   | 0.99 &euro; **x6** | e.g. [Reichelt](https://bit.ly/3Nm92qw){target=_blank}                                                                     |
    | Acrylic Glas (2 mm) 40x40 mm             | ~5.00 &euro;       | e.g. [S-Polytec](https://bit.ly/3dkMGWh){target=_blank}                                                                    |
    | EPDM Sealing Strip 20x5 mm               | 6.25 &euro;        | e.g. [Amazon](https://amzn.to/3pcBM7J){target=_blank}                                                                      |
    | Silica Gel Pack 50 g                     | 15.99 &euro;       | e.g. [Amazon](https://amzn.to/3JR5zfV){target=_blank}                                                                      |
    | Solar Panel 6V 9W                        | 98.30 &euro;       | e.g. [Voltaic Systems](https://bit.ly/3VdIyZN){target=_blank} or [Kiwi Electronics](https://bit.ly/3QI5AVl){target=_blank} |
    | *Solar Panel Bracket, Medium* (optional) | *11.75 &euro;*     | e.g. [Voltaic Systems](https://bit.ly/3Ths3tK){target=_blank} or [Kiwi Electronics](https://bit.ly/3v5NWCb){target=_blank} |
    | Solar Panel Extension Cable, 1 ft        | ~7.00 &euro;       | e.g. [Voltaic Systems](https://bit.ly/3CRUGZm){target=_blank} or [Funky Leisure](https://bit.ly/3CTGAqs){target=_blank}    |
    | Solar Panel Micro USB Adapter            | 3.88 &euro;        | e.g. [Voltaic Systems](https://bit.ly/3s237et){target=_blank} or [Kiwi Electronics](https://bit.ly/3Ew9FJB){target=_blank} |
    | **Total cost**                           | **~531 &euro;**    |                                                                                                                            |
    | *Total cost with optional components*    | *~547 &euro;*      |                                                                                                                            |

=== "Test Setup"

    | Component             | Price             | Product Link                                                                                                  |
    | --------------------- | ----------------- | ------------------------------------------------------------------------------------------------------------- |
    | OAK-1 (OpenCV AI Kit) | 145.99 &euro;     | e.g. [Luxonis shop](https://bit.ly/3Ew7PbM){target=_blank} or [Mouser](https://bit.ly/3dl8ukF){target=_blank} |
    | Raspberry Pi Zero 2 W | ~22.00 &euro;     | [rpilocator](https://rpilocator.com){target=_blank}                                                           |
    | Micro SDHC Card 32 GB | 8.95 &euro;       | e.g. [Reichelt](https://bit.ly/3QghI06){target=_blank}                                                        |
    | RPi Micro USB Adapter | 1.85 &euro;       | e.g. [Reichelt](https://bit.ly/3pbJPlp){target=_blank}                                                        |
    | RPi Power Supply      | 7.99 &euro;       | e.g. [Reichelt](https://bit.ly/3w30QBV){target=_blank}                                                        |
    | **Total cost**        | **~187 &euro;**   |                                                                                                               |

---

## Alternative components

While the
[OAK-1](https://docs.luxonis.com/projects/hardware/en/latest/pages/BW1093.html){target=_blank}
camera is recommended for the Insect Detect camera trap, other versions with
different image sensors are available. The cheaper
[OAK-1 Lite](https://docs.luxonis.com/projects/hardware/en/latest/pages/NG9096.html){target=_blank}
has a lower quality image sensor, which might be good enough depending on your use case.
While the image quality is similar to the OAK-1 under optimal conditions, it will get
noticeably worse under challenging conditions (e.g. low light). The more expensive
[OAK-1 MAX](https://docs.luxonis.com/projects/hardware/en/latest/pages/NG9096max.html){target=_blank}
with a 48MP sensor can take still images with a resolution of up to 5312x6000 pixel.
However, the video output that is used for the HQ frames in the
[automated monitoring script](../software/programming.md#automated-monitoring-script){target=_blank}
is limited to a maximum resolution of 3840x2160 pixel (4K). Therefore the OAK-1 MAX
will only be worth the higher price if you are using e.g. the
[still capture script](../software/programming.md#still-capture){target=_blank}
to take still images at the highest possible sensor resolution. The also more expensive
[OAK-1 W](https://docs.luxonis.com/projects/hardware/en/latest/pages/NG9096w.html){target=_blank}
includes a wide FOV 120° sensor, which might be beneficial depending on your camera
trap setup. This version was not tested, so there is no guarantee that all provided
Python scripts will work in the same way as for the OAK-1, OAK-1 Lite and OAK-1 MAX.

The following table shows all compatible Raspberry Pi and OAK versions, that
were tested with the Insect Detect camera trap
[software](https://github.com/maxsitt/insect-detect){target=_blank}.

| Raspberry Pi | OAK-1                   | OAK-1 Lite              | OAK-1 MAX               | OAK-D Lite              |
| ------------ | ----------------------- | ----------------------- | ----------------------- | ----------------------- |
| Zero 2 W     | :material-check-circle: | :material-check-circle: | :material-check-circle: | :material-check-circle: |
| Zero W       | :material-check-circle: | :material-check-circle: | :material-check-circle: | :material-check-circle: |
| 4 Model B    | :material-check-circle: | :material-check-circle: | :material-check-circle: | :material-check-circle: |

The [Witty Pi 4 Mini](https://www.uugear.com/product/witty-pi-4-mini/){target=_blank}
or the newer version
[Witty Pi 4 L3V7](https://www.uugear.com/product/witty-pi-4-l3v7/){target=_blank},
with the ability to connect 3.7V Li-Ion or LiPo batteries, could be used as
alternative component to the PiJuice Zero pHAT, especially if your camera trap
is connected to the power grid without the need of solar power input and the
power management capabilities of the PiJuice Zero. Both were not tested, so no
guarantee can be given for their full functionality.
