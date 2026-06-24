# Adaptive Web Platform

> **Status:** Future Plan / Concept
> **Last updated:** 2026-06-24
> **Tagline:** เว็บแอปที่แก้ไขตัวเองตามภาษาธรรมชาติจากผู้ใช้

---

## 1. Overview

Web application ที่ผู้ใช้สามารถ "สั่ง" เปลี่ยนแปลงระบบได้ด้วยภาษาธรรมชาติ (natural language) โดยมี LLM เป็นแกนกลางที่แปลคำสั่งเป็นการแก้ไขจริง — ทั้ง UI, feature, และ workflow — แบบ dynamic

แนวคิดหลัก: LLM มีอิสระในการตัดสินใจและดำเนินการเอง แต่ทุกการเปลี่ยนแปลง **มองเห็นได้** และ **ย้อนกลับได้** ผ่านระบบ admin ที่กำกับดูแลและตรวจสอบได้เสมอ

---

## 2. Core Components

### 2.1 Input Layer — ช่องทางรับ Message

รับคำสั่งจากผู้ใช้ผ่านช่องทางที่ไม่มีค่าใช้จ่าย รองรับทั้ง **ข้อความ (natural language)** และ **ภาพ (image)**

| ตัวเลือก | รายละเอียด | ข้อดี |
|---------|-----------|-------|
| **A. Discord Bot** *(แนะนำเป็นหลัก)* | ฟรี, API เปิด, รองรับ attachment (รูปภาพ) ในตัว, มี thread/channel แยกบริบทได้ | community-based, รับทั้ง text + image ได้เลย, จัดการ multi-user ง่าย |
| **B. In-app command bar** | chat input ในตัวเว็บแอป | ไม่พึ่ง third-party, จัดการ context/auth ง่าย |
| **C. Telegram Bot** | ทางเลือกแทน Discord | rate ผ่อนปรน |

**Decision:** เริ่มที่ **Discord Bot** เป็น primary channel → เพิ่ม in-app command bar เป็น optional channel ภายหลัง

#### Image Input Support

ผู้ใช้สามารถส่งภาพประกอบคำสั่งได้ (multimodal input) เช่น:

- แนบ **screenshot/mockup** ของ UI ที่ต้องการ → ระบบแปลภาพเป็นการเปลี่ยนแปลง layout/component
- แนบ **diagram/flowchart** ที่วาดมือ → ระบบตีความเป็น workflow
- แนบ **ภาพ error/bug** ที่เจอ → ระบบใช้ประกอบการ diagnose

กลไก: ภาพถูกส่งผ่าน Discord attachment → ผ่าน vision-capable LLM (image-to-instruction) → รวมกับ text command ใน context เดียวกันก่อนตัดสินใจ accept/reject

> **ข้อพิจารณา:** vision model มีต้นทุนสูงกว่า text-only — ต้องชั่งน้ำหนักกับข้อจำกัด API cost / GPU (ดู Open Questions)

### 2.2 Oversight Layer — Admin Site

หน้าจัดการแยกต่างหาก (web-based dashboard) ทำหน้าที่กำกับและตรวจสอบ

- **Change Log / Audit** — สรุปว่าโมเดลแก้อะไรไปบ้าง: diff ของแต่ละการเปลี่ยนแปลง, timestamp, คำสั่งต้นทาง, และเหตุผลที่ LLM ตัดสินใจ
- **Feature Map** — รายการ feature ที่มีในระบบ พร้อม dependency graph แสดงว่าแต่ละ feature เชื่อมโยงกันอย่างไร (เช่น feature A เรียกใช้ B → แก้ A อาจกระทบ C)
- **Diff Viewer** — เปรียบเทียบ before/after ก่อน deploy จริง
- **Rollback** — ย้อนกลับเวอร์ชันได้หากการแก้ไขผิดพลาด

### 2.3 Agentic Loop — Self-Validation

ระบบตรวจสอบและตัดสินใจเองก่อนนำคำสั่งไปใช้

- **หาบัคเอง** — รัน validation/test กับโค้ดหรือ config ที่ generate ออกมา ตรวจ syntax, type, และ logic conflict ก่อน apply
- **Accept / Reject User Request** — ประเมินคำสั่งของผู้ใช้:
  - ปลอดภัยและทำได้ → **accept** และดำเนินการ
  - ขัดแย้งกับ feature เดิม / เสี่ยงพังระบบ / เกินสิทธิ์ → **reject** พร้อมอธิบายเหตุผลกลับไปหาผู้ใช้
