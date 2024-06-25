---
lab:
  title: 使用預先建置的文件智慧服務模型
  module: Module 6 - Document Intelligence
---

# 使用預先建置的文件智慧服務模型

在此練習中，您會在 Azure 訂用帳戶中設定 Azure AI 文件智慧服務資源。 您將使用 Azure AI 文件智慧服務工作室和 C# 或 Python，將表格提交至該資源以進行分析。

## 建立 Azure AI 文件智慧服務資源

在您呼叫 Azure AI 文件智慧服務之前，您必須在 Azure 中建立資源以裝載該服務：

1. 在瀏覽器索引標籤中，開啟 Azure 入口網站 (位於 [https://portal.azure.com](https://portal.azure.com?azure-portal=true))，使用與您的 Azure 訂用帳戶相關聯的 Microsoft 帳戶登入。
1. 在 Azure 入口網站首頁上，瀏覽至頂端搜尋方塊，輸入 [文件智慧服務]****，然後按 **Enter** 鍵。
1. 在 [文件智慧服務]**** 頁面上，選取 [建立]****。
1. 在 [建立文件智慧服務]**** 頁面上，使用下列項目來設定您的資源：
    - 訂用帳戶：您的 Azure 訂用帳戶。
    - **資源群組**：選取或建立具有唯一名稱的資源群組，例如 DocIntelligenceResources**。
    - **區域**：選取您附近的區域。
    - **名稱**：輸入全域唯一名稱。
    - **定價層**：選取 [免費 F0]**** (如果您沒有可用的免費層，請選取 [標準 S0]****)。
1. 接著選取 [檢閱 + 建立]****，然後選取 [建立]****。 等候 Azure 建立 Azure AI 文件智慧服務資源。
1. 部署完成後，請選取**前往資源**。 讓此頁面保持開啟，以供此練習的其餘部分使用。

## 使用讀取模型

讓我們從使用 [Azure AI 文件智慧服務工作室]**** 和讀取模型開始，以使用多種語言分析文件。 您會將 Azure AI 文件智慧服務工作室連線至您剛才建立的資源以執行分析：

1. 開啟新的瀏覽器索引標籤，然後移至 **Azure AI 文件智慧服務工作室** (位於 [https://documentintelligence.ai.azure.com/studio](https://documentintelligence.ai.azure.com/studio))。
1. 在 [文件分析]**** 底下，選取 [讀取]**** 圖格。
1. 如果系統要求您登入您的帳戶，請使用您的 Azure 認證。
1. 如果系統詢問您要使用哪一項 Azure AI 文件智慧服務資源，請選取您在建立 Azure AI 文件智慧服務資源時所使用的訂用帳戶和資源名稱。
1. 在左側的文件清單中，選取 [read-german.pdf]****。

    ![顯示 Azure AI 文件智慧服務工作室中 [讀取] 頁面的螢幕擷取畫面。](../media/read-german-sample.png#lightbox)

1. 從左上方選取 [分析選項]****，然後在 [分析選項]**** 窗格中啟用 [語言]**** 核取方塊 (在 [選擇性偵測]**** 底下)，然後按一下 [儲存]****。 
1. 從左上方選取 [執行分析]****。
1. 在分析完成之後，系統從影像中擷取的文字會顯示在右側的 [Content]**** \(內容\) 索引標籤中。請檢閱此文字並將其與原始影像中的文字比較以確認精確度。
1. 選取 [Result]**** \(結果\) 索引標籤。此索引標籤會顯示擷取的 JSON 程式碼。 
1. 在 [Result]**** \(結果\) 索引標籤中捲動至 JSON 程式碼的底部。請注意，讀取模型已偵測到每個範圍的語言。 大部分範圍都是德文 (語言代碼 `de`)，但您可以在範圍中找到其他語言代碼 (例如最後某個範圍中的英文 (語言代碼 `en`))。

    ![顯示在 Azure AI 文件智慧服務工作室的讀取模型結果中，針對兩個範圍進行語言偵測的螢幕擷取畫面。](../media/language-detection.png#lightbox)

## 準備在 Visual StudioCode 中開發應用程式

現在我們來探索使用 Azure 文件智慧服務 SDK 的應用程式。 您將使用 Visual Studio Code 開發應用程式。 您應用程式的程式碼檔案已在 GitHub 存放庫中提供。

> **秘訣**：如果您已經複製 **mslearn-ai-document-intelligence** 存放庫，請在 Visual Studio 程式碼中將其開啟。 否則，請遵循下列步驟將其複製到您的開發環境。

1. 啟動 Visual Studio Code。
1. 開啟選擇區 (SHIFT+CTRL+P) 並執行 **Git：複製 ** 命令，將 `https://github.com/MicrosoftLearning/mslearn-ai-document-intelligence` 存放庫複製到本機資料夾 (哪個資料夾無關緊要)。
1. 複製存放庫後，請在 Visual Studio Code 中開啟此資料夾。

    > **注意**：如果 Visual Studio Code 顯示快顯訊息，提示您信任您所開啟的程式碼，請按一下快顯項目中的 [是，我信任作者]**** 選項。

1. 等候其他檔案安裝以支援存放庫中的 C# 程式碼專案。

    > **注意**：如果系統提示您新增必要的資產來組建和偵錯，請選取 [現在不要]****。 如果出現提示訊息：「在資料夾中偵測到 Azure 函式專案」**，您可以放心地關閉該訊息。

## 設定您的應用程式

已提供 C# 和 Python 的應用程式，以及您將用來測試文件智慧服務的範例 pdf 檔案。 這兩個應用程式都有相同的功能。 首先，您將完成應用程式的一些重要部分，以開始使用您的 Azure 文件智慧服務資源。

1. 檢查下列發票，並記下其部分欄位和值。 這是將由您程式碼分析的發票。

    ![顯示範例發票文件的螢幕擷取畫面。](../media/sample-invoice.png#lightbox)

1. 在 Visual Studio Code 的 [總管]**** 窗格中，瀏覽至 [Labfiles/01-prebuild-models]**** 資料夾，然後根據您的語言偏好設定展開 [CSharp]**** 或 [Python]**** 資料夾。 每個資料夾都包含應用程式的特定語言檔案，您將在其中整合 Azure 文件智慧服務功能。

1. 以滑鼠右鍵按一下包含程式碼檔案的 [CSharp]**** 或 [Python]**** 資料夾，然後選取 [開啟整合式終端]****。 然後，執行使用語言的適當命令，以安裝 Azure 表格辨識器 (文件智慧服務的先前名稱) SDK 套件：

    **C#：**

    ```powershell
    dotnet add package Azure.AI.FormRecognizer --version 4.1.0
    ```

    **Python**：

    ```powershell
    pip install azure-ai-formrecognizer==3.3.0
    ```

## 新增程式碼以使用 Azure 文件智慧服務

現在您已準備好使用 SDK 來評估 pdf 檔案。

1. 切換至顯示 Azure 入口網站中 Azure AI 文件智慧服務概觀的瀏覽器索引標籤。 在左側窗格的 [資源管理]** 底下，選取 [金鑰和端點]****。 在 [端點]**** 值的右側，按一下 [複製到剪貼簿]**** 按鈕。
1. 在 [總管]**** 窗格中的 [CSharp]**** 或 [Python]**** 資料夾中，開啟慣用語言的程式碼檔案，並以您剛才複製的字串取代 `<Endpoint URL>`：

    **C#**：***Program.cs***

    ```csharp
    string endpoint = "<Endpoint URL>";
    ```

    **Python**：***document-analysis.py***

    ```python
    endpoint = "<Endpoint URL>"
    ```

1. 在 Azure 入口網站中，切換至顯示 Azure AI 文件智慧服務**金鑰和端點**的瀏覽器索引標籤。 在 [金鑰 1]**** 值的右側，按一下 [複製到剪貼簿]*** 按鈕。
1. 在 Visual Studio Code 的程式碼檔案中，找出這一行，並將 `<API Key>` 取代為您剛才複製的字串：

    **C#**

    ```csharp
    string apiKey = "<API Key>";
    ```

    **Python**

    ```python
    key = "<API Key>"
    ```

1. 找出 `Create the client` 註解。 接下來，在新的一行輸入下列程式碼：

    **C#**

    ```csharp
    var cred = new AzureKeyCredential(apiKey);
    var client = new DocumentAnalysisClient(new Uri(endpoint), cred);
    ```

    **Python**

    ```python
    document_analysis_client = DocumentAnalysisClient(
        endpoint=endpoint, credential=AzureKeyCredential(key)
    )
    ```

1. 找出 `Analyze the invoice` 註解。 接下來，在新的一行輸入下列程式碼：

    **C#**

    ```csharp
    AnalyzeDocumentOperation operation = await client.AnalyzeDocumentFromUriAsync(WaitUntil.Completed, "prebuilt-invoice", fileUri);
    ```

    **Python**

    ```python
    poller = document_analysis_client.begin_analyze_document_from_url(
        fileModelId, fileUri, locale=fileLocale
    )
    ```

1. 找出 `Display invoice information to the user` 註解。 接下來，在新的一行輸入下列程式碼：

    **C#**

    ```csharp
    AnalyzeResult result = operation.Value;
    
    foreach (AnalyzedDocument invoice in result.Documents)
    {
        if (invoice.Fields.TryGetValue("VendorName", out DocumentField? vendorNameField))
        {
            if (vendorNameField.FieldType == DocumentFieldType.String)
            {
                string vendorName = vendorNameField.Value.AsString();
                Console.WriteLine($"Vendor Name: '{vendorName}', with confidence {vendorNameField.Confidence}.");
            }
        }
    ```

    **Python**

    ```python
    receipts = poller.result()
    
    for idx, receipt in enumerate(receipts.documents):
    
        vendor_name = receipt.fields.get("VendorName")
        if vendor_name:
            print(f"\nVendor Name: {vendor_name.value}, with confidence {vendor_name.confidence}.")
    ```

    > [!NOTE]
    > 您已新增程式碼以顯示廠商名稱。 入門專案也包括用來顯示「客戶名稱」** 和「發票總計」** 的程式碼。

1. 將變更儲存至程式碼檔案。

1. 在互動式終端窗格中，確定資料夾內容是您慣用語言的資料夾。 然後輸入下列命令來執行應用程式。

1. (僅適用於 C#)****** 若要建置專案，請輸入下列命令：

    **C#：**

    ```powershell
    dotnet build
    ```

1. 若要執行程式碼，請輸入下列命令：

    **C#：**

    ```powershell
    dotnet run
    ```

    **Python**：

    ```powershell
    python document-analysis.py
    ```

程式會搭配信賴等級顯示廠商名稱、客戶名稱和發票總計。 將其所報告的值，與您在此節開頭開啟的範例發票互相比較。

## 清理

如果您已完成使用 Azure 資源，請記得刪除 [Azure 入口網站](https://portal.azure.com/?azure-portal=true)中的資源，以避免產生更多費用。

## 其他相關資訊

如需有關文件智慧服務的詳細資訊，請參閱[文件智慧服務的文件](https://learn.microsoft.com/azure/ai-services/document-intelligence/?azure-portal=true)。
