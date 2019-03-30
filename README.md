# electron-preconnect-test-cases

This repo contains several test cases of preconnect feature to electron (https://github.com/electron/electron).

BUILDING ELECTRON (for linux)
1) Check prerequisites from guide (https://electronjs.org/docs/development/build-instructions-linux#prerequisites).
2) // fetching code  (following instructions from https://electronjs.org/docs/development/build-instructions-gn)

mkdir electron && cd electron
gclient config --name "src/electron" --unmanaged https://github.com/krunt/electron
gclient sync --with_branch_heads --with_tags

3) building  (following instructions from https://electronjs.org/docs/development/build-instructions-gn)

cd src
export CHROMIUM_BUILDTOOLS_PATH=`pwd`/buildtools
export GN_EXTRA_ARGS="${GN_EXTRA_ARGS} cc_wrapper=\"${PWD}/electron/external_binaries/sccache\""
gn gen out/Debug --args="import(\"//electron/build/args/debug.gn\") $GN_EXTRA_ARGS"
ninja -C out/Debug electron

RUNNING TEST CASES
to run any cases:
1) <path-to-elector-bin-dir>/electron --log-net-log=net.log.json electron-quick-start-rheader
	or
   <path-to-elector-bin-dir>/electron --log-net-log=net.log.json electron-quick-start-rlink
  or
   <path-to-elector-bin-dir>/electron --log-net-log=net.log.json electron-quick-start-heur

2) open net.log.json with chromium-log-viewer https://netlog-viewer.appspot.com
   look at events section

description of test cases:

a) directory electron-quick-start-rheader - preconnect through response header to fonts.gstatic.com
initial loading is to https://preconnect.herokuapp.com, 
response header contains 'Link:' header to fonts.gstatic.com

in events section
HTTP_STREAM_JOB_CONTROLLER (is_preconnect=false) and associated HTTP_STREAM_JOB to https://preconnect.herokuapp.com/
and then HTTP_STREAM_JOB_CONTROLLER (is_preconnect=true) and associated HTTP_STREAM_JOB (num_sockets=2) to https://preconnect.herokuapp.com/
this is chromium default preconnect behaviour
then must be HTTP_STREAM_JOB_CONTROLLER and HTTP_STREAM_JOB to fonts.gstatic.com (is_preconnect=true, num_sockets=1)
- this are requested by explicit response header (num_sockets==1 because origin of url != origin of frame url)

b) directory electron-quick-start-rlink - preconnect through html-link-tag to https://cdn.domain.com
in events section
HTTP_STREAM_JOB_CONTROLLER (is_preconnect=true) and associated HTTP_STREAM_JOB (num_sockets=1) to https://cdn.domain.com/

c) directory electron-quick-start-heur - preconnect to https://dailyfx.com/gbp-usd 
																		  with explicit BrowserWindow option numSocketsToPreconnect=6
in events section
HTTP_STREAM_JOB_CONTROLLER (is_preconnect=false) and associated HTTP_STREAM_JOB to https://dailyfx.com/
and then HTTP_STREAM_JOB_CONTROLLER (is_preconnect=true) and associated HTTP_STREAM_JOB (num_sockets=6) to https://dailyfx.com/
