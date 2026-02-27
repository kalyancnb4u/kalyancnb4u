---
title: "Complete C Language Mastery Part 8: Embedded Systems Programming"
date: 2024-02-21 17:00:00 +0000
categories: [C, Programming, Embedded, Systems]
tags: [c-language, embedded-systems, microcontrollers, bare-metal, rtos, freertos, device-drivers, hardware]
---

# Complete C Language Mastery Part 8: Embedded Systems Programming

Welcome to Part 8 of the C Language Mastery series. This part explores embedded systems programming—writing C code that runs directly on hardware without a traditional operating system, or with lightweight real-time operating systems (RTOS). This is where C truly shines, powering everything from microcontrollers to automotive systems.

**Prerequisites**: Parts 1-7 (especially memory management and platform-specific programming)

**Note**: Examples use ARM Cortex-M architecture (common in embedded systems) but concepts apply broadly.

---

## 8.1 Embedded C Fundamentals

### Constraints of Embedded Systems

**Limited Resources:**
```c
/* Typical microcontroller constraints:
   - RAM: 8KB - 512KB (vs GB on desktop)
   - Flash: 32KB - 2MB (vs GB on desktop)
   - CPU: 16-200 MHz (vs GHz on desktop)
   - No dynamic memory allocation (often)
   - No floating-point unit (often)
   - Real-time requirements
   - Power constraints
*/

/* ✅ Embedded-friendly code */
#define BUFFER_SIZE 256
static uint8_t buffer[BUFFER_SIZE];  /* Static allocation */
static uint16_t buffer_index = 0;

/* ❌ Avoid on embedded */
void *ptr = malloc(1024);  /* Dynamic allocation - risky */
double result = 3.14 * 2.5;  /* Slow without FPU */
```

**Code Optimization:**
```c
/* Size optimization flags: -Os */
/* Speed optimization: -O2 or -O3 */
/* Link-time optimization: -flto */

/* Code size matters */
/* ❌ Bloated */
#include <stdio.h>  /* Pulls in lots of code */
printf("Value: %d\n", x);

/* ✅ Minimal */
void uart_print_int(int value);  /* Custom lightweight function */
uart_print_int(x);
```

### Memory-Mapped I/O

**Hardware registers accessed as memory addresses:**

```c
#include <stdint.h>

/* Memory-mapped registers (ARM Cortex-M example) */
#define GPIOA_BASE   0x40020000
#define RCC_BASE     0x40023800

/* GPIO Port A registers */
#define GPIOA_MODER   (*(volatile uint32_t *)(GPIOA_BASE + 0x00))
#define GPIOA_ODR     (*(volatile uint32_t *)(GPIOA_BASE + 0x14))
#define GPIOA_IDR     (*(volatile uint32_t *)(GPIOA_BASE + 0x10))

/* RCC (Reset and Clock Control) */
#define RCC_AHB1ENR   (*(volatile uint32_t *)(RCC_BASE + 0x30))

/* Enable GPIO Port A clock */
void init_gpio(void) {
    RCC_AHB1ENR |= (1 << 0);  /* Enable GPIOA clock */
    
    /* Configure PA5 as output */
    GPIOA_MODER &= ~(3 << (5 * 2));  /* Clear bits */
    GPIOA_MODER |= (1 << (5 * 2));   /* Set as output */
}

/* Toggle LED on PA5 */
void toggle_led(void) {
    GPIOA_ODR ^= (1 << 5);
}

/* Read button on PA0 */
uint8_t read_button(void) {
    return (GPIOA_IDR & (1 << 0)) ? 1 : 0;
}
```

**Structured register access:**

```c
/* Better approach: Structure for registers */
typedef struct {
    volatile uint32_t MODER;    /* Mode register */
    volatile uint32_t OTYPER;   /* Output type register */
    volatile uint32_t OSPEEDR;  /* Speed register */
    volatile uint32_t PUPDR;    /* Pull-up/pull-down register */
    volatile uint32_t IDR;      /* Input data register */
    volatile uint32_t ODR;      /* Output data register */
    volatile uint32_t BSRR;     /* Bit set/reset register */
    volatile uint32_t LCKR;     /* Lock register */
    volatile uint32_t AFR[2];   /* Alternate function registers */
} GPIO_TypeDef;

#define GPIOA ((GPIO_TypeDef *)GPIOA_BASE)

/* Usage */
GPIOA->ODR |= (1 << 5);   /* Set bit 5 */
GPIOA->ODR &= ~(1 << 5);  /* Clear bit 5 */
uint32_t input = GPIOA->IDR;
```

### Volatile Keyword Usage

**Critical for hardware registers and ISR communication:**

```c
/* ❌ Without volatile - compiler may optimize away reads */
uint32_t *status_reg = (uint32_t *)0x40000000;
while (*status_reg == 0) {
    /* Compiler might read once and loop forever */
}

/* ✅ With volatile - forces read every time */
volatile uint32_t *status_reg = (volatile uint32_t *)0x40000000;
while (*status_reg == 0) {
    /* Reads from hardware each iteration */
}

/* Shared with ISR */
volatile uint8_t flag = 0;  /* Modified by ISR, read by main */

void interrupt_handler(void) {
    flag = 1;  /* ISR sets flag */
}

int main(void) {
    while (flag == 0) {
        /* Wait for interrupt */
    }
}
```

