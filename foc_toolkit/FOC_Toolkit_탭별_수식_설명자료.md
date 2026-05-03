# FOC Toolkit 탭별 수식 설명 자료

이 문서는 현재 앱 레지스트리와 계산 엔진 기준으로 각 계산기 탭에서 사용하는 수식을 정리한 자료이다. 화면에 표시되는 대표 수식뿐 아니라 실제 계산에 사용되는 보정식, 포화 처리, 차트용 샘플링 식도 함께 포함한다.

## 0. 공통 기호

| 기호 | 의미 | 단위 |
| --- | --- | --- |
| $P$ | 극쌍수 | - |
| $\theta_e$ | 전기각 | rad |
| $\omega_m$ | 기계 각속도 | rad/s |
| $\omega_e$ | 전기 각속도 | rad/s |
| $f_m$ | 기계 회전 주파수 | Hz |
| $f_e$ | 전기각 주파수 | Hz |
| $V_{dc}$ | DC 링크 전압 | V |
| $V_{max}$ | dq 전압 벡터의 출력 가능 상한, 상전압 피크 기준 | V |
| $R_s$ | 고정자 저항 | $\Omega$ |
| $L_s$ | SPMSM 동기 인덕턴스, $L_d=L_q=L_s$ | H |
| $\psi_f$, $\lambda_m$ | 영구자석 쇄교 자속 | Wb |
| $I_{rated}$, $I_{max}$ | 정격 피크 전류, dq 전류 벡터 크기 기준 | Apk |

변조 방식별 전압 한계는 다음과 같이 계산한다.

$$
V_{max}=V_{dc}\cdot k_v
$$

| 변조 방식 | $k_v$ | 의미 |
| --- | --- | --- |
| SVPWM | $\frac{1}{\sqrt{3}}$ | 선형 SVPWM의 상전압 피크 한계 |
| SPWM | $\frac{1}{2}$ | 정현파 PWM의 상전압 피크 한계 |

## 1. Clarke 변환 (abc -> alpha beta)

입력은 3상 전류 $i_a, i_b, i_c$이고 출력은 정지 좌표계 전류 $i_\alpha, i_\beta$이다. 앱은 전역 설정에 따라 진폭불변과 전력불변 두 모드를 지원한다.

### 진폭불변 모드

$$
i_\alpha=\frac{2}{3}\left(i_a-\frac{1}{2}i_b-\frac{1}{2}i_c\right)
$$

$$
i_\beta=\frac{i_b-i_c}{\sqrt{3}}
$$

균형 3상 조건 $i_a+i_b+i_c=0$에서는 $i_\alpha=i_a$이고 $i_\beta=(i_a+2i_b)/\sqrt{3}$인 간이식과 같은 결과가 나온다. 이 모드는 상전류의 피크 크기를 직관적으로 유지하는 데 유리하다.

### 전력불변 모드

$$
i_\alpha=\sqrt{\frac{2}{3}}\left(i_a-\frac{1}{2}i_b-\frac{1}{2}i_c\right)
$$

$$
i_\beta=\sqrt{\frac{2}{3}}\cdot\frac{\sqrt{3}}{2}(i_b-i_c)
$$

전력불변 모드는 좌표 변환 후 벡터 전력이 보존되도록 스케일을 적용한다.

## 2. Clarke 역변환 (alpha beta -> abc)

입력은 $i_\alpha, i_\beta$이고 출력은 $i_a, i_b, i_c$이다. 영상분 전류는 별도로 다루지 않으므로 3상 합이 0인 성분으로 복원한다.

### 진폭불변 모드

$$
i_a=i_\alpha
$$

$$
i_b=\frac{-i_\alpha+\sqrt{3}i_\beta}{2}
$$

$$
i_c=\frac{-i_\alpha-\sqrt{3}i_\beta}{2}
$$

### 전력불변 모드

$$
i_a=\sqrt{\frac{2}{3}}i_\alpha
$$

$$
i_b=\sqrt{\frac{2}{3}}\left(-\frac{1}{2}i_\alpha+\frac{\sqrt{3}}{2}i_\beta\right)
$$

$$
i_c=\sqrt{\frac{2}{3}}\left(-\frac{1}{2}i_\alpha-\frac{\sqrt{3}}{2}i_\beta\right)
$$

## 3. Park 변환 (alpha beta -> dq)

정지 좌표계 벡터를 전기각 $\theta_e$만큼 회전한 동기 회전 좌표계로 변환한다.

