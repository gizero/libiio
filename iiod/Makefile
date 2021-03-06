# libiio - Library for interfacing industrial I/O (IIO) devices
#
# Copyright (C) 2014 Analog Devices, Inc.
# Author: Paul Cercueil <paul.cercueil@analog.com>
#
# This library is free software; you can redistribute it and/or
# modify it under the terms of the GNU Lesser General Public
# License as published by the Free Software Foundation; either
# version 2.1 of the License, or (at your option) any later version.
#
# This library is distributed in the hope that it will be useful,
# but WITHOUT ANY WARRANTY; without even the implied warranty of
# MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the GNU
# Lesser General Public License for more details.

PREFIX ?= /usr/local

COMPILER ?= gcc
CC := $(CROSS_COMPILE)$(COMPILER)
ANALYZER := clang --analyze
INSTALL ?= install

DAEMON = iiod

CFLAGS = -Wall -ansi -D_POSIX_C_SOURCE=200809L \
	-DLIBIIO_VERSION_GIT="\"$(VERSION_GIT)\"" \
	-DLIBIIO_VERSION_MAJOR=$(VERSION_MAJOR) \
	-DLIBIIO_VERSION_MINOR=$(VERSION_MINOR)
LDFLAGS = $(wildcard ../libiio.so.*) -lpthread

ifeq ($(WITH_AVAHI),yes)
	CFLAGS += -DHAVE_AVAHI
	LDFLAGS += -lavahi-client -lavahi-common
endif

ifdef DEBUG
	CFLAGS += -g -ggdb
	CPPFLAGS += -DLOG_LEVEL=4 #-DWITH_COLOR_DEBUG
else
	CFLAGS += -O2
endif

ifdef V
	CMD:=
	SUM:=@\#
else
	CMD:=@
	SUM:=@echo
endif

C_OBJS = iiod.o ops.o
OBJS := $(C_OBJS) lexer.o parser.o

.PHONY: all clean analyze install uninstall

all: $(DAEMON)

$(DAEMON): $(OBJS)
	$(SUM) "  LD      $@"
	$(CMD)$(CC) -o $@ $^ $(LDFLAGS)

clean:
	$(SUM) "  CLEAN   ."
	$(CMD)rm -f $(DAEMON) $(OBJS) lexer.c parser.h parser.c $(OBJS:%.o=%.plist)

analyze:
	$(ANALYZER) $(CFLAGS) $(C_OBJS:%.o=%.c)

%.o: %.c
	$(SUM) "  CC      $@"
	$(CMD)$(CC) $(CPPFLAGS) $(CFLAGS) -c $< -o $@

%.c: %.l
	$(SUM) "  LEX     $@"
	$(CMD)flex -o $@ $<

%.c: %.y
	$(SUM) "  YACC    $@"
	$(CMD)yacc --defines=$(@:%.c=%.h) --output=$@ $<

parser.h: parser.c
lexer.o ops.o: parser.h
lexer.o: CFLAGS += -Wno-unused-function

ifeq ($(PREFIX),/usr)
ifneq ($(shell which update-rc.d),)

# System V init system
define INSTALL_IIOD_SCRIPT
	$(INSTALL) -m 0755 iiod.sh $(DESTDIR)/etc/init.d/iiod.sh
	update-rc.d iiod.sh defaults 99 01
endef
define REMOVE_IIOD_SCRIPT
	rm -f /etc/rc{0..1}.d/K01iiod.sh /etc/rc{2..5}.d/S99iiod.sh \
		/etc/init.d/iiod.sh
endef
endif
endif

install: $(DAEMON)
	$(INSTALL) -D -s -m 0755 $(DAEMON) $(DESTDIR)$(PREFIX)/sbin/$(DAEMON)
	$(INSTALL_IIOD_SCRIPT)

uninstall:
	rm -f $(DESTDIR)$(PREFIX)/sbin/$(DAEMON)
	$(REMOVE_IIOD_SCRIPT)
