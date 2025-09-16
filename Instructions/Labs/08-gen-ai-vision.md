---
lab:
  title: 비전 지원 채팅 앱 개발
  description: Azure AI Foundry를 사용하여 이미지 입력을 지원하는 생성형 AI 앱을 빌드합니다.
---

# 비전 지원 채팅 앱 개발

이 연습에서는 *Phi-4-multimodal-instruct* 생성형 AI 모델을 사용하여 이미지를 포함하는 프롬프트에 대한 응답을 생성합니다. Azure AI 파운드리 및 Azure AI 모델 추론 서비스를 사용하여 식료품점에서 신선 식품으로 AI 지원을 제공하는 앱을 개발합니다.

> **참고**: 이 연습은 사전 출시된 SDK 소프트웨어를 기반으로 하며, 변경될 수 있습니다. 필요한 경우 특정 버전의 패키지를 사용했으며, 이는 최신 버전이 반영되지 않을 수 있습니다. 예기치 않은 동작, 경고 또는 오류가 발생할 수 있습니다.

이 연습은 Azure AI Foundry Python SDK를 기반으로 하지만 다음을 포함하여 여러 언어별 SDK를 사용하여 AI 채팅 애플리케이션을 개발할 수 있습니다. 포함 사항:

- [Python용 Azure AI 프로젝트](https://pypi.org/project/azure-ai-projects)
- [Microsoft .NET용 Azure AI 프로젝트](https://www.nuget.org/packages/Azure.AI.Projects)
- [JavaScript용 Azure AI 프로젝트](https://www.npmjs.com/package/@azure/ai-projects)

이 연습에는 약 **30**분이 소요됩니다.

## Azure AI Foundry 포털 열기

Azure AI 파운드리 포털에 로그인하여 시작해 보겠습니다.

1. 웹 브라우저에서 [Azure AI 파운드리 포털](https://ai.azure.com)(`https://ai.azure.com`)을 열고 Azure 자격 증명을 사용하여 로그인합니다. 처음 로그인할 때 열리는 팁이나 빠른 시작 창을 닫고, 필요한 경우 왼쪽 위에 있는 **Azure AI 파운드리** 로고를 사용하여 다음 이미지와 유사한 홈페이지로 이동합니다(**도움말** 창이 열려 있는 경우 닫습니다).

    ![Azure AI Foundry 포털의 스크린샷.](./media/ai-foundry-home.png)

1. 홈페이지의 정보를 리뷰합니다.

## 프로젝트를 시작할 모델 선택

Azure AI *프로젝트*는 AI 개발을 위한 공동 작업 영역을 제공합니다. 작업할 모델을 선택하고 이를 사용할 프로젝트를 만드는 것부터 시작하겠습니다.

> **참고**: AI 파운드리 프로젝트는 AI 에이전트 및 채팅 솔루션 개발을 위한 AI 모델(Azure OpenAI 포함), Azure AI 서비스 및 기타 리소스에 대한 액세스를 제공하는 *Azure AI 파운드리* 리소스를 기반으로 할 수 있습니다. 또는 프로젝트는 보안 스토리지, 컴퓨팅 및 특수 도구에 대한 Azure 리소스 연결을 포함하는 *AI 허브* 리소스를 기반으로 할 수 있습니다. Azure AI 파운드리 기반 프로젝트는 AI 에이전트 또는 채팅 앱 개발을 위한 리소스를 관리하려는 개발자에게 적합합니다. AI 허브 기반 프로젝트는 복잡한 AI 솔루션을 개발하는 엔터프라이즈 개발 팀에 더 적합합니다.

1. 홈페이지의 **모델 및 기능 탐색** 섹션에서 프로젝트에 사용할 `Phi-4-multimodal-instruct` 모델을 검색합니다.

1. 검색 결과에서 **Phi-4-multimodal-instruct** 모델을 선택하여 세부 정보를 확인한 다음, 해당 모델 페이지 상단에서 **이 모델 사용**을 선택합니다.

1. 프로젝트를 만들라는 메시지가 표시되면 프로젝트의 유효한 이름을 입력하고 **고급 옵션**을 펼칩니다.

1. **사용자 지정**을 선택하고 허브에 대해 다음 설정을 지정합니다.
    - **Azure AI 파운드리 리소스**: *Azure AI 파운드리 리소스의 유효한 이름*
    - **구독**: ‘Azure 구독’
    - **리소스 그룹**: ‘리소스 그룹 만들기 또는 선택’
    - **지역**: ***AI Foundry 권장 사항 선택***\*

    > \* 일부 Azure AI 리소스는 지역 모델 할당량에 의해 제한됩니다. 연습 후반부에 할당량 한도를 초과하는 경우 다른 지역에서 다른 리소스를 만들어야 할 수도 있습니다.

1. **만들기**를 선택하고 선택한 Phi-4-multimodal-instruct 모델 배포를 포함하여 프로젝트가 생성될 때까지 기다립니다.

    > 참고: 모델 선택에 따라 프로젝트 만들기 프로세스 중에 추가 프롬프트가 표시될 수 있습니다. 조건에 동의하고 배포를 완료합니다.

1. 프로젝트가 만들어지면 모델이 **모델 + 엔드포인트** 페이지에 표시됩니다.

    ![모델 배포 페이지 스크린샷](./media/ai-foundry-model-deployment.png)

## 플레이그라운드의 모델 테스트

이제 채팅 플레이그라운드에서 이미지 기반 프롬프트를 사용하여 다중 모달 모델 배포를 테스트할 수 있습니다.

1. 모델 배포 페이지에서 **플레이그라운드에서 열기**를 선택합니다.

1. 새 브라우저 탭에서 [mango.jpeg](https://github.com/MicrosoftLearning/mslearn-ai-vision/raw/refs/heads/main/Labfiles/gen-ai-vision/mango.jpeg)(`https://github.com/MicrosoftLearning/mslearn-ai-vision/raw/refs/heads/main/Labfiles/gen-ai-vision/mango.jpeg`)를 다운로드하여 로컬 파일 시스템의 폴더에 저장합니다.

1. 채팅 플레이그라운드 페이지의 **설정** 창에서 **Phi-4-multimodal-instruct** 모델의 모델 배포가 선택되어 있는지 확인합니다.

1. 기본 채팅 세션 패널의 채팅 입력 상자 아래에서 연결 단추(**&#128206;**)를 사용하여 *mango.jpg* 이미지 파일을 업로드한 다음 텍스트(`What desserts could I make with this fruit?`)를 추가하고 프롬프트를 제출합니다.

    ![채팅 플레이그라운드 페이지 스크린샷](../media/chat-playground-image.png)

1. 응답을 검토하면 망고를 사용하여 만들 수 있는 디저트에 대한 관련 참고 자료를 얻을 수 있을 것입니다.

## 클라이언트 애플리케이션 생성하기

이제 모델을 배포했으므로 클라이언트 애플리케이션에서 배포를 사용할 수 있습니다.

### 애플리케이션 구성 준비

1. Azure AI 파운드리 포털에서 프로젝트의 **개요** 페이지를 봅니다.

1. **엔드포인트 및 키** 영역에서 **Azure AI Foundry** 라이브러리가 선택되어 있는지 확인하고 **Azure AI Foundry 프로젝트 엔드포인트**를 확인합니다. 이 연결 문자열 사용하여 클라이언트 응용 프로그램에서 프로젝트에 연결합니다.

1. 새 브라우저 탭을 엽니다(Azure AI 파운드리 포털을 기존 탭에서 열어 두기). 그런 다음 새 탭에서 [Azure Portal](https://portal.azure.com)(`https://portal.azure.com`)을 열고 메시지가 나타나면 Azure 자격 증명을 사용하여 로그인합니다.

    Azure Portal 홈페이지를 보려면 환영 알림을 닫습니다.

1. 페이지 상단의 검색 창 오른쪽에 있는 **[\>_]** 단추를 사용하여 Azure Portal에서 새 Cloud Shell을 만들고 구독에 저장소가 없는 ***PowerShell*** 환경을 선택합니다.

    Cloud Shell은 Azure Portal 하단의 창에서 명령줄 인터페이스를 제공합니다. 보다 쉽게 작업할 수 있도록 이 창의 크기를 조정하거나 최대화할 수 있습니다.

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
   cd mslearn-ai-vision/Labfiles/gen-ai-vision/python
    ```

1. Cloud Shell 명령줄 창에서 다음 명령을 입력하여 사용할 라이브러리를 설치합니다.

    ```
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install -r requirements.txt azure-identity azure-ai-projects openai
    ```

1. 제공된 구성 파일을 편집하려면 다음 명령을 입력합니다.

    ```
   code .env
    ```

    코드 편집기에서 파일이 열립니다.

1. 코드 파일에서 **your_project_endpoint** 자리 표시자를 Foundry 프로젝트 엔드포인트(Azure AI Foundry 포털의 프로젝트 **개요** 페이지에서 복사)로 바꾸고 **your_model_deployment** 자리 표시자를 Phi-4-multimodal-instruct 모델 배포에 할당한 이름으로 바꿉니다.

1. 자리 표시자를 바꾼 후 코드 편집기에서 **CTRL+S** 명령 또는 **마우스 오른쪽 단추 클릭 > 저장**을 사용하여 변경 내용을 저장한 다음 **CTRL+Q** 명령 또는 **마우스 오른쪽 단추 클릭 > 종료**를 사용하여 Cloud Shell 명령줄을 열어둔 채 코드 편집기를 닫습니다.

### 프로젝트에 연결하고 모델에 대한 채팅 클라이언트를 가져오는 코드를 작성합니다.

> **팁**: 코드를 추가할 때 올바른 들여쓰기를 유지해야 합니다.

1. 제공된 코드 파일을 편집하려면 다음 명령을 입력합니다.

    ```
   code chat-app.py
    ```

1. 코드 파일에서 파일 상단에 추가된 기존 문에 주목하여 필요한 SDK 네임스페이스를 가져옵니다. 그런 다음 **Add references** 주석을 찾아 이전에 설치한 라이브러리의 네임스페이스를 참조하도록 다음 코드를 추가합니다.

    ```python
   # Add references
   from azure.identity import DefaultAzureCredential
   from azure.ai.projects import AIProjectClient
   from openai import AzureOpenAI
    ```

1. **main** 함수의 **구성 설정 가져오기** 주석 아래에서 해당 코드가 구성 파일에서 정의한 프로젝트 연결 문자열 및 모델 배포 이름 값을 로드한다는 점에 유의합니다.
1. **main** 함수의 **구성 설정 가져오기** 주석 아래에서 해당 코드가 구성 파일에서 정의한 프로젝트 연결 문자열 및 모델 배포 이름 값을 로드한다는 점에 유의합니다.
1. **프로젝트 클라이언트 초기화** 주석을 찾아 다음 코드를 추가하여 Azure AI Foundry 프로젝트에 연결합니다.

    > **팁**: 코드의 들여쓰기 수준을 올바르게 유지하도록 주의하세요.

    ```python
   # Initialize the project client
   project_client = AIProjectClient(            
            credential=DefaultAzureCredential(
                exclude_environment_credential=True,
                exclude_managed_identity_credential=True
            ),
            endpoint=project_endpoint,
        )
    ```

1. **채팅 클라이언트 가져오기** 주석을 찾아 다음 코드를 추가하여 모델과 채팅할 클라이언트 개체를 만듭니다.

    ```python
   # Get a chat client
   openai_client = project_client.get_openai_client(api_version="2024-10-21")
    ```

### URL 기반 이미지 프롬프트를 제출하는 코드 작성

1. 코드에는 사용자가 "종료"를 입력할 때까지 프롬프트를 입력할 수 있도록 하는 루프가 포함되어 있습니다. 그런 다음 루프 섹션에서 **Get a response to image input** 주석을 찾아 다음 코드를 추가하여 다음 이미지가 포함된 프롬프트를 제출합니다.

    ![망고 사진.](../media/orange.jpeg)

    ```python
   # Get a response to image input
   image_url = "https://github.com/MicrosoftLearning/mslearn-ai-vision/raw/refs/heads/main/Labfiles/gen-ai-vision/orange.jpeg"
   image_format = "jpeg"
   request = Request(image_url, headers={"User-Agent": "Mozilla/5.0"})
   image_data = base64.b64encode(urlopen(request).read()).decode("utf-8")
   data_url = f"data:image/{image_format};base64,{image_data}"

   response = openai_client.chat.completions.create(
        model=model_deployment,
        messages=[
            {"role": "system", "content": system_message},
            { "role": "user", "content": [  
                { "type": "text", "text": prompt},
                { "type": "image_url", "image_url": {"url": data_url}}
            ] } 
        ]
   )
   print(response.choices[0].message.content)
    ```

1. **Ctrl+S** 명령을 사용하여 변경 내용을 코드 파일에 저장합니다. 아직 닫지 마세요.

## Azure에 로그인하고 앱 실행

1. Cloud Shell 명령줄 창에서 다음 명령을 입력하여 Azure에 로그인합니다.

    ```
   az login
    ```

    **<font color="red">Cloud Shell 세션이 이미 인증되었더라도 Azure에 로그인해야 합니다.</font>**

    > **참고**: 대부분의 시나리오에서는 *az login*을 사용하는 것만으로도 충분합니다. 그러나 여러 테넌트에 구독이 있는 경우 *--tenant* 매개 변수를 사용하여 테넌트 지정해야 할 수 있습니다. 자세한 내용은 [Sign into Azure interactively using the Azure CLI](https://learn.microsoft.com/cli/azure/authenticate-azure-cli-interactively)를 참조하세요.
    
1. 메시지가 표시되면 지침에 따라 새 탭에서 로그인 페이지를 열고 제공된 인증 코드와 Azure 자격 증명을 입력합니다. 그런 다음 명령줄에서 로그인 프로세스를 완료하고 메시지가 표시되면 Azure AI 파운드리 허브가 포함된 구독을 선택합니다.

1. 로그인한 후 다음 명령을 입력하여 애플리케이션을 실행합니다.

    ```
   python chat-app.py
    ```

1. 메시지가 표시되면 다음 프롬프트를 입력합니다.

    ```
   Suggest some recipes that include this fruit
    ```

1. 응답을 검토합니다. 그런 다음 `quit`(을)를 입력하여 프로그램을 종료합니다.

### 로컬 이미지 파일을 업로드하도록 코드 수정

1. 앱 코드의 코드 편집기에서 루프 섹션의 **Get a response to image input** 주석 아래에서 이전에 추가한 코드를 찾습니다. 그런 다음 다음과 같이 코드를 수정하여 이 로컬 이미지 파일을 업로드합니다.

    ![용과 사진.](../media/mystery-fruit.jpeg)

    ```python
   # Get a response to image input
   script_dir = Path(__file__).parent  # Get the directory of the script
   image_path = script_dir / 'mystery-fruit.jpeg'
   mime_type = "image/jpeg"

   # Read and encode the image file
   with open(image_path, "rb") as image_file:
        base64_encoded_data = base64.b64encode(image_file.read()).decode('utf-8')

   # Include the image file data in the prompt
   data_url = f"data:{mime_type};base64,{base64_encoded_data}"
   response = openai_client.chat.completions.create(
            model=model_deployment,
            messages=[
                {"role": "system", "content": system_message},
                { "role": "user", "content": [  
                    { "type": "text", "text": prompt},
                    { "type": "image_url", "image_url": {"url": data_url}}
                ] } 
            ]
   )
   print(response.choices[0].message.content)
    ```

1. **CTRL+S** 명령을 사용하여 변경 내용을 코드 파일에 저장합니다. 원하는 경우 코드 편집기를 닫을 수도 있습니다(**CTRL+Q**).

1. 코드 편집기 아래의 Cloud Shell 명령줄 창에서 다음 명령을 입력하여 앱을 실행합니다.

    ```
   python chat-app.py
    ```

1. 메시지가 표시되면 다음 프롬프트를 입력합니다.

    ```
   What is this fruit? What recipes could I use it in?
    ```

15. 응답을 검토합니다. 그런 다음 `quit`(을)를 입력하여 프로그램을 종료합니다.

    > **참고**: 이 간단한 앱에서는 대화 내용을 유지하는 논리를 구현하지 않았으므로 모델은 각 프롬프트를 이전 프롬프트의 컨텍스트 없이 새 요청으로 처리합니다.

## 정리

Azure AI Foundry 포털 탐색을 완료한 경우 불필요한 Azure 비용이 발생하지 않도록 이 연습에서 만든 리소스를 삭제해야 합니다.

1. [Azure Portal](https://portal.azure.com)을 열고 이 연습에 사용된 리소스를 배포한 리소스 그룹의 내용을 확인합니다.
1. 도구 모음에서 **리소스 그룹 삭제**를 선택합니다.
1. 리소스 그룹 이름을 입력하고 삭제할 것인지 확인합니다.
