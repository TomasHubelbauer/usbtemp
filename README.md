# USBTemp

I am playing around with one of these USBTemp sensors you can get on
https://usbtemp.com.

The person who makes them is not a Mac owner so the installation and usage guide
is only available for Windows and Linux.

In order to get the sensor working to the extent that I already have, I've had
to do a couple of steps that I collect here.

Right out of the box, when plugged into a Mac, the sensor won't show up in
`/dev` when doing `ls /dev` and comparing the output with and without the sensor
in the laptop.

I found a macOS application called Serial which is great at troubleshooting USB
to serial issues on macOS:
https://www.decisivetactics.com/products/serial

This application bundles its own drivers so it can see even devices the rest of
the OS doesn't.
It was able to detect this sensor and open serial console so I could talk to it.
It also detected that the sensor uses a Prolific make sensor part.
I think it might be a PL2303 thermometer.

This information helped me realize I'll need a Prolific USB to serial driver in
order to get this dongle to work system-wide.
I looked up information about Prolific PL2303 on macOS and found this page:
https://www.prolific.com.tw/US/ShowProduct.aspx?pcid=41

It lists a couple of driver downloads, among which is the macOS one.
It downloads a macOS application installer package which install an application
named PLVCExtensionApp.
The installer also installs a kernel extension, which is something that needs to
be allowed manually in System Preferences > Security during the installation
process.

Once installed, the application can be pinned in the Dock.
I am not sure if the application itself needs to be running.
I don't think it needs to be running, the kernel extension should take care of
the USB to serial support alone.

With this driver installed, the Serial application will show more information
about the dongle, incluing the OS path to it. In my case that is
`/dev/cu.usbserial-CDDBb115819`.

I think the `CDDBb115819` might be the device serial number, but I am not sure.

With this path in mind, it is possible to try to use the Node SDK to read the
temperature off the device.
The Node SDK is available here:
https://github.com/usbtemp/usbtemp-nodejs

When I first tried it, I ran into issues with the `serialport` dependency being
too old and it would no longer compile with the version of Python that ships
with macOS.
It uses Node Gyp to compile native modules as a part of the package installation
script.

I managed to update the SDK to the latest version of `serialport` and the author
of the SDK was responsive as well and did essentially the same changes which he
pushed to the GitHub repository of the SDK.

The GitHub SDK is now working with an up-to-date version of `serialport` then
and the `npm install` step goes smoothly with no hiccup.
However, I am still not able to get the device to work.
For some reason, unlike for the developer who uses Debian to test the script,
in my case the `data` event of the serial port abstraction doesn't get called
in the right way.
At first, only `readable` will get call, but a call to `open` will return data.
When using `readable` to get past this initial exchange, I can progress in the
communication flow between the script and the device.
But in later stages, it is `data` that is being invoked and not `readable`.
It is possible that `readable` is only called once.

Maybe the behavior of the macOS driver is different from the one used in Linux.

I might invest more time to getting this to work in the future or I might end up
using a Nuc or a Raspi instead to talk to the sensor.

- [ ] Retry with the changes made by the SDK maintainer in the Node SDK repo
