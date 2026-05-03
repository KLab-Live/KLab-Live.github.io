# Bode Lab 탭별 수식 설명 자료

이 문서는 현재 구현 기준으로 `단일 루프` 탭과 `캐스케이드` 탭에서 계산에 쓰이는 수식을 정리한다. 앱의 표시 단위는 사용자가 입력하기 쉬운 단위이고, Math Core 진입 시 SI 단위로 변환된다.

관련 코드: `Bode_Lab/Core/DesignInputs.swift`, `Bode_Lab/Core/FrequencyResponse.swift`, `Bode_Lab/Core/SpeedFrequencyResponse.swift`, `Bode_Lab/Core/Metrics.swift`

## 0. 공통 표기

| 기호 | 의미 |
|---|---|
| \(s\) | 라플라스 변수. 주파수 응답 계산에서는 \(s = j\omega\) |
| \(\omega\) | 각주파수 \([\mathrm{rad/s}]\) |
| \(f\) | 주파수 \([\mathrm{Hz}]\), \(f = \omega / 2\pi\) |
| \(|X|\) | 복소 응답 \(X(j\omega)\)의 크기 |
| \(\angle X\) | 복소 응답 \(X(j\omega)\)의 위상 |
| dB | \(20\log_{10}|X|\) |
| \(T_s\) | 전류 루프 제어 주기 |
| \(T_{sw}\) | 속도 루프 샘플링 주기 |
| \(T(s)\) | 폐루프 전달함수. 각 루프에서 \(T = L/(1+L)\) |

복소수 크기와 위상은 다음처럼 계산한다.

\[
|z| = \sqrt{\operatorname{Re}(z)^2 + \operatorname{Im}(z)^2}
\]

\[
\angle z = \operatorname{atan2}(\operatorname{Im}(z), \operatorname{Re}(z))
\]

Bode 플롯은 위상을 인접 샘플 기준으로 unwrap해서 \(\pm 180^\circ\) 점프가 이어져 보이도록 보정한다.

## 1. 단일 루프 탭

단일 루프 탭은 선택한 모터와 축의 전류 루프를 계산한다. 화면의 `개루프 L(jω)` / `폐루프 T(jω)` 토글은 같은 입력으로 다음 두 응답 중 무엇을 그릴지 선택한다.

### 1.1 입력 단위 변환

앱 입력값은 표시 단위로 저장되며, 계산에는 다음 SI 단위를 사용한다.

| 입력 | 계산값 |
|---|---|
| \(L, L_d, L_q\) [mH] | \(L_\mathrm{H} = L \times 10^{-3}\) [H] |
| \(T_s\) [\(\mu s\)] | \(T_s = T_s^\mathrm{input} \times 10^{-6}\) [s] |
| \(\tau_i\) 또는 \(\tau_s\) [\(\mu s\)] | \(\tau_i = \tau_i^\mathrm{input} \times 10^{-6}\) [s] |
| \(\tau_d\) | \(\tau_d = \alpha T_s\) |

활성 인덕턴스는 모터 타입과 축 선택에 따라 달라진다.

\[
L_\mathrm{active} =
\begin{cases}
L & \text{DC}\\
L_s & \text{SPMSM}\\
L_d & \text{IPMSM d축}\\
L_q & \text{IPMSM q축}
\end{cases}
\]

### 1.2 전류 PI 제어기

전류 제어기는 병렬형 PI를 사용한다.

\[
C_i(s) = K_p + \frac{K_i}{s}
\]

주파수 응답에서는 \(s=j\omega\)이므로 다음과 같다.

\[
C_i(j\omega) = K_p - j\frac{K_i}{\omega}
\]

DC와 SPMSM은 공통 \(K_p, K_i\)를 사용한다. IPMSM은 축별 게인을 사용한다.

\[
C_{id}(s) = K_{p,d} + \frac{K_{i,d}}{s}
\]

\[
C_{iq}(s) = K_{p,q} + \frac{K_{i,q}}{s}
\]

