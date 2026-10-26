# 01 — Introduction to Zephyr RTOS 🚀

> **A beginner's guide in very simple language.**  
> Imagine you are 10 years old, but you are also curious about how tiny computers work.
>
> **What we will learn:** Why an RTOS is useful, why Zephyr was created when FreeRTOS and ThreadX already existed, the good and bad sides of Zephyr, every directory in *your* workspace, and how to run Hello World in QEMU on Windows.

---

## 1. First, what is a tiny computer?

A **microcontroller (MCU)** is a tiny computer inside a toy, washing machine, smartwatch, robot, or car part.

It may need to do many jobs:

- Blink an LED.
- Read a temperature sensor.
- Listen for a button press.
- Send a Bluetooth message.
- Watch for a dangerous condition and react quickly.

The software we put inside it is called **firmware**.

### What happens without an RTOS?

You can write one big `while (1)` loop:

```c
while (1) {
    check_button();
    read_sensor();
    blink_led();
}
```

For small projects, this is **perfectly fine**. It is often called *bare-metal programming* when no operating system is used.

But imagine `read_sensor()` sometimes takes a long time. Then the button may have to wait. As jobs grow, this big loop can become difficult to manage.

### What is an RTOS?

**RTOS = Real-Time Operating System.**

Think of an RTOS as a **school teacher who organizes many children doing different jobs**.

| Child's job | Tiny computer's job |
|---|---|
| Ring the bell | Blink an LED |
| Check attendance | Read a sensor |
| Answer an emergency call | Handle an urgent event |
| Send a message | Send data using Bluetooth or Wi-Fi |

The RTOS helps decide **which job should use the CPU next**.

Each independent job is often called a **thread** or **task**. The part of the RTOS that chooses which one runs is called the **scheduler**.

> **Important:** “Real-time” does not just mean “very fast.” It means the system can respond **within a required amount of time**. For example, a safety signal may need attention before a deadline.

---

## 2. We already had FreeRTOS and ThreadX. Why did Zephyr appear?

**FreeRTOS and ThreadX are not bad!** They solve real problems, and both are still useful.

Imagine three builders making robots:

- **FreeRTOS:** “Here is a small, reliable engine for managing tasks. Choose the other parts you need.”
- **ThreadX (now Eclipse ThreadX):** “Here is an efficient RTOS with a family of companion tools and software, too.”
- **Zephyr:** “Let us create a shared toolbox that combines an RTOS, hardware descriptions, drivers, networking, testing, and one common build approach for lots of boards.”

**Zephyr was announced in 2016 as a Linux Foundation-hosted, community-driven project.** One reason it was started was to help different companies work together on software for small connected devices, instead of everyone solving the same problems separately.

That does **not** mean Zephyr was invented because FreeRTOS or ThreadX could not work. It was built with a **different goal and development model**.

### Simple example

You want to build a small weather station that:

1. Reads temperature.
2. Shows it on a display.
3. Sends it over Bluetooth.
4. Saves readings to flash memory.

With any of these RTOSes, a good engineer can build it. **Zephyr can be convenient** because many drivers, software services, board definitions, and build tools already follow a shared system. With FreeRTOS or ThreadX, you may use their own libraries, vendor tools, or extra components to do the same job.

---

## 3. Zephyr vs FreeRTOS vs ThreadX — easy comparison

| Question | FreeRTOS | Eclipse ThreadX | Zephyr |
|---|---|---|---|
| Main idea | Small RTOS kernel + optional libraries | Efficient RTOS + companion middleware | Broad, integrated embedded software platform |
| Can it run threads/tasks? | **Yes** | **Yes** | **Yes** |
| Can it use drivers and networking? | **Yes**, through its ecosystem and integrations | **Yes**, through companion components and integrations | **Yes**, many are in the Zephyr ecosystem |
| Easy for a first small MCU project? | Often easier to start | Can be straightforward with supported tools | More concepts to learn at first |
| Board configuration | Often vendor/IDE/project specific | Often vendor/IDE/project specific | Common **Devicetree + Kconfig + CMake** approach |
| Best reason to pick it | Small, focused RTOS projects | Existing ThreadX-based or supported product ecosystems | Projects needing many integrated features or boards |
| Biggest beginner challenge | Choosing and connecting extra pieces | Learning the ecosystem/tooling | Understanding its larger build/configuration system |