$$
i_d=i_\alpha\cos\theta_e+i_\beta\sin\theta_e
$$

$$
i_q=-i_\alpha\sin\theta_e+i_\beta\cos\theta_e
$$

$d$축은 로터 자속 방향, $q$축은 토크를 만드는 직교 방향으로 해석한다.

## 4. Park 역변환 (dq -> alpha beta)

dq 전류나 전압 지령을 정지 좌표계로 되돌린다.

$$
i_\alpha=i_d\cos\theta_e-i_q\sin\theta_e
$$

$$
i_\beta=i_d\sin\theta_e+i_q\cos\theta_e
$$

전압 지령에도 같은 형태를 적용한다.

$$
V_\alpha=V_d\cos\theta_e-V_q\sin\theta_e
$$

$$
V_\beta=V_d\sin\theta_e+V_q\cos\theta_e
$$

## 5. 통합: abc -> dq

이 탭은 Clarke 변환과 Park 변환을 순서대로 적용한다.

$$
(i_a,i_b,i_c)\xrightarrow{\text{Clarke}}(i_\alpha,i_\beta)\xrightarrow{\text{Park}}(i_d,i_q)
$$

실제 계산 순서는 다음과 같다.

$$
(i_\alpha,i_\beta)=\text{Clarke}(i_a,i_b,i_c)
$$

$$
i_d=i_\alpha\cos\theta_e+i_\beta\sin\theta_e
$$

$$
i_q=-i_\alpha\sin\theta_e+i_\beta\cos\theta_e
$$

Clarke 단계의 스케일은 전역 Clarke 모드에 따라 달라진다.

## 6. 통합: dq -> abc

이 탭은 Park 역변환과 Clarke 역변환을 순서대로 적용한다.

$$
(i_d,i_q)\xrightarrow{\text{Inverse Park}}(i_\alpha,i_\beta)\xrightarrow{\text{Inverse Clarke}}(i_a,i_b,i_c)
$$

실제 계산 순서는 다음과 같다.

$$
i_\alpha=i_d\cos\theta_e-i_q\sin\theta_e
$$

$$
i_\beta=i_d\sin\theta_e+i_q\cos\theta_e
$$

$$
(i_a,i_b,i_c)=\text{Clarke}^{-1}(i_\alpha,i_\beta)
$$

Clarke 역변환 단계의 스케일은 전역 Clarke 모드에 따라 달라진다.

## 7. 통합: dq -> SVPWM / SPWM

이 탭은 dq 전압 지령을 $\alpha\beta$ 전압으로 바꾼 뒤 선택한 PWM 방식으로 듀티를 계산한다.

$$
(V_d,V_q)\xrightarrow{\text{Inverse Park}}(V_\alpha,V_\beta)\xrightarrow{\text{PWM}}(d_a,d_b,d_c)
$$

### dq -> alpha beta

$$
V_\alpha=V_d\cos\theta_e-V_q\sin\theta_e
$$

$$
V_\beta=V_d\sin\theta_e+V_q\cos\theta_e
$$

### SVPWM 선택 시

아래 8장의 SVPWM 수식을 그대로 사용한다.

### SPWM 선택 시

아래 9장의 SPWM 수식을 사용하되, 이 통합 탭에서는 변조율을 고정값 $m=0.9$로 계산한다.

## 8. SVPWM

입력은 $V_\alpha, V_\beta, V_{dc}$이고 출력은 섹터, $T_1$, $T_2$, $T_0$, 3상 듀티이다.

### 기준 벡터와 섹터

$$
\theta=\operatorname{atan2}(V_\beta,V_\alpha)
$$

음수 각도는 $2\pi$를 더해 $0\le\theta<2\pi$ 범위로 변환한다.

$$
|V|=\sqrt{V_\alpha^2+V_\beta^2}
$$

$$
\text{sector}=\min\left(\left\lfloor\frac{\theta}{\pi/3}\right\rfloor+1,6\right)
$$

$$
\theta_s=\theta-(\text{sector}-1)\frac{\pi}{3}
$$

$\theta_s$는 현재 섹터 내부 각도이다.

### Dwell time

앱은 진폭불변 Clarke 기준으로 다음 변조 계수를 사용한다.

$$
m=\frac{\sqrt{3}|V|}{V_{dc}}
$$

$$
T_1=m\sin\left(\frac{\pi}{3}-\theta_s\right)
$$

$$
T_2=m\sin\theta_s
$$

$$
T_0=1-T_1-T_2
$$

