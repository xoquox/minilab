# Epson V600 SDK infrared capture

This fork keeps an optional Epson V600/GT-X820 capture path on Linux. Upstream
0.9.0 scans IR through drivers that expose it openly; this path uses Epson Scan 2
for a V600 opened through epkowa. Proprietary driver files are not included.

## New captures

Select the V600 and turn on **Infrared pass** in Scan. SDK capture uses square
pixels at 800, 1600 or 3200 dpi. It bypasses custom SANE raw settings and uses
linear gamma with the colour matrix off. Previews still use SANE normally.
Turning the pass off restores the ordinary and native X/Y resolution controls.

The helper releases SANE's handle before taking separate RGB and IR passes.
The RGB stays at the requested depth, including 16 bits. The SDK's 8-bit IR is
widened losslessly to a 16-bit plane. A missing selected SDK or acquisition
failure fails the frame instead of falling back silently to an RGB-only scan.

New captures now keep the unmodified RGB and the aligned IR plane in upstream's
format, `scans/01.tif` and `scans/01.ir.tif`. Develop uses the normal 0.9.0 dust
controls, including its IR threshold, grow and display of removed defects.
Exports follow those settings. See [generic infrared processing](infrared.md).
This uses minilab's own processing, not Epson Digital ICE.

Each intermediate raw file is limited to 512 MiB. Use frame-sized areas at
higher resolutions. Acquisition takes two passes. Silver-based B&W and
Kodachrome are unsuitable for IR cleaning; chromogenic B&W can be suitable.
Review fine detail and large defects before exporting.

## Existing V600 captures

Earlier versions of this fork stored an original RGB scan, an IR PGM and a
precomputed corrected TIFF. Those files and their selection remain supported.
**V600 IR CORRECTION (LEGACY)** switches those frames between their untouched
original and the existing corrected copy, preserving crop and develop settings.
The normal 0.9.0 controls apply to newly captured or imported IR planes.

## Driver setup

The normal V600 SANE driver, including its interpreter, must already work.
Install Epson Scan 2 with its non-free interpreter and firmware as well.
Linux builds install `minilab-epson-ir` next to `minilab`. Set:

```sh
export MINILAB_EPSON_IR_HELPER=/path/to/bin/minilab-epson-ir
export MINILAB_EPSON_COMMAND_LIBRARY=/path/to/epsonscan2/libes2command.so
```

On NixOS the helper needs the same dynamic-loader wrapper as minilab; point
`MINILAB_EPSON_IR_HELPER` at that wrapper. The command library and its adjacent
`non-free-exec/es2intif` must come from the same Epson installation. The helper
matches the selected USB address to its serial and refuses ambiguous devices.
Driver failures stay in the helper process rather than terminating the app.

The command line retains its older corrected-file and sidecar workflow:

```sh
minilab scan frame.tiff --infrared --dpi 800 --area 10,30,10,10
```

It saves the corrected output, `frame.tiff.original.tiff` and
`frame.tiff.infrared.pgm`. Original and IR are saved before correction.

The SDK acquisition was checked with Epson Scan 2 6.7.70 and a real V600 at
800 dpi with 16-bit RGB in the earlier fork. The 0.9.0 integration is covered
by software tests; no new hardware scan was made during the update. Larger
captures and other film stocks need visual checks. The SDK's Linux C++ binary
interface is version-specific; future Epson releases may need changes.
