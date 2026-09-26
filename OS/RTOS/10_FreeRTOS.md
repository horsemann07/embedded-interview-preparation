# FreeRTOS Interview Questions
> FreeRTOS Specific — API, Internals, Porting, Debugging  
> M.Tech Graduate + 10 Years Experience  
> Basic → Intermediate → Advanced → Expert

---

## BASIC LEVEL

### Introduction

1. What is FreeRTOS?
   - Follow-up: Who created it? Current maintainer? (Richard Barry → Amazon AWS)
   - Follow-up: What is AWS FreeRTOS / Amazon FreeRTOS?
   - Follow-up: What is OpenRTOS and SafeRTOS relationship to FreeRTOS?

2. What license does FreeRTOS use?
   - Follow-up: MIT License since 2018 — what changed from GPL?

3. What platforms does FreeRTOS support?
   - Follow-up: ARM Cortex-M0/M3/M4/M7/M33/A, AVR, PIC, RISC-V, ESP32, STM32.

4. What is the FreeRTOS kernel folder structure?
   ```
   FreeRTOS/
   ├── Source/
   │   ├── tasks.c
   │   ├── queue.c
   │   ├── list.c
   │   ├── timers.c
   │   ├── event_groups.c
   │   ├── stream_buffer.c
   │   ├── croutine.c
   │   ├── include/
   │   └── portable/
   │       ├── GCC/ARM_CM4F/
   │       │   ├── port.c
   │       │   └── portmacro.h
   │       └── MemMang/
   │           ├── heap_1.c ... heap_5.c
   ```
   - Follow-up: What is `portable/` folder purpose?
   - Follow-up: What is `FreeRTOSConfig.h`?

---

### Tasks

5. How do you create a task in FreeRTOS?
   ```c
   BaseType_t xTaskCreate(
       TaskFunction_t pvTaskCode,
       const char * const pcName,
       configSTACK_DEPTH_TYPE usStackDepth,
       void *pvParameters,
       UBaseType_t uxPriority,
       TaskHandle_t *pxCreatedTask
   );
   ```
   - Follow-up: What does return value `pdPASS` vs `errCOULD_NOT_ALLOCATE_REQUIRED_MEMORY` mean?
   - Follow-up: What is `xTaskCreateStatic()`? When to use it?

6. How do you delete a task?
   ```c
   vTaskDelete(xHandle);  // delete other task
   vTaskDelete(NULL);     // delete self
   ```
   - Follow-up: Who frees the stack memory of deleted task?
   - Follow-up: What is the idle task's role in cleanup?

7. What is task priority in FreeRTOS?
   - Follow-up: Range: 0 (lowest) to `configMAX_PRIORITIES - 1` (highest).
   - Follow-up: What priority should idle task have? (0 — `tskIDLE_PRIORITY`)
   - Follow-up: What is `configMAX_PRIORITIES` and memory impact?

8. How do you suspend and resume a task?
   ```c
   vTaskSuspend(xHandle);
   vTaskResume(xHandle);
   xTaskResumeFromISR(xHandle);
   ```

9. Task states in FreeRTOS?
   - Running, Ready, Blocked, Suspended, Deleted
   - Follow-up: `eTaskGetState()` — what states does it return?

---

### Delays

10. Difference between `vTaskDelay()` and `vTaskDelayUntil()`?
    ```c
    // Relative delay — drifts over time
    vTaskDelay(pdMS_TO_TICKS(100));

    // Absolute delay — no drift, true periodic
    TickType_t xLastWakeTime = xTaskGetTickCount();
    vTaskDelayUntil(&xLastWakeTime, pdMS_TO_TICKS(100));
    ```
    - Follow-up: Why `vTaskDelayUntil()` is preferred for periodic tasks?
    - Follow-up: What is `pdMS_TO_TICKS()` macro?

---

### Queues

11. What is a FreeRTOS queue?
    ```c
    QueueHandle_t xQueueCreate(UBaseType_t uxQueueLength, UBaseType_t uxItemSize);
    xQueueSend(xQueue, &data, xTicksToWait);
    xQueueReceive(xQueue, &data, xTicksToWait);
    ```
    - Follow-up: What is `xTicksToWait = portMAX_DELAY`?
    - Follow-up: What is `xQueueSendToFront()` vs `xQueueSendToBack()`?
    - Follow-up: What is `xQueuePeek()`?

