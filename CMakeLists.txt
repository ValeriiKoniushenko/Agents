# Shared Agents skill installer.
#
# Include this file from the host project's root CMakeLists.txt after placing this
# repository at <host>/.agents/Agents. It creates one relative directory symlink
# per shared skill in <host>/.agents/skills, so project-local skills can coexist
# with the shared set.

# This installer belongs to the host root project. If the Agents repository is added as
# a CMake subdirectory, it is library content rather than the host's skill installer.
if(NOT CMAKE_CURRENT_SOURCE_DIR STREQUAL CMAKE_SOURCE_DIR)
    message(STATUS "[Agents] Skipping shared-skill installer outside the root project.")
    return()
endif()

include_guard(GLOBAL)

if(CMAKE_VERSION VERSION_LESS "3.20")
    message(FATAL_ERROR
        "[Agents] CMake ${CMAKE_VERSION} is too old. The shared-skill installer "
        "requires CMake 3.20 or newer."
    )
endif()

get_filename_component(_agents_repository_dir "${CMAKE_CURRENT_LIST_DIR}" REALPATH)
get_filename_component(_agents_expected_host_root "${_agents_repository_dir}/../.." REALPATH)
get_filename_component(_agents_host_root "${CMAKE_SOURCE_DIR}" REALPATH)

if(NOT _agents_expected_host_root STREQUAL _agents_host_root)
    message(FATAL_ERROR
        "[Agents] This installer must be included from the host project's root CMakeLists.txt.\n"
        "[Agents] Expected host root: ${_agents_expected_host_root}\n"
        "[Agents] CMAKE_SOURCE_DIR: ${_agents_host_root}\n"
        "[Agents] Expected layout: <host>/.agents/Agents/CMakeLists.txt"
    )
endif()

set(_agents_shared_skills_dir "${_agents_repository_dir}/skills")
set(_agents_link_directory "${_agents_host_root}/.agents/skills")

if(NOT IS_DIRECTORY "${_agents_shared_skills_dir}")
    message(FATAL_ERROR
        "[Agents] Shared-skill directory is missing or is not a directory: "
        "${_agents_shared_skills_dir}"
    )
endif()

find_program(_agents_git_executable NAMES git)
if(NOT _agents_git_executable)
    message(FATAL_ERROR
        "[Agents] Git is required to validate symlink support, but no git executable was found."
    )
endif()

execute_process(
    COMMAND "${_agents_git_executable}" rev-parse --is-inside-work-tree
    WORKING_DIRECTORY "${_agents_host_root}"
    RESULT_VARIABLE _agents_host_git_result
    OUTPUT_VARIABLE _agents_host_git_output
    ERROR_VARIABLE _agents_host_git_error
    OUTPUT_STRIP_TRAILING_WHITESPACE
    ERROR_STRIP_TRAILING_WHITESPACE
)
if(NOT _agents_host_git_result EQUAL 0 OR NOT _agents_host_git_output STREQUAL "true")
    message(FATAL_ERROR
        "[Agents] The host source directory must be a Git work tree before shared skills can be linked.\n"
        "[Agents] git rev-parse output: ${_agents_host_git_output}\n"
        "[Agents] git rev-parse error: ${_agents_host_git_error}"
    )
endif()

execute_process(
    COMMAND "${_agents_git_executable}" rev-parse --show-toplevel
    WORKING_DIRECTORY "${_agents_repository_dir}"
    RESULT_VARIABLE _agents_repository_git_result
    OUTPUT_VARIABLE _agents_repository_git_root
    ERROR_VARIABLE _agents_repository_git_error
    OUTPUT_STRIP_TRAILING_WHITESPACE
    ERROR_STRIP_TRAILING_WHITESPACE
)
if(NOT _agents_repository_git_result EQUAL 0)
    message(FATAL_ERROR
        "[Agents] ${_agents_repository_dir} is not an initialized Git repository.\n"
        "[Agents] Run: git init .agents/Agents\n"
        "[Agents] Git error: ${_agents_repository_git_error}"
    )
endif()

