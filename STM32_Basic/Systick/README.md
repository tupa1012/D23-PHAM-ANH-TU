# D23-PHAM-ANH-TU

## I. Tìm hiểu về Systick

- SysTick (System Tick Timer) là một bộ đếm thời gian 24-bit đơn giản được tích hợp sẵn trong lõi ARM Cortex-M (M0, M3, M4, M7...).

- Đăc điểm 

    + Không phải là ngoại vi của STM32 (như TIM1, TIM2...)
    + Là một phần của ARM Core --> có trong TẤT CẢ chip dùng Cortex-M
    + Đếm ngược (count-down) từ giá trị LOAD về 0
    + Chỉ có 1 SysTick duy nhất trong mỗi MCU

- Dùng systick để tạo thời gian delay cho hệ thống nhúng , đơn giản , có trên tất cả ARM

- Cấu trúc phần cứng Systick

![alt text](image.png)


* các thanh ghi quan trọng

1) SysTick->CTRL (Control and Status Register)

![alt text](image-1.png)

            ```cpp
            C1:

            SysTick->CTRL = (1<<2) | (1<<1) | (1<<0); // bật clk, có ngắt , enable

            C2: 
            SysTick->CTRL = SysTick_CTRL_CLKSOURCE_Msk | 
                            SysTick_CTRL_TICKINT_Msk | 
                            SysTick_CTRL_ENABLE_Msk;
            ```



2) SysTick->LOAD (Reload Value Register)

    + Giá trị max : `0xFFFFFF` (2^24 - 1 = 16,777,215)
    + Giá trị min : `0x000001`

    + Cách tính 

    ```cpp
        Thời gian tràn (s) = (LOAD + 1) / Tần số clock

        --> LOAD = (t * clk ) - 1  
    ```
3) SysTick->VAL (Current Value Register)

    + chứa các giá trị đang đếm ( từ load -> 0 )
    + Xóa val về 0 và xóa countflag
    + lấy giá trị hiện tại của bộ đếm

- Ứng dụng :

    + Tạo time base cho hệ thống (như tạo tick 1ms cho RTOS)
    + Đo thời gian thực thi code (như bạn đang làm với bubble sort)
    + Tạo delay chính xác (delay_ms, delay_us)
    + Tạo soft timer (đếm thời gian chạy hệ thống)







