
✿ 개요


- Vision-Language 모델 기반 스케치 인식 및 추천 서비스 개발
    - Vision-Language(ViL) 모델을 활용하여 손그림 스케치 객체 분류 모델을 개발
    - 사진 이미지 분류에 최적화된 기존 ViL 모델에 Domain Adaptation 기법을 적용하여,
        
        선 중심·추상적 표현을 갖는 스케치 이미지 도메인에서도 안정적인 인식 성능 확보
        
    - 스케치 인식 결과를 기반으로 유사 그림 레퍼런스 자동 추천 서비스 설계 및 구현
    - 스케치 입력부터 레퍼런스 추천까지의 전체 파이프라인을 갖춘 모바일 앱 개발 및 배포
    


✿ 역할


- 스케치 인식 서비스의 백엔드 및 모델 학습 파이프라인 설계·구현
- YOLO 모델을 Focal Loss로 Fine-tuning하여 스케치 이미지에서 다중 객체 탐지를 수행
- 사진 이미지 중심으로 학습된 Vision-Language 모델에 Domain Adaptation 기법을 적용하여 스케치 이미지 분류 성능 개선
    - 스케치 도메인의 클래스 불균형 및 경계 모호성 문제를 해결하기 위해 Quantile Margin Weighted Loss(QMW Loss) 제안 및 적용
    - LoRA(Low-Rank Adaptation) 기반 Fine-tuning 전략 수립을 통해 파라미터 효율적인 성능 향상 달성
- 스케치 인식 결과를 기반으로 한 그림 레퍼런스 추천 파이프라인 구축 및 모바일 앱 서비스 연동·배포

✿ 코드
- yolov11_train_forcls_using_focal.ipynb
    - Focal Loss를 활용한 YOLO Fine-tuning 코드
 
- train_qwmLora.ipynb
    - LoRA + QMW Loss를 적용한 CLIP Fine-tuning 코드
