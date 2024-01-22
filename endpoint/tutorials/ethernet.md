# Ethernet
---
Ethernet is supported through the internal MAC, by adding an external PHY (100BASE), and also by using an ENC28J60 over [SPI](spi.md) bus (10BASE). Some available modules include the necessary PHY, so the user will only need to add an Ethernet connector with magnets.

>[!IMPORTANT]
>You will need to ship each device with a valid and unique MAC address. More information about MAC addresses can be found [here](networking-core.md).

## Built-in Ethernet

Here is a simple example:

>[!TIP]
>Needed NuGets: GHIElectronics.TinyCLR.Devices.Network, GHIElectronics.TinyCLR.Devices.Gpio, GHIElectronics.TinyCLR.Pins

```cs
static bool linkReady = false;

static void EthernetTest() {
    //Reset external phy.
    var gpioController = GpioController.GetDefault();
    var resetPin = gpioController.OpenPin(SC20260.GpioPin.PG3);

    resetPin.SetDriveMode(GpioPinDriveMode.Output); 
  
    resetPin.Write(GpioPinValue.Low);
    Thread.Sleep(100);

    resetPin.Write(GpioPinValue.High);
    Thread.Sleep(100);

    var networkController = NetworkController.FromName(SC20260.NetworkController.EthernetEmac);

    var networkInterfaceSetting = new EthernetNetworkInterfaceSettings();

    var networkCommunicationInterfaceSettings = new BuiltInNetworkCommunicationInterfaceSettings();

    networkInterfaceSetting.Address = new IPAddress(new byte[] { 192, 168, 1, 122 });
    networkInterfaceSetting.SubnetMask = new IPAddress(new byte[] { 255, 255, 255, 0 });
    networkInterfaceSetting.GatewayAddress = new IPAddress(new byte[] { 192, 168, 1, 1 });

    networkInterfaceSetting.DnsAddresses = new IPAddress[] 
        {new IPAddress(new byte[] { 75, 75, 75, 75 }),
         new IPAddress(new byte[] { 75, 75, 75, 76 })};

    networkInterfaceSetting.MacAddress = new byte[] { 0x00, 0x4, 0x00, 0x00, 0x00, 0x00 };
    networkInterfaceSetting.DhcpEnable = true;
    networkInterfaceSetting.DynamicDnsEnable = true;

    networkController.SetInterfaceSettings(networkInterfaceSetting);
    networkController.SetCommunicationInterfaceSettings(networkCommunicationInterfaceSettings);

    networkController.SetAsDefaultController();

    networkController.NetworkAddressChanged += NetworkController_NetworkAddressChanged;
    networkController.NetworkLinkConnectedChanged += NetworkController_NetworkLinkConnectedChanged;

    networkController.Enable();

    while (linkReady == false) ;
    Debug.WriteLine("Network is ready to use");
    Thread.Sleep(Timeout.Infinite);
}

private static void NetworkController_NetworkLinkConnectedChanged
    (NetworkController sender, NetworkLinkConnectedChangedEventArgs e) {
    //Raise connect/disconnect event.
}

private static void NetworkController_NetworkAddressChanged
    (NetworkController sender, NetworkAddressChangedEventArgs e) {

    var ipProperties = sender.GetIPProperties();
    var address = ipProperties.Address.GetAddressBytes();

    linkReady = address[0] != 0;
    Debug.WriteLine("IP: " + address[0] + "." + address[1] + "." + address[2] + "." + address[3]);
}
```

---

## ENC28J60

This example uses the ENC28J60 click on our SC20260D Dev Board.

>[!TIP]
>Needed NuGets: GHIElectronics.TinyCLR.Devices.Network, GHIElectronics.TinyCLR.Devices.Gpio, GHIElectronics.TinyCLR.Devices.Spi, GHIElectronics.TinyCLR.Pins

