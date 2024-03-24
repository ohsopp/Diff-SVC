# Diff-SVC PJT

References for Diffusion Singing Voice Conversion <br/><br/>

Youtube Link: https://youtu.be/RI_NoGU0Lro?si=bJ4-HkuyhJdpzefL <br/><br/>

<h1> ✔ 모델 </h1>

Diffusion Singing Voice Conversion(이하  Diff-SVC)은 음성 변환 기술의 한 형태이다. 여기서 말하는 음성 변환은 한 목소리의 특징을 다른 목소리로 변환하는 기술이다. 이는 보다 구체적으로는 음성의 높낮이, 음조, 발음 등을 변환하여 다른 사람의 목소리로 재생성하는 것을 의미한다.

 

SVC 모델은 content encoder를 학습시켜 input data로부터 content feature를 추출하고 conversion model을 학습시켜 추출한 content feature를 다른 feature로 교체한다. 보통 conversion model은 GAN 또는 regression model을 사용한다. GAN이 사용되는 경우 content feature로부터 파형을 바로 생성하고, regression model이 사용되는 경우 content feature를 spectral feature로 변환한 후 추가로 학습시킨 vocoder로 파형을 생성한다. Diff-SVC에서는 후자의 경우를 사용하며 conversion model로 regression model 대신 diffusion model을 사용한다. 이름에서 알 수 있듯이 diffusion model은 Diff-SVC의 기반 모델이다.

 

Diffusion model은 주로 image processing과 computer vision 분야에서 활용된다. Diffusion model은 이미지를 픽셀 단위로 처리하며, 각 픽셀은 주변 픽셀과의 상호 작용을 통해 정보를 전파한다. 이때 정보는 픽셀 간의 유사성을 기반으로 이루어지고 픽셀 값이 서로 유사한 경우 해당 정보를 주변 픽셀로 전달한다. 위 과정들이 반복적으로 진행되며 횟수를 거듭하면서 이미지의 노이즈가 감소하거나 복원되는 효과를 얻을 수 있다. 일반적인 SVC 기술은 딥러닝 신경망을 사용하여 입력 음성과 대상 음성간의 매핑 함수를 학습하지만 diffusion model을 활용하면 음성 특징을 시간적으로 확산시켜 input voice와 target voice 간의 상관 관계를 모델링한다.

 

Diffusion model에는 forward process와 reverse process가 있다. Forward process는 데이터에 점진적으로 약간의 Gaussian noise를 더하여 완전한 가우시안 분포로 만드는 과정이다. 반대로 reverse process는 Gaussian noise로부터 원데이터로 복구해가는 과정이다. 이는 각 process가 마르코프 체인(Markov chain)이기에 가능한 것이다.
<br/>

![image](https://github.com/ohsopp/Diff-SVC/assets/28973935/a1e23a69-bde5-4621-bf64-9136f5a5aedb)


<br/><br/><br/>

 

<h1> ✔ 데이터셋 준비 </h1>
배경음이 없는, 노래 음성과 말하는 음성을 준비한다.

파일 형식은 샘플 전송률 44.1kHz, 16 bits 모노채널인 wav 파일로 변환한다.

각 파일의 최대 길이는 15초 정도를 넘지 않는 것이 좋으며 음의 높낮이가 다양하고 음성과 음성 사이에 공백이 적을수록 양질의 데이터이다.

데이터셋의 총합 길이가 길수록 자연스럽운 결과를 뽑아낼 수 있다. 통상 1시간 정도의 데이터셋이 효율적이다.

zip 파일로 압축한다.

<br/><br/><br/>
 


<h1> ✔ 모델 학습 </h1>
<h3> Check Setup </h3>

사용가능한 코랩 gpu 가속기를 확인한다. 기본 gpu 유형은 T4이며 구글 코랩 pro 구독시 제한된 컴퓨팅 단위로 A100 변경 가능하다. gpu 성능에 따라 학습 속도에 차이가 꽤 발생한다.

 

<h3> Mount Google Drive </h3>

구글 드라이브와 코랩을 연동한다. 데이터셋을 업로드하거나 학습된 모델을 저장하기 위해 구글 드라이브 연동이 필요하다.



<h4> - Step 1: Install Diff-SVC </h4>
Diff-SVC 모델을 설치하는 과정이다. 디폴트로 UtaUtaUtau's Repo와 44.1kHz의 sample rate이 설정되어 있으며 이 설정은 변경할 필요없다.
<br/>
 
<h4> - Step 2: Decompress dataset </h4>
zip 데이터 파일을 압축해제하는 과정이다. singer_name란을 Shiki에서 자신이 사용할 이름으로 변경해주면 편하다.

코랩 좌측에서 폴더 아이콘을 클릭하여 업로드된 데이터셋 파일의 경로명을 찾아 dataset_location란에 넣어준다.

해당 스텝을 실행하기 이전에 구글 드라이브의 [내 드라이브] - [diff-svc] - [singer_name] 폴더 생성, 그 안에 singer_name.zip 이름으로 압축된 데이터셋 파일을 업로드 해둔다.

*singer_name은 literally singer_name이 아닌 앞서 변경한 singer_name란이다.
<br/><br/><br/>
 

<h3> Training Options/Parameters </h3>

트레이닝 옵션을 설정하는 스텝이다.

pretrain_model_select에서는 원하는 pretrain model을 선택한다.

save_dir란에 /content/drive/Mydrive/diff-svc/singer_name 경로명을 넣어준다. 후에 학습할 때 2000번의 배수마다 학습된 모델이 저장되는 디렉토리이기도 하다. (최대 10개 저장, 이후 오래된 모델 순으로 드라이브 휴지통으로 이동된다.)

저장된 모델이 있고 그 모델의 학습을 이어서 수행하려면 resume_training_from_ckpt를 체크하고 해당 모델의 .ckpt파일 경로를 넣어준다. 다른 새로운 모델을 학습하는 것이라면 체크해제한다.

<br/><br/>
<h3> Training </h3>

학습을 진행하는 스텝이다. 앞서 적어놓았듯 2000번째마다 모델이 최대 10개까지 저장된다. 데이터셋이 길지 않다면 높은 step은 큰 효과가 없다. 1시간 남짓의 데이터셋이라면 80000 step에서 120000 step 사이 정도로 학습한 모델이 괜찮다.

 

 

모델을 이용하여 결과물을 추론하는 과정은 다른 코랩 링크에서 수행할 것이기 때문에 이후로는 따져보지 않아도 된다.

 <br/><br/>


<h1> ✔ 결과 추론 </h1>
(작성 중)