선형 영역을 넘어서 $T_0<0$이면 활성 벡터 시간을 다음처럼 정규화해 과변조 클리핑을 수행한다.

$$
T_1'=\frac{T_1}{T_1+T_2},\quad T_2'=\frac{T_2}{T_1+T_2},\quad T_0'=0
$$

### 듀티 계산

$$
h=\frac{T_0}{2}
$$

| 섹터 | $d_a$ | $d_b$ | $d_c$ |
| --- | --- | --- | --- |
| 1 | $h+T_1+T_2$ | $h+T_2$ | $h$ |
| 2 | $h+T_1$ | $h+T_1+T_2$ | $h$ |
| 3 | $h$ | $h+T_1+T_2$ | $h+T_2$ |
| 4 | $h$ | $h+T_1$ | $h+T_1+T_2$ |
| 5 | $h+T_2$ | $h$ | $h+T_1+T_2$ |
| 6 | $h+T_1+T_2$ | $h$ | $h+T_1$ |

$V_{dc}\le0$이면 안전 기본값으로 $T_1=0$, $T_2=0$, $T_0=1$, $d_a=d_b=d_c=0.5$를 반환한다.

## 9. SPWM

입력은 $V_\alpha, V_\beta, V_{dc}$, 변조율 $m$이고 출력은 3상 듀티이다.

### alpha beta -> 3상 전압

SPWM 엔진은 진폭불변 역 Clarke 형태로 3상 기준 전압을 만든다.

$$
v_a=V_\alpha
$$

$$
v_b=\frac{-V_\alpha+\sqrt{3}V_\beta}{2}
$$

$$
v_c=\frac{-V_\alpha-\sqrt{3}V_\beta}{2}
$$

### 듀티

$$
d_a=\operatorname{clip}\left(0.5+\frac{v_a}{V_{dc}}m,0,1\right)
$$

$$
d_b=\operatorname{clip}\left(0.5+\frac{v_b}{V_{dc}}m,0,1\right)
$$

$$
d_c=\operatorname{clip}\left(0.5+\frac{v_c}{V_{dc}}m,0,1\right)
$$

$\operatorname{clip}(x,0,1)$은 값을 0과 1 사이로 제한한다. $V_{dc}\le0$이면 안전 기본값으로 $d_a=d_b=d_c=0.5$를 반환한다.

### 파형 표시

차트에는 360개 샘플로 정현파와 삼각 캐리어를 표시한다.

$$
t_i=\frac{i}{360}2\pi,\quad i=0,1,\ldots,359
$$

$$
w_a(t)=m\sin t
$$

$$
w_b(t)=m\sin\left(t-\frac{2\pi}{3}\right)
$$

$$
w_c(t)=m\sin\left(t+\frac{2\pi}{3}\right)
$$

$$
c(t)=\frac{2}{\pi}\arcsin(\sin 9t)
$$

캐리어 식은 화면 표시용 삼각파이며 실제 듀티 계산은 위의 정적 듀티식을 사용한다.

## 10. 속도, 주파수 변환

이 탭은 기계 속도와 전기각 속도를 양방향으로 동기화한다.

### 기계 RPM -> 기계 각속도

$$
\omega_m=\text{RPM}\cdot\frac{2\pi}{60}
$$

### 기계 각속도 -> RPM

$$
\text{RPM}=\omega_m\cdot\frac{60}{2\pi}
$$

### RPM과 기계 주파수

$$
f_m=\frac{\text{RPM}}{60}
$$

$$
\text{RPM}=60f_m
$$

### 기계 주파수와 전기각 주파수

$$
f_e=f_mP
$$

$$
f_m=\frac{f_e}{P}
$$

### 전기각 주파수와 전기 각속도

$$
\omega_e=2\pi f_e
$$

$$
f_e=\frac{\omega_e}{2\pi}
$$

### 전기 각속도와 기계 각속도

$$
\omega_e=P\omega_m
$$

$$
\omega_m=\frac{\omega_e}{P}
$$

## 11. 전기각/샘플

입력은 전기각 주파수 $f_e$와 제어 주파수 $f_c$이다.

$$
T_s=\frac{1}{f_c}
$$

$$
\Delta\theta_e=2\pi f_eT_s=\frac{2\pi f_e}{f_c}
$$

화면에는 rad, deg, 1회전 대비 비율을 함께 표시한다.

$$
\Delta\theta_{e,\deg}=\Delta\theta_e\cdot\frac{180}{\pi}
$$

$$
\text{1회전 대비}=\frac{\Delta\theta_e}{2\pi}\cdot100
$$