**A fair rule:** There is no single “best RTOS.” Choose based on your MCU, RAM/flash size, drivers, deadlines, team experience, product needs, and maintenance requirements.

> **Not true:** “FreeRTOS has no networking” or “ThreadX has no USB.” Both have ecosystems that provide these capabilities. The difference is often **how the pieces are packaged and configured**, not whether they exist.

---

## 4. What extra power does Zephyr give us?

Think of Zephyr like a **LEGO kit with labeled boxes**.

| Feature | What it means in simple words | Why it helps |
|---|---|---|
| **Threads + scheduler** | Lets several jobs take turns using the CPU | A sensor and LED can be managed separately |
| **Drivers** | Ready-made software for talking to hardware | Less low-level code to write yourself |
| **Devicetree** | A map describing the board's hardware | Change boards without rewriting every hardware detail |
| **Kconfig** | A menu of software switches | Turn features on/off without editing their source code |
| **West** | A helper command tool | Download projects, build, flash, and debug |
| **CMake + Ninja** | Tools that organize the build | Compile and link the correct pieces |
| **Connectivity stacks** | Software for talking over networks | Useful for Bluetooth, IP networking, etc. |
| **Security and update support** | Tools and integrations for protecting/updating devices | Useful for connected products |
| **Tests and samples** | Practice projects and checks | Learn and catch mistakes more easily |
| **QEMU support** | A pretend board running inside the PC | Learn before buying physical hardware |

**A crucial idea:** Zephyr is **modular**. The firmware does not automatically include every driver and network feature. The build is meant to select what your application needs.

### Three words you will see everywhere

- **Devicetree:** *What hardware do I have?* Example: “UART 1 exists, and this board uses it.”
- **Kconfig:** *What software features should be enabled?* Example: “Enable UART console support.”
- **CMake:** *What should be compiled and linked?* Example: “Build the application with the selected kernel and driver code.”

---

## 5. What are Zephyr's disadvantages?

Zephyr has many useful features, but it is not magic.

1. **Steeper learning curve:** You meet Kconfig, Devicetree, West, CMake, and toolchains. That is a lot of new names.
2. **More moving parts:** A build error might come from configuration, hardware description, source code, or installed tools.
3. **Not always smallest or simplest:** For an LED-only project, bare metal or a small RTOS setup may be easier. Actual memory use depends on configuration.
4. **Hardware support varies:** A board may be listed as supported while a particular peripheral or feature still needs work.
5. **Debugging takes practice:** You may need to inspect generated `.config`, `zephyr.dts`, logs, and the linker map.
6. **Upgrading can require effort:** APIs, board definitions, or dependencies can change across Zephyr versions.
7. **Real-time performance is not automatic:** You still must design thread priorities, memory use, interrupts, and time limits carefully.

### So which one should I choose?

- **One simple LED or sensor:** Bare metal may be enough.
- **A few tasks and tight memory:** FreeRTOS or ThreadX may be a comfortable choice.
- **Many peripherals, connectivity features, and a reusable board/build setup:** Zephyr is especially worth considering.
- **An existing product already using one RTOS:** Keeping the working RTOS is often sensible unless there is a strong reason to change.

---

## 6. Understand your Zephyr workspace first

Your Windows workspace is:

```text
C:\Users\ragha\workspace\personal\zephyrproject
```

It has **two levels**:

1. **`zephyrproject/` = the whole workshop** (tools, other projects, generated output).
2. **`zephyrproject/zephyr/` = the Zephyr operating-system source code**.

### Level 1 — `zephyrproject/`

```text
zephyrproject/
├── .venv/
├── .west/
├── bootloader/
├── build/
├── build.py
├── doc/
├── modules/
├── tools/
└── zephyr/                 <-- Main Zephyr source repository
```

