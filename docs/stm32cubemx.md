# Use devicetree generated by STM32CubeMX

The [STM32CubeMX tool](https://www.st.com/en/development-tools/stm32cubemx.html "STM32CubeMX tool")
allows you to automatize the generation of devicetrees with an easy to use
interface. You can configure the state of each pin of the stm32mp1 component
from the pinout view. The tool generates devicetree for Linux, U-Boot and
Arm-Trusted-Firmware (TF-A).

For the example in STM32CubeMX, I create a test project, select the
STM32MP157C DK2 board and click on "generate code". Then I will find in the
test project directory the generated devicetree for kernel, TF-A and U-boot:

```
CA7/DeviceTree/test/kernel:
stm32mp157c-test-mx.dts

CA7/DeviceTree/test/tf-a:
stm32mp157c-test-mx.dts  stm32mp157c-test-mx-fw-config.dts  stm32mp15-mx.dtsi

CA7/DeviceTree/test/u-boot:
stm32mp157c-test-mx.dts  stm32mp157c-test-mx-u-boot.dtsi  stm32mp15-mx.dtsi
```

The generated devicetrees need to be copied in *BR2_EXTERNAL* tree. Here we
have chosen to create three folders *kernel-dts*, *uboot-dts* and *tfa-dts*
where we will put the dedicated devicetrees.

```
$ cp -a ${STM32CubeMX_output}/CA7/DeviceTree/test/kernel/* ${buildroot_external}/board/stmicroelectronics/stm32mp157/kernel-dts
$ cp -a ${STM32CubeMX_output}/CA7/DeviceTree/test/u-boot/* ${buildroot_external}/board/stmicroelectronics/stm32mp157/uboot-dts
$ cp -a ${STM32CubeMX_output}/CA7/DeviceTree/test/tf-a/* ${buildroot_external}/board/stmicroelectronics/stm32mp157/tfa-dts
```

The same have been done for the *DK1*.

To use these devicetrees in Buildroot you need to change 2 things:
* The Buildroot defconfig. Update the devicetree name and the
  `BR2_*_CUSTOM_DTS_PATH` configurations (following the paths chosen), for
  the Kernel, U-Boot and TF-A.
```
BR2_LINUX_KERNEL_INTREE_DTS_NAME="stm32mp157c-dk2-mx"
BR2_LINUX_KERNEL_CUSTOM_DTS_PATH="$(BR2_EXTERNAL_ST_PATH)/board/stmicroelectronics/stm32mp157/linux-dts/*"
BR2_TARGET_ARM_TRUSTED_FIRMWARE_CUSTOM_DTS_PATH="$(BR2_EXTERNAL_ST_PATH)/board/stmicroelectronics/stm32mp157/tfa-dts/*
BR2_TARGET_ARM_TRUSTED_FIRMWARE_ADDITIONAL_VARIABLES="STM32MP_SDMMC=1 AARCH32_SP=optee DTB_FILE_NAME=stm32mp157c-dk2-mx.dtb BL33_CFG=$(BINARIES_DIR)/u-boot.dtb"
BR2_TARGET_UBOOT_CUSTOM_DTS_PATH="$(BR2_EXTERNAL_ST_PATH)/board/stmicroelectronics/stm32mp157/uboot-dts/*"
BR2_TARGET_UBOOT_CUSTOM_MAKEOPTS="dtb-y=stm32mp157c-dk2-mx.dtb DEVICE_TREE=stm32mp157c-dk2-mx"
```
  The option `dtb-y=stm32mp157c-dk2-mx.dtb` has been added to U-boot make
  options to build the external devicetree.

* Update the devicetree name described in the `extlinux.conf` file. This
  file is located in the overlay.

You might need to modify the post-image.sh script to use other naming of TF-A
binary. Please see the commit
_configs/st: support external devicetrees for kernel, U-boot and TF-A_
as an example of using devicetree generated by STM32CubeMX.

---

The default devicetree generated by STM32MPCubeMX for the DK2 has an issue
on the display pinctrl. You need to add the pintcrl of the ltdc node
in the devicetree. Here is the change to add to the stm32mp157c-dk2-mx.dts
devicetree:

```
diff --git a/board/stmicroelectronics/stm32mp157/linux-dts/stm32mp157c-dk2-mx.dts b/board/stmicroelectronics/stm32mp157/linux-dts/stm32mp157c-dk2-mx.dts
index 0c3c6ac..eb54799 100644
--- a/board/stmicroelectronics/stm32mp157/linux-dts/stm32mp157c-dk2-mx.dts
+++ b/board/stmicroelectronics/stm32mp157/linux-dts/stm32mp157c-dk2-mx.dts
@@ -503,6 +503,74 @@
 	};
 
 	/* USER CODE BEGIN pinctrl */
+	ltdc_pins_a: ltdc-0 {
+		pins {
+			pinmux = <STM32_PINMUX('G',  7, AF14)>, /* LCD_CLK */
+				 <STM32_PINMUX('I', 10, AF14)>, /* LCD_HSYNC */
+				 <STM32_PINMUX('I',  9, AF14)>, /* LCD_VSYNC */
+				 <STM32_PINMUX('F', 10, AF14)>, /* LCD_DE */
+				 <STM32_PINMUX('H',  2, AF14)>, /* LCD_R0 */
+				 <STM32_PINMUX('H',  3, AF14)>, /* LCD_R1 */
+				 <STM32_PINMUX('H',  8, AF14)>, /* LCD_R2 */
+				 <STM32_PINMUX('H',  9, AF14)>, /* LCD_R3 */
+				 <STM32_PINMUX('H', 10, AF14)>, /* LCD_R4 */
+				 <STM32_PINMUX('C',  0, AF14)>, /* LCD_R5 */
+				 <STM32_PINMUX('H', 12, AF14)>, /* LCD_R6 */
+				 <STM32_PINMUX('E', 15, AF14)>, /* LCD_R7 */
+				 <STM32_PINMUX('E',  5, AF14)>, /* LCD_G0 */
+				 <STM32_PINMUX('E',  6, AF14)>, /* LCD_G1 */
+				 <STM32_PINMUX('H', 13, AF14)>, /* LCD_G2 */
+				 <STM32_PINMUX('H', 14, AF14)>, /* LCD_G3 */
+				 <STM32_PINMUX('H', 15, AF14)>, /* LCD_G4 */
+				 <STM32_PINMUX('I',  0, AF14)>, /* LCD_G5 */
+				 <STM32_PINMUX('I',  1, AF14)>, /* LCD_G6 */
+				 <STM32_PINMUX('I',  2, AF14)>, /* LCD_G7 */
+				 <STM32_PINMUX('D',  9, AF14)>, /* LCD_B0 */
+				 <STM32_PINMUX('G', 12, AF14)>, /* LCD_B1 */
+				 <STM32_PINMUX('G', 10, AF14)>, /* LCD_B2 */
+				 <STM32_PINMUX('D', 10, AF14)>, /* LCD_B3 */
+				 <STM32_PINMUX('I',  4, AF14)>, /* LCD_B4 */
+				 <STM32_PINMUX('A',  3, AF14)>, /* LCD_B5 */
+				 <STM32_PINMUX('B',  8, AF14)>, /* LCD_B6 */
+				 <STM32_PINMUX('D',  8, AF14)>; /* LCD_B7 */
+			bias-disable;
+			drive-push-pull;
+			slew-rate = <1>;
+		};
+	};
+
+	ltdc_sleep_pins_a: ltdc-sleep-0 {
+		pins {
+			pinmux = <STM32_PINMUX('G',  7, ANALOG)>, /* LCD_CLK */
+				 <STM32_PINMUX('I', 10, ANALOG)>, /* LCD_HSYNC */
+				 <STM32_PINMUX('I',  9, ANALOG)>, /* LCD_VSYNC */
+				 <STM32_PINMUX('F', 10, ANALOG)>, /* LCD_DE */
+				 <STM32_PINMUX('H',  2, ANALOG)>, /* LCD_R0 */
+				 <STM32_PINMUX('H',  3, ANALOG)>, /* LCD_R1 */
+				 <STM32_PINMUX('H',  8, ANALOG)>, /* LCD_R2 */
+				 <STM32_PINMUX('H',  9, ANALOG)>, /* LCD_R3 */
+				 <STM32_PINMUX('H', 10, ANALOG)>, /* LCD_R4 */
+				 <STM32_PINMUX('C',  0, ANALOG)>, /* LCD_R5 */
+				 <STM32_PINMUX('H', 12, ANALOG)>, /* LCD_R6 */
+				 <STM32_PINMUX('E', 15, ANALOG)>, /* LCD_R7 */
+				 <STM32_PINMUX('E',  5, ANALOG)>, /* LCD_G0 */
+				 <STM32_PINMUX('E',  6, ANALOG)>, /* LCD_G1 */
+				 <STM32_PINMUX('H', 13, ANALOG)>, /* LCD_G2 */
+				 <STM32_PINMUX('H', 14, ANALOG)>, /* LCD_G3 */
+				 <STM32_PINMUX('H', 15, ANALOG)>, /* LCD_G4 */
+				 <STM32_PINMUX('I',  0, ANALOG)>, /* LCD_G5 */
+				 <STM32_PINMUX('I',  1, ANALOG)>, /* LCD_G6 */
+				 <STM32_PINMUX('I',  2, ANALOG)>, /* LCD_G7 */
+				 <STM32_PINMUX('D',  9, ANALOG)>, /* LCD_B0 */
+				 <STM32_PINMUX('G', 12, ANALOG)>, /* LCD_B1 */
+				 <STM32_PINMUX('G', 10, ANALOG)>, /* LCD_B2 */
+				 <STM32_PINMUX('D', 10, ANALOG)>, /* LCD_B3 */
+				 <STM32_PINMUX('I',  4, ANALOG)>, /* LCD_B4 */
+				 <STM32_PINMUX('A',  3, ANALOG)>, /* LCD_B5 */
+				 <STM32_PINMUX('B',  8, ANALOG)>, /* LCD_B6 */
+				 <STM32_PINMUX('D',  8, ANALOG)>; /* LCD_B7 */
+		};
+	};
 	/* USER CODE END pinctrl */
 };
 
@@ -1060,6 +1128,9 @@
 	status = "okay";
 
 	/* USER CODE BEGIN ltdc */
+	pinctrl-names = "default", "sleep";
+	pinctrl-0 = <&ltdc_pins_a>;
+	pinctrl-1 = <&ltdc_sleep_pins_a>;
 
 	port{
 
```