**When to use volatile:**
1. Hardware registers (always)
2. Variables modified by ISRs
3. Variables modified by other threads (but use atomics when possible)
4. Memory-mapped peripherals
5. DMA buffers

### Hardware Registers

**Bit manipulation is fundamental:**

```c
/* Bit operations */
#define BIT(n) (1U << (n))
#define SET_BIT(reg, bit)    ((reg) |= BIT(bit))
#define CLEAR_BIT(reg, bit)  ((reg) &= ~BIT(bit))
#define TOGGLE_BIT(reg, bit) ((reg) ^= BIT(bit))
#define READ_BIT(reg, bit)   (((reg) >> (bit)) & 1U)

/* Multi-bit fields */
#define FIELD_MASK(width, offset) (((1U << (width)) - 1) << (offset))
#define WRITE_FIELD(reg, field, value) \
    ((reg) = ((reg) & ~FIELD_MASK(field##_WIDTH, field##_OFFSET)) | \
             (((value) << field##_OFFSET) & FIELD_MASK(field##_WIDTH, field##_OFFSET)))

/* Example: Configure UART baud rate */
#define USART_BRR_OFFSET 0
#define USART_BRR_WIDTH  16

volatile uint32_t *USART1_BRR = (volatile uint32_t *)0x40011008;
WRITE_FIELD(*USART1_BRR, USART_BRR, 0x683);  /* 9600 baud @ 16MHz */
```

### Interrupt Service Routines (ISR)

**Fast, minimal functions:**

```c
/* ISR requirements:
   - Fast execution (microseconds)
   - No blocking calls
   - Minimal stack usage
   - No floating-point (saves context)
   - Communicate via volatile variables or queues
*/

volatile uint32_t tick_count = 0;

/* SysTick interrupt - every 1ms */
void SysTick_Handler(void) {
    tick_count++;  /* Simple counter */
}

/* UART receive interrupt */
#define RX_BUFFER_SIZE 64
volatile uint8_t rx_buffer[RX_BUFFER_SIZE];
volatile uint16_t rx_head = 0;
volatile uint16_t rx_tail = 0;

void USART1_IRQHandler(void) {
    if (USART1_SR & USART_SR_RXNE) {  /* Receive not empty */
        uint8_t data = USART1_DR;  /* Read data (clears flag) */
        
        uint16_t next = (rx_head + 1) % RX_BUFFER_SIZE;
        if (next != rx_tail) {  /* Buffer not full */
            rx_buffer[rx_head] = data;
            rx_head = next;
        }
    }
}

/* Main loop reads from buffer */
uint8_t uart_getchar(void) {
    while (rx_head == rx_tail) {  /* Buffer empty */
        /* Wait or sleep */
    }
    
    uint8_t data = rx_buffer[rx_tail];
    rx_tail = (rx_tail + 1) % RX_BUFFER_SIZE;
    return data;
}

/* Enable interrupt */
void enable_uart_interrupt(void) {
    NVIC_EnableIRQ(USART1_IRQn);
    USART1_CR1 |= USART_CR1_RXNEIE;  /* RX not empty interrupt enable */
}
```

### Startup Code and Linker Scripts

**What happens before main():**

```c
/* startup.c - Typical ARM Cortex-M startup */

/* Vector table */
extern uint32_t _estack;  /* End of stack (from linker) */
extern void Reset_Handler(void);

__attribute__((section(".isr_vector")))
const uint32_t vectors[] = {
    (uint32_t)&_estack,      /* Initial stack pointer */
    (uint32_t)Reset_Handler, /* Reset handler */
    /* ... more interrupt vectors ... */
};

/* Reset handler - first code executed */
void Reset_Handler(void) {
    extern uint32_t _sdata, _edata, _sidata;  /* Data section */
    extern uint32_t _sbss, _ebss;              /* BSS section */
    
    /* Copy initialized data from flash to RAM */
    uint32_t *src = &_sidata;
    uint32_t *dst = &_sdata;
    while (dst < &_edata) {
        *dst++ = *src++;
    }
    
    /* Zero-initialize BSS */
    dst = &_sbss;
    while (dst < &_ebss) {
        *dst++ = 0;
    }
    
    /* Call static constructors (C++) */
    /* __libc_init_array(); */
    
    /* Call main */
    main();
    
    /* Should never return */
    while (1);
}
```

**Linker script (example):**

