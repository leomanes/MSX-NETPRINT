# NETPRINT — MSX Network Printing

**MSX Network Printing**  
**Version: 1.0 - (2026) Leo Manes**  
**lmanes@protonmail.com**  
**UNAPI driver by DUCASP**  
**https://github.com/ducasp**  
**https://github.com/Konamiman/MSX-UNAPI-specification**
  

NETPRINT is a resident MSX network printer redirector. It allows MSX-DOS and MSX BASIC programs that use the standard MSX BIOS printer output path to print to a network printer through the DUCASP TCP/IP UNAPI driver.

NETPRINT is being developed and tested on an MSX Goa’uld / OCM-style MSX system using the DUCASP ESP/UNAPI network driver.

---

## What NETPRINT does

NETPRINT redirects MSX printer output to a network printer.

Typical supported sources:

- MSX-DOS `COPY file PRN`
- MSX BASIC `LPRINT`
- MSX BASIC `LLIST`
- Programs that use BIOS printer output

The general data path is:

    MSX program / BASIC / DOS
            ↓
    MSX BIOS printer output
            ↓
    NETPRINT resident hook
            ↓
    TCP/IP UNAPI driver by DUCASP
            ↓
    ESP Wi-Fi bridge
            ↓
    Network printer

NETPRINT sends raw printer data to the network printer, normally using TCP port `9100`, also known as JetDirect, AppSocket, or raw TCP printing.

---

## Important limitation

NETPRINT only redirects software that prints through the MSX BIOS printer path.

It may not capture programs that write directly to printer hardware ports.

Also, BASIC programs using:

    OPEN "LPT:" FOR OUTPUT AS #1
    PRINT #1,"HELLO"
    CLOSE #1

are **not supported yet** and may fail or freeze.

Use `LPRINT` instead:

    LPRINT "HELLO"
    LPRINT CHR$(12);

---

## Features

- Resident MSX-DOS network printer driver
- Redirects BIOS printer output through `H.LPTO`
- Sends print data to a network printer over TCP/IP UNAPI
- Default raw printer port: `9100`
- 128-byte transmit buffer
- Supports MSX-DOS printing with `COPY file PRN`
- Supports MSX BASIC `LPRINT`
- Manual form feed command
- Manual close/end-job command
- Manual reconnect command
- Optional PCL color command support
- Quiet/muted mode for selected commands
- Status display with counters and last error information

---

## Commands

    NETPRINT
    NETPRINT /h
    NETPRINT -i x.x.x.x [-p port] [-m]
    NETPRINT -s
    NETPRINT -e [-m]
    NETPRINT -c [PS|PCL|PDL] [-m]
    NETPRINT -r [-m]
    NETPRINT -color <color> [-m]
    NETPRINT -u [-m]

---

## Help

    NETPRINT

or:

    NETPRINT /h

Shows the help screen.

---

## Install NETPRINT

    NETPRINT -i x.x.x.x

Example:

    NETPRINT -i 192.168.1.50

This installs the resident printer driver and opens a TCP connection to the network printer.

Default port:

    9100

To specify another port:

    NETPRINT -i 192.168.1.50 -p 9100

Verbose install output:

    MSX Network Printing
    Version: 1.0 - (2026) Leo Manes
    UNAPI driver by DUCASP

    [1/5] Allocating resident memory... OK
    [2/5] Finding TCP/IP UNAPI... OK
    [3/5] Opening printer connection... OK
    [4/5] Building resident sender... OK
    [5/5] Installing BIOS printer hook... OK

    Printer: 192.168.1.50:9100
    NETPRINT installed.

Muted install:

    NETPRINT -i 192.168.1.50 -m

With `-m`, successful output is silent. Errors still appear.

---

## Print from MSX-DOS

Print a text file:

    COPY LICENSE.TXT PRN

Then send form feed:

    NETPRINT -e

Typical workflow:

    NETPRINT -i 192.168.1.50
    COPY LICENSE.TXT PRN
    NETPRINT -e
    NETPRINT -c

---

## Print from MSX BASIC

Use `LPRINT`:

    LPRINT "HELLO FROM BASIC"
    LPRINT CHR$(12);

`CHR$(12)` is form feed. It tells the printer to print/eject the current page.

Example:

    10 LPRINT "MSX BASIC PRINT TEST"
    20 LPRINT "SECOND LINE"
    30 LPRINT CHR$(12);

`LLIST` may also work because it uses the BASIC printer path.

Do not use this method yet:

    OPEN "LPT:" FOR OUTPUT AS #1
    PRINT #1,"HELLO"
    CLOSE #1

---

## Status

    NETPRINT -s

Example output:

    MSX Network Printing
    Version: 1.0 - (2026) Leo Manes
    UNAPI driver by DUCASP

    Installed: YES
    Printer: 192.168.1.50:9100
    Connection: OPEN
    BIOS hook: ACTIVE
    Buffer: 128 bytes
    Buffered: 0 bytes
    Captured: 30720 bytes
    Sent: 30720 bytes
    Chunks: 240
    Dropped: 0 bytes
    Last error: none

