# D23-PHAM-ANH-TU

## A. Báo cáo công việc đã làm 

### I. CÁCH TÍNH TOÁN BAUDRATE 

-   Trong tài liệu stm32 f1 , có 2 công thức tùy thuộc vào chế độ Oversampling

    + khi over8 = 0 ( oversampling 16x)
        ```cpp
        Baudrate = f_CK / (16 × USARTDIV)
        ```
    + khi over8 = 1 (oversampling 8x)
        ```cpp
        Baudrate = f_CK / (8 × USARTDIV)
        ```
    + Trong đó 

        + f_CK : Tần số clock của peripheral (PCLK1 hoặc PCLK2 )

        + USARTDIV : Hệ số chia, là hệ số có phần nguyên và phần thập phân

- Vậy oversampling là gì ?

- là số lần mà mỗi bit được lấy mẫu

    + oversampling 16x : là lấy 16 lần cho mỗi bit --> tốc độ chính xác 

    + oversampling 8x  : chỉ lấy 8 lần --> baudrate cao hơn

- Thanh ghi USART_BRR có cấu trúc như sau 

    + từ bit 15-> 4 : DIV_mantissa :  phần nguyên của USARTDIV

    + từ bit 3-> 0  : DIIV_Fraction : phần thập phân của USARTDIV

    + NOTE : Khi over8 = 1 thì dùng 3 bit  : 2 -> 0 , bit 3 phải giữ = 0

- cách tính:

    + Eg: baudrate 9600 , clock 16Mhz , over8 = 0

        + đầu tiên tính usartdiv = f_CK / (16 * baudrate) = 104,1666

        + bước 2 ta đi tách phần thập phân và phần nguyên 

            + phần nguyên = 104 = 0x68 ( đổi sang hệ 16 phần nguyên thì chia cho 16)

            + phần thập phân = 0.166666 = 0x03 ( đổi sang hệ 16 , phần tp nhân 16)

        + bước 3 : ghép vào thanh ghi BRR

            ```cpp
            USART_BRR = (Mantissa << 4) | Fraction // dùng toán tử or 
            USART_BRR = (0x68 << 4) | 0x03        // mantissa phải dịch trái 4 bit để nhường cho
            USART_BRR = 0x680 | 0x03 = 0x683      // 4 bit phần thập phân
            ```
- NOTE : đẻ chạy được baudrate như mong muốn thì ta phải tính đc BRR ( tức là bộ chia tần ) vì 

clk đi qua bộ chia tần rồi tạo ra tốc độ bit (baudrate) . BRR là thanh ghi cấu hình baudrate

- Clock quan trọng hay dùng

```cpp
USART1 → APB2 (PCLK2) → 72 MHz
USART2 → APB1 (PCLK1) → 36 MHz
USART3 → APB1 (PCLK1) → 36 MHz

```

- Tuy nhiên theo e nghiên cứu người ta có cách tính hay dùng khi code mà kết quả vẫn đúng như 

cách trên 

```cpp
BRR = APBCLK / BAUD
```

- Đây là 2 source code để trính BRR

```cpp
// cách 1: hay dùng

void USART2_Init_Simple(uint32_t baudrate) {
    RCC->APB1ENR |= RCC_APB1ENR_USART2EN; // bật clk
    

    USART2->BRR = 36000000 / baudrate;
    
    USART2->CR1 |= USART_CR1_TE | USART_CR1_RE | USART_CR1_UE;
}

// cách 2 như công thức lí thuyết

void USART2_Init_Detailed(uint32_t baudrate) {
    RCC->APB1ENR |= RCC_APB1ENR_USART2EN;
    
    float usartdiv = 36000000.0f / (16.0f * baudrate);
    uint16_t mantissa = (uint16_t)usartdiv;
    uint16_t fraction = (uint16_t)((usartdiv - mantissa) * 16);
    
    USART2->BRR = (mantissa << 4) | fraction;
    
    USART2->CR1 |= USART_CR1_TE | USART_CR1_RE | USART_CR1_UE;
}
```

### II. Ưu nhược điểm của UART

- ƯU ĐIỂM CỦA UART

    + Chỉ cần 2 dây: TX và RX (+ GND)
    + Không cần clock đồng bộ như SPI hay I2C
    + Với RS-232: có thể truyền 15-30 mét
    + Với RS-485: lên đến 1200 mét

- NHƯỢC ĐIỂM CỦA UART
    + Tốc độ hạn chế

    + Thường chỉ đạt 115200 baud (≈ 14.4 KB/s) ổn định
    + Tối đa vài Mbps, chậm hơn nhiều so với:

            SPI: 10-50 Mbps
            USB: 480 Mbps
    + Một master chỉ kết nối một slave 
    + Cần nhiều UART nếu kết nối nhiều thiết bị
    + Tiêu tốn năng lượng hơn khi idle
    + Một số module UART luôn chạy
    + Dễ bị nhiễu

### III. Ứng dụng trong các module ?

- Module giao tiếp ngoại vi

    + GPS (NEO-6M, NEO-7M…)
    + Bluetooth (HC-05, HC-06, BLE)

    + WiFi (ESP8266, ESP32 ở chế độ AT)

    + GSM / LTE (SIM800, SIM7600…)

    + RF module (LoRa UART, RF433)
- ngoài gửi dữ liệu uart còn

    + trong Module hiển thị & điều khiển

       --> nó gửi lệnh điều khiển 
    + trong Modul lưu trữ và đo lường

       --> gửi cấu hình , và kết quả đo

### IV. Tổng kết

- uart là giao thức không đồng bộ vì không có clock chung của tx và rx mà phải thống nhất nhau 

trước về baudrate và format khung frame data

- khi không đồng bộ có 1 vài lợi ích sau

    + không cần clk , ít chân 

    + cấu hình đơn giản

    + ở tốc độ thấp thì ít nhiễu hơn spi và i2c ở clock dài

    + sai lệch 1 chút baudrate vẫn chạy được 

- nhược điểm khi không đồng bộ

    + tốc độ thấp hơn so với spi và usb

    + vì phải mất data cho start bip và stop bit

    + lệch baudrate từ 2 -> 5 % dễ lỗi frame truyền

    + không truyền được ở tốc độ cao

    






