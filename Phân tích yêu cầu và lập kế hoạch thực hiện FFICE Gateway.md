Dưới đây là phân tích đầy đủ về thứ tự và quan hệ phụ thuộc giữa các công việc.

---

## 1. Sơ đồ phụ thuộc — Cái gì chặn cái gì

```mermaid
flowchart TD
    subgraph P0["PHASE 0 — Prerequisites (trước khi code)"]
        PR1["PR-001: FIXM 4.3 XSD schemas\n(fixm.aero)"]
        PR2["PR-002: Mapping docs từ CAAS\n(cần formal request)"]
        PR3["PR-003: Xác nhận CAAS dùng FIXM 4.2 hay 4.3"]
        PR5["PR-005: Solace PubSub+ setup\n(internal)"]
        PR8["PR-008: ICAO Doc 8643\nAircraft type DB"]
        TRAIN["Team training:\nFIXM XML + GUFI concepts"]
    end

    subgraph P1["PHASE 1 — Core Foundation (tuần tự, không thể skip)"]
        GUFI["GUFI State Machine\n(CV-F04, CV-F05)\n⚠️ CRITICAL GAP"]
        PARSER["FPL2012 Parser\n(CV-F01)"]
        FPL2FIXM["FPL → FIXM XML\n(CV-F02)"]
        FIXM2FPL["FIXM → FPL text\n(CV-F03)"]
        MAPENG["Mapping Engine\n(CV-F06, CV-F07)\nRoute/4D trajectory"]
        DBSCHEMA["DB Schema\n(PostgreSQL + Flyway)"]
    end

    subgraph P2["PHASE 2 — Parallel Development"]
        direction LR
        subgraph A["Track A: Adapter"]
            AD1["AD-F01: Nhận FPL từ AMHS"]
            AD2["AD-F02/F03: Publish/Subscribe Solace"]
            AD3["AD-F04/F05/F06/F07:\nResilience + DLQ + Dedup"]
        end
        subgraph B["Track B: Filing Service"]
            FS1["FS-F01/F02/F04/F05/F06:\nCore: receive, validate schema,\npersist, respond FIXM XML"]
            VRGRP1["Validation Group 1+2\n(VR-001→025) — REJECT rules\n⚠️ làm trước"]
            VRGRP2["Validation Group 3+4\n(VR-026→050) — WARNING rules\nlàm sau"]
        end
        subgraph C["Track C: Flight Data Request"]
            FDR1["FDR-F01/F02: Query by GUFI\n+ flight number"]
            FDR2["FDR-F03/F04/F07/F08:\nQuery by route, time range,\npagination, GUFI status"]
        end
    end

    subgraph P3["PHASE 3 — Security & Integration"]
        SEC["TLS 1.3 + mTLS X.509\n(NF-S01, NF-S02)"]
        JWT["JWT Authentication\n(NF-S03 → NF-S06)"]
        XXE["XXE Prevention\n(NF-S08)"]
        PR4["PR-004: VATM FDPS staging\n(NOT production)"]
        PR6["PR-006: Sample eFPL từ VATM-CAAS 2025"]
        PR7["PR-007: X.509 certs test env"]
    end

    subgraph P4["PHASE 4 — End-to-End Testing"]
        E2E["E2E Test: 100 FPL samples\nAC-001 → AC-004"]
        PERF["Performance test:\n1,000 msg/hr, p95 < 2s\nAC-009, AC-013"]
        CAAS["CAAS Sandbox Integration\nAC-011: ≥10 eFPL exchange"]
        COMPAT["FIXM 4.2 compat layer\n(nếu CAAS dùng 4.2)\nNF-I04"]
    end

    subgraph P5["PHASE 5 — Packaging (Q4)"]
        DOCKER["Docker Compose\nNF-O01, AC-012"]
        SWAGGER["Swagger/OpenAPI 3.0\nNF-O06"]
        METRICS["Prometheus metrics\n+ Grafana (nice-to-have)"]
    end

    %% Dependencies Phase 0 → Phase 1
    PR1 --> GUFI
    PR1 --> FPL2FIXM
    PR2 --> MAPENG
    PR8 --> VRGRP1
    TRAIN --> GUFI
    TRAIN --> PARSER

    %% Internal Phase 1 dependencies
    PARSER --> FPL2FIXM
    PARSER --> FIXM2FPL
    FPL2FIXM --> MAPENG
    GUFI --> DBSCHEMA
    MAPENG --> DBSCHEMA

    %% Phase 1 → Phase 2
    GUFI --> FS1
    GUFI --> AD1
    FPL2FIXM --> AD1
    FIXM2FPL --> AD1
    MAPENG --> FPL2FIXM
    DBSCHEMA --> FS1
    DBSCHEMA --> FDR1

    %% Internal Phase 2
    AD1 --> AD2
    AD2 --> AD3
    FS1 --> VRGRP1
    VRGRP1 --> VRGRP2
    FS1 --> FDR1
    FDR1 --> FDR2

    %% Phase 2 → Phase 3
    FS1 --> JWT
    AD2 --> SEC
    PR7 --> SEC
    PR3 --> COMPAT

    %% Phase 3 → Phase 4
    SEC --> CAAS
    JWT --> CAAS
    PR4 --> E2E
    PR6 --> E2E
    AD3 --> PERF
    VRGRP2 --> E2E
    FDR2 --> E2E
    COMPAT --> CAAS

    %% Phase 4 → Phase 5
    E2E --> DOCKER
    PERF --> DOCKER
    CAAS --> DOCKER
```

