cmake_minimum_required (VERSION 3.25.2)

project (demo_addsub LANGUAGES CUDA CXX)

# --- C++ / CUDA Config ---
set (CMAKE_CXX_STANDARD 20)
set (CMAKE_CXX_STANDARD_REQUIRED ON)
set (CMAKE_CUDA_ARCHITECTURES "86;89")

# --- Packages ---
find_package (OpenMP REQUIRED)
find_package (FIDESlib REQUIRED CONFIG)
find_package (OpenFHE REQUIRED CONFIG)

# --- Source Files (暴力法：直接指定檔案) ---
# 設定原始碼的基礎路徑
set (SRC_PATH "examples/demo_addsub/src")

# 明確列出所有需要的檔案，不要用 GLOB
# 這樣如果檔案路徑錯了，CMake 配置階段就會直接報錯，而不是等到編譯才失敗
add_executable (demo_addsub
    "${SRC_PATH}/main.cu"
    "${SRC_PATH}/context.cpp"
    # !!! 請在下方加入 load_csv 所在的檔案 !!!
    # 例如： "${SRC_PATH}/utils.cpp" 或 "${SRC_PATH}/csv_reader.cpp"
    # 如果找不到 load_csv 的檔案，請暫時註解掉下面這行，但編譯還是會失敗
    # "${SRC_PATH}/YOUR_MISSING_FILE_HERE.cpp"
)

# --- Include Directories ---
target_include_directories(demo_addsub PRIVATE 
    "${SRC_PATH}"
    ${OPENMP_INCLUDES}
)

# --- Link Libraries (修正順序) ---
# 強制指定順序：BINFHE -> PKE -> CORE
# 使用完整路徑確保順序不被優化掉

# 這裡假設你的 OpenFHE 安裝在 /usr/local/lib (根據你的錯誤訊息)
# 如果你是用 conda 或其他路徑，請改用 ${OpenFHE_LIBDIR}
set(LIB_PATH "/usr/local/lib") 

target_link_libraries(demo_addsub PRIVATE
    # OpenFHE Libraries (順序極為重要)
    "${LIB_PATH}/libOPENFHEbinfhe_static.a"
    "${LIB_PATH}/libOPENFHEpke_static.a"     # PKE 依賴 Core
    "${LIB_PATH}/libOPENFHEcore_static.a"    # Core 被依賴，放最後
    
    # 其他依賴
    FIDESlib::fideslib
    OpenMP::OpenMP_CXX
)

# --- Compile Options ---
target_compile_options(demo_addsub PRIVATE
    $<$<COMPILE_LANGUAGE:CXX>:${OpenMP_CXX_FLAGS}>
    $<$<COMPILE_LANGUAGE:CUDA>:-Xcompiler=${OpenMP_CXX_FLAGS}>
)

target_link_options(demo_addsub PRIVATE 
    $<$<LINK_LANGUAGE:CUDA>:-Xcompiler=${OpenMP_CXX_FLAGS}>
)