12. What is queue set in FreeRTOS?
    - Follow-up: When do you use queue sets?

---

## INTERMEDIATE LEVEL

### Semaphores & Mutexes

1. FreeRTOS semaphore types:
   - Binary semaphore
   - Counting semaphore
   - Mutex
   - Recursive mutex
   ```c
   SemaphoreHandle_t xSemaphoreCreateBinary();
   SemaphoreHandle_t xSemaphoreCreateCounting(uxMaxCount, uxInitialCount);
   SemaphoreHandle_t xSemaphoreCreateMutex();
   SemaphoreHandle_t xSemaphoreCreateRecursiveMutex();
   ```

2. How are semaphores implemented internally in FreeRTOS?
   - Follow-up: Semaphores are implemented as queues of length 1 (binary) or N (counting).

3. Does FreeRTOS mutex support priority inheritance?
   - Follow-up: Yes — standard mutex only, NOT binary semaphore.
   - Follow-up: Why binary semaphore must NOT be used for mutual exclusion?

4. What is recursive mutex and when is it needed?
   ```c
   xSemaphoreTakeRecursive(xMutex, portMAX_DELAY);
   // ... nested call that also takes same mutex
   xSemaphoreGiveRecursive(xMutex);
   ```

5. ISR-safe semaphore API:
   ```c
   xSemaphoreGiveFromISR(xSemaphore, &xHigherPriorityTaskWoken);
   xSemaphoreTakeFromISR(xSemaphore, &xHigherPriorityTaskWoken);
   portYIELD_FROM_ISR(xHigherPriorityTaskWoken);
   ```
   - Follow-up: Why is `portYIELD_FROM_ISR()` important?
   - Follow-up: What happens if you forget it?

---

### Event Groups

6. What are event groups in FreeRTOS?
   ```c
   EventGroupHandle_t xEventGroupCreate();
   xEventGroupSetBits(xEventGroup, BIT_0 | BIT_1);
   xEventGroupWaitBits(xEventGroup, BIT_0|BIT_1, pdTRUE, pdTRUE, portMAX_DELAY);
   ```
   - Follow-up: AND wait vs OR wait (`xWaitForAllBits` parameter).
   - Follow-up: `xEventGroupSetBitsFromISR()` limitations.
   - Follow-up: Why event group bits are limited to 24 bits (8 bits reserved)?

---

### Software Timers

7. What is a software timer in FreeRTOS?
   ```c
   TimerHandle_t xTimerCreate(
       pcTimerName, xTimerPeriodInTicks,
       uxAutoReload, pvTimerID, pxCallbackFunction
   );
   xTimerStart(xTimer, xTicksToWait);
   xTimerStop(xTimer, xTicksToWait);
   xTimerReset(xTimer, xTicksToWait);
   ```
   - Follow-up: One-shot vs auto-reload timer (`uxAutoReload`).
   - Follow-up: Timer daemon task — what is it? Default priority?
   - Follow-up: What is `configTIMER_TASK_PRIORITY`?
   - Follow-up: Why must timer callback be short and non-blocking?

---

### Task Notifications

8. What are task notifications in FreeRTOS?
   ```c
   xTaskNotify(xTaskHandle, ulValue, eAction);
   xTaskNotifyWait(ulBitsToClearOnEntry, ulBitsToClearOnExit, &ulValue, xTicksToWait);
   ulTaskNotifyTake(pdTRUE, portMAX_DELAY); // simple semaphore-like
   xTaskNotifyGive(xTaskHandle);
   ```
   - Follow-up: Task notifications vs semaphore — performance difference?
   - Follow-up: `eTaskNotifyAction` options: `eSetBits`, `eIncrement`, `eSetValueWithOverwrite`, `eSetValueWithoutOverwrite`.
   - Follow-up: Limitation — only one notification per task.
   - Follow-up: `xTaskNotifyFromISR()` and `vTaskNotifyGiveFromISR()`.

---

### Stream Buffers & Message Buffers

