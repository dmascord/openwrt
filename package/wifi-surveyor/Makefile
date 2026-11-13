include $(TOPDIR)/rules.mk

PKG_NAME:=wifi-surveyor
PKG_RELEASE:=4

include $(INCLUDE_DIR)/package.mk

define Package/wifi-surveyor-agent
  SECTION:=net
  CATEGORY:=Network
  TITLE:=Wi-Fi survey agent
  PKGARCH:=all
  DEPENDS:=+iw +jsonfilter
endef

define Package/wifi-surveyor-agent/description
  Lightweight Wi-Fi survey tooling for OpenWrt.
  Provides AP-side collection and optional push-to-core behavior.
endef

define Package/wifi-surveyor-core
  SECTION:=net
  CATEGORY:=Network
  TITLE:=Wi-Fi survey core API/aggregator
  PKGARCH:=all
  DEPENDS:=+wifi-surveyor-agent +rpcd +uhttpd +jsonfilter
endef

define Package/wifi-surveyor-core/description
  Main-router API and aggregator for Wi-Fi survey data.
endef

define Build/Compile
endef

define Package/wifi-surveyor-agent/install
	$(INSTALL_DIR) $(1)/etc/config
	$(INSTALL_CONF) ./files/etc/config/wifi-surveyor $(1)/etc/config/wifi-surveyor
	$(INSTALL_DIR) $(1)/etc/init.d
	$(INSTALL_BIN) ./files/etc/init.d/wifi-surveyor $(1)/etc/init.d/wifi-surveyor
	$(INSTALL_DIR) $(1)/usr/sbin
	$(INSTALL_BIN) ./files/usr/sbin/wifi-surveyor-agent $(1)/usr/sbin/wifi-surveyor-agent
	$(INSTALL_BIN) ./files/usr/sbin/wifi-surveyor-authorize-key $(1)/usr/sbin/wifi-surveyor-authorize-key
	$(INSTALL_BIN) ./files/usr/sbin/wifi-surveyor-cache $(1)/usr/sbin/wifi-surveyor-cache
endef

define Package/wifi-surveyor-core/install
	$(INSTALL_DIR) $(1)/www/cgi-bin
	$(INSTALL_BIN) ./files/www/cgi-bin/wifi-surveyor-api $(1)/www/cgi-bin/wifi-surveyor-api
	$(INSTALL_BIN) ./files/www/cgi-bin/wifi-surveyor-dashboard $(1)/www/cgi-bin/wifi-surveyor-dashboard
	$(INSTALL_BIN) ./files/www/cgi-bin/wifi-surveyor-throughput $(1)/www/cgi-bin/wifi-surveyor-throughput
endef

$(eval $(call BuildPackage,wifi-surveyor-agent))
$(eval $(call BuildPackage,wifi-surveyor-core))
