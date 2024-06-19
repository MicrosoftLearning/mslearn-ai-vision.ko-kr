---
lab:
  title: Azure AI 비전으로 이미지 분석
  module: Module 2 - Develop computer vision solutions with Azure AI Vision
---

# Azure AI 비전으로 이미지 분석

Azure AI 비전은 소프트웨어 시스템이 이미지를 분석하여 시각적 입력을 해석하는 데 사용할 수 있는 인공 지능 기능입니다. Microsoft Azure에서 **Vision** Azure AI 서비스는 캡션과 태그를 제안하는 이미지 분석, 공통 개체 및 사람 검색 등 일반적인 Computer Vision 작업을 위해 사전 빌드된 모델을 제공합니다. Azure AI 비전 서비스를 사용하여 백그라운드를 제거하거나 이미지의 포그라운드 매트를 만들 수도 있습니다.

## 이 과정용 리포지토리 복제

이 랩에서 작업 중인 환경에 아직 **Azure AI 비전** 코드 리포지토리를 복제하지 않은 경우 다음 단계에 따라 복제합니다. 리포지토리를 복제한 경우에는 Visual Studio Code에서 복제한 폴더를 엽니다.

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

이 연습에서는 Azure AI 비전 SDK를 사용하여 이미지를 분석하는 부분적으로 구현된 클라이언트 애플리케이션을 완료합니다.

> **참고**: **C#** 또는 **Python**용 SDK 사용을 선택할 수 있습니다. 아래 단계에서 선호하는 언어에 적합한 작업을 수행하세요.

1. Visual Studio Code의 **탐색기** 창에서 **Labfiles/01-analyze-images** 폴더를 찾은 다음 언어 선택에 따라 **C-Sharp** 또는 **Python** 폴더를 확장합니다.
2. **image-analysis** 폴더를 마우스 오른쪽 단추로 클릭하고 통합 터미널을 엽니다. 그런 다음, 언어 선택에 적절한 명령을 실행하여 Azure AI 비전 SDK 패키지를 설치합니다.

    **C#**
    
    ```
    dotnet add package Azure.AI.Vision.ImageAnalysis -v 1.0.0-beta.1
    ```

    > **참고**: 개발 키트 확장을 설치하라는 메시지가 표시되면 메시지를 안전하게 닫을 수 있습니다.

    **Python**
    
    ```
    pip install azure-ai-vision-imageanalysis==1.0.0b1
    ```
    
3. **image-analysis** 폴더의 내용을 표시하여 구성 설정용 파일이 포함되어 있음을 확인합니다.
    - **C#**: appsettings.json
    - **Python**: .env

    구성 파일을 열고 Azure AI 서비스 리소스용 **엔드포인트** 및 인증 **키**를 반영하여 해당 파일에 포함된 구성 값을 업데이트합니다. 변경 내용을 저장합니다.
4. **image-analysis** 폴더에는 클라이언트 애플리케이션용 코드 파일이 포함되어 있습니다.

    - **C#**: Program.cs
    - **Python**: image-analysis.py

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
    
## 분석할 이미지 확인

이 연습에서는 Azure AI 비전 서비스를 사용하여 여러 이미지를 분석합니다.

1. Visual Studio Code에서 **image-analysis** 폴더와 이 폴더에 포함된 **images** 폴더를 차례로 확장합니다.
2. 각 이미지 파일을 차례로 선택하여 Visual Studio Code에 표시합니다.

## 이미지를 분석하여 캡션 추천

이제 SDK를 사용하여 Vision 서비스를 호출하고 이미지를 분석할 준비가 되었습니다.

1. 클라이언트 애플리케이션용 코드 파일(**Program.cs** 또는 **image-analysis.py**)의 **Main** 함수에서 구성 설정 로드를 위한 코드가 제공되어 있음을 확인합니다. 그런 다음 **Azure AI Vision 클라이언트 인증** 주석을 찾습니다. 그런 다음, 이 주석 아래에 다음 언어별 코드를 추가하여 Azure AI 비전 클라이언트 개체를 만들고 인증합니다.

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

