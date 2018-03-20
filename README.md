Installing Graphene on Ubuntu 17.10
-----------------------------------

* sudo apt install gcc-7 g++-7

* sudo apt install gcc-4.8 g++-4.8
    
* git clone https://github.com/oscarlab/graphene.git

* git clone -b sgx_driver_1.9 https://github.com/intel/linux-sgx-driver.git
    * sudo apt install make
    * sudo openssl req -new -x509 -newkey rsa:4096 -keyout /usr/src/linux-headers-$(uname -r)/certs/signing_key.pem -nodes -days 36500 -subj "/CN=MyDevMachine/" -out /usr/src/linux-headers-$(uname -r)/certs/signing_key.x509
    * sudo make install
    * sudo ln -s /lib/modules/$(uname -r)/kernel/drivers/intel/sgx/isgx.ko /lib/modules/$(uname -r)
    * sudo depmod -a
    * sudo modprobe sgx
    
* git clone -b sgx_1.9 https://github.com/intel/linux-sgx.git
    * sudo apt install libssl-dev libcurl4-openssl-dev protobuf-compiler libprotobuf-dev build-essential ocaml python ocamlbuild automake libtool
    * cd linux-sgx; ./download_prebuilt.sh
    * cp sgx_sdk_psw.patch linux-sgx
    * cd linux-sgx
    * git apply sgx_sdk_psw.patch --stat
    * make sdk_install_pkg DEBUG=1
    * sudo ./linux/installer/bin/sgx_linux_x64_sdk_1.9.100.39124.bin
        > Choose no
        > Choose /opt/intel
    
* Install Intel's Capability Licensing Service (iclsClient):
    (ICLS is the activation site at Intel where client installations are tracked)
        > sudo dpkg -i iclsclient_1.45.449.12-2_all.deb
    * make psw_install_pkg DEBUG=1
    * sudo ./linux/installer/bin/sgx_linux_x64_psw_1.9.100.39124.bin
    * sudo sh -c 'echo export SGX_ENCLAVE_KEY=/opt/intel/sgxkey/enclave-key.pem >> /opt/intel/sgxsdk/environment'
    * sh -c 'echo source /opt/intel/sgxsdk/environment >> ~/.bashrc'
    * sudo chown <uid>:<gid> /opt/intel -R

* Install Graphene:  
    * cd graphene; git submodule update --init
    * cd Pal/src/host/Linux-SGX/sgx-driver
    * make DEBUG=1
        > Input the sgx driver directory you already installed
    * sudo cp graphene-sgx.ko /lib/modules/`uname -r`/kernel/drivers/intel/sgx/
    * sudo ln -s /lib/modules/$(uname -r)/kernel/drivers/intel/sgx/graphene-sgx.ko /lib/modules/$(uname -r)
    * sudo depmod -a
    * sudo modprobe graphene-sgx
    * sudo crontab -e
        > Select vim (:>)
    * sudo crontab -l | { cat; echo "@reboot /usr/bin/sudo /sbin/insmod /lib/modules/`/bin/uname -r`/kernel/drivers/intel/sgx/graphene-sgx.ko"; } | sudo crontab -
    * sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-4.8 100 --slave /usr/bin/g++ g++ /usr/bin/g++-4.8
    * cd Pal/src
    * make SGX=1
    * cd ../..
    * cd LibOS
    * make SGX=1 DEBUG=1 SGX_SIGNER_KEY=/opt/intel/sgxkey/enclave-key.pem
        > Grab a cup of coffee/tea
        
Enjoy
