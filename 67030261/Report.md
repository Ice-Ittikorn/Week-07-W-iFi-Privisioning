# ใบงานที่ 7.1 การศึกษากลไก Reset Provisioning 3 รูปแบบ และ NVS Memory Forensics

## 5. กิจกรรมถอดรหัสซอร์สโค้ดและเขียนผังงาน (Code Deconstruction & Flowchart Assignment)

### ภารกิจที่ 1  ผังงานการตัดสินใจช่วง Bootstrapping & Reset Decision
<img width="216" height="651" alt="image" src="https://github.com/user-attachments/assets/7eb1b8d0-b165-4bff-a9f2-65e5a2f45fe1" />
### ภารกิจที่ 2 ผังสถานะการเปลี่ยนจังหวะไฟ LED 1 (Wi-Fi STA Indicator)
<img width="593" height="484" alt="image" src="https://github.com/user-attachments/assets/75f5288c-fde5-41b2-88fb-1af642cabe06" />


## 6. ตารางบันทึกผลการทดลอง (Experiment Results)
 
| รูปแบบการ Reset | คำสั่ง / พฤติกรรมที่ทำ | พฤติกรรมของ LED แต่ละดวงหลังเปิดเครื่อง | สถานะใน Serial Monitor |
| :--- | :--- | :--- | :--- |
| **1. CLI Erase** | `idf.py erase-flash` | LED ดับ ไม่ติดเลย | <pre>I (312) LAB7_1_RESET: Hold GPIO 18 button for 3 seconds to trigger Factory Reset...<br>I (322) wifi:wifi driver task: 3ffc1a5c, prio:23, stack:6656<br>W (398) LAB7_1_RESET: --------------------------------------------------<br>W (398) LAB7_1_RESET: [STATUS]: Device is NOT provisioned (NVS is empty)<br>W (408) LAB7_1_RESET: Ready for Provisioning Lab 7-2 (SoftAP) or 7-3 (BLE)!<br>W (418) LAB7_1_RESET: --------------------------------------------------</pre> |
| **2. Menuconfig Flag** | `CONFIG_EXAMPLE_RESET_PROVISIONED=y` | กระพริบถี่ ๆ | <pre>I (313) LAB7_1_RESET: Hold GPIO 18 button for 3 seconds to trigger Factory Reset...<br>I (399) LAB7_1_RESET: --------------------------------------------------<br>I (399) LAB7_1_RESET: [STATUS]: Already provisioned! Starting Wi-Fi Station<br>I (409) LAB7_1_RESET: --------------------------------------------------<br>I (529) wifi:mode : sta (3c:61:05:12:ab:cd)<br>I (2189) esp_netif_handlers: sta ip: 192.168.1.42, mask: 255.255.255.0, gw: 192.168.1.1<br>I (2189) LAB7_1_RESET: =================================================<br>I (2189) LAB7_1_RESET: [ONLINE]: Connected with IP: 192.168.1.42<br>I (2199) LAB7_1_RESET: =================================================</pre> |
| **3. Hardware Button (GPIO 18)** | กดปุ่ม GPIO 18 ค้าง 3 วินาที | LED ดับ ไม่ติดเลย | <pre>I (311) LAB7_1_RESET: Hold GPIO 18 button for 3 seconds to trigger Factory Reset...<br>I (1311) LAB7_1_RESET: Holding button... 1/3 seconds<br>I (2311) LAB7_1_RESET: Holding button... 2/3 seconds<br>I (3311) LAB7_1_RESET: Holding button... 3/3 seconds<br>W (3311) LAB7_1_RESET: =================================================<br>W (3311) LAB7_1_RESET: &gt;&gt;&gt; FACTORY RESET TRIGGERED! ERASING NVS FLASH &lt;&lt;&lt;<br>W (3321) LAB7_1_RESET: =================================================<br>W (3331) LAB7_1_RESET: [FORENSIC]: User requested Flash Erase!<br>W (3491) LAB7_1_RESET: --------------------------------------------------<br>W (3491) LAB7_1_RESET: [STATUS]: Device is NOT provisioned (NVS is empty)<br>W (3501) LAB7_1_RESET: Ready for Provisioning Lab 7-2 (SoftAP) or 7-3 (BLE)!<br>W (3511) LAB7_1_RESET: --------------------------------------------------</pre> |

---

