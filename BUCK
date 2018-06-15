def merge_dicts (xs):
  d = {}
  for x in xs: 
    d.update(x)
  return d

arch = 'x86_64'
# arch = 'i386'

def headers_for_arch(arch): 
  # Take the arch-specific headers with "generic" as a fallback
  return merge_dicts([
    subdir_glob([
      ('arch/generic', '**/*.h'), 
    ]), 
    subdir_glob([
      ('arch/' + arch, '**/*.h'), 
    ])
  ])

genrule(
  name = 'alltypes', 
  out = 'alltypes.h', 
  srcs = [
    'include/alltypes.h.in', 
    'arch/' + arch + '/bits/alltypes.h.in', 
    'tools/mkalltypes.sed', 
  ],
  cmd = ' '.join([
    'sed -f $SRCDIR/tools/mkalltypes.sed', 
    '$SRCDIR/arch/' + arch + '/bits/alltypes.h.in', 
    '$SRCDIR/include/alltypes.h.in > $OUT', 
  ]), 
)

genrule(
  name = 'syscall', 
  out = 'syscall.h', 
  srcs = [
    'arch/' + arch + '/bits/syscall.h.in', 
  ],
  cmd = 'cp $SRCS $OUT && sed -n -e s/__NR_/SYS_/p < $SRCS >> $OUT', 
)

genrule(
  name = 'version', 
  out = 'version.h', 
  srcs = [
    '.git', 
    'VERSION', 
    'tools/version.sh', 
  ],
  cmd = 'cd $SRCDIR && printf \'#define VERSION "%s"\n\' \$(sh $SRCDIR/tools/version.sh) > $OUT', 
)

linux_srcs = glob([
  'src/*/x86_64/**/*.c', 
  # 'src/*/x86_64/**/*.s', 
])

cxx_library(
  name = 'musl-internal', 
  header_namespace = '',
  exported_headers = {
    'bits/alltypes.h': ':alltypes', 
    'bits/syscall.h': ':syscall', 
  }, 
  headers = merge_dicts([
    subdir_glob([
      ('include', '**/*.h'), 
    ], excludes = [
      'src/internal/syscall.h',
    ]), 
    subdir_glob([
      ('src', '**/*.h'), 
      ('src/passwd', '**/*.h'), 
      ('src/ctype', '**/*.h'), 
      ('src/time', '**/*.h'), 
      ('src/internal', '**/*.h'), 
    ]), 
    headers_for_arch(arch), 
    {
      'version.h': ':version', 
      'bits/alltypes.h': ':alltypes', 
      'bits/syscall.h': ':syscall', 
    }, 
  ]), 
  srcs = glob([
    'src/**/*.c', 
  ], excludes = glob([
    'src/math/*/**/*.c', 
    'src/*/*/**/*.c', 
  ])), 
  platform_srcs = [
    ('^linux.*', linux_srcs), 
    ('.*', linux_srcs), 
  ], 
  preprocessor_flags = [
    '-D_XOPEN_SOURCE=700', 
  ],
  compiler_flags = [
    '-std=c11', 
    '-nostdinc', 
    '-ffreestanding', 
    '-fexcess-precision=standard', 
    '-frounding-math', 
    '-Os', 
    '-pipe', 
    '-fomit-frame-pointer', 
    '-fno-unwind-tables', 
    '-fno-asynchronous-unwind-tables', 
    '-ffunction-sections', 
    '-fdata-sections', 
    '-Werror=implicit-function-declaration', 
    '-Werror=implicit-int', 
    '-Werror=pointer-sign', 
    '-Werror=pointer-arith',
    '-fno-stack-protector', 
    '-shared', 
  ],
  visibility = [
    'PUBLIC', 
  ], 
)

prebuilt_cxx_library(
  name = 'musl', 
  header_namespace = '', 
  header_only = True, 
  exported_headers = subdir_glob([
    ('include', '**/*.h'), 
  ]), 
  deps = [
    ':musl-internal', 
  ], 
  visibility = [
    'PUBLIC', 
  ], 
)
