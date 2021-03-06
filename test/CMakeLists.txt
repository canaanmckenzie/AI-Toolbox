cmake_minimum_required(VERSION 3.9) # CMP0069 NEW

function (AddTestGlobal name)
    set(exename Global_${name})
    add_executable(${exename}Tests ${name}Tests.cpp ${PROJECT_SOURCE_DIR}/src/Impl/Seeder.cpp)
    target_link_libraries(${exename}Tests ${Boost_UNIT_TEST_FRAMEWORK_LIBRARY} ${ARGN})
    add_test(NAME ${exename} WORKING_DIRECTORY ${CMAKE_RUNTIME_OUTPUT_DIRECTORY} COMMAND $<TARGET_FILE:${exename}Tests>)
    set_target_properties(${exename}Tests PROPERTIES INTERPROCEDURAL_OPTIMIZATION ${LTO_SUPPORTED})
endfunction (AddTestGlobal)

function (AddTestMDP name)
    set(exename MDP_${name})
    add_executable(${exename}Tests MDP/${name}Tests.cpp)
    target_link_libraries(${exename}Tests AIToolboxMDP ${Boost_UNIT_TEST_FRAMEWORK_LIBRARY} ${ARGN})
    add_test(NAME ${exename} WORKING_DIRECTORY ${CMAKE_RUNTIME_OUTPUT_DIRECTORY} COMMAND $<TARGET_FILE:${exename}Tests>)
    set_target_properties(${exename}Tests PROPERTIES INTERPROCEDURAL_OPTIMIZATION ${LTO_SUPPORTED})
endfunction (AddTestMDP)

function (AddTestPOMDP name)
    set(exename POMDP_${name})
    add_executable(${exename}Tests POMDP/${name}Tests.cpp)
    target_link_libraries(${exename}Tests AIToolboxMDP AIToolboxPOMDP ${Boost_UNIT_TEST_FRAMEWORK_LIBRARY} ${ARGN})
    add_test(NAME ${exename} WORKING_DIRECTORY ${CMAKE_RUNTIME_OUTPUT_DIRECTORY} COMMAND $<TARGET_FILE:${exename}Tests>)
    set_target_properties(${exename}Tests PROPERTIES INTERPROCEDURAL_OPTIMIZATION ${LTO_SUPPORTED})
endfunction (AddTestPOMDP)

function (AddTestFactored name)
    set(exename Factored_${name})
    add_executable(${exename}Tests Factored/${name}Tests.cpp)
    target_link_libraries(${exename}Tests AIToolboxMDP AIToolboxFMDP ${Boost_UNIT_TEST_FRAMEWORK_LIBRARY} ${ARGN})
    add_test(NAME ${exename} WORKING_DIRECTORY ${CMAKE_RUNTIME_OUTPUT_DIRECTORY} COMMAND $<TARGET_FILE:${exename}Tests>)
    set_target_properties(${exename}Tests PROPERTIES INTERPROCEDURAL_OPTIMIZATION ${LTO_SUPPORTED})
endfunction (AddTestFactored)

function (AddTestMDPPython name)
    set(exename MDP_Python_${name})
    add_test(NAME ${exename} WORKING_DIRECTORY ${CMAKE_LIBRARY_OUTPUT_DIRECTORY} COMMAND ${PYTHON_EXECUTABLE} ${CMAKE_CURRENT_SOURCE_DIR}/Python/MDP/${name}Tests.py)
endfunction (AddTestMDPPython)

function (AddTestPOMDPPython name)
    set(exename POMDP_Python_${name})
    add_test(NAME ${exename} WORKING_DIRECTORY ${CMAKE_LIBRARY_OUTPUT_DIRECTORY} COMMAND ${PYTHON_EXECUTABLE} ${CMAKE_CURRENT_SOURCE_DIR}/Python/POMDP/${name}Tests.py)
endfunction (AddTestPOMDPPython)

