project(CFS_LC C)

include(CheckSourceCompiles)

# Determine if math library required for "isnan" and "isfinite"
check_source_compiles(
  C
  "#include <float.h>
   #include <math.h>  
   int main()
   {
     float FloatValue = 1.1f;
     if (isnan(FloatValue) || !isfinite(FloatValue))
        return 1;
     else
        return 0;
  }"
  HAS_MATH_LIB
)

# Add math library dependency if test code failed to compile and link
if (NOT HAS_MATH_LIB)
  message(STATUS "Adding math library dependency to lc app")
  link_libraries(m)
endif()

set(APP_SRC_FILES
  fsw/src/lc_custom.c
  fsw/src/lc_app.c
  fsw/src/lc_cmds.c
  fsw/src/lc_watch.c
  fsw/src/lc_action.c
  fsw/src/lc_utils.c
)

if (CFE_EDS_ENABLED)
  list(APPEND APP_SRC_FILES
    fsw/src/lc_eds_dispatch.c
  )
else()
  list(APPEND APP_SRC_FILES
    fsw/src/lc_dispatch.c
  )
endif()

# Create the app module
add_cfe_app(lc ${APP_SRC_FILES})

# This permits direct access to public headers in the fsw/inc directory
target_include_directories(lc PUBLIC fsw/inc)

set(APP_TABLE_FILES
  fsw/tables/lc_def_wdt.c
  fsw/tables/lc_def_adt.c
)

add_cfe_tables(lc ${APP_TABLE_FILES})

# If UT is enabled, then add the tests from the subdirectory
# Note that this is an app, and therefore does not provide
# stub functions, as other entities would not typically make
# direct function calls into this application.
if(ENABLE_UNIT_TESTS)
  add_subdirectory(unit-test)
endif()
