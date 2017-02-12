# Copyright (c) 2017 Stan Grishin (stangri@melmac.net)
# This is free software, licensed under the GNU General Public License v3.

include $(TOPDIR)/rules.mk

PKG_NAME:=openvpn-policy-routing
PKG_VERSION:=4.1.4
PKG_RELEASE:=25
PKG_LICENSE:=GPL-3.0+
PKG_MAINTAINER:=Stan Grishin <stangri@melmac.net>

include $(INCLUDE_DIR)/package.mk

define Package/$(PKG_NAME)
	SECTION:=net
	CATEGORY:=Network
	DEPENDS:=+ipset +iptables +resolveip
	CONFLICTS:=vpnbypass
	TITLE:=OpenVPN Policy-Based Routing Service
	PKGARCH:=all
endef

define Package/$(PKG_NAME)/description
This service allows policy-based routing for OpenVPN tunnels and WAN interface.
Policies can specify domains, local IPs/subnets and ports, as well as remote IPs/subnets and ports.
endef

define Package/$(PKG_NAME)/conffiles
/etc/config/openvpn-policy-routing
endef

define Build/Prepare
	mkdir -p $(PKG_BUILD_DIR)/files/
	$(CP) ./files/openvpn-policy-routing.init $(PKG_BUILD_DIR)/files/openvpn-policy-routing.init
	sed -i "s|^\(PKG_VERSION\).*|\1='$(PKG_VERSION)-$(PKG_RELEASE)'|" $(PKG_BUILD_DIR)/files/openvpn-policy-routing.init
endef

define Build/Configure
endef

define Build/Compile
endef

define Package/$(PKG_NAME)/install
	$(INSTALL_DIR) $(1)/etc/init.d
	$(INSTALL_BIN) $(PKG_BUILD_DIR)/files/openvpn-policy-routing.init $(1)/etc/init.d/openvpn-policy-routing
	$(INSTALL_DIR) $(1)/etc/config
	$(INSTALL_CONF) ./files/openvpn-policy-routing.conf $(1)/etc/config/openvpn-policy-routing
	$(INSTALL_DIR) $(1)/etc/hotplug.d/firewall
	$(INSTALL_DATA) ./files/openvpn-policy-routing.hotplug $(1)/etc/hotplug.d/firewall/99-openvpn-policy-routing
endef

$(eval $(call BuildPackage,$(PKG_NAME)))
