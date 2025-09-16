---
lab:
  title: 이미지에서 텍스트 읽기
  module: Module 11 - Reading Text in Images and Documents
---

# 이미지에서 텍스트 읽기

OCR(광학 인식)은 이미지와 문서에서 텍스트 읽기를 처리하는 Computer Vision 하위 서비스입니다. **Azure AI 비전** 서비스는 텍스트를 읽기 위한 API를 제공하며, 이 연습에서 살펴볼 것입니다.

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
   cd mslearn-ai-vision/Labfiles/05-ocr
    ```

## Azure AI 비전 SDK 사용 준비

이 연습에서는 Azure AI 비전 SDK를 사용해 텍스트를 읽는 부분 구현 클라이언트 애플리케이션을 완성합니다.

> **참고**: **C#** 또는 **Python**용 SDK 사용을 선택할 수 있습니다. 아래 단계에서 선호하는 언어에 적합한 작업을 수행하세요.

1. 기본 설정 언어에 대한 애플리케이션 코드 파일이 포함된 폴더로 이동합니다.  

    **C#**

    ```
   cd C-Sharp/read-text
    ```
    
    **Python**

    ```
   cd Python/read-text
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
1. 자리 표시자를 바꾼 후 코드 편집기에서 **CTRL+S** 명령 또는 **마우스 오른쪽 단추 클릭 > 저장**을 사용하여 변경 내용을 저장한 다음 **CTRL+Q** 명령 또는 **마우스 오른쪽 단추 클릭 > 종료**를 사용하여 Cloud Shell 명령줄을 열어둔 채 코드 편집기를 닫습니다.

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
    using SkiaSharp;
    ```
    
    **Python**
    
    ```Python
    # import namespaces
    from azure.ai.vision.imageanalysis import ImageAnalysisClient
    from azure.ai.vision.imageanalysis.models import VisualFeatures
    from azure.core.credentials import AzureKeyCredential
    ```

1. 클라이언트 애플리케이션용 코드 파일의 **Main** 함수에서 구성 설정 로드를 위한 코드가 제공되었습니다. 그런 다음 **Azure AI 비전 클라이언트 인증** 주석을 찾습니다. 그런 다음, 이 주석 아래에 다음 언어별 코드를 추가하여 Azure AI 비전 클라이언트 개체를 만들고 인증합니다.

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

1. **Main** 함수에서 방금 추가한 코드 아래에서 코드는 이미지 파일의 경로를 지정한 다음 이미지 경로를 **GetTextRead** 함수에 전달합니다. 이 함수는 아직 완전히 구현되지 않았습니다.

1. **GetTextRead** 함수의 본문에 일부 코드를 추가해 보겠습니다. **이미지 분석 기능을 사용하여 이미지의 텍스트 읽기**라는 주석을 찾습니다. 그런 다음, 이 주석 아래에 다음 언어별 코드를 추가하여 `Analyze` 함수를 호출할 때 시각적 기능이 지정됨을 확인합니다.

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
    
        // Load the image using SkiaSharp
        using SKBitmap bitmap = SKBitmap.Decode(imageFile);
        // Create canvas to draw on the bitmap
        using SKCanvas canvas = new SKCanvas(bitmap);

        // Create paint for drawing polygons (bounding boxes)
        SKPaint paint = new SKPaint
        {
            Color = SKColors.Cyan,
            StrokeWidth = 3,
            Style = SKPaintStyle.Stroke,
            IsAntialias = true
        };

        foreach (var line in result.Read.Blocks.SelectMany(block => block.Lines))
        {
            // Return the text detected in the image
    
    
        }
            
        // Save the annotated image using SkiaSharp
        using (SKFileWStream output = new SKFileWStream("text.jpg"))
        {
            // Encode the bitmap into JPEG format with full quality (100)
            bitmap.Encode(output, SKEncodedImageFormat.Jpeg, 100);
        }

        Console.WriteLine("\nResults saved in text.jpg\n");
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

1. **GetTextRead** 함수에서 방금 추가한 코드의 **이미지에서 검색된 텍스트 반환** 주석 아래에 다음 코드를 추가합니다(이 코드는 이미지 텍스트를 콘솔에 인쇄하고 이미지의 텍스트를 강조 표시하는 이미지 **text.jpg**를 생성함).

    **C#**
    
    ```C#
    // Return the text detected in the image
    Console.WriteLine($"   '{line.Text}'");
    
    // Draw bounding box around line
    bool drawLinePolygon = true;
    
    // Return the position bounding box around each line
    
    
    
    // Return each word detected in the image and the position bounding box around each word with the confidence level of each word
    
    
    
    // Draw line bounding polygon
    if (drawLinePolygon)
    {
        var r = line.BoundingPolygon;
        SKPoint[] polygonPoints = new SKPoint[]
        {
            new SKPoint(r[0].X, r[0].Y),
            new SKPoint(r[1].X, r[1].Y),
            new SKPoint(r[2].X, r[2].Y),
            new SKPoint(r[3].X, r[3].Y)
        };

    DrawPolygon(canvas, polygonPoints, paint);
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

1. C# 프로그램 파일의 경우에만 다각형을 그리려면 도우미 함수가 필요합니다. **Helper method to draw a polygon given an array of SKPoints** 주석 아래에 다음 코드를 추가합니다.

    **C#**
   
    ```C#
    // Helper method to draw a polygon given an array of SKPoints
    static void DrawPolygon(SKCanvas canvas, SKPoint[] points, SKPaint paint)
    {
        if (points == null || points.Length == 0)
            return;

        using (var path = new SKPath())
        {
            path.MoveTo(points[0]);
            for (int i = 1; i < points.Length; i++)
            {
                path.LineTo(points[i]);
            }
            path.Close();
            canvas.DrawPath(path, paint);
        }
    }
    ```

1. **Main** 함수에서 사용자가 메뉴 옵션 **1**을 선택하는 경우 실행되는 코드를 검사합니다. 이 코드는 **GetTextRead** 함수를 호출하여 *Lincoln.jpg* 이미지 파일의 경로를 전달합니다.

1. 변경 내용을 저장하고 코드 편집기를 닫습니다.

1. Cloud Shell 도구 모음에서 **파일 업로드/다운로드**를 선택한 다음, **다운로드**를 선택합니다. 새 대화 상자에서 다음 파일 경로를 입력하고 **다운로드**를 선택합니다.

    **C#**
   
    ```
    mslearn-ai-vision/Labfiles/05-ocr/C-Sharp/read-text/images/Lincoln.jpg
    ```

    **Python**

    ```
    mslearn-ai-vision/Labfiles/05-ocr/Python/read-text/images/Lincoln.jpg
    ```
       
1. **Lincoln.jpg** 이미지를 열어서 봅니다.

1. 코드가 처리하는 이미지를 본 후 다음 명령을 입력하여 프로그램을 실행합니다.

    **C#**
    
    ```
    dotnet run
    ```
    
    **Python**
    
    ```
    python read-text.py
    ```

1. 메시지가 표시되면 **1**을 입력하고 출력을 살펴봅니다. 출력은 이미지에서 추출된 텍스트입니다.

1. **read-text** 폴더에 **text.jpg** 이미지가 생성되었습니다. 파일 경로 `mslearn-ai-vision/Labfiles/05-ocr/***C-Sharp or Python***/read-text/text.jpg`를 사용하여 다운로드하고 어떻게 각 텍스트 *줄* 주위에 다각형이 표시되는지 확인할 수 있습니다.

1. 코드 파일을 다시 열고 **Return the position bounding box around each line** 주석을 찾습니다. 그런 다음, 이 주석 아래에 다음 코드를 추가합니다.

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

1. 변경 내용을 저장하고 코드 편집기를 닫고 다음 명령을 입력하여 프로그램을 실행합니다.

    **C#**
    
    ```
    dotnet run
    ```
    
    **Python**
    
    ```
    python read-text.py
    ```

1. 메시지가 표시되면 **1**을 입력하고 출력값을 관찰하며, 이 출력은 이미지의 각 텍스트 줄과 이미지의 해당 위치여야 합니다.

1. 코드 파일을 다시 열고 **Return each word detected in the image and the position bounding box around each word with the confidence level of each word** 주석을 찾습니다. 그런 다음, 이 주석 아래에 다음 코드를 추가합니다.

    **C#**
    
    ```C#
    // Return each word detected in the image and the position bounding box around each word with the confidence level of each word
    foreach (DetectedTextWord word in line.Words)
    {
        Console.WriteLine($"     Word: '{word.Text}', Confidence {word.Confidence:F4}, Bounding Polygon: [{string.Join(" ", word.BoundingPolygon)}]");
        
        // Draw word bounding polygon
        drawLinePolygon = false;
        var r = word.BoundingPolygon;
    
        // Convert the bounding polygon into an array of SKPoints    
        SKPoint[] polygonPoints = new SKPoint[]
        {
            new SKPoint(r[0].X, r[0].Y),
            new SKPoint(r[1].X, r[1].Y),
            new SKPoint(r[2].X, r[2].Y),
            new SKPoint(r[3].X, r[3].Y)
        };

        // Draw the word polygon on the canvas
        DrawPolygon(canvas, polygonPoints, paint);
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

1. 변경 내용을 저장하고 코드 편집기를 닫고 다음 명령을 입력하여 프로그램을 실행합니다.

    **C#**
    
    ```
    dotnet run
    ```
    
    **Python**
    
    ```
    python read-text.py
    ```

1. 메시지가 표시되면 **1**을 입력하고 출력값을 관찰하며, 이 출력은 이미지의 각 텍스트 단어와 이미지의 해당 위치여야 합니다. 각 단어의 신뢰도 수준도 어떻게 반환되는지 확인합니다.

1. **text.jpg** 이미지를 다시 다운로드하고 어떻게 각 *단어* 주위에 다각형이 표시되는지 확인합니다.

## Azure AI 비전 SDK를 사용하여 이미지에서 필기 텍스트 읽기

이전 연습에서는 이미지에서 잘 정의된 텍스트를 읽었지만 때로는 손으로 쓴 메모나 종이의 텍스트를 읽을 수도 있습니다. 좋은 소식은 **Azure AI 비전 SDK**가 잘 정의된 텍스트를 읽는 데 사용한 것과 동일한 정확한 코드를 사용하여 필기 텍스트를 읽을 수도 있다는 것입니다. 이전 연습과 동일한 코드를 사용하지만 이번에는 다른 이미지를 사용합니다.

1. 파일 경로 `mslearn-ai-vision/Labfiles/05-ocr/***C-Sharp or Python***/read-text/images/Note.jpg`를 사용하여 **Note.jpg**를 다운로드하고 코드가 처리하는 다음 이미지를 확인합니다.

1. 애플리케이션용 코드 파일의 **Main** 함수에서 사용자가 메뉴 옵션 **2**를 선택하면 실행되는 코드를 살펴봅니다. 이 코드는 **GetTextRead** 함수를 호출하여 *Note.jpg* 이미지 파일의 경로를 전달합니다.

1. 터미널에서 다음 명령을 입력하여 프로그램을 실행합니다.

    **C#**
    
    ```
    dotnet run
    ```
    
    **Python**
    
    ```
    python read-text.py
    ```

1. 메시지가 표시되면 **2**를 입력하고 출력을 살펴봅니다. 출력은 노트 이미지에서 추출된 텍스트입니다.

1. **text.jpg** 이미지를 다시 다운로드하고 어떻게 메모의 각 *단어* 주위에 다각형이 표시되는지 확인합니다.

## 리소스 정리

다른 학습 모듈을 위해 이 랩에서 만들어진 Azure 리소스를 사용하지 않는 경우 해당 리소스를 삭제하여 추가 요금이 발생하지 않도록 할 수 있습니다. 이 경우 가능한 방법은 다음과 같습니다.

1. `https://portal.azure.com`의 Azure Portal을 열고 Azure 구독과 연관된 Microsoft 계정을 사용하여 로그인합니다.

1. 위쪽 검색 창에서 *Computer Vision*을 검색하고 이 랩에서 만든 Computer Vision 리소스를 선택합니다.

1. 리소스 페이지에서 **삭제**를 선택하고 지침에 따라 리소스를 삭제합니다.

## 자세한 정보

**Azure AI 비전** 서비스를 사용하여 텍스트를 읽는 방법에 대한 자세한 내용은 [Azure AI 비전 설명서](https://learn.microsoft.com/azure/ai-services/computer-vision/concept-ocr)를 참조하세요.
