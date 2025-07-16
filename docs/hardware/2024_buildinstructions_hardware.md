# Hardware Integration

??? abstract "Required Tools & Components"

    **Tools**

    1. Soldering Iron (+ Solder)
    2. Mounting Putty (e.g. patafix or Blu Tack)
    3. Crosshead Screwdrivers (small for M2.5 screws + large for enclosure screws)
    4. Flathead Screwdriver (to open electronics enclosure lid)
    5. Pliers
    6. Scissors
    7. Wire Stripper Tool (or scissors if not available)

    **Components**

    1. Electronics Enclosure (with drilled holes)
    2. **2x** Mounting Plate Electronics Enclosure (with drilled holes)
    3. **2x** Mounting Plate Screw Set B M-SHR
    4. Raspberry Pi Zero 2 WH
    5. Raspberry Pi CPU Heatsink
    6. Raspberry Pi Stacking GPIO Header
    7. Micro-USB to USB-A Adapter
    8. Witty Pi 4 L3V7
    9. Single Row Pin Header, angled (7 pins)
    10. **4x** Standoff M2.5, female-male (5 mm)
    11. **4x** Standoff M2.5, female-female (11-12 mm)
    12. **8x** Screw M2.5 (5 mm)
    13. Li-Ion Battery Pack 3.7V (4400mAh)
    14. Voltaic V75 USB Battery Pack + USB-C Cable
    15. **8x** Cable Tie, 3.6 mm (length: 250+ mm)
    16. **2x** Binding Wire (e.g. from OAK/Voltaic USB cables)
    17. **2x** Silica Gel Pack (5 g)
    18. **4x** Jumper Wire, 1-2 female ends (20 cm)
    19. **4x** WAGO Inline Splicing Connector
    20. LED Push Button (3V) + Socket
    21. **2x** Cable Gland MBF 20-RJ45
    22. **2x** Cable Gland Counter Nut M20
    23. **2x** Cable Gland Sealing Ring M20
    24. **2x** Ventilation Plug M6
    25. Cable Gland M20
    26. Cable Gland Counter Nut M20
    27. Cable Gland Sealing Ring M20
    28. Voltaic 3.5x1.1mm USB-C Adapter
    29. Voltaic 3.5x1.1mm Cable (30 cm)
    30. Camera Enclosure (with drilled holes)
    31. Mounting Plate Camera Enclosure (with drilled holes)
    32. OAK-1 (Auto-Focus) + USB-C Cable
    33. **2x** Screw M4, internal hexagon (16 mm)
    34. **2x** Hex Nut M4, flat

## 1 Electronics Enclosure

We will start with integrating the hardware into the larger electronics enclosure.
Don't connect the lid yet, as we will do this later on when everything is in place.

<figure markdown="span">
  ![Electronics Enclosure Hardware](assets/images/2024_electronics_enclosure_hardware.jpg){ width="400" }
  <figcaption>Electronics enclosure with fully integrated hardware</figcaption>
</figure>

### 1.1 Solder Headers