function (AddTestFactoredPython name)
    set(exename Factored_Python_${name})
    add_test(NAME ${exename} WORKING_DIRECTORY ${CMAKE_LIBRARY_OUTPUT_DIRECTORY} COMMAND ${PYTHON_EXECUTABLE} ${CMAKE_CURRENT_SOURCE_DIR}/Python/Factored/${name}Tests.py)
endfunction (AddTestFactoredPython)

if (MAKE_MDP)
    find_package(Boost 1.53 COMPONENTS unit_test_framework REQUIRED)
    include_directories(SYSTEM ${Boost_INCLUDE_DIRS})

    find_package(Eigen3 3.2.92 REQUIRED)
    include_directories(SYSTEM ${EIGEN3_INCLUDE_DIR})

    AddTestGlobal(Utils)

    AddTestMDP(Types)

    AddTestMDP(Experience)
    AddTestMDP(SparseExperience)
    AddTestMDP(Model)
    AddTestMDP(SparseModel)
    AddTestMDP(RLModel)
    AddTestMDP(SparseRLModel)

    AddTestMDP(QGreedyPolicy)
    AddTestMDP(WoLFPolicy)

    AddTestMDP(PrioritizedSweeping)
    AddTestMDP(QLearning)
    AddTestMDP(HystereticQLearning)
    AddTestMDP(SARSA)
    AddTestMDP(ExpectedSARSA)
    AddTestMDP(SARSAL)
    AddTestMDP(DynaQ)
    AddTestMDP(Dyna2)
    AddTestMDP(PolicyEvaluation)
    AddTestMDP(PolicyIteration)
    AddTestMDP(ValueIteration)
    AddTestMDP(MCTS)

    if (MAKE_PYTHON)
        # Normally one loads PythonInterp first, but since
        # here the interpreter is only optional then libs
        # come first. The version must be the same though.
        find_package(PythonInterp ${PYTHONLIBS_VERSION_STRING_STRIPPED} EXACT)
        if (PYTHONINTERP_FOUND)
            AddTestMDPPython(Experience)
            AddTestMDPPython(QLearning)
            AddTestMDPPython(ValueIteration)
            AddTestMDPPython(PolicyIteration)
        endif()
    endif()

    add_custom_command(
        TARGET Global_UtilsTests
        POST_BUILD
        COMMAND rsync
        ARGS -r
                "${PROJECT_SOURCE_DIR}/test/data"
                "${CMAKE_CURRENT_BINARY_DIR}"
        COMMENT "Copying data to test folder...")
endif()

if (MAKE_FMDP)
    AddTestFactored(Utils)
    AddTestFactored(FactoredContainer)
    AddTestFactored(FactorGraph)

    AddTestFactored(FactoredLP)

    AddTestFactored(LRPPolicy)
    AddTestFactored(ESRLPolicy)

    AddTestFactored(LLR)
    AddTestFactored(MultiObjectiveVariableElimination)
    AddTestFactored(SparseCooperativeQLearning)
    AddTestFactored(JointActionLearner)
    AddTestFactored(VariableElimination)

    if (MAKE_PYTHON)
        AddTestFactoredPython(JointActionLearner)
    endif()
endif()

if (MAKE_POMDP)
    AddTestPOMDP(Types)
    AddTestPOMDP(Utils)

    AddTestPOMDP(Model)
    AddTestPOMDP(SparseModel)

    AddTestPOMDP(AMDP)
    AddTestPOMDP(IncrementalPruning)
    AddTestPOMDP(PBVI)
    AddTestPOMDP(POMCP)
    AddTestPOMDP(BlindStrategies)
    AddTestPOMDP(FastInformedBound)
    AddTestPOMDP(rPOMCP)
    AddTestPOMDP(RTBSS)
    AddTestPOMDP(Witness)
    AddTestPOMDP(GapMin)

    if (MAKE_PYTHON)
        # Here we have surely passed in the MDP Python test branch, so we know
        # we have everything we need.
        AddTestPOMDPPython(IncrementalPruning)
        AddTestPOMDPPython(GapMin)
    endif()
endif()
