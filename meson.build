project('forpy', 'fortran')

# gfortran is cutting of free format source code, we should disable this here
fc = meson.get_compiler('fortran')
if fc.get_id() == 'gcc'
  add_project_arguments('-ffree-line-length-none', language: 'fortran')
endif

# forpy needs fypp as preprocessor for its code to allow template-like programming
# note that the generated source is preprocessed again
fypp = find_program('fypp', 'fypp.py', required: true)
fypp_gen = generator(fypp, arguments: '@INPUT@', capture: true,
                     output: '@BASENAME@.F90')

py_version = get_option('PY_VERSION')
if py_version == 3
  python_dep = dependency('python3')
elif py_version == 2
  python_dep = dependency('python2')
else
  error('Unknown python version')
endif

forpy_source = fypp_gen.process('forpy_mod.fypp')

forpy = library(meson.project_name(), forpy_source,
                dependencies: python_dep, pic: true)

forpy_dep = declare_dependency(link_with: forpy)

# testsuite for forpy
incdir = include_directories('tests')

testmod = ['tests/unittest_mod.F90', 'tests/forpy_tests_common_mod.F90']
testlib = static_library('unittest_mod', testmod, link_with: forpy)

test('basics test',
     executable('test_basics',
                ['tests/test_basics.F90', 'tests/test_basics_mod.F90'],
                dependencies: python_dep,
                include_directories: incdir,
                link_with: [testlib, forpy]))

test('cast test',
     executable('test_cast',
                ['tests/test_cast.F90', 'tests/test_cast_mod.F90'],
                dependencies: python_dep, include_directories: incdir,
                link_with: [testlib, forpy]))

test('datastructures test',
     executable('test_datastructures',
                ['tests/test_datastructures.F90', 'tests/test_datastructures_mod.F90'],
                dependencies: python_dep, include_directories: incdir,
                link_with: [testlib, forpy]))

# meson cannot find dependencies correctly from generated sources
# we add the dependency manually by creating an archive and link it in as dependency
ndarray_source = fypp_gen.process('tests/test_ndarray.fypp')
ndarray_mod_source = fypp_gen.process('tests/test_ndarray_mod.fypp')
ndarray_mod_lib = static_library('ndarray_mod', ndarray_mod_source,
                                 include_directories: incdir,
                                 link_with: [testlib, forpy])

test('ndarray test',
     executable('test_ndarray',
                sources: [ndarray_source],
                dependencies: python_dep, include_directories: incdir,
                link_with: [ndarray_mod_lib, testlib, forpy]))
