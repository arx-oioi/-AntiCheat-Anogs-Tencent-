# Tencent Protection (ANO / GP / TSS / ACE)

## Author @Tg ARXMODER

<img width="1919" height="887" alt="Image" src="https://github.com/user-attachments/assets/52f82f26-3e68-4d89-af5b-2f5c22589d83" />



![Image](https://github.com/user-attachments/assets/aede6d6a-c0c3-4764-bf53-ca1192dd2453)


`libanogs.so` คำสั่งเหล่านี้มักอยู่ใน:

### 🔍 1. ระบบตรวจจับ (Detection Core)

* scan /proc/self/maps
* scan memory region
* ตรวจ hook (plt / inline)
* ตรวจ GG / frida / ptrace
* ตรวจ syscall abnormal

### 🧬 2. ระบบป้องกันตัวเอง (Self‑Defense)

* anti-debug
* anti-dump
* anti-inject
* anti-fork
* anti-trace

### 🔄 3. ระบบ scheduler ภายใน

* loop ตรวจซ้ำเป็นช่วง
* trigger ตาม event
* background thread

### 🧨 4. ระบบตอบโต้ (Reaction)

* kill process
* fake crash
* corrupt memory
* report server
* delay ban




┌─────────────┐
│ Application │
│ (Java)      │
└─────┬───────┘
      │ JNI
      ▼
┌─────────────┐
│ Native Core │  ← anogs / anort
│ (Anti)      │
└─────┬───────┘
      │
      ▼
┌─────────────┐
│ State Engine│
│ + Checks    │
└─────────────┘


[BOOT]
  ↓
[INIT]
  ↓
[ENV_CHECK]
  ↓
[VERIFY_APP]
  ↓
[RUNTIME_MONITOR]
  ↓
[OK] ────────────────┐
  │                  │
  └──> [VIOLATION] → [TERMINATE]


---

![Image](https://github.com/user-attachments/assets/b4ecf149-4c15-4e92-805d-2b347533b635)

![Image](https://github.com/user-attachments/assets/1e453745-6442-4db2-b3c5-8d2a3a09badd)

![Image](https://github.com/user-attachments/assets/a267c81a-777b-48c4-b775-76c59983576c)

### รู้เกี่ยวกับ ANOGS (Tencent) – Any Cheat ก็ไม่รอดจริงไหม?

ถ้าคุณเคยเจออาการ:

เกมเด้งแบบไม่มี error

เข้าเกมไม่กี่วิแล้วปิดเอง

หรือ patch inline inject ต่างๆ 

ผ่าน Play Integrity แต่ยังโดนเตะ แปลว่าคุณน่าจะเคยเจอ ANOGS แล้ว

---

### ANOGS คืออะไร

ANOGS คือแกนกลางระบบป้องกันของ Tencent ฝั่ง client
มันไม่ใช่ anti-cheat แบบจับโปรโกงตรง ๆ
แต่เป็นระบบตรวจ “พฤติกรรมของสภาพแวดล้อมที่เกมกำลังรันอยู่”

แนวคิดหลักของ ANOGS คือ
ไม่ถามว่าผู้เล่นใช้เครื่องมืออะไร
แต่ถามว่า environment นี้ “ควรจะเป็นของผู้เล่นปกติหรือไม่”


---

### แนวคิดหลักที่ Tencent ใช้

Tencent ไม่เชื่อผลลัพธ์ที่ระบบรายงานตัวเอง เช่น

Android API

ชื่อ process

ค่า root flag


สิ่งที่ Tencent เชื่อคือ พฤติกรรมจริงที่เกิดขึ้นใน runtime และ memory

ถ้าพฤติกรรมไม่เหมือนเครื่องผู้เล่นทั่วไป
ระบบจะเริ่มให้คะแนนความเสี่ยงทันที


---

### โครงสร้างภาพรวมของระบบ

GP / ANOGS ทำหน้าที่ตรวจพฤติกรรมและสภาพแวดล้อมขณะเกมทำงาน

Integrity / Trust State ประเมินความน่าเชื่อถือของเครื่องในหลายมิติ

TSS สะสมข้อมูลพฤติกรรมระยะยาว สร้าง risk profile

ACE เป็นตัวตัดสินขั้นสุดท้าย สามารถแบนย้อนหลังได้


---

### ANOGS ตรวจอะไรบ้าง

* Memory (จุดสำคัญที่สุด)



การใช้ ptrace

process_vm_readv

การเปลี่ยนแปลง memory map

page permission ผิดปกติ

opcode หรือ inline patch


ANOGS ไม่จำเป็นต้องรู้ว่าอ่านหรือเขียนอะไร
แค่มีร่องรอยว่ามีการแตะ memory ก็ถือว่าผิดปกติ


---

* Runtime / ART



classloader ผิดปกติ

Zygisk injection timing

hook framework

native library injection


แม้ระบบจะไม่ถูกแก้ไขแบบ static
แต่ runtime ที่ผิดธรรมชาติก็ถูกมองว่าเสี่ยง


---

* Environment



root (ไม่ใช่แค่ su)

mount namespace แปลก

emulator หรือ VM behavior

CPU / timing anomaly

Kernel Module Loaded 

sensor entropy


ค่า spoof ทำได้
แต่พฤติกรรมระยะยาวปลอมได้ยาก


---

* Network



region anomaly

packet replay pattern

MITM signal


การใช้ VPN ไม่ได้ผิดทันที
แต่เป็นตัวเพิ่มคะแนนความเสี่ยง


---

### Crash ของ Tencent คืออะไร

Crash ที่เกิดจาก ANOGS ไม่ใช่ bug
แต่เป็นกลไกตอบโต้

วัตถุประสงค์ของ crash คือ

ตัดการทำงานทันที

ทำให้ trace ยาก

ทำให้ cheat และ reverse ทำงานไม่เสถียร

คือสามารถ patch ได้ memory inject หรือ อะไร ต่างๆ มันก็ทำได้
เมื่อเข้าไปสู่ lobby หรือแค่ ภายใน login ระบบจะเริ่ม 
ตรวจสอบ ความเสี่ยง จากนั้น จะ Crash หรือ พังโดยไร้ เหตุผล
คือในลักษณะหลายรูปแบบที่คุณพยายามที่จะทำมันจะตรวจสอบได้ยากมากๆ
ถ้าต้องการ Fix Crash จริงๆ มันจะอยู่ใน GP4 แหละใน C Path ต่างๆ
มันเชื่อมโยงกันคุณจะต้องหาทางเอาเอง Fix Crash ที่ ไม่มี Undetected มันอยู่ที่นั้น 
ไม่เชื่อก็หามันให้เจอ แค่ต้องผ่าน ไม่กี่2 หมื่น cpp แค่นั้น 

ในหลายกรณี crash จะเกิดแบบ delay
เพื่อทำให้การวิเคราะห์ยากขึ้น


---

### ทำไมบางคนโกงแล้วไม่โดนทันที

เพราะ Tencent ไม่แบนจากเหตุการณ์เดียว
แต่ใช้การสะสมพฤติกรรม

TSS จะเก็บข้อมูลเป็นระยะ
เมื่อ risk profile สูงพอ
ACE จึงค่อยตัดสิน

ในกรณีนี้ ที่ไม่โดนบางทีอาจจะเป็นอาการ ของ Error หรือในบางจุดที่
ไม่สามารถทำงานในจุดๆนั้นๆ แบบลงตัว เลยทำให้มีอาการ delay
สำหรับ เหตุผลเกี่ยวกับ การ Bypass Ca Cd หรือ Xa อะไรถึงจะดีกว่า 
ระบบหลักๆมันถูกเข้ารหัสแหละแชร์ library กันในการตรวจสอบเป็นหลัก
เลยไม่สามารถ Bypass Fully AntiCheat ได้ เพียงแค่ Patch Onec Offset
คุณก็อาจจะถูกแบนในไม่กี่น่าที ไม่ก็เกม Crash ก่อนที่จะถูกแบนมันเลยเป็นไปได้ยาก
ที่จะตรวจสอบ หา offset ที่มีช่องว่างแหละ การทำงาน Fix Crash Offset มันใช้งานได้
ในหลายๆ กรณีเพราะในส่วนใหญ่จะเป็นการ patch ในข้อมูลใหญ่ ไม่ได้เจาะจงที่ตัวระบบหลัก
ของ การตรวจสอบที่แท้จริง มันส่งกันผ่านเพียงแค่ ไม่กี่ Byte แหละ Return 0 กับ 1 แค่นั้น
ทันเลยยากมากๆที่จะตรวจสอบในพร้อมสำหรับทกสอบระบบของพวกเขา 

นี้แหละเหตุผลทำไมบางทีบาง offset ถึงถูกแบนแหละไม่ถูกแบน เพราะมันแค่ ชั่วคราว
แหละระบบใหม่ ที่สามารถโดนแบนได้ทันที คือการ report + Device Fingerprint ของ ระบบ anogs 
สร้างขึ้นมาเองเป็นไปได้ยากที่จะปลดล็อกหรือSpoof

จึงเกิดกรณีโดนแบนย้อนหลัง


---

### “3 เขียว” ทำไมยังโดน

Play Integrity ตรวจสภาพระบบ
ANOGS ตรวจพฤติกรรม runtime

เครื่องที่ผ่าน Strong Integrity
แต่มี hook, memory anomaly หรือ runtime patch
ยังถือว่าไม่น่าเชื่อถือในมุม Tencent


---

### สิ่งที่ควรเข้าใจ

ANOGS ไม่ได้โง่

แต่ก็ไม่รู้ทุกอย่าง

สิ่งที่รอด ไม่ได้แปลว่าปลอดภัย

แค่ยังไม่ชน policy ในตอนนั้น

ก็ตามที่ว่าม่นั้น ในบางจุดทุกคนก็สามารถทดสอบ
กลโกงของตัวเองได้แต่มันจะไม่นานขนาดที่จะทำให้คุณหารายได้
ไปตลอดชีวิตหรอก ไอ่โง่

---

* สรุป

Tencent ไม่ได้จับว่าใช้โกงอะไร
แต่จับว่าเครื่องยังทำตัวเหมือนผู้เล่นปกติหรือไม่

ถ้า memory ถูกแตะ
runtime ผิดธรรมชาติ
behavior แปลกจากผู้เล่นทั่วไป

ไม่ว่าจะผ่าน Integrity หรือไม่
ระบบก็จะเริ่มมองว่าเสี่ยงทันที

สิ่งที่จะข้ามมันไปได้คือความพยายามแหละทดสอบไปเลื่อยๆ

แหละเพิ่มเติมเกมไม่ได้เหมือนกันไปซะทุกเกมในแต่ละเกม ก็ มี Api ที่
แตกต่างกัน self protect 
---

README นี้มีไว้เพื่อความเข้าใจระบบ
ไม่ใช่คู่มือโกง
ไม่สนับสนุนการละเมิดกติกาของเกม