## 7. คำถามท้ายการทดลอง (Post-Lab Questions)
1. เพราะเหตุใดการกดปุ่ม BOOT (GPIO 0) ค้างไว้ในจังหวะรีเซ็ตบอร์ด จึงทำให้โปรแกรมค้างอยู่ที่ ROM Bootloader และไม่ยอมทำงานต่อ?
```
GPIO 0 เป็น Strapping Pin ที่ชิปใช้เลือกโหมดบูต ชิป ESP32 จะแลตช์ค่าลอจิกของขานี้  ทีขอบขาขึ้นของสัญญาณ reset  ถ้าตอนนั้นเป็น LOW (0)  ROM code จะตัดสินใจเข้าสู่ ROM Download Bootloader แทนที่จะไปโหลด Second-stage bootloader ทำให Application จาก flash ผลคือ CPU หยุดรออยู่ที่สถานะ waiting for download บน UART ไม่มีการเรียก app_main() เลย ต่อให้ปล่อยปุ่มทีหลังก็ไม่ช่วย เพราะการตัดสินใจเกิดไปแล้วตอน reset ต้องกด EN ใหม่โดยไม่กด BOOT 
```
2. เพราะเหตุใดคำสั่ง `idf.py erase-flash` จึงทำให้ข้อมูลเฟิร์มแวร์ Application หายไปด้วย ในขณะที่ `nvs_flash_erase()` ไม่ทำให้เฟิร์มแวร์หาย?
```
    - idf.py erase-flash เป็นคำสั่งฝั่ง PC ที่คุยกับ ROM bootloader ผ่าน esptool สั่ง Chip Erase ทั้งชิป โดยไม่สนใจ Partition Table เลย ทุก sector ถูกเขียนเป็น 0xFF หมด ทั้ง bootloader, partition table, factory , nvs, phy_init จึงต้อง flash ใหม่ทั้งหมด
    - nvs_flash_erase() เป็นฟังก์ชันที่รันอยู่บนเฟิร์มแวร์เอง มันอ่าน Partition Table แล้วลบเฉพาะ sector ที่อยู่ในพาร์ทิชันชนิด data/nvs ซึ่งเป็นที่เก็บ Wi-Fi credentials เท่านั้น พาร์ทิชัน factory ที่เก็บโค้ดแอปไม่ถูกแตะ 
```
3. การออกแบบปุ่ม Factory Reset บนอุปกรณ์ IoT เชิงพาณิชย์ เหตุใดจึงต้องกำหนดให้ผู้ใช้กดปุ่มค้างไว้ 3-5 วินาที แทนที่จะสั่งลบข้อมูลทันทีที่แตะปุ่มเพียงเสี้ยววินาที?
```
เพราะ Factory Reset เป็นการกระทำที่ กู้คืนไม่ได้ ข้อมูลที่หายคือ credentials ที่ผู้ใช้ต้องเดินกลับไปทำ provisioning ใหม่ทั้งกระบวนการ การกดค้างจึงทำหน้าที่เป็น การยืนยันสิงที่อยากทำจริงๆ ป้องกัน 3 อย่างต่อไปนี้
    1. การกดโดนโดยบังเอิญ ปุ่มถูกชน กดทับตอนขนย้าย หรือเด็กเผลอกด
    2. สัญญาณรบกวนทางไฟฟ้า  ปุ่มจริงมีการเด้ง ระดับมิลลิวินาที ถ้าลบทันทีที่เห็น LOW ครั้งแรก noise หรือไฟกระชากอาจทริกเกอร์เองได้ การนับต่อเนื่อง 30 รอบ × 100ms อย่างในโค้ดจึงทำหน้าที่ debounce ในตัว
    3. ให้โอกาสยกเลิก  ระหว่างกดค้าง เฟิร์มแวร์รายงาน Holding button... 1/3 seconds และมักกระพริบไฟเตือน ผู้ใช้ที่กดผิดปล่อยมือได้ทัน โดยไม่มีอะไรถูกลบ
```
4. หากอุปกรณ์ IoT ถูกติดตั้งอยู่บนเสาสูงหรือฝังอยู่ในผนัง วิธีการ Reset ทางกายภาพรูปแบบใดเหมาะสมที่สุด?
```
    ใช้แหล่งจ่ายไฟ โดยออกแบบเป็น Power-Cycle Reset  คือให้เฟิร์มแวร์นับจำนวนครั้งที่ถูกตัด-ต่อไฟติดต่อกันอย่างรวดเร็ว เก็บ counter ไว้ใน NVS หรือ RTC memory ถ้าครบตามกำหนด เช่น เปิด-ปิดเบรกเกอร์ 5 ครั้งติดภายใน 10 วินาที จึงสั่ง nvs_flash_erase() แล้วกลับเข้าโหมด Provisioning ส่วนถ้าบูตแล้วอยู่นานเกิน 10 วินาทีก็รีเซ็ต counter กลับเป็นศูนย์ ถือว่าเป็นการเปิดใช้งานปกติ
```

