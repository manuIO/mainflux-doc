# Mainflux CLI
Mainflux CLI is a command line tool that can be used to provision, configure and administer Mainflux server.

It is basically a HTTP client program that connects to the Mainflux HTTP server and sends
HTTP requests to it's device management API. This saves you form issuing manual `curl` commands
and makes development, testing and administration of Mainflux IoT Server easier.

## Install
Mainflux CLI can be installed from it's [GitHub repo](https://github.com/mainflux/mainflux-cli):
```bash
go get github.com/mainflux/mainflux-cli
```

## Usage
Start the CLI with:
```
$GOBIN/mainflux-cli
```

And then you can inspect the help:

```
███╗   ███╗ █████╗ ██╗███╗   ██╗███████╗██╗     ██╗   ██╗██╗  ██╗
████╗ ████║██╔══██╗██║████╗  ██║██╔════╝██║     ██║   ██║╚██╗██╔╝
██╔████╔██║███████║██║██╔██╗ ██║█████╗  ██║     ██║   ██║ ╚███╔╝ 
██║╚██╔╝██║██╔══██║██║██║╚██╗██║██╔══╝  ██║     ██║   ██║ ██╔██╗ 
██║ ╚═╝ ██║██║  ██║██║██║ ╚████║██║     ███████╗╚██████╔╝██╔╝ ██╗
╚═╝     ╚═╝╚═╝  ╚═╝╚═╝╚═╝  ╚═══╝╚═╝     ╚══════╝ ╚═════╝ ╚═╝  ╚═╝

                == Industrial IoT System ==
       
                Made with <3 by Mainflux Team
[w] http://mainflux.io
[t] @mainflux


>>> help
Mainflux CLI is a command line tool for administration and provisioning of Mainflux IoT server.
More information can be found on project's website: http://mainflux.io

Commands:
	status          Mainflux server health check
	devices         Manipulation with devices. Do 'devices help' for info.
	channels        Manipulation with channels. Do 'channels help' for info.
	
	clear           Clears the screen
	exit            Exits the CLI
	
	help            Prints this help
>>>  
```

You can also obain the help for the individual commands like:
```
>>> devices help
Commands:
	create                                  Creates new device and generates it's UUID
	get                                     Gets all devices
	get <device_id>                         Gets device by id
	update <device_id> <JSON_string>        Updates device record
	delete <device_id                       Removes device
	plug <device_d> <JSON_channels_list>    Plugs device into the channel(s)
>>>
```

Commands are self-explanatory. For example, listing all the
channels provisioned in the Mainflux system is as easy as `channels get`:
```
>>> channels get
[
    {
        "id": "f786a740-7716-46df-82a4-f820ddca5f6f",
        "visibility": "private",
        "owner": "",
        "entries": [],
        "devices": [
            "472dceec-9bc2-4cd4-9f16-bf3b8d1d3c52",
            "a76539f2-8cdf-4568-bdbd-f1c617e5516a"
        ],
        "created": "2016-12-10T17:45:28Z",
        "updated": "2016-12-10T17:46:50Z",
        "metadata": {}
    },
    {
        "id": "66a83b9a-9ed6-4257-ad85-7a2542ac2167",
        "visibility": "private",
        "owner": "",
        "entries": [],
        "devices": [
            "472dceec-9bc2-4cd4-9f16-bf3b8d1d3c52",
            "a76539f2-8cdf-4568-bdbd-f1c617e5516a"
        ],
        "created": "2016-12-10T17:45:27Z",
        "updated": "2016-12-10T17:46:50Z",
        "metadata": {}
    }
]
>>>
```

Maybe only `plug` command demands a bit more explanation.
This is a command that actually binds a device to a channel and vice versa.
Channel should be observed as a data bus, and devices plug and unplug on/from that bus.

To plug list of devices into a channel execute the following command:
```
>>> channels plug a57cc963-c152-4fd2-9398-59495916babe '["66990b46-8736-4182-ba21-8dabaadb27ff", "97bd76d3-8f8f-4e1f-8c20-4a6c84d3575f", "a76539f2-8cdf-4568-bdbd-f1c617e5516a"]'
{
    "response": "plugged",
    "id": "a57cc963-c152-4fd2-9398-59495916babe"
}
```

where `a57cc963-c152-4fd2-9398-59495916babe` is `ChannelID` of the channel you are plugging the devices into, and second argument is JSON array of `DeviceID`s.

> Note that	`DeviceID` is of type `string`, so it has to be quoted in the JSON array.

Then observe that channels have been registred in Channel's
```
>>> channels get
[
    {
        "id": "a57cc963-c152-4fd2-9398-59495916babe",
        "visibility": "private",
        "owner": "",
        "entries": [],
        "devices": [
            "66990b46-8736-4182-ba21-8dabaadb27ff",
            "97bd76d3-8f8f-4e1f-8c20-4a6c84d3575f",
            "a76539f2-8cdf-4568-bdbd-f1c617e5516a"
        ],
        "created": "2016-12-11T18:17:06Z",
        "updated": "2016-12-11T19:16:24Z",
        "metadata": {}
    }
]
>>>
```

You can verify that channel is also recorded in the Device's registry of channels it is plugged into:
```
>>> devices get 66990b46-8736-4182-ba21-8dabaadb27ff
{
    "id": "66990b46-8736-4182-ba21-8dabaadb27ff",
    "name": "Some Name",
    "description": "",
    "online": false,
    "connected_at": "",
    "disconnected_at": "",
    "channels": [
        "a57cc963-c152-4fd2-9398-59495916babe"
    ],
    "created": "2016-12-11T19:09:50Z",
    "updated": "2016-12-11T19:16:24Z",
    "metadata": {}
}
>>>  
```


