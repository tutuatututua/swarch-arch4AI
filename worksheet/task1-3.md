# Tutorial 6 worksheet: serving patterns

**Name / ชื่อ:** นที จิระพงษา

**Student ID / รหัสนิสิต:** 6970144221

This is **individual work**: each student completes and submits their own copy, in their own fork.

*งานนี้เป็น**งานเดี่ยว** นิสิตแต่ละคนทำและส่งงานของตัวเอง โดยใช้ forked repository ของตัวเอง*

Copy result rows from `results/runs.csv` (or the load test's output) into the tables. Keep answers short: one to three sentences each. Answer in Thai or English.

ตอนที่ตอบ ให้นิสิตคัดลอกผลจาก `results/runs.csv` (หรือจาก output ของ load test) ลงในตาราง ตอบสั้นๆ ข้อละ 1–3 ประโยค จะตอบเป็นภาษาไทยหรือภาษาอังกฤษก็ได้

## Task 0: Baseline (copy the baseline results from `results/runs.csv` here / copy ผลลัพธ์ของ baseline จาก `results/runs.csv` มาวางที่นี่)

| Run | p50 ms | p95 ms | accuracy | large calls |
|---|---|---|---|---|
| t0-large | 507 | 615 | 93.5% | 100% |
| t0-small | 131 | 259 | 72.5% | 0% |

**Q0.** What does the large model buy, and what does it cost?

*large model ให้อะไรเพิ่ม และต้องแลกด้วยอะไร*

> large model ทำให้ accuracy เพิ่มจาก 72.5% เป็น 93.5% แต่ช้ากว่าประมาณ 4 เท่า และเสียเงินทุกรีวิว เพราะเรียก large model 100%

## Task 1: Fallback prediction (copy the Task 1 results from `results/runs.csv` here / copy ผลลัพธ์ของ Task 1 จาก `results/runs.csv` มาวางที่นี่)

| Run | run time (s) | p50 ms | p95 ms | errors | pending | accuracy |
|---|---|---|---|---|---|---|
| t1-before | 34.6 | 256 | 400 | 27 | 0 | 92.5% |
| t1-after | 9.8 | 392 | 1045 | 0 | 25 | 93.1% |

**Q1.1** In `t1-before`, p50 can look *better* than the baseline. Why is p50 misleading here? Which column tells the real story?

*ใน `t1-before` ค่า p50 อาจดู "ดีกว่า" baseline ทำไม p50 จึงทำให้เข้าใจผิดในกรณีนี้ และ column ไหนบอกความจริง*

> p50 นับแค่ request ที่สำเร็จเท่านั้น จะไม่เห็น request ที่ค้างจนล้มเหลว column ที่บอกความจริงคือ errors  และ run time

**Q1.2** The hang affects only 10% of large-model calls. Why is it worse for the system than 10% of reviews being slow? Think about what a waiting request holds on to in the review service and the gateway.

*การค้างเกิดกับ calls ไปยัง large model แค่ 10% ทำไมจึงส่งผลต่อระบบแย่กว่าการที่รีวิว 10% ช้าลงเฉยๆ ลองคิดว่า request ที่รออยู่ถือทรัพยากรอะไรไว้ใน review service และ gateway*

> request ที่ค้างถือ connection ไว้ทั้งใน review service และ gateway จนกว่าจะหมดเวลา ทำให้ request อื่นที่ไม่เกี่ยวกับการค้างต้องรอคิวหรือ error ไปด้วย

**Q1.3** Pending reviews have no label. Suggest what should happen to them, and name the topology from the lecture that does it.

*รีวิวที่ pending ยังไม่มี label ควรจัดการกับรีวิวเหล่านี้อย่างไร และ topology ใดจาก lecture ที่ทำแบบนั้น*

> เก็บไว้ในคิวแล้วให้ worker มาติด label ใหม่ภายหลังเมื่อ large model กลับมาปกติ เป็น topology แบบ asynchronous / batch

**Q1.4** We chose a 1-second timeout. What goes wrong if it is 0.1 s? And if it is 10 s?

*เราเลือก timeout 1 วินาที จะเกิดปัญหาอะไรถ้าตั้งเป็น 0.1 วินาที และถ้าตั้งเป็น 10 วินาที*

> 0.1 วินาที จะทำให้ large model timeout เกือบหมดและรีวิวกลายเป็น pending แทบทั้งหมด ส่วน 10 s ยาวเกินไป request ที่ค้างจะถือทรัพยากรนานจน pool เต็มและ p95/run time จะสูงขึ้นมาก

## Task 2: Cascade (copy the Task 2 results from `results/runs.csv` here / copy ผลลัพธ์ของ Task 2 จาก `results/runs.csv` มาวางที่นี่)

| Run | threshold | p50 ms | p95 ms | accuracy | large calls |
|---|---|---|---|---|---|
| t2-0.6 | 0.6 | 62 | 499 | 91.5% | 38% |
| t2-0.7 | 0.7 | 358 | 668 | 93.0% | 53% |
| t2-0.8 | 0.8 | 435 | 658 | 93.5% | 64% |
| t2-0.9 | 0.9 | 461 | 584 | 93.5% | 77% |

**Q2.1** Which threshold would you deploy for CampusEats, and why? Name the quality you give up.

*จะเลือก threshold ใดสำหรับ deploy ใน CampusEats เพราะอะไร และต้องยอมเสีย quality ใดไป*

> เลือก 0.6 เพราะ p50 เหลือ 62 ms และเรียก large model แค่ 38% โดยยอมเสีย accuracy ลงเล็กน้อย

**Q2.2** Look at p50 and p95 for your threshold. Why does the cascade help p50 much more than p95?

*ดูค่า p50 และ p95 ของ threshold ที่เลือก ทำไม cascade จึงช่วย p50 ได้มากกว่า p95 มาก*

> รีวิวส่วนใหญ่ใช้ small model จึงเร็ว และเป็นตัวกำหนด p50 แต่ p95 ตกอยู่ในกลุ่ม 38% ที่ต้องเรียก large model

**Q2.3** The cascade trusts the small model's confidence. Give one situation where that trust is misplaced, and its effect on users.

*cascade เชื่อ confidence ของ small model ยกตัวอย่างหนึ่งสถานการณ์ที่ไม่ควรเชื่อ และผลที่เกิดกับผู้ใช้*

> รีวิวประชดหรือใช้สแลง เช่น "อร่อยมาก รอแค่ชั่วโมงเดียวเอง" small model อาจเห็นคำว่า "อร่อยมาก" แล้วมั่นใจสูงว่า positive จึงไม่ส่งต่อให้ large model ผลคือได้ label ผิด และเจ้าของร้านเห็น sentiment ดีเกินจริง

## Task 3: Cached predictions (copy the Task 3 results from `results/runs.csv` here / copy ผลลัพธ์ของ Task 3 จาก `results/runs.csv` มาวางที่นี่)

| Run | p50 ms | p95 ms | accuracy | large calls | cache hits |
|---|---|---|---|---|---|
| t3-cache | 67 | 497 | 93.0% | 38% | 35% |

**Q3.1** Which reviews benefit from the cache, and which never will?

*รีวิวแบบไหนได้ประโยชน์จาก cache และแบบไหนไม่มีวันได้ประโยชน์*

> รีวิวสั้นที่ข้อความซ้ำกันเป๊ะเช่น "อร่อย", "ดีมาก" ได้ประโยชน์ ส่วนรีวิวยาวที่มีข้อความเฉพาะตัวจะไม่มีวัน hit cache เพราะ key ต้องตรงกันทุกตัวอักษร จะไม่ได้ประโยชน์

**Q3.2** Next week you deploy a better large model. What must happen to the cache, and how would you design for it?

*สัปดาห์หน้าจะ deploy large model ตัวใหม่ที่ดีกว่าเดิม ต้องทำอะไรกับ cache และจะออกแบบรองรับเรื่องนี้อย่างไร*

> ต้อง invalidate cache เพราะ label เดิมมาจาก model เก่า ควรออกแบบให้ cache key มี model version เมื่อ deploy model ใหม่ entry เก่าจะไม่ถูกใช้และหมดอายุไปเอง

## Debrief

**Q4.** Name one thing you learned from these measurements that the lecture did not cover.

*ยกตัวอย่างหนึ่งสิ่งที่ได้เรียนรู้จากการวัดผลครั้งนี้ ที่ไม่มีพูดถึงใน lecture*

> p50 ที่ดีขึ้นอาจหมายถึงระบบที่แย่ลงเช่น t1-before มี p50 256 ms ดีกว่า baseline แต่มี 27 errors และใช้เวลานานกว่า t1-after ถึง 3.5 เท่า จึงต้องดูหลาย factor รวมกัน