get_filename_component(_agents_repository_git_root "${_agents_repository_git_root}" REALPATH)
if(NOT _agents_repository_git_root STREQUAL _agents_repository_dir)
    message(FATAL_ERROR
        "[Agents] The shared-skills repository root is unexpected.\n"
        "[Agents] Expected: ${_agents_repository_dir}\n"
        "[Agents] Git reports: ${_agents_repository_git_root}"
    )
endif()

# Git controls whether a checked-out symlink remains a symlink. An explicit false
# setting breaks this layout on every platform. On Windows an unset value is unsafe,
# because Git for Windows commonly defaults to disabled symlink support.
execute_process(
    COMMAND "${_agents_git_executable}" config --type=bool --get core.symlinks
    WORKING_DIRECTORY "${_agents_host_root}"
    RESULT_VARIABLE _agents_symlink_config_result
    OUTPUT_VARIABLE _agents_symlink_config
    ERROR_VARIABLE _agents_symlink_config_error
    OUTPUT_STRIP_TRAILING_WHITESPACE
    ERROR_STRIP_TRAILING_WHITESPACE
)

if(_agents_symlink_config_result EQUAL 0)
    string(TOLOWER "${_agents_symlink_config}" _agents_symlink_config)
    if(_agents_symlink_config STREQUAL "false")
        message(FATAL_ERROR
            "[Agents] ================================================================\n"
            "[Agents] GIT CORE.SYMLINKS IS FALSE — SHARED SKILLS CANNOT WORK SAFELY.\n"
            "[Agents] Enable it in the host repository, then reconfigure:\n"
            "[Agents]   git config --local core.symlinks true\n"
            "[Agents] ================================================================"
        )
    elseif(NOT _agents_symlink_config STREQUAL "true")
        message(FATAL_ERROR
            "[Agents] Git returned an invalid core.symlinks value: '${_agents_symlink_config}'.\n"
            "[Agents] Git error: ${_agents_symlink_config_error}"
        )
    endif()
    message(STATUS "[Agents] Git core.symlinks=true")
elseif(WIN32)
    message(FATAL_ERROR
        "[Agents] ==================================================================\n"
        "[Agents] WINDOWS REQUIRES GIT CORE.SYMLINKS=TRUE FOR SHARED SKILL LINKS.\n"
        "[Agents] core.symlinks is unset or unreadable in this host repository.\n"
        "[Agents] Run: git config --local core.symlinks true\n"
        "[Agents] Then ensure Windows Developer Mode or equivalent symlink permission is enabled.\n"
        "[Agents] Git error: ${_agents_symlink_config_error}\n"
        "[Agents] =================================================================="
    )
else()
    message(STATUS
        "[Agents] Git core.symlinks is unset; continuing with the platform default on this non-Windows host."
    )
endif()

if(IS_SYMLINK "${_agents_link_directory}")
    message(FATAL_ERROR
        "[Agents] ${_agents_link_directory} is itself a symlink. This installer needs a real "
        "directory so shared and project-local skills can coexist."
    )
elseif(EXISTS "${_agents_link_directory}" AND NOT IS_DIRECTORY "${_agents_link_directory}")
    message(FATAL_ERROR
        "[Agents] Skill link destination exists but is not a directory: ${_agents_link_directory}"
    )
elseif(NOT EXISTS "${_agents_link_directory}")
    file(MAKE_DIRECTORY "${_agents_link_directory}")
    if(NOT IS_DIRECTORY "${_agents_link_directory}")
        message(FATAL_ERROR
            "[Agents] Failed to create the shared-skill link directory: ${_agents_link_directory}"
        )
    endif()
endif()

file(GLOB _agents_skill_entries LIST_DIRECTORIES true RELATIVE "${_agents_shared_skills_dir}"
    "${_agents_shared_skills_dir}/*"
)
list(SORT _agents_skill_entries)

