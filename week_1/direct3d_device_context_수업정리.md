# Direct3D Device Context 수업 정리

## 1. 렌더링 파이프라인 전체 구조

Direct3D의 렌더링은 **3D 모델 데이터를 최종 화면 픽셀로 바꾸는 과정**이다. 큰 흐름은 다음과 같다.

```text
Model
  ↓
VP : Vertex Processing / 정점 처리
  ↓
GP : Geometry Processing / 기하 처리
  ↓
PP : Pixel Processing / 픽셀 처리
  ↓
Render Target
```

### 단계별 역할

| 단계 | 의미 | 주요 작업 |
|---|---|---|
| Model | 입력 모델 데이터 | 정점, 인덱스, 텍스처, 머티리얼 등의 원본 데이터 |
| VP | 정점 처리 | 정점 변환, 조명 계산, 좌표 변환 |
| GP | 기하 처리 | 3D 데이터를 2D 화면 공간으로 변환, 래스터화 전 준비 |
| PP | 픽셀 처리 | 픽셀 색상 계산, 텍스처 샘플링, 색상 혼합 |
| Render Target | 출력 대상 | 최종 렌더링 결과가 저장되는 버퍼 |

---

## 2. Direct3D 12.x 렌더링 파이프라인 구성

Direct3D 렌더링 파이프라인은 여러 단계로 나뉘며, 각 단계마다 특정 역할을 담당한다.

### 주요 파이프라인 단계

| 단계 | 한글명 | 역할 |
|---|---|---|
| Input Assembler | 입력 어셈블러 | 정점 버퍼, 인덱스 버퍼, 입력 레이아웃을 받아 기본 도형 구성 |
| Vertex Shader | 정점 셰이더 | 각 정점을 변환하고 정점 단위 연산 수행 |
| Tessellation | 테셀레이션 | 폴리곤을 더 세밀하게 분할 |
| Geometry Shader | 기하 셰이더 | 도형 단위 처리, 정점 추가/삭제 가능 |
| Rasterizer Stage | 래스터 스테이지 | 3D 도형을 화면의 픽셀 후보로 변환 |
| Pixel Shader | 픽셀 셰이더 | 각 픽셀의 최종 색상 계산 |
| Output Merger Stage | 출력 병합 스테이지 | 깊이, 스텐실, 알파, 블렌딩 처리 후 렌더타겟에 출력 |
| Render Target | 렌더타겟 | 최종 출력 이미지 저장 |

### 파이프라인 흐름 요약

```text
입력 어셈블러
  ↓
정점 셰이더
  ↓
테셀레이션
  ↓
기하 셰이더
  ↓
래스터 스테이지
  ↓
픽셀 셰이더
  ↓
출력 병합 스테이지
  ↓
렌더타겟 출력
```

---

## 3. ID3D11DeviceContext 개념

`ID3D11DeviceContext`는 Direct3D 11에서 **렌더링 파이프라인을 관리하고 GPU에 렌더링 명령을 제출하는 인터페이스**이다.

쉽게 말하면 다음과 같다.

```text
ID3D11Device        → 리소스와 렌더링 객체를 생성하는 장치
ID3D11DeviceContext → 생성된 객체를 파이프라인에 연결하고 실제 그리기 명령을 내리는 장치 목록
```

### Device와 DeviceContext의 차이

| 구분 | ID3D11Device | ID3D11DeviceContext |
|---|---|---|
| 역할 | 객체 생성 | 렌더링 명령 실행 |
| 담당 | 리소스, 셰이더, 상태 객체 생성 | 파이프라인 설정, 버퍼 바인딩, Draw 호출 |
| 비유 | 공장 / 생성기 | 작업 지시자 / 실행 관리자 |
| 예시 | CreateBuffer, CreateTexture2D, CreateVertexShader | IASetVertexBuffers, VSSetShader, PSSetShader, Draw |

---

## 4. ID3D11DeviceContext::Draw 메서드

`Draw`는 인덱스를 사용하지 않는 기본 도형을 그릴 때 사용하는 메서드이다.

```cpp
void Draw(
    UINT VertexCount,
    UINT StartVertexLocation
);
```

### 매개변수

