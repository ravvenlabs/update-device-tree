The below lines should be run on the Zynq SoC within the directory containing the image.ub file.

apt-get install u-boot-tools
apt-get install device-tree-compiler
dumpimage -T flat_dt -p 0 -i image.ub zImage
dumpimage -T flat_dt -p 1 -i image.ub system.dtb

dumpimage -l image.ub
use this information and a sample image.its file to create your custom image.its file
note that an image.its file for the Zynq 7020 is included in this repository.

sudo dtc -I dtb -O dts -o system.dts system.dtb
modify device tree via the instructions below
sudo dtc -I dts -O dtb -o system.dtb system.dts
mkimage -f image.its image.ub

image.its is not supplied via image.ub originally.  It needs to be created by the user.  
https://www.gibbard.me/linux_fit_images/

The device tree snippets that must be added are listed below.  Advanced users can change the buffer memory addresses such as 0x31000000 
as they were chosen arbitrarily.  The 0x43000000 reg memory address must match the AXI lite memory address of the Xilinx VDMA block.  This
device tree modification will allow for one to run the user space VDMA driver in tripple buffering mode.

		vdma1-read-reg {
			compatible = "generic-uio";
			reg = <0x43000000 0x1000>;
		};
		
		vdma1-read-buf1 {
			compatible = "generic-uio";
			reg = <0x31000000 0x1000000>;
		};
		
		vdma1-read-buf2 {
			compatible = "generic-uio";
			reg = <0x32000000 0x1000000>;
		};
		
		vdma1-read-buf3 {
			compatible = "generic-uio";
			reg = <0x33000000 0x1000000>;
		};

		reserved-memory {
			#address-cells = <0x1>;
			#size-cells = <0x1>;
			ranges;
			no-map;
			reserved@31000000 {
				reg = <0x31000000 0x3000000>;
			};
		};

also the boot args must be updated to allow for UIO as shown below.

bootargs = "root=/dev/mmcblk0p2 rw earlyprintk rootfstype=ext4 rootwait devtmpfs.mount=1 uio_pdrv_genirq.of_id=\"generic-uio\" clk_ignore_unused";

A complete .dts file is also included in the repository for reference.