- **Self-Healing** — หากพบบัคหลัง apply พยายามแก้เองและ validate ซ้ำในลูป

### 2.4 Autonomous Workflow Engine

เว็บแอปสามารถสร้างและรัน workflow ได้เอง

- ผู้ใช้อธิบายขั้นตอนที่ต้องการเป็นภาษาธรรมชาติ → ระบบแปลเป็น workflow (ลำดับขั้น, เงื่อนไข, trigger)
- รองรับ multi-step automation ที่ทำงานต่อเนื่องโดยไม่ต้องสั่งทีละขั้น
- workflow ที่สร้างถูกบันทึกใน feature map และตรวจสอบได้ผ่าน admin

---

## 3. Design Principles

1. **Visible & Reversible** — ทุกการเปลี่ยนแปลงต้องมองเห็นได้และย้อนกลับได้
2. **Guardrailed Autonomy** — LLM มีอิสระในการตัดสินใจ แต่อยู่ภายใต้ guardrail ของ admin
3. **Transparency** — การ accept/reject และ self-validation ทำงานอัตโนมัติ แต่ผลลัพธ์ทั้งหมดถูกบันทึกเพื่อความโปร่งใส

---

## 4. Method — Scalability, Token Efficiency, Cost Optimization

หลักการออกแบบเชิงวิศวกรรมเพื่อให้ระบบ scale ได้และคุม cost ภายใต้ข้อจำกัด API budget

### 4.1 Token Efficiency

| เทคนิค | รายละเอียด | ผลที่ได้ |
|--------|-----------|---------|
| **Prompt caching** | cache system prompt + feature map ที่ stable (Anthropic คิด ~10% ของ input rate, OpenAI/Gemini ลดได้ถึง 90% บน cached portion) | ลด input cost 70–90% บนส่วนที่ซ้ำ |
| **Structured output (JSON)** | บังคับ output เป็น JSON + ตั้ง `max_tokens` แทน prose ยาว | output cost ลด (output แพงกว่า input 3–10 เท่า) |
| **Context compression / RAG** | ไม่ส่ง feature map หรือ codebase ทั้งก้อน — ดึงเฉพาะส่วนที่เกี่ยวข้องผ่าน retrieval | input token ลดมหาศาลเมื่อระบบโตขึ้น |
| **Lazy image processing** | เรียก vision model เฉพาะเมื่อมี attachment เท่านั้น | ตัด vision cost ออกจาก request ที่เป็น text ล้วน |
| **Summarize history** | สรุป conversation history เก่าแทนส่งดิบทั้งหมด | กัน context โตแบบ linear |

### 4.2 Cost Optimization

- **Model routing (สำคัญที่สุด)** — แยกงานตามความยาก: คำสั่งง่าย (rename, แก้สี, toggle) → cheap model; งานยาก (refactor หลายไฟล์, ออกแบบ workflow ใหม่) → frontier model เท่านั้น การ route ลด cost ได้ 10x+
- **Batch API** — งานที่ไม่ต้อง real-time (สรุป change log, วิเคราะห์ dependency แบบ async) → batch API ลด 50%
- **Cheap validation** — ใช้ rule-based / linter / type-checker ก่อนเรียก LLM diagnose (หาบัคด้วยเครื่องมือ deterministic ก่อนค่อย escalate ไป LLM)
- **Spending alerts** — ตั้ง budget alert + track cost per command เพื่อจับ spike

### 4.3 Scalability

- **Stateful checkpointing** — เก็บ state แต่ละขั้นของ agent เพื่อ resume/retry ได้โดยไม่รันใหม่ทั้ง loop (LangGraph มี PostgresSaver checkpointer ในตัว)
- **Per-channel context isolation** — แยก context ตาม Discord channel/thread กัน state ปนกันเมื่อ multi-user
- **Queue + worker** — คำสั่งเข้า queue แล้วประมวลผลด้วย worker pool กัน rate limit ชนและรองรับ concurrent users
- **Config-driven over code-gen** — apply การเปลี่ยนแปลงผ่าน config/feature-flag schema แทน generate โค้ดดิบ → ตรวจสอบและ rollback ง่าย ปลอดภัยกว่ามากเมื่อ scale

