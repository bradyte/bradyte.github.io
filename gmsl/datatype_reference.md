# Datatype Reference  
I don't think we explicitly outline these details but I got tired of searching our documentation. Here is a list of MIPI datatypes supported when using GMSL devices in pixel mode and you need to route specific datatypes.  

| hex code  | datatype      |  
| --------  | --------      |
| 0x12      | EMBEDDED      |
| 0x1E      | YUV422 8-bit  |
| 0x1F      | YUV422 10-bit |
| 0x30      | YUV422 12-bit |
| 0x22      | RGB565        |
| 0x23      | RGB666        |
| 0x24      | RGB888        |
| 0x2A      | RAW8          |
| 0x2B      | RAW10         |
| 0x2C      | RAW12         |
| 0x2D      | RAW14         |
| 0x2E      | RAW16         |
| 0x2F      | RAW20         |


## example  
Using the MAX96717 as an example, here is the template for how you would set this. _Please use tunnel mode and save the hassle._

| Reg Name | Address | Bit Field | Example | Notes |
| -------- | -------- | -------- | -------- | ------ |
| FRONTTOP__16 | 0x0318 | mem_dt1_selz[6:0] | 0x6b | The IMX219 sensor data is RAW10 which is a datatype of 0x2b. Bit 6 enables the data filtering.
| FRONTTOP__17 | 0x0319 | mem_dt2_selz[6:0] | 0x52 | The IMX219 sensor also sends embedded data which is a datatype of 0x2b. Bit 6 enables the data filtering.