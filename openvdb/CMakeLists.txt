cmake_minimum_required(VERSION 3.24)

project(OpenVDB)

if(MSVC)
    option(USE_MSVC_RUNTIME_LIBRARY_DLL "Bulid as MultithreadedDLL" ON)
endif()

# specify msvc runtime library
if(MSVC)
    if(USE_MSVC_RUNTIME_LIBRARY_DLL)
        cmake_policy(SET CMP0091 NEW)
        set(CMAKE_MSVC_RUNTIME_LIBRARY "MultiThreaded$<$<CONFIG:Debug>:Debug>DLL")
        add_compile_definitions(OPENVDB_DLL)
        add_compile_definitions(OPENVDB_PRIVATE)
    else()
        cmake_policy(SET CMP0091 NEW)
        set(CMAKE_MSVC_RUNTIME_LIBRARY "MultiThreaded$<$<CONFIG:Debug>:Debug>")
    endif()

    message(${CMAKE_MSVC_RUNTIME_LIBRARY})
endif()

set(HEADERS
    Exceptions.h 
    Grid.h 
    io/Archive.h 
    io/Compression.h 
    io/File.h 
    io/GridDescriptor.h 
    io/Queue.h 
    io/Stream.h 
    math/BBox.h 
    math/Coord.h 
    math/FiniteDifference.h 
    math/Hermite.h 
    math/LegacyFrustum.h 
    math/Maps.h 
    math/Mat.h 
    math/Mat3.h 
    math/Mat4.h 
    math/Math.h 
    math/Operators.h 
    math/Proximity.h 
    math/QuantizedUnitVec.h 
    math/Quat.h 
    math/Ray.h 
    math/Stats.h 
    math/Stencils.h 
    math/Transform.h
    math/Tuple.h
    math/Vec2.h 
    math/Vec3.h 
    math/Vec4.h 
    Metadata.h 
    metadata/Metadata.h 
    metadata/MetaMap.h 
    metadata/StringMetadata.h 
    openvdb.h 
    Platform.h 
    PlatformConfig.h 
    tools/Composite.h 
    tools/Dense.h 
    tools/Filter.h 
    tools/GridOperators.h 
    tools/GridTransformer.h 
    tools/Interpolation.h 
    tools/LevelSetAdvect.h 
    tools/LevelSetFilter.h 
    tools/LevelSetFracture.h 
    tools/LevelSetMeasure.h 
    tools/LevelSetMorph.h 
    tools/LevelSetRebuild.h 
    tools/LevelSetSphere.h 
    tools/LevelSetTracker.h 
    tools/LevelSetUtil.h 
    tools/MeshToVolume.h 
    tools/Morphology.h 
    tools/ParticlesToLevelSet.h 
    tools/PointAdvect.h 
    tools/PointScatter.h 
    tools/RayIntersector.h 
    tools/RayTracer.h 
    tools/Statistics.h 
    tools/ValueTransformer.h 
    tools/VectorTransformer.h 
    tools/VolumeToMesh.h 
    tools/VolumeToSpheres.h 
    tree/InternalNode.h 
    tree/Iterator.h 
    tree/LeafManager.h 
    tree/LeafNode.h 
    tree/LeafNodeBool.h 
    tree/NodeUnion.h 
    tree/RootNode.h 
    tree/Tree.h 
    tree/TreeIterator.h 
    tree/Util.h 
    tree/ValueAccessor.h 
    Types.h 
    util/Formats.h 
    util/logging.h 
    util/MapsUtil.h 
    util/Name.h 
    util/NodeMasks.h 
    util/NullInterrupter.h 
    util/Util.h 
    version.h)

set(SRC
    Grid.cc 
    io/Archive.cc 
    io/Compression.cc 
    io/File.cc 
    io/GridDescriptor.cc 
    io/Queue.cc 
    io/Stream.cc 
    math/Hermite.cc 
    math/Maps.cc 
    math/Proximity.cc 
    math/QuantizedUnitVec.cc 
    math/Transform.cc 
    metadata/Metadata.cc 
    metadata/MetaMap.cc 
    openvdb.cc 
    Platform.cc 
    util/Formats.cc 
    util/Util.cc )

# specify project filters
foreach(FILE ${SRC} ${HEADERS}) 
    # Get the directory of the source file
    get_filename_component(PARENT_DIR "${FILE}" DIRECTORY)

    # Remove common directory prefix to make the group
    string(REPLACE "${CMAKE_CURRENT_SOURCE_DIR}" "" GROUP "${PARENT_DIR}")

    # Make sure we are using windows slashes
    string(REPLACE "/" "\\" GROUP "${GROUP}")

    # Group into "Source Files" and "Header Files"
    if ("${FILE}" MATCHES ".*\\.cpp")
         set(GROUP "Files\\${GROUP}")
    elseif("${FILE}" MATCHES ".*\\.c")
         set(GROUP "Files\\${GROUP}")
    elseif("${FILE}" MATCHES ".*\\.cxx")
         set(GROUP "Files\\${GROUP}")
    elseif("${FILE}" MATCHES ".*\\.h")
         set(GROUP "Files\\${GROUP}")
    elseif("${FILE}" MATCHES ".*\\.mm")
        set(GROUP "Files\\${GROUP}")
    endif()

    source_group("${GROUP}" FILES "${FILE}")
endforeach()


find_package(Boost REQUIRED)
find_package(OpenEXR REQUIRED)
find_package(TBB REQUIRED)

add_library(openvdb STATIC ${SRC} ${HEADERS})

if(MSVC)
    target_compile_definitions(openvdb PRIVATE NOMINMAX ZLIBWINAPI)
endif()

target_include_directories(openvdb PRIVATE .. ${Boost_INCLUDE_DIRS} ${OpenEXR_INCLUDE_DIRS} ${TBB_INCLUDE_DIRS})
target_link_directories(openvdb PRIVATE ${Boost_LIBRARY_DIR} ${OpenEXR_LIBRARY_DIR} ${TBB_LIBRARY_DIR})

install(TARGETS openvdb 
        EXPORT openvdb
        ARCHIVE DESTINATION lib
        LIBRARY DESTINATION lib)
install(DIRECTORY "${CMAKE_SOURCE_DIR}/" # source directory
    DESTINATION "include/openvdb" # target directory
    FILES_MATCHING # install only matched files
    PATTERN "*.h") # select header files