9. What is a stream buffer?
   ```c
   StreamBufferHandle_t xStreamBufferCreate(xBufferSizeBytes, xTriggerLevelBytes);
   xStreamBufferSend(xStreamBuffer, pvTxData, xDataLengthBytes, xTicksToWait);
   xStreamBufferReceive(xStreamBuffer, pvRxData, xBufferLengthBytes, xTicksToWait);
   ```
   - Follow-up: Difference between stream buffer and message buffer.
   - Follow-up: Stream buffer is designed for single sender + single receiver only.
   - Follow-up: Why is stream buffer more efficient than queue for byte streams?

10. What is a message buffer?
    - Follow-up: How does it preserve message boundaries (stream buffer does not)?

---

### Memory Management

11. FreeRTOS heap schemes:
    | Scheme | Free? | Coalesce? | Use Case |
    |--------|-------|-----------|----------|
    | heap_1 | No | No | Simplest, safety-critical |
    | heap_2 | Yes | No | Legacy, fragmentation risk |
    | heap_3 | Yes | Yes | Wraps malloc/free |
    | heap_4 | Yes | Yes | Most common, good general use |
    | heap_5 | Yes | Yes | Multiple memory regions |

    - Follow-up: Why is heap_1 used in safety-critical? (no free = no fragmentation)
    - Follow-up: How does heap_4 coalesce adjacent free blocks?
    - Follow-up: How to use heap_5 for SRAM + CCMRAM on STM32?
    ```c
    HeapRegion_t xHeapRegions[] = {
        { (uint8_t*)0x20000000, 0x10000 },  // SRAM1
        { (uint8_t*)0x10000000, 0x8000  },  // CCMRAM
        { NULL, 0 }
    };
    vPortDefineHeapRegions(xHeapRegions);
    ```

12. Static vs Dynamic allocation:
    ```c
    // Dynamic
    xTaskCreate(vTask, "T", 128, NULL, 1, NULL);

    // Static — no heap needed
    StaticTask_t xTaskBuffer;
    StackType_t xStack[128];
    xTaskCreateStatic(vTask, "T", 128, NULL, 1, xStack, &xTaskBuffer);
    ```
    - Follow-up: `configSUPPORT_STATIC_ALLOCATION` and `configSUPPORT_DYNAMIC_ALLOCATION`.
    - Follow-up: Why static allocation is mandatory for some safety standards?

---

### Stack Overflow Detection

13. FreeRTOS stack overflow detection methods:
    ```c
    // FreeRTOSConfig.h
    #define configCHECK_FOR_STACK_OVERFLOW  2
    ```
    - Method 1: Check SP against stack boundary at context switch.
    - Method 2: Fill stack with known pattern (0xA5), check last 16 bytes.
    - Follow-up: Which is more reliable?
    - Follow-up: Stack overflow hook:
    ```c
    void vApplicationStackOverflowHook(TaskHandle_t xTask, char *pcTaskName) {
        // Handle overflow — usually halt or reset
    }
    ```
    - Follow-up: `uxTaskGetStackHighWaterMark(xHandle)` — how to use for sizing.

---

## ADVANCED LEVEL

### FreeRTOS Internals

1. How does FreeRTOS ready list work?
   - Follow-up: Array of lists indexed by priority.
   - Follow-up: `pxReadyTasksLists[configMAX_PRIORITIES]`.
   - Follow-up: `pxCurrentTCB` — pointer to running task TCB.

2. How does FreeRTOS scheduler select next task?
   - Follow-up: `taskSELECT_HIGHEST_PRIORITY_TASK()` macro.
   - Follow-up: Generic vs port-optimized task selection.
   - Follow-up: `configUSE_PORT_OPTIMISED_TASK_SELECTION` — uses CLZ instruction.

3. How does `vTaskSwitchContext()` work?
   - Follow-up: Where is it called from? (PendSV handler on Cortex-M)

4. What is the idle task in FreeRTOS?
   - Follow-up: Priority 0 — always ready.
   - Follow-up: Idle task frees memory of deleted tasks.
   - Follow-up: Idle hook: `vApplicationIdleHook()`.
   - Follow-up: Can you block in idle hook? (No — must return promptly)

