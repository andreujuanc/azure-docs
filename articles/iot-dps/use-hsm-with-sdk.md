---
title: Azure How to - How to use different Hardware Security Module with the Device Provisioning Service Client SDK in Azure 
description: Azure How to - How to use different Hardware Security Module with the Device Provisioning Service Client SDK in Azure
services: iot-dps 
keywords: 
author: yzhong94
ms.author: yizhon
ms.date: 03/28/2018
ms.topic: hero-article
ms.service: iot-dps

documentationcenter: ''
manager: 
ms.devlang: na
ms.custom: mvc
---

# How to use different Hardware Security Modules with Device Provisioning Service Client SDK for C

This article shows you how to use different [Hardware Security Modules (HSM)](https://azure.microsoft.com/blog/azure-iot-supports-new-security-hardware-to-strengthen-iot-security/) with the Device Provisioning Service Client SDK for C. You can use either a physical device or a simulator. The provisioning service supports authentication for two types of attestation mechanisms: X**.**509 and Trusted Platform Module (TPM).

## Prerequisites

Prepare your development environment according to the section titled "Prepare the development environment" in the [Create and provision simulated device](./quick-create-simulated-device.md) guide.

### Choose a Hardware Security Module

As a device manufacturer, you first need to choose Hardware Security Modules (or HSMs) that are based on one of the supported types. Currently, the [Device Provisioning Service client SDK for C](https://github.com/Azure/azure-iot-sdk-c/tree/master/provisioning_client) provides support for the following HSMs: 

- [Trusted Platform Module (TPM)](https://en.wikipedia.org/wiki/Trusted_Platform_Module): TPM is an established standard for most Windows-based device platforms, as well as a few Linux/Ubuntu based devices. As a device manufacturer, you may choose this HSM if you have either of these OSes running on your devices, and you are looking for an established standard for HSMs. With TPM chips, you can only enroll each device individually to the Device Provisioning Service. For development purposes, you can use the TPM simulator on your Windows or Linux development machine.

- [X.509](https://cryptography.io/en/latest/x509/): X.509 based HSMs are relatively newer chips. Work is also progressing within Microsoft, on RIoT or DICE chips, which implement the X.509 certificates. With X.509 chips, you can do bulk device enrollment in the portal. It also supports certain non-Windows OSes like embedOS. For development purpose, the Device Provisioning Service client SDK supports an X.509 device simulator. 

For more information, see [IoT Hub Device Provisioning Service security concepts](concepts-security.md). 

## Enable authentication for supported HSMs

Authentication mode (X**.**509 or TPM) must be enabled for the physical device or simulator before they can be enrolled in the Azure portal. First, navigate to the root folder for azure-iot-sdk-c. Then run the specified command, depending on the authentication mode you choose:

### Use X**.**509 with simulator

The provisioning service ships with a Device Identity Composition Engine (DICE) emulator that generates an X**.**509 certificate for authenticating the device. To enable X**.**509 authentication, run the following command: 

```
cmake -Ddps_auth_type=x509 ..
```

Information regarding hardware with DICE can be found [here](https://azure.microsoft.com/blog/azure-iot-supports-new-security-hardware-to-strengthen-iot-security/).

### Use X**.**509 with hardware

The provisioning service can be used with X**.**509 on other hardware. An interface between hardware and the SDK is needed to establish connection. Talk to your HSM manufacturer for information on the interface.

### Use TPM

The provisioning service can connect to Windows and Linux hardware TPM chips with SAS Token. To enable TPM authentication, run the following command:

```
cmake -Ddps_auth_type=tpm ..
```

### Use TPM with simulator

If you don't have a device with TPM chips, you can use a simulator for development purpose on Windows OS. To enable TPM authentication and run the TPM simulator, run the following command:

```
cmake -Ddps_auth_type=tpm_simulator ..
```

## Build the SDK
Build the SDK prior to creating device enrollment.

### Linux
- To build the SDK in Linux:
    ```
    cd azure-iot-sdk-c
    mkdir cmake
    cd cmake
    cmake ..
    cmake --build .  # append '-- -j <n>' to run <n> jobs in parallel
    ```
- To build Debug binaries, add the corresponding CMake option to the project generation command, for example:
    ```
    cmake -DCMAKE_BUILD_TYPE=Debug ..
    ```

- There are many [CMake configuration options](https://cmake.org/cmake/help/v3.6/manual/cmake.1.html) available for building the SDK. For example, you can disable one of the available protocol stacks by adding an argument to the CMake project generation command:
    ```
    cmake -Duse_amqp=OFF ..
    ```
- You can also build and run unit test:
    ```
    cmake -Drun_unittests=ON ..
    cmake --build .
    ctest -C "debug" -V
    ```

### Windows
- To build the SDK in Windows, take the following steps to generate project files:
    - Open a "Developer Command Prompt for VS2015"
    - Run the following CMake commands from the root of the repository:
      ```
      cd azure-iot-sdk-c
      mkdir cmake
      cd cmake
      cmake -G "Visual Studio 14 2015" ..
      ```
    This command builds x86 libraries. To build for x64, modify the cmake generator argument: 
    ```
    cmake .. -G "Visual Studio 14 2015 Win64"
    ```

- If project generation completes successfully, you should see a Visual Studio solution file (.sln) under the `cmake` folder. To build the SDK:
    - Open **cmake\azure_iot_sdks.sln** in Visual Studio and build it, **OR**
    - Run the following command in the command prompt you used to generate the project files:
      ```
      cmake --build . -- /m /p:Configuration=Release
      ```
- To build Debug binaries, use the corresponding MSBuild argument: 
    ```
    cmake --build . -- /m /p:Configuration=Debug`
    ```
- There are many CMake configuration options available for building the SDK. For example, you can disable one of the available protocol stacks by adding an argument to the CMake project generation command:
    ```
    cmake -G "Visual Studio 14 2015" -Duse_amqp=OFF ..
    ```
- Also, you can build and run unit tests:
    ```
    cmake -G "Visual Studio 14 2015" -Drun_unittests=ON ..
    cmake --build . -- /m /p:Configuration=Debug
    ctest -C "debug" -V
    ```
  
### Libraries to include
- These libraries should be included in your SDK:
    - The provisioning service: dps_http_transport, dps_client, dps_security_client
    - IoTHub Security: iothub_security_client

## Create a device enrollment entry in Device Provisioning Services

### TPM
If you are using TPM, follow instructions in ["Create and provision a simulated device using IoT Hub Device Provisioning Service"](./quick-create-simulated-device.md) to create a device enrollment entry in your Device Provisioning Service and simulate first boot.

### X**.**509
1. To enroll a device in the provisioning service, you need note down the Endorsement Key and Registration ID for each device, which are displayed in the Provisioning Tool provided by Client SDK. Run the following command to print out the root CA certificate (for enrollment groups) and the signer certificate (for individual enrollment):
      ```
      ./azure-iot-sdk-c/dps_client/tools/x509_device_provision/x509_device_provision.exe
      ```
2. Sign in to the Azure portal, click on the **All resources** button on the left-hand menu and open your DPS service.
   - X**.**509 Individual Enrollment: On the provisioning service summary blade, select **Manage enrollments**. Select **Individual Enrollments** tab and click the **Add** button at the top. Select **X**.**509** as the identity attestation *Mechanism*, upload the signer certificate as required by the blade. Once complete, click the **Save** button. 
   - X**.**509 Group Enrollment: On the provisioning service  summary blade, select **Manage enrollments**. Select **Group Enrollments** tab and click the **Add** button at the top. Select **X**.**509** as the identity attestation *Mechanism*, enter a group name and certification name, upload the root CA certificate as required by the blade. Once complete, click the **Save** button. 

## Enable authentication for custom TPM and X.509 devices (optional)

> [!NOTE]
> This section is only applicable to devices that require support for a custom platform or HSM, that is not currently supported by the Device Provisioning Service Client SDK for C.

First you need to develop your custom HSM repository and library:

1. Develop a library to access your HSM. This project needs to produce a static library for the Device Provisioning SDK to consume.

2. Implement the functions defined in the following header file, in your library: 

    - For a custom TPM: implement the Custom HSM functions defined under [HSM TPM API](https://github.com/Azure/azure-iot-sdk-c/blob/master/provisioning_client/devdoc/using_custom_hsm.md#hsm-tpm-api).  
    - For a custom X.509: implement the Custom HSM functions defined under [HSM X509 API](https://github.com/Azure/azure-iot-sdk-c/blob/master/provisioning_client/devdoc/using_custom_hsm.md#hsm-x509-api). 

Once your library successfully builds on its own, you'll need to integrate it with the Device Provisioning Service Client SDK, by linking against your library. :

1. Supply the custom HSM GitHub repository, the library path and its name in the following `cmake` command:
    ```cmd/sh
    cmake -Duse_prov_client:BOOL=ON -Dhsm_custom_lib=<path_and_name_of_library> <PATH_TO_AZURE_IOT_SDK>
    ```
   
2. Open the Visual Studio solution file built by CMake (`\azure-iot-sdk-c\cmake\azure_iot_sdks.sln`), and build it. 

    - The build process compiles the SDK library.
    - The SDK attempts to link against the custom HSM defined in the `cmake` command.

3. Run the "prov_dev_client_ll_sample" sample app under "Provision_Samples" (under `\azure-iot-sdk-c\cmake\provisioning_client\samples\prov_dev_client_ll_sample`), to verify that your HSM is implemented correctly.

## Connecting to IoT Hub after provisioning

Once the device has been provisioned with the provisioning service, this API uses the HSM authentication mode to connect with IoT Hub: 
  ```
  IOTHUB_CLIENT_LL_HANDLE handle = IoTHubClient_LL_CreateFromDeviceAuth(iothub_uri, device_id, iothub_transport);
  ```

