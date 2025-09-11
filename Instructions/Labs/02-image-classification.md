---
lab:
  title: Azure AI 비전 사용자 지정 모델을 사용하여 이미지 분류
  module: Module 2 - Develop computer vision solutions with Azure AI Vision
---

# Azure AI 비전 사용자 지정 모델을 사용하여 이미지 분류

Azure AI 비전을 사용하면 사용자 지정 모델을 학습하여 지정한 레이블로 개체를 분류하고 검색할 수 있습니다. 이 랩에서는 과일 이미지를 분류하기 위한 사용자 지정 이미지 분류 모델을 빌드해 보겠습니다.

## Computer Vision 리소스 프로비전저닝

구독에 아직 없는 경우 **Computer Vision** 리소스를 프로비전해야 합니다.

1. `https://portal.azure.com`의 Azure Portal을 열고 Azure 구독과 연관된 Microsoft 계정을 사용하여 로그인합니다.
1. **리소스 만들기**를 선택합니다.
1. 검색 창에서 *Computer Vision*을 검색하고, **Computer Vision**을 선택한 후 다음 설정을 사용하여 리소스를 만듭니다.
    - **구독**: ‘Azure 구독’
    - **리소스 그룹**: 리소스 그룹 선택 또는 만들기(제한된 구독을 사용 중이라면 새 리소스 그룹을 만들 권한이 없을 수도 있으므로 제공된 리소스 그룹 사용)**
    - **지역**: *미국 동부, 미국 서부, 프랑스 중부, 한국 중부, 북유럽, 동남 아시아, 서유럽 또는 동아시아 중에서 선택\**
    - **이름**: *고유 이름 입력*
    - 가격 책정 계층: 무료 F0

    \*Azure AI 비전 4.0 전체 기능 세트는 현재 이 지역에서만 사용할 수 있습니다.

1. 필요한 확인란을 선택하고 리소스를 만듭니다.
<!--4. When the resource has been deployed, go to it and view its **Keys and Endpoint** page. You will need the endpoint and one of the keys from this page in a future step. Save them off or leave this browser tab open.-->

학습 이미지를 저장하려면 스토리지 계정도 필요합니다.

1. Azure Portal에서 **스토리지 계정**을 검색하여 선택하고 다음 설정을 사용하여 새 스토리지 계정을 만듭니다.
    - **구독**: ‘Azure 구독’
    - **리소스 그룹**: *Custom Vision 리소스를 만든 것과 동일한 리소스 그룹을 선택합니다.*
    - **스토리지 계정 이름**: customclassifySUFFIX 
        - *참고: 리소스 이름이 전역적으로 고유하도록 `SUFFIX` 토큰을 이니셜이나 다른 값으로 바꿉니다.*
    - **지역**: *Azure AI Service 리소스에 사용한 것과 동일한 지역을 선택합니다.*
    - **기본 서비스**: Azure Blob Storage 또는 Azure Data Lake Storage Gen 2
    - **기본 워크로드**: 기타
    - **성능**: 표준
    - **중복도**: LRS(로컬 중복 스토리지)

1. 리소스를 배포했으면 **리소스로 이동**을 선택합니다.
1. 스토리지 계정에 대한 공용 액세스를 사용하도록 설정합니다. 왼쪽 창에서 **설정** 그룹의 **구성**으로 이동하여 *Blob 익명 액세스 허용*을 사용하도록 설정합니다. **저장**을 선택합니다.
1. 왼쪽 창의 **데이터 스토리지**에서 **컨테이너**를 선택하고 `fruit`라는 이름의 컨테이너를 새로 만든 다음 **익명 액세스 수준**을 *컨테이너(컨테이너 및 블롭에 대한 익명 읽기 권한)* 로 설정합니다.

    > **참고**: **익명 액세스 수준**이 사용하지 않도록 설정된 경우 브라우저 페이지를 새로 고침합니다.
   
## 이 과정용 리포지토리 복제