### 4.4 Estimated Token Usage

ประมาณการต่อ **1 คำสั่งของผู้ใช้** (รวม agentic loop ที่วนหลายรอบ) — ตัวเลขสมมติฐาน ควร log จริงใน pilot แล้ว extrapolate

| สถานการณ์ | Input tokens | Output tokens | หมายเหตุ |
|-----------|-------------|---------------|---------|
| คำสั่งง่าย (text, 1–2 loop) | 8,000–15,000 | 1,000–3,000 | system prompt + feature map ย่อ + คำสั่ง |
| คำสั่งกลาง (text, validate + retry) | 15,000–30,000 | 3,000–8,000 | มี self-validation loop |
| คำสั่งซับซ้อน (text, multi-step workflow) | 30,000–60,000+ | 8,000–20,000 | วน loop หลายรอบ + diagnose บัค |
| + Image input | +1,500–3,000 / ภาพ | — | vision model |

**ตัวอย่างคำนวณ cost จริง** (สมมติ 500 คำสั่ง/วัน, เฉลี่ย 20K input + 5K output, ใช้ mid-tier model ~$3/$15 per 1M):

```
Input:  500 × 20,000 × ($3 / 1,000,000)  = $30/วัน
Output: 500 × 5,000  × ($15 / 1,000,000) = $37.5/วัน
รวม ≈ $67.5/วัน → ~$2,025/เดือน (ก่อน optimize)
```

> ราคา LLM API ปี 2026: budget models เริ่มที่ ~$0.10–0.15/1M input (Gemini Flash, GPT Nano), mid-tier ~$2.5–3/1M input (GPT-5.4, Claude Sonnet), frontier ~$5/1M+ (Claude Opus, GPT-5.5). ตัวเลขเปลี่ยนเร็ว ควรเช็คล่าสุด

**หลัง optimize** (prompt caching 90% บน system prompt ที่ stable + model routing ส่ง 70% ของคำสั่งไป cheap model + batch งาน async):
- คาดลด cost ลง **60–80%** → เหลือราว **$400–800/เดือน** ที่ volume เดียวกัน
- ปกติ plan ลด cost ได้ ~20–30% ในไตรมาสแรกหลัง implement caching/routing/compression และควร buffer งบ 1.7–2x จากค่าคำนวณ base

#### Personal Use Scenario (Claude API)

สำหรับใช้ส่วนตัวคนเดียว volume ต่ำกว่ามาก (~10–30 คำสั่ง/วัน ไม่ได้สั่งตลอดเวลา) บนราคา Claude API (มิ.ย. 2026): **Opus 4.8 $5/$25, Sonnet 4.6 $3/$15, Haiku 4.5 $1/$5** ต่อ 1M token

สมมติ 20K input + 5K output/คำสั่ง, เดือน = ~22 วันใช้งานจริง:

| Volume (คำสั่ง/วัน) | Sonnet ตรงๆ | Sonnet + cache | Routing + cache |
|---|---|---|---|
| เบา (~10/วัน) | ~$30/เดือน | ~$21/เดือน | ~$12/เดือน |
| กลาง (~20/วัน) | ~$59/เดือน | ~$43/เดือน | ~$24/เดือน |
| หนัก (~30/วัน) | ~$89/เดือน | ~$64/เดือน | ~$36/เดือน |

ข้อสังเกตสำหรับ personal use:

- **ถูกกว่าที่คิดเยอะ** — volume ต่ำทำให้ cost อยู่หลักสิบดอลลาร์/เดือน ไม่ใช่หลักพันแบบ production
- **caching อาจไม่คุ้ม TTL** — ถ้าสั่งห่างกันเกิน 5 นาที (cache TTL default) cache หมดอายุก่อนใช้ซ้ำ (cache write คิด 1.25x สำหรับ TTL 5 นาที, 2.0x สำหรับ 1 ชม.) — personal use ที่สั่งห่างๆ caching ช่วยเฉพาะตอนสั่งรัวติดกัน ไม่งั้นจ่าย cache-write เปล่า
- **เริ่มแบบไม่ต้อง optimize ก่อนได้** — เริ่มด้วย Sonnet 4.6 ตัวเดียว (~$40–60/เดือน) แล้วค่อยเพิ่ม routing ทีหลังถ้ารู้สึกแพง
- **image input** บวกเพิ่มราว $2–5/เดือน (vision คิดเป็น token ปกติ ไม่ใช่ค่าแยก)

