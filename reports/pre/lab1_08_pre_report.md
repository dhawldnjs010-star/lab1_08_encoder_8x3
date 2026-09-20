# LAB1-08 8:3 인코더 — 실험 전 레포트

- 과목: 전자전기컴퓨터설계실험Ⅱ / LAB1 조합논리 (교육 번호 08, 기존 번호 5)
- 작성자: 엄상혁 (학번 2025440084) / 조: ______ / 작성일: 2026-09-__
- 설계 모듈: `encoder8x3` / TB: `tb_encoder8x3` / 저장소: https://github.com/dhawldnjs010-star/lab1_08_encoder_8x3
- 소스 커밋: `5312d40` (`5312d4073f0199f3e0bd1496af668f14866c16b4`)

## 1. 실험 목적과 확인할 것

입력 조합과 출력의 관계를 **진리표·파형으로 설명**한다. 이 실험의 입출력은 입력 `i[7:0]`(원-핫) / 출력 `a[2:0]`.

## 2. 예상 입력·출력 (진리표, 비트 순서: MSB가 왼쪽)

| i[7:0] | 형태 | a[2:0] |
|---|---|---|
| 10000000 | 원-핫 | 000 |
| 01000000 | 원-핫 | 001 |
| 00100000 | 원-핫 | 010 |
| 00010000 | 원-핫 | 011 |
| 00001000 | 원-핫 | 100 |
| 00000100 | 원-핫 | 101 |
| 00000010 | 원-핫 | 110 |
| 00000001 | 원-핫 | 111 |
| 00000000 | all-zero | 000 |
| 00000011 | 다중-핫 | 000 |
| 11111111 | 다중-핫 | 000 |

`case`가 정확한 원-핫 패턴만 인코딩하고 그 외는 `default: a=0`이다(우선순위 인코더가 아님). 256개 조합 전부 TB가 검사한다.

## 3. 직접 작성한 코드와 파일 역할

작성 파일: `src/encoder8x3.v`, `sim/tb_encoder8x3.sv`, `constraints/pins.xdc`

```verilog
// src/encoder8x3.v
module encoder8x3(input wire [7:0] i, output reg [2:0] a);
    always @* begin
        case (i)
            8'h80: a=0; 8'h40: a=1; 8'h20: a=2; 8'h10: a=3;
            8'h08: a=4; 8'h04: a=5; 8'h02: a=6; 8'h01: a=7;
            default: a=0;
        endcase
    end
endmodule
```

**설계 설명.** `case (i)`가 8개 원-핫 패턴(8'h80~8'h01)만 0~7로 대응시키고 `default: a=0`이다. all-zero·다중-핫 입력은 모두 0이 되므로 우선순위 인코더가 아니다.

| 파일 | 역할 |
|---|---|
| `src/*.v` | 설계(RTL). 합성 대상이며 TB를 넣지 않는다. |
| `sim/tb_encoder8x3.sv` | DUT 연결, 입력 자극, 예상값 검사(`$fatal`), `wave.vcd` 덤프(`$dumpfile`·`$dumpvars`), `$finish`. 입력을 10 ns 간격으로 바꾸며 256개 조합을 검사한다. |
| `constraints/pins.xdc` | 보드 핀 배정(PACKAGE_PIN·IOSTANDARD·get_ports). Icarus 기능 시뮬레이션의 입력이 아니다. |
| `simulation.json` | `sources`, `testbench`(`sim/tb_encoder8x3.sv`), `simulation_top`(`tb_encoder8x3`) 지정. |

## 4. 핀 제약(XDC) 설명과 상태 — **정상**

모든 포트(11개)에 PACKAGE_PIN과 IOSTANDARD(LVCMOS33)가 지정되어 있다.

| 포트 | PACKAGE_PIN | IOSTANDARD |
|---|---|---|
| a[2] | L4 | LVCMOS33 |
| a[1] | M4 | LVCMOS33 |
| a[0] | M2 | LVCMOS33 |
| i[7] | K4 | LVCMOS33 |
| i[6] | N8 | LVCMOS33 |
| i[5] | N4 | LVCMOS33 |
| i[4] | N1 | LVCMOS33 |
| i[3] | P6 | LVCMOS33 |
| i[2] | N6 | LVCMOS33 |
| i[1] | L5 | LVCMOS33 |
| i[0] | J2 | LVCMOS33 |

- `get_ports`의 이름·대괄호 표기가 RTL 포트명과 일치해야 한다. XDC는 시뮬레이션이 검증하지 않으므로 핀 배정은 Vivado 구현·보드에서 별도로 확인한다.

## 5. 사전 시뮬레이션 결과 (VS Code + Icarus Verilog)

| 항목 | 결과 |
|---|---|
| 콘솔 PASS 문구 | `LAB1_PASS encoder8x3 cases=256` |
| 검사한 입력 조합 수 | 256개 (진리표 전 조합 검사) |
| 종료 시각 | 2560 ns (`$finish`) |
| 로그·파형 | `evidence/simulation.txt`, `evidence/wave.vcd` (저장소에 커밋됨) |
| 파형 캡처 | ______ (VaporView에서 입력·출력 확대 후 캡처, `evidence/`에 저장) |

> `LAB1_PASS`는 TB가 계산한 기대값과 출력이 전부 일치했다는 자기검사 결과이다. TB의 기대식이 설계와 같은 관점으로 쓰였는지는 위 진리표와 파형으로 직접 대조해 설명한다.

- 파형에서 확인한 대표 구간(시간 · 입력 → 출력): ______
- 진리표와 어긋난 부분과 원인: ______

## 6. 수정 전후 결과 (실패 → 복구 실험)

| 단계 | 내용 |
|---|---|
| 정상 | 위 5절의 PASS 로그를 보관 |
| 수정 제안 | `8'h80: a=0` → `a=7` |
| 기대되는 실패 | i=8'h80 입력 한 행에서만 a 불일치 — 실패 로그에서 vector·expected·actual 확인 |
| 실제 실패 로그 | ______ (`build/sim/run-.../simulation.log`) |
| 복구 후 | 원래대로 복구, Save All → `02 Simulate` → PASS 재확인: ☐ |
| 변경 이유·원인·복구 결과 | ______ |

## 7. 실험 당일 보드 확인 계획

- 장비: Spartan-7 XC7S75 교육용 보드(part `xc7s75fgga484-1`). 연결 전 전원·핀 기능·I/O 전압(LVCMOS33)을 확인하고, 배선 변경은 전원을 끈 상태에서 한다.
- 확인 계획: 스위치 한 개만 켜 대응 값이 나오는지, 0개 또는 2개 이상일 때 a=0인지 확인한다(설계상 정상).
- 조합회로라 클록이 없다. TB의 10 ns 간격은 검사를 빠르게 하기 위한 값이며, 보드에서는 스위치를 손으로 바꾸므로 **입력 조합**으로만 동작을 해석한다.
- 사진에는 보드 연결과 입력·출력 위치가 함께 보이게 촬영한다.
- 조교가 무작위로 고른 2개 실험 번호는 출석부에 기록된다: ☐ 선정됨 ☐ 시연 완료

## 8. 제출 점검

- [x] 진리표(2절), 코드·XDC 설명(3·4절), 사전 시뮬레이션 로그(5절)
- [ ] 사전 파형 캡처(5절), 수정 전후 실험(6절)
- [ ] `reports/pre/`에 저장 후 commit·push
