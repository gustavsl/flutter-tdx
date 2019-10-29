# flutter-tdx
A light-weight Flutter Engine Embedder for the Toradex Colibri iMX6. Inspired by https://github.com/chinmaygarde/flutter_from_scratch.
Flutter-tdx also runs without X11.

You can now theoretically run every flutter app you want using flutter-pi, also including extensions & plugins, just that you'd have to build the platform side of the plugins you'd like to use yourself.

_The difference between extensions and plugins is that extensions don't include any native code, they are just pure dart. Plugins (like the [connectivity plugin](https://github.com/flutter/plugins/tree/master/packages/connectivity)) include platform-specific code._

## Running your App on the Colibri iMX6
### Patching the App
First, you need to override the default target platform in your flutter app, i.e. add the following line to your _main_ method, before the _runApp_ call:
```dart
debugDefaultTargetPlatformOverride = TargetPlatform.fuchsia;
```
The _debugDefaultTargetPlatformOverride_ property is in the foundation library, so you need to import that.

Your main dart file should probably look similiar to this now:
```dart
import 'package:flutter/foundation.dart';

. . .

void main() {
  debugDefaultTargetPlatformOverride = TargetPlatform.fuchsia;
  runApp(MyApp());
}

. . .
```

### Building the Asset bundle
Then to build the asset bundle, run the following commands. I'm using flutter_gallery in this example. (note that the flutter_gallery example **does not work** with flutter-tdx, since it includes plugins that have no platform-side implementation for the embedded yet)
```bash
cd flutter/examples/flutter_gallery
flutter build bundle
```

After that `flutter/examples/flutter_gallery/build/flutter_assets` would be a valid path to pass as an argument to flutter-tdx.

### Running your App with flutter-tdx
You need to tell flutter-pi which input device to use and whether it's a touchscreen or mouse. Input devices are typically located at `/dev/input/...`. Just run `evtest` (`apt install evtest`) to find out which exact path you should use. Currently only one input device is supported by flutter-pi. In the future, I will probably let flutter-pi search for an input device by itself.

Run using
```bash
./flutter-tdx [flutter-tdx options...] /path/to/assets/bundle/directory [flutter engine arguments...]
```

`[flutter-tdx options...]` are:
- `-t /path/to/device` where `/path/to/device` is a path to a touchscreen input device (typically `/dev/input/event0` or similiar)
- `-m /path/to/device` where `/path/to/device` is a path to a mouse input device (typically `/dev/input/mouse0` or `/dev/input/event0` or similiar)

`/path/to/assets/bundle/directory` is the path of the flutter asset bundle directory (i.e. the directory containing the kernel_blob.bin)
of the flutter app you're trying to run.

`[flutter engine arguments...]` will be passed as commandline arguments to the flutter engine. You can find a list of commandline options for the flutter engine [Here](https://github.com/flutter/engine/blob/master/shell/common/switches.h).

## Dependencies
### flutter engine
flutter-tdx needs `libflutter_engine.so` and `flutter_embedder.h` to compile. It also needs the flutter engine's `icudtl.dat` at runtime.
You have to options here:

- you build the engine yourself. takes a lot of time, and it most probably won't work on the first try. But once you have it set up, you have unlimited freedom on which engine version you want to use. You can find some rough guidelines [here](https://medium.com/flutter/flutter-on-raspberry-pi-mostly-from-scratch-2824c5e7dcb1). [Andrew jones](https://github.com/andyjjones28) is working on some more detailed instructions.
- you can use the pre-built engine binaries I am providing [in the _engine-binaries_ branch of this project.](https://github.com/ardera/flutter-pi/tree/engine-binaries). I will only provide binaries for some engine versions though (most likely the stable ones).

### graphics libs
Additionally, flutter-tdx depends on mesa's OpenGL, OpenGL ES, EGL implementation and libdrm & libgbm.
You can easily install those with `apt install libgl1-mesa-dev libgles2-mesa-dev libegl-mesa0 libdrm-dev libgbm-dev`.

### fonts
The flutter engine, by default, uses the _Arial_ font. You need to install it using:
```bash
sudo apt install ttf-mscorefonts-installer
sudo fc-cache
```

## Compiling flutter-tdx (on the Colibri iMX6
fetch all the dependencies, clone this repo and run:
```bash
mkdir out
cc -D_GNU_SOURCE \
-lEGL -ldrm -lgbm -lGLESv2 -lrt -lflutter_engine -lpthread -ldl \
-I./include -I/usr/include -I/usr/include/libdrm ./src/flutter-pi.c \
./src/platformchannel.c ./src/pluginregistry.c ./src/plugins/services-plugin.c -o out/flutter-tdx 
```
## Running on Torizon
TODO