모델을 학습시키기 위한 이미지 파일이 GitHub 리포지토리에 제공되었습니다. Azure Portal에서 Cloud Shell을 사용하여 리포지토리를 복제하고 스토리지 계정에 이미지를 업로드합니다. 

> **팁**: 최근에 **mslearn-ai-vision** 리포지토리를 이미 복제한 경우 복제 작업을 건너뛰어도 됩니다. 그렇지 않은 경우에는 다음 단계에 따라 개발 환경에 리포지토리를 복제합니다.

1. Azure Portal에서 페이지 상단의 검색 창 오른쪽에 있는 **[\>_]** 버튼으로 Azure Portal에서 새 Cloud Shell을 생성하고 ***PowerShell*** 환경을 선택합니다. Cloud Shell은 다음과 같이 Azure Portal 아래쪽 창에 명령줄 인터페이스를 제공합니다.

    > **참고**: 이전에 *Bash* 환경을 사용하는 Cloud Shell을 만든 경우 ***PowerShell***로 전환합니다.

1. Cloud Shell 도구 모음의 **설정** 메뉴에서 **클래식 버전으로 이동**을 선택합니다(코드 편집기를 사용하는 데 필요).

    > **팁**: CloudShell에 명령을 붙여넣을 때, 출력이 화면 버퍼의 많은 부분을 차지할 수 있습니다. `cls` 명령을 입력해 화면을 지우면 각 작업에 더 집중할 수 있습니다.

1. PowerShell 창에서 다음 명령을 입력하여 이 연습이 포함된 GitHub 리포지토리를 복제합니다.

    ```
    rm -r mslearn-ai-vision -f
    git clone https://github.com/microsoftlearning/mslearn-ai-vision mslearn-ai-vision
    ```

1. 리포지토리가 복제된 후 연습 파일이 들어 있는 폴더로 이동합니다.  

    ```
   cd mslearn-ai-vision/Labfiles/02-image-classification
    ```

1. `code replace.ps1` 명령을 실행하고 코드를 검토합니다. 이후 단계에서 사용하는 JSON 파일(COCO 파일)의 자리 표시자에 대한 스토리지 계정 이름이 바뀌는 것을 확인할 수 있습니다.
1. 파일의 *첫 번째 줄에만* 자리 표시자를 스토리지 계정 이름으로 바꿉니다.
1. 자리 표시자를 바꾼 후 코드 편집기에서 **CTRL+S** 명령 또는 **마우스 오른쪽 단추 클릭 > 저장**을 사용하여 변경 내용을 저장한 다음, **CTRL+Q** 명령 또는 **마우스 오른쪽 단추 클릭 > 끝내기**를 사용하여 Cloud Shell 명령줄을 열어둔 채 코드 편집기를 닫습니다.
1. 다음 명령을 사용하여 스크립트를 실행합니다.

    ```powershell
    ./replace.ps1
    ```

1. COCO 파일을 검토하여 스토리지 계정 이름이 있는지 확인할 수 있습니다. `code training-images/training_labels.json`을 실행하고 처음 몇 개의 항목을 확인합니다. *absolute_url* 필드에 *"https://myStorage.blob.core.windows.net/fruit/...* 과 유사한 내용이 표시되어야 합니다. 예상된 변경 내용이 표시되지 않으면 PowerShell 스크립트에서 첫 번째 자리 표시자만 업데이트했는지 확인합니다.
1. 코드 편집기를 닫습니다.
1. `<your-storage-account>`를 스토리지 계정의 이름으로 바꾼 상태로 다음 명령을 실행하여 **training-images** 폴더의 내용을 이전에 만든 `fruit` 컨테이너에 업로드합니다.

    ```powershell
    az storage blob upload-batch --account-name <your-storage-account> -d fruit -s ./training-images/
    ```

1. `fruit` 컨테이너를 열고 파일이 올바르게 업로드되었는지 확인합니다.

## 사용자 지정 모델 학습 프로젝트 만들기

다음으로 Vision Studio에서 사용자 지정 이미지 분류를 위한 새 학습 프로젝트를 만듭니다.

