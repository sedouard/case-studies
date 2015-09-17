---
layout: post
title: "Talking AllJoyn with Universal Windows Platform (UWP)"
author: "Jason Poon"
author-link: "http://www.jasonpoon.ca"
date: "2015-09-15 23:00:00"
tags: uwp, alljoyn, win10
color: "blue"
coderesource: "https://github.com/jpoon/HeavenFresh-AllJoyn-Example"
image: "images/2015-09-15-Talking-AllJoyn-with-Universal-Windows-Platform/alljoyn-architecture.png"
excerpt: "Talking AllJoyn with Universal Windows Platform"
---

Windows 10 introduced support for AllJoyn giving any Windows 10 device -- phone, desktop, tablet -- the ability to connect and control multiple Internet of Things (IoT) devices like [@HeavenFresh](https://twitter.com/heavenfresh) air purifiers and humidifiers. In this case study, we will delve into the process of how we added AllJoyn support to the HeavenFresh Universal Windows Platform (UWP) app enabling the app to discover and control AllJoyn-compliant HeavenFresh air purifiers and humidifers.

## AllJoyn: A Brief Introduction

Driven by the [AllSeen Alliance](https://allseenalliance.org/), [AllJoyn](https://allseenalliance.org/developers/learn) is an open-source framework designed for interoperablity between Internet of Things (IoT) devices. The framework provides a common language for devices to discover, communicate, and interact directly with other nearby devices enabling scenarios like an AllJoyn-compliant air quality monitor directly controlling the settings of an AllJoyn-compliant purifier and, at the same time, sending notifications to an AllJoyn-compliant television to display the current air quality on-screen.

## Customer Problem

![HeavenFresh Logo]({{site.baseurl}}/images/2015-09-15-Talking-AllJoyn-with-Universal-Windows-Platform/heavenfresh-logo.jpg)

[HeavenFresh](http://www.heavenfresh.com/) makes a number of products for the home including [AllJoyn](https://ms-iot.github.io/content/en-US/win10/AllJoyn.htm) connected air purifiers and humidifers. In preparation for [IFA Berlin 2015](http://b2b.ifa-berlin.com/), HeavenFresh needed a Windows 10 app able to:

* discover the AllJoyn-compliant air purifiers and humidiers on the local network
* view and monitor current settings
* adjust configurations to desired levels

For example, in the case of the humidifer, through the application, a user should be able to view the current state of the humidifer (Is it on? What is the current humidity level?) and adjust settings such as turning on the mist level to high or turning on/off the humidifer in 2 hours. 

## Overview of the Solution

With Windows 10, the [AllJoyn Router](https://allseenalliance.org/developers/learn/architecture) runs as a Windows Service; as a result, Windows 10 devices have native support for discovering and communicating with other AllJoyn devices/apps including the HeavenFresh air purifier and humidifer. We held a hackfest to build the HeavenFresh [Universal Windows Platform (UWP)](https://msdn.microsoft.com/en-us/library/windows/apps/Dn958439.aspx) app which would act as a controller able to onboard HeavenFresh devices to the wireless network and view and adjust the current settings of the air purifiers/humidifers. 

AllJoyn devices/apps generally fall into two categories: producer or consumer. A producer produces data/notifications and listens for instructions. A consumer, on the otherhand, connects to one or more producers and consumes and controls producers. The UWP application is a consumer that connects to two producers: the HeavenFresh air purifier and the HeavenFresh humidifer.

![HeavenFresh UWP Network Architecture]({{site.baseurl}}/images/2015-09-15-Talking-AllJoyn-with-Universal-Windows-Platform/alljoyn-architecture.png)

## Implementation

The following will outline the steps taken to create the UWP application with native AllJoyn support.

### Setup

Before we begin writing code, let's ensure our developer machine is:

1. Running Windows 10 
2. [Visual Studio 2015](https://www.visualstudio.com/en-us/downloads/download-visual-studio-vs.aspx) with [Windows developer tools](https://dev.windows.com/en-us/downloads) installed
3. Producer AllJoyn devices are turned on and connected to the same network and subnet as the developer machine

### About Service

In order to be discovered by Windows 10 UWP applications, the AllJoyn producer device needs to implement [About-based discovery](https://allseenalliance.org/developers/learn/core/system-description/advertisement-discovery). The [About Service](https://allseenalliance.org/developers/learn/core/about-announcement) serves as a way for a device to advertise itself and the various service interfaces it implements. Given the interface definitions which are defined in [D-Bus introspection](http://go.microsoft.com/fwlink/p/?LinkId=616545) format, a client application (consumer) can build the necessary method calls to communicate to said AllJoyn device. 

Both HeavenFresh devices, the air purifier and humidifer, have implemented the About Service. Shown below is the introspection XML for the HeavenFresh humidifer illustrating the methods, properties, and signals that the device supports.

```
<node name="org/alljoyn/Bus/ControlPanel">
    <interface name="org.alljoyn.ControlPanel.Humidifier">
        
        <method name="PowerON"/>
        <method name="PowerOFF"/>
        <method name="SendSoftwareUpgradeFile">
            <arg name="currentIndex" type="u" direction="in">
            </arg>
            <arg name="fileData" type="ay" direction="in">
            </arg>
        </method>
        
        <property name="powerStatus" type="i" access="readwrite"/>
        <property name="humidityValue" type="i" access="readwrite"/>
        <property name="ionStatus" type="i" access="readwrite"/>
        <property name="warmMistStatus" type="i" access="readwrite"/>
        <property name="mistVolumeValue" type="i" access="readwrite"/>
        <property name="timerValue" type="i" access="readwrite"/>
        <property name="currentRoomTempValue" type="i" access="read"/>
        <property name="currentRoomHumidityValue" type="i" access="read"/>
        <property name="waterTankStatus" type="i" access="read"/>
        
        <signal name="currentRoomTempValueChanged">
            <arg name="newCurrentRoomTempValue" type="i">
            </arg>
        </signal>
        <signal name="currentRoomHumidityValueChanged">
            <arg name="newCurrentRoomHumidityValue" type="i">
            </arg>
        </signal>
        <signal name="waterTankStatusChanged">
            <arg name="newWaterTankStatus" type="i">
            </arg>
        </signal>
        <signal name="powerStatusChanged">
            <arg name="newPowerStatus" type="i">
            </arg>
        </signal>
        <signal name="humidityValueChanged">
            <arg name="newHumidityValue" type="i">
            </arg>
        </signal>
        <signal name="ionStatusChanged">
            <arg name="newIonStatus" type="i">
            </arg>
        </signal>
        <signal name="warmMistStatusChanged">
            <arg name="newWarmMistStatus" type="i">
            </arg>
        </signal>
        <signal name="mistVolumeValueChanged">
            <arg name="newMistVolumeValue" type="i">
            </arg>
        </signal>
        <signal name="timerValueChanged">
            <arg name="newTimerValue" type="i">
            </arg>
        </signal>
    </interface>
</node>
```

### AllJoyn Studio

The [AllJoyn Studio Visual Studio Extension](https://visualstudiogallery.msdn.microsoft.com/064e58a7-fb56-464b-bed5-f85914c89286) will query the local network for AllJoyn producers, extract their introspection XML, and based on the introspection XML auto-generate C++ [Windows Runtime (WinRT) Component(s)](https://msdn.microsoft.com/en-us/library/windows/apps/hh441572.aspx) that can be directly consumed by our UWP application. 

![AllJoyn Studio]({{site.baseurl}}/images/2015-09-15-Talking-AllJoyn-with-Universal-Windows-Platform/codegen.png)


#### Creating an AllJoyn App

Installing the AllJoyn Studio extension will add a new project template: AllJoyn App. 

![Visual Studio - AllJoyn App]({{site.baseurl}}/images/2015-09-15-Talking-AllJoyn-with-Universal-Windows-Platform/alljoyn-studio-new-alljoyn-app.png)

The next step of the wizard will query the network for AllJoyn devices that are advertising itself. As a result, ensure that your AllJoyn producer devices are turned on and are connected to the same network as your current Windows 10 developer machine. All discovered interfaces are then displayed allowing us to choose which interfaces to implement. 

With the HeavenFresh humidifer and air purifier in our local network, we see the following interfaces:

![Visual Studio - Discovery]({{site.baseurl}}/images/2015-09-15-Talking-AllJoyn-with-Universal-Windows-Platform/alljoyn-studio-discovered-interfaces.png)

After selecting the interfaces, Visual Studio and the AllJoyn Studio extension will use the introspection XMLs and [AllJoyn Code Generator](https://ms-iot.github.io/content/en-US/win10/AllJoynCodeGen.htm) to generate the WinRT components. The resulting WinRT component can then be referenced in any UWP-supported language (C#, C++, JavaScript, Visual Basic). 

![Visual Studio - Solution]({{site.baseurl}}/images/2015-09-15-Talking-AllJoyn-with-Universal-Windows-Platform/alljoyn-studio-solution.png)

At this point, we can start implementing AllJoyn functionality into our application. 

### Get on the (AllJoyn) Bus

The generated code includes two classes: watcher and a consumer.

#### Watcher

The watcher monitors the local network for the given AllJoyn producer and fires an event when the producer is found or stopped. For instance, the [AirPurifierWatcher](https://github.com/jpoon/HeavenFresh-AllJoyn-Example/blob/bc4931fe17a59168673effef233228d7d6646104/org.alljoyn.ControlPanel.AirPurifier/AirPurifierWatcher.cpp) will fire every time an air purifier is discovered.

In order to start the watcher, we first create an AllJoyn connection using the [AllJoyn Bus Attachment](https://msdn.microsoft.com/en-us/library/windows/apps/windows.devices.alljoyn.alljoynbusattachment).

```
_busAttachment = new AllJoynBusAttachment();

var airPurifierWatcher = new AirPurifierWatcher(_busAttachment);
airPurifierWatcher.Added += AirPurifierWatcher_Added;
airPurifierWatcher.Start();
```

The `Added` event fires every time an air purifier is discovered. If multiple purifiers are on the same local network, an instance of `AirPurifierConsumer` is instantiated per purifier found. We track the instances in the `NavMenuItem` which will eventually be passed along to the `AirPurifierPage` when a user chooses a new `NavMenuItem` in the navigation pane. 

```
private async void AirPurifierWatcher_Added(AirPurifierWatcher sender, AllJoynServiceInfo args)
{
    AirPurifierJoinSessionResult joinResult = await AirPurifierConsumer.JoinSessionAsync(args, sender);
    
    if (joinResult.Status == AllJoynStatus.Ok)
    {
        var airPurifierConsumer = joinResult.Consumer;

        // Add a new navigation menu item
        var newMenuItem = new NavMenuItem
        {
            Label = "Air Purifier",
            DestPage = typeof(AirPurifierPage),
            Arguments = airPurifierConsumer
        };
        
        // Update the UI
        await Dispatcher.RunAsync(CoreDispatcherPriority.Normal, delegate { _deviceList.Add(newMenuItem); })
    }
    else
    {
        // error
    }
}
```

#### Consumer

The consumer class exposes the functionality to control the AllJoyn producer. The auto-generated [AirPurifierConsumer](https://github.com/jpoon/HeavenFresh-AllJoyn-Example/blob/bc4931fe17a59168673effef233228d7d6646104/org.alljoyn.ControlPanel.AirPurifier/AirPurifierConsumer.cpp) contains the following implementations:

```
// AirPurifierConsumer
public IAsyncOperation<AirPurifierGetFlowValueResult> GetFlowValueAsync();
public IAsyncOperation<AirPurifierGetPowerStatusResult> GetPowerStatusAsync();
public IAsyncOperation<AirPurifierGetSensorAllergenValueResult> GetSensorAllergenValueAsync();
public IAsyncOperation<AirPurifierGetSensorCleanMetalGridValueResult> GetSensorCleanMetalGridValueAsync();
public IAsyncOperation<AirPurifierGetSensorCleanMonitorValueResult> GetSensorCleanMonitorValueAsync();
public IAsyncOperation<AirPurifierGetSensorDustValueResult> GetSensorDustValueAsync();
public IAsyncOperation<AirPurifierGetSensorOdorValueResult> GetSensorOdorValueAsync();
public IAsyncOperation<AirPurifierGetSensorReplaceFilterValueResult> GetSensorReplaceFilterValueAsync();
public IAsyncOperation<AirPurifierGetTimerValueResult> GetTimerValueAsync();
public IAsyncOperation<AirPurifierPowerOFFResult> PowerOFFAsync();
public IAsyncOperation<AirPurifierPowerONResult> PowerONAsync();
public IAsyncOperation<AirPurifierResetResult> ResetAsync();
public IAsyncOperation<AirPurifierSendSoftwareUpgradeFileResult> SendSoftwareUpgradeFileAsync(uint interfaceMemberCurrentIndex, IReadOnlyList<byte> interfaceMemberFileData);
public IAsyncOperation<AirPurifierSetFlowValueResult> SetFlowValueAsync(int value);
public IAsyncOperation<AirPurifierSetPowerStatusResult> SetPowerStatusAsync(int value);
public IAsyncOperation<AirPurifierSetTimerValueResult> SetTimerValueAsync(int value);
```

Note that each method is directly related to the introspection XML that was shown earlier for the air purifier.

## Opportunities for Reuse

The approach described in this case study can be leveraged to add AllJoyn support to any UWP application. In addition, the WinRT components generated against the HeavenFresh air purifier and humidifers are available on [GitHub](https://github.com/jpoon/HeavenFresh-AllJoyn-Example) if you would like to write your own HeavenFresh controller.