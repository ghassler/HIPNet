
This CableLabs HIPnet software is based on code from the OpenWRT project and is offered under GPLv2 license terms.

Each code base must be placed in the proper place to properly build.



## How-to: Compiling Source Code 

The first step of adding or maintaining code to the code base is to pull down the current stable release of OpenWRT.  Please see the following link: https://openwrt.org/. The build used for our Buffalo WZR-HP-G300NH2 can be found at http://wiki.openwrt.org/toh/buffalo/wzr-hp-g300h with instrunctions on its installation. 

Please note that the unstable branch called trunk has given issues.  We recommend the stable branch for this reason.
 
The build system then needs to be set up to build working images and software packages.  The following link has an explanation of how to pull down all software packages and build them. http://wiki.openwrt.org/doc/howto/build 
 
We have found that the following target options should be selected when building an image.  Please start in the root directory of your build tree and run make menuconfig to bring up the build options.  
 
For building for an x86 virtual machine choose target x86.Â  For building for the routers please choose target Atheros AR7xxx/AR9xxx and Target Profile Buffalo WZR-HP-G300NH2.
 
Once these build options are selected the following packages must be checked for inclusion in the image building process.Â  (To do this hit space bar until the box next to the package is marked *):
Network --->
    IP Addresses and Names --->
	 <*> isc-dhcp-client-ipv6
         <*> isc-dhcp-server-ipv6
    Routing and Redirection --->
         <*> ip
 
After these are selected, continually hit < Exit > until you end up saving your configuration.  At this point an initial build will be kicked off by running make -j4.  If you have more than 4 cores to throw at the build please do; this can be a very long process.
 
The following instructions are for the method used to build the package for the currently selected build environment (either x86 or Atheros for the routers).  The build method is complicated and round about due to short combings of the provided SDK that does not allow for ipv6 code to be created.  
 
After the build is finished, please go to the directory package/feeds/packages/isc-dhcp/files/ and replace the files dhclient6.init and dhcpd6.init with the files provided. Then return to the root directory of the build system and run the make command again.
 
After the second build, which should have run much more quickly, we are now ready to move code into the build directory.  Enter the directory build_dir/linux-ar71xx_generic/packages/isc-dhcp and move the following files into place.  The files dhclient6.c and dhc6.c need to be moved in the the client/ directory.  The file dhcpd6.c needs to be moved into the server/ directory.  Once these files are in place return to the root directory and run the following commands to build down installable binaries for the images on the routers.  Alternatively, rerun the whole build for an image that includes our modified code.
 
make package/feeds/packages/isc-dhcp/compile
make package/feeds/packages/isc-dhcp/install
Once these commands have been run, the following should exist in the directory bin/ar71xx/packages/:
    isc-dhcp-client-ipv6_4.2.4-2_ar71xx.ipk
    isc-dhcp-server-ipv6_4.2.4-2_ar71xx.ipk
Or something very similar.Â  Using the OpenSSH utility SCP or SFTP, copy these files over to the router and install them using a command like the following:
 
opkg install client.ipk
 
Once this is complete, the code that was used in the files mentioned above will be running on the router.



## How-to: Install OpenWRT on Buffalo WZR-HP-G300HP

Perquisites

Ubuntu OS (tested on)
Compiled build of OpenWRT (tftp, extension .bin)

Steps

    sudo apt-get update
    sudo apt-get install tftp-hpa
    sudo /etc/init.d/networking stop
    sudo ifconfig eth0 192.168.11.2
    sudo ifconfig eth0 netmask 255.255.255.0
    sudo arp -s 192.168.11.1 02:AA:BB:CC:DD:1A
    change to the folder where you put the firmware image
    tftp 192.168.11.1
    tftp> verbose
    tftp> binary
    tftp> trace
    tftp> rexmt 1
    tftp> timeout 60
    tftp> put name_of_image.bin

First Login

As long as the DIAG light is not flashing twice in succession, the flash did not fail. Wait until the DIAG light turns solid red, then follow steps for first login.

Known Issues

Router will have solid red DIAG light even when flashed successfully.

Installing the custom package

The following section contains the files required for the HIPNET routers to auto configure them selves.

Files