### 1.3 전류 플랜트

전류 루프 관점에서 DC, SPMSM, IPMSM은 모두 1차 \(RL\) 플랜트로 계산한다. 차이는 사용하는 인덕턴스 값이다.

\[
G_i(s) = \frac{1}{R + sL_\mathrm{active}}
\]

주파수 응답:

\[
G_i(j\omega) = \frac{1}{R + j\omega L_\mathrm{active}}
\]

### 1.4 디지털 지연

제어 지연은 Padé 근사를 쓰지 않고 정확한 지수 형태로 점별 평가한다.

\[
\tau_d = \alpha T_s
\]

\[
D_i(j\omega) = e^{-j\omega\tau_d}
              = \cos(\omega\tau_d) - j\sin(\omega\tau_d)
\]

보통 동기 PWM 평균 지연 \(0.5T_s\)와 한 샘플 계산/갱신 지연 \(1T_s\)를 합쳐 \(\alpha \approx 1.5\)를 기본 감각으로 둔다.

### 1.5 전류 센서 LPF

센서 필터가 `이상`이면 필터 응답은 1이다.

\[
H_i(j\omega) = 1
\]

센서 필터가 `1차 LPF`이면 다음 응답을 곱한다.

\[
H_i(j\omega) = \frac{1}{1 + j\omega\tau_i}
\]

참고로 1차 LPF 차단 주파수는 다음과 같다.

\[
f_c = \frac{1}{2\pi\tau_i}
\]

### 1.6 보상 ON 모드의 전류 개루프

보상 ON은 디커플링과 역기전력이 이상적으로 보상된 상태를 가정한다.

\[
L_i(j\omega) =
C_i(j\omega)\,G_i(j\omega)\,D_i(j\omega)\,H_i(j\omega)
\]

폐루프는 다음과 같다.

\[
T_i(j\omega) = \frac{L_i(j\omega)}{1 + L_i(j\omega)}
\]

### 1.7 보상 OFF 모드의 전류 개루프

보상 OFF는 PMSM의 d-q 교차결합을 운전점 주변 소신호 보정항으로 반영한다. DC 모터는 d-q 교차결합이 없으므로 보정항이 1이다.

\[
\omega_{e0} = P\omega_{m0}
\]

선택 축을 \(A\), 반대 축을 \(B\)라고 두면:

\[
G_A(s) = \frac{1}{R + sL_A}
\]

\[
C_B(s) = K_{p,B} + \frac{K_{i,B}}{s}
\]

반대 축 폐루프가 잡혀 있다고 가정했을 때 교차결합 항은 다음처럼 계산된다.

\[
\Delta_A(s) =
\frac{\omega_{e0}^2 L_A L_B}{R + sL_B + C_B(s)}
\]

이상화 플랜트 대비 보정 계수:

\[
K_A(s) =
\frac{R + sL_A}{R + sL_A + \Delta_A(s)}
\]

따라서 보상 OFF의 유효 플랜트는:

\[
G'_A(s) = K_A(s)G_A(s)
        = \frac{1}{R + sL_A + \Delta_A(s)}
\]

보상 OFF 전류 개루프:

\[
L_i(j\omega) =
C_A(j\omega)\,G'_A(j\omega)\,D_i(j\omega)\,H_i(j\omega)
\]

SPMSM은 \(L_A=L_B=L_s\), 반대 축 제어기도 공통 \(K_p, K_i\)를 쓴다. IPMSM은 선택 축이 d이면 반대 축 q의 \(K_{p,q}, K_{i,q}\), 선택 축이 q이면 반대 축 d의 \(K_{p,d}, K_{i,d}\)를 쓴다.

현재 입력의 \(I_\mathrm{load}\)는 UI에 존재하지만 이 소신호 보정 공식에는 직접 들어가지 않는다.

## 2. 캐스케이드 탭

캐스케이드 탭은 전류 루프와 속도 루프를 같은 입력 상태에서 함께 계산한다. 전류 루프 계산은 단일 루프 탭과 동일하며, 속도 루프는 내부 전류 루프 응답을 포함한다.

