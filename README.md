# Golang-iDevice
[![go doc](https://godoc.org/github.com/electricbubble/gidevice?status.svg)](https://pkg.go.dev/github.com/electricbubble/gidevice?tab=doc#pkg-index)
[![go report](https://goreportcard.com/badge/github.com/electricbubble/gidevice)](https://goreportcard.com/report/github.com/electricbubble/gidevice)
[![license](https://img.shields.io/github/license/electricbubble/gidevice)](https://github.com/electricbubble/gidevice/blob/master/LICENSE)

> much more easy to use 👉 [electricbubble/gidevice-cli](https://github.com/electricbubble/gidevice-cli)

## Installation

```shell script
go get github.com/electricbubble/gidevice
```

#### Devices

```go
package main

import (
	giDevice "github.com/electricbubble/gidevice"
	"log"
)

func main() {
	usbmux, err := giDevice.NewUsbmux()
	if err != nil {
		log.Fatalln(err)
	}

	devices, err := usbmux.Devices()
	if err != nil {
		log.Fatal(err)
	}

	for _, dev := range devices {
		log.Println(dev.Properties().SerialNumber, dev.Properties().ProductID, dev.Properties().DeviceID)
	}
}

```

#### DeveloperDiskImage

```go
package main

import (
	"encoding/base64"
	giDevice "github.com/electricbubble/gidevice"
	"log"
)

func main() {
	usbmux, err := giDevice.NewUsbmux()
	if err != nil {
		log.Fatal(err)
	}

	devices, err := usbmux.Devices()
	if err != nil {
		log.Fatal(err)
	}

	if len(devices) == 0 {
		log.Fatal("No Device")
	}

	d := devices[0]

	imageSignatures, err := d.Images()
	if err != nil {
		log.Fatalln(err)
	}

	for i, imgSign := range imageSignatures {
		log.Printf("[%d] %s\n", i+1, base64.StdEncoding.EncodeToString(imgSign))
	}

	dmgPath := "/Applications/Xcode.app/Contents/Developer/Platforms/iPhoneOS.platform/DeviceSupport/14.4/DeveloperDiskImage.dmg"
	signaturePath := "/Applications/Xcode.app/Contents/Developer/Platforms/iPhoneOS.platform/DeviceSupport/14.4/DeveloperDiskImage.dmg.signature"

	err = d.MountDeveloperDiskImage(dmgPath, signaturePath)
	if err != nil {
		log.Fatalln(err)
	}
}

```

#### App

```go
package main

import (
	giDevice "github.com/electricbubble/gidevice"
	"log"
	"path/filepath"
)

func main() {
	usbmux, err := giDevice.NewUsbmux()
	if err != nil {
		log.Fatalln(err)
	}

	devices, err := usbmux.Devices()
	if err != nil {
		log.Fatalln(err)
	}

	if len(devices) == 0 {
		log.Fatalln("No Device")
	}

	d := devices[0]

	bundleID := "com.apple.Preferences"
	pid, err := d.AppLaunch(bundleID)
	if err != nil {
		log.Fatalln(err)
	}

	err = d.AppKill(pid)
	if err != nil {
		log.Fatalln(err)
	}

	runningProcesses, err := d.AppRunningProcesses()
	if err != nil {
		log.Fatalln(err)
	}

	for _, process := range runningProcesses {
		if process.IsApplication {
			log.Printf("%4d\t%-24s\t%-36s\t%s\n", process.Pid, process.Name, filepath.Base(process.RealAppName), process.StartDate)
		}
	}
}

```

#### Screenshot

```go
package main

import (
	giDevice "github.com/electricbubble/gidevice"
	"image"
	"image/jpeg"
	"image/png"
	"log"
	"os"
)

func main() {
	usbmux, err := giDevice.NewUsbmux()
	if err != nil {
		log.Fatalln(err)
	}

	devices, err := usbmux.Devices()
	if err != nil {
		log.Fatalln(err)
	}

	if len(devices) == 0 {
		log.Fatalln("No Device")
	}

	d := devices[0]

	raw, err := d.Screenshot()
	if err != nil {
		log.Fatalln(err)
	}

	img, format, err := image.Decode(raw)
	if err != nil {
		log.Fatalln(err)
	}
	userHomeDir, _ := os.UserHomeDir()
	file, err := os.Create(userHomeDir + "/Desktop/s1." + format)
	if err != nil {
		log.Fatalln(err)
	}
	defer func() { _ = file.Close() }()
	switch format {
	case "png":
		err = png.Encode(file, img)
	case "jpeg":
		err = jpeg.Encode(file, img, nil)
	}
	if err != nil {
		log.Fatalln(err)
	}
	log.Println(file.Name())
}

```

#### SimulateLocation

```go
package main

import (
	giDevice "github.com/electricbubble/gidevice"
	"log"
)

func main() {
	usbmux, err := giDevice.NewUsbmux()
	if err != nil {
		log.Fatalln(err)
	}

	devices, err := usbmux.Devices()
	if err != nil {
		log.Fatalln(err)
	}

	if len(devices) == 0 {
		log.Fatalln("No Device")
	}

	d := devices[0]

	// https://api.map.baidu.com/lbsapi/getpoint/index.html
	if err = d.SimulateLocationUpdate(116.024067, 40.362639, giDevice.CoordinateSystemBD09); err != nil {
		log.Fatalln(err)
	}

	// https://developer.amap.com/tools/picker
	// https://lbs.qq.com/tool/getpoint/index.html
	// if err = d.SimulateLocationUpdate(120.116979, 30.252876, giDevice.CoordinateSystemGCJ02); err != nil {
	// 	log.Fatalln(err)
	// }

	// if err = d.SimulateLocationUpdate(121.499763, 31.239580,giDevice.CoordinateSystemWGS84); err != nil {
	// if err = d.SimulateLocationUpdate(121.499763, 31.239580); err != nil {
	// 	log.Fatalln(err)
	// }

	// err = d.SimulateLocationRecover()
	// if err != nil {
	// 	log.Fatalln(err)
	// }
}

```

#### XCTest

```go
package main

import (
	"fmt"
	giDevice "github.com/electricbubble/gidevice"
	"log"
	"os"
	"os/signal"
)

func main() {
	usbmux, err := giDevice.NewUsbmux()
	if err != nil {
		log.Fatal(err)
	}

	devices, err := usbmux.Devices()
	if err != nil {
		log.Fatal(err)
	}

	if len(devices) == 0 {
		log.Fatal("No Device")
	}

	d := devices[0]

	out, cancel, err := d.XCTest("com.leixipaopao.WebDriverAgentRunner.xctrunner")
	if err != nil {
		log.Fatal(err)
	}

	done := make(chan os.Signal, 1)
	signal.Notify(done, os.Interrupt)

	go func() {
		for s := range out {
			fmt.Print(s)
		}
	}()

	<-done
	cancel()
	fmt.Println()
	log.Println("DONE")
}

```

## Thanks

| |About|
|---|---|
|[libimobiledevice/libimobiledevice](https://github.com/libimobiledevice/libimobiledevice)|A cross-platform protocol library to communicate with iOS devices|
|[anonymous5l/iConsole](https://github.com/anonymous5l/iConsole)|iOS usbmuxd communication impl iTunes protocol|
|[alibaba/taobao-iphone-device](https://github.com/alibaba/taobao-iphone-device)|tidevice can be used to communicate with iPhone device|

Thank you [JetBrains](https://www.jetbrains.com/?from=gwda) for providing free open source licenses