| 매개변수 | 타입 | 의미 |
|---|---|---|
| VertexCount | UINT | 그릴 꼭짓점 수 |
| StartVertexLocation | UINT | 정점 버퍼에서 시작할 첫 번째 정점의 인덱스 |

### 반환 값

없음.

### 설명

`Draw`를 호출하면 현재 파이프라인에 바인딩된 정점 버퍼의 데이터를 사용하여 렌더링 작업이 제출된다.

정점 버퍼가 없더라도 `SV_VertexID` 시스템 값을 사용하면 런타임이 현재 처리 중인 정점 번호를 제공할 수 있다. 이 값을 이용해 정점 셰이더 내부에서 고유한 정점 데이터를 생성할 수도 있다.

### Draw 호출의 의미

```text
현재 설정된 파이프라인 상태 + 바인딩된 리소스 + 지정한 정점 수
= GPU에 그리기 명령 제출
```

---

## 5. Direct3D Device Context의 역할

Device Context는 Direct3D에서 가장 자주 사용되는 인터페이스 중 하나이다.

### 핵심 역할

1. 렌더링 파이프라인 전반 관리
2. 정점 버퍼, 인덱스 버퍼, 상수 버퍼 설정
3. 셰이더 설정
4. 렌더타겟 설정
5. 렌더링 상태 설정
6. Draw / DrawIndexed 호출
7. 리소스를 파이프라인에 바인딩

---

## 6. Immediate Context와 Deferred Context

Direct3D 11의 Device Context는 두 가지 종류가 있다.

| 종류 | 한글명 | 특징 |
|---|---|---|
| Immediate Context | 즉시 장치 목록 | 직접 렌더링 명령을 실행 |
| Deferred Context | 지연 장치 목록 | 명령을 기록해두고 나중에 실행 |

### Immediate Context

Immediate Context는 **직접 렌더링**을 수행한다.

특징은 다음과 같다.

- 싱글 스레드 중심
- 보통 하나만 존재
- GPU에 즉시 명령 제출
- 일반적인 렌더링 흐름에서 주로 사용

```text
D3D11Device
  ↓
Immediate Context
  ↓
GPU 실행
```

### Deferred Context

Deferred Context는 **간접 렌더링**을 수행한다.

특징은 다음과 같다.

- 멀티스레드 렌더링 대응
- 여러 개 생성 가능
- 여러 스레드에서 렌더링 명령을 기록 가능
- 기록된 명령 리스트를 Immediate Context가 나중에 실행

```text
Deferred Context 1 ┐
Deferred Context 2 ├─ 명령 기록 → Immediate Context → GPU 실행
Deferred Context 3 ┘
```

---

## 7. Direct3D 9/10과 Direct3D 11의 차이

Direct3D 9, 10에서는 Device가 많은 기능을 직접 담당했다. Direct3D 11부터는 **Device와 DeviceContext의 역할이 분리**되었다.

### 구형 DirectX 구조

```text
D3D9 Device / D3D10 Device
  └─ 리소스 생성, 상태 설정, 렌더링 명령까지 대부분 담당
```

### Direct3D 11 구조

```text
ID3D11Device
  └─ 리소스와 렌더링 객체 생성

ID3D11DeviceContext
  └─ 렌더링 파이프라인 관리와 명령 실행
```

### DX11에서 분리된 이유

1. 멀티스레드 렌더링 강화
2. 고전적인 Device의 역할 분리
3. 리소스 생성과 렌더링 실행 흐름을 명확히 구분
4. 파이프라인 제어를 더 유연하게 하기 위함

---

## 8. 리소스 뷰 Resource View

렌더링은 메모리 자원을 사용한다. Direct3D에서 리소스는 단순한 메모리 덩어리가 아니라 **파이프라인에서 특정 용도로 해석되는 데이터**이다.

`Resource View`는 하나의 리소스를 어떤 방식으로 사용할지 지정하는 인터페이스이다.

### 예시

```text
Back Buffer라는 텍스처 리소스
  ↓
RenderTargetView로 보면 → 렌더링 결과를 기록하는 대상
ShaderResourceView로 보면 → 셰이더에서 읽는 텍스처
```

### 리소스 뷰의 핵심 의미

```text
리소스 = 실제 메모리 데이터
뷰 = 그 리소스를 파이프라인에서 어떤 용도로 사용할지 정하는 해석 방식
```

