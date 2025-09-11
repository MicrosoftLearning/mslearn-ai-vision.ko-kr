---
lab:
  title: 얼굴 감지 및 분석
  module: Module 4 - Detecting and Analyze Faces
---

# 얼굴 감지 및 분석

사람의 얼굴을 감지 및 분석하는 기능은 AI의 핵심 기능입니다. 이 연습에서는 이미지에 포함된 얼굴로 작업을 하는 데 사용할 수 있는 두 가지 Azure AI Services인 **Azure AI Vision** 서비스와 **Face** 서비스에 대해 살펴봅니다.

> **중요**: 이 랩은 제한된 기능에 대한 추가 액세스를 요청하지 않고도 완료할 수 있습니다.

> **참고**: 2022년 6월 21일부터 개인 식별 정보를 반환하는 Azure AI 서비스의 기능은 [제한된 액세스 권한](https://docs.microsoft.com/azure/cognitive-services/cognitive-services-limited-access)이 부여된 고객으로 제한됩니다. 또한 감정 상태를 유추하는 기능은 더 이상 사용할 수 없습니다. Microsoft가 변경한 내용 및 그 이유에 대한 자세한 내용은 [얼굴 인식에 대한 책임 있는 AI 투자 및 보호 조치](https://azure.microsoft.com/blog/responsible-ai-investments-and-safeguards-for-facial-recognition/)를 참조하세요.

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
1. 배포가 완료될 때까지 기다린 다음, 배포 세부 정보를 봅니다.
1. 리소스가 배포되면 해당 리소스로 이동하여 **키 및 엔드포인트** 페이지를 확인합니다. 다음 절차에서 이 페이지에 표시되는 키 중 하나와 엔드포인트가 필요합니다.

## 이 과정용 리포지토리 복제

Azure Portal에서 Cloud Shell을 사용하여 코드를 개발합니다. 앱의 코드 파일은 GitHub 리포지토리에 제공되었습니다.

> **팁**: 최근에 **mslearn-ai-vision** 리포지토리를 이미 복제한 경우 이 작업을 건너뛰어도 됩니다. 그렇지 않은 경우에는 다음 단계에 따라 개발 환경에 복제합니다.

1. Azure Portal에서 페이지 상단의 검색 창 오른쪽에 있는 **[\>_]** 버튼으로 Azure Portal에서 새 Cloud Shell을 생성하고 ***PowerShell*** 환경을 선택합니다. Cloud Shell은 다음과 같이 Azure Portal 아래쪽 창에 명령줄 인터페이스를 제공합니다.

    > **참고**: 이전에 *Bash* 환경을 사용하는 Cloud Shell을 만든 경우 ***PowerShell***로 전환합니다.

1. Cloud Shell 도구 모음의 **설정** 메뉴에서 **클래식 버전으로 이동**을 선택합니다(코드 편집기를 사용하는 데 필요).

    > **팁**: CloudShell에 명령을 붙여넣을 때, 출력이 화면 버퍼의 많은 부분을 차지할 수 있습니다. `cls` 명령을 입력해 화면을 지우면 각 작업에 더 집중할 수 있습니다.

1. PowerShell 창에서 다음 명령을 입력하여 이 연습이 포함된 GitHub 리포지토리를 복제합니다.

    ```
    rm -r mslearn-ai-vision -f
    git clone https://github.com/microsoftlearning/mslearn-ai-vision mslearn-ai-vision
    ```

1. 리포지토리가 복제된 후 애플리케이션 코드 파일이 포함된 폴더로 이동합니다.  

    ```
   cd mslearn-ai-vision/Labfiles/04-face
    ```

## Azure AI 비전 SDK 사용 준비

이 연습에서는 Azure AI 비전 SDK를 사용하여 이미지를 분석하는 부분적으로 구현된 클라이언트 애플리케이션을 완료합니다.

> **참고**: **C#** 또는 **Python**용 SDK 사용을 선택할 수 있습니다. 아래 단계에서 선호하는 언어에 적합한 작업을 수행하세요.

1. 기본 설정 언어에 대한 애플리케이션 코드 파일이 포함된 폴더로 이동합니다.  

    **C#**

    ```
   cd C-Sharp/computer-vision
    ```
    
    **Python**

    ```
   cd Python/computer-vision
    ```

1. 언어 기본 설정에 적합한 명령을 실행하여 Azure AI 비전 SDK 패키지 및 필수 종속성을 설치합니다.

    **C#**
    
    ```
    dotnet add package Azure.AI.Vision.ImageAnalysis -v 1.0.0
    dotnet add package SkiaSharp --version 3.116.1
    dotnet add package SkiaSharp.NativeAssets.Linux --version 3.116.1
    ``` 

    **Python**
    
    ```
    pip install azure-ai-vision-imageanalysis==1.0.0
    pip install dotenv
    pip install matplotlib
    ```

1. `ls` 명령을 사용하여 **computer-vision** 폴더의 내용을 볼 수 있습니다. 여기에는 구성 설정을 위한 파일이 포함되어 있습니다.

    - **C#**: appsettings.json
    - **Python**: .env

1. 제공된 구성 파일을 편집하려면 다음 명령을 입력합니다.

    **C#**

    ```
   code appsettings.json
    ```

    **Python**

    ```
   code .env
    ```

    코드 편집기에서 파일이 열립니다.

1. 코드 파일에서 Computer Vision 리소스에 대한 **엔드포인트** 및 인증 **키**를 반영하도록 포함된 구성 값을 업데이트합니다.
1. 자리 표시자를 바꾼 후 **Ctrl+S** 명령을 사용하여 변경 내용을 저장한 다음 **Ctrl+Q** 명령을 사용하여 Cloud Shell 명령줄을 열어 두고 코드 편집기를 닫습니다.

## 분석할 이미지 확인

이 연습에서는 Azure AI 비전 서비스를 사용하여 사람 이미지를 분석합니다.

1. Cloud Shell 도구 모음에서 **파일 업로드/다운로드**를 선택한 다음, **다운로드**를 선택합니다. 새 대화 상자에서 다음 파일 경로를 입력하고 **다운로드**를 선택합니다.

    ```
    mslearn-ai-vision/Labfiles/04-face/C-Sharp/computer-vision/images/people.jpg
    ```

1. **people.jpg** 이미지를 열어 표시합니다.

## 이미지에서 얼굴 감지

이제 SDK를 사용해 Vision 서비스를 호출하고 이미지의 얼굴을 감지할 준비가 되었습니다.

1. **computer-vision** 폴더에는 클라이언트 애플리케이션용 코드 파일이 포함되어 있습니다.

    - **C#**: Program.cs
    - **Python**: detect-people.py

1. 다음 명령을 입력하여 클라이언트 응용 프로그램의 코드 파일을 엽니다:

    **C#**

    ```
   code Program.cs
    ```

    **Python**

    ```
   code image-analysis.py
    ```

1. **Import namespaces** 주석 아래에 다음 언어별 코드를 추가하여 Azure AI 비전 SDK를 사용하는 데 필요한 네임스페이스를 가져옵니다.

    **C#**
    
    ```C#
    // Import namespaces
    using Azure.AI.Vision.ImageAnalysis;
    using SkiaSharp;
    ```
    
    **Python**
    
    ```Python
    # import namespaces
    from azure.ai.vision.imageanalysis import ImageAnalysisClient
    from azure.ai.vision.imageanalysis.models import VisualFeatures
    from azure.core.credentials import AzureKeyCredential
    ```
    
1. **Main** 함수에서 구성 설정 로드를 위한 코드가 제공되어 있음을 확인합니다. 그런 다음 **Azure AI 비전 클라이언트 인증** 주석을 찾습니다. 그런 다음, 이 주석 아래에 다음 언어별 코드를 추가하여 Azure AI 비전 클라이언트 개체를 만들고 인증합니다.

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

1. **Main** 함수의 방금 추가한 코드 아래에 있는 코드가 이미지 파일 경로를 지정한 다음 **AnalyzeImage** 함수로 해당 이미지 경로를 전달함을 확인합니다. 이 함수는 아직 완전히 구현되지 않았습니다.

1. **AnalyzeImage** 함수의 **검색할 기능이 지정된 결과 가져오기(사용자)** 주석 아래에 다음 코드를 추가합니다.

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

1. **AnalyzeImage** 함수의 **감지된 사용자 주변에 경계 상자 그리기** 주석 아래에 다음 코드를 추가합니다.

    **C#**

    ```C
    // Draw bounding box around detected people
    foreach (DetectedPerson person in result.People.Values)
    {
        if (person.Confidence > 0.5) 
        {
            // Draw object bounding box
            var r = person.BoundingBox;
            SKRect rect = new SKRect(r.X, r.Y, r.X + r.Width, r.Y + r.Height);
            canvas.DrawRect(rect, paint);
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

1. 변경 내용을 저장하고, 코드 편집기를 닫고, 다음 명령을 입력하여 프로그램을 실행합니다.

    **C#**

    ```
    dotnet run
    ```

    **Python**

    ```
    python detect-people.py
    ```

6. 출력을 살펴봅니다. 감지된 얼굴 수가 표시됩니다.
7. 코드 파일과 같은 폴더에 생성된 **people.jpg** 파일을 다운로드하여 주석이 추가된 얼굴을 확인합니다. 여기서 코드는 얼굴 특성을 사용해 상자의 왼쪽 상단 위치에 레이블을 지정하고 경계 상자 좌표를 사용해 각 얼굴 주위에 사각형을 그렸습니다.

서비스에서 감지한 모든 사용자의 신뢰도 점수를 확인하기 위해 주석 `Return the confidence of the person detected` 아래 코드 줄의 주석 처리를 제거하고 코드를 다시 실행할 수 있습니다.

## Face SDK 사용 준비

**Azure AI 비전** 서비스는 기본적인 얼굴 감지 기능을 제공(기타 여러 이미지 분석 기능도 제공함)하는 반면 **Face** 서비스에서는 얼굴 분석 및 인식을 위한 더욱 포괄적인 기능을 제공합니다.

1. `cd ../face-api` 명령을 실행하여 **face-api** 폴더로 이동합니다. 그런 다음 언어 기본 설정에 적합한 명령을 실행하여 Face SDK 패키지를 설치합니다.

    **C#**

    ```
    dotnet add package Azure.AI.Vision.Face -v 1.0.0-beta.2
    ```

    **Python**

    ```
    pip install azure-ai-vision-face==1.0.0b2
    ```
    
1. **face-api** 폴더의 내용을 표시하여 구성 설정용 파일이 포함되어 있음을 확인합니다.
    - **C#**: appsettings.json
    - **Python**: .env

1. 구성 파일을 열고 Azure AI 서비스 리소스용 **엔드포인트** 및 인증 **키**를 반영하여 해당 파일에 포함된 구성 값을 업데이트합니다. 변경 내용을 저장합니다.

1. **face-api** 폴더에는 클라이언트 애플리케이션용 코드 파일이 포함되어 있습니다.

    - **C#**: Program.cs
    - **Python**: analyze-faces.py

1. 코드 파일을 열고 파일 맨 윗부분의 기존 네임스페이스 참조 아래에 있는 **네임스페이스 가져오기** 주석을 찾습니다. 그런 다음 이 주석 아래에 다음 언어별 코드를 추가하여 Vision SDK를 사용하는 데 필요한 네임스페이스를 가져옵니다.

    **C#**

    ```C#
    // Import namespaces
    using Azure;
    using Azure.AI.Vision.Face;
    using SkiaSharp;
    ```

    **Python**

    ```Python
    # Import namespaces
    from azure.ai.vision.face import FaceClient
    from azure.ai.vision.face.models import FaceDetectionModel, FaceRecognitionModel, FaceAttributeTypeDetection03
    from azure.core.credentials import AzureKeyCredential
    ```

1. **Main** 함수에서 구성 설정 로드를 위한 코드가 제공되어 있음을 확인합니다. 그런 다음 **Face 클라이언트 인증** 주석을 찾습니다. 그 후에 이 주석 아래에 다음 언어별 코드를 추가하여 **FaceClient** 개체를 만들고 인증합니다.

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

1. **Main** 함수의 방금 추가한 코드 아래에 있는 코드가 메뉴를 표시함을 확인합니다. 이 메뉴를 사용하면 코드의 함수를 호출하여 Face 서비스 기능을 살펴볼 수 있습니다. 이 연습의 나머지 부분에서 이러한 함수를 구현합니다.

## 얼굴 감지 및 분석

Face 서비스의 가장 기본적인 기능 중 하나는 이미지에서 얼굴을 감지하고 해당 특성을 확인하는 것입니다. 이러한 특성으로는 머리 자세, 흐릿한 형체, 마스크 유무 등이 있습니다.

1. 애플리케이션용 코드 파일의 **Main** 함수에서 사용자가 메뉴 옵션 **1**을 선택하면 실행되는 코드를 살펴봅니다. 이 코드는 **DetectFaces** 함수를 호출하여 이미지 파일의 경로를 전달합니다.
1. 코드 파일에서 **DetectFaces** 함수를 찾은 다음 **검색할 얼굴 기능 지정** 주석 아래에 다음 코드를 추가합니다.

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

1. **DetectFaces** 함수의 방금 추가한 코드 아래에서 **얼굴 가져오기** 주석을 찾은 후 다음 코드를 추가합니다.

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

            // Load the image using SkiaSharp
            using SKBitmap bitmap = SKBitmap.Decode(imageFile);
            using SKCanvas canvas = new SKCanvas(bitmap);

            // Set up paint styles for drawing
            SKPaint rectPaint = new SKPaint
            {
                Color = SKColors.LightGreen,
                StrokeWidth = 3,
                Style = SKPaintStyle.Stroke,
                IsAntialias = true
            };

            SKPaint textPaint = new SKPaint
            {
                Color = SKColors.White,
                TextSize = 16,
                IsAntialias = true
            };

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

                // Create an SKRect from the face rectangle data
                SKRect rect = new SKRect(r.Left, r.Top, r.Left + r.Width, r.Top + r.Height);
                canvas.DrawRect(rect, rectPaint);

                string annotation = $"Face number {faceCount}";
                canvas.DrawText(annotation, r.Left, r.Top, textPaint);
            }

            // Save annotated image
            using (SKFileWStream output = new SKFileWStream("detected_faces.jpg"))
            {
                bitmap.Encode(output, SKEncodedImageFormat.Jpeg, 100);
            }

            Console.WriteLine(" Results saved in detected_faces.jpg");   
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

1. **DetectFaces** 함수에 추가한 코드를 살펴봅니다. 이미지 파일을 분석하고 머리 자세, 흐릿한 형체 및 마스크 유무 특성 등 이미지 파일에 포함된 모든 얼굴을 감지합니다. 각 얼굴에 할당되는 고유 얼굴 식별자를 비롯한 각 얼굴의 세부 정보가 표시됩니다. 그리고 경계 상자를 사용하여 이미지상의 얼굴 위치가 표시됩니다.
1. 변경 내용을 저장하고 코드 편집기를 닫고 다음 명령을 입력하여 프로그램을 실행합니다.

    **C#**

    ```
    dotnet run
    ```

    *C# 출력에 **await** 연산자를 사용하는 비동기 함수 관련 경고가 표시될 수 있습니다. 이러한 경고는 이러한 메시지는 무시해도 됩니다.*

    **Python**

    ```
    python analyze-faces.py
    ```

1. 메시지가 표시되면 **1**을 입력하고 출력을 살펴봅니다. 출력에는 감지된 각 얼굴의 ID와 특성이 포함되어 있습니다.
1. 코드 파일과 같은 폴더에 생성된 **detected_faces.jpg** 파일을 다운로드하여 주석이 추가된 얼굴을 확인합니다.

## 리소스 정리

다른 학습 모듈을 위해 이 랩에서 만들어진 Azure 리소스를 사용하지 않는 경우 해당 리소스를 삭제하여 추가 요금이 발생하지 않도록 할 수 있습니다.

1. `https://portal.azure.com`에서 Azure Portal을 열고 상단 검색 창에서 이 랩에서 만든 리소스를 검색합니다.

1. 리소스 페이지에서 **삭제**를 선택하고 지침에 따라 리소스를 삭제합니다. 또는 전체 리소스 그룹을 삭제하여 모든 리소스를 동시에 정리할 수 있습니다.

## 자세한 정보

**Face** 서비스 내에서 사용할 수 있는 몇 가지 추가 기능이 있지만 [요구되는 AI 표준](https://aka.ms/aah91ff)에 따라 제한된 액세스 정책 하에 제한됩니다. 이러한 기능에는 얼굴 인식 모델 식별, 검증 및 생성이 있습니다. 자세한 내용을 알아보고 액세스를 신청하려면 [Azure AI 서비스에 대한 제한된 액세스](https://docs.microsoft.com/en-us/azure/cognitive-services/cognitive-services-limited-access)를 참조하세요.

얼굴 감지를 위한 **Azure AI 비전** 서비스 사용에 대한 자세한 내용은 [Azure AI 비전 설명서](https://docs.microsoft.com/azure/cognitive-services/computer-vision/concept-detecting-faces)를 참조하세요.

**Face** 서비스에 대해 자세히 알아보려면 [Face 설명서](https://learn.microsoft.com/azure/ai-services/computer-vision/overview-identity)를 참조하세요.
