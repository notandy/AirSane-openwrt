include $(TOPDIR)/rules.mk

PKG_NAME:=airsaned
PKG_RELEASE:=1

PKG_SOURCE_PROTO:=git
PKG_SOURCE_URL:=https://github.com/SimulPiscator/AirSane.git
PKG_SOURCE_VERSION:=04cd417c26d60a609d264cc16224f1da0682da69
PKG_SOURCE_DATE:=
PKG_VERSION:=
PKG_MIRROR_HASH:=e6c09a05252a9363c409b9baf69d3899e7190bddbaaa0e1cae97f4b77db9ce5e

PKG_MAINTAINER:=SimulPiscator <example@example.com>
PKG_LICENSE:=GPL-3.0

PKG_INSTALL:=1
PKG_FIXUP:=autoreconf
PKG_BUILD_PARALLEL:=1
PKG_BUILD_DIR:=$(BUILD_DIR)/$(PKG_NAME)-$(PKG_VERSION)

include $(INCLUDE_DIR)/package.mk
include $(INCLUDE_DIR)/cmake.mk

define Package/airsaned
  TITLE:=A SANE WebScan frontend that supports Apple's AirScan protocol
  URL:=https://github.com/SimulPiscator/AirSane
  SECTION:=utils
  CATEGORY:=Utilities
  DEPENDS:=+libsane +libjpeg +libpng +libavahi-client +libusb-1.0 +libstdcpp
endef

define Package/airsaned/description
  A SANE WebScan frontend that supports Apple's AirScan protocol
endef

define Package/airsaned/conffiles
	/etc/config/airsaned
endef

define Package/airsaned/install 
	$(INSTALL_DIR) $(1)/usr/bin
	$(INSTALL_BIN) $(PKG_BUILD_DIR)/airsaned $(1)/usr/bin/
	$(INSTALL_DIR) $(1)/etc/init.d/
	$(INSTALL_BIN) files/airsaned.init $(1)/etc/init.d/airsaned
	$(INSTALL_DIR) $(1)/etc/config/
	$(CP) files/airsaned.default $(1)/etc/config/airsaned
endef

$(eval $(call BuildPackage,airsaned))
