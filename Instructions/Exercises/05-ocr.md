---
lab:
  title: 이미지에서 텍스트 읽기
  module: Module 11 - Reading Text in Images and Documents
---

# 이미지에서 텍스트 읽기

OCR(광학 인식)은 이미지와 문서에서 텍스트 읽기를 처리하는 Computer Vision 하위 서비스입니다. **Azure AI 비전** 서비스는 텍스트를 읽기 위한 API를 제공하며, 이 연습에서 살펴볼 것입니다.

## 이 과정용 리포지토리 복제

이 과정용 코드 리포지토리를 아직 복제하지 않았으면 복제해야 합니다.

1. Visual Studio Code 시작
2. 팔레트를 열고(Shift+Ctrl+P) **Git: Clone** 명령을 실행하여 `https://github.com/MicrosoftLearning/mslearn-ai-vision` 리포지토리를 로컬 폴더(아무 폴더나 관계없음)에 복제합니다.
3. 리포지토리가 복제되면 Visual Studio Code에서 폴더를 엽니다.
4. 리포지토리의 C# 코드 프로젝트를 지원하는 추가 파일이 설치되는 동안 기다립니다.

    > **참고**: 빌드 및 디버깅에 필요한 자산을 추가하라는 메시지가 표시되면 **나중에**를 선택합니다. *폴더에서 Azure 함수 프로젝트를 발견했습니다*라는 메시지가 표시되면 해당 메시지를 안전하게 닫을 수 있습니다.

## Azure AI 서비스 리소스 프로비전

구독에 아직 없는 경우 **Azure AI 서비스** 리소스를 프로비전해야 합니다.

1. `https://portal.azure.com`의 Azure Portal을 열고 Azure 구독과 연관된 Microsoft 계정을 사용하여 로그인합니다.
2. 상단 검색 창에서 *Azure AI 서비스*를 검색하고 **Azure AI 서비스**를 선택한 후 다음 설정을 사용하여 Azure AI 서비스 다중 서비스 계정 리소스를 만듭니다.
    - **구독**: ‘Azure 구독’
    - **리소스 그룹**: 리소스 그룹 선택 또는 만들기(제한된 구독을 사용 중이라면 새 리소스 그룹을 만들 권한이 없을 수도 있으므로 제공된 리소스 그룹 사용)**
    - **지역**: *미국 동부, 프랑스 중부, 한국 중부, 북유럽, 동남 아시아, 서유럽, 미국 서부 또는 동아시아 중에서 선택\**
    - **이름**: *고유 이름 입력*
    - **가격 책정 계층**: 표준 S0

    \*Azure AI 비전 4.0 기능은 현재 이 지역에서만 사용할 수 있습니다.

3. 필요한 확인란을 선택하고 리소스를 만듭니다.
4. 배포가 완료될 때까지 기다린 다음, 배포 세부 정보를 봅니다.
5. 리소스가 배포되면 해당 리소스로 이동하여 **키 및 엔드포인트** 페이지를 확인합니다. 다음 절차에서 이 페이지에 표시되는 키 중 하나와 엔드포인트가 필요합니다.

## Azure AI 비전 SDK 사용 준비

이 연습에서는 Azure AI 비전 SDK를 사용해 텍스트를 읽는 부분 구현 클라이언트 애플리케이션을 완성합니다.

> **참고**: **C#** 또는 **Python**용 SDK 사용을 선택할 수 있습니다. 다음 단계에서는 기본 설정 언어에 적합한 작업을 수행합니다.

1. Visual Studio Code의 **탐색기** 창에서 **Labfiles\05-ocr** 폴더를 찾은 다음 언어 기본 설정에 따라 **C-Sharp** 또는 **Python** 폴더를 확장합니다.
2. **read-text** 폴더를 마우스 오른쪽 단추로 클릭하고 통합 터미널을 엽니다. 그런 다음, 언어 선택에 적절한 명령을 실행하여 Azure AI 비전 SDK 패키지를 설치합니다.

    **C#**
    
    ```
    dotnet add package Azure.AI.Vision.ImageAnalysis -v 1.0.0-beta.1
    ```

    > **참고**: 개발 키트 확장을 설치하라는 메시지가 표시되면 메시지를 안전하게 닫을 수 있습니다.

    **Python**
    
    ```
    pip install azure-ai-vision-imageanalysis==1.0.0b1
    ```