---

## 9. Back Buffer와 Render Target

### Back Buffer

Back Buffer는 화면에 보여주기 직전의 렌더링 결과가 저장되는 버퍼이다.

예를 들어 해상도가 `800 x 600`이고 컬러 비트가 `32bit ARGB`라면, Back Buffer는 해당 해상도의 픽셀 색상 정보를 저장하는 메모리 영역이다.

### Render Target

Render Target은 렌더링 결과를 출력하는 대상이다. 보통 스왑체인의 Back Buffer를 Render Target으로 사용한다.

```text
GPU가 그림
  ↓
Back Buffer에 저장
  ↓
Present / Swap
  ↓
Front Buffer로 이동하여 화면에 표시
```

### Front Buffer와 Back Buffer

| 버퍼 | 의미 |
|---|---|
| Front Buffer | 현재 화면에 표시 중인 버퍼 |
| Back Buffer | 다음 프레임을 그리는 버퍼 |

더블 버퍼링에서는 Back Buffer에 먼저 그리고, 그 결과를 Front Buffer와 교체하여 화면 깜빡임을 줄인다.

---

## 10. Direct3D 리소스 개념

Direct3D 리소스는 렌더링에 필요한 모든 데이터를 의미한다.

### 리소스 예시

- 정점 버퍼 Vertex Buffer, VB
- 인덱스 버퍼 Index Buffer, IB
- 상수 버퍼 Constant Buffer, CB
- 텍스처 Texture
- 셰이더 Shader
- 렌더타겟 Render Target
- 깊이/스텐실 버퍼 Depth-Stencil Buffer
- 구조화 버퍼 Structured Buffer
- 바이트 주소 버퍼 Byte Address Buffer
- 읽기/쓰기 버퍼 RWBuffer
- UAV Unordered Access View

### 리소스의 특징

1. 렌더링 데이터 저장용 메모리 버퍼이다.
2. 기하도형, 텍스처, 셰이더 데이터 등을 담는다.
3. 렌더링 파이프라인에서 사용된다.
4. Direct3D 11에서는 리소스가 통합 관리되며, 사용할 때 용도를 지정한다.
5. 사용이 끝난 리소스는 `Release`로 해제한다.

---

## 11. Direct3D 10/11 리소스의 특징

Direct3D 10/11에서는 리소스 관리가 더 유연해졌다.

### 주요 특징

- 메모리 형식과 사용 용도를 결정 가능
- 관리의 유연성 증가
- 다중 스레드 렌더링에서 공유 가능
- 형식 자원 Typed Resource와 무형식 자원 Typeless Resource 지원
- 전용 인터페이스를 통해 접근
- 사용 후 `Release` 필요

### 리소스 생성과 사용 흐름

```text
1. ID3D11Device로 리소스 생성
2. ID3D11DeviceContext로 파이프라인에 바인딩
3. Draw / Dispatch 등으로 사용
4. 사용 후 Release
```

---

## 12. Typed Resource와 Typeless Resource

Direct3D 리소스는 크게 두 가지로 나눌 수 있다.

| 종류 | 의미 | 특징 |
|---|---|---|
| Typed Resource | 형식 자원 | 데이터 형식이 명확히 정해진 리소스 |
| Typeless Resource | 무형식 자원 | 생성 시 형식을 고정하지 않고, View를 통해 해석 |

### Typed Resource

형식이 정해진 리소스이다.

예를 들어 `DXGI_FORMAT_R8G8B8A8_UNORM`처럼 픽셀 형식이 명확히 지정된다.

### Typeless Resource

형식을 고정하지 않은 리소스이다. 같은 메모리를 여러 View를 통해 다르게 해석할 수 있다.

예를 들어 하나의 텍스처를 상황에 따라 DepthStencilView 또는 ShaderResourceView로 사용할 수 있다.

---

## 13. Direct3D 자원 종류 표

