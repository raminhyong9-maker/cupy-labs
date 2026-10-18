# Cupy Labs — ห้องทดลองของเรา

รวมงานทดลองหน้าเว็บ/กลไกที่ **วัดผลได้** ใช้ในห้องเรียนและต่อยอดในโปรเจกต์จริง
ทุกชิ้นเขียนขึ้นเองทั้งหมด ไม่มีโค้ดของบุคคลที่สาม

**เว็บสด:** https://raminhyong9-maker.github.io/cupy-labs/

## Labs

| Lab | คืออะไร | เปิดดู |
|---|---|---|
| **ZUI — แคนวาสซูมได้** | กรอบหลายใบบนผืนเดียว คลิกแล้วกล้องบินเข้า (ถอยออก → เคลื่อน → เข้า) · wheel ซูมที่เคอร์เซอร์ · ลากผืน · breadcrumb ย้อนขึ้น | [`labs/zui.html`](labs/zui.html) |
| **คอลเลกชันเทมเพลต 6 สไตล์** | Aurora Glass · Paper Editorial · Swiss Grid · Brutal Pop · Pastel Soft · Kinetic Type — เนื้อหาชุดเดียวกัน เทียบกันตรง ๆ | [`labs/templates/`](labs/templates/index.html) |

## Docs

| เอกสาร | เนื้อหา |
|---|---|
| [`docs/prezi-mechanics.md`](docs/prezi-mechanics.md) | แกะกลไก Prezi: WebGL2 viewer, PDOM + MAC token, enum การนำทาง, สูตรกล้อง, 2 สิ่งที่ไม่ควรลอก |
| [`docs/rea-playbook.md`](docs/rea-playbook.md) | วิธีติดตั้ง/ใช้ REA บนเครื่องนี้ + evidence workflow + ข้อจำกัด RAM |

## หลักที่ใช้กับทุก lab

1. **วัดก่อนสรุป** — ตัวเลขมาจาก pixel/DOM/network จริง ไม่ใช่การเดา
2. **0 KB ถ้าทำได้** — CSS/JS เองก่อน ค่อยเพิ่มไลบรารีเมื่อจำเป็นจริง
3. **มือถือต้องผ่าน** — เป้ากด ≥44px, ฟอนต์ ≥13px, ไม่ล้นจอ
4. **เข้าถึงได้** — เคารพ `prefers-reduced-motion`, คอนทราสต์ ≥4.5:1

## License

MIT — ดู [LICENSE](LICENSE)
