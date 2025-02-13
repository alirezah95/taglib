import os
import sys

from SCons.Script import *


sys.tracebacklimit = 0

# Try to detect the host platform automatically.
# This is used if no `platform` argument is passed
if sys.platform.startswith('linux'):
    host_platform = 'linux'
elif sys.platform == 'darwin':
    host_platform = 'osx'
elif sys.platform == 'win32' or sys.platform == 'msys':
    host_platform = 'windows'
else:
    raise ValueError(
        'Could not detect host platform automatically, please specify with '
        'host_platform=<platform>'
    )


colors = {}
def no_verbose():
    if sys.stdout.isatty():
        colors["purple"] = "\033[95m"
        colors["blue"] = "\033[94m"
        colors["green"] = "\033[92m"
        colors["yellow"] = "\033[93m"
        colors["end"] = "\033[0m"
    else:
        colors["purple"] = ""
        colors["blue"] = ""
        colors["green"] = ""
        colors["yellow"] = ""
        colors["end"] = ""
    
    compile_shared_source_message = "{}Compiling shared ==> {}$SOURCE{}"\
        .format(colors["green"], colors["yellow"], colors["end"])
    link_shared_library_message = "{}Linking Shared Library ==> {}$TARGET{}"\
        .format(colors["blue"], colors["yellow"], colors["end"])
    
    platform_print_prefix = "{}[{} {}] ".format(colors["purple"], env['platform'],
        env['android_arch'] if env['platform'] == "android" else env['bits'])
    
    env["SHCCCOMSTR"] = platform_print_prefix + compile_shared_source_message
    env["SHCXXCOMSTR"] = platform_print_prefix + compile_shared_source_message
    env["ARCOMSTR"] = platform_print_prefix + link_shared_library_message
    env["RANLIBCOMSTR"] = platform_print_prefix + link_shared_library_message
    env["SHLINKCOMSTR"] = platform_print_prefix + link_shared_library_message
    env["LINKCOMSTR"] = platform_print_prefix + link_shared_library_message


options = Variables([], ARGUMENTS)

options.Add(EnumVariable(
    key='platform',
    help='Target Platform',
    default='none',
    allowed_values=['none', 'linux', 'android', 'windows', 'javascript'],
    ignorecase=2
))

options.Add(EnumVariable(
	key="bits",
	help="Target platform bits",
	default=64,
	allowed_values=["32", "64"]
))

options.Add(EnumVariable(
	key="target",
	help="release or debug target",
	default="release",
	allowed_values=["release", "debug"]
))

options.Add(PathVariable(
	key="ANDROID_NDK_ROOT",
	help="""Path to your Anroid NDK installation root. By default uses
		ANDROID_NDK_ROOT from your environment variablies.""",
	default=os.environ.get("ANDROID_NDK_ROOT", None),
	validator=PathVariable.PathIsDir
))

options.Add(EnumVariable(
    key='android_arch',
    help='Target android architecture',
    default='armv7',
    allowed_values=['armv7', 'arm64v8', 'x86', 'x86_64']
))

options.Add(
    key='android_api_level',
    help='Target Android API level',
    default='18' if ARGUMENTS.get("android_arch", 'armv7') in [
        'armv7', 'x86'] else '21'
)
options.Add(
    key='library_dir',
    help='Path to desired out put foldel.',
    default='../Lib/'
)

env = Environment(CXXFLAGS="-std=c++17", sources=[])
options.Update(env=env)
Help(options.GenerateHelpText(env))

if ARGUMENTS.get("VERBOSE") != "1":
    no_verbose()

var_dir = 'taglib/build/obj/' + env['platform']
if env['platform'] == 'android':
	var_dir += '/' + env['android_arch']
elif env['platform']:
	var_dir += '/' + env['bits']

VariantDir(var_dir, 'taglib', duplicate=0)

if env['platform'] == 'none':
    raise ValueError("""\n\tYou must specify target platform. Specify it by 
        platform=<platform>. <platform> can only be 'linux', 'android'.""")

if env['platform'] == 'linux':
    print('*** Building for Linux ***')
    if env['target'] == 'debug':
        env.Append(CCFLAGS = ['-g3','-Og'])
    elif env['target'] == 'release':
        env.Append(CCFLAGS = ['-O3'])
        # env.Append(LINKFLAGS = ['-s'])

    if env['bits'] == '64':
        env.Append(CCFLAGS=['-m64'])
        env.Append(LINKFLAGS=['-m64'])
    elif env['bits'] == '32':
        env.Append(CCFLAGS=['-m32'])
        env.Append(LINKFLAGS=['-m32'])
elif env['platform'] == 'windows':
    print('*** Building for Windows ***')
    if host_platform == 'windows':
        # MSVC
        env.Append(LINKFLAGS=['/WX'])
        if env['target'] == 'debug':
            env.Append(CCFLAGS=['/Z7', '/Od', '/EHsc', '/D_DEBUG', '/MDd'])
        elif env['target'] == 'release':
            env.Append(CCFLAGS=['/O2', '/EHsc', '/DNDEBUG', '/MD'])

    elif host_platform == 'linux' or host_platform == 'osx':
        # Cross-compilation using MinGW
        if env['bits'] == '64':
            env['CC'] = 'x86_64-w64-mingw32-gcc'
            env['CXX'] = 'x86_64-w64-mingw32-g++'
            env['AR'] = "x86_64-w64-mingw32-ar"
            env['RANLIB'] = "x86_64-w64-mingw32-ranlib"
            env['LINK'] = "x86_64-w64-mingw32-g++"
        elif env['bits'] == '32':
            env['CC'] = 'i686-w64-mingw32-gcc'
            env['CXX'] = 'i686-w64-mingw32-g++'
            env['AR'] = "i686-w64-mingw32-ar"
            env['RANLIB'] = "i686-w64-mingw32-ranlib"
            env['LINK'] = "i686-w64-mingw32-g++"
        
        if env['target'] == 'debug':
            env.Append(CCFLAGS = ['-g3','-Og'])
        elif env['target'] == 'release':
            env.Append(CCFLAGS = ['-O3'])
            env.Append(LINKFLAGS = ['-flto'])
        
        env.Append(CCFLAGS=['-Wwrite-strings'])
        env.Append(CFLAGS=['-Wno-discarded-qualifiers'])
        env.Append(LINKFLAGS=[
            '--static',
            '-Wl,--no-undefined',
            '-static-libgcc',
            '-static-libstdc++',
        ])
