project('forpy', 'fortran')

# gfortran is cutting off free format source code, we should disable this here
fc = meson.get_compiler('fortran')
if fc.get_id() == 'gcc'
  add_project_arguments('-ffree-line-length-none', language: 'fortran')
endif

# forpy needs fypp as preprocessor to allow template-like programming
# note that the generated source is preprocessed again
fypp = find_program('fypp', 'fypp.py', required: true)

py_version = get_option('PY_VERSION')
if py_version == 3
  python_dep = dependency('python3')
elif py_version == 2
  python_dep = dependency('python2')
  add_project_arguments('-DPYTHON2', language: 'fortran')
else
  error('Unknown python version')
endif

if get_option('PY_DEBUG')
  add_project_arguments('-DPy_DEBUG', language: 'fortran')
endif

forpy_source = configure_file(input: 'forpy_mod.fypp', output: '@BASENAME@.F90',
                              command: [fypp, '@INPUT@', '@OUTPUT@'])

forpy = library(meson.project_name(), forpy_source,
                dependencies: python_dep, pic: true)

# carry python dependency from forpy along for wrap dependencies
forpy_dep = [declare_dependency(link_with: forpy), python_dep]

# testsuite for forpy
testdir = include_directories('tests')
testmod = ['tests/unittest_mod.F90', 'tests/forpy_tests_common_mod.F90']
testlib = static_library('unittest_mod', testmod, link_with: forpy)

test('basics test',
     executable('test_basics',
                ['tests/test_basics.F90', 'tests/test_basics_mod.F90'],
                dependencies: forpy_dep, include_directories: testdir,
                link_with: testlib))

test('cast test',
     executable('test_cast',
                ['tests/test_cast.F90', 'tests/test_cast_mod.F90'],
                dependencies: forpy_dep, include_directories: testdir,
                link_with: testlib))

test('datastructures test',
     executable('test_datastructures',
                ['tests/test_datastructures.F90', 'tests/test_datastructures_mod.F90'],
                dependencies: forpy_dep, include_directories: testdir,
                link_with: testlib))

ndarray_sources = []
foreach source : ['tests/test_ndarray.fypp', 'tests/test_ndarray_mod.fypp']
  ndarray_sources += configure_file(input: source, output: '@BASENAME@.F90',
                                    command: [fypp, '@INPUT@', '@OUTPUT@'])
endforeach

test('ndarray test',
     executable('test_ndarray',
                ndarray_sources,
                dependencies: forpy_dep, include_directories: testdir,
                link_with: testlib))
