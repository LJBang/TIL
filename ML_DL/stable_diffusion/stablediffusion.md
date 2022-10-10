# Stable diffusion
요즘 핫한 image generation분야에서 뛰어난 성능을 보이면서도 오픈소스로 공개되어 있는 stable-diffusion에 대해서 정리.  
[참고](https://jalammar.github.io/illustrated-stable-diffusion/)  

## 전체 구조
전체적으로 stable-diffusion은 여러개의 모델로 구성되는데,  
1. text embedding
2. diffusion model
3. image decoder  
위 세 가지로 구성된다고 생각할 수 있다. 순서대로 살펴보자.  

## Text Embedding
stable-dffusion의 경우 텍스트를 그림으로 바꿔주는 text2img task를 수행할 수 있다.  
이를 위해서틑 텍스트의 임베딩과 이미지의 임베딩을 동기화 시켜주어야 한다.  

학습 데이터 셋은 400m개의 이미지와 그 이미지에 맞는 캡션으로 이루어져 있다.  
모델의 경우 릴리스 버전에서 GPT를 기반으로 하는 CLIPText모델을 사용했다.  
이미지와 캡션을 각각 CLIPText모델에 넣어서 임베딩 한 후 코사인 유사도를 계산한다.  
이미지에 따른 임베딩일 경우 코사인 유사도가 1이 나오게끔 학습을 진행한다.  
물론 이미지와 캡션을 섞어 0이 되는 경우도 학습을 진행해주어야 한다.  
이 과정을 거칠 경우 텍스트 입력에 대해서 유사한 이미지 출력이 가능해진다.  

## diffusion model
대망의 diffusion 모델이다. diffusion 모델의 경우 15년쯤에 나왔지만 이제서야 각광을 받는 듯한다.  

디퓨전 모델의 컨셉은 timestep에 따라서 원본 이미지에 Noise를 단계별로 추가해 나가는 것이다.  
원본 이미지를 $I_0$, 노이즈를 $e_1$라 할 때, $I_1 = I_0 + e_1$가 노이즈가 추가된 이미지이다.  
이를 단계별로 진행해 나간다면, $I_n = I_{n-1} + e_{n}$이 되고 50번 혹은 100번 정도를 진행해 나간다.  
모델의 학습은 $I_n$ 이미지와 $n$이라는 time step 정보를 input으로, 노이즈 $e_n$을 label로 하여 진행한다.  
모델은 UNet을 사용하고 이미지는 계속해서 layer를 통과하며 UNet의 컨셉에 맞게 skip connection을 진행하고, time step정보는 layer 통과 없이 각 layer의 input에만 넣어준다.  
학습이 충분히 진행된다면 특정 단계의 이미지 $I_n$에 대한 노이즈 $e_n$을 찾을 수 있고,  
그 후 $I_n$에서 $e_n$을 뺴준다면 노이즈가 입혀지기 전 이미지인 $I_{n-1}$을 찾을 수 있게 된다.  
이 역산의 과정을 처음부터 끝까지 진행하면 충분히 노이즈를 입힌 이미지에서 태초의 원본 이미지를 찾을 수 있게 된다.  

Stable diffusion에서는 초기에 랜덤으로 초기화한 이미지 벡터를 노이즈가 낀 이미지로 보고,  
역산의 과정을 거쳐 새로운 이미지를 만들어내는 작업을 진행한다.

## Image Decoding
원본 이미지로 위 모든 과정을 진행한다면 이미지의 크기 때문에 꽤 오랜 시간이 걸릴 수 있다.  
따라서 위에서 말했던 이미지들을 적절한 latent vector로 인코딩해줄 필요가 있다.  

먼저 AutoEncoder모델을 통해서 이미지 encoder-decoder와 latent vector를 학습한다.  
디퓨전 과정에서 초기화 하는 이미지를 latent vector와 동일한 사이즈의 vector로 초기화해준다.  
충분한 noise 제거 과정을 거친 후 나오는 vector는 생성될 이미지의 latent vector로 간주, decoder를 통해서 복구해낸다.  
최종 이미지의 경우 `3x512x512`의 RGB 그림이라고 한다.  

## Text embedding과 diffusion의 결합 
입력으로 text를 받아야하기 때문에 결합해주는 과정이 필요하다.  
디퓨전 모델의 과정에 text embedding을 추가해주는데 원리는 다음과 같다.

1. Noisey image vector($I_n$)가 있으면,
2. image vector를 ResNet layer에 통과
3. ResNet output과 text embedding을 attention에 통과
4. 2, 3 반복
5. 3번 과정의 output은 다음 layer의 input으로도 사용하지만 skip connection을 위해서도 사용.  
6. 최종적으로 $e_n$을 찾아낸다.  

이렇게 학습을 진행할 경우 텍스트 입력에 대한 노이즈를 구할 수 있고, 위에서 설명한 것과 마찬가지로 노이즈를 제거해나가면서 이미지를 생성하는 방식이다.  

## 생성과정을 전체적으로 정리
1. 사용자가 텍스트를 입력한다.
2. 학습된 CLIPText 모델로 텍스트를 적절하게 임베딩 한다.
    - 이 임베딩은 이미지와 관련되도록 임베딩 되어있다.
3. 랜덤으로 이미지 latent vector를 초기화한다.
4. 이미지 벡터에서 단계적으로 노이즈를 제거한다.  
    - 노이즈 제거 과정에서 Attention layer를 통해서 텍스트 임베딩 정보를 추가해준다.  
5. 노이즈가 제거된 이미지 벡터를 Autoencoder의 Decoder로 복구해낸다.  


## 의문점 & 후기
- 어텐션 layer에서 image와 text를 모두 input으로 넣는데 어떻게 작용하는가?
    - image를 query로 key, value를 text로 사용할 것 같음...
- text embedding을 noise 추출에 사용할 뿐인데 적절한 이미지가 생성되는 것이 신기함.  