### 4.5 Recommended Agentic Loop / Framework

**แนะนำหลัก: LangGraph**

เหตุผลที่ตรงกับโปรเจกต์นี้:

- เป็น production standard สำหรับ stateful, auditable workflow ในงานที่ต้องการ auditability และ human approval — ตรงกับ admin oversight + accept/reject + rollback ของเราเป๊ะ
- โครงสร้างเป็น **directed graph + typed state** ควบคุมลำดับ agent ได้ชัด (validate → decide → apply → self-heal) inspect/test/debug ง่าย
- มี **built-in checkpointing** (resume/retry/time-travel debugging) — สำคัญมากสำหรับ self-validation loop และการ rollback
- รองรับ **human-in-the-loop (HITL) ผ่าน interrupt** — แทรกจุดให้ admin อนุมัติก่อน apply ได้ตรงๆ
- **provider-agnostic** — สลับ model ได้ตาม 4.2 (routing) ไม่ผูกกับเจ้าเดียว เหมาะกับข้อจำกัด cost
- รองรับ MCP natively สำหรับเชื่อม tool ภายนอก

**ทางเลือกอื่น:**

| Framework | จุดเด่น | เหมาะเมื่อ |
|-----------|--------|-----------|
| **CrewAI** | path เร็วที่สุดสู่ multi-agent prototype (role-based) | ต้องการ prototype เร็ว แต่หลายทีม "outgrow" ตอน scale |
| **OpenAI Agents SDK** | low-overhead, handoff pattern, integration แน่นกับ GPT | deploy แบบ OpenAI-native แต่ผูกกับ provider เดียว |
| **Claude Agent SDK** | architecture เดียวกับ Claude Code, hooks/MCP/skills/subagents | สร้าง Claude-native agent ที่อยากได้ Memory + native tool use |

> หมายเหตุ: framework ที่ห่อรอบ model มีผลต่อ performance ได้มาก (มีรายงานว่าต่างกันได้ถึง ~30 percentage points บน model เดียวกัน) — orchestration ไม่ใช่เรื่องรอง

### 4.6 Model Router

หัวใจของ cost optimization สำหรับงานนี้ — แทนที่จะส่งทุกคำสั่งไป model ตัวเดียว (แพง) ให้มี **router** จัดประเภทคำสั่งก่อน แล้วส่งไป model ที่ถูกที่สุดที่ยังทำงานได้ดีพอ เพราะ Haiku ถูกกว่า Sonnet 5 เท่า และถูกกว่า Opus 25 เท่า การ route งานง่ายไป tier ล่างลด cost ได้มากที่สุดโดยไม่เสียคุณภาพ

#### Routing Tiers

| Tier | Model (Claude) | ใช้กับคำสั่งแบบ | สัดส่วนคาดการณ์ |
|------|---------------|----------------|----------------|
| **Light** | Haiku 4.5 ($1/$5) | rename, แก้สี/ข้อความ, toggle flag, query feature map, classify intent | ~70% |
| **Medium** | Sonnet 4.6 ($3/$15) | แก้ component, เพิ่ม field, validate + retry, diagnose บัคทั่วไป | ~25% |
| **Heavy** | Opus 4.8 ($5/$25) | refactor หลายไฟล์, ออกแบบ workflow ใหม่, แก้บัคที่กระทบ dependency ซับซ้อน | ~5% |

> สัดส่วน 70/25/5 (Haiku/Sonnet/Opus) แทนที่จะใช้ Sonnet ล้วน ลด cost รวมได้มากกว่าครึ่งบน workload ทั่วไป

#### How the Router Decides

router ตัดสินใจจาก **ความซับซ้อนของคำสั่ง** ก่อนเข้า main agentic loop — เลือกได้หลายวิธี (เรียงตามต้นทุน):