---

# ใบงานที่ 7.2 การคอนฟิก Wi-Fi ผ่าน SoftAP Scheme และการวิเคราะห์ Protocomm Endpoints

## 5. กิจกรรมถอดรหัสซอร์สโค้ดและเขียนผังงาน (Code Deconstruction & Sequence Flow Assignment)
### ภารกิจที่ 1: ผังลำดับการสื่อสารผ่าน HTTP Endpoints (SoftAP Scheme Sequence Flow)
<img width="404" height="540" alt="image" src="https://github.com/user-attachments/assets/258f19cb-5faa-4e0d-8c80-8341cbaafe64" />

## 6. ตารางบันทึกผลการทดลอง (Experiment Results)

| รายการตรวจสอบ | ค่าที่บันทึกได้จากการทดลอง |
| :--- | :--- |
| **1. ชื่อ SoftAP SSID ของ ESP32** | PROV_4593E4 |
| **2. รหัส PoP (Proof of Possession)** | abcd1234 |
| **3. ข้อความใน QR Code Payload (JSON)** | {"ver":"v1","name":"PROV_4593E4","pop":"abcd1234","transport":"softap"} |
| **4. พฤติกรรมไฟ LED 3 (GPIO 5) ช่วงรอ vs ช่วงส่งข้อมูล** | ช่วงรอ: LED ติดค้าง <br/>ช่วงส่ง: LED ดับ |
| **5. IP Address ที่ ESP32 ได้รับจาก Router** | 172.20.10.2 |
| **6. เวลาที่ใช้ตั้งแต่เริ่มจนจบกระบวนการ (วินาที)** | 63.37 |

---

## 7. คำถามท้ายการทดลอง (Post-Lab Questions)
1. ในโหมด SoftAP Scheme สมาร์ตโฟนส่งข้อมูลหา ESP32 ผ่านโปรโตคอลและ IP Address ใด?
```
ใช้ HTTP POST ยิงไปที่ ESP32 ตอนที่มันเป็น SoftAP ซึ่ง ESP32 จะตั้งตัวเองเป็นเกตเวย์ที่ 192.168.4.1 เสมอ มือถือต้องต่อ Wi-Fi เข้า SSID ของ esp32 ก่อน ถึงจะยิง HTTP ไปที่ IP นี้ผ่าน endpoint /prov-session, /prov-scan, /prov-config ได้
```
2. หากผู้ใช้ป้อนรหัสผ่าน Wi-Fi ผิดในแอปมือถือ จะเกิด Event ใดขึ้นบน ESP32 (`WIFI_PROV_CRED_FAIL` หรือไม่) และ ESP32 มีพฤติกรรมอย่างไร?
```
จะเกิด Event NETWORK_PROV_WIFI_CRED_FAIL จะพิมพ์ log ระดับ error ว่า "Wi-Fi Connection failed with provided credentials!" แต่ ESP32 ไม่รีสตาร์ทหรือปิด Provisioning — ตัว Provisioning Manager ยังทำงานต่อ, LED3 (GPIO5) ยังติดค้างอยู่ รอให้มือถือส่ง /prov-config เข้ามาใหม่อีกครั้งได้เรื่อยๆ จนกว่าจะเชื่อมต่อสำเร็จ
```
3. ทำไมผู้ผลิต IoT ส่วนใหญ่จึงมองว่ากระบวนการเชื่อมต่อแบบ SoftAP มีขั้นตอนที่ยุ่งยากสำหรับผู้ใช้ทั่วไปเมื่อเทียบกับ BLE?
```
เพราะ SoftAP บังคับให้ผู้ใช้ต้อง ออกจากแอปไปที่หน้า Wi-Fi Settings ของมือถือเอง เพื่อสลับไปต่อ SSID ของ ESP32 ก่อน ระหว่างนั้นจะหลุดจากอินเทอร์เน็ต หรือยังเด้งเตือนหรือตัดการเชื่อมต่อ Wi-Fi ที่ไม่มีอินเทอร์เน็ตออกเองอัตโนมัติ ทำให้ session หลุดกลางคัน ส่วน BLE ไม่ต้องสลับเครือข่ายเลย แอปคุยกับ ESP32 ผ่าน BLE ควบคู่กับ Wi-Fi เดิมได้ตลอด ประสบการณ์ผู้ใช้เลยลื่นไหลกว่ามาก
```