$f_c\le0$이면 계산값은 0으로 처리한다.

## 12. 역기전력 상수

입력은 쇄교 자속 $\lambda_m$, 극쌍수 $P$, 속도 RPM이다.

### 역기전력 상수

$$
K_e=\lambda_mP
$$

앱의 결과 단위는 $V/(rad/s)$이다. 여기서 각속도는 기계 각속도 기준으로 해석된다.

### 전기각 속도와 전기각 주파수

$$
\omega_m=\text{RPM}\cdot\frac{2\pi}{60}
$$

$$
\omega_e=P\omega_m
$$

$$
f_e=\frac{\text{RPM}}{60}P
$$

### 역기전력 피크

$$
e_{peak}=\lambda_m\omega_e
$$

### 파형 표시

차트에는 기본적으로 2 전기 주기의 사인파를 500개 샘플로 표시한다.

$$
T_e=\frac{1}{f_e}
$$

$$
t_i=\frac{2T_e}{500}i,\quad i=0,1,\ldots,499
$$

$$
e(t)=e_{peak}\sin(2\pi f_et)
$$

화면 표시용 파형은 최대 절대값으로 나누어 정규화한다.

## 13. SPMSM 파라미터 분석

이 화면은 내부적으로 두 개의 분석 탭을 가진다.

- 파라미터 분석: 모터 파라미터로 전압, 속도, 토크, 전력 한계를 계산한다.
- 동작점 계산: 특정 속도와 전류 또는 토크 지령에서 전압, 전력, 운전 가능성을 계산한다.

### 13.1 전압 한계

$$
V_{max}=V_{dc}\cdot k_v
$$

$$
k_v=
\begin{cases}
\frac{1}{\sqrt{3}}, & \text{SVPWM}\\
\frac{1}{2}, & \text{SPWM}
\end{cases}
$$

### 13.2 기저속도

화면 상단 대표식은 $R_s=0$일 때의 단순식이다.

$$
\omega_{base}=\frac{V_{max}}{\sqrt{(L_sI_{rated})^2+\psi_f^2}}
$$

실제 엔진은 $R_s$ 전압강하를 포함해 다음 조건의 양의 근을 사용한다.

$$
V_{max}^2=(\omega L_s I_{rated})^2+(R_sI_{rated}+\omega\psi_f)^2
$$

이를 2차방정식으로 쓰면 다음과 같다.

$$
A\omega^2+B\omega+C=0
$$

$$
A=(L_sI_{rated})^2+\psi_f^2
$$

$$
B=2\psi_fR_sI_{rated}
$$

$$
C=(R_sI_{rated})^2-V_{max}^2
$$

$$
D=B^2-4AC
$$

$$
\omega_{base}=\max\left(0,\frac{-B+\sqrt{D}}{2A}\right)
$$

$D<0$이면 기저속도는 0으로 처리한다.

### 13.3 무부하 최대속도

$$
\omega_{max,0}=\frac{V_{max}}{\psi_f}
$$

기계 RPM으로 환산할 때는 다음 식을 사용한다.

$$
\text{RPM}=\frac{\omega_e}{P}\cdot\frac{60}{2\pi}
$$

### 13.4 약자속 최대속도

$I_d=-I_{rated}$, $I_q=0$ 근사에서 계산한다.

$$
V_{max}^2=(R_sI_{rated})^2+\omega^2(\psi_f-L_sI_{rated})^2
$$

조건 $\psi_f>L_sI_{rated}$ 및 $V_{max}>R_sI_{rated}$를 만족하면 다음 값이 유한하다.

$$
\omega_{max,fw}=\frac{\sqrt{V_{max}^2-(R_sI_{rated})^2}}{\psi_f-L_sI_{rated}}
$$

조건을 만족하지 않으면 화면에는 이론상 무한대로 표시한다.

### 13.5 특성 전류

$$
I_{ch}=\frac{\psi_f}{L_s}
$$

$I_{ch}$는 전압 제한 원의 중심과 MTPV 영역 판단에 사용된다.

### 13.6 정격 토크와 정격 전력

SPMSM에서 $I_d=0$일 때 토크는 다음과 같다.

$$
T=\frac{3}{2}P\psi_fI_q
$$

정격 토크는 $I_q=I_{rated}$로 계산한다.

$$
T_{rated}=\frac{3}{2}P\psi_fI_{rated}
$$

정격 기계 전력은 기저속도에서 계산한다.

$$
P_{rated}=T_{rated}\omega_{m,base}
$$