```ld
/* memory.ld */
MEMORY
{
    FLASH (rx)  : ORIGIN = 0x08000000, LENGTH = 256K
    RAM (rwx)   : ORIGIN = 0x20000000, LENGTH = 64K
}

SECTIONS
{
    .text :
    {
        KEEP(*(.isr_vector))  /* Vector table at start */
        *(.text*)             /* Code */
        *(.rodata*)           /* Read-only data */
    } > FLASH
    
    .data :
    {
        _sdata = .;
        *(.data*)
        _edata = .;
    } > RAM AT > FLASH
    
    .bss :
    {
        _sbss = .;
        *(.bss*)
        *(COMMON)
        _ebss = .;
    } > RAM
    
    _estack = ORIGIN(RAM) + LENGTH(RAM);
}
```

---

## 8.2 Bare-Metal Programming

### No Operating System Environment

**You control everything:**

```c
/* No malloc/free - use static allocation */
static uint8_t heap[4096];
static size_t heap_offset = 0;

void *simple_alloc(size_t size) {
    if (heap_offset + size > sizeof(heap)) {
        return NULL;
    }
    void *ptr = &heap[heap_offset];
    heap_offset += size;
    return ptr;
}

/* No printf - implement your own */
void uart_putc(char c) {
    while (!(USART1_SR & USART_SR_TXE));  /* Wait for TX empty */
    USART1_DR = c;
}

void uart_puts(const char *str) {
    while (*str) {
        uart_putc(*str++);
    }
}

void uart_print_hex(uint32_t value) {
    const char hex[] = "0123456789ABCDEF";
    uart_puts("0x");
    for (int i = 28; i >= 0; i -= 4) {
        uart_putc(hex[(value >> i) & 0xF]);
    }
}
```

### Initializing Hardware

**Clock configuration:**

```c
/* STM32 example: Configure system clock to 72 MHz */
void SystemClock_Config(void) {
    /* Enable HSE (external oscillator, 8 MHz) */
    RCC->CR |= RCC_CR_HSEON;
    while (!(RCC->CR & RCC_CR_HSERDY));  /* Wait for HSE ready */
    
    /* Configure PLL: HSE * 9 = 72 MHz */
    RCC->CFGR |= RCC_CFGR_PLLSRC_HSE | RCC_CFGR_PLLMULL9;
    
    /* Enable PLL */
    RCC->CR |= RCC_CR_PLLON;
    while (!(RCC->CR & RCC_CR_PLLRDY));  /* Wait for PLL */
    
    /* Configure flash latency for 72 MHz */
    FLASH->ACR |= FLASH_ACR_LATENCY_2;
    
    /* Select PLL as system clock */
    RCC->CFGR |= RCC_CFGR_SW_PLL;
    while ((RCC->CFGR & RCC_CFGR_SWS) != RCC_CFGR_SWS_PLL);
}
```

### GPIO Programming

**Complete GPIO control:**

```c
/* Configure pin as output */
void gpio_init_output(GPIO_TypeDef *port, uint8_t pin) {
    /* Enable clock */
    if (port == GPIOA) RCC->APB2ENR |= RCC_APB2ENR_IOPAEN;
    else if (port == GPIOB) RCC->APB2ENR |= RCC_APB2ENR_IOPBEN;
    
    /* Configure as push-pull output, 50 MHz */
    uint8_t pos = pin * 4;
    port->CRL &= ~(0xF << pos);
    port->CRL |= (0x3 << pos);  /* Output 50MHz, push-pull */
}

/* Set/clear pin */
void gpio_write(GPIO_TypeDef *port, uint8_t pin, uint8_t value) {
    if (value) {
        port->BSRR = (1 << pin);      /* Set */
    } else {
        port->BSRR = (1 << (pin + 16)); /* Reset */
    }
}

/* Toggle pin */
void gpio_toggle(GPIO_TypeDef *port, uint8_t pin) {
    port->ODR ^= (1 << pin);
}

/* Read input */
uint8_t gpio_read(GPIO_TypeDef *port, uint8_t pin) {
    return (port->IDR & (1 << pin)) ? 1 : 0;
}

/* Configure pin as input with pull-up */
void gpio_init_input_pullup(GPIO_TypeDef *port, uint8_t pin) {
    uint8_t pos = pin * 4;
    port->CRL &= ~(0xF << pos);
    port->CRL |= (0x8 << pos);  /* Input with pull-up/down */
    port->ODR |= (1 << pin);    /* Pull-up */
}
```

### Timers and PWM

**Hardware timers for precise timing:**

