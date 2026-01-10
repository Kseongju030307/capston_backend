<aside>
**✿ 개요**

</aside>

- Vision-Language 모델 기반 스케치 인식 및 추천 서비스 개발
    - Vision-Language(ViL) 모델을 활용하여 손그림 스케치 객체 분류 모델을 개발
    - 사진 이미지 분류에 최적화된 기존 ViL 모델에 Domain Adaptation 기법을 적용하여,
        
        선 중심·추상적 표현을 갖는 스케치 이미지 도메인에서도 안정적인 인식 성능 확보
        
    - 스케치 인식 결과를 기반으로 유사 그림 레퍼런스 자동 추천 서비스 설계 및 구현
    - 스케치 입력부터 레퍼런스 추천까지의 전체 파이프라인을 갖춘 모바일 앱 개발 및 배포
    

<aside>
**✿ 역할**

</aside>

- 스케치 인식 서비스의 백엔드 및 모델 학습 파이프라인 설계·구현
- YOLO 모델을 Focal Loss로 Fine-tuning하여 스케치 이미지에서 다중 객체 탐지를 수행
- 사진 이미지 중심으로 학습된 Vision-Language 모델에 Domain Adaptation 기법을 적용하여 스케치 이미지 분류 성능 개선
    - 스케치 도메인의 클래스 불균형 및 경계 모호성 문제를 해결하기 위해 Quantile Margin Weighted Loss(QMW Loss) 제안 및 적용
    - LoRA(Low-Rank Adaptation) 기반 Fine-tuning 전략 수립을 통해 파라미터 효율적인 성능 향상 달성
- 스케치 인식 결과를 기반으로 한 그림 레퍼런스 추천 파이프라인 구축 및 모바일 앱 서비스 연동·배포

<aside>
**✿ 과정**

</aside>

- YOLO(You Only Look Once)를 활용한 다중 객체 탐지
    - 문제 인식
        - YOLO 모델 학습에서 발생한 Class Imbalance가 주요 성능 저하 요인이라고 판단
        - 원인
            - YOLO는 객체의 클래스를 예측하는 과정에서 **Cross Entropy(CE) Loss**를 기본 Loss로 사용
            - CE Loss의 한계
                - 쉬운 예제(배경 또는 정확도가 높은 클래스)와 어려운 예제(정확도가 낮은 클래스)를 동일하게 취급
                - 데이터 분포상 쉬운 예제가 상대적으로 많을 경우, 전체 Loss에서 어려운 예제가 차지하는 비중이 매우 작아짐
                
            
            ⇒ CE 대신 Focal Loss를 이용하여 학습 진행
            
            Focal Loss : 분류하기 어려운 예제의 중요도는 상대적으로 높여 Class Imbalance 문제 해결
            
            $\mathrm{FL}(p_t) = - \alpha_t (1 - p_t)^{\gamma} \log(p_t)$
            
            $\gamma$  : 감쇠 계수, $p_t$가 높은 예제의 Loss를 줄여 $p_t$가 낮은 예제에 더 집중하게 만드는 역할
            
            $\alpha_t =\begin{cases}\alpha & \text{Foreground} \\1 - \alpha & \text{Background}\end{cases}$ 
            
            - 분류해야 하는 Class가 있는 Foreground와 배경이 있는 Background의 불균형을 보정하기 위한 가중치
            
            $p_t =\begin{cases}p & \text{if } y = 1 \\1 - p & \text{otherwise}\end{cases}$
            
            - 클래스 $t$에 대한 정답을 맞힐 확률
        - 결과
            - Focal Loss를 적용하여 YOLO 모델을 Fine-tuning한 경우 기존 Cross Entropy(CE) Loss 대비 성능 개선이 명확하게 나타남
            - 기존 모델에서 P-score와 R-score가 낮았던 하위 클래스(Bottom classes)에서 높은 성능 향상을 보임
            
            ![image.png](attachment:6d1f07aa-0219-47dc-8494-76b55a1f10a2:image.png)
            
            ![image.png](attachment:3619eec0-d50a-48e4-99d0-71898b64054c:image.png)
            
- LoRA(Low-Rank Adaptation)
    - 문제 인식
        - 스케치 인식 과제에서는 사진 이미지에 비해 표현 정보가 제한적이고 도메인 차이가 큰 데이터 특성으로 인해, 사전학습된 Vision-Language 모델을 단순 Fine-tuning 할 경우 아래와 같은 문제가 발생
            - 전체 파라미터 Fine-tuning 시 과적합 위험 증가 및 학습 안정성 저하
            - 제한된 스케치 데이터 규모로 인해 기존 시각 표현(사진 도메인)의 일반화 성능 손실
            - 학습 비용 및 메모리 사용량 증가로 실험 반복 및 구조 비교의 한계
        - 사전학습된 표현은 최대한 유지하면서 도메인 적응만 수행할 수 있는 방법이 필요하다고 판단
        
        ⇒ 사전학습된 표현을 보존하면서 도메인 차이만 학습할 수 있도록 CLIP LoRA를 적용
        
    - 적용 과정
        - Transformer 블록 내에서 LoRA 적용 위치에 따른 성능 영향을 분석
            - Attention 모듈의 Query, Key, Value, Output Projection 계층에 LoRA 적용
            - Value 및 Output Projection 계층에 선택적으로 LoRA 적용
            - Transformer 블록의 MLP 계층에 LoRA 적용
            
    - 결과
        - MLP 계층에 LoRA를 적용한 경우 가장 높은 분류 정확도 달성
        
        ![image.png](attachment:75d1b87f-e430-4cc4-9310-67e56a9abda2:image.png)
        

