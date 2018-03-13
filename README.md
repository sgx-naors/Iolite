# Project Iolite - Work In Progress

First thing, let's utilize linux-sgx:

1. Download and install Ubuntu Server 17.10.1 (Artful Aardvark)

2. Install the SGX Driver by fulfilling the following instructions:
  * git clone https://github.com/intel/linux-sgx-driver.git
  * Check if matching Kernel headers are installed: 
	    + dpkg-query -s linux-headers-$(uname -r)
      If not, run: 
	    + sudo apt-get install linux-headers-$(uname -r)
      
  * In order to build the driver perform the following :
	    + sudo apt install make gcc
	    + make
      
	* In order to install your freshly built driver perform the following:
    Latest support in the secure boot feature is a bit problematic.
    Either you disable it (didn't work for me) or create the kernel signing keys for the new isgx.ko module:
      +	sudo openssl req -new -x509 -newkey rsa:4096 -keyout /usr/src/linux-headers-$(uname -r)/certs/signing_key.pem -nodes -days 36500 -subj "/CN=MyDevMachine/" -out /usr/src/linux-headers-$(uname -r)/certs/signing_key.x509
	    +	sudo make install
			
3. Build Linux SGX's SDK and PSW
	* Install addition required tools to build the SDK and PSW:
		  + sudo apt install libssl-dev libcurl4-openssl-dev protobuf-compiler libprotobuf-dev build-essential ocaml ocamlbuild automake libtool
	* Download precompiled optimized IPP/string/math libraries:
	    + cd linux-sgx-master; ./download_prebuilt.sh
	* Fix a compilation warning which is considered as an error:
	    + sed -i 's/# additional warnings flags for C++/# additional warnings flags for C++\nCXXFLAGS += \-Wno\-error=unused\-parameter/' buildenv.mk
		
		
  * For building the SDK and PSW choose between the following options:
	    + make
      or
		  + make DEBUG=1
		The latter is for supporting debug information.
	  
  * For building the SDK Installer choose between the following options:
	    + make sdk_install_pkg
      + make sdk_install_pkg DEBUG=1
		The latter is for supporting debug information
		
  * For building the PSW Installer choose between the following options:
      + make psw_install_pkg
      + make psw_install_pkg DEBUG=1
    The latter is for supporting debug information
		
  * Install Intel's Capability Licensing Service (iclsClient):
    (ICLS is the activation site at Intel where client installations are tracked)
      + sudo apt install alien
      + cd /tmp; wget http://registrationcenter-download.intel.com/akdlm/irc_nas/11414/iclsClient-1.45.449.12-1.x86_64.rpm
      + sudo alien --scripts ./iclsClient-1.45.449.12-1.x86_64.rpm
      + sudo dpkg -i ./iclsclient_1.45.449.12-2_amd64.deb
		
  * Install JHI - Dynamic Application Loader (DAL) Host Interface 
    (A daemon and libraries which allow user space applications to install Java applets on DAL FW and communicate with them.)
      + git clone https://github.com/intel/dynamic-application-loader-host-interface.git
      + cd ./dynamic-application-loader-host-interface
      + cmake .
      + sudo make install
      + sudo systemctl enable jhi
		
   * Install the SDK:
      + cd linux/installer/bin/
      + sudo ./sgx_linux_sdk_x64_${version}.bin
    Choose target directory : /opt/intel
    It is now installed in /opt/intel/sgxsdk
			
  * Install the PSW:
    (Platform Software - Installs a full set of Intel Management Engine software components which include Intel Dynamic Application Loader Host Interface Service (DAL Host Interface Service))
      + cd linux/installer/bin/
      + sudo ./sgx_linux_sdk_x64_${version}.bin
    It is now installed in /opt/intel/sgxpsw
		
   * Give yourself permissions for both the SDK and PSW:
      + sudo chown ${uid}:${gid}  /opt/intel -R
	
  * aesmd Service - Should be approached only if you are behind a proxy server
    AESM : SGX Application Enclave Service Manager included in the PSW (Platform Software) Installation, AESM runs as a service (daemon) with aesmd permissions
    + Stop the aesmd service:
        > sudo service aesmd stop
    + Edit /etc/aesmd.conf to include your proxy setup
    + Start the aesmd service:
        > sudo service aesmd start
        
 * Insall Eclipse Mars - TBD
