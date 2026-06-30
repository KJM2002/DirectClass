# DirectX 및 렌더링 파이프라인 정리

---

# 1. 그래픽 API의 발전

## 1.1 Fixed Function Pipeline 시대

초기의 그래픽 API는 GPU 내부에 미리 구현된 기능만 사용할 수 있었다.

대표적인 API:

- OpenGL 1.x
- DirectX 7 이하

이 시기의 렌더링 구조를 **고정 함수 파이프라인(Fixed Function Pipeline)** 이라고 부른다.

개발자가 조작 가능한 것은 제한적이었다.

```cpp
SetRenderState();
SetTransform();
SetLight();
SetTextureStageState();
```

즉,

- 정점 변환
- 조명 계산
- 텍스처 혼합

등의 동작 방식이 GPU 내부에 고정되어 있었다.

---

## 1.2 TnL (Transform & Lighting)

DirectX 7 시절의 핵심 기능이다.

### Transform

정점 좌표를 화면 좌표계로 변환한다.

```text
Model Space
↓
World Space
↓
View Space
↓
Projection Space
↓
Screen Space
```

### Lighting

빛 계산을 수행한다.

대표적인 조명:

- Ambient
- Diffuse
- Specular

---

## 1.3 Fixed Function Pipeline의 한계

다음과 같은 표현은 구현이 불가능했다.

- 셀 셰이딩
- 물 표현
- 피부 표현
- PBR 렌더링
- 특수 포스트 프로세스

새로운 기능은 GPU 제조사가 직접 하드웨어에 추가해야 했다.

---

# 2. Shader 시대의 시작

## DirectX 8

출시:

```text
2000년
```

최초의 프로그래밍 가능한 GPU 파이프라인이 등장했다.

### Shader Model 1.0

사용 언어:

```text
Assembly
```

---

## DirectX 9

출시:

```text
2002년
```

지원:

- Shader Model 2.0
- Shader Model 3.0

사용 언어:

- ASM
- HLSL

이 시점부터 현대적인 그래픽 프로그래밍이 시작되었다.

---

# 3. HLSL

## HLSL이란?

HLSL

```text
High Level Shader Language
```

Microsoft가 만든 GPU 전용 프로그래밍 언어이다.

---

## CPU와 GPU의 프로그래밍 언어

| CPU | GPU |
|-----|-----|
| C | HLSL |
| C++ | HLSL |

---

## CPU 프로그램 실행 과정

```text
C/C++
↓
컴파일
↓
EXE
↓
CPU 실행
```

---

## GPU 프로그램 실행 과정

```text
HLSL
↓
컴파일
↓
Shader Binary
↓
GPU 실행
```

---

## CPU와 GPU 구조 차이

### CPU

특징:

- 복잡한 연산 처리
- 분기 처리 강함
- 병렬 처리 약함

구조:

```text
CISC
```

---

### GPU

특징:

- 단순 연산 반복
- 대규모 병렬 처리
- 벡터 연산 특화

구조:

```text
RISC
```

---

# 4. Programmable Pipeline

DirectX 8 이후부터 GPU 내부 동작을 개발자가 직접 작성할 수 있게 되었다.

이를

```text
Programmable Pipeline
```

또는

```text
Shader Pipeline
```

이라고 부른다.

---

## Fixed Function Pipeline

```text
GPU 제조사가 기능 제공
↓
개발자는 사용만 가능
```

---

## Shader Pipeline

```text
개발자가 GPU 동작 직접 작성
```

대표적인 예시:

- Toon Shader
- Water Shader
- PBR
- Raymarching
- Volumetric Cloud
- Skin Shader

---

# 5. DirectX 버전별 특징

| 항목 | DX9 | DX10 | DX11 | DX12 |
|------|-----|------|------|------|
| 출시년도 | 2002 | 2006 | 2009 | 2015 |
| Shader Model | SM2/3 | SM4 | SM5 | SM6 |
| HLSL | O | O | O | O |
| Geometry Shader | X | O | O | O |
| Compute Shader | X | X | O | O |
| Tessellation | X | X | O | O |
| 멀티스레드 렌더링 | X | 일부 | O | O |

---

# 6. DirectX 11 렌더링 파이프라인

```text
Input Assembler
↓
Vertex Shader
↓
Hull Shader
↓
Tessellator
↓
Domain Shader
↓
Geometry Shader
↓
Rasterizer
↓
Pixel Shader
↓
Output Merger
```

