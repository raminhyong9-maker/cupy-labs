# กลไก Prezi — บันทึกเชิงเทคนิค

> วัดจาก Prezi viewer รุ่น `viewer-container.0.2.1955` · ตัวอย่าง `prezi.com/p/xmpasupfkavr/embed/?autoplay=1`
> อ้างอิงผล REA: `evidence/rea-analysis.json` (evidence `ev_4144a6d2…`)

---

## 1. สถาปัตยกรรม

| องค์ประกอบ | ค่าที่วัดได้ |
|---|---|
| คอนเทนเนอร์ | `#webgl-viewer-app` · `.next-webgl-viewer-container` · `.webgl-viewer-ui-container` |
| ผิวเรนเดอร์ | `<canvas>` 1 ใบ **1440×866** (เต็ม viewport) |
| คอนเทกซ์ | `WebGL2RenderingContext` — `getParameter(VERSION)` = *WebGL 2.0 (OpenGL ES 3.0)* |
| เฟรมเรต | 60 rAF/s (นับ callback 2 วินาที) |
| DOM ตอนเล่น | ~1,300 elements แต่ **ไม่มีเนื้อหา** อยู่ใน DOM |
| สถาปัตยกรรมแอป | React 17 + webpack chunks (ไม่ใช่ Electron — REA พบ IPC/browser-window = 0) |

**นัยสำคัญ:** ข้อความ การ์ด รูป ทุกอย่างถูก bake เป็น texture แล้ววาดเป็น quad ในฉากเดียว → ซูมได้ไม่จำกัดโดยไม่เสีย layout, แต่แลกด้วยการค้นหา/คัดลอก/เข้าถึงด้วย screen reader ที่ทำไม่ได้

---

## 2. รูปแบบข้อมูล

```
GET https://rosetta0701.prezi.com/presentation/<oid>/
    → 200 application/json
      { header: "MAC id=…, ts=…, nonce=…, ext=…, mac=…",
        method: "GET", url: "https://0701.static.prezi.com/presentation/<oid>/",
        name: "readpresentationtoken" }

GET <url ข้างบน>  พร้อม header  authorization: MAC …
    → binary · magic 00 00 00 04 + zlib stream · Content-Type: application/xml (หลอก)
    → ตัวอย่างที่วัดได้ 36,872 ไบต์
```

- ชื่อ request header `X-PdomVersion` → เนื้อหาเป็น **PDOM** (Prezi Document Object Model) แบบไบนารี ไม่ใช่ JSON
- เรียก endpoint ตรง ๆ ด้วย curl → **401 Authentication is possible but has failed** (ต้องมี MAC token ราย request) — รีโปนี้ไม่พยายามฝ่า

---

## 3. โมเดลการนำทาง

จาก enum ใน `webgl-viewer-app.chunk.js`:

```
None · Pan · Zoom
StepNextKeyboard · StepPreviousKeyboard · GoUpKeyboard
VideoPlayEmbedded · VideoPlayYoutube · VideoPlayVimeo
ClickToObject · SnapToObject
Topic · Subtopic · Animation
ZoomArea · TopicZoomArea · Baseline · Thumbnail · Ppt · Multiple
```

API ของ navigator ที่มีอยู่จริง:

```
goToNext() · goToPrevious() · goToStart() · goToEnd()
goToNavigablePathIndex(i)   ← ลำดับเป็น "เส้นทาง" ไม่ใช่ดัชนีสไลด์
goToOverview() · zoomIn() · zoomOut() · goUp()
```

ชนิดออบเจกต์ (20): `Topic · Subtopic · Text · Chart · Image · Video · Shape · Arrow · Symbol · Pdf · Line · TextSnippet · Url · Background · Animation · ZoomArea · Thumbnail · Baseline · TopicZoomArea · Ppt`

**อ่านความหมาย:** `goUp()` + `Topic → Subtopic` = **semantic zoom** (ถอยออกไปชั้นแม่ของเนื้อหา) ไม่ใช่แค่ย่อขนาด — จุดนี้คือสิ่งที่ต่างจากสไลด์โชว์ทั่วไป

