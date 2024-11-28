<img src="./images/IFX_LOGO_600.gif" align="right" width="150"/>

# PROT SMPU Configuration
**This code example shows how to configure the SMPU (Shared Memory Protection Unit) and describes its operation and initial settings.**

## Device
The device used in this code example (CE) is:
- [TRAVEO™ T2G CYT4DN Series](https://www.infineon.com/cms/en/product/microcontroller/32-bit-traveo-t2g-arm-cortex-microcontroller/32-bit-traveo-t2g-arm-cortex-for-cluster/traveo-t2g-cyt4dn/)

## Board
The board used for testing is:
- TRAVEO&trade; T2G Cluster 6M Lite Kit ([KIT_T2G_C-2D-6M_LITE](https://www.infineon.com/cms/en/product/evaluation-boards/kit_t2g_c-2d-6m_lite/))

## Scope of work
In this example, the SMPU is used for protection. If access violation is detected, a fault is generated.

## Introduction  
**Protection Unit**  
- An address range that is accessed by the transfer
    - Subregion: An address range is partitioned into eight equally-sized subregions and subregion can individual disables
- Access attributes such as:
    - Read/write attribute
    - Execute attribute to distinguish a code access from a data access
    - User/privilege attribute to distinguish access; for example, OS/kernel access from a task/thread access
    - Secure/non-secure attribute to distinguish a secure access from a non-secure access; the Arm Cortex-M CPUs do not
natively support this attribute
    - A protection context attribute to distinguish accesses from different protection contexts; for Peripheral-DMA (P-DMA) and Memory-DMA (M-DMA), this attribute is extended with a channel identifier, to distinguish accesses from different channels
- Memory protection
- Provided by memory protection units (MPUs) and shared memory protection units (SMPUs)
    - MPUs distinguish user and privileged accesses from a single bus master
    - SMPUs distinguish between different protection contexts and between secure and non-secure accesses
- Peripheral protection
    - Provided by peripheral protection units (PPUs)
    - The PPUs distinguish between different protection contexts; they also distinguish secure from non-secure accesses and user mode accesses from privileged mode accesses
- Protection pair structure
- Software Protection Unit (SWPU): SWPUs define flash write (or erase) permissions, and eFuse read and write permissions. An SWPU comprises of the following:
    - Flash Write Protection Unit (FWPU)
    - eFuse Read Protection Unit (ERPU)
    - eFuse Write Protection Unit (EWPU)

More details can be found in:
- TRAVEO&trade; T2G CYT4DN
  - [Technical Reference Manual (TRM)](https://www.infineon.com/dgdl/?fileId=8ac78c8c8691902101869f03007d2d87)
  - [Registers TRM](https://www.infineon.com/dgdl/?fileId=8ac78c8c8691902101869ef098052d79)
  - [Data Sheet](https://www.infineon.com/dgdl/?fileId=8ac78c8c869190210186f0cceff43fd0)

## Hardware setup
This CE has been developed for:
- TRAVEO&trade; T2G Cluster 6M Lite Kit ([KIT_T2G_C-2D-6M_LITE](https://www.infineon.com/cms/en/product/evaluation-boards/kit_t2g_c-2d-6m_lite/))<BR>

**Figure 1. KIT_T2G_C-2D-6M_LITE (Top View)**

<img src="./images/kit_t2g_c-2d-6m_lite.png" width="800" /><BR>
No changes are required from the board's default settings.

## Implementation
This design consists of a shared memory region for processor communication and uses the CM0+, the CM7_0 and the CM7_1 core. The PPU and UART communication is configured in the CM0+ core. Access to the SMPU protected memory region is done using the CM7_0 and CM7_1 core. CM7_0 can read the value of the shared memory region 1, CM7_1 can read the value of the shared memory region 2. Every other access will cause the system to generate a fault, which requires a reset to recover from.

**STDIN / STDOUT setting**

Initialization of the GPIO for UART is done in the <a href="https://infineon.github.io/retarget-io/html/group__group__board__libs.html#gaddff65f18135a8491811ee3886e69707"><i>cy_retarget_io_init()</i></a> function.
- Initialize the pin specified by CYBSP_DEBUG_UART_TX as UART TX, the pin specified by CYBSP_DEBUG_UART_RX as UART RX (these pins are connected to KitProg3 COM port)
- The serial port parameters are set to 8N1 and 115200 baud

<a href="https://infineon.github.io/mtb-pdl-cat1/pdl_api_reference_manual/html/group__group__scb__uart__low__level__functions.html#ga86ab3686a98a0e215c1f2eeed3ce254f"><i>Cy_SCB_UART_Get()</i></a> returns the user input from the terminal as received data.


**Shared memory region**

A shared SRAM memory region is defined in the linker file (*linker.ld*) of both core types. The region has the same address for all cores, which makes content stored in it accessible by these cores. The region is specified to be an address range of 0x100 and is dynamically placed after the private memory placeholder by using variables. In this example, the shared memory is evaluated to be at address 0x28000800.

**Bus master configuration**

In this example, two different protection context (PC) are used:

- PC5: Read and write access is not allowed for user and privileged
- PC6: Read and write access is allowed for user and privileged

CM7_0 is configured to be in protection context 6 and CM7_1 is configured to be in protection content 5.
Bus masters are configured, using the <a href="https://infineon.github.io/mtb-pdl-cat1/pdl_api_reference_manual/html/group__group__prot__functions__busmaster.html#ga4b69f79e24b30f22a706e973b05dd079"><i>Cy_Prot_ConfigBusMaster()</i></a> and <a href="https://infineon.github.io/mtb-pdl-cat1/pdl_api_reference_manual/html/group__group__prot__functions__busmaster.html#ga2d3a54039578a9fae98f6c7b4c4cff41"><i>Cy_Prot_SetActivePC()</i></a> function.

**Memory protection**

Two different memory addresses (TEST0_ADDR and TEST1_ADDR) are protected by receiving and configuring a SMPU structure. This is done by first receiving a free SMPU structure using <a href="https://infineon.github.io/mtb-pdl-cat1/pdl_api_reference_manual/html/group__group__prot__functions__smpu.html#ga9d10062d2d4cb3d656141b7d3217d7a9"><i>Cy_Prot_GetSmpuStruct()</i></a>, followed by disabling and configuring this structure using <a href="https://infineon.github.io/mtb-pdl-cat1/pdl_api_reference_manual/html/group__group__prot__functions__smpu.html#ga7d5ad654f90ef163ce8dcb14a21379f3"><i>Cy_Prot_DisableSmpuSlaveStruct()</i></a> and <a href="https://infineon.github.io/mtb-pdl-cat1/pdl_api_reference_manual/html/group__group__prot__functions__smpu.html#gac5c238971d86bc638507259c9a461f54"><i>Cy_Prot_ConfigSmpuSlaveStruct()</i></a>. To make the protection available, the corresponding SMPU structure needs to be enabled again, using <a href="https://infineon.github.io/mtb-pdl-cat1/pdl_api_reference_manual/html/group__group__prot__functions__smpu.html#ga3398e43534912ca2b980d08d08287801"><i>Cy_Prot_EnableSmpuSlaveStruct()</i></a>.

**Enabling CM7s**

After that, CM0+ enable both CM7_0/1 cores, using the <a href="https://infineon.github.io/mtb-pdl-cat1/pdl_api_reference_manual/html/group__group__system__config__cm7__functions.html#gacac741f6dd29eb66dc7e4fb31eef52fe"><i>Cy_SysEnableCM7()</i></a> function.

**Core communication**

To allow communication between different cores, a variable is defined and periodically checked by all cores. Depending on an ENUM value, placed by CM0+ and based on user input, the CM7_0 and CM7_1 core act and try to return the content from the specified memory address by using the shared memory.

**Detection of access violation**

A bus master access to a memory address protected by the SMPU, will be evaluated by the bus master's protection context. If the PC matches the requested access, it will be allowed, if not, the SMPU prevents the access and raises a pending fault.
Pending faults are checked by the CM0+ core by calling <a href="https://infineon.github.io/mtb-pdl-cat1/pdl_api_reference_manual/html/group__group__sysfault__functions.html#ga3da70af9e3a434ac08d9d5d63a09c480"><i>Cy_SysFault_GetPendingFault()</i></a> and prohibit further functioning of this example.

## Run and Test
For this example, a terminal emulator is required to display outputs and receive keys pressed. You can install a terminal emulator if you do not have one. In this example, [Tera Term](https://teratermproject.github.io/index-en.html) was used as the terminal emulator.

After code compilation, perform the following steps to flashing the device:
1. Connect the board to your PC using the provided USB cable through the KitProg3 USB connector.
2. Open a terminal program and select the KitProg3 COM port. Set the serial port parameters to 8N1 and 115200 baud.
3. Program the board using one of the following:
    - Select the code example project in the Project Explorer.
    - In the **Quick Panel**, scroll down, and click **[Project Name] Program (KitProg3_MiniProg4)**.
4. After programming, the code example starts automatically. Confirm that the messages are displayed on the UART terminal.

    - *Terminal output on program startup*<BR><img src="./images/terminal.png" width="640" />

5. When `1` and `4` is pressed, the cores try a memory read, which is allowed.
    - *Terminal output of cores when reading is allowed*<BR><img src="./images/terminal_read_allowed.png" width="640" />
6. When `2` is pressed, CM7_0 tries to read an address without proper rights.
    - *Terminal output after CM7_0 tries to read a protected address*<BR><img src="./images/terminal_read_prevented_CM7_0.png" width="640" />
7. When `3` is pressed, CM7_1 tries to read an address without proper rights.
    - *Terminal output after CM7_1 tries to read a protected address*<BR><img src="./images/terminal_read_prevented_CM7_1.png" width="640" />
8. You can debug the example to step through the code. In the IDE, use the **[Project Name] Debug (KitProg3_MiniProg4)** configuration in the **Quick Panel**. For details, see the "Program and debug" section in the [Eclipse IDE for ModusToolbox™ software user guide](https://www.infineon.com/dgdl/?fileId=8ac78c8c8929aa4d0189bd07dd6113f9).

**Note:** **(Only while debugging)** On the CM7 CPU, some code in *main()* may execute before the debugger halts at the beginning of *main()*. This means that some code executes twice: once before the debugger stops execution, and again after the debugger resets the program counter to the beginning of *main()*. See [KBA231071](https://community.infineon.com/t5/Knowledge-Base-Articles/PSoC-6-MCU-Code-in-main-executes-before-the-debugger-halts-at-the-first-line-of/ta-p/253856) to learn about this and for the workaround.


## References  
Relevant Application notes are:
- [AN235305](https://www.infineon.com/dgdl/?fileId=8ac78c8c8b6555fe018c1fddd8a72801) - Getting started with TRAVEO&trade; T2G family MCUs in ModusToolbox&trade;
- [AN219843](https://www.infineon.com/dgdl/?fileId=8ac78c8c7cdc391c017d0d3abf4b6772) - Protection configuration in TRAVEO™ T2G MCU

ModusToolbox™ is available online:
- <https://www.infineon.com/modustoolbox>

Associated TRAVEO™ T2G MCUs can be found on:
- <https://www.infineon.com/cms/en/product/microcontroller/32-bit-traveo-t2g-arm-cortex-microcontroller/>

More code examples can be found on the GIT repository:
- [TRAVEO™ T2G Code examples](https://github.com/orgs/Infineon/repositories?q=mtb-t2g-&type=all&language=&sort=)

For additional trainings, visit our webpage:  
- [TRAVEO™ T2G trainings](https://www.infineon.com/cms/en/product/microcontroller/32-bit-traveo-t2g-arm-cortex-microcontroller/32-bit-traveo-t2g-arm-cortex-for-cluster/traveo-t2g-cyt4dn/#!trainings)

For questions and support, use the TRAVEO™ T2G Forum:  
- <https://community.infineon.com/t5/TRAVEO-T2G/bd-p/TraveoII>  