# D23-PHAM-ANH-TU

## Báo cáo về cảm biến INMP441

### I. INMP441 là gì ?

1. Là microphone MEM kỹ thuật số . Đây là dòng micro nhỏ gọn , được thiết kế thu âm thanh và xuất trực tiếp dữ liệu dạng I^2S , không cần thêm bộ ADC bên ngoài

2. Các đặc điểm nổi bật
  + Giao tiếp I^2S trực tiếp
  + Tích hợp sẵn preamp và bộ lọc số 
  + Dải tần đáp ứng : 60Hz -> 15 Khz
  + Độ nhạy cao : -26 DBFS 
  + HĐ ở 1.62 V - 3.6 V

3. Cấu tạo và các chân kết nối

| Số chân | Tên chân   | Chức năng                                                                 |
| ------- | ---------- | ------------------------------------------------------------------------- |
| 1       | **SCK**    | I²S Bit Clock (BCLK) – Xung clock điều khiển truyền bit dữ liệu.          |
| 2       | **SD**     | I²S Data Out – Dữ liệu âm thanh xuất ra từ micro.                         |
| 3       | **WS**     | I²S Word Select (LRCLK) – Xác định dữ liệu là kênh trái (L) hay phải (R). |
| 4       | **L/R**    | Chọn kênh: Low = Left (Trái), High = Right (Phải).                        |
| 5   | **GND**    | Mass (nối đất).                                                           |
| 6      | **VDD**    | Nguồn cấp 1.62–3.63 V (thường 3.3 V).                                     |

4. Nguyên lí hoạt động

- INMP441 là micro hđ trên giao thức I^2S

    + MCU đóng vai trò là 1 master , phát ra SCK và WS
    + INMP441 nhận clk này và xuất dữ liệu âm thanh 24 Bit dạng two's complement trên chân SD 
    + Dữ liệu sẽ MSB first , tri-state sau bit cuối cùng
    + fsCK = 64 * fWS
        + Vd : fs = 16 khz --> SCK = 1024 MHZ
    + Nếu có 2 micro dùng chung bus:
        + 1 mic có L/R = GND
        + 1 mic có l/R = VDD (RIGHT channel)
        + SD của 2 mic nối chung thành bus stereo

5. Cắm chân với STM 32 

| INMP441 Pin | STM32F103C8T6 Pin | Ghi chú                            |
| ----------- | ----------------- | ---------------------------------- |
| VDD         | 3.3V              | Có tụ 100 nF sát chân VDD          |
| GND         | GND chung         | Mass chung toàn hệ thống           |
| SCK         | PA5 (SPI1\_SCK)   | I²S Bit Clock                      |
| WS          | PA4 (SPI1\_NSS)   | I²S Word Select                    |
| SD          | PA7 (SPI1\_MOSI)  | I²S Data In (MCU nhận)             |
| L/R         | GND               | Mono - kênh trái                   |