---

## 4. คณิตศาสตร์กล้อง

```js
// world → screen
screenX = (worldX - cam.x) * cam.scale + vw / 2
screenY = (worldY - cam.y) * cam.scale + vh / 2

// fit กรอบ (x, y, w, h) โดยเหลือขอบ p
scale = min(vw / (w * (1 + 2p)), vh / (h * (1 + 2p)))
cam   = { x: x + w/2, y: y + h/2, scale }

// ซูมที่ตำแหน่งเมาส์
wx = (mx - vw/2) / cam.scale + cam.x
cam.x = wx - (mx - vw/2) / newScale      // เช่นเดียวกันกับแกน y
```

**วิถี 3 ช่วง (แบบที่ Prezi ใช้ และวัดได้จากภาพ)**

| ช่วง | สิ่งที่เกิด | หลัก |
|---|---|---|
| 0–35% | ถอยออก | `scale → min(from, to) × 0.5` |
| 35–68% | เคลื่อน | x,y ไปยังเป้า สเกลคงที่ |
| 68–100% | เข้า | `scale → to` |

เวลา ~1.4–2.2 s ตามระยะ (ไกล = นาน) · easing ประมาณ ease-in-out cubic

**หลักฐาน:** กด `ArrowRight` แล้วเทียบภาพหน้าจอ (resize 360×225, threshold 12/255)

| คู่ที่เทียบ | พิกเซลเปลี่ยน |
|---|---|
| ก่อน → 420 ms | 34.7% |
| ก่อน → 2 s | 40.5% |
| 420 ms → 2 s | 44.8% |

ภาพที่ 420 ms เป็น **มุมกว้าง** (เห็นวงเหลือง/กากบาทม่วงรอบกรอบ) → ยืนยันการถอยออกกลางเที่ยวบิน

---

## 5. ผลจาก REA (static analysis)

| เมตริก | ค่า |
|---|---|
| ไฟล์ที่วิเคราะห์ | 4 (1,551,405 ไบต์) |
| AST nodes | 321,225 |
| graph nodes / edges | 35 / 45 |
| modules | 4 (IIFE bundle — ไม่มี ESM/CJS boundary) |
| Electron IPC / browser windows | 0 / 0 |
| unknowns ที่ระบบระบุ | **48** |
| parse failures | 0 |

REA ยืนยันว่าเป็น **เว็บแอป React + webpack ล้วน** ไม่มีชั้น Electron และไม่สามารถ fingerprint ไลบรารีได้ (โค้ด minify + ไม่มี source map สาธารณะ) — ข้อจำกัดนี้บันทึกไว้ใน `docs/EVIDENCE.md`

---

## 6. 2 สิ่งที่ไม่ควรลอก

1. **ถอยออกทุกครั้ง** — ในแอปทำงาน (ใบงาน/ฟอร์ม) ทำให้เสียเวลา 0.6–1.2 s ต่อคลิกและเวียนหัว
   → ถอยออกเฉพาะเมื่อกระโดดไกลกว่า ~1.5 เท่าของ viewport; ระยะใกล้ใช้ซูมตรง 180–260 ms
2. **ยัดเนื้อหาทั้งหมดลง canvas** — ค้นหา, คัดลอก, screen reader, SEO ใช้ไม่ได้
   → ใช้ DOM สำหรับข้อความ + transform layer เดียว (แบบ `demo/zui.html`) และใช้ WebGL เฉพาะฉากที่จำเป็น

---

## 7. ถ้าจะนำไปใช้

- **กรอบ = เป้ากล้อง** สำหรับแผนภาพบทเรียน (แผนหน่วย → บท → แบบฝึก)
- **`goUp()`** = ปุ่ม "ย้อนขึ้นหนึ่งชั้น" ในหน้าบทเรียน
- **แถบความคืบหน้าแบบ path** ไม่ใช่เลขหน้า
- ลำดับทดลอง: เปิด `demo/zui.html` → ใส่เนื้อหาจริง 1 บท → วัดเวลาถึงเนื้อหา ≤2 s → ค่อยตัดสินใจทำในแอปจริง
