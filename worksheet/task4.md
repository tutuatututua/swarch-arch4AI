# Task 4: Deployment topologies and trade-offs

**Name / ชื่อ:** นที จิระพงษา

**Student ID / รหัสนิสิต:** 6970144221

This is **individual work**: each student completes and submits their own copy, in their own fork.

*งานนี้เป็น**งานเดี่ยว** นิสิตแต่ละคนทำและส่งงานของตัวเอง ใน fork ของตัวเอง*

**Scenario (A, B or C) / Scenario ที่เลือก:** B. Canteen kiosk

Choose **one** scenario below. If the last digit of your student ID is odd, take A or B; if it is even, take B or C. Use your measurements from Tutorial 6 as evidence: cite run names from `results/runs.csv`. Answer in Thai or English.

เลือก scenario **หนึ่ง** ข้อด้านล่าง ถ้าเลขตัวสุดท้ายของรหัสนิสิตเป็นเลขคี่ ให้เลือก A หรือ B ถ้าเป็นเลขคู่ ให้เลือก B หรือ C ใช้ผลการวัดจาก tutorial ที่ทำไปเป็นข้อมูลอ้างอิง โดยอ้างชื่อ run จาก `results/runs.csv` จะตอบเป็นภาษาไทยหรือภาษาอังกฤษก็ได้

Facts that hold in every scenario / ข้อเท็จจริงที่ใช้กับทุก scenario:

- The large model serves at most **20 calls per second** (4 calls at once × about 200 ms each), and every call costs money.

  *large model รับได้สูงสุด **20 calls ต่อวินาที** (พร้อมกัน 4 calls × ประมาณ 200 ms ต่อ call) และทุก call มีค่าใช้จ่าย*
- The small model is fast and cheap enough to run anywhere, including inside the review service or on a kiosk, but it is much less accurate on its own.

  *small model เร็วและถูกพอที่จะรันที่ไหนก็ได้ รวมถึงภายใน review service หรือบน kiosk แต่ถ้าใช้ตัวเดียวจะแม่นน้อยกว่ามาก*

## Scenarios

**A. Owner alerts.** When a customer posts a negative review, the restaurant owner gets a push notification within **2 seconds**. At the lunch peak, CampusEats receives **40 reviews per second**. A missed alert is acceptable, but a wrong alert (for example, "your customer is unhappy" when they are happy) makes owners stop trusting alerts and, in the end, ignore them.

*เมื่อลูกค้าโพสต์รีวิวเชิงลบ เจ้าของร้านจะต้องได้รับ push notification ภายใน **2 วินาที** เวลาช่วงพีคตอนเที่ยง CampusEats ได้รับรีวิวจำนวน **40 รายการต่อวินาที** การพลาด alert ไปบ้างนั้นยอมรับได้ แต่ alert ที่ผิด (เช่น "ลูกค้าไม่พอใจ" ทั้งที่จริงๆ แล้วพอใจ) จะทำให้เจ้าของร้านไม่เชื่อถือและเลิกสนใจ alert ในที่สุด*

**B. Canteen kiosk.** A tablet in each canteen lets students leave a review on the spot. However, the canteen Wi-Fi is poor: the tablets lose their connection several times a day, for minutes at a time. The system must accept every review, even while a tablet is offline. Restaurant owners need the labels **by the end of the day**.

*มี tablet ในแต่ละโรงอาหารให้นิสิตรีวิวได้ทันที อย่างไรก็ดี การเชื่อมต่อ Wi-Fi ในโรงอาหารไม่ดี ทำให้ tablet หลุดการเชื่อมต่อวันละหลายครั้ง ครั้งละหลายนาที ระบบจะต้องรับรีวิวได้ทุกรายการแม้ขณะที่กำลัง offline อยู่ และเจ้าของร้านต้องการ label **ภายในสิ้นวัน***

**C. Weekly report.** Restaurant owners want a weekly sentiment report. Nothing in producing this report has to be real-time. **Accuracy matters most** and **cost must be minimal**.

*เจ้าของร้านต้องการรายงาน sentiment รายสัปดาห์ ไม่มีอะไรที่จะต้องทำแบบ real-time ในการออกรายงานนี้ ความแม่นยำ **(accuracy) สำคัญที่สุด** และ **ค่าใช้จ่ายต้องน้อยที่สุด***

