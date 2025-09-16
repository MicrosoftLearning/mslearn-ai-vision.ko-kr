---
lab:
  title: AI를 사용하여 이미지 생성
  description: Azure AI Foundry에서 OpenAI A DALL-E 모델을 사용하여 이미지를 생성합니다.
---

# AI를 사용하여 이미지 생성

이 연습에서는 OpenAI DALL-E 생성형 AI 모델을 사용하여 이미지를 생성합니다. 또한 OpenAI Python SDK를 사용하여 프롬프트에 따라 이미지를 생성하는 간단한 앱을 만듭니다.

> **참고**: 이 연습은 사전 출시된 SDK 소프트웨어를 기반으로 하며, 변경될 수 있습니다. 필요한 경우 특정 버전의 패키지를 사용했으며, 이는 최신 버전이 반영되지 않을 수 있습니다. 예기치 않은 동작, 경고 또는 오류가 발생할 수 있습니다.

이 연습은 OpenAI Python SDK를 기준으로 하지만 다음을 포함하여 여러 언어별 SDK를 사용하여 AI 채팅 애플리케이션을 개발할 수 있습니다. 포함된 내용은 다음과 같습니다.

* [Microsoft .NET용 OpenAI 프로젝트](https://www.nuget.org/packages/OpenAI)
* [JavaScript용 OpenAI 프로젝트](https://www.npmjs.com/package/openai)

이 연습에는 약 **30**분이 소요됩니다.

## Azure AI Foundry 포털 열기

Azure AI 파운드리 포털에 로그인하여 시작해 보겠습니다.

1. 웹 브라우저에서 [Azure AI 파운드리 포털](https://ai.azure.com)(`https://ai.azure.com`)을 열고 Azure 자격 증명을 사용하여 로그인합니다. 처음 로그인할 때 열리는 팁이나 빠른 시작 창을 닫고, 필요한 경우 왼쪽 위에 있는 **Azure AI 파운드리** 로고를 사용하여 다음 이미지와 유사한 홈페이지로 이동합니다(**도움말** 창이 열려 있는 경우 닫습니다).

    ![Azure AI Foundry 포털의 스크린샷.](./media/ai-foundry-home.png)

1. 홈페이지의 정보를 리뷰합니다.

## 프로젝트를 시작할 모델 선택

Azure AI *프로젝트*는 AI 개발을 위한 공동 작업 영역을 제공합니다. 작업할 모델을 선택하고 이를 사용할 프로젝트를 만드는 것부터 시작하겠습니다.

> **참고**: AI 파운드리 프로젝트는 AI 에이전트 및 채팅 솔루션 개발을 위한 AI 모델(Azure OpenAI 포함), Azure AI 서비스 및 기타 리소스에 대한 액세스를 제공하는 *Azure AI 파운드리* 리소스를 기반으로 할 수 있습니다. 또는 프로젝트는 보안 스토리지, 컴퓨팅 및 특수 도구에 대한 Azure 리소스 연결을 포함하는 *AI 허브* 리소스를 기반으로 할 수 있습니다. Azure AI 파운드리 기반 프로젝트는 AI 에이전트 또는 채팅 앱 개발을 위한 리소스를 관리하려는 개발자에게 적합합니다. AI 허브 기반 프로젝트는 복잡한 AI 솔루션을 개발하는 엔터프라이즈 개발 팀에 더 적합합니다.

1. 홈페이지의 **모델 및 기능 탐색** 섹션에서 프로젝트에 사용할 `dall-e-3` 모델을 검색합니다.

1. 검색 결과에서 **dall-e-3** 모델을 선택하여 세부 정보를 확인한 다음, 해당 모델 페이지 상단에서 **이 모델 사용**을 선택합니다.

1. 프로젝트를 만들라는 메시지가 표시되면 프로젝트의 유효한 이름을 입력하고 **고급 옵션**을 펼칩니다.

1. **사용자 지정**을 선택하고 허브에 대해 다음 설정을 지정합니다.
    - **Azure AI 파운드리 리소스**: *Azure AI 파운드리 리소스의 유효한 이름*
    - **구독**: ‘Azure 구독’
    - **리소스 그룹**: ‘리소스 그룹 만들기 또는 선택’
    - **지역**: ***AI Foundry 권장 사항 선택***\*

    > \* 일부 Azure AI 리소스는 지역 모델 할당량에 의해 제한됩니다. 연습 후반부에 할당량 한도를 초과하는 경우 다른 지역에서 다른 리소스를 만들어야 할 수도 있습니다.

1. **만들기**를 선택하고 선택한 dall-e-3 모델 배포를 포함하여 프로젝트가 만들어질 때까지 기다립니다.

    > 참고: 모델 선택에 따라 프로젝트 만들기 프로세스 중에 추가 프롬프트가 표시될 수 있습니다. 조건에 동의하고 배포를 완료합니다.

1. 프로젝트가 만들어지면 모델이 **모델 + 엔드포인트** 페이지에 표시됩니다.

## 플레이그라운드의 모델 테스트

클라이언트 애플리케이션을 만들기 전에 플레이그라운드에서 DALL-E 모델을 테스트해 보겠습니다.

1. **플레이그라운드**를 선택한 다음, **이미지 플레이그라운드**를 선택합니다.

1. DALL-E 모델 배포가 선택되어 있는지 확인합니다. 그런 다음, 페이지 아래쪽에 있는 상자에 `Create an image of an robot eating spaghetti`와 같은 프롬프트를 입력하고 **생성**을 선택합니다.

1. 플레이그라운드에서 결과 이미지를 검토합니다.

    ![생성된 이미지가 있는 이미지 플레이그라운드의 스크린샷.](../media/images-playground.png)

1. 후속 프롬프트(예:`Show the robot in a restaurant`)를 입력하고 결과 이미지를 검토합니다.

1. 새 프롬프트를 사용하여 테스트를 계속하여 만족할 때까지 이미지를 구체화합니다. 

1. **\</\> 코드 보기** 단추를 선택하고 **Entra ID 인증** 탭에 있는지 확인합니다. 그런 다음, 연습의 뒷부분에서 사용할 예정이므로 다음 정보를 기록합니다. 값은 예제일 뿐입니다. 사용하는 배포의 정보를 기록해야 합니다.

    * OpenAI 엔드포인트: *https://dall-e-aus-resource.cognitiveservices.azure.com/*
    * OpenAI API 버전: *2024-04-01-preview*
    * 배포 이름(모델 이름): *dall-e-3*

## 클라이언트 애플리케이션 생성하기

플레이그라운드에서 일하는 것 같은 모델입니다. 이제 OpenAI SDK를 사용하여 클라이언트 애플리케이션에서 사용할 수 있습니다.

### 애플리케이션 구성 준비

1. 새 브라우저 탭을 엽니다(Azure AI 파운드리 포털을 기존 탭에서 열어 두기). 그런 다음 새 탭에서 [Azure Portal](https://portal.azure.com)(`https://portal.azure.com`)을 열고 메시지가 나타나면 Azure 자격 증명을 사용하여 로그인합니다.

1. 페이지 상단의 검색 창 오른쪽에 있는 **[\>_]** 단추를 사용하여 Azure Portal에서 새 Cloud Shell을 만들고 ***PowerShell*** 환경을 선택합니다. Cloud Shell은 다음과 같이 Azure Portal 아래쪽 창에 명령줄 인터페이스를 제공합니다.

    > **참고**: 이전에 *Bash* 환경을 사용하는 Cloud Shell을 만든 경우 ***PowerShell***로 전환합니다.

    > **참고**: 포털에서 파일을 보관할 스토리지를 선택하라는 메시지가 표시되면 **필요한 스토리지 계정 없음**을 선택하고 사용 중인 구독을 선택한 다음, **적용**을 누릅니다.

1. Cloud Shell 도구 모음의 **설정** 메뉴에서 **클래식 버전으로 이동**을 선택합니다(코드 편집기를 사용하는 데 필요).

    **<font color="red">계속하기 전에 Cloud Shell의 클래식 버전으로 전환했는지 확인합니다.</font>**

1. Cloud Shell 창에서 다음 명령을 입력하여 이 연습의 코드 파일이 포함된 GitHub 리포지토리를 복제합니다(명령을 입력하거나 클립보드에 복사한 다음 명령줄을 마우스 오른쪽 단추로 클릭하여 일반 텍스트로 붙여넣습니다).

    ```
    rm -r mslearn-ai-vision -f
    git clone https://github.com/MicrosoftLearning/mslearn-ai-vision
    ```

    > **팁**: CloudShell에 명령을 붙여넣을 때, 출력이 화면 버퍼의 많은 부분을 차지할 수 있습니다. `cls` 명령을 입력해 화면을 지우면 각 작업에 더 집중할 수 있습니다.

1. 리포지토리가 복제된 후 애플리케이션 코드 파일이 포함된 폴더로 이동합니다.  

    ```
   cd mslearn-ai-vision/Labfiles/dalle-client/python
    ```

1. Cloud Shell 명령줄 창에서 다음 명령을 입력하여 사용할 라이브러리를 설치합니다.

    ```
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install -r requirements.txt azure-identity openai requests
    ```

1. 제공된 구성 파일을 편집하려면 다음 명령을 입력합니다.

    ```
   code .env
    ```

    코드 편집기에서 파일이 열립니다.

1. **your_endpoint**, **your_model_deployment** 및 **your_api_version** 자리 표시자를 **이미지 플레이그라운드**에서 기록한 값으로 바꿉니다.

1. 자리 표시자를 바꾼 후 **Ctrl+S** 명령을 사용하여 변경 내용을 저장한 다음 **Ctrl+Q** 명령을 사용하여 Cloud Shell 명령줄을 열어 두고 코드 편집기를 닫습니다.

### 프로젝트에 연결하고 모델과 채팅하는 코드를 작성합니다.

> **팁**: 코드를 추가할 때 올바른 들여쓰기를 유지해야 합니다.

1. 제공된 코드 파일을 편집하려면 다음 명령을 입력합니다.

    ```
   code dalle-client.py
    ```

1. 코드 파일에서 파일 상단에 추가된 기존 문에 주목하여 필요한 SDK 네임스페이스를 가져옵니다. 그런 다음 **참조 추가** 주석 아래에 다음 코드를 추가하여 이전에 설치한 라이브러리의 네임스페이스를 참조합니다.

    ```python
   # Add references
   from dotenv import load_dotenv
   from azure.identity import DefaultAzureCredential, get_bearer_token_provider
   from openai import AzureOpenAI
   import requests
    ```

1. **main** 함수의 **Get configuration settings** 주석 아래에서 해당 코드가 구성 파일에서 정의한 프로젝트 연결 문자열 및 모델 배포 이름 값을 로드한다는 사실을 확인합니다.

1. **Initialize the client** 주석 아래에 다음 코드를 추가하여 현재 로그인한 Azure 자격 증명을 사용하여 모델에 연결합니다.

    ```python
   # Initialize the client
   token_provider = get_bearer_token_provider(
       DefaultAzureCredential(exclude_environment_credential=True,
           exclude_managed_identity_credential=True), 
       "https://cognitiveservices.azure.com/.default"
   )
    
   client = AzureOpenAI(
       api_version=api_version,
       azure_endpoint=endpoint,
       azure_ad_token_provider=token_provider
   )
    ```

1. 코드에는 사용자가 "종료"를 입력할 때까지 프롬프트를 입력할 수 있도록 하는 루프가 포함되어 있습니다. 그런 다음 루프 섹션의 **이미지 생성하기** 주석 아래에 다음 코드를 추가하여 프롬프트를 제출하고 모델에서 생성된 이미지 URL을 검색합니다.

    **Python**

    ```python
   # Generate an image
   result = client.images.generate(
        model=model_deployment,
        prompt=input_text,
        n=1
    )

   json_response = json.loads(result.model_dump_json())
   image_url = json_response["data"][0]["url"] 
    ```

1. **주** 함수의 나머지 부분에 있는 코드는 이미지 URL과 파일 이름을 제공된 함수에 전달하여 생성된 이미지를 다운로드하고 .png 파일로 저장합니다.

1. **Ctrl+S** 명령을 사용하여 변경 내용을 코드 파일에 저장한 다음 **Ctrl+Q** 명령을 사용하여 Cloud Shell 명령줄을 열어 둔 채 코드 편집기를 닫습니다.

### 클라이언트 애플리케이션 실행

1. Cloud Shell 명령줄 창에서 다음 명령을 입력하여 Azure에 로그인합니다.

    ```
   az login
    ```

    **<font color="red">Cloud Shell 세션이 이미 인증되었더라도 Azure에 로그인해야 합니다.</font>**

    > **참고**: 대부분의 시나리오에서는 *az login*을 사용하는 것만으로도 충분합니다. 그러나 여러 테넌트에 구독이 있는 경우 *--tenant* 매개 변수를 사용하여 테넌트 지정해야 할 수 있습니다. 자세한 내용은 [Sign into Azure interactively using the Azure CLI](https://learn.microsoft.com/cli/azure/authenticate-azure-cli-interactively)를 참조하세요.

1. 메시지가 표시되면 지침에 따라 새 탭에서 로그인 페이지를 열고 제공된 인증 코드와 Azure 자격 증명을 입력합니다. 그런 다음 명령줄에서 로그인 프로세스를 완료하고 메시지가 표시되면 Azure AI 파운드리 허브가 포함된 구독을 선택합니다.

1. Cloud Shell 명령줄 창에서 다음 명령을 입력하여 앱을 실행합니다.

    ```
   python dalle-client.py
    ```

1. 메시지가 표시되면 이미지에 대한 요청(예: `Create an image of a robot eating pizza`)을 입력합니다. 잠시 후 앱은 이미지가 저장되었는지 확인합니다.

1. 몇 가지 프롬프트를 더 시도해 보겠습니다. 완료되면 `quit`를 입력하여 프로그램을 종료합니다.

    > **참고**: 이 간단한 앱에서는 대화 내용을 유지하는 논리를 구현하지 않았으므로 모델은 각 프롬프트를 이전 프롬프트의 컨텍스트 없이 새 요청으로 처리합니다.

1. 앱에서 생성된 이미지를 다운로드하고 보려면 Cloud Shell **다운로드** 명령을 사용하여 생성된 .png 파일을 지정합니다.

    ```
   download ./images/image_1.png
    ```

    다운로드 명령은 브라우저의 오른쪽 아래에 팝업 링크를 만듭니다. 이 링크를 선택하여 파일을 다운로드하고 열 수 있습니다.

## 요약

이 연습에서는 Azure AI 파운드리 및 Azure OpenAI SDK를 사용하여 DALL-E 모델을 통해 이미지를 생성하는 클라이언트 애플리케이션을 만들었습니다.

## 정리

DALL-E 탐색을 마쳤으면 이 연습에서 만든 리소스를 삭제하여 불필요한 Azure 비용이 발생하지 않도록 해야 합니다.

1. Azure Portal이 포함된 브라우저 탭으로 돌아가서(또는 새 브라우저 탭의 `https://portal.azure.com`에서 [Azure Portal](https://portal.azure.com)을 다시 열고) 이 연습에 사용된 리소스를 배포한 리소스 그룹의 콘텐츠를 확인합니다.
1. 도구 모음에서 **리소스 그룹 삭제**를 선택합니다.
1. 리소스 그룹 이름을 입력하고 삭제할 것인지 확인합니다.
