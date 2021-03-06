# non-recursive prologue
sp := $(sp).x
dirstack_$(sp) := $(d)
d := $(abspath $(lastword $(MAKEFILE_LIST))/..)

ifeq ($(origin GUARD_$(d)), undefined)
GUARD_$(d) := 1


#
# E N V I R O N M E N T  C O N F I G U R A T I O N
#
include $(d)/../GNUmakefile

include $(d)/lib/GNUmakefile


#
# C O M P I L A T I O N  F L A G S
#
VENDOR_$(d) = $(or $(CQUEUES_VENDOR),$(shell $(<D)/../mk/changelog author))
VERSION_$(d) = $(or $(CQUEUES_VERSION),$(shell $(<D)/../mk/changelog version))
COMMIT_$(d) = $(shell $(<D)/../mk/changelog commit)

CPPFLAGS_$(d) = $(ALL_CPPFLAGS) -DCOMPAT53_PREFIX=cqueues
CFLAGS_$(d) = $(ALL_CFLAGS)
SOFLAGS_$(d) = $(ALL_SOFLAGS)
LDFLAGS_$(d) = $(ALL_LDFLAGS)
LIBS_$(d) = $(ALL_LIBS)

CPPFLAGS_$(d) += $(and $(VENDOR_$(<D)),-DCQUEUES_VENDOR='"$(VENDOR_$(<D))"')
CPPFLAGS_$(d) += $(and $(VERSION_$(<D)),-DCQUEUES_VERSION=$(VERSION_$(<D))L)
CPPFLAGS_$(d) += $(and $(COMMIT_$(<D)),-DCQUEUES_COMMIT='"$(COMMIT_$(<D))"')

#
# C O M P I L A T I O N  R U L E S
#
OBJS_$(d) = cqueues.o socket.o errno.o signal.o thread.o notify.o dns.o

# NOTE: M4 might fail so delay creating $@
$(d)/errno.c: $(d)/errno.c.m4
	cd $(@D) && $(M4) $< >| $@.tmp
	mv $@.tmp $@

$(d)/config.h: $(abspath $(d)/..)/config.h
	$(CP) $< $@

define BUILD_$(d)

$$(d)/$(1)/cqueues.so: $$(addprefix $$(d)/$(1)/, $$(OBJS_$(d))) $$(d)/lib/libnonlua.a
	$$(CC) -o $$@ $$^ $$(SOFLAGS_$$(abspath $$(@D)/..)) $$(LDFLAGS_$$(abspath $$(@D)/..)) $$(LIBS_$$(abspath $$(@D)/..))

$$(d)/$(1)/_cqueues.a: $$(addprefix $$(d)/$(1)/, $$(OBJS_$(d))) $$(d)/lib/libnonlua.a $$(d)/lib/socket.o $$(d)/lib/dns.o $$(d)/lib/notify.o
	$$(AR) rcs $$@ $$^


$$(d)/$(1)/%.o: $$(d)/%.c $$(d)/cqueues.h $$(d)/../vendor/compat53/c-api/compat-5.3.h $$(d)/config.h
	$$(MKDIR) -p $$(@D)
	$$(CC) $$(CFLAGS_$$(<D)) $$(ALL_LUA$(subst .,,$(1))_CPPFLAGS) $$(CPPFLAGS_$$(<D)) -c -o $$@ $$<

$$(d)/$(1)/cqueues.o: $$(d)/lib/llrb.h

$$(d)/$(1)/socket.o: $$(d)/lib/socket.h $$(d)/lib/dns.h $$(d)/lib/fifo.h

$$(d)/$(1)/errno.o: $$(d)/lib/socket.h $$(d)/lib/dns.h

$$(d)/$(1)/dns.o: $$(d)/lib/dns.h

$$(d)/$(1)/thread.o: $$(d)/lib/llrb.h

$$(d)/$(1)/notify.o: $$(d)/lib/notify.h

ifneq ($(1), 5.3)
$$(d)/$(1)/compat53.o: $$(d)/../vendor/compat53/c-api/compat-5.3.c $$(d)/../vendor/compat53/c-api/compat-5.3.h $$(d)/config.h
	$$(MKDIR) -p $$(@D)
	$$(CC) $$(CFLAGS_$(d)) $$(ALL_LUA$(subst .,,$(1))_CPPFLAGS) $$(CPPFLAGS_$(d)) -c -o $$@ $$<

