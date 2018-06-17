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
  name = 'musl-crt1',
  srcs = ['crt/crt1.c'],
  linker_flags = [
    '-nostdlib'
  ],
  compiler_flags = [
    '-nostdinc',
    '-ffreestanding',
    '-ffunction-sections',
    '-fdata-sections'
  ],
  preprocessor_flags = [
    '-DCRT'
  ],
  headers = subdir_glob([
    ('arch/'+arch, '**/*.h'),
    ('include', '**/*.h')
  ])
)

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
      #('ldso', '**/*.c'), 
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
    'src/*/x86_64/*.s',
    'src/**/*.c', 
  ], excludes = glob([
      'src/*/*/**/*.c',
    ]) + [ x.replace('/'+arch,'').replace('.s','.c') for x in glob(['src/*/x86_64/*.s'])]
  ),
  platform_srcs = [
    ('^linux.*', linux_srcs),
    ('.*', linux_srcs),
  ],
  preprocessor_flags = [
    '-D_XOPEN_SOURCE=700'
  ],
  linker_flags=["-nostdlib" ],
  compiler_flags = [
    '-std=c99',
    '-nostdinc',
    '-nostdlib',
    '-nodefaultlibs',
    '-fno-stack-protector', # only needed for .s files
    '-fno-unwind-tables',
    #'-fno-tree-loop-distribute-patterns', # only needed for memove.c for gcc
    '-fno-asynchronous-unwind-tables',
    '-fomit-frame-pointer',
    '-ffreestanding',
    '-ffunction-sections',
    '-fdata-sections',
    '-fPIC', # required for clang / when link_style=shared then buck adds automatically 
    #'-fexcess-precision=standard', #not required
    #'-frounding-math', #not required
    '-Os', '-O3' # not required
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
    ':musl-crt1', # different versions of crt available
    ':musl-internal',
  ], 
  visibility = [
    'PUBLIC', 
  ], 
)