---

## 2. Gantt theo Quý

```mermaid
gantt
    title FF-ICE Gateway PoC 2026 — Kế hoạch công việc
    dateFormat  YYYY-MM-DD
    axisFormat  %b %Y

    section PHASE 0 — Prerequisites
    Lấy FIXM 4.3 XSD (PR-001)         :done, pr1, 2026-01-05, 2w
    Formal request CAAS mapping (PR-002):done, pr2, 2026-01-05, 3w
    Xác nhận FIXM version CAAS (PR-003) :done, pr3, 2026-01-12, 2w
    Setup Solace PubSub+ (PR-005)       :done, pr5, 2026-01-05, 2w
    Lấy ICAO Doc 8643 (PR-008)          :done, pr8, 2026-01-05, 1w
    Team training FIXM/GUFI             :done, train, 2026-01-05, 4w

    section PHASE 1 — Core Foundation (tuần tự)
    GUFI State Machine ⚠️              :crit, gufi, 2026-01-19, 3w
    FPL2012 Parser                      :crit, parser, 2026-01-19, 2w
    FPL → FIXM XML Converter            :crit, fpl2fixm, after parser, 3w
    FIXM → FPL text Converter           :crit, fixm2fpl, after parser, 2w
    Mapping Engine (route/4D)           :crit, mapeng, after fpl2fixm, 2w
    DB Schema + Flyway                  :dbschema, after gufi, 2w

    section PHASE 2A — Adapter (Track A)
    Nhận FPL từ AMHS (AD-F01)          :ad1, after mapeng, 2w
    Publish/Subscribe Solace (AD-F02/03):ad2, after ad1, 2w
    Resilience + DLQ + Dedup           :ad3, after ad2, 2w

    section PHASE 2B — Filing Service (Track B)
    Core Filing Service                 :fs1, after dbschema, 3w
    Validation Rules REJECT (VR-001~025):vrrej, after fs1, 3w
    Validation Rules WARNING (VR-026~050):vrwarn, after vrrej, 2w

    section PHASE 2C — Flight Data Request (Track C)
    Query by GUFI + flight number       :fdr1, after dbschema, 2w
    Query by route + time range + page  :fdr2, after fdr1, 2w

    section PHASE 3 — Security
    JWT Authentication                  :jwt, 2026-04-01, 2w
    TLS 1.3 + mTLS X.509               :sec, 2026-04-01, 3w
    XXE Prevention                      :xxe, 2026-04-01, 1w
    FIXM 4.2 compat layer (nếu cần)    :compat, 2026-04-15, 3w
    VATM FDPS staging (PR-004)          :pr4, 2026-04-01, 4w
    X.509 certs test (PR-007)           :pr7, 2026-04-01, 2w

    section PHASE 4 — Testing & Integration
    E2E test 100 FPL samples            :e2e, 2026-07-01, 4w
    Performance test 1k msg/hr          :perf, 2026-07-15, 3w
    CAAS Sandbox Integration            :caas, 2026-08-01, 6w

    section PHASE 5 — Packaging
    Docker Compose                      :docker, 2026-10-01, 3w
    Swagger/OpenAPI docs                :swagger, 2026-10-01, 2w
    Prometheus metrics                  :metrics, 2026-10-15, 2w
```

---

## 3. Phân tích quan trọng — Quyết định về thứ tự

### 🔴 Phải làm tuần tự (không thể đảo)

| Thứ tự | Lý do |
|--------|-------|
| Prerequisites → tất cả code | Không có XSD thì không thể viết converter |
| GUFI State Machine → Filing Service | Filing Service dùng state machine để chuyển FILED→ACCEPTED |
| FPL Parser → FPL→FIXM → Mapping Engine | Phụ thuộc kỹ thuật rõ ràng |
| DB Schema → Filing + Flight Data Request | Cả hai service cùng dùng DB |
| Filing Service lưu dữ liệu → Flight Data Request query | FDR không có gì để query nếu Filing chưa persist |
| Security (TLS + JWT) → CAAS integration | CAAS sẽ từ chối kết nối không có mTLS |

### 🟢 Có thể làm song song sau Phase 1

| Track A | Track B | Track C |
|---------|---------|---------|
| Adapter (AMHS/Solace) | Filing Service + Validation Rules | Flight Data Request Service |
| Độc lập về interface | Dùng chung DB schema | Dùng chung DB schema |

**Lưu ý quan trọng nhất:** Track B và Track C **chia sẻ DB schema** — cần thống nhất data model trước khi tách team làm song song, nếu không sẽ conflict migration.

### ⚠️ 3 rủi ro thứ tự hay bị bỏ sót

1. **GUFI State Machine** (RSK-005) — tài liệu chỉ rõ đây là *critical gap*. Nếu để cuối Q2 mới làm thì Filing Service đã được code sai ngay từ đầu — phải **refactor toàn bộ**.

2. **FIXM version của CAAS** (RSK-001) — nếu Q3 mới biết CAAS dùng 4.2, thì phải build compat layer gấp trong khi đang integration test — cực kỳ tốn thời gian. Xác nhận **trước khi code Q1**.

3. **Validation Rules: REJECT trước, WARNING sau** (RSK-006) — 15 REJECT rules là điều kiện để Filing Service "chạy được", 35 WARNING rules là refinement. Làm song song sẽ phân tán effort.
