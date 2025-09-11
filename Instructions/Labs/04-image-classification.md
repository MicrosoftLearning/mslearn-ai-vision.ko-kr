---
lab:
  title: 이미지 분류
  description: Azure AI Custom Vision 서비스를 사용하여 이미지 분류 모델을 학습시킵니다.
---

# 이미지 분류

**Custom Vision** 서비스에서는 사용자 고유의 이미지로 학습되는 Computer Vision 모델을 만들 수 있습니다. 이 서비스를 사용하여 *이미지 분류* 및 *개체 감지* 모델을 학습시킨 다음 게시하여 애플리케이션에서 사용할 수 있습니다.

이 연습에서는 Custom Vision 서비스를 사용하여 이미지에서 3가지 과일 클래스(사과, 바나나, 오렌지)을 식별할 수 있는 이미지 분류 모델을 학습시킵니다.

이 연습은 Azure Custom Vision Python SDK를 기준으로 하지만 다음을 포함하여 여러 언어별 SDK를 사용하여 비전 애플리케이션을 개발할 수 있습니다. 다음과 같은 내용이 포함되어 있습니다.

* [JavaScript용 Azure Custom Vision(학습)](https://www.npmjs.com/package/@azure/cognitiveservices-customvision-training)
* [JavaScript용 Azure Custom Vision(예측)](https://www.npmjs.com/package/@azure/cognitiveservices-customvision-prediction)
* [Microsoft .NET용 Azure Custom Vision(학습)](https://www.nuget.org/packages/Microsoft.Azure.CognitiveServices.Vision.CustomVision.Training/)
* [Microsoft .NET용 Azure Custom Vision(예측)](https://www.nuget.org/packages/Microsoft.Azure.CognitiveServices.Vision.CustomVision.Prediction/)
* [Java용 Azure Custom Vision(학습)](https://search.maven.org/artifact/com.azure/azure-cognitiveservices-customvision-training/1.1.0-preview.2/jar)
* [Java용 Azure Custom Vision(예측)](https://search.maven.org/artifact/com.azure/azure-cognitiveservices-customvision-prediction/1.1.0-preview.2/jar)

이 연습에는 약 **45**분이 소요됩니다.

## Custom Vision 리소스 만들기

모델을 학습시키려면 *학습* 및 *예측*용 Azure 리소스가 필요합니다. 이러한 각 작업용으로 **Custom Vision** 리소스를 만들 수도 있고, 리소스 하나를 만든 다음, 두 작업에 사용할 수도 있습니다. 이 연습에서는 학습 및 예측을 위한 **Custom Vision** 리소스를 만듭니다.

1. `https://portal.azure.com`의 [Azure Portal](https://portal.azure.com)을 열고 Azure 자격 증명을 사용하여 로그인합니다. 표시되는 환영 메시지 또는 팁을 닫습니다.
1. **리소스 만들기**를 선택합니다.
1. 검색 창에서 검색하고 `Custom Vision`을 검색하고 **Custom Vision**을 선택하고, 다음 설정을 사용하여 리소스를 만듭니다.
    - **만들기 옵션**: 모두
    - **구독**: ‘Azure 구독’
    - **리소스 그룹**: ‘리소스 그룹 만들기 또는 선택’
    - **지역**: *사용 가능한 지역 선택*
    - **이름**: *Custom Vision 리소스의 유효한 이름*
    - **학습 가격 책정 계층**: F0
    - **예측 가격 책정 계층**: F0

1. 리소스를 만들고 배포가 완료되기를 기다린 다음, 배포 세부 정보를 봅니다. 두 개의 Custom Vision 리소스가 프로비전됩니다. 하나는 학습용이고 다른 하나는 예측용입니다.

    > **참고**: 각 리소스에는 자체 *엔드포인트* 및 *키*가 있습니다. 엔드포인트와 키는 코드에서 액세스를 관리하는 데 사용됩니다. 이미지 분류 모델을 학습시키려면 코드가 *학습* 리소스(해당 엔드포인트와 키 포함)를 사용해야 합니다. 그리고 학습된 모델을 사용하여 이미지 클래스를 예측하려면 코드가 *예측* 리소스(해당 엔드포인트와 키 포함)를 사용해야 합니다.

1. 리소스가 배포되면 리소스 그룹으로 이동하여 해당 리소스를 확인합니다. 두 개의 Custom Vision 리소스가 표시되는데, 하나에는 접미사 ***-Prediction***이 붙어 있습니다.

## Custom Vision 포털에서 Custom Vision 프로젝트 만들기

이미지 분류 모델을 학습시키려면 학습 리소스를 기반으로 Custom Vision 프로젝트를 만들어야 합니다. 이렇게 하려면 Custom Vision 포털을 사용합니다.

1. 새 브라우저 탭을 엽니다(나중에 돌아갈 예정이므로 Azure Portal 탭 열어 두기).
1. 새 브라우저 탭에서 `https://customvision.ai`의 [Custom Vision 포털](https://customvision.ai)을 엽니다. 로그인하라는 메시지가 표시되면 Azure 자격 증명을 사용하여 로그인하고 서비스 약관에 동의합니다.
1. Custom Vision 포털에서 다음 설정을 사용하여 새 프로젝트를 만듭니다.
    - **이름**: `Classify Fruit`
    - **설명**: `Image classification for fruit`
    - **리소스**: *Custom Vision 리소스*
    - 프로젝트 형식: 분류
    - **분류 형식**: 다중 클래스(이미지당 단일 태그)
    - **도메인**: 식품

### 이미지 업로드 및 태그 지정

1. 새 브라우저 탭에서 `https://github.com/MicrosoftLearning/mslearn-ai-vision/raw/main/Labfiles/image-classification/training-images.zip`의 [학습 이미지](https://github.com/MicrosoftLearning/mslearn-ai-vision/raw/main/Labfiles/image-classification/training-images.zip)를 다운로드하고 zip 폴더의 압축을 풀어 콘텐츠를 봅니다. 이 폴더에는 사과, 바나나, 오렌지 이미지의 하위 폴더가 포함되어 있습니다.
1. Custom Vision 포털의 이미지 분류 프로젝트에서 **이미지 추가**를 클릭하고 이전에 다운로드하여 압축을 푼 **training-images/apple** 폴더의 모든 파일을 선택합니다. 그런 후 다음과 같이 `apple` 태그를 지정하여 이미지 파일을 업로드합니다.

    ![apple 태그를 사용하여 사과를 업로드하는 스크린샷](../media/upload_apples.jpg)

1. **이미지 추가**(**[+]**) 도구 모음 아이콘으로 이전 단계를 반복하고 태그 `banana`를 지정하여 **banana** 폴더에 이미지를 업로드하고, 태그 `orange`를 지정하여 **orange** 폴더에 이미지를 업로드합니다.
1. Custom Vision 프로젝트에서 업로드한 이미지를 살펴봅니다. 다음과 같이 각 클래스에 15개 이미지가 있습니다.

    ![태그가 지정된 과일 이미지 - 사과 15개, 바나나 15개, 오렌지 15개](../media/fruit.jpg)

### 모델 학습

1. Custom Vision 프로젝트에서 이미지 위에 있는 **학습**(&#9881;<sub>&#9881;</sub>)을 클릭하여 태그가 지정된 이미지로 분류 모델을 학습시킵니다. **빠른 학습** 옵션을 선택한 다음, 학습 반복이 완료될 때까지 기다립니다(1분 정도 걸릴 수 있음).
1. 모델 반복을 학습시킨 후에는 정밀도, 재현율 및 *AP* 성능 메트릭을 검토합니다. 이러한 메트릭은 분류 모델의 예측 정확도를 측정하며 모두 높게 나타납니다.****

    ![모델 메트릭의 스크린샷](../media/custom-vision-metrics.png)

> **참고**: 성능 메트릭은 각 예측의 가능성 임계값인 50%를 기준으로 측정됩니다(즉, 모델에서 이미지가 특정 클래스일 가능성이 50% 이상으로 계산되면 해당 클래스가 예측으로 반환됨). 페이지 왼쪽 위에서 이 임계값을 조정할 수 있습니다.

### 모델 테스트

1. 성능 메트릭 위에 있는 **빠른 테스트**를 클릭합니다.
1. 이미지 URL** 상자에 빠른 테스트 이미지(&#10132;)* 단추를 입력 `https://aka.ms/test-apple` 하고 클릭합니다*.**
1. 모델에서 반환된 예측 보기 - 다음과 같이 *apple*의 확률 점수가 가장 높습니다.

    ![apple 클래스 예측이 있는 이미지의 스크린샷](../media/test-apple.jpg)

1. 다음 이미지를 테스트해 보세요.
    - `https://aka.ms/test-banana`
    - `https://aka.ms/test-orange`

1. **빠른 테스트** 창을 닫습니다.

### 프로젝트 설정 확인

여기서 만든 프로젝트에는 고유 식별자가 할당되었습니다. 프로젝트와 상호 작용하는 코드에서 이 식별자를 지정해야 합니다.

1. **성능** 페이지 오른쪽 위에 있는 *설정*(&#9881;) 아이콘을 클릭하여 프로젝트 설정을 표시합니다.
1. **일반**(왼쪽에 있음) 아래에서 이 프로젝트를 고유하게 식별하는 **프로젝트 ID**를 확인합니다.
1. 오른쪽의 **리소스** 아래에 키와 엔드포인트가 표시됩니다. 학습 리소스에 대한 세부 정보입니다(Azure Portal에서 리소스를 확인하여 이 정보를 얻을 수도 있음).**

## *학습* API 사용

Custom Vision 포털에서는 이미지 업로드, 태그 지정 및 모델 학습에 사용할 수 있는 편리한 사용자 인터페이스가 제공됩니다. 하지만 Custom Vision 교육 API를 사용하여 모델 학습을 자동화하려는 경우도 있습니다.

### 애플리케이션 구성 준비

1. Azure Portal이 포함된 브라우저 탭으로 돌아갑니다(나중에 돌아갈 예정이므로 Custom Vision 포털 탭 열어 두기).
1. Azure Portal에서 페이지 상단의 검색 창 오른쪽에 있는 **[\>_]** 단추를 사용하여 Azure Portal에 새 Cloud Shell을 만들고 구독에 저장소가 없는 ***PowerShell*** 환경을 선택합니다.

    Cloud Shell은 Azure Portal 하단의 창에서 명령줄 인터페이스를 제공합니다.

    > **참고**: 이전에 *Bash* 환경을 사용하는 Cloud Shell을 만든 경우 ***PowerShell***로 전환합니다.

    > **참고**: 포털에서 파일을 보관할 스토리지를 선택하라는 메시지가 표시되면 **필요한 스토리지 계정 없음**을 선택하고 사용 중인 구독을 선택한 다음, **적용**을 누릅니다.

1. Cloud Shell 도구 모음의 **설정** 메뉴에서 **클래식 버전으로 이동**을 선택합니다(코드 편집기를 사용하는 데 필요).

    **<font color="red">계속하기 전에 Cloud Shell의 클래식 버전으로 전환했는지 확인합니다.</font>**

1. 더 많은 내용을 볼 수 있도록 Cloud Shell 창의 크기를 조정합니다.

    > **팁**: 위쪽 테두리를 끌어 창의 크기를 조정할 수 있습니다. 최소화 및 최대화 단추를 사용하여 Cloud Shell과 기본 포털 인터페이스 사이를 전환할 수도 있습니다.

1. Cloud Shell 창에서 다음 명령을 입력하여 이 연습의 코드 파일이 포함된 GitHub 리포지토리를 복제합니다(명령을 입력하거나 클립보드에 복사한 다음 명령줄을 마우스 오른쪽 단추로 클릭하여 일반 텍스트로 붙여넣습니다).

    ```
    rm -r mslearn-ai-vision -f
    git clone https://github.com/MicrosoftLearning/mslearn-ai-vision
    ```

    > **팁**: CloudShell에 명령을 붙여넣을 때, 출력이 화면 버퍼의 많은 부분을 차지할 수 있습니다. `cls` 명령을 입력해 화면을 지우면 각 작업에 더 집중할 수 있습니다.

1. 리포지토리가 복제된 후 다음 명령을 사용하여 애플리케이션 코드 파일로 이동합니다.

    ```
   cd mslearn-ai-vision/Labfiles/image-classification/python/train-classifier
   ls -a -l
    ```

    폴더에는 앱용 애플리케이션 구성 및 코드 파일이 포함되어 있습니다. 또한 모델의 추가 학습을 수행하는 데 사용할 일부 이미지 파일이 포함된 **/more-training-images** 하위 폴더가 포함되어 있습니다.

1. 다음 명령을 실행하여 학습 및 기타 필수 패키지용 Azure AI Custom Vision SDK 패키지를 설치합니다.

    ```
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install -r requirements.txt azure-cognitiveservices-vision-customvision
    ```

1. 다음 명령을 입력하여 앱의 구성 파일을 편집합니다.

    ```
   code .env
    ```

    코드 편집기에서 파일이 열립니다.

1. 코드 파일에서 Custom Vision *학습* 리소스의 **엔드포인트** 및 인증 **키**, 그리고 이전에 만든 Custom Vision 프로젝트의 **프로젝트 ID**를 반영하도록 해당 파일에 포함된 구성 값을 업데이트합니다.
1. 자리 표시자를 바꾼 후 코드 편집기에서 **CTRL+S** 명령을 사용하여 변경 내용을 저장한 다음 **CTRL+Q** 명령을 사용하여 Cloud Shell 명령줄을 열어둔 채 코드 편집기를 닫습니다.

### 모델 학습을 수행하기 위한 코드 작성

1. Cloud Shell 명령줄에서 다음 명령을 입력하여 클라이언트 애플리케이션에 대한 코드 파일을 엽니다.

    ```
   code train-classifier.py
    ```

1. 코드 파일에서 다음 세부 정보를 확인합니다.
    - Azure AI Custom Vision SDK의 네임스페이스를 가져왔는지 확인합니다.
    - **Main** 함수가 구성 설정을 검색하며 키와 엔드포인트를 사용하여 인증된 클라이언트를 만듭니다.
    - **CustomVisionTrainingClient**: 프로젝트 ID와 함께 프로젝트에 대한 **Project** 참조를 만드는 데 사용됩니다.
    - **Upload_Images** 함수가 Custom Vision 프로젝트에 정의된 태그를 검색한 다음 해당 이름이 지정된 폴더의 이미지를 프로젝트에 업로드하고 적절한 태그 ID를 할당합니다.
    - **Train_Model** 함수가 프로젝트용으로 새 학습 반복을 만들고 학습이 완료될 때까지 기다립니다.

1. 코드 편집기를 닫고(*Ctrl+Q*) 다음 명령을 입력하여 프로그램을 실행합니다.

    ```
   python train-classifier.py
    ```

1. 프로그램이 종료될 때까지 기다립니다. 그런 다음, Custom Vision 포털을 포함하는 브라우저 탭으로 돌아와 프로젝트의 **학습 이미지** 페이지를 확인합니다(필요한 경우 브라우저를 새로 고침).
1. 새로 태그가 지정된 이미지 몇 개가 프로젝트에 추가되었음을 확인합니다. 그런 다음 **성능** 페이지를 표시하여 새 반복이 작성되었음을 확인합니다.

## 클라이언트 애플리케이션에서 이미지 분류자 사용

이제 학습시킨 모델을 게시하고 클라이언트 애플리케이션에서 사용할 준비가 되었습니다.

### 이미지 분류 모델 게시

1. Custom Vision 포털의 **성능** 페이지에서 **&#128504; 게시**를 클릭하여 다음 설정을 사용해 학습시킨 모델을 게시합니다.
    - **모델 이름**: `fruit-classifier`
    - **예측 리소스**: 이전에 만든 **예측** “-Prediction”으로 끝나는 리소스(학습 리소스 <u>아님</u>)**
1. **프로젝트 설정** 페이지 왼쪽 위에서 *프로젝트 설정*(&#128065;) 아이콘을 클릭하여 Custom Vision 포털 홈 페이지로 돌아옵니다. 이제 홈 페이지에 프로젝트가 나열됩니다.
1. Custom Vision 포털 홈 페이지 오른쪽 위의 *설정*(&#9881;) 아이콘을 클릭하여 Custom Vision 서비스의 설정을 확인합니다. 그런 다음 **리소스** 아래에서 “-Prediction”으로 끝나는 예측 리소스(학습 리소스 <u>아님</u>)를 찾아 **키** 및 **엔드포인트** 값을 확인합니다(Azure Portal에서 리소스를 표시해도 이 정보를 확인할 수 있음).**

### 클라이언트 애플리케이션에서 이미지 분류자 사용

1. Azure Portal과 Cloud Shell 창이 포함된 브라우저 탭으로 돌아갑니다.
1. Cloud Shell에서 다음 명령을 실행하여 클라이언트 애플리케이션에 대한 폴더로 전환하고 포함된 파일을 봅니다.

    ```
   cd ../test-classifier
   ls -a -l
    ```

    폴더에는 앱용 애플리케이션 구성 및 코드 파일이 포함되어 있습니다. 또한 모델을 테스트하는 데 사용할 일부 이미지 파일이 포함된 **/test-images** 하위 폴더가 포함되어 있습니다.

1. 다음 명령을 실행하여 예측 및 기타 필수 패키지용 Azure AI Custom Vision SDK 패키지를 설치합니다.

    ```
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install -r requirements.txt azure-cognitiveservices-vision-customvision
    ```

1. 다음 명령을 입력하여 앱의 구성 파일을 편집합니다.

    ```
   code .env
    ```

    코드 편집기에서 파일이 열립니다.

1. Custom Vision *<u>예측</u>* 리소스의 **엔드포인트** 및 **키**, 분류 프로젝트의 **프로젝트 ID** 및 게시된 모델의 이름(*fruit-classifier*)을 반영하도록 구성 값을 업데이트합니다. 변경 내용을 저장하고(*CTRL+S*), 코드 편집기를 닫습니다(*CTRL+Q*).
1. Cloud Shell 명령줄에서 다음 명령을 입력하여 클라이언트 애플리케이션에 대한 코드 파일을 엽니다.

    ```
   code test-classifier.py
    ```

1. 코드를 검토하고 다음 세부 사항을 확인합니다.
    - Azure AI Custom Vision SDK의 네임스페이스를 가져왔는지 확인합니다.
    - **Main** 함수가 구성 설정을 검색하며 키와 엔드포인트를 사용하여 인증된 **CustomVisionPredictionClient**를 만듭니다.
    - 예측 클라이언트 개체를 사용하여 각 요청에 대해 프로젝트 ID와 모델 이름을 지정해 **test-images** 폴더에 포함된 각 이미지의 클래스를 예측합니다. 각 예측에는 예측 가능한 각 클래스의 가능성이 포함되며, 가능성이 50%보다 높은 것으로 예측된 태그만 표시됩니다.

1. 코드 편집기를 닫고 다음 명령을 입력하여 프로그램을 실행합니다.

    ```
   python test-classifier.py
    ```

    프로그램은 분류를 위해 다음 각 이미지를 모델에 제출합니다.

    ![사과 이미지](../media/IMG_TEST_1.jpg)

    **IMG_TEST_1.jpg**

    <br/><br/>

    ![바나나 이미지](../media/IMG_TEST_2.jpg)

    **IMG_TEST_2.jpg**

    <br/><br/>

    ![오렌지 이미지](../media/IMG_TEST_3.jpg)

    **IMG_TEST_3.jpg**

1. 각 예측의 레이블(태그) 및 가능성 점수를 확인합니다.

## 리소스 정리

Azure AI Custom Vision 탐색을 완료한 경우 불필요한 Azure 비용이 발생하지 않도록 이 연습에서 만든 리소스를 삭제해야 합니다.

1. `https://portal.azure.com`에서 Azure Portal을 열고 상단 검색 창에서 이 랩에서 만든 리소스를 검색합니다.

1. 리소스 페이지에서 **삭제**를 선택하고 지침에 따라 리소스를 삭제합니다. 또는 전체 리소스 그룹을 삭제하여 모든 리소스를 동시에 정리할 수 있습니다.
