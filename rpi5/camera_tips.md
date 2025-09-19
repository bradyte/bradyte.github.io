### Manually Setting the Camera Selection
If you do not use an official Raspberry Pi camera or don't want to use camera auto detection, you need to configure the `config.txt` file. 

If you use the latest Bookworm system, you need to configure `/boot/firmware/config.txt`.

If using the Bookworm system:
```
sudo nano /boot/firmware/config.txt
```
Else:
```
sudo nano /boot/config.txt
```

Find `camera-auto-detect=1` and modify it to `camera_auto_detect=0`.

At the end of the file, add the following setting statements according to the camera model.
* For the Raspberry Pi Camera V2 with IMX219: `dtoverlay=imx219`

### Using Bookworm
#### libcamera no more  
Reference from this [forum post](https://forums.raspberrypi.com/viewtopic.php?t=389906).  

The symlinks between `libcamera` and `rpicam` are being removed and `rpicam` is the naming convention that will be used moving forward.