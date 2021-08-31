# `gcr.io/paketo-buildpacks/native-image`

The Paketo Native Image Buildpack is a Cloud Native Buildpack that uses the [GraalVM Native Image builder][native-image] (`native-image`) to compile a standalone executable from an executable JAR.

Most users should not use this component buildpack directly and should instead use the [Paketo Java Native Image][bp/java-native-image], which provides the full set of buildpacks required to build a native image application.

## Behavior

This buildpack will participate if one the following conditions are met:

* `$BP_NATIVE_IMAGE` is set.
*  An upstream buildpack requests `native-image-application` in the build plan.

The buildpack will do the following:

* Requests that the Native Image builder be installed by requiring `native-image-builder` in the build plan.
* If `$BP_BINARY_COMPRESSION_METHOD` is set to `upx`, requests that UPX be installed by requiring `upx` in the buildplan.
* Uses `native-image` a to build a GraalVM native image and removes existing bytecode.
* Uses `$BP_BINARY_COMPRESSION_METHOD` if set to `upx` or `gzexe` to compress the native image.

## Configuration
| Environment Variable               | Description                                                                                   |
| ---------------------------------- | --------------------------------------------------------------------------------------------- |
| `$BP_NATIVE_IMAGE`                 | Whether to build a native image from the application.  Defaults to false.                     |
| `$BP_NATIVE_IMAGE_BUILD_ARGUMENTS` | Arguments to pass to directly to the `native-image` command. These arguments must be valid and correctly formed or the `native-image` command will fail. |
| `$BP_BINARY_COMPRESSION_METHOD`    | Compression mechanism used to reduce binary size. Options: `none` (default), `upx` or `gzexe` |

### Compression Caveats

1. Using `gzexe` if you intend to run your application on the Paketo Tiny image is not currently supported. The `gzexe` utility will compress your executable into what is a shell script, which executes and extracts the actual binary to a temp location. This process requires `/bin/sh` and that is not in the Tiny image. If you try using `gzexe` with the Tiny stack, it'll build OK but fail to run saying a file is missing.

2. Using `upx` will create a compressed executable that fails to run on M1 Macs. There is at the time of writing a bug in the emulation layer used by Docker on M1 Macs that is triggered when you try to run amd64 executable that has been compressed using `upx`. This is a known issue and will hopefully be patched in a future release.

## License
This buildpack is released under version 2.0 of the [Apache License][a].

[a]: http://www.apache.org/licenses/LICENSE-2.0
[native-image]: https://www.graalvm.org/reference-manual/native-image/
[bp/java-native-image]: https://github.com/paketo-buildpacks/java-native-image

