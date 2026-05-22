# superx-core — build notes (SuperX)

Verified 2026-05-22 on Windows. Produces `bin/libcore.dll` (Go c-shared, CGO).

## Toolchain (exact — do not drift)
- **Go 1.25.6** via `GOTOOLCHAIN=go1.25.6`. NOT 1.26.x: tfo-go v2.3.1 references internal
  symbols (`net.(*netFD).init`, `internal/poll.execIO`) removed in Go 1.26 → link fails.
  go.mod requires ≥1.25.6, so 1.25.6 is the only working version in the window.
- **mingw-w64 GCC 16.1.0** (portable winlibs) at `C:\src\toolchain\mingw64\bin`
  (provides `gcc.exe` and `x86_64-w64-mingw32-gcc.exe`).

## Windows DLL build (working command)
```bash
export PATH="/c/src/toolchain/mingw64/bin:$PATH"
cd superx-core
TAGS="with_gvisor,with_quic,with_wireguard,with_utls,with_clash_api,with_grpc,with_awg,tfogo_checklinkname0,with_conntrack,with_purego"
env GOTOOLCHAIN=go1.25.6 GOOS=windows GOARCH=amd64 CC=x86_64-w64-mingw32-gcc CGO_ENABLED=1 \
  go build -trimpath -ldflags="-checklinkname=0" -buildmode=c-shared -tags "$TAGS" \
  -o bin/libcore.dll ./platform/desktop
```

## SuperX deviations from upstream Makefile windows-amd64 target
- Dropped tag **`with_naive_outbound`** → skips the cronet `build-naive extract-lib` step
  (heavy external cronet lib; naive is not in SuperX transport matrix: XHTTP/gRPC/Reality/H2).
  Re-add only if naive outbound is ever needed.
- Added `-ldflags=-checklinkname=0` (tfo-go linkname guard on Go ≥1.23).

## Submodules
hiddify-core uses git@ SSH submodule URLs (fail without SSH key). Fixed all `.gitmodules`
recursively to `https://`. Populate: `git submodule update --init --recursive`.
Local mirrors of every dep exist at C:\Users\turti\Projects\HiddifyRepos & SNrepos & XTLSrepo.

## Status / TODO
- [x] baseline libcore.dll builds (on hiddify-sing-box, XHTTP present).
- [ ] superx-singbox swap (ТЗ maintainability) — hiddify-core is coupled to hiddify-sing-box
      extensions (with_awg etc. + replace/* deps); clean swap needs code work. Separate effort.
- [ ] rebrand hiddify→superx ; wire superx-client submodule → superx-core ; full client build (needs MSVC for Flutter Windows shell).
