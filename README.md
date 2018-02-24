 This document describes how to prepare your development environment to run the Microsoft Azure IoT SDKs on KIWI MINI platform ,and communication with iot cloud and control BLE devices.

# 1. Hardware platform preparation.

- KIWI MINI gateway device 1*PCS, as shown below :

     ![](https://i.imgur.com/rurn84O.png)
  
- HMT BLE sensor 1*PCS , as shown below :

    ![](https://i.imgur.com/8qLktgi.png)
   
# 2. Build development environment and Compile SDKs.
  
- Install necessary development tools on your windows PC.
  
    Ensure that the desired SecureCRT and WinSCP are installed on your windows PC. if not, you can click the follows link web page to download and install.

    - [SecureCRT](https://www.vandyke.com/download/securecrt/download.html)

        This tool contains telnet, SSH, serial etc. and the developer can use those features in development process. Of course ,the developer can use other tools which had supported those features too.
         
    
    - [WinSCP](https://winscp.net/eng/download.php)
    
       We can use this tool to access gateway system, and dump out some files to analyze issues or copy app image to system to run in development stage.
 
- Install some dependent libs on your linux compiling environment.

        apt-get update
		apt-get upgrade
        apt-get install -y git cmake build-essential curl libcurl4-openssl-dev libssl-dev uuid-dev
	
- How to get toolchain for compiling ?

    There are two ways to get the toolchain .One is download toolchain from [openwrt.org](http://archive.openwrt.org/barrier_breaker/14.07/ar71xx/generic). and select "OpenWrt-Toolchain-ar71xx-for-mips_34kc-gcc-4.8-linaro_uClibc-0.9.33.2.tar.bz2" to download.

    ![](https://i.imgur.com/o1KQF5F.png)

    Because the toolchain contains basic libraries only,but in compiling process ,sdk depends some shared libraries which toolchain does't have ,as curl,openssl,etc. so the developoer need download those libraries source code from open source web site one by one by yourself, and compile and copy libraries and header files to toolchain directory.

    The second way is download openwrt source code and compile out the toolchain .if sdk depends some shared libraries ,the developer can configure libraries with "make menuconfig" only , when compiling , sdk will download libraries from net automatically.
    So i recommend the second way to get toolchain.
    
     -  Download openwrt source code.
   
		``` 
		      git clone  https://github.com/lixiantai/barrier_breaker.git
		``` 
     - Configure and Compile source code.
		

		        cd barrier_breaker
		        make package/symlinks
		        make defconfig
		        make menuconfig
		        make V=s 
		  The developer can reference the packages configuration(make menuconfig).the file is  "configuration\example\.config". and the developer can put this file to "<OPENWRT_ROOT_DIR>/\barrier_breaker" to see the selected packages.
        

     - Known compile issue:
           
          ![](https://i.imgur.com/Ij4LAn8.png)

         The reason is that the MAC80211 WIFI drivers comppiled in defult configuration ,but developer needn't to slect and compile this type of WIFI. So cancel this WIFI drivers selection in "make menuconfig".
  
    
           After comipiled successfully ,the developer can find the toolchain in this folder:

          ![](https://i.imgur.com/bCcSPDs.png)

          ```
          cd <OPENWRT_ROOT_DIR>/barrier_breaker/bin/ar71xx
          ```
          ```
 		  tar -jxvf OpenWrt-Toolchain-ar71xx-for-mips_34kc-gcc-4.8-linaro_uClibc-0.9.33.2.tar.bz2
		  ```

		  ```
          cp -rf ../../staging_dir/target-mips_34kc_uClibc-0.9.33.2/usr/* OpenWrt-Toolchain-ar71xx-for-mips_34kc-gcc-4.8-linaro_uClibc-0.9.33.2/toolchain-mips_34kc_gcc-4.8-linaro_uClibc-0.9.33.2/usr/
          ```

- Download Azure iot SDKs.

     You know Azure iot SDK had implemented by many languages ,as C,Python,C#,JAVA etc. And for  more details ,the developer can access to [Mircrosoft Azure iot SDK](https://github.com/Azure) web site . But in this document,we foucs on Python and C Azure SDK only.
 
     - Download Azure iot SDK for Python.

           ```
           git clone --recursive https://github.com/Azure/azure-iot-sdk-python.git
           ```
     - Download Azure iot SDK for C.

		   ```
           git clone --recursive https://github.com/Azure/azure-iot-sdk-c.git
           ```

- Compile Azure iot SDKs.
          
      - Compile Azure iot SDK for Python.
         
        The developer can reference the example of "Example of Cross Compiling the Azure IoT SDK for Python.md" in SDK. 
        
        - Compile Boostlinux module:
        
	              cd <BOOST_ROOT_DIR>
				  wget https://dl.bintray.com/boostorg/release/1.64.0/source/boost_1_64_0.tar.gz
	              tar -xzf boost_1_64_0.tar.gz
	              cd <BOOST_ROOT_DIR>/boost_1_64_0
	              mkdir boostmlinux-1-64-0
	              mkdir boost-build
	              ./bootstrap.sh --with-libraries=python –-prefix=<BOOST_ROOT_DIR>/boost_1_64_0/boostmlinux-1-64-0
	              vi project-config.jam ###Open this file,and do configuration and save, please see example: "configuration\example\project-config.jam".
	              ./b2 toolset=gcc-mips --build-dir=<BOOST_ROOT_DIR>/boost_1_64_0/boost-build link=static install
        
        - Compile Azure iot sdk for Python:


	              cd <AZURE_PYTHON_ROOT_DIR>/azure-iot-sdk-python
	              mkdir  kiwi_out
	              cd <AZURE_PYTHON_ROOT_DIR>/azure-iot-sdk-python/build_all/linux
	              touch toolchain-kiwimini.cmake
	              vi toolchain-kiwimini.cmake ###Open this file,and do configuration and save, please see example: "configuration\example\toolchain-kiwimini.cmake".
	              #edit build.sh and modify the line which will call "c/build_all/linux/build.sh",for example:
	              ./c/build_all/linux/build.sh --build-python $PYTHON_VERSION $* --provisioning $USE_TPM_SIM --toolchain-file "<AZURE_PYTHON_ROOT_DIR>/azure-iot-sdk-python/build_all/linux/toolchain-kiwimini.cmake" --install-path-prefix "<AZURE_PYTHON_ROOT_DIR>/azure-iot-sdk-python/kiwiout"
	              ./build.sh ###compile azure python sdk

		- Known compile issues:
     
            Issue1: When compiling , Maybe show error log of "relocation R_MIPS16_26 against `PyErr_Occurred' can not be used when making a shared object; recompile with -fPI",so the developer needs to recompile Python lib with "-fPIC". For example: edit "<OPENWRT_ROOT_DIR>/feeds/oldpackages/lang/python/Makefile",add "-fPIC" paramter.
              
            Issue2:When compiling , Maybe show error log of "libboost_python.a: could not read symbols:Bad Value", the reason is same as Issue1, so should modify file of "<BOOST_ROOT_DIR>/boost_1_64_0/tools/build/src/tools/gcc.jam",as:
     ![](https://i.imgur.com/vp2VHmO.png)


   - Compile Azure iot SDK for C
   
           To be continued.???????????????????
	   
	mkdir cmake
	cd  cmake
	cmake .. -DCMAKE_TOOLCHAIN_FILE=kiwimini_openwrt.cmake
	cmake --build .
	cd cmake/iothub_client/samples/iothub_client_sample_mqtt
	make

# 3. Run samples on KIWI MINI.

   Step1: power on the kiwi mini gateway device, the developer needs to check status mode which is AP or STA. If device mode is not STA mode ,the developer shuold configure network first.
  
-     If your device is AP mode ,the gateway can broadcast own address automatically, and the developer can see the WIFI SSID (The format is "Tonly_xxx")on PC,  and connect this point ,for example:
    
   ![](https://i.imgur.com/iJ0kB1T.png)
   
     and then use "192.168.88.1" to login gateway system by telenet in SecureCRT. as:

   ![](https://i.imgur.com/OJftNma.png) 
   
     1)Set password for root account:

   ![](https://i.imgur.com/THDnzpp.png)

     2)Configure network and ensure the gateway device can access internet by connect the upside AP.
       For example:
       
   ![](https://i.imgur.com/hkQMjlw.png)

      At last, this gateway is in STA mode , the upside AP will allocate the IP addr to the gateway.

-     If your device is STA mode ,you can use IP which had been allocated by upside AP and password to login gateway by SSH in SecureCRT. 
      If you forget the password , you can always press key 10 seconds on gateway, and the system will recovery to default configuration and AP mode,and then you need reconfigure network again for STA mode.

  Step2: Put compiled files into the gateway system that are running and dependent on the system.


   - For example of Python:
   
     ![](https://i.imgur.com/r62HIUD.png)
   - For example of C:
     
     ![](https://i.imgur.com/jbjbXxH.png)

  Step3:Get connected string from Azure iot hub cloud. The developer shold follow instructions in [Azure iot hub Cloud](https://portal.azure.com/#resource/subscriptions/81f3357e-37d1-4ecf-85b8-e9c4d6cc5347/resourceGroups/group2/providers/Microsoft.Devices/IotHubs/myIotHub-2018/Overview) website to creat account and regist devices.
  Step4:run samples.    

   - For example of Python:
    ![](https://i.imgur.com/iAf4RDv.png)
    Known issues :
                 ![](https://i.imgur.com/jTNuLQR.png)
                 Because of the Python Version is V2.7.3 ,the developer should ensure the Python version in PC is same as gateway system.Oherwise this issue will be presented.
   - For example of C:
     
     To be continue. ????????????????????????????         