1. 웹 브라우저에서 `https://portal.vision.cognitive.azure.com/`으로 이동하여 Azure AI 리소스를 만든 Microsoft 계정으로 로그인합니다.
1. **이미지로 모델 사용자 지정** 타일(기본 보기에 표시되지 않는 경우 **이미지 분석** 탭에서 찾을 수 있음)을 선택합니다.
1. 생성한 Azure AI 서비스 계정을 선택합니다.
1. 프로젝트 상단에서 **새 데이터 세트 추가**를 선택합니다. 다음 설정을 사용하여 구성합니다.
    - **데이터 세트 이름**: training_images
    - **모델 형식**: 이미지 분류
    - **Azure Blob Storage 컨테이너 선택**: **컨테이너 선택**을 선택합니다.
        - **구독**: ‘Azure 구독’
        - **스토리지 계정**: *만든 스토리지 계정*
        - **Blob 컨테이너**: 과일
    - "Vision Studio가 Blob Storage를 읽고 쓸 수 있도록 허용" 상자를 선택합니다.
1. **training_images** 데이터 세트를 선택합니다.

프로젝트 만들기의 이 시점에서는 일반적으로 **Azure ML 데이터 레이블 지정 프로젝트 만들기**를 선택하고 이미지에 레이블을 지정하여 COCO 파일을 만듭니다. 시간이 있으면 이 방법을 시도해 보시기 바랍니다. 하지만 이 랩의 목적을 위해 우리는 이미 이미지에 레이블을 지정하고 결과 COCO 파일을 제공했습니다.

1. **COCO 파일 추가**를 선택합니다.
1. 드롭다운에서 **Blob 컨테이너에서 COCO 파일 가져오기**를 선택합니다.
1. `fruit`라는 컨테이너를 이미 연결했으므로 Vision Studio는 해당 컨테이너에서 COCO 파일을 검색합니다. 드롭다운에서 **training_labels.json**을 선택하고 COCO 파일을 추가합니다.
1. 왼쪽의 **사용자 지정 모델**로 이동하여 **새 모델 학습**을 선택합니다. 다음 설정을 사용합니다.
    - **모델 이름**: classifyfruit
    - **모델 형식**: 이미지 분류
    - **학습 데이터 세트 선택**: training_images
    - 나머지는 기본값으로 두고 **모델 학습**을 선택합니다.

학습에는 다소 시간이 걸릴 수 있습니다. 기본 예산은 최대 1시간이지만 이 작은 데이터 세트의 경우 일반적으로 그보다 훨씬 빠릅니다. 작업 상태가 *성공*이 될 때까지 몇 분마다 **새로 고침** 단추를 선택합니다. 모델을 선택합니다.

여기에서 학습 작업의 성과를 볼 수 있습니다. 학습된 모델의 정밀도와 정확도를 검토합니다.

## 사용자 지정 모델 테스트

모델이 학습되어 테스트할 준비가 되었습니다.

1. 사용자 지정 모델 페이지 상단에서 **시도**를 선택합니다.
1. 사용할 모델을 지정하는 드롭다운에서 **classifyfruit** 모델을 선택하고 **02-image-classification\test-images** 폴더를 찾습니다.
1. 각 이미지를 선택하고 결과를 확인합니다. 전체 JSON 응답을 검사하려면 결과 상자에서 **JSON** 탭을 선택합니다.

<!-- Option coding example to run-->
## 리소스 정리

다른 학습 모듈을 위해 이 랩에서 만들어진 Azure 리소스를 사용하지 않는 경우 해당 리소스를 삭제하여 추가 요금이 발생하지 않도록 할 수 있습니다.

1. `https://portal.azure.com`에서 Azure Portal을 열고 상단 검색 창에서 이 랩에서 만든 리소스를 검색합니다.

2. 리소스 페이지에서 **삭제**를 선택하고 지침에 따라 리소스를 삭제합니다. 또는 전체 리소스 그룹을 삭제하여 모든 리소스를 동시에 정리할 수 있습니다.