If not installed:

    MSX Network Printing
    Version: 1.0 - (2026) Leo Manes
    UNAPI driver by DUCASP

    Installed: NO

Status fields:

    Installed
        Whether NETPRINT is installed.

    Printer
        Current configured printer IP and port.

    Connection
        Last known TCP connection state.

    BIOS hook
        Whether the MSX BIOS printer hook is active.

    Buffer
        NETPRINT transmit buffer size.

    Buffered
        Number of bytes currently waiting in NETPRINT buffer.

    Captured
        Total printer bytes captured by NETPRINT.

    Sent
        Total bytes sent to the network printer.

    Chunks
        Number of TCP send chunks sent.

    Dropped
        Number of bytes lost because they could not be buffered or sent.

    Last error
        Last TCP/IP or NETPRINT error code.

---

## Form feed

    NETPRINT -e

Sends a form-feed byte:

    0C hex
    CHR$(12)

Output:

    Form feed... OK

Muted:

    NETPRINT -e -m

No output on success. Errors still appear.

Use this when the printer has received data but has not printed/ejected the page yet.

---

## Closing the printer connection

    NETPRINT -c

or:

    NETPRINT -c PS
    NETPRINT -c PCL
    NETPRINT -c PDL

`NETPRINT -c` closes the current TCP printer connection and keeps NETPRINT installed.

It does **not** uninstall NETPRINT.

It does **not** send form feed.

Output:

    Close connection... OK

Muted:

    NETPRINT -c -m

No output on success.

### Close modes

    NETPRINT -c
    NETPRINT -c PS

Default mode. Sends Ctrl-D / EOT, then closes the TCP connection.

Ctrl-D / EOT byte:

    04

    NETPRINT -c PCL

Closes the TCP connection only.

This is usually the best first choice for PCL/plain-text/raw printing.

    NETPRINT -c PDL

Sends:

    00

then closes the TCP connection.

This is an experimental generic binary-PDL end marker.

If the printer does not release the job or continues showing a message such as `Printing Document`, try the alternate close modes:

    NETPRINT -c PCL
    NETPRINT -c PDL
    NETPRINT -c PS

Different printers may respond better to different close/end-of-job methods.

---

## Reconnect

    NETPRINT -r

Reconnects to the stored printer IP and port.

Output:

    Reconnect printer... OK

Muted:

    NETPRINT -r -m

No output on success.

Use this if:

- The printer connection timed out.
- The printer was powered off/on.
- `NETPRINT -c` was used and you want to print again.
- The connection became stale after being idle.

Example:

    NETPRINT -r
    COPY OTHER.TXT PRN
    NETPRINT -e

---

## Color command

    NETPRINT -color <color>

Supported colors:

- black
- red
- green
- yellow
- blue
- magenta
- cyan

Example:

    NETPRINT -color red
    COPY LICENSE.TXT PRN
    NETPRINT -e

Success output:

    Color red... OK

Muted:

    NETPRINT -color red -m

No output on success.

### PCL color sequences

NETPRINT sends these PCL byte sequences:

    Black:
        1B 2A 72 33 55 1B 2A 76 37 53

    Red:
        1B 2A 72 33 55 1B 2A 76 31 53

    Green:
        1B 2A 72 33 55 1B 2A 76 32 53

    Yellow:
        1B 2A 72 33 55 1B 2A 76 33 53

    Blue:
        1B 2A 72 33 55 1B 2A 76 34 53

    Magenta:
        1B 2A 72 33 55 1B 2A 76 35 53

    Cyan:
        1B 2A 72 33 55 1B 2A 76 36 53

Important:

- This is a PCL feature.
- It only works if the printer supports these PCL color commands.
- Some printers may ignore them or print in black.
- The color command affects the current open print job/session.

The color setting may be reset after:

- `NETPRINT -c`
- `NETPRINT -r`
- `NETPRINT -u`
- printer timeout
- printer reset

---

## Uninstall

    NETPRINT -u

Output:

    NETPRINT uninstalled.

Muted:

    NETPRINT -u -m

No output on success.

Uninstalling restores the original BIOS printer hook and removes NETPRINT’s resident driver state.

---

## Mute option

The `-m` option suppresses success messages.

Supported with:

    NETPRINT -i ... -m
    NETPRINT -e -m
    NETPRINT -c ... -m
    NETPRINT -r -m
    NETPRINT -color ... -m
    NETPRINT -u -m

Errors always show.

---

## How modern network printers handle incoming data

Modern network printers normally do not print every byte the instant it arrives. They receive data into an internal printer/job buffer and decide when to process, print, or eject the page.

To force the printer to eject/print the page, use:

    NETPRINT -e

`NETPRINT -e` sends a form-feed byte to the printer. This is useful after printing a short text file or a small BASIC `LPRINT` message.

Example:

    NETPRINT -i 192.168.1.50
    COPY LICENSE.TXT PRN
    NETPRINT -e

After the page is printed, you can close the printer connection with:

    NETPRINT -c

If the printer does not release the job or continues showing a message like `Printing Document`, try the alternate close modes:

    NETPRINT -c PCL
    NETPRINT -c PDL
    NETPRINT -c PS