```cs
static void Enc28Test() {
    var networkController = NetworkController.FromName
        (SC20260.NetworkController.Enc28j60);

    var networkInterfaceSetting = new EthernetNetworkInterfaceSettings();

    var networkCommunicationInterfaceSettings = new
        SpiNetworkCommunicationInterfaceSettings();

    var cs = GpioController.GetDefault().OpenPin(SC20260.GpioPin.PG12);

    var settings = new Spi.SpiConnectionSettings(){
        ChipSelectLine = cs,
        ClockFrequency = 4000000,
        Mode = GHIElectronics.TinyCLR.Devices.Spi.SpiMode.Mode0,
        ChipSelectType = SpiChipSelectType.Gpio,
        ChipSelectHoldTime = TimeSpan.FromTicks(10),
        ChipSelectSetupTime = TimeSpan.FromTicks(10)
    };

    networkCommunicationInterfaceSettings.SpiApiName = SC20260.SpiBus.Spi3;

    networkCommunicationInterfaceSettings.GpioApiName = SC20260.GpioPin.Id;

    
    networkCommunicationInterfaceSettings.SpiSettings = settings;
    networkCommunicationInterfaceSettings.InterruptPin = GetDefault().OpenPin(SC20260.GpioPin.PG6);

    networkCommunicationInterfaceSettings.InterruptEdge = GpioPinEdge.FallingEdge;
    networkCommunicationInterfaceSettings.InterruptDriveMode = GpioPinDriveMode.InputPullUp;
    networkCommunicationInterfaceSettings.ResetPin = GpioController.GetDefault().OpenPin(SC20260.GpioPin.PI8);

    networkCommunicationInterfaceSettings.ResetActiveState = GpioPinValue.Low;

    networkInterfaceSetting.Address = new IPAddress(new byte[] { 192, 168, 1, 122 });
    networkInterfaceSetting.SubnetMask = new IPAddress(new byte[] { 255, 255, 255, 0 });
    networkInterfaceSetting.GatewayAddress = new IPAddress(new byte[] { 192, 168, 1, 1 });
    networkInterfaceSetting.DnsAddresses = new IPAddress[] { new IPAddress(new byte[]
        { 75, 75, 75, 75 }), new IPAddress(new byte[] { 75, 75, 75, 76 }) };

    networkInterfaceSetting.MacAddress = new byte[] { 0x00, 0x4, 0x00, 0x00, 0x00, 0x00 };
    networkInterfaceSetting.DhcpEnable = true;
    networkInterfaceSetting.DynamicDnsEnable = true;

    networkController.SetInterfaceSettings(networkInterfaceSetting);
    networkController.SetCommunicationInterfaceSettings(networkCommunicationInterfaceSettings);

    networkController.SetAsDefaultController();

    networkController.NetworkAddressChanged += NetworkController_NetworkAddressChanged;
    networkController.NetworkLinkConnectedChanged += NetworkController_NetworkLinkConnectedChanged;

    networkController.Enable();

    while (linkReady == false) ;

    System.Diagnostics.Debug.WriteLine("Network is ready to use");

    Thread.Sleep(Timeout.Infinite);
}

private static void NetworkController_NetworkLinkConnectedChanged
    (NetworkController sender, NetworkLinkConnectedChangedEventArgs e) {

    // Raise event connect/disconnect
}

private static void NetworkController_NetworkAddressChanged
    (NetworkController sender, NetworkAddressChangedEventArgs e) {

    var ipProperties = sender.GetIPProperties();
    var address = ipProperties.Address.GetAddressBytes();
           
    linkReady = address[0] != 0;
    Debug.WriteLine("IP: " + address[0] + "." + address[1] + "." + address[2] + "." + address[3]);
}
```
---

## W5500

> [!TIP]
> Needed NuGets: GHIElectronics.TinyCLR.Drivers.BasicNet, GHIElectronics.TinyCLR.Drivers.BasicNet.Sockets, GHIElectronics.TinyCLR.Drivers.WIZnet.W5500

The Wiznet W5500 chipset has a built-in TCP/IP stack making it ideal for SC13048, which doesn't have built-in networking support, but it works with SC20xxx as well.

The example below does 'http GET' using ETH WIZ Click, on SC13048 Development board.

```cs
var cs = GpioController.GetDefault().OpenPin(SC13048.GpioPin.PB2);
var reset = GpioController.GetDefault().OpenPin(SC13048.GpioPin.PB15);
var interrupt = GpioController.GetDefault().OpenPin(SC13048.GpioPin.PA0);

var spiController = SpiController.FromName(SC13048.SpiBus.Spi1);

var networkController = new W5500Controller(spiController, cs, reset, interrupt); 
var isReady = false;

networkController.NetworkAddressChanged += (a, b) =>
{
    isReady = networkController.Address.GetAddressBytes()[0] != 0 && networkController.Address.GetAddressBytes()[1] != 0;
};

NetworkInterfaceSettings networkSettings = new NetworkInterfaceSettings()
{
    Address = new IPAddress(new byte[] { 192, 168, 0, 200 }),
    SubnetMask = new IPAddress(new byte[] { 255, 255, 255, 0 }),
    GatewayAddress = new IPAddress(new byte[] { 192, 168, 0, 1 }),
    DnsAddresses = new IPAddress[] { new IPAddress(new byte[] { 75, 75, 75, 75 }), new IPAddress(new byte[] { 75, 75, 75, 76 }) },

    MacAddress = new byte[] { your mac },
    DhcpEnable = false,
    DynamicDnsEnable = false,                
};
            

networkController.SetInterfaceSettings(networkSettings);
networkController.Enable();

while (isReady == false) ; // wait for valid IP address. 

var dns = new Dns(networkController);
var host = dns.GetHostEntry("www.example.com");
var ep = new IPEndPoint(host.AddressList[0], 80);

// Streaming socket
var socket = new Socket(networkController, AddressFamily.InterNetwork, SocketType.Stream, ProtocolType.Tcp);
socket.Connect(ep);


var content = "GET / HTTP/1.1\r\nHost: www.example.com\r\nConnection: close\r\n\r\n";
socket.Send(Encoding.UTF8.GetBytes(content));

var received = socket.Receive(SocketFlags.None);

if (received.Length > 0) {
    var recvdContent = new string(Encoding.UTF8.GetChars(received));
    Debug.WriteLine("Received : \r\n" + recvdContent);
}
```

## Event Handlers

NetworkController provides two events:

```cs
private static void NetworkController_NetworkLinkConnectedChanged
    (NetworkController sender, NetworkLinkConnectedChangedEventArgs e) {
    // Raise event connect/disconnect
}

private static void NetworkController_NetworkAddressChanged
    (NetworkController sender, NetworkAddressChangedEventArgs e) {

    var ipProperties = sender.GetIPProperties();
    var address = ipProperties.Address.GetAddressBytes();
    var subnet = ipProperties.SubnetMask.GetAddressBytes();
    var gw = ipProperties.GatewayAddress.GetAddressBytes();
    var interfaceProperties = sender.GetInterfaceProperties();

    for (int i = 0; i < dnsCount; i++){
	    var dns = ipProperties.DnsAddresses[i].GetAddressBytes();
    }
}
```


