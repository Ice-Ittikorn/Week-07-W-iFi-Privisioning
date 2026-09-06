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
