# STM32_Framework_Template
## How to use this template ?
Firstly, get the list of supported microcontroller.
If you don't find your microcontroller, don't worry !
This template is customizable and you can easily add your microcontroller.
## Build project
### Build embedded firmware (cross-compilation)
```bash
mkdir build_target && cd build_target
cmake .. -GNinja -DCMAKE_BUILD_TYPE=<Debug or Release>
ninja
```
### Build tests (native compilation)
```bash
mkdir build_test && cd build_test
cmake .. -GNinja -DCMAKE_BUILD_TYPE=Test
ninja
ctest
ctest -N
ctest -R <Test_Name>
```
### Generate HTMP page for test coverage
```bash
mkdir test-coverage && cd test-coverage
geninfo ../build -b ../Tests -o ./coverage.info
genhtml coverage.info -o generate-html
```
## Flash after building a firmware
```bash
ninja flash
```
## Open a debug session
Open a debug session with openocd :
```bash
ninja debug
```