5. Tickless idle mode internals:
   ```c
   #define configUSE_TICKLESS_IDLE  1
   ```
   - Follow-up: `vPortSuppressTicksAndSleep()` — what does it do?
   - Follow-up: `vTaskStepTick()` — corrects tick count after sleep.
   - Follow-up: How does RTOS resume after deep sleep?

---

### ISR Handling in FreeRTOS

6. Rules for calling FreeRTOS API from ISR:
   - Only use `FromISR` variants.
   - `configMAX_SYSCALL_INTERRUPT_PRIORITY` — highest ISR priority that can call API.
   - ISRs above this priority MUST NOT call any FreeRTOS API.
   - Follow-up: What happens if you violate this rule?
   - Follow-up: On Cortex-M: lower numerical value = higher priority.

7. `portYIELD_FROM_ISR()` pattern:
   ```c
   void UART_ISR(void) {
       BaseType_t xHigherPriorityTaskWoken = pdFALSE;
       xQueueSendFromISR(xQueue, &data, &xHigherPriorityTaskWoken);
       portYIELD_FROM_ISR(xHigherPriorityTaskWoken);
   }
   ```
   - Follow-up: What does this ensure?
   - Follow-up: What if `xHigherPriorityTaskWoken` is ignored?

---

### Porting FreeRTOS

8. Files required to port FreeRTOS:
   - `port.c` — context switch, tick ISR, critical sections.
   - `portmacro.h` — types, macros, `portYIELD`, `portDISABLE_INTERRUPTS`.
   - `FreeRTOSConfig.h` — application configuration.
   - Follow-up: Minimum hardware needed: 1 timer (SysTick), stack pointer register.

9. ARM Cortex-M FreeRTOS port:
   - SysTick → tick interrupt → `xPortSysTickHandler()`.
   - PendSV → context switch → `xPortPendSVHandler()`.
   - SVC → first task start → `vPortSVCHandler()`.
   - Follow-up: Why PendSV has lowest priority on Cortex-M?
   - Follow-up: What registers does Cortex-M hardware stack on exception entry?
     (xPSR, PC, LR, R12, R3, R2, R1, R0)
   - Follow-up: What does FreeRTOS port additionally save?
     (R4–R11, and FPU registers if enabled)

10. FPU support in FreeRTOS on Cortex-M4F/M7:
    ```c
    #define configUSE_TASK_FPU_SUPPORT  1  // or 2 for all tasks
    ```
    - Follow-up: Lazy vs eager FPU stacking.
    - Follow-up: What is `FPSCR` register and why must it be saved per task?

---

### FreeRTOS Configuration Deep Dive

11. Critical `FreeRTOSConfig.h` parameters:

    | Parameter | Purpose |
    |-----------|---------|
    | `configCPU_CLOCK_HZ` | CPU frequency for tick calculation |
    | `configTICK_RATE_HZ` | Tick frequency (e.g., 1000 = 1ms tick) |
    | `configMAX_PRIORITIES` | Number of priority levels |
    | `configMINIMAL_STACK_SIZE` | Idle task stack size |
    | `configTOTAL_HEAP_SIZE` | Heap size for heap_1 to heap_4 |
    | `configUSE_PREEMPTION` | 1=preemptive, 0=cooperative |
    | `configUSE_TIME_SLICING` | Round-robin for equal priority tasks |
    | `configUSE_IDLE_HOOK` | Enable idle hook function |
    | `configUSE_TICK_HOOK` | Enable tick hook function |
    | `configUSE_TRACE_FACILITY` | Enable trace macros |
    | `configUSE_STATS_FORMATTING_FUNCTIONS` | Enable `vTaskList()` |
    | `configGENERATE_RUN_TIME_STATS` | Enable CPU usage stats |
    | `configUSE_MUTEXES` | Enable mutex support |
    | `configUSE_RECURSIVE_MUTEXES` | Enable recursive mutex |
    | `configUSE_COUNTING_SEMAPHORES` | Enable counting semaphore |
    | `configUSE_TASK_NOTIFICATIONS` | Enable task notifications |
    | `configUSE_STREAM_BUFFERS` | Enable stream buffers |
    | `configSUPPORT_STATIC_ALLOCATION` | Static object creation |
    | `configCHECK_FOR_STACK_OVERFLOW` | Stack overflow detection |
    | `configUSE_MALLOC_FAILED_HOOK` | malloc failure hook |
    | `configUSE_TICKLESS_IDLE` | Low power tickless mode |
    | `configMAX_SYSCALL_INTERRUPT_PRIORITY` | ISR safe priority threshold |

