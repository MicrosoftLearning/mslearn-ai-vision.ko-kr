---
lab:
  title: 얼굴 감지 및 분석
  module: Module 4 - Detecting and Analyze Faces
---

# 얼굴 감지 및 분석

사람의 얼굴을 감지 및 분석하는 기능은 AI의 핵심 기능입니다. 이 연습에서는 이미지에 포함된 얼굴로 작업을 하는 데 사용할 수 있는 두 가지 Azure AI Services인 **Azure AI Vision** 서비스와 **Face** 서비스에 대해 살펴봅니다.

> **중요**: 이 랩은 제한된 기능에 대한 추가 액세스를 요청하지 않고도 완료할 수 있습니다.

> **참고**: 2022년 6월 21일부터 개인 식별 정보를 반환하는 Azure AI 서비스의 기능은 [제한된 액세스 권한](https://docs.microsoft.com/azure/cognitive-services/cognitive-services-limited-access)이 부여된 고객으로 제한됩니다. 또한 감정 상태를 유추하는 기능은 더 이상 사용할 수 없습니다. Microsoft가 변경한 내용 및 그 이유에 대한 자세한 내용은 [얼굴 인식에 대한 책임 있는 AI 투자 및 보호 조치](https://azure.microsoft.com/blog/responsible-ai-investments-and-safeguards-for-facial-recognition/)를 참조하세요.

## 이 과정용 리포지토리 복제

이 과정용 코드 리포지토리를 아직 복제하지 않았으면 복제해야 합니다.

1. Visual Studio Code 시작
2. 팔레트를 열고(Shift+Ctrl+P) **Git: Clone** 명령을 실행하여 `https://github.com/MicrosoftLearning/mslearn-ai-vision` 리포지토리를 로컬 폴더(아무 폴더나 관계없음)에 복제합니다.
3. 리포지토리가 복제되면 Visual Studio Code에서 폴더를 엽니다.
4. 리포지토리의 C# 코드 프로젝트를 지원하는 추가 파일이 설치되는 동안 기다립니다.

    > **참고**: 빌드 및 디버깅에 필요한 자산을 추가하라는 메시지가 표시되면 **나중에**를 선택합니다.

## Azure AI 서비스 리소스 프로비전

구독에 아직 없는 경우 **Azure AI 서비스** 리소스를 프로비전해야 합니다.

1. `https://portal.azure.com`의 Azure Portal을 열고 Azure 구독과 연관된 Microsoft 계정을 사용하여 로그인합니다.
2. 상단 검색 창에서 *Azure AI 서비스*를 검색하고 **Azure AI 서비스**를 선택한 후 다음 설정을 사용하여 Azure AI 서비스 다중 서비스 계정 리소스를 만듭니다.
    - **구독**: ‘Azure 구독’
    - **리소스 그룹**: 리소스 그룹 선택 또는 만들기(제한된 구독을 사용 중이라면 새 리소스 그룹을 만들 권한이 없을 수도 있으므로 제공된 리소스 그룹 사용)**
    - **지역**: *사용 가능한 지역 선택*
    - **이름**: *고유 이름 입력*
    - **가격 책정 계층**: 표준 S0
3. 필요한 확인란을 선택하고 리소스를 만듭니다.
4. 배포가 완료될 때까지 기다린 다음, 배포 세부 정보를 봅니다.
5. 리소스가 배포되면 해당 리소스로 이동하여 **키 및 엔드포인트** 페이지를 확인합니다. 다음 절차에서 이 페이지에 표시되는 키 중 하나와 엔드포인트가 필요합니다.

## Azure AI 비전 SDK 사용 준비

이 연습에서는 Azure AI 비전 SDK를 사용해 이미지의 얼굴을 분석하는 부분 구현 클라이언트 애플리케이션을 완성합니다.

> **참고**: **C#** 또는 **Python**용 SDK 사용을 선택할 수 있습니다. 아래 단계에서 선호하는 언어에 적합한 작업을 수행하세요.

1. Visual Studio Code의 **탐색기** 창에서 **Labfiles/04-face** 폴더를 찾은 다음 언어 선택에 따라 **C-Sharp** 또는 **Python** 폴더를 확장합니다.
2. **computer-vision** 폴더를 마우스 오른쪽 단추로 클릭하고 통합 터미널을 엽니다. 그런 다음, 언어 선택에 적절한 명령을 실행하여 Azure AI 비전 SDK 패키지를 설치합니다.

    **C#**

    ```
    dotnet add package Azure.AI.Vision.ImageAnalysis -v 1.0.0-beta.3
    ```

    **Python**

    ```
    pip install azure-ai-vision-imageanalysis==1.0.0b3
    ```
    
3. **computer-vision** 폴더의 내용을 표시하여 구성 설정용 파일이 포함되어 있음을 확인합니다.
    - **C#**: appsettings.json
    - **Python**: .env

4. 구성 파일을 열고 Azure AI 서비스 리소스용 **엔드포인트** 및 인증 **키**를 반영하여 해당 파일에 포함된 구성 값을 업데이트합니다. 변경 내용을 저장합니다.

5. **computer-vision** 폴더에는 클라이언트 애플리케이션용 코드 파일이 포함되어 있습니다.

    - **C#**: Program.cs
    - **Python**: detect-people.py

6. 코드 파일을 열고 파일 맨 윗부분의 기존 네임스페이스 참조 아래에 있는 **네임스페이스 가져오기** 주석을 찾습니다. 그런 다음 이 주석 아래에 다음 언어별 코드를 추가하여 Azure AI 비전 SDK를 사용하는 데 필요한 네임스페이스를 가져옵니다.

    **C#**

    ```C#
    // Import namespaces
    using Azure.AI.Vision.ImageAnalysis;
    ```

    **Python**

    ```Python
    # import namespaces
    from azure.ai.vision.imageanalysis import ImageAnalysisClient
    from azure.ai.vision.imageanalysis.models import VisualFeatures
    from azure.core.credentials import AzureKeyCredential
    ```

## 분석할 이미지 확인

이 연습에서는 Azure AI 비전 서비스를 사용하여 사람 이미지를 분석합니다.

1. Visual Studio Code에서 **computer-vision** 폴더와 이 폴더에 포함된 **images** 폴더를 차례로 확장합니다.
2. **people.jpg** 이미지를 선택하여 표시합니다.

## 이미지에서 얼굴 감지

이제 SDK를 사용해 Vision 서비스를 호출하고 이미지의 얼굴을 감지할 준비가 되었습니다.

1. 클라이언트 애플리케이션용 코드 파일(**Program.cs** 또는 **detect-faces.py**)의 **Main** 함수에서 구성 설정 로드를 위한 코드가 제공되어 있음을 확인합니다. 그런 다음 **Azure AI 비전 클라이언트 인증** 주석을 찾습니다. 그런 다음, 이 주석 아래에 다음 언어별 코드를 추가하여 Azure AI 비전 클라이언트 개체를 만들고 인증합니다.

    **C#**

    ```C#
    // Authenticate Azure AI Vision client
    ImageAnalysisClient cvClient = new ImageAnalysisClient(
        new Uri(aiSvcEndpoint),
        new AzureKeyCredential(aiSvcKey));
    ```

    **Python**

    ```Python
    # Authenticate Azure AI Vision client
    cv_client = ImageAnalysisClient(
        endpoint=ai_endpoint,
        credential=AzureKeyCredential(ai_key)
    )
    ```

2. **Main** 함수의 방금 추가한 코드 아래에 있는 코드가 이미지 파일 경로를 지정한 다음 **AnalyzeImage** 함수로 해당 이미지 경로를 전달함을 확인합니다. 이 함수는 아직 완전히 구현되지 않았습니다.

3. **AnalyzeImage** 함수의 **검색할 기능이 지정된 결과 가져오기(사용자)** 주석 아래에 다음 코드를 추가합니다.

    **C#**

    ```C#
    // Get result with specified features to be retrieved (PEOPLE)
    ImageAnalysisResult result = client.Analyze(
        BinaryData.FromStream(stream),
        VisualFeatures.People);
    ```

    **Python**

    ```Python
    # Get result with specified features to be retrieved (PEOPLE)
    result = cv_client.analyze(
        image_data=image_data,
        visual_features=[
            VisualFeatures.PEOPLE],
    )
    ```

4. **AnalyzeImage** 함수의 **감지된 사용자 주변에 경계 상자 그리기** 주석 아래에 다음 코드를 추가합니다.

    **C#**

    ```C
    // Draw bounding box around detected people
    foreach (DetectedPerson person in result.People.Values)
    {
        if (person.Confidence > 0.5) 
        {
            // Draw object bounding box
            var r = person.BoundingBox;
            Rectangle rect = new Rectangle(r.X, r.Y, r.Width, r.Height);
            graphics.DrawRectangle(pen, rect);
        }

        // Return the confidence of the person detected
        //Console.WriteLine($"   Bounding box {person.BoundingBox.ToString()}, Confidence: {person.Confidence:F2}");
    }
    ```

    **Python**
    
    ```Python
    # Draw bounding box around detected people
    for detected_people in result.people.list:
        if(detected_people.confidence > 0.5):
            # Draw object bounding box
            r = detected_people.bounding_box
            bounding_box = ((r.x, r.y), (r.x + r.width, r.y + r.height))
            draw.rectangle(bounding_box, outline=color, width=3)

        # Return the confidence of the person detected
        #print(" {} (confidence: {:.2f}%)".format(detected_people.bounding_box, detected_people.confidence * 100))
    ```

5. 변경 내용을 저장하고 **computer-vision** 폴더의 통합 터미널로 돌아와서 다음 명령을 입력하여 프로그램을 실행합니다.

    **C#**

    ```
    dotnet run
    ```

    **Python**

    ```
    python detect-people.py
    ```

6. 출력을 살펴봅니다. 감지된 얼굴 수가 표시됩니다.
7. 코드 파일과 같은 폴더에 생성된 **people.jpg** 파일을 표시하여 주석이 추가된 얼굴을 확인합니다. 여기서 코드는 얼굴 특성을 사용해 상자의 왼쪽 상단 위치에 레이블을 지정하고 경계 상자 좌표를 사용해 각 얼굴 주위에 사각형을 그렸습니다.

서비스에서 감지한 모든 사용자의 신뢰도 점수를 확인하기 위해 주석 `Return the confidence of the person detected` 아래 코드 줄의 주석 처리를 제거하고 코드를 다시 실행할 수 있습니다.

## Face SDK 사용 준비

**Azure AI 비전** 서비스는 기본적인 얼굴 감지 기능을 제공(기타 여러 이미지 분석 기능도 제공함)하는 반면 **Face** 서비스에서는 얼굴 분석 및 인식을 위한 더욱 포괄적인 기능을 제공합니다.

1. Visual Studio Code의 **탐색기** 창에서 **Labfiles/04-face** 폴더를 찾은 다음 언어 선택에 따라 **C-Sharp** 또는 **Python** 폴더를 확장합니다.
2. **face-api** 폴더를 마우스 오른쪽 단추로 클릭하고 통합 터미널을 엽니다. 그런 다음 언어 기본 설정에 적합한 명령을 실행하여 Face SDK 패키지를 설치합니다.

    **C#**

    ```
    dotnet add package Azure.AI.Vision.Face -v 1.0.0-beta.2
    ```

    **Python**

    ```
    pip install azure-ai-vision-face==1.0.0b2
    ```
    
3. **face-api** 폴더의 내용을 표시하여 구성 설정용 파일이 포함되어 있음을 확인합니다.
    - **C#**: appsettings.json
    - **Python**: .env

4. 구성 파일을 열고 Azure AI 서비스 리소스용 **엔드포인트** 및 인증 **키**를 반영하여 해당 파일에 포함된 구성 값을 업데이트합니다. 변경 내용을 저장합니다.

5. **face-api** 폴더에는 클라이언트 애플리케이션용 코드 파일이 포함되어 있습니다.

    - **C#**: Program.cs
    - **Python**: analyze-faces.py

6. 코드 파일을 열고 파일 맨 윗부분의 기존 네임스페이스 참조 아래에 있는 **네임스페이스 가져오기** 주석을 찾습니다. 그런 다음 이 주석 아래에 다음 언어별 코드를 추가하여 Vision SDK를 사용하는 데 필요한 네임스페이스를 가져옵니다.

    **C#**

    ```C#
    // Import namespaces
    using Azure;
    using Azure.AI.Vision.Face;
    ```

    **Python**

    ```Python
    # Import namespaces
    from azure.ai.vision.face import FaceClient
    from azure.ai.vision.face.models import FaceDetectionModel, FaceRecognitionModel, FaceAttributeTypeDetection03
    from azure.core.credentials import AzureKeyCredential
    ```

7. **Main** 함수에서 구성 설정 로드를 위한 코드가 제공되어 있음을 확인합니다. 그런 다음 **Face 클라이언트 인증** 주석을 찾습니다. 그 후에 이 주석 아래에 다음 언어별 코드를 추가하여 **FaceClient** 개체를 만들고 인증합니다.

    **C#**

    ```C#
    // Authenticate Face client
    faceClient = new FaceClient(
        new Uri(cogSvcEndpoint),
        new AzureKeyCredential(cogSvcKey));
    ```

    **Python**

    ```Python
    # Authenticate Face client
    face_client = FaceClient(
        endpoint=cog_endpoint,
        credential=AzureKeyCredential(cog_key)
    )
    ```

8. **Main** 함수의 방금 추가한 코드 아래에 있는 코드가 메뉴를 표시함을 확인합니다. 이 메뉴를 사용하면 코드의 함수를 호출하여 Face 서비스 기능을 살펴볼 수 있습니다. 이 연습의 나머지 부분에서 이러한 함수를 구현합니다.

## 얼굴 감지 및 분석

Face 서비스의 가장 기본적인 기능 중 하나는 이미지에서 얼굴을 감지하고 해당 특성을 확인하는 것입니다. 이러한 특성으로는 머리 자세, 흐릿한 형체, 마스크 유무 등이 있습니다.

1. 애플리케이션용 코드 파일의 **Main** 함수에서 사용자가 메뉴 옵션 **1**을 선택하면 실행되는 코드를 살펴봅니다. 이 코드는 **DetectFaces** 함수를 호출하여 이미지 파일의 경로를 전달합니다.
2. 코드 파일에서 **DetectFaces** 함수를 찾은 다음 **검색할 얼굴 기능 지정** 주석 아래에 다음 코드를 추가합니다.

    **C#**

    ```C#
    // Specify facial features to be retrieved
    FaceAttributeType[] features = new FaceAttributeType[]
    {
        FaceAttributeType.Detection03.HeadPose,
        FaceAttributeType.Detection03.Blur,
        FaceAttributeType.Detection03.Mask
    };
    ```

    **Python**

    ```Python
    # Specify facial features to be retrieved
    features = [FaceAttributeTypeDetection03.HEAD_POSE,
                FaceAttributeTypeDetection03.BLUR,
                FaceAttributeTypeDetection03.MASK]
    ```

3. **DetectFaces** 함수의 방금 추가한 코드 아래에서 **얼굴 가져오기** 주석을 찾은 후 다음 코드를 추가합니다.

**C#**

```C#
// Get faces
using (var imageData = File.OpenRead(imageFile))
{    
    var response = await faceClient.DetectAsync(
        BinaryData.FromStream(imageData),
        FaceDetectionModel.Detection03,
        FaceRecognitionModel.Recognition04,
        returnFaceId: false,
        returnFaceAttributes: features);
    IReadOnlyList<FaceDetectionResult> detected_faces = response.Value;

    if (detected_faces.Count() > 0)
    {
        Console.WriteLine($"{detected_faces.Count()} faces detected.");

        // Prepare image for drawing
        Image image = Image.FromFile(imageFile);
        Graphics graphics = Graphics.FromImage(image);
        Pen pen = new Pen(Color.LightGreen, 3);
        Font font = new Font("Arial", 4);
        SolidBrush brush = new SolidBrush(Color.White);
        int faceCount=0;

        // Draw and annotate each face
        foreach (var face in detected_faces)
        {
            faceCount++;
            Console.WriteLine($"\nFace number {faceCount}");
            
            // Get face properties
            Console.WriteLine($" - Head Pose (Yaw): {face.FaceAttributes.HeadPose.Yaw}");
            Console.WriteLine($" - Head Pose (Pitch): {face.FaceAttributes.HeadPose.Pitch}");
            Console.WriteLine($" - Head Pose (Roll): {face.FaceAttributes.HeadPose.Roll}");
            Console.WriteLine($" - Blur: {face.FaceAttributes.Blur.BlurLevel}");
            Console.WriteLine($" - Mask: {face.FaceAttributes.Mask.Type}");

            // Draw and annotate face
            var r = face.FaceRectangle;
            Rectangle rect = new Rectangle(r.Left, r.Top, r.Width, r.Height);
            graphics.DrawRectangle(pen, rect);
            string annotation = $"Face number {faceCount}";
            graphics.DrawString(annotation,font,brush,r.Left, r.Top);
        }

        // Save annotated image
        String output_file = "detected_faces.jpg";
        image.Save(output_file);
        Console.WriteLine(" Results saved in " + output_file);   
    }
}
```

**Python**

```Python
# Get faces
with open(image_file, mode="rb") as image_data:
    detected_faces = face_client.detect(
        image_content=image_data.read(),
        detection_model=FaceDetectionModel.DETECTION03,
        recognition_model=FaceRecognitionModel.RECOGNITION04,
        return_face_id=False,
        return_face_attributes=features,
    )

    if len(detected_faces) > 0:
        print(len(detected_faces), 'faces detected.')

        # Prepare image for drawing
        fig = plt.figure(figsize=(8, 6))
        plt.axis('off')
        image = Image.open(image_file)
        draw = ImageDraw.Draw(image)
        color = 'lightgreen'
        face_count = 0

        # Draw and annotate each face
        for face in detected_faces:

            # Get face properties
            face_count += 1
            print('\nFace number {}'.format(face_count))

            print(' - Head Pose (Yaw): {}'.format(face.face_attributes.head_pose.yaw))
            print(' - Head Pose (Pitch): {}'.format(face.face_attributes.head_pose.pitch))
            print(' - Head Pose (Roll): {}'.format(face.face_attributes.head_pose.roll))
            print(' - Blur: {}'.format(face.face_attributes.blur.blur_level))
            print(' - Mask: {}'.format(face.face_attributes.mask.type))

            # Draw and annotate face
            r = face.face_rectangle
            bounding_box = ((r.left, r.top), (r.left + r.width, r.top + r.height))
            draw = ImageDraw.Draw(image)
            draw.rectangle(bounding_box, outline=color, width=5)
            annotation = 'Face number {}'.format(face_count)
            plt.annotate(annotation,(r.left, r.top), backgroundcolor=color)

        # Save annotated image
        plt.imshow(image)
        outputfile = 'detected_faces.jpg'
        fig.savefig(outputfile)

        print('\nResults saved in', outputfile)
```

4. **DetectFaces** 함수에 추가한 코드를 살펴봅니다. 이미지 파일을 분석하고 머리 자세, 흐릿한 형체 및 마스크 유무 특성 등 이미지 파일에 포함된 모든 얼굴을 감지합니다. 각 얼굴에 할당되는 고유 얼굴 식별자를 비롯한 각 얼굴의 세부 정보가 표시됩니다. 그리고 경계 상자를 사용하여 이미지상의 얼굴 위치가 표시됩니다.
5. 변경 내용을 저장하고 **face-api** 폴더의 통합 터미널로 돌아와서 다음 명령을 입력하여 프로그램을 실행합니다.

    **C#**

    ```
    dotnet run
    ```

    *C# 출력에 **await** 연산자를 사용하는 비동기 함수 관련 경고가 표시될 수 있습니다. 이러한 경고는 이러한 메시지는 무시해도 됩니다.*

    **Python**

    ```
    python analyze-faces.py
    ```

6. 메시지가 표시되면 **1**을 입력하고 출력을 살펴봅니다. 출력에는 감지된 각 얼굴의 ID와 특성이 포함되어 있습니다.
7. 코드 파일과 같은 폴더에 생성된 **detected_faces.jpg** 파일을 표시하여 주석이 추가된 얼굴을 확인합니다.

## 자세한 정보

**Face** 서비스 내에서 사용할 수 있는 몇 가지 추가 기능이 있지만 [요구되는 AI 표준](https://aka.ms/aah91ff)에 따라 제한된 액세스 정책 하에 제한됩니다. 이러한 기능에는 얼굴 인식 모델 식별, 검증 및 생성이 있습니다. 자세한 내용을 알아보고 액세스를 신청하려면 [Azure AI 서비스에 대한 제한된 액세스](https://docs.microsoft.com/en-us/azure/cognitive-services/cognitive-services-limited-access)를 참조하세요.

얼굴 감지를 위한 **Azure AI 비전** 서비스 사용에 대한 자세한 내용은 [Azure AI 비전 설명서](https://docs.microsoft.com/azure/cognitive-services/computer-vision/concept-detecting-faces)를 참조하세요.

**Face** 서비스에 대해 자세히 알아보려면 [Face 설명서](https://learn.microsoft.com/azure/ai-services/computer-vision/overview-identity)를 참조하세요.