$$(d)/$(1)/cqueues.so: $$(d)/$(1)/compat53.o
endif

.SECONDARY: liblua$(1)-cqueues cqueues$(1) cqueues

liblua$(1)-cqueues cqueues$(1) cqueues: $$(d)/$(1)/cqueues.so $$(d)/$(1)/_cqueues.a

endef # BUILD_$(d)

# $(eval $(call BUILD_$(d),5.1))
# $(eval $(call BUILD_$(d),5.2))
$(eval $(call BUILD_$(d),5.3))

ifneq "$(filter $(abspath $(d)/..)/%, $(abspath $(firstword $(MAKEFILE_LIST))))" ""
# .SECONDARY: all all5.1 all5.2 all5.3
.SECONDARY: all all5.3

# all5.1: liblua5.1-cqueues
# all5.2: liblua5.2-cqueues
all5.3: liblua5.3-cqueues
all: $(foreach API,$(strip $(LUA_APIS)),all$(API))

endif


#
# I N S T A L L  &  U N I N S T A L L  R U L E S
#
define INSTALL_$(d)

MODS$(1)_$(d) = \
	$$(DESTDIR)$(2)/_cqueues.so \
	$$(DESTDIR)$(3)/cqueues.lua \
	$$(DESTDIR)$(3)/cqueues/socket.lua \
	$$(DESTDIR)$(3)/cqueues/errno.lua \
	$$(DESTDIR)$(3)/cqueues/signal.lua \
	$$(DESTDIR)$(3)/cqueues/thread.lua \
	$$(DESTDIR)$(3)/cqueues/notify.lua \
	$$(DESTDIR)$(3)/cqueues/condition.lua \
	$$(DESTDIR)$(3)/cqueues/promise.lua \
	$$(DESTDIR)$(3)/cqueues/auxlib.lua \
	$$(DESTDIR)$(3)/cqueues/dns.lua \
	$$(DESTDIR)$(3)/cqueues/dns/resolver.lua \
	$$(DESTDIR)$(3)/cqueues/dns/config.lua \
	$$(DESTDIR)$(3)/cqueues/dns/hosts.lua \
	$$(DESTDIR)$(3)/cqueues/dns/hints.lua \
	$$(DESTDIR)$(3)/cqueues/dns/record.lua \
	$$(DESTDIR)$(3)/cqueues/dns/packet.lua \
	$$(DESTDIR)$(3)/cqueues/dns/resolvers.lua

.SECONDARY: liblua$(1)-cqueues-install cqueues$(1)-install

liblua$(1)-cqueues-install cqueues$(1)-install: $$(MODS$(1)_$$(d))

$$(DESTDIR)$(2)/_cqueues.so: $$(d)/$(1)/cqueues.so
	$$(MKDIR) -p $$(@D)
	cp -p $$< $$@

$$(DESTDIR)$(3)/%.lua: $$(d)/%.lua
	$$(LUAC$(subst .,,$(1))) -p $$<
	$$(MKDIR) -p $$(@D)
	cp -p $$< $$@

$$(DESTDIR)$(3)/cqueues/%.lua: $$(d)/%.lua
	$$(LUAC$(subst .,,$(1))) -p $$<
	$$(MKDIR) -p $$(@D)
	cp -p $$< $$@

$$(DESTDIR)$(3)/cqueues/dns/%.lua: $$(d)/dns.%.lua
	$$(LUAC$(subst .,,$(1))) -p $$<
	$$(MKDIR) -p $$(@D)
	cp -p $$< $$@

.PHONY: liblua$(1)-cqueues-uninstall cqueues$(1)-uninstall

liblua$(1)-cqueues-uninstall cqueues$(1)-uninstall:
	$$(RM) -f $$(MODS$(1)_$(d))
	-$$(RMDIR) $$(DESTDIR)$(3)/cqueues/dns
	-$$(RMDIR) $$(DESTDIR)$(3)/cqueues

endef # INSTALL_$(d)