1. **Rule-based (ถูกสุด, เริ่มที่นี่)** — แมตช์ keyword/pattern: คำสั่งที่ขึ้นต้นด้วย "เปลี่ยนสี/rename/ซ่อน/แสดง" → Light; มีคำว่า "refactor/redesign/ออกแบบใหม่" → Heavy. deterministic, ไม่กิน token, แต่ครอบคลุมได้จำกัด
2. **Classifier ด้วย Haiku** — ส่งคำสั่งให้ Haiku จัดประเภทก่อน (เสีย token น้อยมากเพราะ Haiku ถูก + output สั้นแค่ tier label) แล้วค่อย route ไป model จริง — ยืดหยุ่นกว่า rule-based
3. **Hybrid (แนะนำ)** — rule-based จับเคสชัดเจนก่อน → เคสที่ไม่ชัด fallback ไป Haiku classifier — ได้ทั้งความถูกและความยืดหยุ่น

#### Escalation & Fallback

- **Escalation** — ถ้า Light model ทำแล้ว validate ไม่ผ่าน (เจอบัค/แก้ไม่ครบ) → escalate ขึ้น tier ถัดไปอัตโนมัติ แทนที่จะ reject ทันที
- **Fallback** — ถ้า model ที่ route ไป error/timeout → ลอง tier ที่เสถียรกว่า
- **Image override** — คำสั่งที่มี image attachment ต้อง route ไป vision-capable model (Sonnet/Opus) ข้าม Haiku เสมอ (เพราะ image ต้องการ vision)
- **Cap การ escalate** — จำกัดจำนวนครั้งที่ escalate ได้ต่อ 1 คำสั่ง กัน loop วนขึ้น Opus รัวๆ จน cost พุ่ง

#### Integration กับ LangGraph

router เป็น **node แรก** ใน graph ก่อน main loop:

```
[Input] → [Router node] → เลือก model → [Validate → Decide → Apply → Self-heal]
                ↑                                      │
                └──────── escalate ถ้า validate fail ──┘
```

เนื่องจาก LangGraph เป็น provider-agnostic การสลับ model ต่อ node ทำได้ตรงๆ — router แค่ set ว่า node ถัดไปใช้ model ตัวไหนตาม tier ที่ตัดสิน

> **Personal use note:** สำหรับใช้ส่วนตัว router ช่วยมากเพราะงานส่วนใหญ่เป็นคำสั่งง่ายๆ ที่ Haiku ทำได้สบาย — ดูตาราง Personal Use Scenario ใน 4.4 (คอลัมน์ Routing + cache ถูกกว่า Sonnet ตรงๆ ราวครึ่งหนึ่ง)

---

## 5. Open Questions / TODO

- [ ] เลือก mechanism สำหรับ apply การเปลี่ยนแปลง: generate code จริง vs. config-driven (schema/feature flags) — config-driven ปลอดภัยและ rollback ง่ายกว่า
- [ ] ออกแบบ sandbox/staging สำหรับ validate ก่อน apply เข้า production
- [ ] กำหนด permission model: ใครสั่งอะไรได้บ้าง (RBAC)
- [ ] เลือก LLM ภายใต้ข้อจำกัด API cost / GPU usage
- [ ] เลือก vision model สำหรับ image input — ใช้ทุก request หรือเรียกเฉพาะเมื่อมี attachment (เรียกเฉพาะตอนมีภาพช่วยลด cost)
- [ ] เลือก model router strategy: rule-based / Haiku classifier / hybrid (เริ่ม rule-based ก่อนแล้วค่อยเพิ่ม classifier)
- [ ] กำหนด escalation cap ต่อคำสั่ง กัน loop วนขึ้น Opus จน cost พุ่ง
- [ ] ออกแบบ schema ของ Feature Map + dependency graph
- [ ] กำหนด rollback granularity (ย้อนทีละ change หรือทีละ snapshot)
- [ ] ตั้ง observability/tracing (เช่น LangSmith / Langfuse) เพื่อ track token + cost per command
- [ ] ทำ pilot เพื่อ log token จริง แล้ว extrapolate งบ (buffer 1.7–2x)

---

## 6. Notes

แนวคิดนี้ใกล้เคียงกับ self-modifying / agentic system — จุดเสี่ยงหลักคือการให้ LLM แก้ระบบ production โดยตรง ดังนั้น **staging + diff approval + rollback** คือหัวใจที่ทำให้ปลอดภัยพอจะใช้งานจริง