- Quantile-Margin Weighted Loss (QMW Loss) 제안
    - 문제 인식
        - DomainNet Sketch 데이터셋은 클래스 간 데이터 분포 불균형이 매우 심한 특징을 가짐
            - Square: 727 samples
            - Dresser: 13 samples
            
            ⇒ 다수 클래스에 대한 학습 편향으로 소수 클래스 인식 성능 저하 발생
            
        - Cross Entropy 또는 Focal Loss 기반 학습 시 단순히 샘플 난이도(hard/easy)만 반영되고 클래스 분포 차이는 충분히 고려되지 않음
        - 결과적으로 전체 정확도는 유지되지만, 클래스 간 성능 편차가 크게 발생
        
        ⇒ 모델 성능 향상을 위해샘플의 분류 난이도와 클래스 불균형을 동시에 고려할 수 있는 새로운 Loss 설계가 필요하다고 판단
        
    - 적용 과정
        - 아이디어
            - 정답 클래스와 오답 클래스 간의 Margin를 기반으로 샘플별 중요도를 동적으로 조절하는 Loss 함수
            - 가정 : 데이터 개수가 많은 클래스는 학습 과정에서 높은 confidence를 형성하고, 데이터 개수가 적은 클래스는 상대적으로 낮은 confidence를 가질 것이다.
        - 작동 방식
            - 정답 클래스의 logit 값과, 오답 클래스 중 최대 logit 값의 차이를 통해 각 샘플의 margin을 계산
            - 계산된 margin 분포에서 𝛼-quantile을 기준으로 샘플을 구분
            - margin이 작은 샘플에 더 큰 가중치를, margin이 충분히 큰 샘플에는 상대적으로 작은 가중치를 적용하여 학습
        
        ![image.png](attachment:9fdde372-bebb-412f-9a72-5622536bf765:image.png)
        
        - 수식
            - $z_i^{\text{true}} : \text{샘플 } i \text{에 대해, 정답 클래스 } y_i \text{와의 유사도}$
            - $z_i^{(\text{max-neg})} : \text{정답이 아닌 클래스 중 가장 높은 유사도}$
            - $r : \text{가중치를 결정하는 기준 값으로, 현재 배치 내 margin들의 } \alpha \text{ 분위수}$
            - $n_{y_i} : \text{정답 클래스 } y_i \text{의 빈도수}$
            - $\omega_i : \text{샘플 } i \text{의 가중치}$
                - $z_i^{\text{true}} = z_{i,y_i},\quad z_i^{(\text{max-neg})} = \max_{j \neq y_i} z_{i,j}$
                - $\text{margin}_i = z_i^{\text{true}} - z_i^{(\text{max-neg})}$
                - $r = \operatorname{Quantile}_{\alpha}\left(\{\text{margin}_i\}_{i=1}^{B}\right)$
                - $\omega_i =\begin{cases}\dfrac{1}{\log(n_{y_i} + 1)} & \text{if } \text{margin}_i < r \\\log(n_{y_i} + 1) & \text{otherwise}\end{cases}$
                - $\mathcal{L}_{\text{QMW}} = \omega_i \cdot \mathcal{L}_{\text{CLIP}}$
    
    - Ablation Study
        - LoRA와 QMW Loss를 사용하였을 때 가장 높은 분류 정확도에 도달
        
        ![image.png](attachment:36ab0f79-fa45-4b7c-ae23-17712e4a0b6c:image.png)
        

<aside>
**✿ 결과**

</aside>

- 본 프로젝트의 결과물로, 사용자가 입력한 스케치 이미지를 기반으로 유사한 시각적 레퍼런스를 자동으로 추천하는 모바일 애플리케이션을 구현
- 핵심기능
    - 이미지 전처리 및 객체 탐지
        - 입력된 스케치 이미지를 전처리한 후, YOLO 기반 다중 객체 탐지 모델을 활용하여 스케치 내 주요 객체를 검출
    - 스케치 인식
        - 검출된 객체를 Domain Adaptation을 적용한 CLIP 모델로 인식
    - 레퍼런스 추천
        - 인식된 객체 클래스 정보를 기반으로
        Unsplash API를 활용하여 유사한 이미지 레퍼런스를 자동 추천
    - 흐름
    
    ![image.png](attachment:75ffba81-2a4b-42c5-b166-f120f02d61b2:image.png)
    
- 주요 화면

![image.png](attachment:5d9d460b-a408-4429-8e1b-d24606761921:image.png)