set(_agents_valid_skills)
foreach(_agents_skill_name IN LISTS _agents_skill_entries)
    set(_agents_skill_source "${_agents_shared_skills_dir}/${_agents_skill_name}")

    if(NOT IS_DIRECTORY "${_agents_skill_source}")
        message(WARNING
            "[Agents] Ignoring shared-skills entry that is not a directory: ${_agents_skill_source}"
        )
        continue()
    endif()

    if(NOT _agents_skill_name MATCHES "^[a-z0-9][a-z0-9-]*$")
        message(FATAL_ERROR
            "[Agents] Invalid shared skill directory name '${_agents_skill_name}'. "
            "Use lowercase letters, digits, and hyphens only."
        )
    endif()

    if(NOT EXISTS "${_agents_skill_source}/SKILL.md")
        message(WARNING
            "[Agents] Ignoring incomplete shared skill '${_agents_skill_name}': SKILL.md is missing."
        )
        continue()
    endif()

    file(SIZE "${_agents_skill_source}/SKILL.md" _agents_skill_size)
    if(_agents_skill_size EQUAL 0)
        message(FATAL_ERROR
            "[Agents] Shared skill '${_agents_skill_name}' has an empty SKILL.md."
        )
    endif()

    list(APPEND _agents_valid_skills "${_agents_skill_name}")
endforeach()

if(NOT _agents_valid_skills)
    message(FATAL_ERROR
        "[Agents] No valid shared skills were found under ${_agents_shared_skills_dir}."
    )
endif()

foreach(_agents_skill_name IN LISTS _agents_valid_skills)
    set(_agents_skill_source "${_agents_shared_skills_dir}/${_agents_skill_name}")
    set(_agents_skill_link "${_agents_link_directory}/${_agents_skill_name}")
    get_filename_component(_agents_skill_source_real "${_agents_skill_source}" REALPATH)

    if(IS_SYMLINK "${_agents_skill_link}")
        file(REAL_PATH "${_agents_skill_link}" _agents_skill_link_real)
        if(NOT _agents_skill_link_real STREQUAL _agents_skill_source_real)
            message(FATAL_ERROR
                "[Agents] Refusing to replace shared skill link '${_agents_skill_link}'.\n"
                "[Agents] It resolves to: ${_agents_skill_link_real}\n"
                "[Agents] Expected: ${_agents_skill_source_real}"
            )
        endif()
        message(STATUS "[Agents] Verified shared skill link: ${_agents_skill_name}")
        continue()
    endif()

    if(EXISTS "${_agents_skill_link}")
        message(FATAL_ERROR
            "[Agents] Refusing to overwrite existing project-local skill: ${_agents_skill_link}.\n"
            "[Agents] Move or rename it before linking the shared '${_agents_skill_name}' skill."
        )
    endif()

    file(RELATIVE_PATH _agents_relative_target "${_agents_link_directory}" "${_agents_skill_source_real}")
    file(CREATE_LINK "${_agents_relative_target}" "${_agents_skill_link}" SYMBOLIC
        RESULT _agents_link_result
    )
    if(NOT _agents_link_result STREQUAL "0")
        message(FATAL_ERROR
            "[Agents] Failed to create the '${_agents_skill_name}' skill symlink.\n"
            "[Agents] Link: ${_agents_skill_link}\n"
            "[Agents] Target: ${_agents_relative_target}\n"
            "[Agents] Error: ${_agents_link_result}"
        )
    endif()

    if(NOT IS_SYMLINK "${_agents_skill_link}")
        message(FATAL_ERROR
            "[Agents] CMake reported success but '${_agents_skill_link}' is not a symlink."
        )
    endif()

    file(REAL_PATH "${_agents_skill_link}" _agents_skill_link_real)
    if(NOT _agents_skill_link_real STREQUAL _agents_skill_source_real)
        message(FATAL_ERROR
            "[Agents] Created skill link resolves to the wrong target.\n"
            "[Agents] Link: ${_agents_skill_link}\n"
            "[Agents] Resolves to: ${_agents_skill_link_real}\n"
            "[Agents] Expected: ${_agents_skill_source_real}"
        )
    endif()

    message(STATUS "[Agents] Linked shared skill: ${_agents_skill_name}")
endforeach()

list(LENGTH _agents_valid_skills _agents_skill_count)
message(STATUS "[Agents] Shared skill linking complete (${_agents_skill_count} skill(s)).")