2. **Main** 함수의 방금 추가한 코드 아래에 있는 코드가 이미지 파일 경로를 지정한 다음 다른 2개 함수(**AnalyzeImage** 및 **BackgroundForeground**)에 이미지 경로를 전달함을 확인합니다. 이러한 함수는 아직 완전히 구현되지 않았습니다.

3. **AnalyzeImage** 함수의 **검색할 함수를 지정하여 결과 가져오기** 주석 아래에 다음 코드를 추가합니다.

**C#**

```C#
// Get result with specified features to be retrieved
ImageAnalysisResult result = client.Analyze(
    BinaryData.FromStream(stream),
    VisualFeatures.Caption | 
    VisualFeatures.DenseCaptions |
    VisualFeatures.Objects |
    VisualFeatures.Tags |
    VisualFeatures.People);
```

**Python**

```Python
# Get result with specified features to be retrieved
result = cv_client.analyze(
    image_data=image_data,
    visual_features=[
        VisualFeatures.CAPTION,
        VisualFeatures.DENSE_CAPTIONS,
        VisualFeatures.TAGS,
        VisualFeatures.OBJECTS,
        VisualFeatures.PEOPLE],
)
```
    
4. **AnalyzeImage** 함수의 **분석 결과 표시** 주석 아래에 다음 코드를 추가합니다(나중에 코드를 추가할 위치를 나타내는 주석 포함).

**C#**

```C#
// Display analysis results
// Get image captions
if (result.Caption.Text != null)
{
    Console.WriteLine(" Caption:");
    Console.WriteLine($"   \"{result.Caption.Text}\", Confidence {result.Caption.Confidence:0.00}\n");
}

// Get image dense captions
Console.WriteLine(" Dense Captions:");
foreach (DenseCaption denseCaption in result.DenseCaptions.Values)
{
    Console.WriteLine($"   Caption: '{denseCaption.Text}', Confidence: {denseCaption.Confidence:0.00}");
}

// Get image tags


// Get objects in the image


// Get people in the image
```

**Python**

```Python
# Display analysis results
# Get image captions
if result.caption is not None:
    print("\nCaption:")
    print(" Caption: '{}' (confidence: {:.2f}%)".format(result.caption.text, result.caption.confidence * 100))

# Get image dense captions
if result.dense_captions is not None:
    print("\nDense Captions:")
    for caption in result.dense_captions.list:
        print(" Caption: '{}' (confidence: {:.2f}%)".format(caption.text, caption.confidence * 100))

# Get image tags


# Get objects in the image


# Get people in the image

```
    
5. 변경 내용을 저장하고 **image-analysis** 폴더의 통합 터미널로 돌아옵니다. 그런 후에 다음 명령을 입력하여 **images/street.jpg** 인수를 사용해 프로그램을 실행합니다.

**C#**

```
dotnet run images/street.jpg
```

**Python**

```
python image-analysis.py images/street.jpg
```
    
6. 출력을 살펴봅니다. **street.jpg** 이미지에 대해 제한된 캡션이 포함되어 있습니다.
7. 이번에는 **images/building.jpg** 인수를 사용하여 프로그램을 다시 실행해 **building.jpg** 이미지에 대해 생성되는 캡션을 확인합니다.
8. 이전 단계를 반복하여 **images/person.jpg** 파일에 대한 캡션을 생성합니다.

## 이미지의 추천 캡션 가져오기

이미지 내용에 대한 단서를 제공하는 관련 *태그*를 식별하면 유용한 경우가 있습니다.

1. **AnalyzeImage** 함수의 **이미지 태그 가져오기** 주석 아래에 다음 코드를 추가합니다.

**C#**

```C#
// Get image tags
if (result.Tags.Values.Count > 0)
{
    Console.WriteLine($"\n Tags:");
    foreach (DetectedTag tag in result.Tags.Values)
    {
        Console.WriteLine($"   '{tag.Name}', Confidence: {tag.Confidence:F2}");
    }
}
```

**Python**

```Python
# Get image tags
if result.tags is not None:
    print("\nTags:")
    for tag in result.tags.list:
        print(" Tag: '{}' (confidence: {:.2f}%)".format(tag.name, tag.confidence * 100))
```