| Directory or file | Child-friendly meaning | Actual purpose |
|---|---|---|
| **`.venv/`** | A private Python school bag | Python environment containing project tools/packages |
| **`.west/`** | The workshop's address book | West workspace settings; tells West about the manifest repository |
| **`bootloader/`** | The helper that starts firmware | Other bootloader repositories, commonly MCUboot-related |
| **`build/`** | The finished-project output tray | Generated firmware and intermediate build files; exact contents depend on builds |
| **`build.py`** | Your extra shortcut button | A local/custom Python script; its behavior cannot be known from the filename alone |
| **`doc/`** | Extra documentation shelf | Documentation-related checked-out projects; inspect its contents for exact purpose |
| **`modules/`** | Boxes of extra building blocks | External libraries, vendor HALs, CMSIS and other dependencies |
| **`tools/`** | Extra workshop tools | Auxiliary tooling repositories; exact contents depend on your manifest |
| **`zephyr/`** | The big instruction book plus engine | Zephyr source, drivers, board support, samples, tests, build setup |

**Important:** `.` and `..` shown by Windows `dir` are navigation entries meaning “here” and “parent folder.” They are **not extra Zephyr projects**.

### Why are there two `modules/` folders?

These are different:

- `zephyrproject/modules/` — usually contains **actual external source repositories**.
- `zephyrproject/zephyr/modules/` — contains **Zephyr-side integration/build code** for some modules.

Think: one is the **box of parts**; the other is the **instructions for fitting some of those parts into the robot**.

---

## 7. Level 2 — inside `zephyrproject/zephyr/`

This is the main Zephyr source tree. Below are **all 22 named directories** from your listing (not counting the `.` and `..` navigation entries).

```text
zephyr/
├── .cache/
├── .github/
├── arch/
├── boards/
├── cmake/
├── doc/
├── drivers/
├── dts/
├── include/
├── kernel/
├── lib/
├── LICENSES/
├── misc/
├── modules/
├── samples/
├── scripts/
├── share/
├── snippets/
├── soc/
├── submanifests/
├── subsys/
└── tests/
```

### A. The tiny computer's brain

| Directory | Easy analogy | What it really does |
|---|---|---|
| **`kernel/`** | The classroom teacher | Core RTOS work: threads, scheduling, semaphores, mutexes, timers, work queues |
| **`arch/`** | Rules for each type of brain | CPU architecture code: ARM, x86, RISC-V; context switching, interrupt handling, etc. |
| **`soc/`** | Details of a particular brain chip | MCU/SoC-family support, configuration, and platform initialization |
| **`include/`** | A dictionary of commands | Header files (`.h`) containing APIs used by applications and system code |

**Example:** `#include <zephyr/kernel.h>` tells the compiler about Zephyr kernel APIs.

### B. Talking to real hardware

| Directory | Easy analogy | What it really does |
|---|---|---|
| **`boards/`** | Map of a complete toy robot | Board definitions/configuration, board hardware files, board support |
| **`drivers/`** | Translators for hardware | Code to control GPIO, UART, SPI, I2C, sensors, flash, and other devices |
| **`dts/`** | Hardware address book | Devicetree files and bindings describing hardware connections and properties |

**Difference:** `soc/` describes the **chip**, `boards/` describes the **whole board**, `dts/` helps describe **hardware**, and `drivers/` contains **code that controls hardware**.

### C. Extra software features

| Directory | Easy analogy | What it really does |
|---|---|---|
| **`subsys/`** | Special school clubs | Higher-level services: networking, Bluetooth, USB, logging, filesystems, etc. |
| **`lib/`** | Box of helper functions | Reusable software utilities and libraries |
| **`modules/`** | Directions for extra parts | Integration/build support for external modules |
| **`misc/`** | Miscellaneous drawer | Supporting resources that do not fit other main categories |

### D. Building the firmware

| Directory | Easy analogy | What it really does |
|---|---|---|
| **`cmake/`** | The construction planner | CMake scripts that help select and build source files |
| **`scripts/`** | Automatic helpers | Python and other build, test, code-generation and validation tools |
| **`share/`** | Shared tool instructions | Shared build/integration resources such as Zephyr CMake package support |
| **`snippets/`** | Reusable recipe cards | Named bundles of build/configuration settings |
| **`submanifests/`** | Extra shopping lists | Additional West manifest definitions that can include more projects |

### E. Learning, quality and project information