If you don't already have a Raspberry Pi Zero 2 WH with attached header, you
will have to solder a header to it. Stick the heatsink on the CPU of your
Raspberry Pi to keep the header sitting flush as you solder it. If this is
your first time soldering, you can find detailed instructions
[here](https://magpi.raspberrypi.com/articles/how-to-solder-gpio-pin-headers-to-raspberry-pi-pico){target=_blank}.
As an alternative to soldering the header, you could also use a
[Hammer Header](https://shop.pimoroni.com/products/gpio-hammer-header){target=_blank}.

If you want to use the LED button, you will need to solder a 7-pin angled header
to the Witty Pi 4 L3V7 to connect the LED button to it. Break off seven connected
pins from the angled single row pin header with pliers. Use some kind of
mounting putty (e.g. patafix or Blu Tack) to hold the Witty Pi board and the
7-pin header in place while soldering.

![Witty Pi Soldering Header](assets/images/2024_witty_pi_soldering_header.jpg){ width="700" }

---

### 1.2 Bottom Mounting Plate

In this step, we will mount the Raspberry Pi Zero 2 WH, Witty Pi 4 L3V7 and the
3.7V Li-Ion battery pack to the bottom mounting plate of the electronics enclosure.

!!! info ""

    :material-circle:{ .circle_green } Bocube Mounting Plate B M 2213 (bottom) with drilled holes

    :material-circle:{ .circle_blue } Raspberry Pi Zero 2 WH (+ header and attached heatsink)

    :material-circle:{ .circle_orange } Stacking GPIO Header

    :material-circle:{ .circle_violet } Witty Pi 4 L3V7 (+ 7-pin angled header to connect LED button)

    :material-circle:{ .circle_red } **4x** Standoff M2.5 female-male (5 mm), **4x** Standoff M2.5 female-female (11-12 mm), **8x** Screw M2.5 (5 mm)

    :material-circle:{ .circle_yellow } Small Crosshead Screwdriver (not shown in image)

![Overview Mounting Plate Bottom](assets/images/2024_overview_mounting_plate_bottom.jpg){ width="700" }

Push the stacking header on the GPIO header of the Raspberry Pi and make sure that it sits flush.

![Raspberry Pi Stacking Header](assets/images/2024_raspberrypi_stacking_header.jpg){ width="500" }

![Raspberry Pi Stacking Header Attached](assets/images/2024_raspberrypi_stacking_header_attached.jpg){ width="500" }

Push the M2.5 screws through the four smaller 3 mm holes in the mounting plate.
While holding the screws in place, fix the 5 mm standoffs to the plate from the
other side. Don't fasten them too tight yet, we will do this later.

![Mounting Plate Bottom Screws](assets/images/2024_mounting_plate_bottom_screws.jpg){ width="600" }

![Mounting Plate Bottom Standoffs](assets/images/2024_mounting_plate_bottom_standoffs.jpg){ width="600" }

Place the Raspberry Pi on the standoffs with the stacking header facing the
four holes at the edge and make sure that it sits flush. If it doesn't fit,
loosen one or several of the standoffs and adjust their position, until the
Raspberry Pi slides onto them.

![Mounting Plate Bottom Raspberry Pi](assets/images/2024_mounting_plate_bottom_rpi.jpg){ width="600" }

Screw all four 11 (or 12) mm female-female standoffs onto the 5 mm standoffs.

![Mounting Plate Bottom Raspberry Pi Standoffs](assets/images/2024_mounting_plate_bottom_rpi_standoffs.jpg){ width="600" }

Push the Witty Pi board onto the stacking header. You will have to use
a little bit of force, but be careful to push only at the corners of the
board without touching the electronic components.

![Mounting Plate Bottom Witty Pi](assets/images/2024_mounting_plate_bottom_wittypi.jpg){ width="600" }

The Witty Pi board should ideally sit flush on the stacking header as shown in
the following image. If you used 12 mm instead of 11 mm standoffs or a different
kind of stacking header, the GPIO pins might be a little bit exposed under the
Witty Pi board which is okay.

![Mounting Plate Bottom Witty Pi Flush](assets/images/2024_mounting_plate_bottom_wittypi_flush.jpg){ width="600" }

Use the remaining four M2.5 screws to fasten the Witty Pi to the Raspberry Pi.
Turn the mounting plate and also fasten the screws at the underside tightly.

![Mounting Plate Bottom Witty Pi Fixed](assets/images/2024_mounting_plate_bottom_wittypi_fixed.jpg){ width="600" }

Next, we will fix the battery pack to the mounting plate.

![Overview Mounting Plate Bottom Battery Pack](assets/images/2024_overview_mounting_plate_bottom_battery.jpg){ width="750" }

Push the cable ties through the 4 mm holes at the edge of the plate, with the
opening of the head oriented towards the Witty Pi. Then push them through the
other two holes from underneath the plate. Align the position of the cable tie
heads with the height of the battery pack (~4 cm).

![Mounting Plate Bottom Battery Pack Cable Ties](assets/images/2024_mounting_plate_bottom_battery_cable_ties.jpg){ width="600" }

Place the battery pack between the cable ties with the cable oriented
towards the white connector on the Witty Pi and align it with both boards.
Fasten the cable ties, while holding the battery pack in an upright position.

![Mounting Plate Bottom Battery Pack Fixed](assets/images/2024_mounting_plate_bottom_battery_fixed.jpg){ width="600" }

Cut off the protruding ends of the cable ties. Connect the battery pack cable
to the Witty Pi. Optionally, put the cable behind the cable tie head for cleaner
cable management.

![Mounting Plate Bottom Battery Pack Cable](assets/images/2024_mounting_plate_bottom_battery_cable.jpg){ width="400" }

To attach the silica gel pack to the mounting plate, it is recommended to use
something that can be easily reopened to replace a saturated pack with a new
one (e.g. releasable cables ties).

In our example, we will reuse the black binding wires that came with the USB
cables of the OAK camera and V75 battery (keep the second one for the camera
enclosure). Push both ends of one wire through the two remaining holes from
underneath the plate and slightly twist them for fixing. Use the wire to attach
the silica gel pack to the mounting plate only shortly before field deployment
to keep it as effective as possible.

![Mounting Plate Bottom Silica Gel Pack Wire](assets/images/2024_mounting_plate_bottom_silica_pack_wire.jpg){ width="600" }

![Mounting Plate Bottom Silica Gel Pack](assets/images/2024_mounting_plate_bottom_silica_pack.jpg){ width="600" }

---

### 1.3 LED Button

**Continue with [1.4 Cable Glands + Vent Plug](#14-cable-glands-vent-plug) if you are not using the LED Button!**

We will start by preparing the jumper wires that we are going to use in the next
steps to connect the LED button to the Witty Pi and Raspberry Pi. The jumper
wires come in different colors which can be used for orientation. Strip off two
strands of two still connected wires.

![Jumper Wires](assets/images/2024_jumper_wires.jpg){ width="600" }

Cut off the connectors at one end of the wire, while keeping the female connectors.

![Jumper Wires Cut](assets/images/2024_jumper_wires_cut.jpg){ width="600" }

For the next step, you will need to strip about 1 cm of the insulation from the
wires. You can do this carefully with normal scissors or use a specific wire
stripper tool.

![Jumper Wires Strip Tools](assets/images/2024_jumper_wires_strip_tools.jpg){ width="600" }

Remove about 1 cm of the insulation at the previously cut cable ends.

![Jumper Wires Strip](assets/images/2024_jumper_wires_strip.jpg){ width="600" }

Twist the open wire ends with your fingers to make them more robust.

![Jumper Wires Twist](assets/images/2024_jumper_wires_twist.jpg){ width="600" }

Open the latches on both sides of the four inline splicing connectors.

![Jumper Wires Connectors](assets/images/2024_jumper_wires_connectors.jpg){ width="600" }

Insert the open wire ends into each of the connectors until they hit the back wall.
While holding the wires in this position, close the latches to secure them.

![Jumper Wires Connectors Closed](assets/images/2024_jumper_wires_connectors_closed.jpg){ width="700" }

For the next steps, prepare the disassembled LED Button (button, sealing ring,
counter nut, socket), the jumper wires + connectors and optionally a short cable tie.

![Overview Jumper Wires Button](assets/images/2024_overview_jumper_wires_button.jpg){ width="750" }

Put the sealing ring onto the thread of the button.

![Button Sealing Ring](assets/images/2024_button_sealing_ring.jpg){ width="600" }

!!! warning "Different Button Types"

    Please check the cable colors and their respective assignment for your
    specific product on either the button itself or the button socket. Even if
    the button looks identical, the cable assignments/colors can be different
    depending on the producer or batch.

Take a look at the socket and note the colors of the cables that are labeled
with **C** and **NO**. In our case this is green for **C** and yellow for **NO**.
We won't use the white cable labeled with **NC**.

The green **C** cable will be connected to the **GND** (Ground) pin and the
yellow **NO** cable will be connected to the **SWITCH** pin of the Witty Pi board.
The red cable is the **+3V** supply and the black cable is the **Ground** for the
LED ring. We will connect both to the Raspberry Pi GPIO pins.

![Overview Button Wires](assets/images/2024_overview_button_wires.jpg){ width="600" }

Take off the insulation on the ends of the button cables and twist the open ends.
Cut off the **NC** cable (white in our case) to close its end as we won't need it.

![Button Wires Twist](assets/images/2024_button_wires_twist.jpg){ width="600" }

Guide all button cables under the Raspberry Pi between the standoffs, as shown
in the following image. Make sure that the latch of the socket is pointing upwards.

![Button Wire Management](assets/images/2024_button_wire_management.jpg){ width="700" }

Connect the **+3V** (red) and **Ground** (black) button cables to one pair of
the jumper wires and the **C** (green) and **NO** (yellow) cables to the other
pair. Push the cables completely inside the connectors until they hit the back
wall and hold them into position while closing the latches.

![Button Wires Connectors](assets/images/2024_button_wires_connectors.jpg){ width="700" }

For better cable management you can optionally use a cable tie to keep all four
connectors together. Make sure to orient the latches to the outside of your
bundle that the connectors can still be opened later on if necessary.

![Button Connectors Cable Tie](assets/images/2024_button_connectors_cable_tie.jpg){ width="600" }

Push the female end of the jumper wire that is connected to the **C** (green) cable
of the button onto the left angled pin (**GND**) on the Witty Pi board. Push the
female end of the jumper wire that is connected to the **NO** (yellow) cable of the
button onto the third angled pin (**SWITCH**).

![Button Wires Witty Pi](assets/images/2024_button_wires_wittypi.jpg){ width="600" }

Push the female end of the jumper wire that is connected to the **+3V** (red) LED
cable of the button onto the sixth pin on the outer GPIO row of the Raspberry Pi
(= GPIO 18). Push the female end of the jumper wire that is connected to the **Ground**
(black) LED cable of the button onto the seventh pin in the same row (= Ground).

![Button Wires LED Raspberry Pi](assets/images/2024_button_wires_led_raspberrypi.jpg){ width="600" }

Insert the mounting plate into the enclosure with the USB ports facing towards
the drilled holes and fix it with the four mounting plate screws.

![Mounting Plate Bottom Fixed](assets/images/2024_mounting_plate_bottom_fixed.jpg){ width="600" }

Push the LED button from the outside of the enclosure through the 19 mm hole on the left.

![Enclosure Button Integration](assets/images/2024_enclosure_button_integration.jpg){ width="600" }

First, secure the button with the counter nut. Try to close it as tight as
possible while pressing the sealing ring against the outer wall of the enclosure.
Make sure to align the connectors of the button with the corresponding slots
in the button socket and push the socket onto the button with the latch of
the socket aligned with its counterpart on the button.

![Enclosure Button Fixed](assets/images/2024_enclosure_button_fixed.jpg){ width="600" }

---

### 1.4 Cable Glands + Vent Plug

Insert the mounting plate into the enclosure with the USB ports facing towards
the drilled holes and fix it with the four mounting plate screws if you skipped
the LED button integration.

!!! info ""

    :material-circle:{ .circle_green } Cable Gland MBF 20-RJ45 + Sealing Ring M20 + Counter Nut M20

    :material-circle:{ .circle_blue } Ventilation Plug M6

    :material-circle:{ .circle_orange } Micro-USB to USB-A Adapter

    :material-circle:{ .circle_violet } OAK-1 USB-C to USB-A cable

![Overview Cable Gland](assets/images/2024_overview_cable_gland.jpg){ width="750" }

Open the cable gland and use a screwdriver or pencil to push out the slotted sealing insert.

![Cable Gland Disassembled](assets/images/2024_cable_gland_disassembled.jpg){ width="600" }

Guide the USB-C end of the OAK-1 camera cable through the cable gland from the
threaded side.

![Cable Gland USB Cable](assets/images/2024_cable_gland_usb_cable.jpg){ width="600" }

Open the slot of the sealing insert and put it around the USB cable.

![Cable Gland USB Cable Slot](assets/images/2024_cable_gland_usb_cable_slot.jpg){ width="600" }

![Cable Gland USB Cable Sealing](assets/images/2024_cable_gland_usb_cable_sealing.jpg){ width="600" }

Push the sealing insert back into the cable gland completely. Screw the cable
gland nut a little bit on the cable gland. Don't fasten it yet, the USB cable
has to move freely.

![Cable Gland USB Cable Sealing Fixed](assets/images/2024_cable_gland_usb_cable_sealing_fixed.jpg){ width="600" }

Slide the cable gland to the USB-A side of the camera cable. Connect the
Micro-USB to USB-A adapter. Prepare the cable gland sealing ring and counter nut.

![Cable Gland Electronics Enclosure](assets/images/2024_cable_gland_electronics_enclosure.jpg){ width="600" }

First, put the sealing ring onto the thread of the cable gland. Then insert the
USB cable through the 20 mm hole in the center of the enclosure. From the inside,
guide the cable through the cable gland counter nut and connect the adapter to
the inner Micro-USB port of the Raspberry Pi.

![Cable Gland Electronics Enclosure Fixing](assets/images/2024_cable_gland_electronics_enclosure_fixing.jpg){ width="700" }

Fasten the cable gland counter nut tightly from inside the enclosure while
pushing the sealing ring to the outer wall of the enclosure. Be careful to
keep the USB cable in position so that it doesn't lose connection to the
Raspberry Pi. Finally, also fasten the cable gland nut outside of the enclosure
tightly, so that the sealing insert is pressed together around the USB cable.

![Cable Gland Electronics Enclosure Fixed](assets/images/2024_cable_gland_electronics_enclosure_fixed.jpg){ width="700" }

Remove the counter nut of one of the M6 ventilation plugs and push the plug from
the outside of the enclosure through the 6 mm hole. Fasten the counter nut tightly
from the inside of the enclosure while pressing the sealing ring against the outer
wall of the enclosure.

![Ventilation Plug Electronics Enclosure Fixed](assets/images/2024_vent_plug_electronics_enclosure_fixed.jpg){ width="700" }

If you are using the solar panel, also attach the M20 cable gland to the
electronics enclosure. Put the sealing ring onto the cable gland thread, push
the cable gland through the 20 mm hole on the right side of the enclosure and
fasten it with the counter nut from the inside. Don't fasten the cable gland
nut at the outside of the enclosure yet.

![Cable Gland Solar Panel Electronics Enclosure Fixed](assets/images/2024_cable_gland_solar_electronics_enclosure_fixed.jpg){ width="700" }

![Cable Glands Electronics Enclosure Front](assets/images/2024_cable_glands_electronics_enclosure_front.jpg){ width="700" }

---

### 1.5 Top Mounting Plate

In the following steps, we will attach the V75 Battery Pack to the top mounting
plate that will be fixed in the lid of the enclosure. Prepare the top mounting
plate with drilled holes, the remaining four mounting plate screws, the Voltaic
V75 USB Battery Pack and the remaining six cable ties. Depending on the length
of your cable ties, you might only need four of them (see next step).

Start with pushing the ends of two cable ties through the two holes that are
farther away from the edge of the plate while orienting the cable tie heads
towards the center of the plate. Leave about 5 cm of the cable ties sticking out
while turning the plate and pushing the cable tie ends through the holes at the
edge from the other side. You can now already put the battery pack on the mounting
plate to check if you need to extend the cable ties, as in our example below.

![Mounting Plate Top Cable Ties](assets/images/2024_mounting_plate_top_cable_ties.jpg){ width="400" }

Put the battery pack onto the mounting plate with the USB ports facing towards the
cable tie heads. Connect and tighten the cable ties. Cut off the protruding ends.

![Mounting Plate Top Battery Cable Ties](assets/images/2024_mounting_plate_top_battery_cable_ties.jpg){ width="500" }

Push the two remaining cable ties through the holes on the right side of the
battery while orienting the cable tie heads towards the center of the plate.
Leave about 4 cm of the cable ties sticking out while pushing the cable tie
ends through the holes from the other side.

![Mounting Plate Top Battery Cable Ties Across](assets/images/2024_mounting_plate_top_battery_cable_ties_across.jpg){ width="500" }

Connect and tighten the cable ties. Cut off the protruding ends. The V75 battery
should not move when shaking the mounting plate.

![Mounting Plate Top Battery Cable Ties Fixed](assets/images/2024_mounting_plate_top_battery_cable_ties_fixed.jpg){ width="500" }

For the mounting plate integration, prepare the following tools and components:

!!! info ""

    :material-circle:{ .circle_blue } Electronics Enclosure Lid

    :material-circle:{ .circle_green } Mounting Plate + Voltaic V75 USB Battery Pack

    :material-circle:{ .circle_red } **4x** Mounting Plate Screws B M-SHR

    :material-circle:{ .circle_orange } Voltaic USB-C to USB-A cable

    :material-circle:{ .circle_yellow } Crosshead Screwdriver

![Overview Lid Mounting Plate Top Battery](assets/images/2024_overview_lid_mounting_plate_top_battery.jpg){ width="750" }

Insert the mounting plate with the battery pack into the lid of the enclosure
and fix it with the four mounting plate screws.

![Lid Mounting Plate Top Battery Fixed](assets/images/2024_lid_mounting_plate_top_battery_fixed.jpg){ width="500" }

Put the lid on the electronics enclosure with the USB ports of the battery pack
facing towards the USB-C port of the Witty Pi board. Close the latch on the
USB port side, this latch will function as a hinge from now on and should not
be opened again as long as the battery pack cables are still connected (for more
info, check [1.6 Closing the Enclosure](#16-closing-the-enclosure)). Connect the
battery pack to the Witty Pi board with the USB-C to USB-A cable.

![Lid Mounting Plate Top Battery Connected](assets/images/2024_lid_mounting_plate_top_battery_connected.jpg){ width="400" }

If you are using the solar panel, follow the next steps to prepare the connection
between the Voltaic V75 battery pack and the solar panel input cable.

First, connect the Voltaic USB-C adapter to the 3.5x1.1mm cable with a twisting motion.

![Voltaic USB-C Adapter Cable](assets/images/2024_voltaic_usb_adapter_cable.jpg){ width="500" }

Plug the USB-C adapter into the USB-C side port of the V75 battery pack and guide
the other end of the cable through the M20 cable gland. Don't fasten the cable
gland nut yet.

![Voltaic Battery Solar Panel Cable](assets/images/2024_voltaic_battery_solar_panel_cable.jpg){ width="600" }

Push the solar panel input cable end through the cable gland while making sure
that the grooved part is just barely covered by the sealing insert. Hold the
cable in this position and fasten the cable gland nut outside of the enclosure
tightly, so that the sealing insert is pressed together around the cable end.

![Voltaic Solar Panel Cable Gland](assets/images/2024_voltaic_solar_panel_cable_gland.jpg){ width="600" }

You are now finished with integrating the hardware into the electronics enclosure!

![Electronics Enclosure Hardware](assets/images/2024_electronics_enclosure_hardware.jpg){ width="400" }

---

### 1.6 Closing the Enclosure

You can open the lid of the enclosure by inserting a flathead screwdriver
about 0.5 cm into the slit in the middle of the latch. Open the latch with the
screwdriver by pushing it into the direction of the enclosure. Watch this
[video clip](https://youtu.be/8cvUxL9tM7k?si=uUJ0UXP5dwj4VZj4&t=87){target=_blank}
as reference before you open the latches for the first time. In this way, you can
also completely remove the lid.

To make sure that the enclosure is only opened at the correct side, you can use a special
[blanking plug](https://www.bopla.de/en/enclosure-technology/bocube/accessories-27/blanking-plugs-to-determine-the-hinge-side/b-vs-7035){target=_blank}
to determine the hinge side. It is also possible to close the enclosure with
[screws](https://www.bopla.de/en/enclosure-technology/bocube/accessories-27/sets-of-screws/b-shr){target=_blank}.
For more information, check the [Bocube info page](https://www.bopla.de/en/enclosure-technology/bocube){target=_blank}.

---

## 2 Camera Enclosure

In the next steps, we will integrate the OAK-1 camera into the camera enclosure.

<figure markdown="span">
  ![Camera Enclosure Closed](assets/images/2024_camera_enclosure_closed.jpg){ width="400" }
  <figcaption>Camera enclosure with integrated OAK-1 camera</figcaption>
</figure>

### 2.1 Mounting Plate

Push two M4 16 mm screws through the two drilled holes in the middle of the
mounting plate. Screw two M4 nuts onto the screws until you reach the middle
of the screw.

![Mounting Plate Camera Screws](assets/images/2024_mounting_plate_camera_screws.jpg){ width="600" }

Use the 3 mm allen key to screw the M4 screws into both M4 holes of the OAK-1
camera in an alternating way. Only rotate a few turns for each screw and then
switch to the other screw. Hold the M4 nut in place with your finger to avoid
it blocking the screw, it shouldn't touch the mounting plate yet. Do this until
the screws are completely inserted into the camera.

![Mounting Plate Camera Fixing](assets/images/2024_mounting_plate_camera_fixing.jpg){ width="600" }

Now push the screws against the mounting plate from the other side and use pliers
to fasten both nuts to the plate. Make sure that the camera is mounted parallel
to the plate and does not move when shaking it a little bit.

![Mounting Plate Camera Fasten Nuts](assets/images/2024_mounting_plate_camera_fasten_nuts.jpg){ width="600" }

To attach the silica gel pack, we will reuse one of the black binding wires that
came with the USB cables of the OAK camera and V75 battery. Push both ends of the
wire through the two small holes from underneath the plate and twist them around
the silica gel pack for fixing.

![Mounting Plate Camera Silica Gel Pack](assets/images/2024_mounting_plate_camera_silica_pack.jpg){ width="500" }

Insert the mounting plate into to enclosure with the USB-C port of the OAK
camera facing towards the 20 mm hole and fix it with the four mounting plate
screws. This will be a little more difficult compared to the bigger electronics
enclosure. Make sure to hold the screws perfectly vertical while screwing them
in the holes of the enclosure to avoid any blocking.

![Mounting Plate Camera Enclosure](assets/images/2024_mounting_plate_camera_enclosure.jpg){ width="500" }

---

### 2.2 Cable Gland + Vent Plug

Prepare the MBF 20-RJ45 cable gland, M20 sealing ring, M20 counter nut and
M6 ventilation plug. To connect the camera, we will also need the USB-C cable
end from the already prepared electronics enclosure.

![Overview Cable Gland Camera Enclosure](assets/images/2024_overview_cable_gland_camera_enclosure.jpg){ width="750" }

The following steps assume that you already connected the cable gland of the
electronics enclosure. For more detailed explanations, check
[1.4 Cable Glands + Vent Plug](#14-cable-glands-vent-plug).

Put the USB-C end of the cable first through the cable gland nut and then
through the cable gland. Put the sealing insert around the cable between
both components and make sure that everything is oriented as in the following
image.

![Cable Gland Camera USB Cable Sealing](assets/images/2024_cable_gland_camera_usb_cable_sealing.jpg){ width="600" }

Push the sealing insert back into the cable gland completely. Screw the cable
gland nut a little bit on the cable gland. Don't fasten it yet, the USB cable
has to move freely.

![Cable Gland Camera USB Cable Sealing Fixed](assets/images/2024_cable_gland_camera_usb_cable_sealing_fixed.jpg){ width="600" }

First, put the sealing ring onto the thread of the cable gland. Then insert the
USB cable through the 20 mm hole in the enclosure. From the inside, guide the cable
through the cable gland counter nut and connect it to the OAK camera.

![Cable Gland Camera Enclosure Fixing](assets/images/2024_cable_gland_camera_enclosure_fixing.jpg){ width="400" }

Push the counter nut against the inner wall of the enclosure and fasten the
cable gland from the outside of the enclosure. Be careful to keep the USB
cable in position that it doesn't lose connection to the OAK camera. Finally,
also fasten the cable gland nut outside of the enclosure tightly, so that the
sealing insert is pressed together around the USB cable.

![Cable Gland Camera Enclosure Fixed](assets/images/2024_cable_gland_camera_enclosure_fixed.jpg){ width="400" }

Remove the counter nut of the remaining M6 ventilation plug and push the plug from
the outside of the enclosure through the 6 mm hole. Fasten the counter nut tightly
from the inside of the enclosure while pressing the sealing ring against the outer
wall of the enclosure.

![Ventilation Plug Camera Enclosure Fixed](assets/images/2024_vent_plug_camera_enclosure_fixed.jpg){ width="400" }

---

### 2.3 Closing the Enclosure

If you insert the screws for the transparent lid, you will have to screw them
through the holes in the lid first. Seal the camera enclosure by fastening all
four screws.

![Camera Enclosure Closed](assets/images/2024_camera_enclosure_closed.jpg){ width="400" }

---

Both your electronics and camera enclosure are now connected securely and are
completely waterproof. Be careful during transportation and deployment to not
pull the USB cables, as this could loosen the connection between Raspberry Pi
and OAK camera.

![Enclosures Connected](assets/images/2024_enclosures_connected.jpg){ width="750" }
