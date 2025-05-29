# 뇌파 힐링 프로그램 분석

## 1. 개발 환경
- Visual Studio (C# 개발을 위한 IDE)
- .NET Framework (C# 응용 프로그램 실행을 위한 프레임워크)
- NeuroSky ThinkGear SDK (뇌파 측정 장비 연동을 위한 SDK)

## 2. 프로젝트 구조
### 주요 파일들:
1. `Program.cs`: 메인 프로그램 코드
2. `Light.cs`: 보조 기능 코드
3. `HelloEEG.csproj`: 프로젝트 설정 파일
4. `packages.config`: 외부 라이브러리 설정 파일

## 3. 코드 분석

### 사용된 라이브러리
```csharp
using System;  // 기본 C# 기능
using System.Media;  // 소리 재생 관련
using System.Collections.Generic;  // 리스트, 딕셔너리 등 자료구조
using System.Threading;  // 쓰레드 관련 기능
using System.IO;  // 파일 입출력
using System.IO.Ports;  // 시리얼 포트 통신
using System.Timers;  // 타이머 기능
using NeuroSky.ThinkGear;  // 뇌파 측정 장비 SDK
using NeuroSky.ThinkGear.Algorithms;  // 뇌파 분석 알고리즘
```

### 주요 변수들
```csharp
static Connector connector;  // NeuroSky 장비 연결 객체
static ArrayList OneBuffer = new ArrayList();  // 1초 동안의 뇌파 데이터 저장
static ArrayList FiveBuffer = new ArrayList();  // 5초 동안의 뇌파 데이터 저장
static ArrayList Result = new ArrayList();  // 분석 결과 저장

static double rp = 0;  // SMR파의 상대 파워
static double total = 0;  // 전체 주파수 파워
static double hbeta = 0;  // 고베타파 파워
```

## 4. 프로그램 흐름

### 1) 프로그램 시작 (Main 메소드)
```csharp
public static void Main(string[] args)
{
    // NeuroSky 장비 연결 설정
    connector = new Connector();
    
    // 이벤트 핸들러 등록
    connector.DeviceConnected += new EventHandler(OnDeviceConnected);
    connector.DeviceConnectFail += new EventHandler(OnDeviceFail);
    connector.DeviceValidating += new EventHandler(OnDeviceValidating);
    
    // COM5 포트로 장비 연결 시도
    connector.ConnectScan("COM5");
    
    // EKG 트레이닝 시작
    connector.EKGstartLongTraining("Neraj");
    
    // 프로그램 실행 유지
    Thread.Sleep(4500000);
}
```

### 2) 장비 연결 시 (OnDeviceConnected 메소드)
```csharp
static void OnDeviceConnected(object sender, EventArgs e)
{
    // 연결된 장비 정보 출력
    Console.WriteLine("Device found on: " + de.Device.PortName);
    
    // 데이터 수신 이벤트 핸들러 등록
    de.Device.DataReceived += new EventHandler(OnDataReceived);
}
```

### 3) 데이터 수신 시 (OnDataReceived 메소드)
```csharp
static void OnDataReceived(object sender, EventArgs e)
{
    // 수신된 데이터 파싱
    TGParser tgParser = new TGParser();
    tgParser.Read(de.DataRowArray);
    
    // Raw 데이터 처리
    for (int i = 0; i < tgParser.ParsedData.Length; i++)
    {
        if (tgParser.ParsedData[i].ContainsKey("Raw"))
        {
            // 1초, 5초 버퍼에 데이터 저장
            OneBuffer.Add(tgParser.ParsedData[i]["Raw"]);
            FiveBuffer.Insert(0, tgParser.ParsedData[i]["Raw"]);
            
            // 버퍼가 가득 차면 FFT 분석 수행
            if (OneBuffer.Count >= 512)
            {
                // FFT 분석
                double[] _FiveBuffer = FiveBuffer.ToArray(typeof(double)) as double[];
                double[] pf = FFT(_FiveBuffer, 1000);
                
                // 주파수 대역별 파워 계산
                for (int j = 0; j < 256; j++)
                {
                    // 전체 주파수 파워 계산 (1-58Hz)
                    if ((double)(df * j) >= 1.0 && (double)(df * j) < 58.0)
                        total += pf[j];
                    
                    // 고베타파 파워 계산 (20-30Hz)
                    if ((double)(df * j) >= 20.0 && (double)(df * j) <= 30.0)
                        hbeta += pf[j];
                }
                
                // 상대 파워 계산
                rp = hbeta / total;
            }
        }
    }
}
```

## 5. 프로그램의 주요 기능

### 1) 뇌파 측정
- NeuroSky MindWave Mobile 2 장비로부터 뇌파 데이터 수신
- Raw 데이터를 1초, 5초 단위로 버퍼에 저장

### 2) 뇌파 분석
- FFT(고속 푸리에 변환)를 사용하여 주파수 분석
- 전체 주파수(1-58Hz)와 고베타파(20-30Hz) 파워 계산
- 상대 파워 계산으로 스트레스 레벨 측정

### 3) 결과 처리
- 분석 결과를 콘솔에 출력
- 필요시 파일로 저장 가능

## 6. 핵심 알고리즘
1. 뇌파 중 고베타파(20-30Hz)의 상대적 강도를 측정
2. 고베타파가 강할수록 스트레스/긴장 상태로 판단
3. 이 결과를 바탕으로 적절한 영상/음악을 재생

## 7. 면접 준비를 위한 주요 포인트

### 1) 신호 처리 관련 예상 질문
- Q: FFT를 선택한 이유와 다른 방법과의 비교?
  - A: FFT는 시간 도메인의 신호를 주파수 도메인으로 변환하는 가장 효율적인 알고리즘
  - 실시간 처리가 필요한 상황에서 O(n log n)의 시간 복잡도로 빠른 분석 가능
  - 다른 방법(예: DFT)은 O(n²) 시간 복잡도로 실시간 처리에 부적합

- Q: 샘플링 레이트를 512Hz로 선택한 이유?
  - A: 나이퀴스트 이론에 따라 분석하고자 하는 최대 주파수(30Hz)의 2배 이상 필요
  - 뇌파 분석에서 일반적으로 사용되는 표준 샘플링 레이트
  - 노이즈 제거와 정확한 주파수 분석을 위해 충분한 샘플 수 확보

### 2) 알고리즘 설계 관련 예상 질문
- Q: 스트레스 판별 기준을 어떻게 정했는지?
  ```csharp
  // 스트레스 지수 계산
  rp = hbeta / total;  // 고베타파의 상대적 비율
  ```
  - A: 13명의 테스트 데이터를 수집하여 기준값 설정
  - 스트레스 상황과 평온한 상황에서의 데이터를 비교 분석
  - 의료 논문 및 연구 자료를 참고하여 20-30Hz 대역 선정

- Q: 실시간 처리를 위한 버퍼 설계는 어떻게 했는지?
  ```csharp
  static ArrayList OneBuffer = new ArrayList();  // 1초 데이터
  static ArrayList FiveBuffer = new ArrayList();  // 5초 데이터
  ```
  - A: 1초 버퍼는 즉각적인 반응을 위해 사용
  - 5초 버퍼는 더 정확한 분석을 위한 컨텍스트 제공
  - 원형 버퍼 구조로 메모리 사용 최적화

### 3) 개발 시 어려웠던 점과 해결 방법
1. 하드웨어 통신 문제
   - 문제: 시리얼 통신 불안정성과 데이터 손실
   - 해결: 버퍼 시스템 도입 및 에러 핸들링 강화
   ```csharp
   if (poorSig != 200)  // 신호 품질 체크
   {
       // 데이터 처리
   }
   ```

2. 실시간 처리 성능
   - 문제: FFT 연산 부하로 인한 처리 지연
   - 해결: C++ DLL을 통한 네이티브 코드 최적화
   ```csharp
   [DllImport("coclib")]
   extern public static IntPtr fft(double[] data, int length, int mag_alpha);
   ```

3. 노이즈 처리
   - 문제: 움직임, 전기 신호 등으로 인한 노이즈
   - 해결: 대역 통과 필터 적용 및 신호 품질 체크
   ```csharp
   public static double[] BandPass(double[] data, int sampling_rate, double low_cut, double high_cut)
   ```

### 4) 확장성 및 개선 포인트
1. 데이터 저장 및 분석
   ```csharp
   // 현재: 텍스트 파일로 저장
   StreamWriter wr = new StreamWriter(@"C:\Users\정민지\Desktop\DB\" + count + ".txt");
   ```
   - 개선점: 데이터베이스 도입으로 데이터 관리 체계화
   - 머신러닝 모델 적용으로 정확도 향상 가능

2. 사용자 인터페이스
   - 현재: 콘솔 기반 출력
   - 개선점: GUI 추가로 사용자 경험 개선
   - 실시간 그래프 표시 기능 추가

3. 멀티스레딩 최적화
   ```csharp
   Thread.Sleep(4500000);  // 현재: 단순 대기
   ```
   - 개선점: 이벤트 기반 처리로 변경
   - 비동기 처리로 성능 향상

### 5) 기술 스택 관련 예상 질문
- Q: C#을 선택한 이유?
  - A: 하드웨어 통신 라이브러리 지원
  - 윈도우 환경에서의 안정성
  - 빠른 프로토타이핑 가능

- Q: 외부 라이브러리 의존성 관리?
  - A: NuGet 패키지 매니저 사용
  - 버전 관리 및 호환성 확인
  - 필요한 최소한의 라이브러리만 사용

### 6) 프로젝트 아키텍처 관련 예상 질문
- Q: 이벤트 기반 설계를 선택한 이유?
  ```csharp
  connector.DeviceConnected += new EventHandler(OnDeviceConnected);
  ```
  - A: 하드웨어 이벤트에 대한 효율적인 처리
  - 느슨한 결합으로 유지보수성 향상
  - 실시간 처리에 적합한 구조

- Q: 확장성을 고려한 설계 포인트?
  - A: 모듈화된 구조로 새로운 기능 추가 용이
  - 인터페이스 기반 설계로 컴포넌트 교체 가능
  - 설정 외부화로 유연한 환경 대응 