# nwjs-macos

## Steps to build NW.js binaries on MacOS 

* MacOS Version : MacoS Monterey - 12.2
* pyhton3 version: 3.10.8


## Prerequisites

### System requirements

```bash
$ ls `xcode-select -p`/Platforms/MacOSX.platform/Developer/SDKs
```
To check whether you have it, and what version you have. 

output:
```bash
MacOSX.sdk	MacOSX12.3.sdk
```

### Install depot_tools

Clone the depot_tools repository:

```bash
$ git clone https://chromium.googlesource.com/chromium/tools/depot_tools.git
```

Add depot_tools to the end of your PATH (you will probably want to put this in your ~/.bash_profile or ~/.zshrc). Assuming you cloned depot_tools to /path/to/depot_tools (note: you must use the absolute path or Python will not be able to find infra tools):

```bash
$ export PATH="$PATH:/path/to/depot_tools"
```

### Get the Code
#### Step1:
```bash
mkdir -p $HOME/nwjs
cd $HOME/nwjs
gclient config --name=src https://github.com/nwjs/chromium.src.git@origin/nw71
```
This step will create a hidden file named ".gclient" 
Note: Here I have checked out the latest version of nw which is nw71

Generally if you are not interested in running Chromium tests, you don’t have to sync the test cases and reference builds, which saves you lot of time. Open the .gclient file you just created and replace custom_deps section with followings:

```bash
"custom_deps" : {
    "src/third_party/WebKit/LayoutTests": None,
    "src/chrome_frame/tools/test/reference_build/chrome": None,
    "src/chrome_frame/tools/test/reference_build/chrome_win": None,
    "src/chrome/tools/test/reference_build/chrome": None,
    "src/chrome/tools/test/reference_build/chrome_linux": None,
    "src/chrome/tools/test/reference_build/chrome_mac": None,
    "src/chrome/tools/test/reference_build/chrome_win": None,
}
```

Manually clone and checkout correct branches (in my case it was nw71) for following repositories:

	                  
* path: src/content/nw	        repo: https://github.com/nwjs/nw.js (create a src folder, cd to src folder , clone, and rename the nw.js folder to nw)
* path: src/third_party/node-nw	repo: https://github.com/nwjs/node (cd to src folder , clone, and rename the node folder to node-nw)
* path: src/v8	                repo: https://github.com/nwjs/v8 (cd to src folder , clone)

![content](https://user-images.githubusercontent.com/2243744/207940680-810fcd76-8625-41c3-bcbc-f1de6936eeb1.png)

#### Step2:
Run following command in your terminal:

```bash
cd nwjs
gclient sync --no-history 
```

This usually downloads 20G+ from GitHub and Google’s Git repos. Make sure you have a good network provider and be patient.

When finished, you will see a src folder created in the same folder as .gclient.

## Generate ninja build files with GN for Chromium

```bash
export GYP_DEFINES="target_arch=x64 building_nw=1"
gn gen out/nw '--args=is_debug=false is_component_ffmpeg=true target_cpu="x64" symbol_level=1 nwjs_sdk=false proprietary_codecs=true ffmpeg_branding="Chromium" enable_stripping=true enable_dsyms=true enable_precompiled_headers=false' --root=.
```

Note: For above cmd I received follwoing error: 

```bashERROR Unresolved dependencies.
//content/nw:dump(//build/toolchain/mac:clang_x64)
  needs //chrome:nw_sym_archive(//build/toolchain/mac:clang_x64)
  ```
  
  Resolution : Remove this line in nw/BUILD.gn
  
  ```bash
  "//chrome:nw_sym_archive" 
  ```
  
  For more info refer to this link : https://github.com/nwjs/nw.js/issues/7438

## Generate ninja build files with GYP for Node

```bash
export GYP_CHROMIUM_NO_ACTION=0
export GYP_DEFINES="target_arch=x64 building_nw=1 clang=1 nwjs_sdk=1 disable_nacl=0 buildtype=Official"
export GYP_GENERATORS=ninja
export GYP_GENERATOR_FLAGS=output_dir=out
```

```bash
python3 src/third_party/node-nw/tools/gyp/gyp_main.py -I src/third_party/node-nw/common.gypi -D build_type=Release src/third_party/node-nw/node.gyp   
```

## Build nwjs

 ```bash
cd src
ninja -C out/nw nwjs 
```

Note: While building nwjs , I encountered following missing files error. They were in different folder , I simply copied them where the error was coming from.

```bash
Error processing node <?xml version="1.0" encoding="UTF-8"?>
<include file="${root_gen_dir}\chrome\browser\resources\inline_login\preprocessed\inline_login_app.js" name="IDR_INLINE_LOGIN_APP_JS" type="BINDATA" use_base_dir="false" />: [Errno 2] No such file or directory: '../../out/nw/gen/chrome/browser/resources/inline_login/preprocessed/inline_login_app.js'

Error processing node <?xml version="1.0" encoding="UTF-8"?>
<include file="${root_gen_dir}\chrome\browser\resources\inline_login\preprocessed\inline_login_browser_proxy.js" name="IDR_INLINE_LOGIN_BROWSER_PROXY_JS" type="BINDATA" use_base_dir="false" />: [Errno 2] No such file or directory: '../../out/nw/gen/chrome/browser/resources/inline_login/preprocessed/inline_login_browser_proxy.js'
```

## Build Node

```bash
cd src
ninja -C out/Release node
```

After building Node, the final step is to copy the build Node library to the nwjs binary folder:

```bash
cd src
ninja -C out/nw copy_node
```

## Test your binaries with test app 

Follow the steps listed under following section : https://nwjs.readthedocs.io/en/latest/For%20Users/Getting%20Started/

```bash 
Write NW.js App
Example 1 - Hello World
```

Reference link for old build steps : 
https://nwjs.readthedocs.io/en/latest/For%20Developers/Building%20NW.js/






