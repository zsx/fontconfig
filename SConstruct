# vim: ft=python expandtab
import os
import subprocess
from site_init import *

opts = Variables()
opts.Add(PathVariable('PREFIX', 'Installation prefix', os.path.expanduser('~/FOSS'), PathVariable.PathIsDirCreate))
opts.Add(BoolVariable('DEBUG', 'Build with Debugging information', 0))
opts.Add(PathVariable('PERL', 'Path to the executable perl', r'C:\Perl\bin\perl.exe', PathVariable.PathIsFile))
opts.Add(BoolVariable('WITH_OSMSVCRT', 'Link with the os supplied msvcrt.dll instead of the one supplied by the compiler (msvcr90.dll, for instance)', 0))

env = Environment(variables = opts,
                  ENV=os.environ, tools = ['default', GBuilder], PDB = 'libfontconfig.pdb')

Initialize(env)

env.Append(CFLAGS=env['DEBUG_CFLAGS'])
env.Append(CPPDEFINES=env['DEBUG_CPPDEFINES'])

if env['WITH_OSMSVCRT']:
    env['LIB_SUFFIX'] = '-1'
    env.Append(CPPDEFINES=['MSVCRT_COMPAT_IO'])

env_tmp = Environment(ENV=os.environ)
env_tmp.ParseConfig('pkg-config libxml-2.0 --libs')
LIBXML2_LIBS = env_tmp['LIBS']

env_tmp = Environment(ENV=os.environ)
env_tmp.ParseConfig('pkg-config freetype2 --libs')
LIBFREETYPE_LIBS = env_tmp['LIBS']

VERSION="2.7.2"
env['VERSION'] = VERSION
env['DOT_IN_SUBS'] = {'@VERSION@': VERSION,
                      '@prefix@': env['PREFIX'],
                      '@exec_prefix@': '${prefix}/bin',
                      '@libdir@': '${prefix}/lib',
                      '@includedir@': '${prefix}/include',
                      '@LIBXML2_LIBS@': " ".join(map(lambda x: '-l' + x, LIBXML2_LIBS)),
                      '@FREETYPE_LIBS@': " ".join(map(lambda x: '-l' + x, LIBFREETYPE_LIBS)),
                      '@ICONV_LIBS@': '-liconv',
                      '@CONFDIR@': '"%s"' % (env['PREFIX'].replace('\\', r'\\') + r'\\etc'),
                      '@FC_DEFAULT_FONTS@': '"%s"' % (env['ENV']['systemroot'].replace('\\', r'\\') + r'\\fonts')}

env.DotIn('fontconfig.pc', 'fontconfig.pc.in')
env.Alias('install', env.Install('$PREFIX/lib/pkgconfig', 'fontconfig.pc'))
env['DOT_IN_SUBS']['@PCS@'] = generate_file_element('fontconfig.pc', r'lib/pkgconfig', env)

env.DotIn('config.h', 'config.h.win32.in')
def generate_fonts_conf(target, source, env):
    fs = file(str(source[0]), 'r')
    c = fs.read()
    fs.close()
    c.replace('@FC_CACHEDIR@', (env['PREFIX'] + '\\etc') .replace('\\', '\\\\'))
    c.replace('@FC_DEFAULT_FONTS@', 'Arial')
    c.replace('@FC_FONTPATH@', (os.environ['SYSTEMROOT'] + '\\fonts').replace('\\', '\\\\'))
    c.replace('@PACKAGE@', 'FontConfig')
    c.replace('@VERSION@', env['VERSION'])
    ft = file(str(target[0]), 'w')
    ft.write(c)
    ft.close()

env.Command('fonts.conf', ['fonts.conf.in', 'SConstruct'], generate_fonts_conf)
env.Alias('install', env.Install('$PREFIX/etc', 'fonts.conf'))
env['DOT_IN_SUBS']['@CONFS@'] = generate_file_element('fonts.conf', r'etc', env)

env.Alias('install', env.Install('$PREFIX/include/fontconfig', ['fontconfig/fontconfig.h', 'fontconfig/fcfreetype.h', 'fontconfig/fcprivate.h']))
env['DOT_IN_SUBS']['@HEADERS@'] = generate_file_element(['fontconfig.h', 'fcfreetype.h', 'fcprivate.h'], r'include/fontconfig', env)

env.Append(CPPDEFINES=['HAVE_CONFIG_H'])

env.SConscript(['src/SConscript',
                'fc-arch/SConscript',
                'fc-glyphname/SConscript',
                'fc-lang/SConscript',
                'fc-case/SConscript'], exports='env')
