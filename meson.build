project('forpy', 'fortran', version: '0.0')

# there is a bug in fypp that prevents using it with python3
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
                version: meson.project_version(),
                dependencies: python_dep, pic: true)

forpy_dep = declare_dependency(link_with: forpy)

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

ndarray_source = fypp_gen.process('tests/test_ndarray.fypp')
ndarray_mod_source = fypp_gen.process('tests/test_ndarray_mod.fypp')

test('ndarray test',
     executable('test_ndarray',
                sources: [ndarray_mod_source, ndarray_source],
                dependencies: python_dep, include_directories: incdir,
                link_with: [testlib, forpy]))
