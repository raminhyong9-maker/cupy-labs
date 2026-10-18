# REA Playbook — ใช้ REA ถอดกลไกบนเครื่องนี้

> อัปเดต 18 ต.ค. 2569 · ทดสอบจริงบน Ubuntu 24.04 · Node 26.7.0 · RAM 1.9 GB

## REA คืออะไร

`morluto/rea` (MIT) — MCP server + CLI สำหรับ reverse engineering: อ่าน **shipped artifact** (JS bundle, ASAR, APK, binary, firmware) แล้วคืน **Evidence** ที่อ้างอิงได้ (มี `evidence_id`, digest, limitations) แทนการเดา

- ใช้เมื่อ: ต้องรู้ว่าของที่ deploy แล้วทำงานอย่างไร, เทียบเวอร์ชัน, decompile, เก็บหลักฐานพฤติกรรม
- **ไม่ใช้** เมื่อ: อ่าน source repo ที่มีครบอยู่แล้ว (ใช้ tools ปกติ)

## สถานะบนเครื่องนี้

```
สกิล:  ~/.hermes/skills/reverse-engineer-anything   (v42, 6 references)
MCP:   ~/.hermes/config.yaml → mcp_servers.rea = npx -y rea-agents@6.5.0 mcp
ตรวจ:  hermes mcp test rea   → ✓ Connected · 207 tools
CLI:   npx -y rea-agents@6.5.0 <command>
```

## ใช้ผ่าน CLI (ไม่ต้องพึ่ง MCP)

```bash
# 1) วิเคราะห์ทั้งโฟลเดอร์ (JS/Electron/browser)
npx -y rea-agents@6.5.0 analyze-javascript-application ./artifacts --json > evidence/rea-analysis.json

# 2) สรุปย่อจาก Evidence (ไม่ต้องอ่าน 2 MB เข้า context)
node -e 'const d=require("./evidence/rea-analysis.json");
const s=d.normalized_result.statistics, g=d.normalized_result.graph;
console.log(s, {nodes:g.nodes.length, edges:g.edges.length});'

# 3) ตรวจความพร้อมของเอนจิน
npx -y rea-agents@6.5.0 doctor --json
```

**ตัวเลขจริงที่วัดได้ (โฟลเดอร์ 1.5 MB / 4 ไฟล์):** 113–200 วินาที · RAM สูงสุด **420 MB** · Evidence ~190 KB · AST 321,225 nodes

## กฎการทำงาน

1. **อ่าน Evidence ไม่ใช่ดิบ** — `--json > file.json` เสมอ แล้วค่อยสกัดสรุป (ผลจริงอาจใหญ่หลาย MB)
2. **เก็บ limitations ไว้** — REA ระบุ unknowns/ข้อจำกัดตรง ๆ (ตัวอย่าง: 48 unknowns) ห้ามตัดทิ้งเพื่อให้รายงานดูสมบูรณ์
3. **ไม่ฝ่า auth** — endpoint ที่ต้องใช้ token ให้บันทึกว่า 401 แล้วหยุด; ใช้เฉพาะสิ่งที่เบราว์เซอร์โหลดตามปกติ
4. **Static ≠ runtime** — REA บอกโครงสร้าง/ความสัมพันธ์; พฤติกรรม (fps, การเคลื่อนกล้อง) ต้องวัดด้วย CDP/browser เพิ่ม
5. **Bundle ของบุคคลที่สาม** — ศึกษาได้ แต่ **อย่า commit โค้ดเขา**; commit แค่ hash + สคริปต์ดาวน์โหลด + บทวิเคราะห์ของเรา

## ข้อจำกัดบนเครื่อง RAM 1.9 GB

| งาน | สถานะ |
|---|---|
| JavaScript / browser / Electron bundle | ✅ ใช้ได้ (420 MB peak) |
| Runtime observation (process/PTY) | ✅ ใช้ได้ |
| ถอด native (Ghidra / IDA / Hopper) | ❌ ยังไม่มีเอนจิน + แรมไม่พอ |
| Android (JADX) | ❌ ต้องมี `REA_JADX_MCP_JAR` เอง |

## MCP tools ใน Hermes

หลังลงทะเบียนแล้ว tool ชุด `mcp_rea_*` (207 ตัว) จะโผล่ **ในเซสชันใหม่** เพราะ Hermes ค้นพบ MCP ตอนเริ่มเซสชัน — ถ้าจำเป็นต้องใช้ทันทีในเซสชันเดิม ให้ใช้ CLI ข้างบน

## ตัวอย่างผลลัพธ์ที่ใช้ได้จริง

ดู [`prezi-mechanics.md`](prezi-mechanics.md) — งานแกะ Prezi ที่ใช้ทั้ง REA (โครงสร้าง) และ CDP (พฤติกรรม) แล้วประกอบเป็นข้อสรุปเดียว