3. **read-text** 폴더의 내용을 표시하여 구성 설정용 파일이 포함되어 있음을 확인합니다.

    - **C#**: appsettings.json
    - **Python**: .env

    구성 파일을 열고 Azure AI 서비스 리소스용 **엔드포인트** 및 인증 **키**를 반영하여 해당 파일에 포함된 구성 값을 업데이트합니다. 변경 내용을 저장합니다.


## Azure AI 비전 SDK를 사용하여 이미지에서 텍스트 읽기

**Azure AI 비전 SDK**의 기능 중 하나는 이미지에서 텍스트를 읽는 것입니다. 이 연습에서는 Azure AI 비전 SDK를 사용하여 이미지에서 텍스트를 읽는 부분 구현 클라이언트 애플리케이션을 완료합니다.

1. **read-text** 폴더에는 클라이언트 애플리케이션용 코드 파일이 포함되어 있습니다.

    - **C#**: Program.cs
    - **Python**: read-text.py

    코드 파일을 열고 파일 맨 윗부분의 기존 네임스페이스 참조 아래에 있는 **네임스페이스 가져오기** 주석을 찾습니다. 그런 다음 이 주석 아래에 다음 언어별 코드를 추가하여 Azure AI 비전 SDK를 사용하는 데 필요한 네임스페이스를 가져옵니다.

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

