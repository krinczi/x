# WeightGyroAccCAN\_Seeder\_STM32

# 📘 Dokumentacja komunikacji CAN (STM32 G431RB + HX711 + LSM9DS1)

## 📱 Ramki wysyłane (TX)

| ID (hex) | Nazwa                                   | Typ ramki | Długość | Dane (offsety)                                                            | Opis                                                    |
| -------- | --------------------------------------- | --------- | ------- | ------------------------------------------------------------------------- | ------------------------------------------------------- |
| 0x700    | Statystyki wagi 1 (HX711)               | Standard  | 8       | \[0..1]=avg, \[2..3]=stddev, \[4..5]=min, \[6..7]=max                     | Każda wartość jako int16\_t, little-endian z HX711 nr 1 |
| 0x701    | Statystyki wagi 2 (HX711)               | Standard  | 8       | \[0..1]=avg, \[2..3]=stddev, \[4..5]=min, \[6..7]=max                     | Każda wartość jako int16\_t, little-endian z HX711 nr 2 |
| 0x730    | Statystyki wagi 1 avg+std (HX711 float) | Standard  | 8       | \[0..3]=avg (float), \[4..7]=stddev (float)                               | Średnia i odchylenie standardowe float HX711 nr 1       |
| 0x731    | Statystyki wagi 1 min+max (HX711 float) | Standard  | 8       | \[0..3]=min (float), \[4..7]=max (float)                                  | Min i max float HX711 nr 1                              |
| 0x732    | Statystyki wagi 2 avg+std (HX711 float) | Standard  | 8       | \[0..3]=avg (float), \[4..7]=stddev (float)                               | Średnia i odchylenie standardowe float HX711 nr 2       |
| 0x733    | Statystyki wagi 2 min+max (HX711 float) | Standard  | 8       | \[0..3]=min (float), \[4..7]=max (float)                                  | Min i max float HX711 nr 2                              |
| 0x721    | Potwierdzenie wykonania (ACK)           | Standard  | 8       | \[0]=0x01, \[1]=cmd, \[2]=waga, \[3..7]=0                                 | Potwierdzenie wykonania danej komendy przez urządzenie  |
| 0x722    | Dane kalibracyjne cz.1 (HX711)          | Standard  | 8       | \[0]=waga, \[1..4]=raw\_zero (int32), \[5..6]=raw\_known (int16)          | Kalibracja tensometru – część 1                         |
| 0x723    | Dane kalibracyjne cz.2 (HX711)          | Standard  | 8       | \[0..3]=known\_mass\_kg (float), \[4..7]=0                                | Kalibracja tensometru – część 2                         |
| 0x760    | Statystyki IMU1 (ACC)                   | Standard  | 8       | \[0..1]=ax, \[2..3]=ay, \[4..5]=az, \[6]=acc\_flags, \[7]=gyro\_flags     | Średnia rolling avg z akcelerometru IMU1                |
| 0x765    | Statystyki IMU1 (GYRO)                  | Standard  | 8       | \[0..1]=gx, \[2..3]=gy, \[4..5]=gz, \[6]=acc\_flags, \[7]=gyro\_flags     | Średnia rolling avg z żyroskopu IMU1                    |
| 0x764    | Tilt pitch IMU1 (LSM9DS1)               | Standard  | 8       | \[0..1]=avg\_pitch×100, \[2..3]=min×100, \[4..5]=max×100, \[6..7]=std×100 | Statystyki przechylenia pitch IMU1 (offset korygowany)  |
| 0x761    | Tilt roll IMU1 (LSM9DS1)                | Standard  | 8       | \[0..1]=avg\_roll×100, \[2..3]=min×100, \[4..5]=max×100, \[6..7]=std×100  | Statystyki przechylenia roll IMU1 (offset korygowany)   |

## 📥 Ramki odbierane (RX)

| ID (hex) | Nazwa                | Typ ramki | Długość | Dane (offsety)                                                                 | Opis                                              |
| -------- | -------------------- | --------- | ------- | ------------------------------------------------------------------------------ | ------------------------------------------------- |
| 0x720    | Komenda kalibracyjna | Standard  | 8       | \[0]=cmd, \[1..4]=wartość\_kg (float), \[5]=target, \[6..7]=opcjonalnie window | Komendy kalibracyjne dla HX711 i IMU, patrz niżej |

### 🔢 Komendy kalibracyjne (`cmd` w bajcie `RxData[0]` z ramki `0x720`)

| Kod  | Znaczenie                        | Opis                                                                   |
| ---- | -------------------------------- | ---------------------------------------------------------------------- |
| 0x01 | Kalibracja 0 kg                  | Zapamiętaj bieżący odczyt jako wartość zerową (raw\_zero)              |
| 0x02 | Kalibracja X kg                  | Zapamiętaj bieżący odczyt jako odpowiadający X kg                      |
| 0x03 | Zapis do Flash                   | Trwale zapisuje raw\_zero, raw\_known, known\_mass\_kg (target = 0xFF) |
| 0x04 | Odczyt z Flash                   | Odczyt danych kalibracyjnych (ramki 0x722 i 0x723)                     |
| 0x05 | Kalibracja IMU                   | Kalibruje IMU, wysyła ramkę 0x11 z danymi kalibracji                   |
| 0x06 | Ustaw okno uśredniania dla HX711 | Ustawia rozmiar okna (bajty 6-7: uint16\_t, target: 0x01/0x02)         |
| 0x11 | Kalibracja IMU                   | Kalibruje acc+gyro. Bajty 1-4: liczba próbek (float), bajt 5: ID IMU   |
| 0x12 | Kalibracja tilt                  | Oblicza i zapisuje offsety pitch/roll (z bufora) dla wskazanego IMU    |

## 🧭 Kodowanie flag IMU (dla ramek ACC 0x760–0x763 i GYRO 0x765–0x768)

| acc\_flags (bajt 6) | Zakres akcelerometru | Wartość w driverze STM |
| ------------------- | -------------------- | ---------------------- |
| 0                   | ±2g                  | 0                      |
| 1                   | ±4g                  | 1                      |
| 2                   | ±8g                  | 2                      |
| 3                   | ±16g                 | 3                      |

| gyro\_flags (bajt 7) | Zakres żyroskopu | Wartość w driverze STM |
| -------------------- | ---------------- | ---------------------- |
| 0                    | 245 dps          | 0                      |
| 1                    | 500 dps          | 1                      |
| 2                    | 2000 dps         | 2                      |
