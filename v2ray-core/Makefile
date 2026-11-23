# SPDX-License-Identifier: GPL-3.0-only
#
# v2raya_core - the merged core (xray-core + MultiObservatory) required by
# v2rayA 2.5.16. Since 2.5.x v2rayA no longer accepts standalone xray-core or
# v2ray-core: it insists on a binary named "v2raya_core" whose version matches
# the v2rayA main program exactly. Official releases have no mipsel build, so
# we compile it from the same v2rayA source tarball's core/ subtree.
#
# Reuses the v2rayA source archive (same PKG_SOURCE/PKG_HASH) -> shared download.

include $(TOPDIR)/rules.mk

PKG_NAME:=v2raya-core
PKG_VERSION:=2.5.16
PKG_RELEASE:=1

PKG_SOURCE:=v2rayA-$(PKG_VERSION).tar.gz
PKG_SOURCE_URL:=https://codeload.github.com/v2rayA/v2rayA/tar.gz/v$(PKG_VERSION)?
PKG_HASH:=07813e4bb244363f6e989c9268ce86a6f2b1dc724dc20a2097a607800b83d9da

PKG_BUILD_DIR:=$(BUILD_DIR)/v2raya-core-$(PKG_VERSION)

PKG_MAINTAINER:=WorkBuddy
PKG_LICENSE:=MPL-2.0

PKG_BUILD_DEPENDS:=golang/host
PKG_BUILD_PARALLEL:=1
PKG_BUILD_FLAGS:=no-mips16

GO_PKG:=github.com/v2rayA/v2raya-core
GO_PKG_BUILD_PKG:=github.com/v2rayA/v2raya-core/main
GO_PKG_LDFLAGS_X:=main.Version=$(PKG_VERSION)

include $(INCLUDE_DIR)/package.mk
include $(TOPDIR)/feeds/packages/lang/golang/golang-package.mk

# core/go.mod requires Go 1.27; SDK toolchain is older -> let Go fetch it
GO_PKG_DEFAULT_LDFLAGS:=$(filter-out -linkmode external,$(GO_PKG_DEFAULT_LDFLAGS))
GO_PKG_BUILD_VARS:=$(filter-out GOTOOLCHAIN=%,$(GO_PKG_BUILD_VARS))
GO_PKG_BUILD_VARS+=GOTOOLCHAIN=auto

# Extract only the core/ subtree (strip the two leading path components) so
# that go.mod, main/, dns/ etc. land directly in PKG_BUILD_DIR.
define Build/Prepare
	mkdir -p $(PKG_BUILD_DIR)
	$(HOST_TAR) -xzf $(DL_DIR)/$(PKG_SOURCE) -C $(PKG_BUILD_DIR) --strip-components=2 v2rayA-$(PKG_VERSION)/core
endef

define Package/v2raya-core
  TITLE:=Merged core binary required by v2rayA 2.5.x
  SECTION:=net
  CATEGORY:=Network
  SUBMENU:=Web Servers/Proxies
  DEPENDS:=$(GO_ARCH_DEPENDS)
  URL:=https://v2raya.org
endef

define Package/v2raya-core/description
  v2raya_core is the merged proxy core (xray-core + MultiObservatory) required
  by v2rayA 2.5.x. Its version must match the v2rayA main program exactly.
endef

define Package/v2raya-core/install
	$(call GoPackage/Package/Install/Bin,$(PKG_INSTALL_DIR))
	$(INSTALL_DIR) $(1)/usr/bin/
	$(INSTALL_BIN) $(PKG_INSTALL_DIR)/usr/bin/main $(1)/usr/bin/v2raya_core
endef

$(eval $(call BuildPackage,v2raya-core))