### 2.1 전류 루프 블록

캐스케이드 탭의 전류 루프는 단일 루프 탭의 \(L_i(j\omega)\), \(T_i(j\omega)\)와 같은 수식을 사용한다. 단, 화면 토글은 단일 루프 탭의 토글과 독립이다.

### 2.2 토크 상수

속도 플랜트의 입력은 전류이고 출력은 기계 각속도다. 토크 상수는 모터 타입별로 다음처럼 계산한다.

\[
K_t =
\begin{cases}
K_e & \text{DC}\\
1.5P\lambda_f & \text{SPMSM, IPMSM}
\end{cases}
\]

여기서 \(P\)는 극쌍수, \(\lambda_f\)는 영구자석 자속쇄교다.

### 2.3 속도 PI 제어기

속도 제어기도 병렬형 PI를 사용한다.

\[
C_\omega(s) = K_{p,\omega} + \frac{K_{i,\omega}}{s}
\]

\[
C_\omega(j\omega) = K_{p,\omega} - j\frac{K_{i,\omega}}{\omega}
\]

### 2.4 내부 전류 루프 근사

속도 루프 안쪽에는 전류 루프가 있다. 앱은 두 가지 모드를 제공한다.

`폐루프 Ti 사용` 모드:

\[
I_i(j\omega) = T_i(j\omega) = \frac{L_i(j\omega)}{1 + L_i(j\omega)}
\]

`1차 근사` 모드:

\[
I_i(j\omega) = \frac{1}{1 + j\omega\tau_{ci}}
\]

\[
\tau_{ci} = \frac{1}{2\pi BW_i}
\]

\(BW_i\)는 전류 루프 폐루프 대역폭이다. 값이 없으면 구현상 500 Hz를 임시 기준으로 사용한다.

### 2.5 속도 플랜트

기계계는 1차 플랜트로 계산한다.

\[
G_\omega(s) = \frac{K_t}{Js + B}
\]

주파수 응답:

\[
G_\omega(j\omega) = \frac{K_t}{B + j\omega J}
\]

\(J\)는 관성, \(B\)는 점성 마찰이다. \(B=0\)이면 이상적인 적분형 기계 플랜트가 된다.

### 2.6 속도 루프 지연과 센서 LPF

속도 루프 입력 단위 변환:

| 입력 | 계산값 |
|---|---|
| \(T_{sw}\) [\(\mu s\)] | \(T_{sw} = T_{sw}^\mathrm{input} \times 10^{-6}\) [s] |
| \(\tau_\omega\) [ms] | \(\tau_\omega = \tau_\omega^\mathrm{input} \times 10^{-3}\) [s] |

속도 루프 등가 지연:

\[
\tau_{d\omega} = \alpha_\omega T_{sw}
\]

\[
D_\omega(j\omega) =
e^{-j\omega\tau_{d\omega}}
= \cos(\omega\tau_{d\omega}) - j\sin(\omega\tau_{d\omega})
\]

속도 센서가 `이상`이면:

\[
H_\omega(j\omega) = 1
\]

속도 센서가 `1차 LPF`이면:

\[
H_\omega(j\omega) = \frac{1}{1 + j\omega\tau_\omega}
\]

앱 입력에서 \(\tau_\omega\)는 ms 단위이고 계산 시 초 단위로 변환된다.

### 2.7 속도 개루프와 폐루프

속도 개루프:

\[
L_\omega(j\omega) =
C_\omega(j\omega)\,
I_i(j\omega)\,
G_\omega(j\omega)\,
D_\omega(j\omega)\,
H_\omega(j\omega)
\]

속도 폐루프:

\[
T_\omega(j\omega) =
\frac{L_\omega(j\omega)}{1 + L_\omega(j\omega)}
\]

### 2.8 캐스케이드 건전성 비율

전류 루프가 속도 루프보다 충분히 빠른지 다음 비율로 판단한다.