---

# ใบงานที่ 7.3 การคอนฟิก Wi-Fi ผ่าน BLE Scheme และการสืบสวน GATT Services (BLE Forensics)
## 5. กิจกรรมถอดรหัสซอร์สโค้ดและเขียนผังงาน (Code Deconstruction & BLE GATT Architecture Assignment)
### ภารกิจที่ 1: ผังโครงสร้าง GATT Tree & Endpoint Mapping
<img width="946" height="410" alt="image" src="https://github.com/user-attachments/assets/3839d837-c6d3-428d-88f0-21b65c01dba1" />

### ภารกิจที่ 2: ผังลำดับการคืนหน่วยความจำ Bluetooth (BLE Lifecycle & Memory Reclaim Flow)
<img width="697" height="487" alt="image" src="https://github.com/user-attachments/assets/3cf53233-2291-41fc-85a4-e107da17c5e6" />

## 6. ตารางบันทึกผลการทดลอง (Experiment Results)

| รายการตรวจสอบ | ผลการทดลอง / ข้อมูลที่สังเกตได้ |
| :--- | :--- |
| **1. BLE Device Name ที่สแกนเจอ** | PROV_4593E4 |
| **2. Primary Service UUID (128-bit)** | 021a9004-0382-4aea-bff4-6b3f1c5adfb4 |
| **3. Characteristic Endpoint ที่พบ (0x2901)** | 1. 021aff50-0382-4aea-bff4-6b3f1c5adfb4 <br/>2. 021aff51-0382-4aea-bff4-6b3f1c5adfb4<br/>3. 	021aff52-0382-4aea-bff4-6b3f1c5adfb4 |
| **4. พฤติกรรมไฟ LED 2 (GPIO 4) ช่วงรอ vs ช่วงต่อ BLE** | ช่วงรอ: LED 2 ติดค้างสว่างตลอด <br/>ช่วงต่อ: LED 2 ติดค้างสว่างตลอด <br/> หลัง provision สำเร็จ: LED 2 ดับ และ LED 1 ติดเมื่อได้ IP|
| **5. พฤติกรรมเมื่อต่อ Wi-Fi สำเร็จ** | มี Log คืนหน่วยความจำ Bluetooth  |

---

## 7. คำถามท้ายการทดลอง (Post-Lab Questions)
1. เหตุใด BLE Provisioning จึงไม่ส่งผลให้สัญญาณ Wi-Fi บนสมาร์ตโฟนของผู้ใช้หลุดระหว่างทำรายการ?
```
   เพราะ BLE กับ Wi-Fi เป็นคนละ radio interface กันบนโทรศัพท์ การรับส่งข้อมูล provisioning วิ่งผ่าน GATT บนช่อง Bluetooth ทั้งหมด ตัว Wi-Fi interface ของโทรศัพท์จึงยังเกาะ AP เดิมอยู่ตลอด
ต่างจาก SoftAP Scheme ที่โทรศัพท์ต้องสลับ Wi-Fi ไปเกาะ AP ของ ESP32 เอง ทำให้หลุดจากอินเทอร์เน็ตชั่วคราว ต้องสลับกลับเองหลังเสร็จ และเสี่ยงที่ระบบ Android/iOS จะดีดกลับ AP เดิมกลางคัน เพราะ AP ของ ESP32 ไม่มีอินเทอร์เน็ต ทำให้ provisioning ล้มเหลว
```
2. Descriptor `0x2901` มีความสำคัญอย่างไรต่อการที่แอปพลิเคชันมือถือจะทราบว่า Characteristic แต่ละตัวใช้ทำหน้าที่อะไร?
```
   Descriptor 0x2901  เป็นสตริงข้อความที่ผูกกับ characteristic นั้น ๆ ทำหน้าที่บอกชื่อ Protocomm Endpoint เช่น prov-session, prov-config, prov-scan แอปฝั่งมือถือจึงใช้วิธี discover service แล้วอ่าน 0x2901 ของทุก characteristic เพื่อ map ชื่อ endpoint handle แบบ dynamic ได้เอง โดยไม่ต้อง hard-code UUID ไว้ในแอป
ข้อดีคือ firmware เปลี่ยน UUID ได้ แอปเดิมก็ยังใช้งานได้ และในทางกลับกัน ในมุม forensic นี่คือช่องที่ทำให้เราใช้ nRF Connect ส่องเห็นโครงสร้างภายในของอุปกรณ์ได้ทั้งหมดโดยไม่ต้องมี source code
```
3. การที่ ESP-IDF มีฟังก์ชัน `esp_bt_mem_release()` มีประโยชน์อย่างไรต่อการทำงานของแอปพลิเคชัน IoT หลังเชื่อมต่อ Wi-Fi สำเร็จ?
```
   BLE stack  กินหน่วยความจำ DRAM ประมาณ 60–70 KB ซึ่งเป็นสัดส่วนที่สูงมากเมื่อเทียบกับ RAM ทั้งหมดของ ESP32 ประมาณ 320 KB แต่ในงานลักษณะนี้ BLE ถูกใช้แค่ช่วง provisioning ครั้งแรกครั้งเดียว หลังจากได้ SSID Password แล้วอุปกรณ์จะสื่อสารผ่าน Wi-Fi อย่างเดียวตลอดอายุการใช้งาน NETWORK_PROV_SCHEME_BLE_EVENT_HANDLER_FREE_BTDM ตอน network_prov_mgr_deinit() จึงคืน RAM กลับสู่ heap ให้แอปพลิเคชันหลักใช้ต่อ เช่น TLS/MQTT buffer 
```

