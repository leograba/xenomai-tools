# Xenomai Tools

Container images for Xenomai tools.

## Xenomai 4 libevl Container Image

How to run the pre-built image:

```bash
docker run --privileged --rm -it -v /dev:/dev leograba/libevl:r50
```

Then you can run `evl`, `latmus`, and `hectic` from the container shell.

## Xenomai 3 Libs and Tools Container Image

How to run the pre-built image:

```bash
docker run --privileged --rm -it -v /dev:/dev leograba/xeno3:v3.2.6 bash
```

Then you can run the tools from the container shell.