$$
\omega_{m,base}=\frac{\omega_{base}}{P}
$$

### 13.7 Id = 0 전압포화 q축 전류

파라미터 분석 탭의 현재 속도 슬라이더에서 $I_{q,sat}(\omega_e)$를 계산한다. 실제 엔진은 $R_s$를 포함한 2차방정식의 양의 근을 사용한다.

$$
V_{max}^2=I_q^2(\omega_e^2L_s^2+R_s^2)+2I_q\omega_e\psi_fR_s+\omega_e^2\psi_f^2
$$

$$
A=\omega_e^2L_s^2+R_s^2
$$

$$
B=2\omega_e\psi_fR_s
$$

$$
C=\omega_e^2\psi_f^2-V_{max}^2
$$

$$
I_{q,sat}=\max\left(0,\frac{-B+\sqrt{B^2-4AC}}{2A}\right)
$$

$\omega_e<0$이거나 판별식이 음수이면 포화 전류, 토크, 전력은 표시하지 않는다. $\omega_e=0$이고 $R_s=0$이면 전압 제약이 사라지므로 정격 전류로 제한한다.

$R_s=0$이면 다음 단순식과 같다.

$$
I_{q,sat}=\frac{\sqrt{V_{max}^2-(\omega_e\psi_f)^2}}{\omega_eL_s}
$$

### 13.8 속도별 토크와 전력

전류 한계와 전압 한계를 함께 고려해 실제 사용할 q축 전류를 제한한다.

$$
I_{q,eff}=\min(I_{q,sat},I_{rated})
$$

$$
T(\omega_e)=\frac{3}{2}P\psi_fI_{q,eff}
$$

$$
P(\omega_e)=T(\omega_e)\frac{\omega_e}{P}
$$

차트는 $0$부터 $1.2\omega_{max,0}$까지 300개 점을 샘플링한다.

## 14. SPMSM 동작점 계산

입력은 속도 RPM과 $I_q$ 또는 토크이다. $I_q$와 토크 입력은 같은 식으로 서로 연동된다.

### 14.1 토크와 전류 변환

$$
T=\frac{3}{2}P\psi_fI_q
$$

$$
I_q=\frac{T}{\frac{3}{2}P\psi_f}
$$

화면 입력에서는 음수 속도, 음수 전류, 음수 토크를 0 이상으로 제한한다.

### 14.2 속도 변환

$$
\omega_m=\text{RPM}\cdot\frac{2\pi}{60}
$$

$$
\omega_e=P\omega_m
$$

### 14.3 Id = 0 동작점 전압

동작점 결과 카드는 $I_d=0$ 제어 가정에서 $V_d$, $V_q$, $|V|$를 계산한다.

$$
V_d=-\omega_eL_sI_q
$$

$$
V_q=\omega_e\psi_f+R_sI_q
$$

$$
V_{phase}=|V|=\sqrt{V_d^2+V_q^2}
$$

### 14.4 기계 출력

$$
P_{mech}=T\omega_m
$$

### 14.5 Id = 0 기준 상태 판정

동작점 분석 결과 자체의 상태는 다음 규칙으로 계산한다.

| 조건 | 상태 |
| --- | --- |
| $V_{phase}>V_{max}$ | 운전 불가 |
| $V_{phase}\le V_{max}$ 그리고 $\omega_e>\omega_{base}$ | 약자속 |
| 그 외 | 정상 |

현재 결과 카드와 차트 마커는 아래 TNI 엔벨로프 기반 도달 가능성 판정을 함께 사용해 정토크, 약계자, MTPV, 엔벨로프 초과를 구분한다.

## 15. SPMSM TNI 운전 영역 엔벨로프

동작점 계산 탭의 차트는 SPMSM의 FOC 최대 토크 운전 영역을 속도별로 그린다. 본 장의 모든 폐형해는 고속 근사로 $R_s$를 무시하며, $\omega_{base}$, $\omega_{max,fw}$ 도 동일하게 $R_s=0$ 근사식이라 13장의 $R_s$ 포함식과 수치가 다를 수 있다. 이는 약계자/MTPV 영역식과 진입 경계 모델을 일관되게 유지하기 위한 선택이며, 누락된 $R_s$ 강하 효과는 15.6의 도달 가능성 판정에서 전압 게이트로 별도 차단한다.

### 15.1 정토크 영역

$$
\omega_e\le\omega_{base}
$$

$$
I_d=0,\quad I_q=I_{max}
$$

