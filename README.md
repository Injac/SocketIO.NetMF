SocketIO.NetMF
==============


SocketIO implementation for NetMF and .NET Gadgeteer.

Based on jasdev55's <a href="http://jdiwebsocketclient.codeplex.com/">JDI WebSocket Client </a> and on
Matt Weimer's <a href="https://github.com/mweimer/Json.NetMF">Json.MF</a> implementation


Requirements
==============

Microsoft .NET Micro Framework 4.2 or higher


Example Gadgeteer application
==============


<code>
    
    class SocketIOClient : SocketIO 
    {
        
        override public void onConnect() 
        { 
            Debug.Print("SocketIO connected");

            // after connected, client can start emiting events, e.g. login event with
            emit("login", new ArrayList() { "my_identity_goes_here" });

        }

        override public void onDisconnect() { Debug.Print("1 disconnected"); }
        override public void onHeartbeat() { Debug.Print("got heartbeat"); }
        override public void onMessage(string message) { Debug.Print("got messag: " + message); }
        override public void onJsonMessage(Hashtable jsonObject) { Debug.Print("got json object"); }
        override public void onEvent(string name, ArrayList args) { Debug.Print("got event: " + name); }

        override public void onError(string reason) { throw new Exception(reason); }

    }

    public partial class Program
    {
        // host server details
        private static string _host = "localhost";
        private static string _port = "8080";

        // instance of socketIO client
        private SocketIOClient socketIOClient = null;

        // WIFI details
        private static string wlanName = "my_ssid";
        private static string wlanPassword = "my_passwd";



        // This method is run when the mainboard is powered up or reset.
        void ProgramStarted()
        {

            // initializes wifi
            initWifiConnection();

            // create new socketIO client
            socketIOClient = new SocketIOClient();

            // connect to socketIO server when button is pressed
            button.ButtonPressed += new GTM.GHIElectronics.Button.ButtonEventHandler((o,s) => {
                socketIOClient.connect(_host, _port);
                Debug.Print("button pressed!");
            });

            
            Debug.Print("Program Started");
        }




        void initWifiConnection()
        {
            Debug.Print("connecting to: " + wlanName);
            if (wifi.Interface.IsOpen)
            {
                Debug.Print("interface was open");
            }
            else
            {
                Debug.Print("interface was not open");
                wifi.Interface.Open();
            }
            wifi.Interface.WirelessConnectivityChanged += new GHI.Premium.Net.WiFiRS9110.WirelessConnectivityChangedEventHandler(Interface_WirelessConnectivityChanged);

            wifi.DebugPrintEnabled = true;
            wifi.UseDHCP();

            GHI.Premium.Net.WiFiNetworkInfo info = new GHI.Premium.Net.WiFiNetworkInfo();
            info.SSID = wlanName;
            info.SecMode = GHI.Premium.Net.SecurityMode.WPA2;
            info.networkType = GHI.Premium.Net.NetworkType.AccessPoint;

            wifi.Interface.Join(info, wlanPassword);
            wifi.UseThisNetworkInterface();
        }

        void Interface_WirelessConnectivityChanged(object sender, GHI.Premium.Net.WiFiRS9110.WirelessConnectivityEventArgs e)
        {
            Debug.Print("wifi conn changed!");
            if (e.IsConnected)
            {
                Debug.Print("WIFI ("+wlanName+") connected!");
            }
            else
            {
                Debug.Print("WIFI ("+wlanName+") disconnected..");
            }
        }
    }

</code>