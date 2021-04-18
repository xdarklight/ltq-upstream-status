# SoC families

## FALCON

This family should be dropped upstream since it has been [marked source-only in OpenWrt](https://git.openwrt.org/?p=openwrt/openwrt.git;a=commitdiff;h=c65faa62be94f3c693af0bca621d199e74b8dd1c)

## All XWAY SoCs

This includes the following SoC families:
* Amazon-SE
* Danube
* xRX100
* xRX200
* xRX300

### TODOs

* Update and use the [LDMA driver](https://github.com/torvalds/linux/tree/v5.12-rc7/drivers/dma/lgm) instead of the lantiq-specific custom DMA driver API
  * this requires updating the upstream `lantiq_xrx200.c` and `lantiq_etop.c` Ethernet drivers
  * it will give us compile-testing support for the Ethernet drivers
* porting the PCI controller driver to the "standard" PCI driver framework `PCI_DRIVERS_GENERIC`
  * this is a pre-condition for the PCIe controller driver
  * while doing this we should clean up our PCI driver (honoring the reset GPIO polarity, describing all memory regions in .dts, ...)
  * this will require getting rid of `0035-owrt-lantiq-wifi-and-ethernet-eeprom-handling.patch` in OpenWrt (probably not used anymore as all boards requiring this are marked as broken/disabled)
  * latest patches from [@xdarklight](https://github.com/xdarklight/linux/tree/lantiq-pcie-20210109)
* Update and use the [pcie-intel-gw](https://github.com/torvalds/linux/blob/v5.12-rc7/drivers/pci/controller/dwc/pcie-intel-gw.c) PCIe driver (xRX200 and xRX300 only)
  * the PCIe PHY driver is already upstreamed and used in OpenWrt, so only the PCIe controller driver is missing
  * latest patches from [@xdarklight](https://github.com/xdarklight/linux/tree/lantiq-pcie-20210109)
* Refactoring of the IRQ controller driver to use a hierarchical chained IRQ chip
  * this means we can describe the MIPS CPU interrupt-controller in our .dts(i)
  * required as soon as we want to properly implement the EIU or MSI interrupt-controller drivers
  * latest patches from [@xdarklight](https://github.com/xdarklight/linux/tree/lantiq-pcie-20210109)
* introduction of a common clock framework based CGU (and PMU?) driver
  * some [documentation](https://github.com/Mandrake-Lee/Lantiq_XWAY_CGU) was created by @Mandrake-Lee
  * also some [patches were written by @Mandrake-Lee](https://github.com/Mandrake-Lee/openwrt/commit/9a79bc33df053e82b0eb4aed41191b251a83fbaa)
* implement a soc-info driver (for example as in `drivers/soc/amlogic/meson-mx-socinfo.c`)
  * this can then be used to identify the SoC in other drivers, for example the `ltq_cputemp.c` driver
  * see `meson_drm_soc_attrs` in `drivers/gpu/drm/meson/meson_drv.c` how this can be used from another driver
* switch to the `drivers/mtd/nand/raw/intel-nand-controller.c` NAND driver
  * `xway_nand.c` uses legacy raw NAND APIs which need to be updated anyways
  * switching to the `intel-nand-controller.c` probably also means that we gain HW ECC support on xRX300 (xRX200 does not have HW ECC support yet)
* switch to the `reset-intel-gw.c`
  * according to the Intel LGM maintainers the reset IP is backwards compatible
  * also according to them the `reset-lantiq.c` implementation has some issues, [see the full discussion](https://lkml.org/lkml/2019/8/23/18)

## xRX100

### TODOs

* Update the `lantiq_xrx200.c` driver to support the Ethernet controller (MAC) in this SoC
  * The register layout is very similar and the DMA integration also seems to be (mostly?) identical
* Write a DSA tag and DSA driver for the 3 port switch

## xRX200

### TODOs

* properly describe the registers for `ltq-cputemp.c` in .dts
* upstream solution for the hardware-driven Ethernet PHY LEDs
  * another PHY has similar functionality so there was an [RFC patch-series](https://www.spinics.net/lists/linux-leds/msg17241.html) some time ago