| 종류 | 인터페이스 | 설명 | 타입 |
|---|---|---|---|
| 리소스 뷰 | ID3D11DepthStencilView | 깊이-스텐실 버퍼 | Typed |
| 리소스 뷰 | ID3D11RenderTargetView | 렌더타겟 | Typed |
| 리소스 뷰 | ID3D11ShaderResourceView | 상수버퍼, 텍스처, 샘플러 등 셰이더 리소스 | Typed |
| 리소스 뷰 | ID3D11UnorderedAccessView | 픽셀 셰이더, 컴퓨트 셰이더 등에서 임의 접근 | Typed |
| 로우 뷰 | ID3D11Device::CreateShaderResourceView | 셰이더 리소스 뷰 생성 | Typeless |
| 로우 뷰 | RWBuffer, RWTexture2D, RWTexture2DArray | 읽기/쓰기 버퍼 | Typeless |
| 로우 뷰 | Structured Buffer | 구조화 버퍼 | Typeless |
| 로우 뷰 | Byte Address Buffer | 바이트 주소 버퍼, DWORD 정렬 | Typeless |
| 로우 뷰 | Unordered Access View, UAV | 임의 접근 버퍼, 다중 스레드 지원 | Typeless |

---

## 14. 주요 Direct3D 대표 인터페이스

### 렌더링 장치 관련

| 인터페이스 | 설명 | 관련 정보 |
|---|---|---|
| ID3D11Device | 렌더링 장치. 가상 어댑터, 리소스, 렌더링 객체 생성 | D3D_FEATURE_LEVEL |
| ID3D11DeviceContext | 장치 목록. 렌더링 파이프라인 관리 | - |
| ID3D11RasterizerState | 래스터 장치 상태 | D3D11_RASTERIZER_DESC |

### 기하도형 / Topology 관련

| 인터페이스 | 설명 | 관련 정보 |
|---|---|---|
| ID3D11InputLayout | 정점 선언, 정점 규격 | D3D11_INPUT_ELEMENT_DESC |

### 리소스 관련

| 인터페이스 | 설명 | 관련 정보 |
|---|---|---|
| ID3D11Buffer | 정점 버퍼, 인덱스 버퍼, 상수 버퍼, 다용도 데이터 버퍼 | D3D11_BUFFER_DESC |
| ID3D11RenderTargetView | 렌더타겟 | ID3D11Texture2D |
| ID3D11DepthStencilView | 깊이-스텐실 버퍼 | D3D11_DEPTH_STENCIL_DESC |
| ID3D11Texture2D | 텍스처 | D3D11_TEXTURE2D_DESC |

### DXGI 관련

| 인터페이스 | 설명 | 관련 정보 |
|---|---|---|
| IDXGIFactory | DXGI 인터페이스 생성 | - |
| IDXGISwapChain | 스왑체인 | DXGI_SWAP_CHAIN_DESC |

---

## 15. Direct3D 대표 명령 / 메서드

수업 자료에는 일부 이름이 `DXGI::`처럼 표기되어 있지만, 실제 Direct3D 11 코드에서는 대부분 `ID3D11DeviceContext` 메서드로 사용된다.

| 메서드 | 설명 |
|---|---|
| IASetVertexBuffers | 정점 버퍼 설정 |
| IASetIndexBuffer | 인덱스 버퍼 설정 |
| IASetInputLayout | 정점 입력 레이아웃 설정 |
| IASetPrimitiveTopology | 기하도형 토폴로지 설정 |
| VSSetConstantBuffers | 정점 셰이더 상수 버퍼 설정 |
| VSSetShader | 정점 셰이더 설정 |
| PSSetShader | 픽셀 셰이더 설정 |
| RSSetState | 래스터라이저 상태 설정 |
| RSSetViewports | 뷰포트 설정 |
| OMSetRenderTargets | 렌더타겟 설정 |
| OMSetBlendState | 블렌드 상태 설정 |
| Draw | 인덱스 없이 그리기 |
| DrawIndexed | 인덱스 버퍼를 사용해 그리기 |

### 핵심 흐름

```cpp
// 1. 입력 조립 단계 설정
context->IASetInputLayout(inputLayout);
context->IASetVertexBuffers(...);
context->IASetIndexBuffer(...);
context->IASetPrimitiveTopology(D3D11_PRIMITIVE_TOPOLOGY_TRIANGLELIST);

// 2. 셰이더 설정
context->VSSetShader(vertexShader, nullptr, 0);
context->PSSetShader(pixelShader, nullptr, 0);

// 3. 렌더타겟 설정
context->OMSetRenderTargets(1, &renderTargetView, depthStencilView);

// 4. 그리기
context->Draw(vertexCount, startVertexLocation);
// 또는
context->DrawIndexed(indexCount, startIndexLocation, baseVertexLocation);
```

