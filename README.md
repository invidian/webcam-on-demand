# On demand stream from H.264 webcam into V4L2 virtual cam

On Linux, browsers and video conference applications doesn't seem to support H.264 encoding,
which makes them use lower quality, lower framerate or having a delay when using the webcam.

`webcam-on-demand` is a script, intended to run as systemd unit service, which watches for connecting
V4L2 virtual webcam clients and when one appears, it starts `ffmpeg` process in the background to stream
high-quality video from your webcam into virtual webcam used by other applications, which produces better
quality image and lower latency.

Script supports customizing output resolution, number of frames per second, selecting specific webcam device
to stream from and setting V4L2 controls for your webcam device to fine-tune the vidoe settings.

Script should be CPU-friendly as `ffmpeg` will only run when webcam is in use.

## Installation

Before you install it, make sure you have `v4l2loopback` kernel module available.

```sh
mkdir -p ~/.local/bin
cp ./webcam-on-demand ~/.local/bin/

mkdir -p ~/.config/systemd/user/
cp webcam-on-demand ~/.config/systemd/user/

sudo cp ./etc/modules-load.d/v4l2loopback.conf /etc/modules-load.d/
sudo cp ./etc/modprobe.d/v4l2loopback.conf /etc/modprobe.d/
sudo modprobe v4l2loopback devices=1 exclusive_caps=1 card_label="VirtualCam"

systemctl --user daemon-reload
systemctl --user enable --now webcam-on-demand.service
```

## Configuration

See available environment variables in [webcam-on-demand](webcam-on-demand) file.

You can modify them by editing the systemd unit or creating a drop-in for it using the command below:

```sh
systemctl --user edit webcam-on-demand.service
```

Then, write to it something like like:

```
[Service]
Environment="V4L2_DEVICE_FILTER='USB  Live camera: USB  Live cam'"
```

## Testing

To verify the script works, you can see it logs:

```sh
journalctl --user -u webcam-on-demand.service -f
```

And open the webcam using any application. Remember to select `VirtualCam` device.
