---
layout: default
title: DSBGpioTutorial
permalink: /en-US/win10/samples/AlljoynDSB_GpioTutorial.htm
lang: en-US
---

## Alljoyn DSB Gpio Sample

This tutorial shows how a GPIO Device can be exposed to the AllJoyn Bus using the AllJoyn Device System Bridge.  

### Prerequisites

1. Alljoyn Explorer

* [AllJoynExplorer_1.0.0.0.zip](https://github.com/ms-iot/samples/blob/develop/AllJoyn/AllJoynExplorer/AllJoynExplorer_1.0.0.0.zip?raw=true){:target="_blank"} - This zip contains the AllJoyn Explorer AppX bundle.
* [AllJoyn_Explorer_Setup_Guide_v1.0.pdf](https://github.com/ms-iot/samples/blob/develop/AllJoyn/AllJoynExplorer/AllJoyn_Explorer_Setup_Guide_v1.0.pdf?raw=true){:target="_blank"} - This pdf contains the documentation for how to deploy the AllJoyn Explorer.
* [AllJoyn_Explorer_User_Guide_v1.0.pdf](https://github.com/ms-iot/samples/blob/develop/AllJoyn/AllJoynExplorer/AllJoyn_Explorer_User_Guide_v1.0.pdf?raw=true){:target="_blank"} - This pdf contains the documentation for how to use the AllJoyn Explorer. 

### Step 1: Hardware Setup  
The sample uses a Raspberry Pi 2 that one of its GPIO pins is connected to a photo resistor as shown in the schematic below. If another device is sues the pin number in the code has to be changed to match the HW setup.

### Step 2: Download and Install the AllJoyn Device System Bridge Template  

The AllJoyn Device System Bridge Template is a Visual Studio extension that enables developers to create an AllJoyn Device System Bridge App project.

1. Go to [Products and Extensions for Visual Studio](https://visualstudiogallery.msdn.microsoft.com/) and search for “AllJoyn DSB” and download the template. 
2. After the download, double-click on the DeviceSystemBridgeTemplate.vsix file to install the extension. 

### Step 4: Expose a GPIO Pin to the AllJoyn Bus  

Open the Adapter.cpp file in the AdapterLib project. The Gpio device information and pin properties are defined here. 

    using namespace Windows::Devices::Gpio; 
     
    // GPIO Device 
    String^ deviceName  	= "Custom_GPIO_Device"; 
    String^ vendorName  	= "Custom_Vendor"; 
    String^ modelName  	= "Custom_Model"; 
    String^ version 	 	= "1.0.0.0"; 
    String^ serialNumber  	= "1111111111111"; 
    String^ description  	= "A Custom GPIO Device"; 
     
    // GPIO Device Pin-5 Property const int PIN_NUMBER  	= 5; 
    String^ pinName 	 	= "Pin-5"; 
     
    // Pin-5 Property Attribute  
    String^ pinValueName  	= "PinValue"; 
    int  pinValueData  	= -1; 
     
    GpioController^  	 	controller; 
    GpioPin^  	 	 	pin; 


In order to expose the GPIO Device to the AllJoyn Bus, a corresponding Bridge Device (IAdapterDevice) instance has been created in the Adapter Constructor: 
    
    Adapter::Adapter() 
    {        
      controller = GpioController::GetDefault();        
      pin = controller->OpenPin(PIN_NUMBER);  	// Open GPIO 5                      
      pin->SetDriveMode(GpioPinDriveMode::Input); 	// Set the IO direction as output  
 
      this->vendor = L"Custom Vendor";
      this->adapterName = L"Custom Adapter";      
      this->version = L"1.0.0.1";   
    } 
 
 
Now Observe the Initialize() function.

    uint32 
    Adapter::Initialize() 
    { 
       // GPIO Device Descriptor: Static data for our device 
       DEVICE_DESCRIPTOR gpioDeviceDesc;        
       gpioDeviceDesc.Name  = deviceName;        
       gpioDeviceDesc.VendorName = vendorName;        
       gpioDeviceDesc.Model  = modelName;        
       gpioDeviceDesc.Version = version;        
       gpioDeviceDesc.SerialNumer = serialNumber;        
       gpioDeviceDesc.Description = description;  
       
       // Define GPIO Pin-5 as device property. Device contains properties 
       AdapterProperty^ gpioPin_Property = ref new AdapterProperty(pinName);  
       // Define and set GPIO Pin-5 value. Device contains properties that have one or more values. 
       pinValueData = static_cast<int>(pin->Read()); 
       AdapterValue^ valueAttr_Value = ref new AdapterValue(pinValueName, pinValueData);        
       gpioPin_Property += valueAttr_Value; 
 
       // Finally, put it all into a new device  
       AdapterDevice^ gpioDevice = ref new AdapterDevice(&gpioDeviceDesc);        
       gpioDevice->AddProperty(gpioPin_Property);        
       devices.push_back(std::move(gpioDevice)); 
 
       return ERROR_SUCCESS; 
    } 
    
       _Use_decl_annotations_     uint32 
      Adapter::EnumDevices( 
          ENUM_DEVICES_OPTIONS Options, 
          IAdapterDeviceVector^* DeviceListPtr, 
          IAdapterIoRequest^* RequestPtr 
          ) 
      { 
          UNREFERENCED_PARAMETER(Options); 
          UNREFERENCED_PARAMETER(RequestPtr);  
          *DeviceListPtr = ref new BridgeRT:: AdapterDeviceVector(this->devices); 
   
          return ERROR_SUCCESS; 
    } 


## EXTRA CREDIT: Signaling when the GPIO pin value changes 
Suppose the applications on the AllJoyn bus do not want to poll the value of the GPIO pin but just to be notified when the GPIO pin value changes. For that, we need to add support for signals in our adapter. The contents below contain all the pieces needed to make the GPIO Device Sample notify the AllJoyn Consumer Applications. 

### Additions to Adapter.h: 

    private: 
        // GPIO pin value change event handler         void pinValueChangedEventHandler( 
        _In_ Windows::Devices::Gpio::GpioPin^ gpioPin,  
        _In_ Windows::Devices::Gpio::GpioPinValueChangedEventArgs^ eventArgs); 
        
### Additions to Adapter.cpp: 

    uint32 
    Adapter::Initialize() 
    { 
        // GPIO Device Descriptor: Static data for our device 
        DEVICE_DESCRIPTOR gpioDeviceDesc;          
        gpioDeviceDesc.Name  = deviceName;         
        gpioDeviceDesc.VendorName = vendorName;         
        gpioDeviceDesc.Model = modelName;         
        gpioDeviceDesc.Version = version;         
        gpioDeviceDesc.SerialNumer = serialNumber;         
        gpioDeviceDesc.Description = description;  
        
        // Define GPIO Pin-5 as device property. Device contains properties 
        AdapterProperty^ gpioPin_Property = ref new AdapterProperty(pinName);  
        // Define the GPIO Pin-5 value. Device has properties with attributes.         
        pinValueData = static_cast<int>(pin->Read()); 
        AdapterValue^ valueAttr_Value = ref new AdapterValue(pinValueName, pinValueData);         
        gpioPin_Property += valueAttr_Value; 
 
        // Create Change_Of_Value signal for the Value Attribute 
        AdapterSignal^ signal = ref new AdapterSignal(CHANGE_OF_VALUE_SIGNAL);         
        signal += ref new AdapterValue(COV__PROPERTY_HANDLE, gpioPin_Property);         
        signal += ref new AdapterValue(COV__ATTRIBUTE_HANDLE, valueAttr_Value); 
 
 
        // Finally, put it all into a new device  
        AdapterDevice^ gpioDevice = ref new AdapterDevice(&gpioDeviceDesc);         
        gpioDevice->AddProperty(gpioPin_Property);         
        gpioDevice->AddSignal(signal); 
        devices.push_back(std::move(gpioDevice)); 
 
        // Pin Value Changed Event Handler 
        pin->ValueChanged += ref new Windows::Foundation::TypedEventHandler<GpioPin ^, GpioPinValueChangedEventArgs ^>(this, &Adapter::pinValueChangedEventHandler); 
 
        return ERROR_SUCCESS;     
      } 
 
    _Use_decl_annotations_ 
    uint32 
    Adapter::RegisterSignalListener( 
        IAdapterSignal^ Signal, 
        IAdapterSignalListener^ Listener, 
        Object^ ListenerContext 
        ) 
    { 
        AutoLock sync(&lock, true); 
 
        signalListeners.insert({ Signal->GetHashCode(), SIGNAL_LISTENER_ENTRY(Signal, Listener, ListenerContext) }); 
 
        return ERROR_SUCCESS;     
    } 
