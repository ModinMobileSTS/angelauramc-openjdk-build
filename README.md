# angelauramc-openjdk-build

**This branch is for OpenJDK 8.**

Based on [Java for Android](http://openjdk.java.net/projects/mobile/android.html) and [the PojavLauncher variant](https://github.com/PojavLauncherTeam/android-openjdk-build-multiarch)

## Building 

The Android AArch64 build applies
`patches/jdk8u_android_aarch64_explicit_null_checks.diff` and
`patches/jdk8u_android_aarch64_c1_explicit_null_checks.diff`, followed by
`patches/jdk8u_android_aarch64_interpreter_explicit_null_checks.diff`.
The first runtime layer makes HotSpot C2 keep its Java null checks explicit instead
of turning them into implicit SIGSEGV traps. The second layer extends the same
signal-free behavior to C1 managed-object field and array loads/stores, including
the direct C1 null-check operation, by branching on the object register to the
normal Runtime1 NPE/deoptimization path. Unsafe/native-address accesses retain
their existing semantics. Java exception behavior is intended to remain unchanged;
this is a runtime build change and does not require or expose a
`-XX:-ImplicitNullChecks` flag. The C1 layer is still a test-only candidate until
device lifecycle, fallback, crash-preservation, correctness, and performance gates
pass. The third layer keeps AArch64 interpreter template null checks on the
interpreter NPE entry, adds explicit checks to C1 locking and inline-cache
receivers, method-handle dispatch, the C2-to-interpreter and JNI receiver
adapters, and vtable/itable receiver stubs. Compiler/adapter stubs use the
normal call-site NPE entry; native and Unsafe address paths are not changed.
This layer is also test-only pending the same device validation. The older universal Android
portability patch also has a current-context companion,
`patches/jdk8u_android_flags_fix.diff`, because its `flags.m4` C++11 hunk no longer
matches the pinned `jdk8u482` source; the build script applies that companion
explicitly and verifies the resulting setting.

### Setup
#### Android
- Download Android NDK r10e from https://developer.android.com/ndk/downloads/older_releases.html and place it in this directory (Can't automatically download because of EULA)
- **Warning**: Do not attempt to build use newer or older NDK, it will lead to compilation errors.

#### iOS
- You should get latest Xcode (tested with Xcode 12).

### Platform and architecture specific environment variables
<table>
      <thead>
        <tr>
          <th></th>
          <th align="center" colspan="7">Environment variables</th>
        </tr>
        <tr>
          <th>Platform - Architecture</th>
          <th align="center">TARGET</th>
          <th align="center">TARGET_JDK</th>
        </tr>
      </thead>
      <tbody>
        <tr>
          <td>Android - armv8/aarch64</td>
          <td align="center">aarch64-linux-android</td>
          <td align="center">aarch64</td>
        </tr>
        <tr>
          <td>Android - armv7/aarch32</td>
          <td align="center">arm-linux-androideabi</td>
          <td align="center">arm</td>
        </tr>
        <tr>
          <td>Android - x86/i686</td>
          <td align="center">i686-linux-android</td>
          <td align="center">x86</td>
        </tr>
        <tr>
          <td>Android - x86_64/amd64</td>
          <td align="center">x86_64-linux-android</td>
          <td align="center">x86_64</td>
        </tr>
        <tr>
          <td>iOS/iPadOS - armv8/aarch64</td>
          <td align="center">aarch64-macos-ios</td>
          <td align="center">aarch64</td>
        </tr>
      </tbody>
	</table>

### Run in this directory:
```
export BUILD_IOS=1 # only when targeting iOS, default is 0 (target Android)

export BUILD_FREETYPE_VERSION=[2.6.2/.../2.10.4] # default: 2.10.4
export JDK_DEBUG_LEVEL=[release/fastdebug/debug] # default: release
export JVM_VARIANTS=[client/server] # default: client (aarch32), server (other architectures)

# Setup NDK, run once (Android only)
./extractndk.sh
./maketoolchain.sh

# Get CUPS, Freetype and build Freetype
./getlibs.sh
./buildlibs.sh

# Clone JDK, run once
./clonejdk.sh

# Configure JDK and build, if no configuration is changed, run makejdkwithoutconfigure.sh instead
./buildjdk.sh

# Pack the built JDK
./removejdkdebuginfo.sh
./tarjdk.sh
```