$$
|I_s|=I_{max}
$$

$$
T=\frac{3}{2}P\psi_fI_{max}
$$

### 15.2 약계자 영역

기저속도 이후 전류 제한 원과 전압 제한 원의 교점을 구한다.

$$
\frac{V_{max}}{\omega_e}=v_\omega
$$

$$
d=\frac{\psi_f^2+(L_sI_{max})^2-v_\omega^2}{2\psi_fL_s}
$$

조건 $d<\min(I_{ch},I_{max})$이면 약계자 교점이 유효하다.

$$
I_d=-d
$$

$$
I_q=\sqrt{I_{max}^2-d^2}
$$

$$
|I_s|=I_{max}
$$

$$
T=\frac{3}{2}P\psi_fI_q
$$

### 15.3 MTPV 영역

$I_{ch}<I_{max}$이고 $d\ge I_{ch}$이면 MTPV 영역으로 처리한다.

$$
I_d=-I_{ch}
$$

$$
I_q=\frac{V_{max}}{\omega_eL_s}
$$

$$
|I_s|=\sqrt{I_{ch}^2+I_q^2}
$$

$$
T=\frac{3}{2}P\psi_fI_q
$$

### 15.4 운전불가 영역

$I_{ch}\ge I_{max}$이고 $d\ge I_{max}$이면 전류 제한 원과 전압 제한 원의 교집합이 없다고 보고 운전불가로 처리한다.

$$
I_d=0,\quad I_q=0,\quad |I_s|=0,\quad T=0
$$

### 15.5 차트 상한과 경계선

TNI 차트는 모터 파라미터에 따라 속도축 상한을 다르게 잡는다.

| 조건 | 속도축 상한 |
| --- | --- |
| $I_{ch}<I_{max}$ | $3\omega_{MTPV}$ |
| $I_{ch}>I_{max}$ | $1.1\omega_{max,fw}$ |
| $I_{ch}=I_{max}$ | $3\omega_{base}$ |

MTPV 진입 속도는 다음 조건이 성립할 때 표시한다.

$$
\omega_{MTPV}=\frac{V_{max}}{\sqrt{(L_sI_{max})^2-\psi_f^2}}
$$

약자속 최대속도 경계는 $\omega_{max,fw}$를 사용한다.

차트의 전류 곡선은 토크 축 위에 함께 표시하기 위해 다음 스케일로 변환한다.

$$
I_{\text{chart}}=|I_s|\cdot\frac{T_{\max,\text{axis}}}{I_{\max,\text{axis}}}
$$

여기서

$$
T_{\max,\text{axis}}=1.1\cdot\frac{3}{2}P\psi_fI_{max}
$$

$$
I_{\max,\text{axis}}=1.1I_{max}
$$

### 15.6 동작점 도달 가능성 판정

현재 지령 토크 $T_{cmd}$를 같은 속도에서의 엔벨로프 최대 토크 $T_{env}$와 비교한다.

$$
\omega_e=\text{RPM}\cdot\frac{2\pi}{60}\cdot P
$$

$$
\epsilon=\max(10^{-6},10^{-3}T_{env})
$$

$$
\epsilon_V=\max(10^{-6},10^{-6}V_{max})
$$

엔벨로프 최적점 $(I_d^{env},I_q^{env})$ 에 13장의 $V_{phase}$ 식($R_s$ 포함)을 대입한 값을 $V_{phase}^{env}$ 로 표기한다. 판정은 다음 순서로 평가한다.

| 조건 | 판정 |
| --- | --- |
| 엔벨로프 영역이 운전불가 | 엔벨로프 바깥, 토크 한계 초과 |
| $T_{cmd}>T_{env}+\epsilon$ | 엔벨로프 바깥, 토크 한계 초과 |
| $V_{phase}^{env}>V_{max}+\epsilon_V$ | 엔벨로프 바깥, 전압 한계 초과 |
| 그 외 | 엔벨로프 안, 해당 영역 색상 표시 |

전압 게이트는 본 장 식이 $R_s=0$ 폐형해라 토크 한계만으로는 잡히지 않는 $R_s$ 강하 효과를 차단하기 위한 안전장치다. 정토크 영역에선 $I_d^{env}=0$ 이라 $V_{phase}^{env}$ 가 $I_d=0$ 가정과 동치이고, 약계자/MTPV 영역에선 $I_d^{env}<0$ 이 $R_s$ 강하 일부를 보상해 FOC 가 실제 도달 가능한 점이 잘못 outside 로 걸리지 않는다.