```c
/* Configure Timer 2 for 1kHz (1ms) interrupt */
void timer2_init(void) {
    RCC->APB1ENR |= RCC_APB1ENR_TIM2EN;  /* Enable clock */
    
    /* Assuming 72 MHz system clock, prescaler = 72 */
    TIM2->PSC = 72 - 1;     /* 72 MHz / 72 = 1 MHz */
    TIM2->ARR = 1000 - 1;   /* 1 MHz / 1000 = 1 kHz */
    
    TIM2->DIER |= TIM_DIER_UIE;  /* Update interrupt enable */
    TIM2->CR1 |= TIM_CR1_CEN;    /* Counter enable */
    
    NVIC_EnableIRQ(TIM2_IRQn);
}

void TIM2_IRQHandler(void) {
    if (TIM2->SR & TIM_SR_UIF) {
        TIM2->SR &= ~TIM_SR_UIF;  /* Clear flag */
        /* Called every 1ms */
    }
}

/* PWM generation */
void pwm_init(void) {
    /* TIM3 Channel 1 on PA6 */
    RCC->APB1ENR |= RCC_APB1ENR_TIM3EN;
    RCC->APB2ENR |= RCC_APB2ENR_IOPAEN;
    
    /* Configure PA6 as alternate function (TIM3_CH1) */
    GPIOA->CRL &= ~(0xF << 24);
    GPIOA->CRL |= (0xB << 24);  /* AF push-pull, 50MHz */
    
    /* Timer configuration */
    TIM3->PSC = 72 - 1;      /* 1 MHz */
    TIM3->ARR = 1000 - 1;    /* 1 kHz PWM frequency */
    
    /* PWM mode 1, enable preload */
    TIM3->CCMR1 |= TIM_CCMR1_OC1M_2 | TIM_CCMR1_OC1M_1 | TIM_CCMR1_OC1PE;
    TIM3->CCER |= TIM_CCER_CC1E;  /* Enable output */
    TIM3->CR1 |= TIM_CR1_CEN;     /* Enable timer */
}

/* Set duty cycle (0-1000) */
void pwm_set_duty(uint16_t duty) {
    TIM3->CCR1 = duty;  /* 0 = 0%, 500 = 50%, 1000 = 100% */
}
```

### ADC and DAC

**Analog interfacing:**

```c
/* ADC - Analog to Digital Converter */
void adc_init(void) {
    RCC->APB2ENR |= RCC_APB2ENR_ADC1EN;
    RCC->APB2ENR |= RCC_APB2ENR_IOPAEN;
    
    /* Configure PA0 as analog input */
    GPIOA->CRL &= ~(0xF << 0);
    
    /* ADC configuration */
    ADC1->SQR3 = 0;  /* Channel 0 */
    ADC1->SQR1 = 0;  /* 1 conversion */
    ADC1->CR2 |= ADC_CR2_ADON;  /* Enable ADC */
    
    /* Wait for ADC to stabilize */
    for (volatile int i = 0; i < 10000; i++);
}

uint16_t adc_read(void) {
    ADC1->CR2 |= ADC_CR2_ADON;  /* Start conversion */
    while (!(ADC1->SR & ADC_SR_EOC));  /* Wait for conversion */
    return ADC1->DR;  /* Read result (0-4095 for 12-bit ADC) */
}

/* DAC - Digital to Analog Converter */
void dac_init(void) {
    RCC->APB1ENR |= RCC_APB1ENR_DACEN;
    RCC->APB2ENR |= RCC_APB2ENR_IOPAEN;
    
    /* PA4 is DAC channel 1 */
    GPIOA->CRL &= ~(0xF << 16);
    
    DAC->CR |= DAC_CR_EN1;  /* Enable DAC channel 1 */
}

void dac_write(uint16_t value) {
    DAC->DHR12R1 = value & 0xFFF;  /* 12-bit value */
}
```

---

## 8.3 Real-Time Operating Systems (RTOS)

### RTOS Concepts

**Key concepts:**
- **Tasks**: Independent threads of execution
- **Priorities**: Higher priority tasks preempt lower
- **Scheduling**: Preemptive or cooperative
- **Synchronization**: Semaphores, mutexes, queues
- **Determinism**: Predictable timing

**Why use RTOS:**
- Manage multiple concurrent activities
- Priority-based execution
- Inter-task communication
- Time management
- Better than complex state machines

### FreeRTOS Basics

**Configuration (FreeRTOSConfig.h):**

```c
#define configUSE_PREEMPTION              1
#define configUSE_IDLE_HOOK               0
#define configUSE_TICK_HOOK               0
#define configCPU_CLOCK_HZ                72000000
#define configTICK_RATE_HZ                1000
#define configMAX_PRIORITIES              5
#define configMINIMAL_STACK_SIZE          128
#define configTOTAL_HEAP_SIZE             10240
#define configMAX_TASK_NAME_LEN           16
```

### Task Management

```c
#include "FreeRTOS.h"
#include "task.h"

/* Task function */
void vTaskLED(void *pvParameters) {
    while (1) {
        gpio_toggle(GPIOA, 5);  /* Toggle LED */
        vTaskDelay(pdMS_TO_TICKS(500));  /* Delay 500ms */
    }
}

/* Create task */
int main(void) {
    /* Hardware initialization */
    SystemClock_Config();
    gpio_init_output(GPIOA, 5);
    
    /* Create task */
    xTaskCreate(
        vTaskLED,          /* Function */
        "LED",             /* Name */
        128,               /* Stack size (words) */
        NULL,              /* Parameters */
        tskIDLE_PRIORITY + 1,  /* Priority */
        NULL               /* Handle */
    );
    
    /* Start scheduler */
    vTaskStartScheduler();
    
    /* Should never reach here */
    while (1);
}

/* Multiple tasks */
void vTaskUART(void *pvParameters) {
    while (1) {
        if (uart_available()) {
            char c = uart_getchar();
            uart_putchar(c);  /* Echo */
        }
        vTaskDelay(pdMS_TO_TICKS(10));
    }
}

void vTaskSensor(void *pvParameters) {
    while (1) {
        uint16_t value = adc_read();
        /* Process sensor value */
        vTaskDelay(pdMS_TO_TICKS(100));
    }
}
```

