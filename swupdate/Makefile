include $(TOPDIR)/rules.mk
-include $(STAGING_DIR)/components.config

PKG_NAME:=swupdate
PKG_VERSION:=gen_2023.05_v0.0.1
SHORT_DESCRIPTION:=description

PKG_SOURCE_PROTO:=git
PKG_SOURCE_VERSION:=gen_2023.05_v0.0.1
PKG_SOURCE_URL:=https://$(SAH_GIT_USER):$(SAH_GIT_TOKEN)@$(SAH_GIT)/$(SAH_GIT_GROUP)/swupdate
PKG_MIRROR_HASH:=skip
PKG_BUILD_DIR:=$(BUILD_DIR)/$(PKG_NAME)-$(PKG_VERSION)
PKG_MAINTAINER:=Soft At Home <support.opensource@softathome.com>
PKG_LICENSE:=GPL
PKG_LICENSE_FILES:=COPYING

PKG_RELEASE:=1
PKG_BUILD_DEPENDS += mtd-utils

include $(INCLUDE_DIR)/package.mk

define Package/$(PKG_NAME)
  CATEGORY:=amx
  SUBMENU:=Plugins
  TITLE:=$(SHORT_DESCRIPTION)
  URL:=git@gitlabinternal.softathome.com:bootloader/swupdate.git
  DEPENDS += +zlib
  DEPENDS += +libopenssl
  DEPENDS += +libconfig
  DEPENDS += +lua
  DEPENDS += +libcurl
  DEPENDS += +libjson-c
  MENU:=1
endef

define Package/$(PKG_NAME)/description
	Implementation of Device.Hosts of the TR-181 datamodel
endef

define KCONFIG_SET_OPT # (option, value, file)
	sed -i "/$(1)/d" $(3)
	echo '$(1)=$(2)' >> $(3)
endef

define KCONFIG_ENABLE_OPT # (option, file)
	sed "/$(1)/d" $(2)
	echo "$(1)=y" >> $(2)
endef

define KCONFIG_DISABLE_OPT # (option, file)
	sed "/$(1)/d" $(2)
	echo "# $(1) is not set" >> $(2)
endef

LIBRARIES:=zlib openssl libconfig mtd-utils

EXTRA_CFLAGS:=$(EXTRA_CFLAGS) $(CFLAGS) $(shell pkg-config --cflags $(LIBRARIES)) -Wall -W
EXTRA_LDFLAGS:=$(EXTRA_LDFLAGS) $(LDFLAGS) $(shell pkg-config --libs $(LIBRARIES))
CFLAGS:=
LDFLAGS:=

OUTPUT_FILES_LIST:=$(STAGINGDIR)/sbl/swupdate-installed.txt
define install_output_files
for file in `cat $(OUTPUT_FILES_LIST)` ; do \
	mkdir -p `dirname $2/$$file`; cp --no-dereference $1/$$file $2/$$file;\
done
endef

define Build/Compile
	$(MAKE) -C $(PKG_BUILD_DIR) softathome_defconfig
	$(call KCONFIG_SET_OPT,CONFIG_EXTRA_CFLAGS,"$(EXTRA_CFLAGS)",$(PKG_BUILD_DIR)/.config)
	$(call KCONFIG_SET_OPT,CONFIG_EXTRA_LDFLAGS,"$(EXTRA_LDFLAGS)",$(PKG_BUILD_DIR)/.config)
	$(call KCONFIG_SET_OPT,CONFIG_HW_COMPATIBILITY_FILE,$(CONFIG_OPENSOURCE_SWUPDATE_HWREVISION),$(PKG_BUILD_DIR)/.config)
	$(call KCONFIG_SET_OPT,CONFIG_SW_VERSIONS_FILE,$(CONFIG_OPENSOURCE_SWUPDATE_SWVERSIONS),$(PKG_BUILD_DIR)/.config)
	$(call KCONFIG_DISABLE_OPT,CONFIG_UBOOT,$(PKG_BUILD_DIR)/.config)
	$(call KCONFIG_ENABLE_OPT,CONFIG_BOOTLOADER_NONE,$(PKG_BUILD_DIR)/.config)
	$(call KCONFIG_ENABLE_OPT,CONFIG_UBIVOL,$(PKG_BUILD_DIR)/.config)
	$(call KCONFIG_DISABLE_OPT,CONFIG_UBIATTACH,$(PKG_BUILD_DIR)/.config)