---

### Debugging FreeRTOS

12. Runtime task inspection:
    ```c
    vTaskList(pcWriteBuffer);        // task name, state, priority, stack, task number
    vTaskGetRunTimeStats(pcBuffer);  // CPU usage per task
    uxTaskGetStackHighWaterMark(xTask); // remaining stack words
    eTaskGetState(xTask);            // current state
    pcTaskGetName(xTask);            // task name
    uxTaskPriorityGet(xTask);        // current priority
    ```

13. Heap monitoring:
    ```c
    xPortGetFreeHeapSize();          // current free heap
    xPortGetMinimumEverFreeHeapSize(); // historical minimum — heap high watermark
    ```

14. SEGGER SystemView integration with FreeRTOS:
    - Follow-up: What events does it record? (task switch, ISR enter/exit, semaphore take/give)
    - Follow-up: How does it timestamp events?

15. Percepio Tracealyzer:
    - Follow-up: Difference between snapshot trace and streaming trace.

16. GDB RTOS awareness:
    - Follow-up: OpenOCD FreeRTOS plugin — how to inspect all tasks.

---

## EXPERT LEVEL

### FreeRTOS MPU Port

1. What is FreeRTOS MPU port?
   - Follow-up: `xTaskCreateRestricted()` for MPU-protected tasks.
   - Follow-up: `TaskParameters_t` — define memory regions per task.
   ```c
   TaskParameters_t xTaskParameters = {
       .pvTaskCode = vTask,
       .pcName = "RestrictedTask",
       .usStackDepth = 128,
       .pvParameters = NULL,
       .uxPriority = 1 | portPRIVILEGE_BIT,
       .puxStackBuffer = xTaskStack,
       .xRegions = {
           { ucSharedMemory, 32, portMPU_REGION_READ_WRITE },
           { 0, 0, 0 },
           { 0, 0, 0 }
       }
   };
   ```

2. Privileged vs unprivileged tasks in FreeRTOS MPU port.
   - Follow-up: `portPRIVILEGE_BIT` in priority field.
   - Follow-up: Syscall mechanism for unprivileged tasks.

---

### SMP FreeRTOS (FreeRTOS-SMP)

3. What is FreeRTOS SMP kernel?
   - Follow-up: Available since FreeRTOS V11.
   - Follow-up: `configNUM_CORES` — number of cores.
   - Follow-up: `configUSE_CORE_AFFINITY` — pin task to core.
   ```c
   vTaskCoreAffinitySet(xHandle, (1 << 0));  // pin to core 0
   ```
4. How does FreeRTOS SMP handle scheduler lock?
   - Follow-up: Per-core vs global scheduler lock.
5. Porting FreeRTOS SMP to RP2040 (dual Cortex-M0+).

---

### FreeRTOS + POSIX

6. What is FreeRTOS-POSIX layer?
   - Follow-up: Implements `pthread`, `sem_t`, `mq_t` on top of FreeRTOS.
   - Follow-up: Why is it useful for porting POSIX applications to embedded?

---

### Advanced Coding Questions

7. Implement producer-consumer with queue:
   ```c
   // Producer Task
   void vProducerTask(void *pvParams) {
       uint32_t data = 0;
       while(1) {
           data++;
           xQueueSend(xQueue, &data, portMAX_DELAY);
           vTaskDelay(pdMS_TO_TICKS(100));
       }
   }

   // Consumer Task
   void vConsumerTask(void *pvParams) {
       uint32_t received;
       while(1) {
           xQueueReceive(xQueue, &received, portMAX_DELAY);
           // process received
       }
   }
   ```

8. Implement ISR → task signaling via binary semaphore:
   ```c
   void EXTI_IRQHandler(void) {
       BaseType_t xHigherPriorityTaskWoken = pdFALSE;
       xSemaphoreGiveFromISR(xBinSem, &xHigherPriorityTaskWoken);
       portYIELD_FROM_ISR(xHigherPriorityTaskWoken);
   }

   void vHandlerTask(void *pvParams) {
       while(1) {
           xSemaphoreTake(xBinSem, portMAX_DELAY);
           // process event
       }
   }
   ```