---

# ใบงานที่ 7.4: การทดสอบ Security Schemes (PoP) และการรับส่ง Custom Data Endpoints
## 6. กิจกรรมถอดรหัสซอร์สโค้ดและเขียนผังงาน (Code Deconstruction & Security Flow Assignment)
### ภารกิจที่ 1: ผังขั้นตอนการตรวจสอบ PoP (Security Handshake Decision Flow)
<img width="427" height="482" alt="image" src="https://github.com/user-attachments/assets/6cbe1773-6345-4fc9-9418-c1ae048a8fea" />

### ภารกิจที่ 2: ผังการรับส่งข้อมูลผ่าน Custom Endpoint (Custom Data Handler Flow)
<img width="564" height="521" alt="image" src="https://github.com/user-attachments/assets/f807226d-82e7-43fc-957b-cea502242419" />


## 7. ตารางบันทึกผลการทดลอง (Experiment Results)

| สถานการณ์ทดสอบ | ค่า PoP ที่ป้อน | ผลลัพธ์บนแอปมือถือ | ข้อความ Log ใน Serial Monitor |
| :--- | :--- | :--- | :--- |
| **1. ป้อน PoP ผิดพลาด** | `wrong1234` |Failed to initialise session with the device |<img width="737" height="141" alt="image" src="https://github.com/user-attachments/assets/538035e4-c791-4fa0-8572-857f9731c66e" />|
| **2. ป้อน PoP ถูกต้อง** | `abcd1234` | Device has been successfully provisioned!|[SECURITY SUCCESS]: Valid PoP! Secured Session OK!|
| **3. ส่ง Custom Data** | `TEST_DATA_999` |แอปส่ง Payload ไปยัง custom-data ได้รับ ACK |[CUSTOM DATA RECEIVED]: TEST_DATA_99 |

---

