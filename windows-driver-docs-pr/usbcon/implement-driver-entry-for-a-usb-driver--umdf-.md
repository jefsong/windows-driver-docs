---
Description: Use the USB User-Mode Driver template provided with Microsoft Visual Studio to write a UMDF client driver.
title: How to write your first USB client driver (UMDF)
author: windows-driver-content
ms.author: windowsdriverdev
ms.date: 04/20/2017
ms.topic: article
ms.prod: windows-hardware
ms.technology: windows-devices
---

# How to write your first USB client driver (UMDF)


In this topic you'll use the **USB User-Mode Driver** template provided with Microsoft Visual Studio 2012 to write a user-mode driver framework (UMDF)-based client driver. After building and installing the client driver, you'll view the client driver in **Device Manager** and view the driver output in a debugger.

UMDF (referred to as the framework in this topic) is based on the component object model (COM). Every framework object must implement [**IUnknown**](https://msdn.microsoft.com/library/windows/desktop/ms680509) and its methods, [**QueryInterface**](https://msdn.microsoft.com/library/windows/desktop/ms682521), [**AddRef**](https://msdn.microsoft.com/library/windows/desktop/ms691379), and [**Release**](https://msdn.microsoft.com/library/windows/desktop/ms682317), by default. The **AddRef** and **Release** methods manage the object's lifetime, so the client driver does not need to maintain the reference count. The **QueryInterface** method enables the client driver to get interface pointers to other framework objects in the Windows Driver Frameworks (WDF) object model. Framework objects perform complicated driver tasks and interact with Windows. Certain framework objects expose interfaces that enable a client driver to interact with the framework.

A UMDF-based client driver is implemented as an in-process COM server (DLL), and C++ is the preferred language for writing a client driver for a USB device. Typically, the client driver implements several interfaces exposed by the framework. This topic refers to a client driver-defined class that implements framework interfaces as a callback class. After these classes are instantiated, the resulting callback objects are partnered with particular framework objects. This partnership gives the client driver the opportunity to respond to device or system-related events that are reported by the framework. Whenever Windows notifies the framework about certain events, the framework invokes the client driver's callback, if one is available. Otherwise the framework proceeds with the default processing of the event. The template code defines driver, device, and queue callback lasses.

For an explanation about the source code generated by the template, see [Understanding the UMDF template code for USB client driver](understanding-the-umdf-template-code-for-usb.md).

### Prerequisites

For developing, debugging, and installing a user-mode driver, you need two computers:

-   A host computer running Windows 7 or a later version of the Windows operating system. The host computer is your development environment, where you write and debug your driver.
-   A target computer running the version of the operating system that you want to test your driver on, for example, Windows 8. The target computer has the user-mode driver that you want to debug and one of the debuggers.

In some cases, where the host and target computers are running the same version of Windows, you can have just one computer running Windows 7 or a later version of the Windows. This topic assumes that you are using two computers for developing, debugging, and installing your user mode driver.

Before you begin, make sure that you meet the following requirements:

**Software requirements**

-   Your host computer has Visual Studio 2012.
-   Your host computer has the latest Windows Driver Kit (WDK) for Windows 8.

    The kit include headers, libraries, tools, documentation, and the debugging tools required to develop, build, and debug a USB client driver. You can get the latest version of the WDK from [How to Get the WDK](http://go.microsoft.com/fwlink/p/?linkid=617585).

-   Your host computer has the latest version of debugging tools for Windows. You can get the latest version from the WDK or you can [Download and Install Debugging Tools for Windows](http://go.microsoft.com/fwlink/p/?linkid=617701).
-   If you are using two computers, you must configure the host and target computers for user-mode debugging. For more information, see [Setting Up User-Mode Debugging in Visual Studio](https://msdn.microsoft.com/library/windows/hardware/hh439381).

**Hardware requirements**

Get a USB device for which you will be writing the client driver. In most cases, you are provided with a USB device and its hardware specification. The specification describes device capabilities and the supported vendor commands. Use the specification to determine the functionality of the USB driver and the related design decisions.

If you are new to USB driver development, use the OSR USB FX2 learning kit to study USB samples included with the WDK. You can get the learning kit from [OSR Online](http://go.microsoft.com/fwlink/p/?linkid=617553). It contains the USB FX2 device and all the required hardware specifications to implement a client driver.

**Recommended reading**

-   [Concepts for All Driver Developers](https://msdn.microsoft.com/library/windows/hardware/ff554731)
-   [Device nodes and device stacks](https://msdn.microsoft.com/library/windows/hardware/ff554721)
-   [Getting started with Windows drivers](https://msdn.microsoft.com/library/windows/hardware/ff554690)
-   [User-Mode Driver Framework](https://msdn.microsoft.com/library/windows/hardware/ff560027)
-   *Developing Drivers with Windows Driver Foundation*, written by Penny Orwick and Guy Smith. For more information, see [Developing Drivers with WDF](http://go.microsoft.com/fwlink/p/?linkid=617702).

Instructions
------------

### <a href="" id="generate-the-umdf-driver-code-by-using-the-visual-studio-2012-usb-driver-template"></a>Step 1: Generate the UMDF driver code by using the Visual Studio 2012 USB driver template

<a href="" id="generate"></a>
For instructions about generating UMDF driver code, see [Writing a UMDF driver based on a template](https://msdn.microsoft.com/library/windows/hardware/hh439659).

**For USB-specific code, select the following options in Visual Studio 2012**

1.  In the **New Project** dialog box, in the left pane, locate and select **USB.**
2.  In the middle pane, select **USB User-Mode Driver**.

The following screen shot shows **New Project** dialog box for the **USB User-Mode Driver** template.

![visual studio new project options](images/umdf-tmpl.png)

This topic assumes that the name of the project is "MyUSBDriver\_UMDF\_". It contains the following files:

| Files                      | Description                                                                                                                                                                                                                                                                                                                                                                                                                                  |
|----------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Driver.h; Driver.c         | Declares and defines a callback class that implements the [**IDriverEntry**](https://msdn.microsoft.com/library/windows/hardware/ff554885) interface. The class defines methods that are invoked by the framework driver object. The main purpose of this class is to create a device object for the client driver.                                                                                                                                                                     |
| Device.h; Device.c         | Declares and defines a callback class that implements the [**IPnpCallbackHardware**](https://msdn.microsoft.com/library/windows/hardware/ff556764) interface. The class defines methods that are invoked by the framework device object. The main purpose of this class is to handle events occurring as a result of Plug and Play (PnP) state changes. The class also allocates and initializes resources required by the client driver as long as it is loaded in the system. |
| IoQueue.h; IoQueue.c       | Declares and defines a callback class that implements the [**IQueueCallbackDeviceIoControl**](https://msdn.microsoft.com/library/windows/hardware/ff556852) interface. The class defines methods that are invoked by the framework queue object. The purpose of this class is to retrieve I/O requests that are queued in the framework.                                                                                                                               |
| Internal.h                 | Provides common declarations shared by the client driver and user applications that communicate with the USB device. It also declares tracing functions and macros.                                                                                                                                                                                                                                                                          |
| Dllsup.cpp                 | Contains the implementation of the driver module's entry point.                                                                                                                                                                                                                                                                                                                                                                              |
| *&lt;Project name&gt;*.inf | INF file that is required to install the client driver on the target computer.                                                                                                                                                                                                                                                                                                                                                               |
| Exports.def                | DEF file that exports the entry point function name of the driver module.                                                                                                                                                                                                                                                                                                                                                                    |

 

### <a href="" id="modify-the-inf-file-to-add-information-about-your-device"></a>Step 2: Modify the INF file to add information about your device

<a href="" id="modify"></a>
Before you build the driver, you must modify the template INF file with information about your device, specifically the hardware ID string.

**To provide the hardware ID string**

1.  Attach your USB device to your host computer and let Windows enumerate the device.
2.  Open **Device Manager** and open properties for your device.
3.  On the **Details** tab, select **Hardward Ids** under **Property.**

    The hardware ID for the device is displayed in the list box. Right-click and copy the hardware ID string.

4.  In **Solution Explorer**, expand **Driver Files**, and open the INF.
5.  Replace the following your hardware ID string.

    `[Standard.NT$ARCH$]`

    `%DeviceName%=MyDevice_Install, USB\VID_vvvv&PID_pppp`

Notice the **AddReg** entries in the driver's information (INF) file.

`[CoInstallers_AddReg] ;`

`HKR,,CoInstallers32,0x00010008,"WudfCoinstaller.dll"`

`HKR,,CoInstallers32,0x00010008,"WudfUpdate_01011.dll"`

`HKR,,CoInstallers32,0x00010008,"WdfCoInstaller01011.dll,WdfCoInstaller"`

`HKR,,CoInstallers32,0x00010008,"WinUsbCoinstaller2.dll"`

-   WudfCoinstaller.dll (configuration co-installer)
-   WUDFUpdate\_*&lt;version&gt;*.dll (redistributable co-installer)
-   Wdfcoinstaller*&lt;version&gt;*.dll (co-installers for KMDF)
-   Winusbcoinstaller2.dll ((co-installers for Winusb.sys)
-   MyUSBDriver\_UMDF\_.dll (client driver module)

If your INF AddReg directive references the UMDF redistributable co-installer (WUDFUpdate\_*&lt;version&gt;*.dll ), you must not make a reference to the configuration co-installer (WUDFCoInstaller.dll). Referencing both co-installers in the INF will lead to installation errors.

All UMDF-based USB client drivers require two Microsoft-provided drivers: the reflector and WinUSB.

-   Reflector—If your driver gets loaded successfully, the reflector is loaded as the top-most driver in the kernel-mode stack. The reflector must be the top driver in the kernel mode stack. To meet this requirement, the template's INF file specifies the reflector as a service and WinUSB as a lower-filter driver in the INF:

    `[MyDevice_Install.NT.Services]`

    `AddService=WUDFRd,0x000001fa,WUDFRD_ServiceInstall  ; flag 0x2 sets this as the service for the device`

    `AddService=WinUsb,0x000001f8,WinUsb_ServiceInstall  ; this service is installed because its a filter.`

-   WinUSB—The installation package must contain coinstallers for Winusb.sys because for the client driver, WinUSB is the gateway to the kernel-mode USB driver stack. Another component that gets loaded is a user-mode DLL, named WinUsb.dll, in the client driver's host process (Wudfhost.exe). Winusb.dll exposes [WinUSB Functions](https://msdn.microsoft.com/library/windows/hardware/ff540046#winusb) that simplify the communication process between the client driver and WinUSB.

### <a href="" id="build-the-usb-client-driver-code"></a>Step 3: Build the USB client driver code

<a href="" id="build"></a>
**To build your driver**

1.  Open the driver project or solution in Visual Studio 2012.
2.  Right-click the solution in the **Solution Explorer** and select **Configuration Manager**.
3.  From the **Configuration Manager**, select your **Active Solution Configuration** (for example, **Windows 8 Debug** or **Windows 8 Release**) and your **Active Solution Platform** (for example, Win32) that correspond to the type of build you are interested in.
4.  From the **Build** menu, click **Build Solution**.

For more information, see [Building a Driver](https://msdn.microsoft.com/windows-drivers/develop/building_a_driver).

### <a href="" id="configure-a-computer-for-testing-and-debugging"></a>Step 4: Configure a computer for testing and debugging

To test and debug a driver, you run the debugger on the host computer and the driver on the target computer. So far, you have used Visual Studio on the host computer to build a driver. Next you need to configure a target computer. To configure a target computer, follow the instructions in [Provision a computer for driver deployment and testing](https://msdn.microsoft.com/library/windows/hardware/dn745909).

### <a href="" id="enable-tracing-for-kernel-debugging"></a>Step 5: Enable tracing for kernel debugging

The template code contains several trace messages (TraceEvents) that can help you track function calls. All functions in the source code contain trace messages that mark the entry and exit of a routine. For errors, the trace message contains the error code and a meaningful string. Because WPP tracing is enabled for your driver project, the PDB symbol file created during the build process contains trace message formatting instructions. If you configure the host and target computers for WPP tracing, your driver can send trace messages to a file or the debugger.

**To configure your host computer for WPP tracing**

1.  Create trace message format (TMF) files by extracting trace message formatting instructions from the PDB symbol file.

    You can use Tracepdb.exe to create TMF files. The tool is located in the *&lt;install folder&gt;*Windows Kits\\8.0\\bin\\*&lt;architecture&gt;* folder of the WDK. The following command creates TMF files for the driver project.

    **tracepdb -f \[PDBFiles\] -p \[TMFDirectory\]**

    The **-f** option specifies the location and the name of the PDB symbol file. The **-p** option specifies the location for the TMF files that are created by Tracepdb. For more information, see [**Tracepdb Commands**](https://msdn.microsoft.com/library/windows/hardware/ff553043).

    At the specified location you'll see three files (one per .c file in the project). They are given GUID file names.

2.  In the debugger, type the following commands:
    1.  **.load Wmitrace**

        Loads the Wmitrace.dll extension.

    2.  **.chain**

        Verify that the debugger extension is loaded.

    3.  **!wmitrace.searchpath +***&lt;TMF file location&gt;*

        Add the location of the TMF files to the debugger extension's search path.

        The output resembles this:

        `Trace Format search path is: 'C:\Program Files (x86)\Microsoft Visual Studio 11.0\Common7\IDE;c:\drivers\tmf'`

**To configure your target computer for WPP tracing**

1.  Make sure you have the Tracelog tool on your target computer. The tool is located in the *&lt;install\_folder&gt;*Windows Kits\\8.0\\Tools\\*&lt;arch&gt;* folder of the WDK. For more information, see [**Tracelog Command Syntax**](https://msdn.microsoft.com/library/windows/hardware/ff553012).
2.  Open a **Command Window** and run as administrator.
3.  Type the following command:

    **tracelog -start MyTrace -guid \#c918ee71-68c7-4140-8f7d-c907abbcb05d -flag 0xFFFF -level 7-rt -kd**

    The command starts a trace session named MyTrace.

    The **guid** argument specifies the GUID of the trace provider, which is the client driver. You can get the GUID from Trace.h in the Microsoft Visual Studio Professional 2012 project. As another option, you can type the following command and specify the GUID in a .guid file. The file contains the GUID in hyphen format:

    **tracelog -start MyTrace -guid c:\\drivers\\Provider.guid -flag 0xFFFF -level 7-rt -kd**

    You can stop the trace session by typing the following command:

    **tracelog -stop MyTrace**

### <a href="" id="deploy-the-driver-on-the-target-computer"></a>Step 6: Deploy the driver on the target computer

1.  In the **Solution Explorer** window, right click the *&lt;project name&gt;***Package** , and choose **Properties**.
2.  In the left pane, navigate to **Configuration Properties &gt; Driver Install &gt; Deployment**.
3.  Check Enable deployment, and check Import into driver store.
4.  For **Remote Computer Name**, specify the name of the target computer.
5.  Select **Install and Verify**.
6.  Click **Ok**.
7.  On the **Debug** menu, choose **Start Debugging**, or press **F5** on the keyboard.

**Note**  Do *not* specify the hardware ID of your device under **Hardware ID Driver Update**. The hardware ID must be specified only in your driver's information (INF) file.

 

### <a href="" id="view-the-driver-in-device-manager"></a>Step 7: View the driver in Device Manager

<a href="" id="devicemanager"></a>
1.  Enter the following command to open **Device Manager**.

    **devmgmt**

2.  Verify that **Device Manager** shows the following node.

    **USB Device**

    **MyUSBDriver\_UMDF\_Device**

### <a href="" id="view-the-output-in-the-debugger"></a>Step 8: View the output in the debugger

Verify that trace messages appear in the **Debugger Immediate Window** on the host computer.

The output should be similar to the following.

``` syntax
[0]0744.05F0::00/00/0000-00:00:00.000 [MyUSBDriver_UMDF_]CMyDevice::OnPrepareHardware Entry
[0]0744.05F0::00/00/0000-00:00:00.000 [MyUSBDriver_UMDF_]CMyDevice::OnPrepareHardware Exit
[1]0744.05F0::00/00/0000-00:00:00.000 [MyUSBDriver_UMDF_]CMyDevice::CreateInstanceAndInitialize Entry
[1]0744.05F0::00/00/0000-00:00:00.000 [MyUSBDriver_UMDF_]CMyDevice::Initialize Entry
[1]0744.05F0::00/00/0000-00:00:00.000 [MyUSBDriver_UMDF_]CMyDevice::Initialize Exit
[1]0744.05F0::00/00/0000-00:00:00.000 [MyUSBDriver_UMDF_]CMyDevice::CreateInstanceAndInitialize Exit
[1]0744.05F0::00/00/0000-00:00:00.000 [MyUSBDriver_UMDF_]CMyDevice::Configure Entry
[1]0744.05F0::00/00/0000-00:00:00.000 [MyUSBDriver_UMDF_]CMyIoQueue::CreateInstanceAndInitialize Entry
[1]0744.05F0::00/00/0000-00:00:00.000 [MyUSBDriver_UMDF_]CMyIoQueue::Initialize Entry
[1]0744.05F0::00/00/0000-00:00:00.000 [MyUSBDriver_UMDF_]CMyIoQueue::Initialize Exit
[1]0744.05F0::00/00/0000-00:00:00.000 [MyUSBDriver_UMDF_]CMyIoQueue::CreateInstanceAndInitialize Exit
[1]0744.05F0::00/00/0000-00:00:00.000 [MyUSBDriver_UMDF_]CMyDevice::Configure Exit
```

Remarks
-------

Let’s take a look at how the framework and the client driver work together to interact with Windows and handle requests sent to the USB device. This illustration shows the modules loaded in the system for a UMDF -based USB client driver.

![user mode client driver architecture](images/umdfstack.png)

The purpose of each module is described here:

-   Application—a user-mode process that issues I/O requests to communicate with the USB device.
-   I/O Manager—a Windows component that creates I/O request packets (IRPs) to represent the received application requests, and forwards them to the top of the kernel-mode device stack for the target device.
-   Reflector—a Microsoft-provided kernel-mode driver installed at the top of the kernel-mode device stack (WUDFRd.sys). The reflector redirects IRPs received from the I/O manager to the client driver host process. Upon receiving the request, the framework and the client driver handle the request.
-   Host process —the process in which the user-mode driver runs (Wudfhost.exe). It also hosts the framework and the I/O dispatcher.
-   Client driver—the user-mode function driver for the USB device.
-   UMDF—the framework module that handles most interactions with Windows on the behalf of the client driver. It exposes the user-mode device driver interfaces (DDIs) that the client driver can use to perform common driver tasks.
-   Dispatcher—mechanism that runs in the host process; determines how to forward a request to the kernel mode after it has been processed by user-mode drivers and has reached the bottom of the user-mode stack. In the illustration, the dispatcher forwards the request to the user-mode DLL, Winusb.dll.
-   Winusb.dll—a Microsoft-provided user-mode DLL that exposes [WinUSB Functions](https://msdn.microsoft.com/library/windows/hardware/ff540046#winusb) that simplify the communication process between the client driver and WinUSB (Winusb.sys, loaded in kernel mode).
-   Winusb.sys—a Microsoft-provided driver that is required by all UMDF client drivers for USB devices. The driver must be installed below the reflector and acts as the gateway to the USB driver stack in the kernel-mode. For more information, see [WinUSB](winusb.md).
-   USB driver stack—a set of drivers, provided by Microsoft, that handle protocol-level communication with the USB device. For more information, see [USB host-side drivers in Windows](usb-3-0-driver-stack-architecture.md).

Whenever an application makes a request for the USB driver stack, the Windows I/O manager sends the request to the reflector, which directs it to client driver in user mode. The client driver handles the request by calling specific UMDF methods, which internally call [WinUSB Functions](https://msdn.microsoft.com/library/windows/hardware/ff540046#winusb) to send the request to WinUSB. Upon receiving the request, WinUSB either processes the request or forwards it to the USB driver stack.

## Related topics
[Understanding the UMDF template code for USB client driver](understanding-the-umdf-template-code-for-usb.md)  
[How to enable USB selective suspend and system wake in the UMDF driver for a USB device](http://go.microsoft.com/fwlink/p/?linkid=617587)  
[Getting started with USB client driver development](getting-started-with-usb-client-driver-development.md)  



