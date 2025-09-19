### Stream an IMX219 sensor
[nvidia forum post](https://forums.developer.nvidia.com/t/failure-to-use-gst-launch-1-0-nvarguscamerasrc-to-capture-image/286500)  
streaming data   
```$ gst-launch-1.0 nvarguscamerasrc ! 'video/x-raw(memory:NVMM),framerate=30/1,format=NV12' ! nvvidconv ! xvimagesink```