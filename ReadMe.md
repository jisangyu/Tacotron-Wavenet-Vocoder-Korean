# 고급 기계 학습 텀 프로젝트
음성 합성 모델의 성능 향상을 테스트 해보기 위한 프로젝트입니다.
베이스 라인 코드에 대한 내용은 [hccho2/Tacotron](https://github.com/hccho2/Tacotron-Wavenet-Vocoder-Korean)를 확인하면 됩니다

## 베이스 모델
son 폴더에 압축을 풀것
aml 폴더에 전처리 완료된 파일 사용하면 됨

## 훈련 시작
```
python train_tacotron.py 
```
디폴트 입력으로 aml 파인튜닝 모드가 지정되어 있음

## loss 결과 확인
```
tensorboard --log_dir={훈련경로} 
```

# 실험 내용
- 하이퍼 파라미터 실험 (러닝레이트 / 배치사이즈)
- 데이터 어규멘테이션 실험 (G2P)
- 데이터 어댑테이션 실험 (Finetune)
- 데이터 필터링 실험



## Data 전처리 
- audio data(e.g. wave 파일)을 다운받고,  1~3초(최대 12초)길이로 잘라주는 작업을 해야 한다. 그리고 잘라진 audio와 text(script)의 sync를 맞추는 것은 고단한 작업이다. Google Speech API를 이용하는 것도 하나의 방법이 될 수 있다.
- Google Speech API로 생성한 text의 Quality가 좋지 못하기 때문에, 수작업으로 (아주) 많이 보정해 주어야 한다.
- 특별히 data를 확보할 방법이 없으면, [carpedm20](https://github.com/carpedm20/multi-speaker-tacotron-tensorflow)에서 설명하고 있는대로 하면 된다. 여기서는 data를 다운받은 후, 침묵(silence)구간을 기준으로 자른 후, Google Speech API를 이용하여 text와 sync를 맞추고 있다.
- 한글 data는 [KSS Dataset](https://www.kaggle.com/bryanpark/korean-single-speaker-speech-dataset)가 있고, 영어 data는 [LJ Speech Dataset](https://keithito.com/LJ-Speech-Dataset/), [VCTK corpus](http://homepages.inf.ed.ac.uk/jyamagis/page3/page58/page58.html) 등이 있다.
- KSS Dataset이나 LJ Speech Dataset는 이미 적당한 길이로 나누어져 있기 때문에, data의 Quality는 우수하다.
- 각 speaker별로 wav 파일을 특정 directory에 모은 후, text와 wav파일의 관계를 설정하는 파일을 만든 후, preprocess.py를 실행하면 된다. 다음의 예는 son.py에서 확인 할 수 있듯이 'son-recognition-All.json'에 필요한 정보를 모아 놓았다.
- 각자의 상황에 맞게 preprocessing하는 코드를 작성해야 한다. 이 project에서는 son, moon 2개의 example이 포함되어 있다.
> python preprocess.py --num_workers 8 --name son --in_dir .\datasets\son --out_dir .\data\son
- 위의 과정을 거치든 또는 다른 방법을 사용하든 speaker별 data 디렉토리에 npz파일이 생성되면 train할수 있는 준비가 끝난다. npz파일에는 dict형의 data가 들어가게 되는데, key는 ['audio', 'mel', 'linear', 'time_steps', 'mel_frames', 'text', 'tokens', 'loss_coeff']로 되어 있다. 중요한 것은 audio의 길이가 mel, linear의 hop_size 배로 되어야 된다는 것이다.


## 음성 합성
> python synthesizer.py --load_path datasets/son --num_speakers 1 --speaker_id 0 --text "오스트랄로피테쿠스 아파렌시스는 멸종된 사람족 종으로, 현재에는 뼈 화석이 발견되어 있다." 