ifneq ($(CONFIG_OPENSOURCE_SURICATTA),)
	$(call KCONFIG_ENABLE_OPT,CONFIG_SURICATTA,$(PKG_BUILD_DIR)/.config)
ifneq ($(CONFIG_OPENSOURCE_SURICATTA_SSL),)
	$(call KCONFIG_ENABLE_OPT,CONFIG_SURICATTA_SSL,$(PKG_BUILD_DIR)/.config)
else
	$(call KCONFIG_DISABLE_OPT,CONFIG_SURICATTA_SSL,$(PKG_BUILD_DIR)/.config)
endif
ifneq ($(CONFIG_OPENSOURCE_SURICATTA_HAWKBIT),)
	$(call KCONFIG_ENABLE_OPT,CONFIG_SURICATTA_HAWKBIT,$(PKG_BUILD_DIR)/.config)
	$(call KCONFIG_DISABLE_OPT,CONFIG_SURICATTA_GENERAL,$(PKG_BUILD_DIR)/.config)
else
	$(call KCONFIG_DISABLE_OPT,CONFIG_SURICATTA_HAWKBIT,$(PKG_BUILD_DIR)/.config)
	$(call KCONFIG_ENABLE_OPT,CONFIG_SURICATTA_GENERAL,$(PKG_BUILD_DIR)/.config)
endif
endif

ifneq ($(CONFIG_OPENSOURCE_SWUPDATE_WEBSERVER),)
	$(call KCONFIG_ENABLE_OPT,CONFIG_WEBSERVER,$(PKG_BUILD_DIR)/.config)
	ifneq ($(CONFIG_OPENSOURCE_SWUPDATE_MONGOOSE),)
		$(call KCONFIG_ENABLE_OPT,CONFIG_MONGOOSE,$(PKG_BUILD_DIR)/.config)
	else
		$(call KCONFIG_DISABLE_OPT,CONFIG_MONGOOSE,$(PKG_BUILD_DIR)/.config)
	endif
	ifneq ($(CONFIG_OPENSOURCE_SWUPDATE_MONGOOSEIPV6),)
		$(call KCONFIG_ENABLE_OPT,CONFIG_MONGOOSEIPV6,$(PKG_BUILD_DIR)/.config)
	else
		$(call KCONFIG_DISABLE_OPT,CONFIG_MONGOOSEIPV6,$(PKG_BUILD_DIR)/.config)
	endif
	ifneq ($(CONFIG_OPENSOURCE_SWUPDATE_MONGOOSESSL),)
		$(call KCONFIG_ENABLE_OPT,CONFIG_MONGOOSESSL,$(PKG_BUILD_DIR)/.config)
	else
		$(call KCONFIG_DISABLE_OPT,CONFIG_MONGOOSESSL,$(PKG_BUILD_DIR)/.config)
	endif
	else
		$(call KCONFIG_DISABLE_OPT,CONFIG_WEBSERVER,$(PKG_BUILD_DIR)/.config)
endif
	$(call Build/Compile/Default)
endef

define Build/Install
	$(call Build/Install/Default,install DESTDIR=$(PKG_INSTALL_DIR))
endef

define Build/InstallDev
	echo ""
endef

define Package/$(PKG_NAME)/install
	$(CP) $(PKG_INSTALL_DIR)/* $(1)/
endef

define Package/$(PKG_NAME)/config
	source "$(SOURCE)/Config.in"
endef


$(eval $(call BuildPackage,$(PKG_NAME)))