### Queues and Semaphores

**Queues for inter-task communication:**

```c
#include "queue.h"

QueueHandle_t xQueue;

void vProducerTask(void *pvParameters) {
    uint32_t value = 0;
    
    while (1) {
        /* Send data to queue */
        if (xQueueSend(xQueue, &value, portMAX_DELAY) == pdPASS) {
            value++;
        }
        vTaskDelay(pdMS_TO_TICKS(1000));
    }
}

void vConsumerTask(void *pvParameters) {
    uint32_t received;
    
    while (1) {
        /* Receive from queue */
        if (xQueueReceive(xQueue, &received, portMAX_DELAY) == pdPASS) {
            printf("Received: %lu\n", received);
        }
    }
}

int main(void) {
    /* Create queue (10 items of uint32_t) */
    xQueue = xQueueCreate(10, sizeof(uint32_t));
    
    xTaskCreate(vProducerTask, "Producer", 128, NULL, 2, NULL);
    xTaskCreate(vConsumerTask, "Consumer", 128, NULL, 1, NULL);
    
    vTaskStartScheduler();
    while (1);
}
```

**Semaphores for synchronization:**

```c
#include "semphr.h"

SemaphoreHandle_t xBinarySemaphore;

/* ISR gives semaphore */
void UART_IRQHandler(void) {
    BaseType_t xHigherPriorityTaskWoken = pdFALSE;
    
    /* Handle interrupt */
    
    /* Signal task */
    xSemaphoreGiveFromISR(xBinarySemaphore, &xHigherPriorityTaskWoken);
    portYIELD_FROM_ISR(xHigherPriorityTaskWoken);
}

/* Task waits for semaphore */
void vHandlerTask(void *pvParameters) {
    while (1) {
        /* Wait for ISR signal */
        if (xSemaphoreTake(xBinarySemaphore, portMAX_DELAY) == pdTRUE) {
            /* Process data */
        }
    }
}

int main(void) {
    xBinarySemaphore = xSemaphoreCreateBinary();
    xTaskCreate(vHandlerTask, "Handler", 128, NULL, 3, NULL);
    vTaskStartScheduler();
    while (1);
}
```

---

## 8.4 Device Drivers

### Driver Architecture

**Layered approach:**

```
Application
    ↓
HAL (Hardware Abstraction Layer)
    ↓
Driver (character/block)
    ↓
Hardware Registers
```

**Driver structure:**

```c
/* led_driver.h */
typedef struct {
    GPIO_TypeDef *port;
    uint8_t pin;
    uint8_t state;
} LED_Driver;

void LED_Init(LED_Driver *led, GPIO_TypeDef *port, uint8_t pin);
void LED_On(LED_Driver *led);
void LED_Off(LED_Driver *led);
void LED_Toggle(LED_Driver *led);
uint8_t LED_GetState(LED_Driver *led);

/* led_driver.c */
void LED_Init(LED_Driver *led, GPIO_TypeDef *port, uint8_t pin) {
    led->port = port;
    led->pin = pin;
    led->state = 0;
    gpio_init_output(port, pin);
    LED_Off(led);
}

void LED_On(LED_Driver *led) {
    gpio_write(led->port, led->pin, 1);
    led->state = 1;
}

void LED_Off(LED_Driver *led) {
    gpio_write(led->port, led->pin, 0);
    led->state = 0;
}
```

### Character Devices

**UART driver example:**

