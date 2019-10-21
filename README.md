# Instructions on deliverables
Author: **Harri Wijaya** ([email](mailto:harri.wijaya@gmail.com)) ([website](https://harriwijaya.github.io/)).

## Installation
Since somehow the [PC Simulator](https://bitbucket.org/suunto/movesense-docs/wiki/EmbeddedSoftware/PC%20Simulator) --> [Build How-to](https://bitbucket.org/suunto/movesense-docs/wiki/EmbeddedSoftware/Build%20how-to.md) does not work as it is, I use the following version of the tools instead. I might attribute this to latest release of Movesense-device-lib.

| Tool's name  | Version | Comment |
| --- | --- | --- |
| GNU Toolchain for ARM Embedded  | 2017q4  | Version 5-2016-q3 (as specified in PC Simulator page) produces LTO mismatch. |
| Ninja  | 1.9.0  |  (_I didn't verified other version._) |
| CMake | 3.15.4 | Version 3.8.2 does not able to find VC++2017. (in PC with VC++2019 is also installed) |
| Visual Studio Redistributable 2015 | 14.0 | (_I didn't verified other version._) |
| nrfutil | 5.2.0 | Note that v6.0.0a0 remove the support for Python 2. |
| Python 2.7 | 2.7 | (_As instructed_) |

And, I use **Visual Studio Express 2017 for Windows Desktop**.

## Fetch Movesense source code repository
Provided you have git installed, issue the following command:
```
git clone --branch master https://bitbucket.org/suunto/movesense-device-lib.git
```
I develop this exercise on top of commit `15c48f9`, do `git checkout 15c48f9` if needed.

## Configure and build for simulation environment
1. Copy the deliverable source code, i.e. folder `harriwijaya_app`, into `\movesense-device-lib\samples\`.
2. At the location of `\movesense-device-lib\`, create a build folder and go into it, example:
```
mkdir mySimuBuild
cd mySimuBuild
```
3. Build the application for **simulator** by issuing the following:
```
cmake -G "Visual Studio 15 2017" -DMOVESENSE_CORE_LIBRARY=../MovesenseCoreLib/ ..\samples\harriwijaya_app
```
4. Open the generated `Project.sln` with Visual Studio 2017. Select `Movesense` project, and then **Rebuild**.
5. Copy the provided `accelerometer.csv` file into build folder.
   - This will create the acceleration value as:
     - from 0 sec to 2 sec: x,y,z = 0,0,0
     - from 2 sec to 4 sec: x,y,z = 2,2,2
     - from 4 sec to 6 sec: x,y,z = 4,4,4
     - from 6 sec to 8 sec: x,y,z = 6,6,6
     - then repeat again (in the cycle of 8 sec).
6. Once rebuilt, do **Debug** → **Start new instance**.
7. Test the new API (**/Exercise/Sumvector/Harri**):
   - From a location where **wbcmd.exe** is accessible, e.g. `movesense-device-lib\tools`, issue the following command:
   ```
   wbcmd.exe --port TCP127.0.0.1:7044 --op subscribe --path /Exercise/Sumvector/Harri
   ```
   - Notice the results:
     - The 1 seconds averaged value of acceleration is displayed.
     - The LED is shortly blinking (indicated by red rectangle in simulator window) one time for every new averaged acceleration value.
8. The method to calculate the average is as the following:
   - Within window of 1 second, sum each components (x, y, z) of the acceleration 3D vector.
   - After 1 second has elapsed, divide each sum values by the number of samples, i.e. get the averaged value.
   - Calculate the averaged acceleration by: SQRT(meansumX^2 + meansumY^2 + meansumZ^2).
   - Other calculation method is possible, e.g. moving average instead of fixed window.

## Configure and build for target environment
1. At the location of `\movesense-device-lib\`, create a build folder and go into it, example:
```
mkdir myTargetBuild
cd myTargetBuild
```
2. Build the application for **target** (release version) by issuing the following 2 commands:
```
cmake -G Ninja -DMOVESENSE_CORE_LIBRARY=../MovesenseCoreLib/ -DCMAKE_TOOLCHAIN_FILE=../MovesenseCoreLib/toolchain/gcc-nrf52.cmake -DCMAKE_BUILD_TYPE=Release ..\samples\harriwijaya_app
ninja
```

## Over the air (OTA) installation package
Continuing from the last section (i.e. target build), issue the following command to generate an OTA firmware update package:
```
ninja dfupkg
```
Notice that _movesense_dfu.zip_ has been created.

(**End of instruction**)