## Topologies and patterns to choose from / Topology และ pattern ที่เลือกใช้ได้

| | Topology | Where the model runs / โมเดลรันที่ไหน |
|---|---|---|
| T1 | Model as a service, synchronous | Inference service, called for each review (today's baseline)<br>*inference service ถูกเรียกทุกรีวิว (baseline ของวันนี้)* |
| T2 | Model as a library | Inside the review service, or on the device<br>*ภายใน review service หรือบนอุปกรณ์* |
| T3 | Hybrid | Small model locally, large model as a remote service<br>*small model รันในเครื่อง large model เป็น remote service* |
| T4 | Asynchronous / batch | Reviews are queued (broker, W10) or stored, and scored later in bulk<br>*รีวิวถูกเข้าคิว (broker, W10) หรือเก็บไว้ แล้วติด label ทีหลังเป็นชุด* |

Combine one topology with any of the patterns from Tutorial 6: **fallback**, **cascade** and **cache**.

*เลือก topology หนึ่งแบบ แล้วผสมกับ pattern จาก Tutorial 6 ได้ตามต้องการ: **fallback**, **cascade** และ **cache***

## 1. Quality requirements

The three most important quality requirements for your scenario, each with a number where possible.

*quality requirements ที่สำคัญที่สุด 3 ข้อของ scenario ที่เลือก ระบุตัวเลขเมื่อทำได้*

| # | Quality | Requirement |
|---|---|---|
| 1 | Availability / durability | tablet รับรีวิวได้ 100% แม้ offline หลายนาที และไม่มีรีวิวหาย|
| 2 | Timeliness | ทุกรีวิวมี label ก่อนสิ้นวันเช่น 23:59 ไม่ต้องเป็น real-time |
| 3 | Accuracy | accuracy ≥ 93% (ระดับ t0-large) โดยเรียก large model ไม่เกิน 65% ของรีวิว |

## 2. Design / การออกแบบ

Your topology, the patterns you add, and a container-level diagram (C4 level 2). The C4 diagram must be written in Mermaid.

*topology ที่เลือก pattern ที่เพิ่ม และ diagram ระดับ container (C4 level 2) เขียนด้วย Mermaid*

Topology ที่เลือกเป็น T4 Asynchronous / batch Patterns เป็น cascade (threshold 0.8) + cache + fallback (call ที่ timeout กลับเข้าคิวให้ retry)

kiosk เก็บรีวิวไว้ใน outbox ในเครื่องก่อน แล้ว sync ขึ้น server เมื่อกลับมา online review service บันทึกรีวิวเป็น pending แล้วส่งเข้าคิว batch worker ดึงจากคิวมาติด label ด้วย cascade + cache ตามจังหวะที่ large model รับไหว

```mermaid
C4Container
    title Scenario B: canteen kiosk (async / batch)
    Person(student, "Student", "เขียนรีวิวที่ kiosk ในโรงอาหาร")
    Person(owner, "Restaurant owner", "ดู label ของร้านภายในสิ้นวัน")
    System_Boundary(campuseats, "CampusEats") {
        Container(kiosk, "Kiosk app", "Tablet app, local SQLite outbox", "รับรีวิวแม้ offline และ sync เมื่อกลับมา online")
        Container(gateway, "API gateway", "Python, FastAPI", "จุดเข้าเดียวของ client")
        Container(review, "Review service", "Python, FastAPI", "บันทึกรีวิวสถานะ pending และ publish เข้าคิว")
        ContainerDb(db, "Review DB", "PostgreSQL", "รีวิวและ label")
        ContainerQueue(queue, "Message broker", "RabbitMQ", "คิวรีวิวที่รอ label")
        Container(worker, "Labeling worker", "Python", "ดึงคิวเป็นชุด ใช้ cascade + cache และ retry เมื่อ timeout")
        Container(inference, "Inference service", "Python, FastAPI, scikit-learn", "small model และ large model")
    }
    Rel(student, kiosk, "เขียนรีวิว")
    Rel(kiosk, gateway, "Sync รีวิวที่ค้างใน outbox", "HTTP/JSON")
    Rel(gateway, review, "ส่งต่อ request", "HTTP/JSON")
    Rel(review, db, "บันทึกรีวิว")
    Rel(review, queue, "Publish review_id", "AMQP")
    Rel(worker, queue, "Consume เป็นชุด", "AMQP")
    Rel(worker, inference, "POST /predict (cascade)", "REST")
    Rel(worker, db, "เขียน label")
    Rel(owner, gateway, "ดู label", "HTTP/JSON")
    UpdateLayoutConfig($c4ShapeInRow="3", $c4BoundaryInRow="1")
```

## 3. Evidence / หลักฐาน

Which runs support your choice? Show the numbers and, for scenario A, the capacity arithmetic: how many large-model calls per second does your design need at the peak?

*run ไหนที่สนับสนุนการตัดสินใจของนิสิต ให้นิสิตแสดงตัวเลขประกอบ และสำหรับ scenario A ให้นิสิตแสดงการคำนวณ capacity ด้วย เพื่อแสดงว่าในช่วงพีคนั้น design ของนิสิตต้องใช้ large model กี่ calls ต่อวินาที*

| Run | What it shows / แสดงอะไร |
|---|---|
| t0-small vs t0-large | small model อย่างเดียวได้แค่ 72.5% เทียบกับ 93.5% จึงไม่ควรติด label บน kiosk ด้วย small model อย่างเดียว |
| t2-0.8 | cascade 0.8 ได้ accuracy 93.5% เท่ากับ t0-large แต่เรียก large แค่ 64% ส่วน latency p95 ที่สูงไม่สำคัญเพราะเป็น batch |
| t3-cache-0.8 | cache hit 35% ลด large calls ลงอีก (เหลือ 40% ที่ threshold 0.8 จาก 64% ใน t2-0.8) และ accuracy ยัง 93.5% จึงประหยัดค่าใช้จ่าย |
| t1-after | timeout + pending ทำให้ errors เป็น 0 รีวิวที่ค้างไม่หายแต่รอติด label ทีหลัง ตรงกับแนวคิด queue + retry |

## 4. ADR

Use the ADR format from W5.

*ใช้รูปแบบ ADR จาก W5*

**Title:** ใช้ asynchronous / batch labeling พร้อม offline outbox สำหรับ canteen kiosk

**Status:** Proposed

**Context:** Wi-Fi ในโรงอาหารหลุดวันละหลายครั้ง ครั้งละหลายนาที แต่ต้องรับรีวิวทุกรายการ เจ้าของร้านต้องการ label แค่ภายในสิ้นวัน ส่วน large model รับได้สูงสุด 20 calls/s และมีค่าใช้จ่ายทุก call

**Decision:** kiosk เก็บรีวิวไว้ใน local outbox แล้ว sync เมื่อ online รีวิวเข้าคิว (T4) และให้ worker ติด label เป็นชุดด้วย cascade (threshold 0.8) + cache ถ้า call ไหน timeout ให้ส่งกลับเข้าคิวเพื่อ retry

**Consequences:**

- Positive: ไม่มีรีวิวหายแม้ offline accuracy เท่ากับ large model (93.5%, t2-0.8) โดยเรียก large น้อยลง worker คุมอัตรา call ให้ไม่เกิน 20/s ได้
- Negative: label ไม่ได้ทันที (eventual consistency) และต้องดูแลส่วนประกอบเพิ่ม (outbox, broker, worker) รวมถึงต้องกันรีวิวซ้ำเวลา sync ด้วย idempotency key

**Alternatives considered:** at least one topology you rejected, and why.

*อย่างน้อยหนึ่ง topology ที่ไม่เลือก และเหตุผล*

> - **T1 synchronous:** ใช้ไม่ได้ตอน kiosk offline และไม่ต้องเร็วขนาดนั้นอยู่แล้ว
> - **T2 small model บน kiosk อย่างเดียว:** ทำงาน offline ได้ แต่ accuracy แค่ 72.5% (t0-small)
> - **T3 hybrid:** ได้ label ชั่วคราวทันที แต่ไม่มีใครต้องการ label ทันที จึงเพิ่มความซับซ้อนบน kiosk โดยไม่จำเป็น