```c
/* uart_driver.h */
typedef struct {
    USART_TypeDef *usart;
    uint8_t tx_buffer[256];
    volatile uint16_t tx_head, tx_tail;
    uint8_t rx_buffer[256];
    volatile uint16_t rx_head, rx_tail;
} UART_Driver;

void UART_Init(UART_Driver *uart, USART_TypeDef *usart, uint32_t baudrate);
int UART_Write(UART_Driver *uart, const uint8_t *data, uint16_t len);
int UART_Read(UART_Driver *uart, uint8_t *data, uint16_t len);
int UART_Available(UART_Driver *uart);

/* uart_driver.c */
void UART_Init(UART_Driver *uart, USART_TypeDef *usart, uint32_t baudrate) {
    uart->usart = usart;
    uart->tx_head = uart->tx_tail = 0;
    uart->rx_head = uart->rx_tail = 0;
    
    /* Configure hardware */
    /* ... clock, pins, baud rate ... */
    
    /* Enable interrupts */
    usart->CR1 |= USART_CR1_RXNEIE;  /* RX not empty */
    usart->CR1 |= USART_CR1_UE;      /* USART enable */
}

int UART_Write(UART_Driver *uart, const uint8_t *data, uint16_t len) {
    for (uint16_t i = 0; i < len; i++) {
        uint16_t next = (uart->tx_head + 1) % 256;
        while (next == uart->tx_tail);  /* Wait if buffer full */
        
        uart->tx_buffer[uart->tx_head] = data[i];
        uart->tx_head = next;
        
        uart->usart->CR1 |= USART_CR1_TXEIE;  /* Enable TX interrupt */
    }
    return len;
}

/* ISR */
void USART1_IRQHandler(void) {
    UART_Driver *uart = &uart1_instance;
    
    /* RX */
    if (uart->usart->SR & USART_SR_RXNE) {
        uint8_t data = uart->usart->DR;
        uint16_t next = (uart->rx_head + 1) % 256;
        if (next != uart->rx_tail) {
            uart->rx_buffer[uart->rx_head] = data;
            uart->rx_head = next;
        }
    }
    
    /* TX */
    if (uart->usart->SR & USART_SR_TXE) {
        if (uart->tx_head != uart->tx_tail) {
            uart->usart->DR = uart->tx_buffer[uart->tx_tail];
            uart->tx_tail = (uart->tx_tail + 1) % 256;
        } else {
            uart->usart->CR1 &= ~USART_CR1_TXEIE;  /* Disable TX interrupt */
        }
    }
}
```

### Polling vs Interrupts vs DMA

**Polling (simple, CPU-intensive):**

```c
void uart_send_polling(const char *str) {
    while (*str) {
        while (!(USART1->SR & USART_SR_TXE));  /* Wait */
        USART1->DR = *str++;
    }
}
/* ❌ CPU blocked waiting */
```

**Interrupts (CPU-efficient, moderate complexity):**

```c
/* See UART_Write above - uses interrupts */
/* ✅ CPU free to do other work */
```

**DMA (most efficient, complex setup):**

```c
void uart_send_dma(const uint8_t *data, uint16_t len) {
    /* Configure DMA */
    DMA1_Channel4->CMAR = (uint32_t)data;  /* Memory address */
    DMA1_Channel4->CPAR = (uint32_t)&USART1->DR;  /* Peripheral address */
    DMA1_Channel4->CNDTR = len;  /* Number of bytes */
    
    /* Enable DMA */
    DMA1_Channel4->CCR |= DMA_CCR_EN;
    USART1->CR3 |= USART_CR3_DMAT;  /* DMA transmit enable */
}

void DMA1_Channel4_IRQHandler(void) {
    if (DMA1->ISR & DMA_ISR_TCIF4) {  /* Transfer complete */
        DMA1->IFCR = DMA_IFCR_CTCIF4;  /* Clear flag */
        /* Transfer done */
    }
}
/* ✅ CPU completely free, hardware transfers data */
```

---

## 8.5 Communication Protocols

### UART/Serial Communication

```c
/* Initialize UART */
void uart_init(uint32_t baudrate) {
    /* Enable clocks */
    RCC->APB2ENR |= RCC_APB2ENR_USART1EN | RCC_APB2ENR_IOPAEN;
    
    /* Configure pins (PA9=TX, PA10=RX) */
    GPIOA->CRH &= ~(0xFF << 4);
    GPIOA->CRH |= (0x0B << 4);   /* PA9: AF push-pull */
    GPIOA->CRH |= (0x04 << 8);   /* PA10: floating input */
    
    /* Configure UART */
    USART1->BRR = SystemCoreClock / baudrate;  /* Baud rate */
    USART1->CR1 = USART_CR1_TE | USART_CR1_RE | USART_CR1_UE;
}

void uart_putchar(char c) {
    while (!(USART1->SR & USART_SR_TXE));
    USART1->DR = c;
}

char uart_getchar(void) {
    while (!(USART1->SR & USART_SR_RXNE));
    return USART1->DR;
}
```

### I2C Protocol

```c
/* I2C master write */
void i2c_write(uint8_t addr, uint8_t reg, uint8_t data) {
    /* Start condition */
    I2C1->CR1 |= I2C_CR1_START;
    while (!(I2C1->SR1 & I2C_SR1_SB));
    
    /* Send address + write */
    I2C1->DR = addr << 1;
    while (!(I2C1->SR1 & I2C_SR1_ADDR));
    (void)I2C1->SR2;  /* Clear ADDR */
    
    /* Send register */
    I2C1->DR = reg;
    while (!(I2C1->SR1 & I2C_SR1_TXE));
    
    /* Send data */
    I2C1->DR = data;
    while (!(I2C1->SR1 & I2C_SR1_TXE));
    
    /* Stop condition */
    I2C1->CR1 |= I2C_CR1_STOP;
}

/* I2C master read */
uint8_t i2c_read(uint8_t addr, uint8_t reg) {
    /* Write register address */
    I2C1->CR1 |= I2C_CR1_START;
    while (!(I2C1->SR1 & I2C_SR1_SB));
    
    I2C1->DR = addr << 1;  /* Write mode */
    while (!(I2C1->SR1 & I2C_SR1_ADDR));
    (void)I2C1->SR2;
    
    I2C1->DR = reg;
    while (!(I2C1->SR1 & I2C_SR1_TXE));
    
    /* Restart for read */
    I2C1->CR1 |= I2C_CR1_START;
    while (!(I2C1->SR1 & I2C_SR1_SB));
    
    I2C1->DR = (addr << 1) | 1;  /* Read mode */
    while (!(I2C1->SR1 & I2C_SR1_ADDR));
    
    I2C1->CR1 &= ~I2C_CR1_ACK;  /* NACK */
    (void)I2C1->SR2;
    I2C1->CR1 |= I2C_CR1_STOP;
    
    while (!(I2C1->SR1 & I2C_SR1_RXNE));
    return I2C1->DR;
}
```