---

## Input Assembler

역할:

- Vertex Buffer 준비
- Index Buffer 준비

---

## Vertex Shader

역할:

- 월드 변환
- 뷰 변환
- 프로젝션 변환
- 스키닝

정점 단위로 실행된다.

---

## Hull Shader

테셀레이션 강도를 결정한다.

---

## Tessellator

삼각형을 더 작은 삼각형으로 분할한다.

---

## Domain Shader

분할된 정점 위치를 계산한다.

---

## Geometry Shader

삼각형 단위로 동작한다.

예시:

- Billboard
- Shadow Volume
- 파티클 생성

---

## Rasterizer

삼각형을 픽셀 단위 데이터로 변환한다.

---

## Pixel Shader

픽셀의 최종 색상을 계산한다.

대표 작업:

- 조명 계산
- 그림자 계산
- PBR
- 포스트 프로세스

---

## Output Merger

최종 렌더 타겟에 출력한다.

---

# 7. Compute Shader

DX11에서 등장한 GPU 범용 연산 기능이다.

렌더링 파이프라인과 독립적으로 동작한다.

```text
렌더링 없이 GPU 계산만 수행 가능
```

---

## 활용 분야

- 입자 시뮬레이션
- 유체 시뮬레이션
- 이미지 처리
- AI 계산
- FFT 연산
- 레이트레이싱 전처리

---

## 실제 사용 사례

- Niagara GPU Particle
- Lumen 일부 기능
- Nanite 일부 처리
- TSR
- DLSS
- FSR

---

# 8. DX11과 DX12의 차이

## DX11

특징:

- 드라이버가 대부분의 작업 수행
- 사용하기 쉬움
- CPU 오버헤드 큼

---

## DX12

특징:

- 개발자가 GPU 자원 직접 관리
- CPU 부하 감소
- 멀티코어 활용 극대화
- 구현 난이도 상승

---

# 9. DX12의 핵심 변화

## Resource 통합

DX11에서는 각각의 리소스가 별도로 존재했다.

```text
RenderTarget
Texture
Buffer
UAV
```

---

DX12에서는

```text
ID3D12Resource
```

하나로 통합되었다.

필요에 따라 View를 생성한다.

대표 View:

- SRV
- RTV
- DSV
- UAV

---

# 10. Resource View 종류

## DSV

```text
DepthStencilView
```

깊이 버퍼.

---

## RTV

```text
RenderTargetView
```

렌더링 결과 출력 대상.

---

## SRV

```text
ShaderResourceView
```

쉐이더가 읽는 리소스.

---

## UAV

```text
UnorderedAccessView
```

쉐이더가 읽고 쓸 수 있는 리소스.

주로 Compute Shader에서 사용한다.

---

# 11. DirectX 발전 과정 요약

## DirectX 7 이전

```text
Fixed Function Pipeline
```

GPU가 제공하는 기능만 사용 가능.

---

## DirectX 8 ~ 9

```text
Shader 도입
```

GPU 프로그래밍 시작.

---

## DirectX 10 ~ 12

```text
완전한 Shader Pipeline
```

렌더링 과정 대부분을 개발자가 직접 제어.

---

# 12. Unreal Engine 5 관점에서 보기

| UE5 기능 | 실제 GPU 단계 |
|----------|--------------|
| Material | Pixel Shader |
| World Position Offset | Vertex Shader |
| Niagara GPU Particle | Compute Shader |
| Nanite | Compute Shader |
| Virtual Shadow Map | Compute Shader |
| Lumen | Compute + Pixel Shader |
| Post Process Material | Pixel Shader |
| Tessellation (UE4) | Hull + Domain Shader |

---

## 중요한 사실

언리얼 엔진의 머티리얼 그래프는 내부적으로 HLSL 코드로 변환된다.

즉,

```text
Material Graph
↓
HLSL 생성
↓
Shader Compile
↓
GPU 실행
```

따라서 언리얼 머티리얼을 작성하는 행위는 사실상 GPU 프로그래밍을 시각적으로 수행하는 것과 동일하다.

---

# 최종 한 줄 요약

```text
CPU는 렌더링 명령을 준비하고,
GPU는 HLSL 셰이더를 실행하여 화면을 그린다.
```