Different printers may respond better to different close/end-of-job methods.

---

## Printer timeout behavior

If the printer receives some data and then no more data arrives for a while, the printer may eventually time out. When this happens, it may print whatever it already has in its internal buffer.

After the printer times out, the TCP connection that NETPRINT opened may no longer be valid. NETPRINT may still be installed, but the old printer connection can be stale or closed by the printer.

If printing stops working after the printer timed out, reconnect NETPRINT manually:

    NETPRINT -r

Then print again:

    COPY OTHER.TXT PRN
    NETPRINT -e

Normal rule:

    If the print job is short or the page does not come out:
        use NETPRINT -e

    If the printer timed out or NETPRINT stops communicating:
        use NETPRINT -r

---

## How NETPRINT works internally

NETPRINT installs a small resident driver in MSX RAM.

It hooks the MSX BIOS printer output hook:

    H.LPTO

When a program prints through the BIOS printer path, the BIOS calls `H.LPTO`. NETPRINT redirects that call to its resident handler.

The handler:

- receives the printed byte
- stores it in a 128-byte buffer
- sends chunks through TCP/IP UNAPI
- prevents the BIOS from hanging on real printer hardware
- returns to the calling program

The reason NETPRINT must skip the normal physical-printer path is that the Goa’uld/OCM system does not have a real Centronics printer interface. If BIOS continued to poll the physical printer port, the system could freeze waiting for printer-ready.

---

## Buffer behavior

NETPRINT uses a 128-byte transmit buffer.

It does not store the entire file in memory.

For a 30 KB file, NETPRINT streams the data in chunks.

Example:

    30 KB = about 30,720 bytes
    30,720 / 128 = about 240 chunks

The buffer helps avoid sending one TCP packet per byte.

Current behavior keeps the known working speed/flush behavior from testing.

---

## Recommended workflows

### Normal DOS text file print

    NETPRINT -i 192.168.1.50
    COPY LICENSE.TXT PRN
    NETPRINT -e
    NETPRINT -c

### If `NETPRINT -c` does not release the printer job

    NETPRINT -c PCL

If still not released:

    NETPRINT -c PDL

If still not released:

    NETPRINT -c PS

### Reconnect later

    NETPRINT -r
    COPY OTHER.TXT PRN
    NETPRINT -e

### Print in red, if supported by printer

    NETPRINT -i 192.168.1.50
    NETPRINT -color red
    COPY LICENSE.TXT PRN
    NETPRINT -e
    NETPRINT -color black

### BASIC LPRINT

    LPRINT "HELLO FROM BASIC"
    LPRINT CHR$(12);

---

## Known limitations

NETPRINT redirects only software that prints through the MSX BIOS printer path.

It may not catch programs that write directly to printer hardware ports.

BASIC `OPEN` / `PRINT #` / `CLOSE` printer-device output is not implemented.

The following BASIC pattern is not supported yet and may fail or freeze:

    OPEN "LPT:" FOR OUTPUT AS #1
    PRINT #1,"HELLO"
    CLOSE #1

Use `LPRINT` instead:

    LPRINT "HELLO"
    LPRINT CHR$(12);

Raw TCP port 9100 printing depends on printer compatibility.

NETPRINT does not convert text to:

- PDF
- PostScript
- PCL graphics
- bitmap graphics
- ANSI colors

Color support is PCL-command based and printer-dependent.

The printer may hold a job until one of these happens:

- form feed is received
- connection is closed
- printer timeout expires
- printer receives an end-of-job marker

A TCP connection can become stale after being idle. Use:

    NETPRINT -r

to reconnect manually.

Automatic idle timeout form feed is not included.

Automatic background reconnect is not included.

---

## Test checklist

After installing a new build, test in this order:

    NETPRINT
    NETPRINT /h
    NETPRINT -i 192.168.1.50
    NETPRINT -s
    COPY LICENSE.TXT PRN
    NETPRINT -e
    NETPRINT -c
    NETPRINT -s
    NETPRINT -r
    NETPRINT -u

BASIC test:

    LPRINT "BASIC PRINT TEST"
    LPRINT CHR$(12);

Color test:

    NETPRINT -i 192.168.1.50
    NETPRINT -color red
    COPY LICENSE.TXT PRN
    NETPRINT -e

Check:

- Does DOS printing work?
- Does BASIC LPRINT work?
- Does the printer eject with `NETPRINT -e`?
- Does `NETPRINT -c` free the printer?
- Does `NETPRINT -r` reconnect?
- Does `NETPRINT -s` show reasonable counters?

---

## License

NETPRINT Personal and Non-Commercial Use License

Copyright (c) 2026 Leo Manes

Permission is granted to use, copy, study, modify, and fork this project for personal, educational, and non-commercial purposes only.

Commercial use is not permitted without prior written permission from the copyright holder.

You may not sell this project, include it in a commercial product, charge for access to it, or use it as part of a paid service without written permission.

Modified versions must keep this license notice and must not be distributed for commercial use.

This software is provided "as is", without warranty of any kind.
