def merge_dicts(x, y):
  z = x.copy()
  z.update(y)
  return z

# We call CMake to generate the config file
# TODO: Port the logic in the CMake files
genrule(
  name = 'pg_config',
  out = 'pg_config.h',
  srcs = glob([
    'configure',
    '*.in',
    '*.info',
    '*.txt',
    'Makefile',
    'cmake/**/*.cmake',
    'cmake/**/*.c',
    'src/**/Makefile',
    'src/**/*.c',
    'src/**/*.h',
    'src/**/*.in',
    'src/**/*.txt',
  ]),
  cmd = 'cmake -Wno-dev $SRCDIR && cp src/include/pg_config.h $OUT',
)

genrule(
  name = 'errcodes',
  out = 'errcodes.h',
  srcs = [
    'src/backend/utils/generate-errcodes.pl',
    'src/backend/utils/errcodes.txt',
  ],
  cmd = 'perl $SRCDIR/src/backend/utils/generate-errcodes.pl $SRCDIR/src/backend/utils/errcodes.txt > $OUT',
)

PG_CONFIG_PATHS = '''#define PGBINDIR \\"bin\\"
#define PGSHAREDIR \\"share/libpq\\"
#define SYSCONFDIR \\"etc\\"
#define INCLUDEDIR \\"include\\"
#define PKGINCLUDEDIR \\"include\\"
#define INCLUDEDIRSERVER \\"include\\"
#define LIBDIR \\"lib\\"
#define PKGLIBDIR \\"share/pgklib\\"
#define LOCALEDIR \\"share/locale\\"
#define DOCDIR \\"share/libpq/doc\\"
#define HTMLDIR \\"share/libpq/doc/html\\"
#define MANDIR \\"share/libpq/doc/man\\"
'''

genrule(
  name = 'pg_config_paths',
  out = 'pg_config_paths.h',
  cmd = 'echo "' + PG_CONFIG_PATHS + '" > $OUT',
)

macos_exported_headers = {
  'pg_config_os.h': 'src/include/port/darwin.h',
}

windows_srcs = glob([
  'src/interfaces/libpq/*win32*.c',
])

platform_srcs = windows_srcs

cxx_library(
  name = 'pq',
  header_namespace = '',
  exported_headers = merge_dicts(subdir_glob([
    ('src/include', '**/*.h'),
  ]), {
    'pg_config.h': ':pg_config',
    'pg_config_paths.h': ':pg_config_paths',
    'utils/errcodes.h': ':errcodes',
  }),
  exported_platform_headers = [
    ('default', macos_exported_headers),
    ('^macos.*', macos_exported_headers),
  ],
  srcs = [
    'src/interfaces/libpq/fe-auth.c',
    'src/interfaces/libpq/fe-connect.c',
    'src/interfaces/libpq/fe-exec.c',
    'src/interfaces/libpq/fe-misc.c',
    'src/interfaces/libpq/fe-print.c',
    'src/interfaces/libpq/fe-lobj.c',
    'src/interfaces/libpq/fe-protocol2.c',
    'src/interfaces/libpq/fe-protocol3.c',
    'src/interfaces/libpq/pqexpbuffer.c',
    'src/interfaces/libpq/pqsignal.c',
    'src/interfaces/libpq/fe-secure.c',
    'src/interfaces/libpq/libpq-events.c',
    'src/interfaces/libpq/chklocale.c',
    'src/interfaces/libpq/inet_net_ntop.c',
    'src/interfaces/libpq/noblock.c',
    'src/interfaces/libpq/pgstrcasecmp.c',
    'src/interfaces/libpq/thread.c',
    'src/interfaces/libpq/ip.c',
    'src/interfaces/libpq/md5.c',
    'src/interfaces/libpq/encnames.c',
    'src/interfaces/libpq/wchar.c',
  ],
  platform_srcs = [
    ('^windows.*', windows_srcs),
  ],
  visibility = [
    'PUBLIC',
  ],
)
