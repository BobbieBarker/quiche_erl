# C NIF build for quiche_erl.
#
# Two stages:
#   1. cargo build --release --features ffi -> libquiche.a
#   2. cc c_src/*.c + libquiche.a -> priv/quiche_erl_nif.so
#
# ERTS include path discovered from the running Erlang installation.

ERTS_INCLUDE_DIR := $(shell erl -noinput -eval 'io:format("~s/erts-~s/include", [code:root_dir(), erlang:system_info(version)])' -s init stop)

PRIV_DIR := ../priv
NIF_SO   := $(PRIV_DIR)/quiche_erl_nif.so
C_SOURCE := $(wildcard *.c)

CC     ?= cc
CFLAGS  = -O2 -fPIC -Wall -Wextra -Werror -std=c11 -I$(ERTS_INCLUDE_DIR)

UNAME_S := $(shell uname -s)
ifeq ($(UNAME_S),Darwin)
    LDFLAGS = -dynamiclib -undefined dynamic_lookup
else
    LDFLAGS = -shared
endif

.PHONY: all clean

all: $(NIF_SO)

ifeq ($(C_SOURCE),)
$(NIF_SO): | $(PRIV_DIR)
	@echo "No C sources yet, skipping NIF build"
else
$(NIF_SO): $(C_SOURCE) | $(PRIV_DIR)
	$(CC) $(CFLAGS) $(LDFLAGS) -o $(NIF_SO) $(C_SOURCE)
endif

$(PRIV_DIR):
	mkdir -p $(PRIV_DIR)

clean:
	rm -f $(NIF_SO)