9. Implement mutex-protected shared resource:
   ```c
   void vTask(void *pvParams) {
       while(1) {
           if(xSemaphoreTake(xMutex, pdMS_TO_TICKS(10)) == pdTRUE) {
               // access shared resource
               xSemaphoreGive(xMutex);
           }
       }
   }
   ```

10. Implement state machine across FreeRTOS tasks using event groups.

11. Implement a thread-safe ring buffer using FreeRTOS stream buffer.

12. Implement software watchdog using FreeRTOS timer.

---

## FREERTOS API QUICK REFERENCE

### Task API
| Function | Description |
|----------|-------------|
| `xTaskCreate()` | Create task dynamically |
| `xTaskCreateStatic()` | Create task statically |
| `vTaskDelete()` | Delete task |
| `vTaskDelay()` | Relative delay |
| `vTaskDelayUntil()` | Absolute periodic delay |
| `vTaskSuspend()` | Suspend task |
| `vTaskResume()` | Resume task |
| `xTaskResumeFromISR()` | Resume from ISR |
| `uxTaskPriorityGet()` | Get priority |
| `vTaskPrioritySet()` | Set priority |
| `xTaskGetTickCount()` | Get current tick |
| `uxTaskGetStackHighWaterMark()` | Stack usage |

### Queue API
| Function | Description |
|----------|-------------|
| `xQueueCreate()` | Create queue |
| `xQueueSend()` | Send to back |
| `xQueueSendToFront()` | Send to front |
| `xQueueReceive()` | Receive and remove |
| `xQueuePeek()` | Receive without remove |
| `uxQueueMessagesWaiting()` | Items in queue |
| `xQueueSendFromISR()` | Send from ISR |
| `xQueueReceiveFromISR()` | Receive from ISR |

### Semaphore / Mutex API
| Function | Description |
|----------|-------------|
| `xSemaphoreCreateBinary()` | Binary semaphore |
| `xSemaphoreCreateCounting()` | Counting semaphore |
| `xSemaphoreCreateMutex()` | Mutex with PI |
| `xSemaphoreCreateRecursiveMutex()` | Recursive mutex |
| `xSemaphoreTake()` | Acquire |
| `xSemaphoreGive()` | Release |
| `xSemaphoreTakeFromISR()` | Acquire from ISR |
| `xSemaphoreGiveFromISR()` | Release from ISR |

---

## SUMMARY TABLE

| FreeRTOS Topic | Basic | Intermediate | Advanced | Expert |
|----------------|-------|-------------|----------|--------|
| Task management | ★★★★★ | ★★★★★ | ★★★★☆ | ★★★☆☆ |
| Queue / IPC | ★★★★☆ | ★★★★★ | ★★★★☆ | ★★★☆☆ |
| Semaphore/Mutex | ★★★★☆ | ★★★★★ | ★★★★★ | ★★★★☆ |
| Task notifications | ★★★☆☆ | ★★★★★ | ★★★★☆ | ★★★☆☆ |
| Event groups | ★★★☆☆ | ★★★★☆ | ★★★★☆ | ★★★☆☆ |
| Timers | ★★★★☆ | ★★★★★ | ★★★★☆ | ★★★☆☆ |
| Memory / Heap | ★★★☆☆ | ★★★★★ | ★★★★★ | ★★★★★ |
| Porting | ★★☆☆☆ | ★★★☆☆ | ★★★★★ | ★★★★★ |
| ISR integration | ★★★☆☆ | ★★★★★ | ★★★★★ | ★★★★☆ |
| MPU port | ★☆☆☆☆ | ★★☆☆☆ | ★★★★☆ | ★★★★★ |
| SMP port | ★☆☆☆☆ | ★★☆☆☆ | ★★★★☆ | ★★★★★ |
| Debugging/Trace | ★★★☆☆ | ★★★★☆ | ★★★★★ | ★★★★★ |

---

*FreeRTOS-specific interview document — covers API to internals to porting, suitable for 10+ years embedded engineer.*