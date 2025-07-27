project(
  'libpng',
  'c',
  version: '1.6.50',
  license: 'libpng-2.0',
  meson_version: '>=0.54.0',
)

png_versions = meson.project_version().split('.')
png_major_version = png_versions[0]
png_minor_version = png_versions[1]
png_micro_version = png_versions[2]
png_libname = 'png@0@@1@'.format(png_major_version, png_minor_version)
png_libversion = '@0@@1@.@2@.0'.format(
  png_major_version,
  png_minor_version,
  png_micro_version,
)

cc = meson.get_compiler('c')

include = include_directories('.')

c_args = []

if host_machine.system() == 'windows'
  add_project_arguments(
    '-D_CRT_NONSTDC_NO_DEPRECATE',
    '-D_CRT_SECURE_NO_DEPRECATE',
    language: 'c',
  )
  if get_option('default_library') != 'static'
    c_args += '-DPNG_BUILD_DLL'
  endif
endif

zlib_dep = dependency('zlib')
libm = cc.find_library(
  'm',
  required: false,
)

libpng_deps = [zlib_dep, libm]

png_src = files(
  'png.c',
  'pngerror.c',
  'pngget.c',
  'pngmem.c',
  'pngpread.c',
  'pngread.c',
  'pngrio.c',
  'pngrtran.c',
  'pngrutil.c',
  'pngset.c',
  'pngtrans.c',
  'pngwio.c',
  'pngwrite.c',
  'pngwtran.c',
  'pngwutil.c',
)

if host_machine.cpu_family() == 'aarch64' or cc.get_define('__ARM_NEON').strip() != ''
  png_src += files(
    'arm/arm_init.c',
    'arm/filter_neon_intrinsics.c',
    'arm/palette_neon_intrinsics.c',
  )
  c_args += '-DPNG_ARM_NEON_OPT=2'
elif host_machine.cpu_family() in ['x86', 'x86_64']
  png_src += files('intel/filter_sse2_intrinsics.c', 'intel/intel_init.c')
  c_args += '-DPNG_INTEL_SSE_OPT=1'
elif host_machine.cpu_family() in ['ppc64', 'ppc64le', 'powerpc64le'] and host_machine.endian() == 'little'
  png_src += files('powerpc/filter_vsx_intrinsics.c', 'powerpc/powerpc_init.c')
  c_args += '-DPNG_POWERPC_VSX_OPT=2'
elif host_machine.cpu_family() in ['riscv32', 'riscv64']
  # As of 1.6.49, upstream CMake defaults RVV to off, while Autotools
  # defaults to on if the compiler supports it.  libpng also supports
  # autodetection at runtime (on Linux only, with substantial skepticism in
  # comments and documentation) and runtime enablement via the libpng API
  # (only on Autotools, which doesn't compile it in by default).  It's not
  # clear that RVV is widely supported by hardware.  Avoid the whole mess
  # for now.
  c_args += '-DPNG_RISCV_RVV_OPT=0'
endif

libpng = library(
  png_libname,
  png_src,
  version: png_libversion,
  dependencies: libpng_deps,
  c_args: c_args,
  install: true,
)

pngincsubdir = 'lib' + png_libname
pngincludedir = get_option('includedir') / pngincsubdir

pkglibconf = configure_file(
  input: 'scripts/pnglibconf.h.prebuilt',
  output: 'pnglibconf.h',
  copy: true,
)

install_headers(
  'png.h',
  'pngconf.h',
  pkglibconf,
  subdir: pngincsubdir,
)

libpng_dep = declare_dependency(
  include_directories: include,
  link_with: libpng,
  dependencies: libpng_deps,
)
# Keep the older dependency name for backwards-compat with old-style
# dependency(..., fallback ['libpng': 'png_dep'])
png_dep = libpng_dep

pkgconfig = import('pkgconfig')

pkgconfig.generate(
  libpng,
  name: 'libpng',
  description: 'Loads and saves PNG files',
  subdirs: pngincsubdir,
)

libpng_pkgconf_alias_names = ['lib' + png_libname]

foreach libpng_pkgconf_alias_name : libpng_pkgconf_alias_names
  if meson.version().version_compare('>=0.61')
    install_symlink(
      libpng_pkgconf_alias_name + '.pc',
      install_dir: get_option('libdir') / 'pkgconfig',
      pointing_to: 'libpng.pc',
    )
  else
    # install_symlink() is unavailable on Meson before 0.61.0, using dangerous workaround
    # TODO: remove this branch in future

    message(
      'Due to old Meson release, ' + libpng_pkgconf_alias_name + '.pc' + ' will be generated as standalone pkgconfig file instead of symbolic link to libpng.pc. This workaround is dangerous as it is possible to produce differ configs',
    )

    # Using positional library object more than once leads to warning which we avoid to pass CI
    # ('libpng| WARNING: Already generated a pkg-config file for png16')
    pkgconfig.generate(
      name: 'libpng',
      description: 'Loads and saves PNG files',
      filebase: libpng_pkgconf_alias_name,
      libraries_private: [libm],
      libraries: ['-L${libdir}', '-l' + libpng.name()],
      requires_private: [zlib_dep],
      subdirs: pngincsubdir,
      install_dir: get_option('libdir') / 'pkgconfig',
    )
  endif
endforeach

png_test = executable(
  'pngtest',
  'pngtest.c',
  dependencies: libpng_dep,
)
test(
  'pngtest',
  png_test,
  args: files('pngtest.png'),
)
