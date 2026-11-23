# SPDX-License-Identifier: GPL-3.0-only
#
# Copyright (C) 2021-2023 ImmortalWrt.org

include $(TOPDIR)/rules.mk

PKG_NAME:=v2rayA
PKG_VERSION:=2.5.16
PKG_RELEASE:=1

PKG_SOURCE:=$(PKG_NAME)-$(PKG_VERSION).tar.gz
PKG_SOURCE_URL:=https://codeload.github.com/v2rayA/v2rayA/tar.gz/v$(PKG_VERSION)?
PKG_HASH:=07813e4bb244363f6e989c9268ce86a6f2b1dc724dc20a2097a607800b83d9da

PKG_BUILD_DIR:=$(BUILD_DIR)/$(PKG_NAME)-$(PKG_VERSION)/service
PKG_LICENSE:=AGPL-3.0-only
PKG_LICENSE_FILES:=LICENSE
PKG_MAINTAINER:=Tianling Shen

PKG_BUILD_DEPENDS:=golang/host
PKG_BUILD_PARALLEL:=1
PKG_BUILD_FLAGS:=no-mips16

GO_PKG:=github.com/v2rayA/v2rayA
GO_PKG_LDFLAGS_X:= \
	$(GO_PKG)/conf.Version=$(PKG_VERSION) \
	$(GO_PKG)/core/iptables.TproxyNotSkipBr=true

include $(INCLUDE_DIR)/package.mk
include $(TOPDIR)/feeds/packages/lang/golang/golang-package.mk

TAR_OPTIONS+= --strip-components 1
TAR_CMD=$(HOST_TAR) -C $(1)/.. $(TAR_OPTIONS)

define Package/v2raya
  TITLE:=A Linux web GUI client of Project V
  SECTION:=net
  CATEGORY:=Network
  SUBMENU:=Web Servers/Proxies
  DEPENDS:=$(GO_ARCH_DEPENDS) +ca-bundle +v2raya-core
  URL:=https://v2raya.org
endef

define Package/v2raya/description
	v2rayA is a V2Ray Linux client supporting global transparent proxy, compatible with SS, SSR, Trojan, PingTunnel protocols.
endef

define Package/v2raya/conffiles
/etc/v2raya/
/etc/config/v2raya
endef

WEB_FILE:=$(PKG_NAME)-web-$(PKG_VERSION).tar.gz
define Download/v2raya-web
  URL:=https://github.com/v2rayA/v2rayA/releases/download/v$(PKG_VERSION)/
  URL_FILE:=web.tar.gz
  FILE:=$(WEB_FILE)
  HASH:=17d7948a48f78485ad6ca43531b0e6edcfc49307ce6e2bd46d3ee79950185456
endef

# v2rayA 2.5.16 uses SQLite. On mips/mipsle the pure-Go driver (modernc.org/sqlite)
# does NOT build (modernc.org/libc has no 32-bit mipsle support), so v2rayA ships a
# CGO-backed driver (mattn/go-sqlite3) for mips. Therefore CGO must stay ENABLED.
# OpenWrt golang-package.mk already sets CGO_ENABLED=1 + CC=$(TARGET_CC) in
# GO_PKG_TARGET_VARS -- do NOT filter CGO_ENABLED out like the old 2.2.74 recipe did.
# Only strip the external-linkmode flag (no need for it here).
GO_PKG_DEFAULT_LDFLAGS:=$(filter-out -linkmode external,$(GO_PKG_DEFAULT_LDFLAGS))

# v2rayA 2.5.16 requires Go 1.26; golang-package.mk forces GOTOOLCHAIN=local,
# strip it and explicitly allow auto-download of the matching toolchain
GO_PKG_BUILD_VARS:=$(filter-out GOTOOLCHAIN=%,$(GO_PKG_BUILD_VARS))
GO_PKG_BUILD_VARS+=GOTOOLCHAIN=auto

define Build/Prepare
	$(call Build/Prepare/Default)
	( \
		mkdir -p $(PKG_BUILD_DIR)/server/router/web ; \
		gzip -dc $(DL_DIR)/$(WEB_FILE) | $(HOST_TAR) -C $(PKG_BUILD_DIR)/server/router/web $(TAR_OPTIONS) ; \
	)
endef

define Package/v2raya/install
	$(call GoPackage/Package/Install/Bin,$(PKG_INSTALL_DIR))
	$(INSTALL_DIR) $(1)/usr/bin/
	$(INSTALL_BIN) $(PKG_INSTALL_DIR)/usr/bin/v2rayA $(1)/usr/bin/v2raya

	$(INSTALL_DIR) $(1)/etc/config/
	$(INSTALL_CONF) $(CURDIR)/files/v2raya.config $(1)/etc/config/v2raya

	$(INSTALL_DIR) $(1)/etc/init.d/
	$(INSTALL_BIN) $(CURDIR)/files/v2raya.init $(1)/etc/init.d/v2raya
	# Point v2rayA at OpenWrt's geodata dir so it does not try to download
	# geoip.dat/geosite.dat on first start (v2fly-geodata installs here).
	sed -i 's|\(append_env "config" "/etc/v2raya"\)|\1\n\tappend_env "v2ray_assetsdir" "/usr/share/v2ray"|' $(1)/etc/init.d/v2raya
endef

$(eval $(call Download,v2raya-web))
$(eval $(call GoBinPackage,v2raya))
$(eval $(call BuildPackage,v2raya))