2. 변경 내용을 저장한 다음 **images** 이미지 폴더의 각 이미지 파일별로 프로그램을 한 번씩 실행합니다. 프로그램을 실행할 때마다 이미지 캡션 외에 추천 태그 목록도 표시됨을 확인합니다.

## 이미지에서 개체 감지 및 찾기

*개체 감지*는 이미지 내의 개별 개체를 식별하고 해당 위치를 경계 상자로 표시하는 특정 형식의 Computer Vision 기능입니다.

1. **AnalyzeImage** 함수의 **이미지에서 개체 가져오기** 주석 아래에 다음 코드를 추가합니다.

**C#**

```C#
// Get objects in the image
if (result.Objects.Values.Count > 0)
{
    Console.WriteLine(" Objects:");

    // Prepare image for drawing
    stream.Close();
    System.Drawing.Image image = System.Drawing.Image.FromFile(imageFile);
    Graphics graphics = Graphics.FromImage(image);
    Pen pen = new Pen(Color.Cyan, 3);
    Font font = new Font("Arial", 16);
    SolidBrush brush = new SolidBrush(Color.WhiteSmoke);

    foreach (DetectedObject detectedObject in result.Objects.Values)
    {
        Console.WriteLine($"   \"{detectedObject.Tags[0].Name}\"");

        // Draw object bounding box
        var r = detectedObject.BoundingBox;
        Rectangle rect = new Rectangle(r.X, r.Y, r.Width, r.Height);
        graphics.DrawRectangle(pen, rect);
        graphics.DrawString(detectedObject.Tags[0].Name,font,brush,(float)r.X, (float)r.Y);
    }

    // Save annotated image
    String output_file = "objects.jpg";
    image.Save(output_file);
    Console.WriteLine("  Results saved in " + output_file + "\n");
}
```

**Python**

```Python
# Get objects in the image
if result.objects is not None:
    print("\nObjects in image:")

    # Prepare image for drawing
    image = Image.open(image_filename)
    fig = plt.figure(figsize=(image.width/100, image.height/100))
    plt.axis('off')
    draw = ImageDraw.Draw(image)
    color = 'cyan'

    for detected_object in result.objects.list:
        # Print object name
        print(" {} (confidence: {:.2f}%)".format(detected_object.tags[0].name, detected_object.tags[0].confidence * 100))
        
        # Draw object bounding box
        r = detected_object.bounding_box
        bounding_box = ((r.x, r.y), (r.x + r.width, r.y + r.height)) 
        draw.rectangle(bounding_box, outline=color, width=3)
        plt.annotate(detected_object.tags[0].name,(r.x, r.y), backgroundcolor=color)

    # Save annotated image
    plt.imshow(image)
    plt.tight_layout(pad=0)
    outputfile = 'objects.jpg'
    fig.savefig(outputfile)
    print('  Results saved in', outputfile)
```

2. 변경 내용을 저장한 다음 **images** 이미지 폴더의 각 이미지 파일별로 프로그램을 한 번씩 실행합니다. 프로그램을 실행할 때마다 감지된 개체를 살펴봅니다. 각 실행 후에는 코드 파일과 같은 폴더에 생성된 **objects.jpg** 파일을 표시하여 주석이 추가된 개체를 확인합니다.

## 이미지에서 사람을 검색하고 찾습니다.

*사람 검색*은 이미지 내의 개별 사람을 식별하고 그 위치를 경계 상자로 표시하는 특정 형태의 Computer Vision입니다.

1. **AnalyzeImage** 함수의 **이미지에서 사람 가져오기** 주석 아래에 다음 코드를 추가합니다.

**C#**