\[
r = \frac{BW_i}{BW_\omega}
\]

현재 상태 판정:

| 조건 | 상태 |
|---|---|
| \(5 \le r \le 10\) | 양호 |
| \(3 \le r \le 15\), 단 양호 범위 밖 | 주의 |
| 그 외 | 위험 |

속도 루프 대역폭을 찾지 못하거나 \(BW_\omega \le 0\)이면 비율은 표시하지 않는다.

## 3. 두 탭 공통 지표

전류 루프와 속도 루프는 같은 지표 추출기를 사용한다. 여기서 \(L\)은 해당 루프의 개루프, \(T\)는 해당 루프의 폐루프다.

### 3.1 이득 교차와 위상 여유

이득 교차 주파수:

\[
|L(j\omega_c)| = 1
\]

dB 기준:

\[
20\log_{10}|L(j\omega_c)| = 0
\]

위상 여유:

\[
PM = 180^\circ + \angle L(j\omega_c)
\]

교차점이 여러 개 있으면 모든 이득 교차점의 PM 중 가장 작은 값을 지표로 사용한다.

### 3.2 위상 교차와 이득 여유

위상 교차 주파수:

\[
\angle L(j\omega_p) = -180^\circ
\]

이득 여유:

\[
GM = -20\log_{10}|L(j\omega_p)|
\]

교차점이 여러 개 있으면 모든 위상 교차점의 GM 중 가장 작은 값을 지표로 사용한다.

### 3.3 폐루프 대역폭

폐루프 대역폭은 폐루프 크기가 -3 dB가 되는 첫 교차점이다.

\[
20\log_{10}|T(j\omega_{BW})| = -3
\]

\[
BW = \frac{\omega_{BW}}{2\pi}
\]

### 3.4 공진 피크

공진 피크는 폐루프 dB 크기의 최댓값이다.

\[
M_p = \max_\omega 20\log_{10}|T(j\omega)|
\]

앱에서는 dB 단위로 표시한다.

### 3.5 저주파 이득과 고주파 롤오프

저주파 이득:

\[
G_\mathrm{low} = 20\log_{10}|L(j\omega_\mathrm{min})|
\]

고주파 롤오프는 샘플 후반부 약 1/3 구간의 dB 기울기를 decade당 dB로 계산한다.

\[
\mathrm{rolloff}
= \frac{\Delta(20\log_{10}|L|)}{\Delta(\log_{10}f)}
\quad [\mathrm{dB/dec}]
\]

### 3.6 시간 영역 추정치

시간 영역 값은 2차계 근사다. 실제 전류 루프와 속도 루프는 PI 영점, 지연, 센서 필터, 보상 OFF 비선형 근사 영향을 포함하므로 UI에서도 주의 문구를 표시한다.

현재 구현은 \(0 < PM \le 70^\circ\), \(BW > 0\)일 때만 시간 영역 추정치를 계산한다. 오버슈트와 정착 시간은 추가로 \(0 < \zeta < 1\)일 때 표시된다.

감쇠비 추정:

\[
\zeta \approx \frac{PM[^\circ]}{100}
\]

자연 주파수 추정:

\[
\omega_n \approx
\frac{\omega_{BW}}
{\sqrt{1 - 2\zeta^2 + \sqrt{4\zeta^4 - 4\zeta^2 + 2}}}
\]

오버슈트 추정:

\[
\%OS =
\exp\left(
\frac{-\pi\zeta}{\sqrt{1-\zeta^2}}
\right) \times 100
\]

2% 기준 정착 시간:

\[
t_s \approx \frac{4}{\zeta\omega_n}
\]

앱은 \(t_s\)를 ms로 표시한다.

## 4. 샘플링과 보간

두 탭 모두 로그 주파수 그리드를 사용한다. 현재 UI/ViewModel은 800점을 샘플링한다.

전류 루프 상한:

\[
f_\mathrm{max,i} = \max\left(10,\frac{1}{2T_s}\right)
\]

속도 루프 상한:

\[
f_\mathrm{max,\omega} = \max\left(10,\frac{1}{2T_{sw}}\right)
\]

하한은 0.1 Hz다.

\[
\log_{10}f_k =
\log_{10}(0.1)
+ \frac{k}{N-1}
\left(\log_{10}f_\mathrm{max} - \log_{10}(0.1)\right)
\]

\[
\omega_k = 2\pi f_k
\]

교차 주파수는 인접 샘플 두 점 사이에서 \(\log_{10}f\) 축 선형 보간으로 구한다.

\[
x_\mathrm{cross} =
x_a +
\frac{y_\mathrm{target}-y_a}{y_b-y_a}(x_b-x_a)
\]

여기서 \(x=\log_{10}f\), \(y\)는 dB 또는 위상 값이다.

## 5. 상태 판정과 경고

### 5.1 지표 카드 상태

| 지표 | 양호 | 주의 | 위험 |
|---|---:|---:|---:|
| PM | \(\ge 45^\circ\) | \(\ge 30^\circ\) | \(<30^\circ\) |
| GM | \(\ge 6\) dB | \(\ge 3\) dB | \(<3\) dB |
| \(M_p\) | \(<3\) dB | \(<6\) dB | \(\ge 6\) dB |

### 5.2 단일 루프 전역 경고

지연 한계 경고:

\[
\omega_{BW} > \frac{1}{3\tau_d}
\]

Hz 기준으로는:

\[
BW > \frac{1}{6\pi\tau_d}
\]

샘플링 한계 경고:

\[
BW > \frac{f_s}{4}
\]

\[
f_s = \frac{1}{T_s}
\]

이 경고가 떠도 플롯과 지표는 계속 표시되지만, 감쇠비와 정착 시간 같은 추정값의 신뢰도는 낮게 봐야 한다.

### 5.3 입력 유효성 조건

입력이 다음 조건을 만족하지 않으면 플롯과 지표 대신 입력 오류 배너를 표시한다.

공통 조건:

\[
R > 0,\quad T_s > 0,\quad \alpha \ge 0,\quad J \ge 0,\quad B \ge 0
\]

속도 루프 기계계 조건:

\[
J > 0 \quad \text{or} \quad B > 0
\]

전류 센서 LPF 사용 시:

\[
\tau_i > 0
\]

속도 센서 LPF 사용 시:

\[
\tau_\omega > 0
\]

DC 모터 조건:

\[
L > 0,\quad K_e > 0
\]

SPMSM 조건:

\[
L_s > 0,\quad \lambda_f > 0,\quad P > 0
\]

IPMSM 조건:

\[
L_d > 0,\quad L_q > 0,\quad \lambda_f > 0,\quad P > 0
\]

PI 게인 조건:

\[
K_p > 0,\quad K_i \ge 0
\]

IPMSM 전류 PI는 d/q축 각각 다음을 만족해야 한다.

\[
K_{p,d} > 0,\quad K_{i,d} \ge 0,\quad K_{p,q} > 0,\quad K_{i,q} \ge 0
\]

속도 PI와 속도 샘플링 조건:

\[
K_{p,\omega} > 0,\quad K_{i,\omega} \ge 0,\quad T_{sw} > 0,\quad \alpha_\omega \ge 0
\]

## 6. 공유 이미지에서 쓰는 수식

공유 이미지는 별도 계산식을 갖지 않고 현재 탭의 샘플과 지표를 그대로 렌더링한다.

단일 루프 공유 이미지:

- 모터, 축, 보상 상태
- 활성 \(R\), \(L_\mathrm{active}\), 활성 전류 PI 게인
- \(T_s\), \(\alpha\)
- PM, GM, BW, \(M_p\)

캐스케이드 공유 이미지:

- 전류 루프와 속도 루프 플롯
- 전류 루프 PM/BW/GM/\(M_p\)
- 속도 루프 PM/BW/GM/\(M_p\)
- \(BW_i/BW_\omega\) 건전성 메시지