elif env['platform'] == 'android':
    print('*** Building for Android ***')
    if 'ANDROID_NDK_ROOT' not in env:
        raise ValueError("""\n\tTo build for Android, ANDROID_NDK_ROOT must be
             defined. Please set ANDROID_NDK_ROOT to the root folder of your
             Android NDK installation.""")

    # Validate API level
    api_level = int(env['android_api_level'])
    if env['android_arch'] in ['x86_64', 'arm64v8'] and api_level < 21:
        print("""WARN: 64-bit Android architectures require an API level of at
             least 21; setting android_api_level=21""")
        env['android_api_level'] = '21'
        api_level = 21

    # Setup android toolchain
    if not env['ANDROID_NDK_ROOT'].endswith('/'):
        env['ANDROID_NDK_ROOT'] += '/'
    toolchain = env['ANDROID_NDK_ROOT'] + 'toolchains/llvm/prebuilt/'
    if host_platform == 'windows':
        toolchain += 'windows'
        import platform as pltfm
        if pltfm.machine().endswith('64'):
            toolchain += '-x86_64'
    elif host_platform == "linux":
        toolchain += "linux-x86_64/"

    # Get architecture info
    arch_info_table = {
        "armv7": {
            "march": "armv7-a", "compiler_path": "armv7a-linux-androideabi"
        },
        "arm64v8": {
            "march": "armv8-a", "compiler_path": "aarch64-linux-android"
        },
        "x86": {
            "march": "i686", "compiler_path": "i686-linux-android"
        },
        "x86_64": {
            "march": "x86-64", "compiler_path": "x86_64-linux-android"
        }
    }

    toolchain += ('bin/' + arch_info_table[env['android_arch']]['compiler_path']
                  + env['android_api_level'])

    env['CC'] = toolchain + '-clang'
    env['CXX'] = toolchain + '-clang++'
elif env['platform'] == 'javascript':
    print('*** Building for Javascript ***')
    print('*** Building for Javascript ***')
    env["ENV"] = os.environ
    env["CC"] = "emcc"
    env["CXX"] = "em++"
    env["AR"] = "emar"
    env["RANLIB"] = "emranlib"
    env.Append(CPPFLAGS=["-s", "SIDE_MODULE=1", '-s', 'ASSERTIONS=1'])
    env.Append(LINKFLAGS=["-s", "SIDE_MODULE=1", '-s', 'ASSERTIONS=1'])
    env["SHOBJSUFFIX"] = ".bc"
    env["SHLIBSUFFIX"] = ".wasm"
    # Use TempFileMunge since some AR invocations are too long for cmd.exe.
    # Use POSIX-style paths, required with TempFileMunge.
    env["ARCOM_POSIX"] = env["ARCOM"].replace("$TARGET", "$TARGET.posix").replace("$SOURCES", "$SOURCES.posix")
    env["ARCOM"] = "${TEMPFILE(ARCOM_POSIX)}"

    # All intermediate files are just LLVM bitcode.
    env["OBJPREFIX"] = ""
    env["OBJSUFFIX"] = ".bc"
    env["PROGPREFIX"] = ""
    # Program() output consists of multiple files, so specify suffixes manually at builder.
    env["PROGSUFFIX"] = ""
    env["LIBPREFIX"] = "lib"
    env["LIBSUFFIX"] = ".a"
    env["LIBPREFIXES"] = ["$LIBPREFIX"]
    env["LIBSUFFIXES"] = ["$LIBSUFFIX"]
    env.Replace(SHLINKFLAGS='$LINKFLAGS')
    env.Replace(SHLINKFLAGS='$LINKFLAGS')

output_file = env['library_dir'] + env['platform'].capitalize() + "/libtag"
if env['platform'] == 'android':
    output_file += env['android_arch']
else:
    output_file += env['bits']

env['sources'] += Glob(var_dir + "/*.cpp")

Export("env")

SConscript(
	[
		var_dir + "/ape/SConscript",
		var_dir + "/asf/SConscript",
		var_dir + "/flac/SConscript",
		var_dir + "/it/SConscript",
		var_dir + "/mod/SConscript",
		var_dir + "/mp4/SConscript",
		var_dir + "/mpc/SConscript",
		var_dir + "/mpeg/SConscript",
		var_dir + "/ogg/SConscript",
		var_dir + "/riff/SConscript",
		var_dir + "/s3m/SConscript",
		var_dir + "/toolkit/SConscript",
		var_dir + "/trueaudio/SConscript",
		var_dir + "/wavpack/SConscript",
		var_dir + "/xm/SConscript",
	]
)

# Getting all the directories in taglib tree
for root, d_name, f_name in os.walk("./taglib"):
    if root.find("build") == -1 and root.find("CMakeFiles") == -1:
        env.Append(CPPPATH=root.replace("./", "") + ":")

env.Append(CPPPATH="./:")
env.Append(CPPPATH="3rdparty:")

TagLib = env.SharedLibrary(
	target=output_file,
	source=env["sources"],
    LIBS=['z']
)

Default(TagLib)