| Directory | Easy analogy | What it really does |
|---|---|---|
| **`samples/`** | Practice exercises | Example applications, such as Hello World and Blinky |
| **`tests/`** | Exam papers | Tests that check kernel features, drivers, and other parts |
| **`doc/`** | Textbook | Source of Zephyr's technical documentation |
| **`.github/`** | Team's online checklist | GitHub automation, CI workflows, templates, etc. |
| **`.cache/`** | Temporary notes | Local cached/generated information; exact contents depend on tools |
| **`LICENSES/`** | Permission papers | License texts used for legal compliance |

**A beginner does not need to understand or edit every folder.** Start with `samples/`, `include/`, `kernel/`, and then `drivers/`, `boards/`, and `dts/`.

---

## 8. Important files in the `zephyr/` root

These are **files**, not folders. You saw them when running `dir`.

| File | Purpose in easy words |
|---|---|
| `CMakeLists.txt` | Main build instructions |
| `Kconfig` | Starting point for software feature choices |
| `Kconfig.zephyr` | Larger Zephyr software configuration definitions |
| `Kconfig.constants` | Common constants used by configuration |
| `west.yml` | List/map of other Git projects in the workspace |
| `VERSION` | Zephyr version information |
| `SDK_VERSION` | SDK version information relevant to this checkout |
| `version.h.in` | Template for generated version information |
| `zephyr-env.cmd` | Environment setup helper for Windows CMD |
| `zephyr-env.sh` | Environment setup helper for shell environments |
| `README.rst` | Introduction to the Zephyr repository |
| `CONTRIBUTING.rst` | How to contribute code to Zephyr |
| `AGENTS.md`, `CLAUDE.md` | Instructions meant for AI coding assistants |
| `.clang-format`, `.editorconfig` | How source code should be formatted |
| `.checkpatch.conf`, `.gitlint`, `.yamllint`, `.ruff.toml`, `.ruff-excludes.toml` | Settings for style and code checks |
| `.codechecker.yml`, `.codecov.yml` | Code checking and coverage-related configuration |
| `.github/` | GitHub settings/workflows **directory** (also listed above) |
| `.gitignore`, `.gitattributes`, `.mailmap` | Git repository settings and contributor name mapping |
| `MAINTAINERS.yml`, `CODEOWNERS` | Who maintains different areas of the project |
| `LICENSE`, `README.license`, `REUSE.toml` | Licensing information and metadata |
| `CODE_OF_CONDUCT.md` | Rules for respectful project collaboration |

Most beginners will mainly encounter `CMakeLists.txt`, `Kconfig`, `west.yml`, and the application-specific files introduced next.

---

## 9. How Zephyr turns your code into firmware

Imagine we want to print a message:

```text
Your application (main.c)
          |
          v
Kconfig chooses software features
Devicetree describes hardware
          |
          v
CMake selects the needed source code
          |
          v
Compiler + linker make firmware (zephyr.elf)
          |
          v
QEMU pretends to be a computer board
          |
          v
Hello World! x86
```

### Four important application files

A small Zephyr application often looks like this:

```text
my_app/
├── CMakeLists.txt     <-- How to build the application
├── prj.conf           <-- Software settings (Kconfig values)
├── src/
│   └── main.c         <-- Your program
└── boards/            <-- Optional board-specific configuration
```

- **`main.c`:** What should my program do?
- **`prj.conf`:** Which software features do I want?
- **`CMakeLists.txt`:** Which application code should be built?
- **Board/devicetree files:** What hardware am I using?

The **source code** and the **build result** are different things. Do not edit generated files inside `build/` to change the application; change your source/configuration and rebuild.

---

## 10. First experiment: Hello World using QEMU on Windows PowerShell

**QEMU** is like a pretend electronic board inside your PC. We can run Zephyr there even if we do not have an STM32 board connected.

### Step 1 — Go to your Zephyr source directory

Open **PowerShell**:

```powershell
cd C:\Users\ragha\workspace\personal\zephyrproject\zephyr
```

### Step 2 — Activate Python's virtual environment

Your `.venv` is in the **parent** folder. From inside `zephyr/`, use:

```powershell
..\.venv\Scripts\Activate.ps1
```

You should see `(.venv)` at the start of the prompt.

**Do not use** `.venv\Scripts\activate.bat` inside PowerShell: `.bat` is for **CMD**, and that path is also incorrect when you are already inside `zephyr/`.

If PowerShell reports that script execution is blocked, you can temporarily allow local scripts in **only this terminal session**, if your PC's security policy permits it:

```powershell
Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass
..\.venv\Scripts\Activate.ps1
```

### Step 3 — Check your tools

```powershell
west --version
cmake --version
ninja --version
```

If one is not recognized, the environment or installation may need fixing first.

### Step 4 — Find the ready-made sample

```powershell
Get-Content .\samples\hello_world\src\main.c
```

We are **using the sample already inside Zephyr**. No need to create a new application yet.

### Step 5 — Build for an imaginary x86 board

Run this **from the `zephyr/` directory**:

```powershell
west build -b qemu_x86 samples\hello_world -d ..\build\hello_world_qemu
```

Meaning:

- `west build` = make the firmware.
- `-b qemu_x86` = choose the pretend x86 board.
- `samples\hello_world` = source program.
- `-d ..\build\hello_world_qemu` = keep results in their own output folder.

### Step 6 — Run it

```powershell
west build -d ..\build\hello_world_qemu -t run
```

The console should show something similar to:

```text
*** Booting Zephyr OS build ... ***
Hello World! x86
```

The version/banner text will differ between installations.

To stop QEMU: press **Ctrl+A**, release, then press **X**.

> **Tip:** QEMU must be available (for example, through the Zephyr SDK or a separate supported installation). Building can succeed even when a missing QEMU executable prevents running.

---

## 11. Tiny glossary (big words made small)

| Word | Simple meaning |
|---|---|
| **MCU** | A tiny computer chip |
| **Firmware** | Software put inside the chip |
| **RTOS** | A system that organizes jobs with timing needs |
| **Task/thread** | One job the program can run |
| **Scheduler** | Chooses which ready job runs next |
| **Interrupt** | A “please pay attention now!” signal |
| **Driver** | Software that talks to hardware |
| **Kernel** | Core of the operating system |
| **API** | A ready-made function/interface you can call |
| **Board** | The whole physical circuit board |
| **SoC** | A chip with CPU and other built-in parts |
| **Devicetree** | Description of hardware |
| **Kconfig** | Software feature selection system |
| **CMake** | Plans how to build code |
| **Ninja** | Performs planned build steps |
| **West** | Zephyr's project/build helper tool |
| **Toolchain** | Compiler and other tools for making firmware |
| **QEMU** | A pretend machine running on your PC |

---

## 12. What should we study after Hello World?

1. **`samples/hello_world/src/main.c`** — Find where execution begins.
2. **`samples/hello_world/CMakeLists.txt`** — See how Zephyr builds that source file.
3. **`samples/hello_world/prj.conf`** — Check which configuration options the sample uses, if any.
4. **`include/zephyr/kernel.h`** — Meet the thread and timer API.
5. **`kernel/thread.c` and `kernel/sched.c`** — Understand thread management and scheduling.
6. **`arch/`** — Later, learn how the CPU switches between threads.
7. **`drivers/`, `boards/`, `dts/`** — Finally, connect programs to real hardware.

### Remember these 5 sentences

1. **Bare metal** is good when the program is simple.
2. **An RTOS** helps organize multiple jobs and timing requirements.
3. **FreeRTOS and ThreadX are good RTOS choices, too.**
4. **Zephyr adds a broad shared ecosystem** of kernel, drivers, configuration, tools, and examples.
5. **Start tiny:** make Hello World work in QEMU, then learn one concept at a time.

---

## Trustworthy places to learn more

These are official project resources:

- [Why the Zephyr Project exists](https://www.zephyrproject.org/learn-about/)
- [Zephyr's 2016 project announcement](https://www.zephyrproject.org/the-linux-foundation-announces-project-to-build-real-time-operating-system-for-internet-of-things-devices/)
- [Zephyr source tree and directory guide](https://docs.zephyrproject.org/latest/contribute/guidelines.html#source-tree-structure)
- [Zephyr Getting Started on Windows](https://docs.zephyrproject.org/latest/develop/getting_started/index.html)
- [Zephyr Hello World sample](https://docs.zephyrproject.org/latest/samples/hello_world/README.html)
- [FreeRTOS project](https://www.freertos.org/)
- [Eclipse ThreadX FAQ](https://threadx.io/faq/)

**End of Chapter 01 — Introduction.** Next chapter idea: **`02 — How Zephyr boots and reaches main()`**.
