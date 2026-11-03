# Game-free tests of the reference code (any C99 compiler; the Windows adapter test needs mingw-w64).
CC ?= cc
CFLAGS ?= -O1 -g -Wall -Wextra -std=c99 -I.
test: build/coop_invite_test build/coop_discover_test
	build/coop_invite_test
	build/coop_discover_test
build/%: tests/%.c coop_invite.h coop_discover.h launcher_profiles.h
	@mkdir -p build
	$(CC) $(CFLAGS) -o $@ $<
win-test:
	x86_64-w64-mingw32-gcc $(CFLAGS) -o build/coop_discover_win_test.exe tests/coop_discover_win_test.c -lws2_32 -liphlpapi
clean:
	rm -rf build
.PHONY: test win-test clean
