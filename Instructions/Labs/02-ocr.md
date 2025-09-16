---
lab:
  title: 이미지 내의 텍스트 읽기
  description: Azure AI 비전 이미지 분석 서비스에서 OCR(광학 문자 인식)을 사용하여 이미지에서 텍스트를 찾아 추출합니다.
---

# 이미지 내의 텍스트 읽기

OCR(광학 인식)은 이미지와 문서에서 텍스트 읽기를 처리하는 Computer Vision 하위 서비스입니다. **Azure AI 비전** 이미지 분석 서비스는 텍스트를 읽기 위한 API를 제공하며, 이 연습에서 살펴볼 것입니다.

> **참고**: 이 연습은 사전 출시된 SDK 소프트웨어를 기반으로 하며, 변경될 수 있습니다. 필요한 경우 특정 버전의 패키지를 사용했으며, 이는 최신 버전이 반영되지 않을 수 있습니다. 예기치 않은 동작, 경고 또는 오류가 발생할 수 있습니다.

이 연습은 Azure 비전 분석 Python SDK를 기준으로 하지만 다음을 포함하여 여러 언어별 SDK를 사용하여 비전 애플리케이션을 개발할 수 있습니다. 다음과 같은 내용이 포함되어 있습니다.

* [JavaScript용 Azure AI 비전 분석](https://www.npmjs.com/package/@azure-rest/ai-vision-image-analysis)
* [Microsoft .NET용 Azure AI 비전 분석](https://www.nuget.org/packages/Azure.AI.Vision.ImageAnalysis)
* [Java용 Azure AI 비전 분석](https://mvnrepository.com/artifact/com.azure/azure-ai-vision-imageanalysis)

이 연습에는 약 **30**분이 소요됩니다.

## Azure AI 비전 리소스 프로비전

구독에 아직 리소스가 없으면 Azure AI 비전 리소스를 프로비전해야 합니다.

> **참고**: 이 연습에서는 독립 실행형 **Computer Vision** 리소스를 사용합니다. 직접 또는 *Azure AI Foundry* 프로젝트에서 *Azure AI 서비스* 다중 서비스 리소스의 Azure AI 비전 서비스를 사용할 수도 있습니다.

1. `https://portal.azure.com`의 [Azure Portal](https://portal.azure.com)을 열고 Azure 자격 증명을 사용하여 로그인합니다. 표시되는 환영 메시지 또는 팁을 닫습니다.
1. **리소스 만들기**를 선택합니다.
1. 검색 창에서 검색하고 `Computer Vision`을 검색하고 **Computer Vision**을 선택하고, 다음 설정을 사용하여 리소스를 만듭니다.
    - **구독**: ‘Azure 구독’
    - **리소스 그룹**: ‘리소스 그룹 만들기 또는 선택’
    - **지역**: ***미국 동부**, **미국 서부**, **프랑스 중부**, **한국 중부**, **북유럽**, **동남 아시아**, **서유럽** 또는 **동아시아**\** 중에서 선택
    - **이름**: *Computer Vision 리소스의 유효한 이름*
    - 가격 책정 계층: 무료 F0

    \*Azure AI 비전 4.0 전체 기능 세트는 현재 이 지역에서만 사용할 수 있습니다.

1. 필요한 확인란을 선택하고 리소스를 만듭니다.
1. 배포가 완료될 때까지 기다린 다음, 배포 세부 정보를 봅니다.
1. 리소스가 배포되면 리소스로 이동하고 탐색 창의 **리소스 관리** 노드 아래에서 해당 **키 및 엔드포인트** 페이지를 확인합니다. 다음 절차에서 이 페이지에 표시되는 키 중 하나와 엔드포인트가 필요합니다.

## Azure AI 비전 SDK를 사용하여 텍스트 추출 앱 개발

이 연습에서는 Azure AI 비전 SDK를 사용하여 이미지에서 텍스트를 추출하는 부분 구현 클라이언트 애플리케이션을 완료합니다.

### 애플리케이션 구성 준비

1. Azure Portal에서 페이지 상단의 검색 창 오른쪽에 있는 **[\>_]** 단추를 사용하여 Azure Portal에 새 Cloud Shell을 만들고 구독에 저장소가 없는 ***PowerShell*** 환경을 선택합니다.

    Cloud Shell은 Azure Portal 하단의 창에서 명령줄 인터페이스를 제공합니다.

    > **참고**: 이전에 *Bash* 환경을 사용하는 Cloud Shell을 만든 경우 ***PowerShell***로 전환합니다.

    > **참고**: 포털에서 파일을 보관할 스토리지를 선택하라는 메시지가 표시되면 **필요한 스토리지 계정 없음**을 선택하고 사용 중인 구독을 선택한 다음, **적용**을 누릅니다.

1. Cloud Shell 도구 모음의 **설정** 메뉴에서 **클래식 버전으로 이동**을 선택합니다(코드 편집기를 사용하는 데 필요).

    **<font color="red">계속하기 전에 Cloud Shell의 클래식 버전으로 전환했는지 확인합니다.</font>**

1. Computer Vision 리소스의 **키 및 엔드포인트** 페이지를 계속 볼 수 있도록 Cloud Shell 창의 크기를 조정합니다.

    > **팁**: 위쪽 테두리를 끌어 창의 크기를 조정할 수 있습니다. 최소화 및 최대화 단추를 사용하여 Cloud Shell과 기본 포털 인터페이스 사이를 전환할 수도 있습니다.

1. Cloud Shell 창에서 다음 명령을 입력하여 이 연습의 코드 파일이 포함된 GitHub 리포지토리를 복제합니다(명령을 입력하거나 클립보드에 복사한 다음 명령줄을 마우스 오른쪽 단추로 클릭하여 일반 텍스트로 붙여넣습니다).

    ```
    rm -r mslearn-ai-vision -f
    git clone https://github.com/MicrosoftLearning/mslearn-ai-vision
    ```

    > **팁**: CloudShell에 명령을 붙여넣으면 출력이 화면 버퍼의 많은 부분을 차지할 수 있습니다. `cls` 명령을 입력해 화면을 지우면 각 작업에 더 집중할 수 있습니다.

1. 리포지토리가 복제된 후 다음 명령을 사용하여 애플리케이션 코드 파일로 이동합니다.

    ```
   cd mslearn-ai-vision/Labfiles/ocr/python/read-text
   ls -a -l
    ```

    폴더에는 앱용 애플리케이션 구성 및 코드 파일이 포함되어 있습니다. 또한 앱에서 분석할 이미지 파일이 포함된 **/images** 하위 폴더도 포함되어 있습니다.

1. 다음 명령을 실행하여 Azure AI 비전 SDK 패키지 및 기타 필수 패키지를 설치합니다.

    ```
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install -r requirements.txt azure-ai-vision-imageanalysis==1.0.0
    ```

1. 다음 명령을 입력하여 앱의 구성 파일을 편집합니다.

    ```
   code .env
    ```

    코드 편집기에서 파일이 열립니다.

1. 코드 파일에서 Computer Vision 리소스에 대한 **엔드포인트** 및 인증 **키**(Azure Portal의 **키 및 엔드포인트** 페이지에서 복사)를 반영하도록 해당 파일이 포함하는 구성 값을 업데이트합니다.
1. 자리 표시자를 바꾼 후 **Ctrl+S** 명령을 사용하여 변경 내용을 저장한 다음 **Ctrl+Q** 명령을 사용하여 Cloud Shell 명령줄을 열어 두고 코드 편집기를 닫습니다.

### 이미지에서 텍스트를 읽기 위한 코드 추가

1. Cloud Shell 명령줄에서 다음 명령을 입력하여 클라이언트 애플리케이션에 대한 코드 파일을 엽니다.

    ```
   code read-text.py
    ```

    > **팁**: 코드를 더 쉽게 볼 수 있도록 Cloud Shell 창을 최대화하고 명령줄 콘솔과 코드 편집기 사이에서 분할 막대를 이동할 수 있습니다.

1. 코드 파일에서 **Import namespaces** 주석을 찾고 다음 코드를 추가하여 Azure AI 비전 SDK를 사용하는 데 필요한 네임스페이스를 가져옵니다.

    ```python
   # import namespaces
   from azure.ai.vision.imageanalysis import ImageAnalysisClient
   from azure.ai.vision.imageanalysis.models import VisualFeatures
   from azure.core.credentials import AzureKeyCredential
    ```

1. **Main** 함수에는 구성 설정을 로드하고 분석할 파일을 확인하는 코드가 제공되었습니다. 그런 다음, **Authenticate Azure AI Vision client** 주석을 찾고 다음 언어별 코드를 추가하여 Azure AI 비전 이미지 분석 클라이언트 개체를 만들고 인증합니다.

    ```python
   # Authenticate Azure AI Vision client
   cv_client = ImageAnalysisClient(
        endpoint=ai_endpoint,
        credential=AzureKeyCredential(ai_key))
    ```

1. **Main** 함수의 방금 추가한 코드 아래에서 **Read text in image** 주석을 찾고 다음 코드를 추가하여 이미지 분석 클라이언트를 통해 이미지의 텍스트를 읽습니다.

    ```python
   # Read text in image
   with open(image_file, "rb") as f:
        image_data = f.read()
   print (f"\nReading text in {image_file}")

   result = cv_client.analyze(
        image_data=image_data,
        visual_features=[VisualFeatures.READ])
    ```

1. **Print the text** 주석을 찾고 다음 코드(마지막 주석 포함)를 추가하여 찾은 텍스트 줄을 인쇄하고, 이미지에서 해당 텍스트 줄에 주석을 추가하는 함수를 호출합니다(각 텍스트 줄에 대해 반환된 **bounding_polygon** 사용).

    ```python
   # Print the text
   if result.read is not None:
        print("\nText:")
    
        for line in result.read.blocks[0].lines:
            print(f" {line.text}")        
        # Annotate the text in the image
        annotate_lines(image_file, result.read)

        # Find individual words in each line
        
    ```

1. 변경 내용을 저장하고(*Ctrl+S*), 입력 오류를 수정해야 하는 경우에는 코드 편집기를 열어 둡니다.

1. 콘솔을 더 많이 볼 수 있도록 창의 크기를 조정하고 다음 명령을 입력하여 프로그램을 실행합니다.

    ```
   python read-text.py images/Lincoln.jpg
    ```

1. 프로그램은 다음과 같이 지정된 이미지 파일(*images/Lincoln.jpg*)의 텍스트를 읽습니다.

    ![에이브러햄 링컨 동상 사진.](../media/Lincoln.jpg)

1. **read-text** 폴더에 **lines.jpg** 이미지가 생성되었습니다. (Azure Cloud Shell 관련) **download** 명령을 사용하여 다운로드합니다.

    ```
   download lines.jpg
    ```

    다운로드 명령은 브라우저의 오른쪽 아래에 팝업 링크를 만듭니다. 이 링크를 선택하여 파일을 다운로드하고 열 수 있습니다. 이 이미지와 다음과 유사하게 표시됩니다.

    ![텍스트가 강조 표시된 이미지.](../media/text.jpg)

1. 프로그램을 다시 실행하는데, 이번에는 매개 변수 *images/Business-card.jpg*를 지정하여 다음 이미지에서 텍스트를 추출합니다.

    ![스캔한 명함 이미지.](../media/Business-card.jpg)

    ```
   python read-text.py images/Business-card.jpg
    ```

1. 결과 **lines.jpg** 파일을 다운로드하고 확인합니다.

    ```
   download lines.jpg
    ```

1. 프로그램을 한 번 더 실행하는데, 이번에는 *images/Note.jpg* 매개 변수를 지정하여 이 이미지에서 텍스트를 추출합니다.

    ![손으로 쓴 쇼핑 목록 사진.](../media/Note.jpg)

    ```
   python read-text.py images/Note.jpg
    ```

1. 결과 **lines.jpg** 파일을 다운로드하고 확인합니다.

    ```
   download lines.jpg
    ```

### 개별 단어의 위치를 반환하기 위한 코드 추가

1. 더 많은 코드 파일을 볼 수 있도록 창의 크기를 조정합니다. 그런 다음, **Find individual words in each line** 주석을 찾고 다음 코드를 추가합니다(주의해서 올바른 들여쓰기 수준 유지).

    ```python
   # Find individual words in each line
   print ("\nIndividual words:")
   for line in result.read.blocks[0].lines:
        for word in line.words:
            print(f"  {word.text} (Confidence: {word.confidence:.2f}%)")
   # Annotate the words in the image
   annotate_words(image_file, result.read)
    ```

1. 변경 내용을 저장합니다(*CTRL+S*). 그런 다음, 명령줄 창에서 프로그램을 다시 실행하여 *images/Lincoln.jpg*에서 텍스트를 추출합니다.
1. 이미지의 각 개별 단어와 예측과 관련된 신뢰도를 포함하는 출력을 확인합니다.
1. **read-text** 폴더에 **words.jpg** 이미지가 생성되었습니다. (Azure Cloud Shell 관련) **download** 명령을 사용하여 다운로드한 후 확인합니다.

    ```
   download words.jpg
    ```

1. *images/Business-card.jpg* 및 *images/Note.jpg* 이미지에 대해 이 프로그램을 다시 실행하고 각 이미지에 대해 생성된 **words.jpg** 파일을 확인합니다.

## 리소스 정리

Azure AI 비전 탐색을 완료한 경우 불필요한 Azure 비용이 발생하지 않도록 이 연습에서 만든 리소스를 삭제해야 합니다.

1. `https://portal.azure.com`의 Azure Portal을 열고 Azure 구독과 연관된 Microsoft 계정을 사용하여 로그인합니다.

1. 위쪽 검색 창에서 *Computer Vision*을 검색하고 이 랩에서 만든 Computer Vision 리소스를 선택합니다.

1. 리소스 페이지에서 **삭제**를 선택하고 지침에 따라 리소스를 삭제합니다.
