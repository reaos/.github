> WARNING: YOU NEED AT LEAST 200GB OF FREE SPACE ON YOUR DISK

# Build reaOS with Docker

## 1. Prepare the environment
- Install Docker
- Install git-lfs
- Install repo

## 2. Fetch the AOSP source code
- Create a directory for the AOSP source code (optional)
```bash
mkdir -p "$HOME/reaosp"
cd "$HOME/reaosp"
```

- Set workdir to the current directory
```bash
REAOS_DIR="$PWD"
```

- Clone the AOSP source code
```bash
repo init -u https://android.googlesource.com/platform/manifest --git-lfs --depth=1 -b android-15.0.0_r36
git clone https://github.com/reaos/local_manifests .repo/local_manifests -b reaos-15
```

- Optional: Enable GAPPS
```bash
cp .repo/local_manifests/gapps.xml{.example,}
```

- Sync the AOSP source code
```bash
repo sync -j8
```

## 3. Prepare building environment
- Create image
```bash
docker buildx create --use
docker buildx build --build-arg userid=$(id -u) --build-arg groupid=$(id -g) --build-arg username=rea -t rea-builder --load .repo/local_manifests/
```

- Run the image
```bash
docker run -it --privileged --rm --hostname rea-builder --name rea-builder -v $REAOS_DIR:/src -w /src rea-builder
```

## 4. Build the AOSP source code
- Enter the Docker container
```bash
docker exec -it rea-builder bash
```