```C#
// Get people in the image
if (result.People.Values.Count > 0)
{
    Console.WriteLine($" People:");

    // Prepare image for drawing
    System.Drawing.Image image = System.Drawing.Image.FromFile(imageFile);
    Graphics graphics = Graphics.FromImage(image);
    Pen pen = new Pen(Color.Cyan, 3);
    Font font = new Font("Arial", 16);
    SolidBrush brush = new SolidBrush(Color.WhiteSmoke);

    foreach (DetectedPerson person in result.People.Values)
    {
        // Draw object bounding box
        var r = person.BoundingBox;
        Rectangle rect = new Rectangle(r.X, r.Y, r.Width, r.Height);
        graphics.DrawRectangle(pen, rect);
        
        // Return the confidence of the person detected
        //Console.WriteLine($"   Bounding box {person.BoundingBox.ToString()}, Confidence: {person.Confidence:F2}");
    }

    // Save annotated image
    String output_file = "persons.jpg";
    image.Save(output_file);
    Console.WriteLine("  Results saved in " + output_file + "\n");
}
```

**Python**

```Python
# Get people in the image
if result.people is not None:
    print("\nPeople in image:")

    # Prepare image for drawing
    image = Image.open(image_filename)
    fig = plt.figure(figsize=(image.width/100, image.height/100))
    plt.axis('off')
    draw = ImageDraw.Draw(image)
    color = 'cyan'

    for detected_people in result.people.list:
        # Draw object bounding box
        r = detected_people.bounding_box
        bounding_box = ((r.x, r.y), (r.x + r.width, r.y + r.height))
        draw.rectangle(bounding_box, outline=color, width=3)

        # Return the confidence of the person detected
        #print(" {} (confidence: {:.2f}%)".format(detected_people.bounding_box, detected_people.confidence * 100))
        
    # Save annotated image
    plt.imshow(image)
    plt.tight_layout(pad=0)
    outputfile = 'people.jpg'
    fig.savefig(outputfile)
    print('  Results saved in', outputfile)
```

2. (선택 사항) 이미지의 특정 위치에서 사람이 검색되어 반환된 신뢰 수준을 검토하려면 **감지된 사람의 신뢰도 반환** 섹션에서 **Console.Writeline** 명령의 주석 처리를 제거합니다.

3. 변경 내용을 저장한 다음 **images** 이미지 폴더의 각 이미지 파일별로 프로그램을 한 번씩 실행합니다. 프로그램을 실행할 때마다 감지된 개체를 살펴봅니다. 각 실행 후에는 코드 파일과 같은 폴더에 생성된 **objects.jpg** 파일을 표시하여 주석이 추가된 개체를 확인합니다.