---

## 16. DirectX 클래스 계층도

DirectX는 COM 기반 인터페이스 구조를 사용한다. 최상위에는 `IUnknown`이 있다.

```text
IUnknown
├─ ID3D11Device
├─ ID3D11DeviceChild
│  ├─ ID3D11View
│  │  └─ ID3D11RenderTargetView
│  └─ ID3D11DeviceContext
└─ IDXGIObject
   └─ IDXGIDeviceSubObject
      └─ IDXGISwapChain
```

### 각 인터페이스 역할

| 인터페이스 | 역할 |
|---|---|
| IUnknown | COM 최상위 인터페이스 |
| ID3D11Device | 렌더링 장치 관리 |
| ID3D11DeviceChild | Device에 종속되는 객체들의 기반 |
| ID3D11View | 리소스를 특정 방식으로 보는 View 기반 |
| ID3D11RenderTargetView | 렌더타겟 View |
| ID3D11DeviceContext | 렌더링 파이프라인 관리 |
| IDXGIObject | DXGI 객체 기반 |
| IDXGIDeviceSubObject | DXGI 장치에 종속된 객체 기반 |
| IDXGISwapChain | 스왑체인 관리 |

---

## 17. Direct3D 애플리케이션 진행 흐름

Direct3D 프로그램의 기본 실행 흐름은 다음과 같다.

```text
1. 윈도우 생성
2. 렌더링 장치 생성
3. 스왑체인 구성
4. 그리기
5. 종료 시 장치 해제
```

### 1단계. 윈도우 생성

렌더링 결과를 표시할 응용 프로그램 윈도우를 만든다.

### 2단계. 렌더링 장치 생성

렌더링에 필요한 핵심 장치를 생성한다.

```text
D3D 장치      : ID3D11Device
장치 목록     : ID3D11DeviceContext
```

### 3단계. 스왑체인 구성

화면 출력용 버퍼 교체 구조를 만든다.

```text
스왑체인      : IDXGISwapChain
렌더타겟      : ID3D11RenderTargetView
```

스왑체인의 Back Buffer를 가져와 `ID3D11RenderTargetView`로 만든 뒤, Device Context에 설정한다.

### 4단계. 그리기

파이프라인에 필요한 리소스와 상태를 설정한 뒤 `Draw` 또는 `DrawIndexed`를 호출한다.

### 5단계. 종료 시 장치 해제

Direct3D 인터페이스는 COM 객체이므로 사용이 끝나면 `Release`를 호출해 해제한다.

---

## 18. Direct3D 기본 초기화 흐름

```text
Win32 Window 생성
  ↓
D3D11CreateDeviceAndSwapChain 호출
  ↓
ID3D11Device 생성
  ↓
ID3D11DeviceContext 생성
  ↓
IDXGISwapChain 생성
  ↓
Back Buffer 가져오기
  ↓
ID3D11RenderTargetView 생성
  ↓
OMSetRenderTargets로 렌더타겟 설정
  ↓
렌더링 루프에서 Clear → Draw → Present
```

---

## 19. 렌더링 루프 기본 구조

Direct3D 렌더링 루프는 보통 다음 순서로 진행된다.

```cpp
while (running)
{
    // 1. 화면 초기화
    context->ClearRenderTargetView(renderTargetView, clearColor);

    // 2. 깊이/스텐실 초기화
    context->ClearDepthStencilView(depthStencilView,
                                   D3D11_CLEAR_DEPTH | D3D11_CLEAR_STENCIL,
                                   1.0f,
                                   0);

    // 3. 파이프라인 상태 설정
    context->IASetInputLayout(inputLayout);
    context->IASetVertexBuffers(...);
    context->IASetPrimitiveTopology(D3D11_PRIMITIVE_TOPOLOGY_TRIANGLELIST);
    context->VSSetShader(vertexShader, nullptr, 0);
    context->PSSetShader(pixelShader, nullptr, 0);
    context->OMSetRenderTargets(1, &renderTargetView, depthStencilView);

    // 4. 그리기
    context->Draw(vertexCount, 0);

    // 5. 화면 출력
    swapChain->Present(1, 0);
}
```