Assuming you have a router with OpenWRT running and the following files, place them in the following places:

HIPNET_client.ipk - your compiled package, install with 'opkg install'. Attached version is most up to date binary.

dhclient-script - /usr/sbin/dhclient-script

radvd - /etc/config/radvd

network - /etc/config/network

dhclient6 - /etc/init.d/dhclient6

dhcpd6 - /etc/init.d/dhcpd6

Example

Here is an example command to run from the router to install everything assuming file are in you user directory.

scp localuser@192.168.1.134:/Users/localuser/router_dns_client.ipk .; opkg install router_dns_client.ipk; scp localuser@192.168.1.134:/Users/localuser/dhclient-script /usr/sbin/dhclient-script; scp localuser@192.168.1.134:/Users/localuser/radvd /etc/config/radvd

Enable client

In order for the HIPNET client to run on boot run the following command once on each router.

'/etc/init.d/dhclient6 enable'



## Unit Testing

Maintenance Manual: Unit Testing
---------------------------------

ISC DHCP has a built in unit testing funtionality that enables developers to
test certain pieces of code in isolation. The functionality uses ATF(Automated 
Testing Framework) to run unit tests. The first step to builing unit tests is 
to create a tests to directory to store unit tests that lies directly beneath the
code that is to be executed. For example:

server/tests/
client/tests/

Note that while these tests directories may not already exist where needed, it
is a simple process to create the directory to store the tests in the appropriate
location.


Creating a Test Program
-------------------------

A test program may be created by simply adding a generic_test.c file to the tests
directory beneath the particular code that is to be tested, and then following the 
template exemplified in the next section. However, if the directory does not exist
already, three things are necessary before tests can be run.

First, create the directory:
$ mkdir $subdir/tests
$ cvs add tests

Second, add this new subdirectory to the build system:

To add the test program to $subdir/Makefile.am:

SUBDIRS = tests

To add to the AC_OUTPUT macro:

$subdir/tests/Makfile

Lastly, Makefile.am has to be created in the new directory, which would look like:

AM_CPPFLAGS = -I../..

check_PROGRAMS = generic_test

TEST = generic_test

generic_test_SOURCES = generic_test.c
generic_test_LDADD = ../../tests.libt_api.a

Now that this is done, tests can be added to the test program.


Writing a Test
----------------

The next step in creating new unit tests is to write the actual tests. This is 
done by adding a new unit test to an existing test program. Below is a template 
for what a generic test program looks like:

1 #include "config.h"
2 #include "t_api.h"
3 
4 #include "dhcpd.h"
5 
6 static void generic_test(void);
7 
8 testspec_t T_testlist[] = {
9 {generic_test, "generic_test"},
10 {NULL, NULL}
11 };
12 
13 static void client_test(void){
14 static const char *test_desc = "A generic test";
15 
16 t_assert("generic_test", 1, T_REQUIRED, test_desc);
17 
18 //.....Code for test.....
19 
20 t_result (T_PASS);
21 }

There are two pieces of this to pay particular attention to. First, it is
necessary to add the test to the T_testlist[] array, which is done on lines
8-11. Second, the creation of the actual test is done on lines 13-21. 
Obviously, since the test is a generic template, there is no code here to 
perform an actual test, but this can be easily remedied by filling in the 
body of the function. Once done, the test can be compiled and run, which 
will be covered next.


Compiling and Running Tests
-------------------------------

Now that unit tests have been written, the next step is to compile and run 
them. This is done with the command:

$ make check

This will run all unit tests. If the intent is to only run a single test, 
the following commands should be used:

$ cd common/tests
$ make generic_test
$ ./generic_test

When the common directory is being used just for example. Other options 
can also be used when running unit tests by using the "-u" flag.



## Future Work

Instead of running every command in the binary, it would be better to run the commands regarding DNS,RADVD,IP address, & route rules in the dhclient-script. For maintainability and ease of use across hardware, these values should be passed to the client script so that the auto config process can be handled with custom scripts for individual hardware. Here is an example for setting the script environment, but we are not sure how to pass values that we have created such as â€œup interfaceâ€":

script_write_params (client, "old_", client -> active);

script_go (client);
