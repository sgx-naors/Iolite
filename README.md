Installing Graphene on Ubuntu Server 17.10.1
--------------------------------------------

In order to compile the SDK, PSW and part of the Graphene-SGX library, install GCC and G++:
* sudo apt install gcc-7 g++-7 

GCC and G++ 4.8 is needed later on for compiling LibOS, specifically for glibc-2.19, install them as well:
* sudo apt install gcc-4.8 g++-4.8
    
Download the latest Graphene from GitHub:
* git clone https://github.com/oscarlab/graphene.git

Graphene-SGX requires Intel-SGX driver version 1.9. Download and install it:
* git clone -b sgx_driver_1.9 https://github.com/intel/linux-sgx-driver.git
* sudo apt install make
* sudo make DEBUG=1 (DEBUG is optional)
* sudo openssl req -new -x509 -newkey rsa:4096 -keyout /usr/src/linux-headers-$(uname -r)/certs/signing_key.pem -nodes -days 36500 -subj "/CN=MyDevMachine/" -out /usr/src/linux-headers-$(uname -r)/certs/signing_key.x509
* sudo make install
* sudo ln -s /lib/modules/$(uname -r)/kernel/drivers/intel/sgx/isgx.ko /lib/modules/$(uname -r)
* sudo depmod -a
* sudo modprobe sgx

Download and install Intel's Capability Licensing Service (iclsClient), it is required for the PSW:
(ICLS is the activation site at Intel where client installations are tracked)
* You can download the debian package from <a href="https://github.com/sgx-naors/Iolite/raw/master/iclsclient_1.45.449.12-2_amd64.deb">here</a>.
* sudo dpkg -i iclsclient_1.45.449.12-2_all.deb - It will be placed in /opt/Intel
    
Generate the Enclave signing key:
* sudo mkdir /opt/intel /opt/intel/sgxkey
* sudo openssl genrsa -3 -out /opt/intel/sgxkey/enclave-key.pem 3072

Graphene-SGX requires Intel-SGX SDK and Intel-SGX PSW version 1.9. Download and install them:
* git clone -b sgx_1.9 https://github.com/intel/linux-sgx.git
* sudo apt install libssl-dev libcurl4-openssl-dev protobuf-compiler libprotobuf-dev build-essential ocaml python ocamlbuild automake libtool
* cd linux-sgx; ./download_prebuilt.sh
* Download the patch needed for the compilation of the SDK and PSW. You can find it in <a href="https://github.com/sgx-naors/Iolite/blob/master/sgx_sdk_psw.patch">here</a>.
* cp sgx_sdk_psw.patch linux-sgx
* cd linux-sgx
* git apply sgx_sdk_psw.patch --stat
* Compile the SDK and PSW:
    * make DEBUG=1
* Install the SDK:
    * make sdk_install_pkg DEBUG=1 (DEBUG is optional)
    * sudo ./linux/installer/bin/sgx_linux_x64_sdk_1.9.100.39124.bin
        > Install it into /opt/intel
* Install the PSW:
    * make psw_install_pkg DEBUG=1 (DEBUG is optional)
    * sudo ./linux/installer/bin/sgx_linux_x64_psw_1.9.100.39124.bin
* Finalize the installation:    
    * sudo sh -c 'echo export SGX_ENCLAVE_KEY=/opt/intel/sgxkey/enclave-key.pem >> /opt/intel/sgxsdk/environment'
    * sh -c 'echo source /opt/intel/sgxsdk/environment >> ~/.bashrc'
    * source ~/.bashrc
    * sudo chown \<uid\>:\<gid\> /opt/intel -R

Install Graphene:  
* cd graphene; git submodule update --init
* Install Graphene's SGX driver:
    * cd Pal/src/host/Linux-SGX/sgx-driver
    * make DEBUG=1
        > Enter the SGX driver's directory compilation directory.
    * sudo cp graphene-sgx.ko /lib/modules/`uname -r`/kernel/drivers/intel/sgx/
    * sudo ln -s /lib/modules/$(uname -r)/kernel/drivers/intel/sgx/graphene-sgx.ko /lib/modules/$(uname -r)
    * sudo depmod -a
    * sudo modprobe graphene-sgx
    * sudo crontab -e
        > Select vim (:>)
    * sudo crontab -l | { cat; echo "@reboot /usr/bin/sudo /sbin/insmod /lib/modules/`/bin/uname -r`/kernel/drivers/intel/sgx/graphene-sgx.ko"; } | sudo crontab -
* Compile PAL
    * cd Pal/src
    * make SGX=1
    * cd ../..
* Compile LibOS    
    * sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-4.8 100 --slave /usr/bin/g++ g++ /usr/bin/g++-4.8
    * cd LibOS
    * make SGX=1 SGX_SIGNER_KEY=/opt/intel/sgxkey/enclave-key.pem DEBUG=1 (DEBUG is optional)
        > Grab a cup of coffee/tea
        
Enjoy