## 8. คำถามท้ายการทดลอง (Post-Lab Questions)
1. การใช้ **Proof-of-Possession (PoP)** ช่วยป้องกันการโจมตีประเภทใดได้บ้าง?
```
    - Rogue/Unauthorized Provisioning — ป้องกันคนแปลกหน้าที่อยู่ในระยะสัญญาณ BLE แต่ไม่รู้รหัส PoP ไม่ให้ยึดอุปกรณ์ไป Provision WiFi ของตัวเองแทน เพราะต่อให้เชื่อมต่อ BLE ได้ ก็ผ่าน handshake ไม่ได้
    - Man-in-the-Middle ระหว่าง Key Exchange — Security 1 ใช้ PoP เป็น input ในการยืนยันตัวตนของทั้งสองฝั่งระหว่างแลกเปลี่ยน public key ทำให้ผู้ดักฟัง ที่อยู่กลางทางไม่สามารถสวมรอยเป็น ESP32 หรือเป็น App เพื่อขโมย Session Key ไปถอดรหัสข้อมูลได้
    - การรั่วไหลของ WiFi Credential — เนื่องจากข้อมูล WiFi SSID/Password ที่ส่งผ่าน BLE ไปยัง ESP32 ถูกเข้ารหัสด้วย Session Key ที่มาจาก PoP ถ้าไม่มี PoP ที่ถูกต้อง ผู้โจมตีจะดักฟังแล้วถอดรหัสข้อมูล WiFi ที่ส่งผ่านไม่ได้
```
2. หากไม่มีการใช้ PoP (เช่น ใน Security 0) ผู้โจมตีที่อยู่ในรัศมีสัญญาณบลูทูธสามารถทำสิ่งใดกับอุปกรณ์ได้บ้าง?
```
    - เชื่อมต่อและ Provision อุปกรณ์แทนเจ้าของจริง ส่ง SSID Password ปลอมเข้าไป ทำให้ ESP32 ไปเชื่อมต่อ WiFi ของผู้โจมตีเอง แล้วดักข้อมูลที่อุปกรณ์ส่งออกไปทั้งหมด 
    - ดักฟัง  ข้อมูล WiFi Credential ที่ส่งผ่าน BLE แบบ Plaintext — เห็น SSID Password ของ WiFi บ้าน องค์กรของเจ้าของอุปกรณ์ตรงๆ โดยไม่ต้องถอดรหัสอะไรเลย
    - ส่งข้อมูลปลอมเข้า Custom Data Endpoint เช่นถ้ามี endpoint ที่ตั้งค่า activation code, MQTT broker URL, Owner ID ผู้โจมตีสามารถยัดค่าที่เป็นอันตราย เข้าไปแทนเจ้าของอุปกรณ์ตัวจริง
    - Denial of Service เชิง Provisioning —ยึด session การเชื่อมต่อ BLE ไว้ก่อนเจ้าของจริง ทำให้เจ้าของอุปกรณ์ Provision อุปกรณ์ของตัวเองไม่ได้
```
3. ในการประยุกต์ใช้งานเชิงพาณิชย์จริง เราสามารถนำ **Custom Data Endpoint** ไปใช้ส่งข้อมูลประเภทใดได้อีกบ้าง (ยกตัวอย่าง 2 กรณี)?
```
    - การผูกอุปกรณ์กับบัญชีผู้ใช้  ตอน Provisioning ส่ง User ID / Owner Email / Activation Token จากแอปมือถือไปเก็บใน NVS ของ ESP32 เพื่อให้อุปกรณ์รู้ว่าเป็นของผู้ใช้คนไหนตั้งแต่แรกเริ่ม ก่อนที่จะเชื่อมต่อ Cloud/Backend ครั้งแรกด้วยซ้ำ ใช้แทนขั้นตอน pairing ทีหลังผ่าน Cloud
    - การตั้งค่า Endpoint การเชื่อมต่อ Cloud/IoT Platform — ส่ง MQTT Broker URL, Server Certificate/Fingerprint, หรือ Device Token สำหรับเชื่อมต่อ IoT Platform เช่น AWS IoT, Azure IoT Hub, หรือ Private MQTT Broker ขององค์กรเอง เพื่อให้ผลิตภัณฑ์ชิ้นเดียวกันสามารถขายให้ลูกค้าหลายรายที่ใช้ Backend คนละตัวกันได้ โดยไม่ต้อง flash firmware ใหม่ทุกครั้ง
```
4. ในฟังก์ชัน `custom_prov_data_handler()` เหตุใดหน่วยความจำที่จัดสรรให้ `*outbuf` จึงถูก Free โดย Protocomm Layer อัตโนมัติหลังจากส่งข้อมูลเสร็จ?
```
    - Handler ของผู้ใช้ เช่น custom_prov_data_handler มีหน้าที่แค่ สร้าง ข้อมูลตอบกลับด้วย malloc()/strdup() แล้วส่ง pointer กลับผ่าน *outbuf เท่านั้น  ไม่ได้เป็นคนส่งข้อมูลออกไปทาง BLE/HTTP เอง
    - หลังจาก Handler return ESP_OK กลับมา ตัว Protocomm Layer ชั้นที่อยู่เหนือ Endpoint Dispatcher จะเป็นคนนำ *outbuf/*outlen ไปเข้ารหัสด้วย Session Key แล้วส่งออกไปยัง Client ต่อ Protocomm คือเจ้าของ pointer นี้ในช่วงเวลาถัดจากนี้ 
    - เมื่อส่งข้อมูลออกไปเรียบร้อยแล้ว Protocomm รู้ตัวว่าไม่มีใครใช้ buffer นี้ต่อแล้ว จึงเป็นผู้รับผิดชอบเรียก free() เอง เพื่อคืนหน่วยความจำกลับสู่ Heap
```

