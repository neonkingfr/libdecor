project('libdecor', 'c',
  version: '0.1.0',
  meson_version: '>= 0.47.0',
  license: 'MIT',
  default_options: [
    'c_std=c99',
  ],
)

version_array = meson.project_version().split('.')
libdecor_major_version = version_array[0].to_int()
libdecor_minor_version = version_array[1].to_int()
libdecor_micro_version = version_array[2].to_int()

libdecor_api_version_string = '@0@'.format(libdecor_major_version)

libdecor_interface_age = libdecor_micro_version
libdecor_binary_age = 100 * libdecor_minor_version + libdecor_micro_version

# Maintain compatibility with libtool versioning
libdecor_soversion = 0
current = libdecor_binary_age - libdecor_interface_age
revision = libdecor_interface_age
libdecor_libversion = '@0@.@1@.@2@'.format(libdecor_soversion, current, revision)

plugin_api_version = 1
plugin_api_version_string = '@0@'.format(plugin_api_version)

libdecor_name = 'decor-' + libdecor_api_version_string
libdecor_full_name = 'libdecor-' + libdecor_api_version_string

cc = meson.get_compiler('c')

add_project_arguments('-D_GNU_SOURCE', language: 'c')

add_project_arguments(cc.get_supported_arguments([
  '-Wno-unused-parameter',
  '-Wno-missing-field-initializers',
  ]), language: 'c')

wayland_client_req = '>= 1.18'
wayland_protocols_req = '>= 1.15'
dbus_req = '>= 1.0'

wayland_client_dep = dependency('wayland-client', version: wayland_client_req)
wayland_protocols_dep = dependency('wayland-protocols', version: wayland_protocols_req)
dl_dep = cc.find_library('dl', required: true)

dbus_dep = dependency('dbus-1', version: dbus_req, required: get_option('dbus'))

# list of (function, prefix)
required_functions = [
  ['memfd_create', '#define _GNU_SOURCE\n#include <sys/mman.h>'],
  ['posix_fallocate', '#define _GNU_SOURCE\n#include <fcntl.h>'],
  ['asprintf', '#define _GNU_SOURCE\n#include <stdio.h>'],
]
foreach function : required_functions
  if not cc.has_function(function[0], prefix: function[1])
    error('Required function ' + function[0] + ' missing')
  endif
endforeach

prefix = get_option('prefix')
libdir = join_paths(prefix, get_option('libdir'))

pkgname = meson.project_name()
pkglibdir = join_paths(libdir, pkgname)

plugindir = join_paths(pkglibdir, 'plugins-@0@'.format(plugin_api_version_string))

libdecor_c_args = [
  '-fvisibility=hidden',
]

cdata = configuration_data()
cdata.set_quoted('VERSION', meson.project_version())
cdata.set_quoted('LIBDECOR_PLUGIN_DIR', plugindir)
cdata.set('LIBDECOR_PLUGIN_API_VERSION', plugin_api_version)
if dbus_dep.found()
  cdata.set('HAS_DBUS', true)
endif

config_h = configure_file(
  input: 'config.h.meson',
  output: 'config.h',
  configuration: cdata
)

top_includepath = include_directories('.')

subdir('src')

if get_option('demo')
  subdir('demo')
endif