$(eval $(call INSTALL_$(d),5.1,$$(lua51cpath),$$(lua51path)))
$(eval $(call INSTALL_$(d),5.2,$$(lua52cpath),$$(lua52path)))
$(eval $(call INSTALL_$(d),5.3,$$(lua53cpath),$$(lua53path)))

ifneq "$(filter $(abspath $(d)/..)/%, $(abspath $(firstword $(MAKEFILE_LIST))))" ""

.SECONDARY: install5.1 install5.2 install5.3 install

install5.1: liblua5.1-cqueues-install
install5.2: liblua5.2-cqueues-install
install5.3: liblua5.3-cqueues-install
install: $(foreach API,$(strip $(LUA_APIS)),install$(API))

.PHONY: uninstall5.1 uninstall5.2 uninstall5.3 uninstall

uninstall5.1: liblua5.1-cqueues-uninstall
uninstall5.2: liblua5.2-cqueues-uninstall
uninstall5.3: liblua5.3-cqueues-uninstall
uninstall: $(foreach API,$(strip $(LUA_APIS)),uninstall$(API))

endif


#
# C L E A N  R U L E S
#
.PHONY: $(d)/clean $(d)/clean~ clean clean~

$(d)/clean:
	rm -rf $(@D)/errno.c $(@D)/errno.c.tmp $(@D)/config.h $(@D)/*.dSYM $(@D)/5.1 $(@D)/5.2 $(@D)/5.3

$(d)/clean~: $(d)/clean
	rm -f $(@D)/*~

clean: $(d)/clean

clean~: $(d)/clean~


#
# H E L P  R U L E S
#
.PHONY: $(d)/help help

$(d)/help:
	@echo
	@echo "src/ targets:"
	@echo ""
	@echo "         all - build all API targets"
	@echo "      all5.1 - build 5.1/cqueues.so"
	@echo "      all5.2 - build 5.2/cqueues.so"
	@echo "      all5.3 - build 5.3/cqueues.so"
	@echo "     install - install all API targets"
	@echo "  install5.1 - install cqueues Lua 5.1 modules"
	@echo "  install5.2 - install cqueues Lua 5.2 modules"
	@echo "  install5.3 - install cqueues Lua 5.3 modules"
	@echo "   uninstall - uninstall all API targets"
	@echo "uninstall5.1 - uninstall cqueues Lua 5.1 modules"
	@echo "uninstall5.2 - uninstall cqueues Lua 5.2 modules"
	@echo "uninstall5.3 - uninstall cqueues Lua 5.3 modules"
	@echo "       clean - rm binary targets, object files, debugging symbols, etc"
	@echo "      clean~ - clean + rm *~"
	@echo "        help - echo this help message"
	@echo ""
	@echo "Some important Make variables:"
	@echo ""
	@echo '  LUA_APIS - default Lua APIs to target ($(LUA_APIS))'
	@echo "    prefix - path to install root ($(value prefix))"
	@echo ' lua51path - install path for Lua 5.1 modules ($(value lua51path))'
	@echo 'lua51cpath - install path for Lua 5.1 C modules ($(value lua51cpath))'
	@echo ' lua52path - install path for Lua 5.2 modules ($(value lua52path))'
	@echo 'lua52cpath - install path for Lua 5.2 C modules ($(value lua52cpath))'
	@echo ' lua53path - install path for Lua 5.3 modules ($(value lua53path))'
	@echo 'lua53cpath - install path for Lua 5.3 C modules ($(value lua53cpath))'
	@echo ""
	@echo 'LUA51_CPPFLAGS - cpp flags for Lua 5.1 headers ($(LUA51_CPPFLAGS))'
	@echo 'LUA52_CPPFLAGS - cpp flags for Lua 5.2 headers ($(LUA52_CPPFLAGS))'
	@echo 'LUA53_CPPFLAGS - cpp flags for Lua 5.3 headers ($(LUA53_CPPFLAGS))'
	@echo ""
	@echo "(NOTE: all the common GNU-style paths are supported, including"
	@echo "prefix, bindir, libdir, datadir, includedir, and DESTDIR.)"
	@echo ""
	@echo "Report bugs to <william@25thandClement.com>"

help: $(d)/help


endif # include guard

# non-recursive epilogue
d := $(dirstack_$(sp))
sp := $(basename $(sp))
