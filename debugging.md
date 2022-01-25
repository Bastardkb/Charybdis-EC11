This is the chain of communication with the Blackpillboard until the USB part everything is on your host.

```
[VSCode + Cortex Debug Extension] <-(GDB Server)-> [OpenOCD] <-(USB)-> [STLink] <-(SWD)-> [STM32F411]
```

**You need:**

* `VSCode` with:
  * Cortex-Debug Extension 
  * Cortex-Debug Device Support Support Pack STM32F4
  * **(1)** A working `.vscode/launch.json` config that is configured for QMK and OpenOCD in the root of QMK folder
  * Optional a `.vscode/tasks.json` to build your firmware on every debug launch
* GCC ARM toolchain to supply `arm-none-eabi-gdb` (or use `gdb-multiarch`) and `arm-none-eabi-objdump`
* `OpenOCD` which communicates with the STLink and spans an GDB server that Cortex-Debug attaches to
* Working SWD connection from STLink to the STM32F411 debug Header 
  * Best to Google for the pinout of STLink V2, double check the connections this is likely to fail
* **(2)** Firmware in `ELF` format with debug symbols, `Og` optimization at max and without link time optimizations aka. `LTO`

**VSCode**

**(1) `launch.json`**

```json
{
    "version": "0.2.0",
    "configurations": [
        {
            "type": "cortex-debug",
            "servertype": "openocd",
            "interface": "swd",
            "request": "launch",
            "name": "Debug launch Blackpill STLink",
            "cwd": "${workspaceRoot}",
            "executable": ".build/karlk90_yaemk_koy.elf", /* Change this to your keyboard */
            "device": "STM32F411CE", /* Gives access to peripherals in Debugger view */
            "configFiles": [
                "/usr/share/openocd/scripts/board/st_nucleo_f4.cfg" /* Change this to the path of st_nucleo_f4.cfg */
            ],
            "runToEntryPoint": "main", /* Stops at the main function on entry */
            "rtos": "ChibiOS", /* Provides debugging of threads */
            "armToolchainPath": "/usr/bin/", /* Change this to the path of your GCC ARM toolchain */
            "gdbPath": "gdb-multiarch", /* Change this to the path of an ARM capable GDB e.g. arm-none-eabi-gdb */
            "preLaunchTask": "make keyboard" /* Run this task before every  */,
        }
    ]
}
```

**`tasks.json`**

```json
{
    "version": "2.0.0",
    "tasks": [
        {
            "label": "make keyboard",
            "type": "shell",
            "command": "make karlk90/yaemk:koy -j 12" /* Change this to your keyboard */
        },
    ]
}
```

**ELF with Debug capabilities**

**rules.mk**

```
LTO_ENABLE =   no
EXTRAFLAGS += -Og -gdwarf-4
ALLOW_WARNINGS =   yes
```
