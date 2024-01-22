# Camera Interface
---

TinyCLR OS supports digital camera interface, sometimes referred to as DCMI or DCI, on devices using the SITCore SC20260 SoC. Typically, cameras need to be configured using [I2C bus](i2c.md). Check the Omnivision/Ov9655 driver under https://github.com/ghi-electronics/TinyCLR-Drivers for an example of how to configure your camera, or refer to the camera's manual to determine the needed configuration.

The following function captures camera images:  
```cs
public void Capture(byte[] data, int timeoutMillisecond) =>
    this.cameraController.Capture(data, timeoutMillisecond);
```

This example configures the camera and sends the images to the 4.3 inch display attached to the SCM20260D Dev Board.

> [!Tip]
> Needed NuGets: GHIElectronics.TinyCLR.Devices.Gpio, GHIElectronics.TinyCLR.Devices.I2c, GHIElectronics.TinyCLR.Drivers.Omnivision.Ov9655, GHIElectronics.TinyCLR.Native, GHIElectronics.TinyCLR.Pins, GHIElectronics.TinyCLR.Devices.Display, GHIElectronics.TinyCLR.Drawing
>
> Needed Namespaces: GHIElectronics.TinyCLR.Devices.Gpio, GHIElectronics.TinyCLR.Devices.I2c, GHIElectronics.TinyCLR.Drivers.Omnivision.Ov9655, GHIElectronics.TinyCLR.Native, GHIElectronics.TinyCLR.Pins, System.Diagnostics, GHIElectronics.TinyCLR.Devices.Display, System.Drawing

```cs
GpioPin backlight = GpioController.GetDefault().OpenPin(SC20260.GpioPin.PA15);
backlight.SetDriveMode(GpioPinDriveMode.Output);
backlight.Write(GpioPinValue.High);

var displayController = Display.DisplayController.GetDefault();

var controllerSetting = 
    new ParallelDisplayControllerSettings {
    Width = 480,
    Height = 272,
    DataFormat = DisplayDataFormat.Rgb565,
    Orientation = DisplayOrientation.Degrees0, //Rotate display.
    PixelClockRate = 10000000,
    PixelPolarity = false,
    DataEnablePolarity = false,
    DataEnableIsFixed = false,
    HorizontalFrontPorch = 2,
    HorizontalBackPorch = 2,
    HorizontalSyncPulseWidth = 41,
    HorizontalSyncPolarity = false,
    VerticalFrontPorch = 2,
    VerticalBackPorch = 2,
    VerticalSyncPulseWidth = 10,
    VerticalSyncPolarity = false,
};

displayController.SetConfiguration(controllerSetting);
displayController.Enable();

var screen = Graphics.FromHdc(displayController.Hdc);
var controller = I2cController.FromName(SC20260.I2cBus.I2c1);

// Camera
var ov9655 = new Ov9655Controller(controller);
ov9655.FrameReceived += Ov9655_FrameReceived;
var id = ov9655.ReadId();

Debug.WriteLine("id = " + id);

ov9655.SetResolution(Ov9655Controller.Resolution.Vga);

byte temp = 0;

while (true) {
    try {
        ov9655.Capture();		
    }
    catch (System.Exception) { 
    }
	
	Thread.Sleep(10);
}

private static void Ov9655_FrameReceived(byte[] data, int size) {
	// 480 is screen width
	// 272 is screen height
	// 640 is original image width with VGA = 640
	displayController.DrawBuffer(0, 0, 0, 0, 480, 272, 640, data, 0);
}
```