### SPI Protocol

```c
/* SPI master mode */
void spi_init(void) {
    RCC->APB2ENR |= RCC_APB2ENR_SPI1EN | RCC_APB2ENR_IOPAEN;
    
    /* PA5=SCK, PA6=MISO, PA7=MOSI */
    GPIOA->CRL &= ~(0xFFF << 20);
    GPIOA->CRL |= (0xB4B << 20);  /* AF push-pull, input, AF push-pull */
    
    /* Configure SPI */
    SPI1->CR1 = SPI_CR1_MSTR |     /* Master mode */
                SPI_CR1_SSM |      /* Software slave management */
                SPI_CR1_SSI |      /* Internal slave select */
                SPI_CR1_BR_2;      /* Baud rate = fPCLK/32 */
    
    SPI1->CR1 |= SPI_CR1_SPE;      /* Enable SPI */
}

uint8_t spi_transfer(uint8_t data) {
    while (!(SPI1->SR & SPI_SR_TXE));
    SPI1->DR = data;
    
    while (!(SPI1->SR & SPI_SR_RXNE));
    return SPI1->DR;
}

/* Chip select control */
void spi_select(void) {
    gpio_write(GPIOA, 4, 0);  /* CS low */
}

void spi_deselect(void) {
    gpio_write(GPIOA, 4, 1);  /* CS high */
}

/* Read from SPI device */
uint8_t spi_read_register(uint8_t reg) {
    spi_select();
    spi_transfer(reg | 0x80);  /* Read command */
    uint8_t value = spi_transfer(0x00);  /* Dummy byte */
    spi_deselect();
    return value;
}
```

### CAN Bus

```c
/* CAN initialization (basic) */
void can_init(void) {
    RCC->APB1ENR |= RCC_APB1ENR_CAN1EN;
    
    /* Enter initialization mode */
    CAN1->MCR |= CAN_MCR_INRQ;
    while (!(CAN1->MSR & CAN_MSR_INAK));
    
    /* Configure bit timing (500 kbps) */
    CAN1->BTR = (7 << 20) |   /* SJW */
                (5 << 16) |   /* TS2 */
                (8 << 0)  |   /* TS1 */
                (4 << 0);     /* BRP */
    
    /* Exit initialization mode */
    CAN1->MCR &= ~CAN_MCR_INRQ;
    while (CAN1->MSR & CAN_MSR_INAK);
}

/* Send CAN message */
void can_send(uint32_t id, uint8_t *data, uint8_t len) {
    /* Wait for empty mailbox */
    while (!(CAN1->TSR & CAN_TSR_TME0));
    
    /* Set ID */
    CAN1->sTxMailBox[0].TIR = (id << 21);
    
    /* Set data length */
    CAN1->sTxMailBox[0].TDTR = len;
    
    /* Copy data */
    CAN1->sTxMailBox[0].TDLR = (data[3] << 24) | (data[2] << 16) |
                                (data[1] << 8)  | data[0];
    CAN1->sTxMailBox[0].TDHR = (data[7] << 24) | (data[6] << 16) |
                                (data[5] << 8)  | data[4];
    
    /* Request transmission */
    CAN1->sTxMailBox[0].TIR |= CAN_TI0R_TXRQ;
}
```

---

## 8.6 Power Management

### Low-Power Modes

**Sleep modes (ARM Cortex-M):**

```c
/* Sleep mode - CPU stopped, peripherals running */
void enter_sleep(void) {
    __WFI();  /* Wait For Interrupt */
}

/* Deep sleep - CPU and some peripherals stopped */
void enter_deep_sleep(void) {
    SCB->SCR |= SCB_SCR_SLEEPDEEP_Msk;
    __WFI();
}

/* Standby - lowest power, RAM lost */
void enter_standby(void) {
    /* Enable wake-up pin */
    PWR->CSR |= PWR_CSR_EWUP;
    
    /* Set standby mode */
    PWR->CR |= PWR_CR_PDDS;
    SCB->SCR |= SCB_SCR_SLEEPDEEP_Msk;
    
    __WFI();
    /* Never reached - system resets on wake */
}
```

### Wake-Up Sources

