# HPLIP Deskjet 1510 scan fix

Open-source fix for HP Deskjet 1510 (and related LEDM/Soap-based HP all-in-ones)
that fail to scan on recent distributions with:

```
scan/sane/http.c 506: http_read: response exceeds buffer (read=8188 len=12 max=8192)
```

## Root cause

HPLIP 3.26.6 double-counts bytes in `scan/sane/http.c` while reading a chunked
HTTP response. A ~4 KB LEDM/XML response is therefore rejected at the 8192-byte
buffer limit.

## Patch

```diff
--- a/scan/sane/http.c
+++ b/scan/sane/http.c
@@ -480,7 +480,6 @@
             if(ret)
             {
                 *bytes_read += len;
-                *bytes_read += len;
                 if(*bytes_read + len > max_size)
                 {
                     BUG("http_read: response exceeds buffer (read=%d len=%d max=%d)\n",
```

The duplicate `*bytes_read += len;` is removed.

## Build (Arch / Omarchy)

```bash
# Install build deps
sudo pacman -S base-devel cups dbus libjpeg-turbo libusb libusb-compat libtool \
  net-snmp python sane openssl avahi

# Source
tar -xzf hplip-3.26.6.tar.gz
cd hplip-3.26.6
patch -p1 < ../hplip-3.26.6-http-buffer-fix.patch

# Configure with modern GCC
CFLAGS='-O2 -std=gnu17 -D_DEFAULT_SOURCE \
  -Wno-error=implicit-function-declaration \
  -Wno-error=incompatible-pointer-types' \
./configure --prefix=/usr --libdir=/usr/lib --sbindir=/usr/bin \
  --disable-qt4 --disable-qt5 --disable-doc-build \
  --disable-cups-drv-install --disable-hpcups-install \
  --disable-hpijs-install --disable-fax-build \
  --enable-network-build --enable-scan-build

# Build only the SANE backend
make -j"$(nproc)" libsane-hpaio.la
```

The patched library is `.libs/libsane-hpaio.so.1.0.0`.

## Install

```bash
sudo cp /usr/lib/sane/libsane-hpaio.so.1.0.0 \
        /usr/lib/sane/libsane-hpaio.so.1.0.0.bak
sudo cp .libs/libsane-hpaio.so.1.0.0 /usr/lib/sane/libsane-hpaio.so.1.0.0
sudo chmod 755 /usr/lib/sane/libsane-hpaio.so.1.0.0
```

## Test

```bash
scanimage -T -d 'hpaio:/usb/Deskjet_1510_series?serial=YOUR_SERIAL'
```

## Affected environment

- OS: Omarchy (Arch-based), kernel 7.2.5-4-omarchy
- Device: HP Deskjet 1510 series, USB ID `03f0:c111`
- Packages: `hplip 1:3.26.6-1`, `sane 1.4.0-4`

## See also

- Launchpad HPLIP bug [#2166824](https://bugs.launchpad.net/hplip/+bug/2166824)