---

## 20. Direct3D에서 자주 등장하는 약어

| 약어 | 의미 | 설명 |
|---|---|---|
| D3D | Direct3D | DirectX의 3D 그래픽 API |
| DXGI | DirectX Graphics Infrastructure | 어댑터, 디스플레이, 스왑체인 관리 계층 |
| COM | Component Object Model | DirectX 인터페이스의 기반 객체 모델 |
| VB | Vertex Buffer | 정점 데이터 저장 버퍼 |
| IB | Index Buffer | 인덱스 데이터 저장 버퍼 |
| CB | Constant Buffer | 셰이더 상수 데이터 저장 버퍼 |
| RTV | Render Target View | 렌더타겟으로 사용할 리소스 뷰 |
| DSV | Depth Stencil View | 깊이/스텐실 버퍼 뷰 |
| SRV | Shader Resource View | 셰이더에서 읽는 리소스 뷰 |
| UAV | Unordered Access View | 순서 없는 읽기/쓰기 접근 뷰 |
| IA | Input Assembler | 입력 조립 단계 |
| VS | Vertex Shader | 정점 셰이더 |
| PS | Pixel Shader | 픽셀 셰이더 |
| RS | Rasterizer Stage | 래스터라이저 단계 |
| OM | Output Merger | 출력 병합 단계 |

---

## 21. 전체 개념 연결 정리

Direct3D에서 렌더링을 하려면 다음 요소들이 연결되어야 한다.

```text
윈도우
  ↓
IDXGISwapChain
  ↓
Back Buffer
  ↓
ID3D11RenderTargetView
  ↓
ID3D11DeviceContext::OMSetRenderTargets
  ↓
파이프라인 설정
  ↓
Draw / DrawIndexed
  ↓
Present
  ↓
화면 출력
```

### 핵심 요약

- `ID3D11Device`는 리소스와 객체를 만든다.
- `ID3D11DeviceContext`는 만든 객체를 파이프라인에 연결하고 렌더링 명령을 실행한다.
- `IDXGISwapChain`은 Front Buffer와 Back Buffer를 관리한다.
- `ID3D11RenderTargetView`는 Back Buffer를 렌더링 대상으로 사용하게 해준다.
- `Draw`는 현재 파이프라인 상태를 기준으로 GPU에 그리기 명령을 제출한다.
- 리소스는 실제 메모리이고, View는 그 메모리를 어떤 용도로 사용할지 정하는 해석 방식이다.
- Direct3D 11부터 Device와 DeviceContext가 분리되어 멀티스레드 렌더링과 파이프라인 관리가 더 유연해졌다.

---

## 22. 시험/복습용 핵심 문장

- Direct3D 렌더링 파이프라인은 모델 데이터를 정점 처리, 기하 처리, 픽셀 처리를 거쳐 렌더타겟에 출력하는 과정이다.
- `ID3D11Device`는 리소스 생성 담당, `ID3D11DeviceContext`는 렌더링 명령 실행 담당이다.
- `DeviceContext`는 정점 버퍼, 셰이더, 렌더타겟, 상태 객체를 파이프라인에 바인딩한다.
- `Draw(VertexCount, StartVertexLocation)`은 인덱스 없이 정점 버퍼 기반으로 도형을 그리는 명령이다.
- Immediate Context는 즉시 실행용이고, Deferred Context는 명령 기록 후 나중에 실행하는 방식이다.
- Resource View는 하나의 리소스를 렌더타겟, 셰이더 리소스, 깊이 버퍼 등 특정 용도로 해석하게 해준다.
- Back Buffer는 다음 프레임을 그리는 버퍼이고, Front Buffer는 현재 화면에 표시되는 버퍼이다.
- SwapChain은 Front Buffer와 Back Buffer를 교체하여 화면에 출력한다.
- Direct3D 리소스는 Typed와 Typeless로 나뉘며, Typeless는 View를 통해 다양한 방식으로 해석될 수 있다.
- 종료 시 Direct3D COM 객체는 `Release`로 해제해야 한다.
