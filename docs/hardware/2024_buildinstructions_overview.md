# Overview

??? warning "Disclaimer"

    Every effort has been made to check the instructions for accuracy and
    completeness. However, the author cannot be held responsible for any errors
    or omissions. The readers remain responsible for their safety when using the
    tools, hardware and software described in the instructions for building
    and deploying the Insect Detect camera trap. The readers must also comply
    with all applicable laws and regulations regarding the deployment of the
    camera trap system including the associated software and deep learning models.
    The author of these instructions cannot be held responsible for the results
    of action, inaction, or otherwise taken as a consequence of the information
    provided in the instructions. All products, brands and product links to any
    online shops are mentioned for illustration purposes only and do not denote
    a commercial relationship between the author and the products/brands/shops.

The following instructions will show you how to build the
[**Full Setup**](2024_components.md#full-setup){target=_blank} of the Insect
Detect DIY camera trap for automated insect monitoring (version 2024).

??? abstract "Required Tools"

    1. Printer
    2. Scissors
    3. Adhesive Tape
    4. Nail/Screw & Hammer
    5. Pointed Tool (e.g. from multi-tool knife)
    6. Rubber Hammer
    7. Drilling Machine
    8. Drill Press (recommended if available)
    9. Bench Vise (recommended if available)
    10. Metal Step Drill Bit (with 20 mm step) - for M20 Cable Glands
    11. Forstner Bit or similar (19 mm) - for LED Button
    12. Metal Drill Bit (4 mm + 6 mm) - for Enclosures, HPL Sheets and Aluminium Square Tube
    13. Metal Drill Bit (8 mm) - if holes in Steel L-profile are smaller than 8 mm
    14. Wood Drill Bit (3 mm + 4 mm) - for Mounting Plates
    15. Allen Keys (3 mm, 5 mm, 6 mm)
    16. Crosshead Screwdrivers (small for M2.5 screws + large for enclosure screws)
    17. Flathead Screwdriver (to open electronics enclosure lid)
    18. Pliers
    19. Microfiber Cloth
    20. Soldering Iron (+ Solder)
    21. Mounting Putty (e.g. patafix or Blu Tack)
    22. Wire Stripper Tool (or scissors if not available)

---

## Tools & Components per section

### Preparing the Enclosures

??? abstract "Required Tools & Components"

    **Tools**

    1. Printer
    2. Scissors
    3. Adhesive Tape
    4. Nail/Screw & Hammer
    5. Pointed Tool (e.g. from multi-tool knife)
    6. Drilling Machine
    7. Drill Press (recommended if available)
    8. Bench Vise (recommended if available)
    9. Metal Step Drill Bit (with 20 mm step) - for M20 Cable Glands
    10. Forstner Bit or similar (19 mm) - for LED Button
    11. Metal Drill Bit (4 mm + 6 mm) - for Enclosures, HPL Sheets and Aluminium Square Tube
    12. Wood Drill Bit (3 mm + 4 mm) - for Mounting Plates
    13. Allen Key (3 mm)
    14. Pliers
    15. Microfiber Cloth

    **Components**

    1. Bocube Enclosure B 221309 PC-V0 (= Electronics Enclosure)
    2. Euromas Enclosure M 215 G (= Camera Enclosure)
    3. **2x** Bocube Mounting Plate B M 2213
    4. Euromas Mounting Plate M 215
    5. HPL Sheet, 3 mm (10x20 cm)
    6. HPL Sheet, 3 mm (12x24 cm)
    7. Alu Square Tube, 13.5 mm (120 mm)
    8. **6x** Screw M4, internal hexagon (16 mm)
    9. **2x** Screw M4, internal hexagon (30 mm)
    10. **16x** Hex Nut M4, flat
    11. **8x** Washer (4.3x15x1 mm)
    12. Screen Protector (~7x9 cm)

### Integrating the Hardware

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

### Mounting Setup

??? abstract "Required Tools & Components"

    **Tools**

    1. Rubber Hammer
    2. Allen Keys (3 mm, 5 mm, 6 mm)
    3. Crosshead Screwdriver
    4. Drilling Machine + 8 mm Metal Drill Bit (if holes in Steel L-profile are smaller than 8 mm)

    **Components**

    1. Insect Detect 2024 Aluminium Mount (multiple parts)
    2. Steel L-profile (50 cm)
    3. Electronics Enclosure (with integrated hardware and attached to HPL sheet)
    4. Camera Enclosure (with integrated OAK camera and attached to HPL sheet)
    5. Voltaic Solar Panel 5 Watt 6 Volt
    6. Voltaic Solar Panel Bracket - Medium
    7. Voltaic ETFE Panel Screws & Washers
    8. **5x** Screw M4, internal hexagon (16 mm)
    9. **2x** Screw M8, internal hexagon (16 mm)
    10. **5x** Washer (4.3x15x1 mm)
    11. **2x** Washer (8.4x24x2 mm)
    12. **2x** Alu Square Tube, 13.5 mm (250 mm)
    13. **5x** Alu Square Tube, 13.5 mm (120 mm)
    14. **4x** Connector, 13.5 mm (right angle)
    15. **2x** Connector, 13.5 mm (+ outlet)
    16. HPL Sheet, green (14x14 cm)
    17. **~720 mm** Magnetic Tape, <13.5 mm
    18. [Components for Platform Design](2024_buildinstructions_mounting.md#2-platform){target=_blank}

### Field Deployment

??? abstract "Required Tools & Components"

    **Tools**

    1. Spade
    2. Allen Keys (3 mm, 5 mm, 6 mm)

    **Components**

    1. Insect Detect 2024 Mount (fully assembled)
    2. Electronics & Camera Enclosures (with integrated hardware)
    3. Voltaic Solar Panel 5 Watt 6 Volt (attached to solar panel bracket)
    4. Platform Frame + Platform
    5. **5x** Screw M4, internal hexagon (16 mm)
    6. **5x** Washer (4.3x15x1 mm)
    7. **2x** Hex Nut M4, flat