```c
/* Configure wake-up from RTC alarm */
void configure_rtc_wakeup(uint32_t seconds) {
    /* Disable write protection */
    PWR->CR |= PWR_CR_DBP;
    RTC->WPR = 0xCA;
    RTC->WPR = 0x53;
    
    /* Disable alarm */
    RTC->CR &= ~RTC_CR_ALRAE;
    while (!(RTC->ISR & RTC_ISR_ALRAWF));
    
    /* Set alarm */
    RTC->ALRMAR = /* configure time */;
    
    /* Enable alarm and interrupt */
    RTC->CR |= RTC_CR_ALRAE | RTC_CR_ALRAIE;
    
    /* Enable write protection */
    RTC->WPR = 0xFF;
}

/* External interrupt wake-up */
void configure_exti_wakeup(uint8_t pin) {
    /* Configure pin as input */
    gpio_init_input_pullup(GPIOA, pin);
    
    /* Configure EXTI */
    EXTI->IMR |= (1 << pin);   /* Interrupt mask */
    EXTI->RTSR |= (1 << pin);  /* Rising trigger */
    
    NVIC_EnableIRQ(EXTI0_IRQn + pin);
}
```

### Power Consumption Optimization

**Techniques:**

```c
/* 1. Clock gating - disable unused peripherals */
void disable_unused_peripherals(void) {
    RCC->APB1ENR &= ~(RCC_APB1ENR_TIM2EN |
                      RCC_APB1ENR_TIM3EN |
                      RCC_APB1ENR_USART2EN);
}

/* 2. Lower clock frequency */
void reduce_clock_speed(void) {
    /* Switch to HSI (8 MHz) from PLL (72 MHz) */
    RCC->CR |= RCC_CR_HSION;
    while (!(RCC->CR & RCC_CR_HSIRDY));
    
    RCC->CFGR &= ~RCC_CFGR_SW;
    RCC->CFGR |= RCC_CFGR_SW_HSI;
    
    /* Can now disable PLL */
    RCC->CR &= ~RCC_CR_PLLON;
}

/* 3. Optimize GPIO */
void optimize_gpio_power(void) {
    /* Configure unused pins as analog (lowest power) */
    GPIOA->CRL = 0x00000000;  /* All analog input */
}

/* 4. Use interrupts instead of polling */
void use_interrupts_not_polling(void) {
    /* ❌ Polling - CPU always active */
    while (1) {
        if (button_pressed()) {
            handle_button();
        }
    }
    
    /* ✅ Interrupt - CPU sleeps until event */
    configure_button_interrupt();
    while (1) {
        __WFI();  /* Sleep until interrupt */
    }
}
```

---

## Best Practices

1. **Memory management**: Use static allocation in embedded systems
2. **Volatile correctness**: Always use volatile for hardware registers
3. **Interrupt efficiency**: Keep ISRs short and fast
4. **Power awareness**: Disable unused peripherals
5. **Real-time guarantees**: Use RTOS for complex timing requirements
6. **Hardware abstraction**: Layer drivers for portability
7. **Testing**: Test on actual hardware early and often
8. **Documentation**: Comment hardware-specific code thoroughly
9. **Safety**: Consider watchdog timers, brownout detection
10. **Standards**: Follow MISRA C for safety-critical systems

---

## Key Takeaways

**Embedded C Fundamentals:**
- Limited resources (RAM, flash, CPU)
- Memory-mapped I/O for hardware access
- Volatile keyword essential for registers
- ISRs must be fast and minimal
- Startup code runs before main()

**Bare-Metal Programming:**
- Direct hardware control
- Custom initialization required
- Implement your own "standard library"
- GPIO, timers, ADC/DAC programming
- PWM for motor control and more

**RTOS (FreeRTOS):**
- Tasks for concurrent activities
- Priorities and scheduling
- Queues for inter-task communication
- Semaphores for synchronization
- Deterministic timing

**Device Drivers:**
- Layered architecture
- Polling vs interrupts vs DMA
- Character and block devices
- HAL for portability

**Communication Protocols:**
- UART: Asynchronous serial
- I2C: Two-wire master/slave
- SPI: High-speed synchronous
- CAN: Robust automotive bus

**Power Management:**
- Multiple sleep modes
- Wake-up sources (RTC, EXTI)
- Clock gating
- Optimization techniques

---

## Part 8 Summary

This part covered embedded systems programming in C, from bare-metal to RTOS-based development. Embedded C is where the language excels, providing low-level control with high-level expressiveness.

**Series Progress:**
- ✅ Parts 1-7: Core C through systems programming (~66,400 words)
- ✅ Part 8: Embedded Systems Programming (~4,200 words)
- **Total: ~70,600 words** across eight comprehensive parts

All parts ready for publication with real-world embedded code examples!

**Note**: Embedded programming requires hardware to truly learn. Consider development boards like:
- STM32 Nucleo/Discovery (ARM Cortex-M)
- Arduino (AVR - good for learning)
- ESP32 (RISC-V/Xtensa - WiFi/Bluetooth)
- Raspberry Pi Pico (ARM Cortex-M0+)