> **참고**: 이전 작업에서는 메서드 하나를 사용해 이미지를 분석한 다음 코드를 계속 추가하여 결과를 구문 분석하고 표시했습니다. SDK에서는 캡션 추천, 태그 식별, 개체 감지 등에 사용할 수 있는 개별 메서드도 제공하므로 가장 적절한 메서드를 사용하여 필요한 정보만 반환할 수 있습니다. 그러면 반환해야 하는 데이터 페이로드 크기가 줄어듭니다. 자세한 내용은 [.NET SDK 설명서](https://learn.microsoft.com/dotnet/api/overview/azure/cognitiveservices/computervision?view=azure-dotnet) 또는 [Python SDK 설명서](https://learn.microsoft.com/python/api/azure-cognitiveservices-vision-computervision/azure.cognitiveservices.vision.computervision)를 참조하세요.

## 백그라운드를 제거하거나 이미지의 포그라운드 매트를 생성합니다.

어떤 경우에는 이미지의 백그라운드를 제거해야 하거나 해당 이미지의 포그라운드 매트를 만들어야 할 수도 있습니다. 백그라운드 제거부터 시작해 보겠습니다.

1. 코드 파일에서 **BackgroundForeground** 함수를 찾습니다. **이미지에서 백그라운드 제거 또는 포그라운드 매트 생성** 주석 아래에 다음 코드를 추가합니다.

**C#**

```C#
// Remove the background from the image or generate a foreground matte
Console.WriteLine($" Background removal:");
// Define the API version and mode
string apiVersion = "2023-02-01-preview";
string mode = "backgroundRemoval"; // Can be "foregroundMatting" or "backgroundRemoval"

string url = $"computervision/imageanalysis:segment?api-version={apiVersion}&mode={mode}";

// Make the REST call
using (var client = new HttpClient())
{
    var contentType = new MediaTypeWithQualityHeaderValue("application/json");
    client.BaseAddress = new Uri(endpoint);
    client.DefaultRequestHeaders.Accept.Add(contentType);
    client.DefaultRequestHeaders.Add("Ocp-Apim-Subscription-Key", key);

    var data = new
    {
        url = $"https://github.com/MicrosoftLearning/mslearn-ai-vision/blob/main/Labfiles/01-analyze-images/Python/image-analysis/{imageFile}?raw=true"
    };

    var jsonData = JsonSerializer.Serialize(data);
    var contentData = new StringContent(jsonData, Encoding.UTF8, contentType);
    var response = await client.PostAsync(url, contentData);

    if (response.IsSuccessStatusCode) {
        File.WriteAllBytes("background.png", response.Content.ReadAsByteArrayAsync().Result);
        Console.WriteLine("  Results saved in background.png\n");
    }
    else
    {
        Console.WriteLine($"API error: {response.ReasonPhrase} - Check your body url, key, and endpoint.");
    }
}
```

**Python**

```Python
# Remove the background from the image or generate a foreground matte
print('\nRemoving background from image...')
    
url = "{}computervision/imageanalysis:segment?api-version={}&mode={}".format(endpoint, api_version, mode)

headers= {
    "Ocp-Apim-Subscription-Key": key, 
    "Content-Type": "application/json" 
}

image_url="https://github.com/MicrosoftLearning/mslearn-ai-vision/blob/main/Labfiles/01-analyze-images/Python/image-analysis/{}?raw=true".format(image_file)  

body = {
    "url": image_url,
}
    
response = requests.post(url, headers=headers, json=body)

image=response.content
with open("backgroundForeground.png", "wb") as file:
    file.write(image)
print('  Results saved in backgroundForeground.png \n')
```
    
2. 변경 내용을 저장한 다음 **images** 이미지 폴더의 각 이미지 파일별로 프로그램을 한 번씩 실행합니다. 프로그램을 실행할 때마다 각 이미지에 대해 코드 파일과 같은 폴더에 생성되는 **background.png** 파일을 엽니다.  각 이미지에서 백그라운드가 어떻게 제거되었는지 확인합니다.

이제 이미지에 대한 포그라운드 매트를 생성해 보겠습니다.

3. 코드 파일에서 **BackgroundForeground** 함수를 찾습니다. **API 버전 및 모드 정의** 주석 아래에서 모드 변수를 `foregroundMatting`으로 변경합니다.

4. 변경 내용을 저장한 다음 **images** 이미지 폴더의 각 이미지 파일별로 프로그램을 한 번씩 실행합니다. 프로그램을 실행할 때마다 각 이미지에 대해 코드 파일과 같은 폴더에 생성되는 **background.png** 파일을 엽니다.  이미지에 대해 포그라운드 매트가 어떻게 생성되었는지 확인합니다.

## 리소스 정리

다른 학습 모듈을 위해 이 랩에서 만들어진 Azure 리소스를 사용하지 않는 경우 해당 리소스를 삭제하여 추가 요금이 발생하지 않도록 할 수 있습니다. 이 경우 가능한 방법은 다음과 같습니다.

1. `https://portal.azure.com`의 Azure Portal을 열고 Azure 구독과 연관된 Microsoft 계정을 사용하여 로그인합니다.

2. 상단 검색 창에서 *Azure AI 서비스 다중 서비스 계정*을 검색하고 이 랩에서 만든 Azure AI 서비스 다중 서비스 계정 리소스를 선택합니다.

3. 리소스 페이지에서 **삭제**를 선택하고 지침에 따라 리소스를 삭제합니다.

## 자세한 정보

이 연습에서는 Azure AI 비전 서비스의 일부 이미지 분석 및 조작 기능을 살펴보았습니다. 이 서비스에는 개체와 사람을 검색하는 기능과 기타 Computer Vision 작업도 포함되어 있습니다.

**Azure AI 비전** 서비스 사용에 대한 자세한 내용은 [Azure AI 비전 설명서](https://learn.microsoft.com/azure/ai-services/computer-vision/)를 참조하세요.
