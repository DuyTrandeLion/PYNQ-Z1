**Note:** Due to the size of this repository is large, [git lfs](https://git-lfs.github.com/) must be used to clone it.

### Setup guide for pynqz1-dpu aka PYNQZ1 AI
1. Run Vivado design and generate hardware.
2. Run petalinux build and copy images to the SD card.
3. Download the *AI_zynq7000_pkg.rar* package from [here](https://drive.google.com/file/d/1U3HBudJJ9hU-t6KoqeeCiWVs7NrQ_y1M/view?usp=sharing).
3. Extract the *DNNDK.rar* from *AI_zynq7000_pkg.rar* into anywhere in the SD card.
4. Boot the device, run ```wget https://bootstrap.pypa.io/get-pip.py```.
5. Run ```python3 get.pip.py```.
6. Run following commands:
    ```
    cd <DNNDK-PATH>
    vim Makefile.cfg
    make
    make install
    make python
    cd n2cube/python/dist/
    pip3 install  Edge_Vitis_AI-1.3-py2.py3-none-any.whl
    ```

### Setup guide for pynqz1-acc aka PYNQZ1 Acceleration
Just run the Vivado design => export platform => petalinux-build => petalinux-build --sdk => Install SDK => create platform and application with Vitis.

> **Limitations:** there are several Vitis accelerated libaries cannot be compiled.

### Setup guide for pynqz1-dev aka PYNQZ1 Embedded
1. Run ```git clone https://github.com/Digilent/vivado-library.git```
2. Open Vivado and source the *pynqz1-dev.tcl* file to create the project (```source ./pynqz1-dev.tcl```), the block diagram will not be create so you have to manually add the vivado-library repository to the project, then re-run the ```source ./pynqz1-dev.tcl``` in Vivado console, the block design then created. all the button/switch/LED (btns_4bits, sws_2bits...) must be manually re-added again.
3. Setup soft framebuffer device driver
	1. Put this in **linux-xlnx/drivers/video/fbdev/Kconfig**
    	```
    	config FB_VDMA
    		tristate "MYIR VDMA frame buffer support"
    		depends on FB && (ARCH_ZYNQ || ARCH_ZYNQMP)
    		select FB_CFB_FILLRECT
    		select FB_CFB_COPYAREA
    		select FB_CFB_IMAGEBLIT
    		select XILINX_AXIVDMA
    		---help---
    		  Include support for the VDMA LCD reference design framebuffer.
        ```
	Example:
	```
	config FB_XILINX
		tristate "Xilinx frame buffer support"
		depends on FB && (XILINX_VIRTEX || MICROBLAZE || ARCH_ZYNQ || ARCH_ZYNQMP)
		select FB_CFB_FILLRECT
		select FB_CFB_COPYAREA
		select FB_CFB_IMAGEBLIT
		---help---
		  Include support for the Xilinx ML300/ML403 reference design
		  framebuffer. ML300 carries a 640*480 LCD display on the board,
		  ML403 uses a standard DB15 VGA connector.

	config FB_VDMA
		tristate "MYIR VDMA frame buffer support"
		depends on FB && (ARCH_ZYNQ || ARCH_ZYNQMP)
		select FB_CFB_FILLRECT
		select FB_CFB_COPYAREA
		select FB_CFB_IMAGEBLIT
		select XILINX_AXIVDMA
		---help---
		  Include support for the VDMA LCD reference design framebuffer.

	config FB_GOLDFISH
		tristate "Goldfish Framebuffer"
		depends on FB
		depends on GOLDFISH || COMPILE_TEST
		select FB_CFB_FILLRECT
		select FB_CFB_COPYAREA
		select FB_CFB_IMAGEBLIT
		---help---
		  Framebuffer driver for Goldfish Virtual Platform
	```
	2. Put this in **linux-xlnx/drivers/video/fbdev/Makefile**
    	```
    	obj-$(CONFIG_FB_VDMA)             += vdmafb.o
        ```
	Example:
	```
	obj-$(CONFIG_FB_IBM_GXT4500)	  += gxt4500.o
	obj-$(CONFIG_FB_PS3)		  += ps3fb.o
	obj-$(CONFIG_FB_SM501)            += sm501fb.o
	obj-$(CONFIG_FB_UDL)		  += udlfb.o
	obj-$(CONFIG_FB_SMSCUFX)	  += smscufx.o
	obj-$(CONFIG_FB_XILINX)           += xilinxfb.o
	obj-$(CONFIG_FB_VDMA)		  += vdmafb.o
	obj-$(CONFIG_FB_SH_MOBILE_LCDC)	  += sh_mobile_lcdcfb.o
	obj-$(CONFIG_FB_OMAP)             += omap/
	```
	3. Copy the source and headers to linux-xlnx/drivers/video/fbdev


4. Setup digilent-dynclk driver
	1. Put this in **linux-xlnx/drivers/clk/Kconfig**
    	```
    	config COMMON_CLK_DGLNT_DYNCLK
    		tristate "Digilent axi_dynclk Driver"
    		depends on ARCH_ZYNQ || MICROBLAZE
    		help
    		---help---
    		  Support for the Digilent AXI Dynamic Clock core for Xilinx
    		  FPGAs.
        ```
	Example:
	```
	config CLK_QORIQ
		bool "Clock driver for Freescale QorIQ platforms"
		depends on (PPC_E500MC || ARM || ARM64 || COMPILE_TEST) && OF
		---help---
		  This adds the clock driver support for Freescale QorIQ platforms
		  using common clock framework.

	config COMMON_CLK_DGLNT_DYNCLK
		tristate "Digilent axi_dynclk Driver"
		depends on ARCH_ZYNQ || MICROBLAZE
		---help---
		  Support for the Digilent AXI Dynamic Clock core for Xilinx
		  FPGAs.

	config COMMON_CLK_XGENE
		bool "Clock driver for APM XGene SoC"
		default ARCH_XGENE
		depends on ARM64 || COMPILE_TEST
		---help---
		  Sypport for the APM X-Gene SoC reference, PLL, and device clocks.
	```
	2. Put this in **linux-xlnx/drivers/clk/Makefile**
    	```
        obj-$(CONFIG_COMMON_CLK_DGLNT_DYNCLK)	+= clk-dglnt-dynclk.o
        ```
	Example:
	```
	obj-$(CONFIG_COMMON_CLK_CDCE925)	+= clk-cdce925.o
	obj-$(CONFIG_ARCH_CLPS711X)		+= clk-clps711x.o
	obj-$(CONFIG_COMMON_CLK_DGLNT_DYNCLK)	+= clk-dglnt-dynclk.o
	obj-$(CONFIG_COMMON_CLK_CS2000_CP)	+= clk-cs2000-cp.o
	obj-$(CONFIG_ARCH_EFM32)		+= clk-efm32gg.o
	obj-$(CONFIG_COMMON_CLK_FIXED_MMIO)	+= clk-fixed-mmio.o
	```
	3. Copy the source and headers to linux-xlnx/drivers/clk


5. Setup digilent-encoder driver
	1. Put this in **linux-xlnx/drivers/gpu/drm/xlnx/Kconfig**
    	```
        config DRM_DIGILENT_ENCODER
    		tristate "Digilent VGA/HDMI DRM Encoder Driver"
    		depends on DRM_XLNX
    		help
    		  DRM slave encoder for Video-out on Digilent boards.
        ```
	Example:
	```
	config DRM_XLNX_PL_DISP
		tristate "Xilinx DRM PL display driver"
		depends on DRM_XLNX
		select VIDEOMODE_HELPERS
		help
		  DRM driver for Xilinx PL display driver, provides drm
		  crtc and plane object to display pipeline. You need to
		  choose this option if your display pipeline needs one
		  crtc and plane object with single DMA connected.

	config DRM_XLNX_SDI
		tristate "Xilinx DRM SDI Subsystem Driver"
		depends on DRM_XLNX
		help
		  DRM driver for Xilinx SDI Tx Subsystem.

	config DRM_DIGILENT_ENCODER
		tristate "Digilent VGA/HDMI DRM Encoder Driver"
		depends on DRM_XLNX
		help
		  DRM slave encoder for Video-out on Digilent boards.
	```
	2. Put this in **linux-xlnx/drivers/gpu/drm/xlnx/Makefile**
    	```
        obj-$(CONFIG_DRM_DIGILENT_ENCODER) += digilent_encoder.o
        ```
	Example:
	```
	obj-$(CONFIG_DRM_XLNX_PL_DISP) += xlnx_pl_disp.o

	xlnx-sdi-objs += xlnx_sdi.o xlnx_sdi_timing.o
	obj-$(CONFIG_DRM_XLNX_SDI) += xlnx-sdi.o

	obj-$(CONFIG_DRM_DIGILENT_ENCODER) += digilent_encoder.o
	```
	3. Copy the source and headers to **linux-xlnx/drivers/gpu/drm/xlnx**, remember to change the source file name from *digilent-encoder.c* to *digilent_encoder.c*

> **Note:** All the source files are in project-spec/meta-user/recipes-modules. Can be built as KLM or if you want to build as built-in kernel drivers, follow below guide. By default, all 3 KLM are not enabled so you need to enable in rootfs by running ```petalinux-config -c rootfs```

> **Limitations:** digilent encoder and soft framebuffer should be used as KLM while digilent-dynclk can be used either KLM or kernel built-in driver.

> Too many words? Don't worry because I already made a patch for it! Have fun with it.

### Reference sources
[Bug report on DRM CRTC DMA engine drive](https://forums.xilinx.com/t5/Video-and-Audio/Bug-report-on-DRM-CRTC-DMA-engine-driver/td-p/1141378)  
[Zybo HDMI output: Wrong colors rendering on Linux](https://forums.xilinx.com/t5/Video-and-Audio/Zybo-HDMI-output-Wrong-colors-rendering-on-Linux/td-p/950612)  
[ZYNQ7000 #1-Linux interface display under PL end analog HDMI signal output environment](https://www.programmersought.com/article/92266162600/)  

