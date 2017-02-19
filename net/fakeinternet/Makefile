# Copyright (c) 2017 Stan Grishin (stangri@melmac.net)
# This is free software, licensed under the GNU General Public License v3.

include $(TOPDIR)/rules.mk

PKG_NAME:=fakeinternet
PKG_VERSION:=0.0.1
PKG_RELEASE:=2
PKG_LICENSE:=GPL-3.0+
PKG_MAINTAINER:=Stan Grishin <stangri@melmac.net>

include $(INCLUDE_DIR)/package.mk

define Package/$(PKG_NAME)
	SECTION:=net
	CATEGORY:=Network
	DEPENDS:=+uhttpd +dnsmasq
	TITLE:=Simple VPN Bypass Service
	PKGARCH:=all
endef

define Package/$(PKG_NAME)/description
This service can be used to fake internet connectivity for local devices.
Can be used on routers with no internet access to suppress warnings on local devices on no internet connectivity.
endef

define Package/$(PKG_NAME)/conffiles
/etc/config/fakeinternet
endef

define Build/Prepare
	mkdir -p $(PKG_BUILD_DIR)/files/
	$(CP) ./files/vpnbypass.init $(PKG_BUILD_DIR)/files/vpnbypass.init
	sed -i "s|^\(PKG_VERSION\).*|\1='$(PKG_VERSION)-$(PKG_RELEASE)'|" $(PKG_BUILD_DIR)/files/vpnbypass.init
endef

define Build/Configure
endef

define Build/Compile
endef

define Package/$(PKG_NAME)/install
	$(INSTALL_DIR) $(1)/etc/init.d
	$(INSTALL_BIN) $(PKG_BUILD_DIR)/files/fakeinternet.init $(1)/etc/init.d/fakeinternet
	$(INSTALL_DIR) $(1)/www_fakeinternet
	$(INSTALL_BIN) ./files/fakeinternet.cgi $(1)/www_fakeinternet/error.cgi
endef

$(eval $(call BuildPackage,$(PKG_NAME)))