2. 클라이언트 애플리케이션용 코드 파일의 **Main** 함수에서 구성 설정 로드를 위한 코드가 제공되었습니다. 그런 다음 **Azure AI 비전 클라이언트 인증** 주석을 찾습니다. 그런 다음, 이 주석 아래에 다음 언어별 코드를 추가하여 Azure AI 비전 클라이언트 개체를 만들고 인증합니다.

    **C#**
    
    ```C#
    // Authenticate Azure AI Vision client
    ImageAnalysisClient client = new ImageAnalysisClient(
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

3. **Main** 함수에서 방금 추가한 코드 아래에서 코드는 이미지 파일의 경로를 지정한 다음 이미지 경로를 **GetTextRead** 함수에 전달합니다. 이 함수는 아직 완전히 구현되지 않았습니다.

4. **GetTextRead** 함수의 본문에 일부 코드를 추가해 보겠습니다. **이미지 분석 기능을 사용하여 이미지의 텍스트 읽기**라는 주석을 찾습니다. 그런 다음, 이 주석 아래에 다음 언어별 코드를 추가하여 `Analyze` 함수를 호출할 때 시각적 기능이 지정됨을 확인합니다.

    **C#**

    ```C#
    // Use Analyze image function to read text in image
    ImageAnalysisResult result = client.Analyze(
        BinaryData.FromStream(stream),
        // Specify the features to be retrieved
        VisualFeatures.Read);
    
    stream.Close();
    
    // Display analysis results
    if (result.Read != null)
    {
        Console.WriteLine($"Text:");
    
        // Prepare image for drawing
        System.Drawing.Image image = System.Drawing.Image.FromFile(imageFile);
        Graphics graphics = Graphics.FromImage(image);
        Pen pen = new Pen(Color.Cyan, 3);
        
        foreach (var line in result.Read.Blocks.SelectMany(block => block.Lines))
        {
            // Return the text detected in the image
    
    
        }
            
        // Save image
        String output_file = "text.jpg";
        image.Save(output_file);
        Console.WriteLine("\nResults saved in " + output_file + "\n");   
    }
    ```
    
    **Python**
    
    ```Python
    # Use Analyze image function to read text in image
    result = cv_client.analyze(
        image_data=image_data,
        visual_features=[VisualFeatures.READ]
    )

    # Display the image and overlay it with the extracted text
    if result.read is not None:
        print("\nText:")

        # Prepare image for drawing
        image = Image.open(image_file)
        fig = plt.figure(figsize=(image.width/100, image.height/100))
        plt.axis('off')
        draw = ImageDraw.Draw(image)
        color = 'cyan'

        for line in result.read.blocks[0].lines:
            # Return the text detected in the image

            
        # Save image
        plt.imshow(image)
        plt.tight_layout(pad=0)
        outputfile = 'text.jpg'
        fig.savefig(outputfile)
        print('\n  Results saved in', outputfile)
    ```

5. **GetTextRead** 함수에서 방금 추가한 코드의 **이미지에서 검색된 텍스트 반환** 주석 아래에 다음 코드를 추가합니다(이 코드는 이미지 텍스트를 콘솔에 인쇄하고 이미지의 텍스트를 강조 표시하는 이미지 **text.jpg**를 생성함).

    **C#**
    
    ```C#
    // Return the text detected in the image
    Console.WriteLine($"   '{line.Text}'");
    
    // Draw bounding box around line
    var drawLinePolygon = true;
    
    // Return each line detected in the image and the position bounding box around each line
    
    
    
    // Return each word detected in the image and the position bounding box around each word with the confidence level of each word
    
    
    
    // Draw line bounding polygon
    if (drawLinePolygon)
    {
        var r = line.BoundingPolygon;
    
        Point[] polygonPoints = {
            new Point(r[0].X, r[0].Y),
            new Point(r[1].X, r[1].Y),
            new Point(r[2].X, r[2].Y),
            new Point(r[3].X, r[3].Y)
        };
    
        graphics.DrawPolygon(pen, polygonPoints);
    }
    ```
    
    **Python**
    
    ```Python
    # Return the text detected in the image
    print(f"  {line.text}")    
    
    drawLinePolygon = True
    
    r = line.bounding_polygon
    bounding_polygon = ((r[0].x, r[0].y),(r[1].x, r[1].y),(r[2].x, r[2].y),(r[3].x, r[3].y))
    
    # Return the position bounding box around each line
    
    
    # Return each word detected in the image and the position bounding box around each word with the confidence level of each word
    
    
    # Draw line bounding polygon
    if drawLinePolygon:
        draw.polygon(bounding_polygon, outline=color, width=3)
    ```

6. **read-text/images** 폴더에서 **Lincoln.jpg**를 선택하여 코드가 처리하는 파일을 봅니다.

7. 애플리케이션용 코드 파일의 **Main** 함수에서 사용자가 메뉴 옵션 **1**을 선택하면 실행되는 코드를 살펴봅니다. 이 코드는 **GetTextRead** 함수를 호출하여 *Lincoln.jpg* 이미지 파일의 경로를 전달합니다.

8. 변경 내용을 저장하고 **read-text** 폴더의 통합 터미널로 돌아와서 다음 명령을 입력하여 프로그램을 실행합니다.

    **C#**
    
    ```
    dotnet run
    ```
    
    **Python**
    
    ```
    python read-text.py
    ```

9. 메시지가 표시되면 **1**을 입력하고 출력을 살펴봅니다. 출력은 이미지에서 추출된 텍스트입니다.

10. **read-text** 폴더에서 **text.jpg** 이미지를 선택하고 각 텍스트 *줄* 주위에 다각형이 있는 것을 확인했습니다.

11. Visual Studio Code의 코드 파일로 돌아가서 **각 줄 주위의 위치 경계 상자 반환** 주석을 찾습니다. 그런 다음, 이 주석 아래에 다음 코드를 추가합니다.

    **C#**
    
    ```C#
    // Return the position bounding box around each line
    Console.WriteLine($"   Bounding Polygon: [{string.Join(" ", line.BoundingPolygon)}]");  
    ```
    
    **Python**
    
    ```Python
    # Return the position bounding box around each line
    print("   Bounding Polygon: {}".format(bounding_polygon))
    ```

12. 변경 내용을 저장하고 **read-text** 폴더의 통합 터미널로 돌아와서 다음 명령을 입력하여 프로그램을 실행합니다.

    **C#**
    
    ```
    dotnet run
    ```
    
    **Python**
    
    ```
    python read-text.py
    ```

13. 메시지가 표시되면 **1**을 입력하고 출력값을 관찰하며, 이 출력은 이미지의 각 텍스트 줄과 이미지의 해당 위치여야 합니다.


14. Visual Studio Code의 코드 파일로 돌아가서 **이미지에서 감지된 각 단어와 각 단어의 신뢰도 수준을 사용하여 각 단어 주위의 위치 경계 상자 반환** 주석을 찾습니다. 그런 다음, 이 주석 아래에 다음 코드를 추가합니다.

    **C#**
    
    ```C#
    // Return each word detected in the image and the position bounding box around each word with the confidence level of each word
    foreach (DetectedTextWord word in line.Words)
    {
        Console.WriteLine($"     Word: '{word.Text}', Confidence {word.Confidence:F4}, Bounding Polygon: [{string.Join(" ", word.BoundingPolygon)}]");
        
        // Draw word bounding polygon
        drawLinePolygon = false;
        var r = word.BoundingPolygon;
    
        Point[] polygonPoints = {
            new Point(r[0].X, r[0].Y),
            new Point(r[1].X, r[1].Y),
            new Point(r[2].X, r[2].Y),
            new Point(r[3].X, r[3].Y)
        };
    
        graphics.DrawPolygon(pen, polygonPoints);
    }
    ```
    
    **Python**
    
    ```Python
    # Return each word detected in the image and the position bounding box around each word with the confidence level of each word
    for word in line.words:
        r = word.bounding_polygon
        bounding_polygon = ((r[0].x, r[0].y),(r[1].x, r[1].y),(r[2].x, r[2].y),(r[3].x, r[3].y))
        print(f"    Word: '{word.text}', Bounding Polygon: {bounding_polygon}, Confidence: {word.confidence:.4f}")
    
        # Draw word bounding polygon
        drawLinePolygon = False
        draw.polygon(bounding_polygon, outline=color, width=3)
    ```

15. 변경 내용을 저장하고 **read-text** 폴더의 통합 터미널로 돌아와서 다음 명령을 입력하여 프로그램을 실행합니다.

    **C#**
    
    ```
    dotnet run
    ```
    
    **Python**
    
    ```
    python read-text.py
    ```

16. 메시지가 표시되면 **1**을 입력하고 출력값을 관찰하며, 이 출력은 이미지의 각 텍스트 단어와 이미지의 해당 위치여야 합니다. 각 단어의 신뢰도 수준도 어떻게 반환되는지 확인합니다.

17. **read-text** 폴더에서 **text.jpg** 이미지를 선택하고 각 *단어* 주위에 다각형이 있는 것을 확인했습니다.

## Azure AI 비전 SDK를 사용하여 이미지에서 필기 텍스트 읽기

이전 연습에서는 이미지에서 잘 정의된 텍스트를 읽었지만 때로는 손으로 쓴 메모나 종이의 텍스트를 읽을 수도 있습니다. 좋은 소식은 **Azure AI 비전 SDK**가 잘 정의된 텍스트를 읽는 데 사용한 것과 동일한 정확한 코드를 사용하여 필기 텍스트를 읽을 수도 있다는 것입니다. 이전 연습과 동일한 코드를 사용하지만 이번에는 다른 이미지를 사용합니다.

1. **read-text/images** 폴더에서 **Note.jpg**를 선택하여 코드가 처리하는 파일을 봅니다.

2. 애플리케이션용 코드 파일의 **Main** 함수에서 사용자가 메뉴 옵션 **2**를 선택하면 실행되는 코드를 살펴봅니다. 이 코드는 **GetTextRead** 함수를 호출하여 *Note.jpg* 이미지 파일의 경로를 전달합니다.

3. **read-text** 폴더의 통합 터미널에서 다음 명령을 입력하여 프로그램을 실행합니다.

    **C#**
    
    ```
    dotnet run
    ```
    
    **Python**
    
    ```
    python read-text.py
    ```

4. 메시지가 표시되면 **2**를 입력하고 출력을 살펴봅니다. 출력은 노트 이미지에서 추출된 텍스트입니다.

5. **read-text** 폴더에서 **text.jpg** 이미지를 선택하고 노트의 각 *단어* 주위에 다각형이 있는 것을 확인했습니다.

## 리소스 정리

다른 학습 모듈을 위해 이 랩에서 만들어진 Azure 리소스를 사용하지 않는 경우 해당 리소스를 삭제하여 추가 요금이 발생하지 않도록 할 수 있습니다. 이 경우 가능한 방법은 다음과 같습니다.

1. `https://portal.azure.com`의 Azure Portal을 열고 Azure 구독과 연관된 Microsoft 계정을 사용하여 로그인합니다.

2. 상단 검색 창에서 *Azure AI 서비스 다중 서비스 계정*을 검색하고 이 랩에서 만든 Azure AI 서비스 다중 서비스 계정 리소스를 선택합니다.

3. 리소스 페이지에서 **삭제**를 선택하고 지침에 따라 리소스를 삭제합니다.

## 자세한 정보

**Azure AI 비전** 서비스를 사용하여 텍스트를 읽는 방법에 대한 자세한 내용은 [Azure AI 비전 설명서](https://learn.microsoft.com/azure/ai-services/computer-vision/concept-ocr)를 참조하세요.
