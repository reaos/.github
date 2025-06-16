> **WARNING:** YOU NEED AT LEAST 200GB OF FREE SPACE ON YOUR DISK

> **For macOS users:**
> * Create a case-sensitive partition for the AOSP source code.
> * VMM selection:
>   * Docker VMM:
>     * Build segfaults and freezes.
>   * Apple Virtualization Framework:
>     * VirtioFS:
>       * "Too many open files" error.
>       * macOS itself becomes unstable and crashes.
>     * gRPC FUSE:
>       * Works, but very slow.
>     * Alternatively, you can use docker's volume mount.

# Building reaOS in Docker

## 1. Prepare the environment
* Install Docker
* Install git-lfs
* Install repo

## 2. Fetch the AOSP source code
- Create a directory for the AOSP source code (example)
```bash
mkdir -p "$HOME/reaosp"
cd "$HOME/reaosp"
```
- **WARNING:** Now you should be in directory where AOSP source code will be placed.
- Clone the AOSP source code
```bash
repo init -u https://android.googlesource.com/platform/manifest --git-lfs --depth=1 -b android-15.0.0_r23
git clone https://github.com/reaos/local_manifests .repo/local_manifests -b reaos-15
```

- **Optional:** Enable GAPPS
```bash
cp .repo/local_manifests/gapps.xml{.example,}
```

- Sync the AOSP source code
```bash
repo sync -j8
```

## 3. Create signing keys
- Run the script to create signing keys
```bash
(cd ./vendor/lineage-priv/keys && ./keys.sh)
```

## 4. Prepare building environment
- Create image
```bash
docker buildx build --build-arg userid=$(id -u) --build-arg groupid=$(id -g) --build-arg username=rea -t rea-builder --load .repo/local_manifests/
```

- Run and enter the image
```bash
docker run -it --privileged --rm --hostname rea-builder -v .:/src rea-builder
```

## 5. Build the AOSP source code
- Load the environment
```bash
. build/envsetup.sh
```

- Choose the target
```bash
lunch2 reaos bp1a user
```

- Build the AOSP source code
```bash
m -j$(nproc)
```

## 6. Import to docker
- Mount images
```bash
(cd out_rea/target/product/reaos \
  && sudo mount system{.img,} -o ro \
  && sudo mount vendor{.img,} -o ro)
```

- Import to docker
```bash
(cd out_rea/target/product/reaos && sudo tar --xattrs -c vendor -C system --exclude="./vendor" . | docker import --platform=linux/arm64 -c 'ENTRYPOINT ["/init", "qemu=1", "androidboot.hardware=reaos", "ro.setupwizard.mode=DISABLED"]' - reaos)
```

- Unmount images
```bash
(cd out_rea/target/product/reaos && sudo umount system vendor)
```