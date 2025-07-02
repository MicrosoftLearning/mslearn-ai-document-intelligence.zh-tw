---
lab:
  title: 從表單擷取資料
  module: Module 6 - Document Intelligence
---

# 從表單擷取資料

假設一家公司目前要求員工手動購買訂單表，並將資料輸入資料庫中。 公司想要您利用 AI 服務來改善資料輸入流程。 您決定組建機器學習模型，以讀取表單並產生結構化資料，可用來自動更新資料庫。

**Azure AI 文件智慧服務**是一項 Azure AI 服務，可讓使用者建置自動化資料處理軟體。 此軟體可以使用光學字元辨識 (OCR)，從表單文件中擷取文字、索引鍵/值組及資料表。 Azure AI 文件智慧服務具有預先建立的模型，可辨識發票、收據及名片。 該服務也提供定型自訂模型的功能。 在此練習中，我們將著重於組建自訂模型。

## 準備在 Visual Studio Code 中開發應用程式

現在，讓我們使用服務 SDK，使用 Visual Studio Code 開發應用程式。 您應用程式的程式碼檔案已在 GitHub 存放庫中提供。

> **秘訣**：如果您已經複製 **mslearn-ai-document-intelligence** 存放庫，請在 Visual Studio 程式碼中將其開啟。 否則，請遵循下列步驟將其複製到您的開發環境。

1. 啟動 Visual Studio Code。
1. 開啟選擇區 (SHIFT+CTRL+P) 並執行 **Git：複製 ** 命令，將 `https://github.com/MicrosoftLearning/mslearn-ai-document-intelligence` 存放庫複製到本機資料夾 (哪個資料夾無關緊要)。
1. 複製存放庫後，請在 Visual Studio Code 中開啟此資料夾。

    > **注意**：如果 Visual Studio Code 顯示快顯訊息，提示您信任您所開啟的程式碼，請按一下快顯項目中的 [是，我信任作者]**** 選項。

1. 等候其他檔案安裝以支援存放庫中的 C# 程式碼專案。

    > **注意**：如果系統提示您新增必要的資產來組建和偵錯，請選取 [現在不要]****。 如果出現提示訊息：「在資料夾中偵測到 Azure 函式專案」**，您可以放心關閉該訊息。

## 建立 Azure AI 文件智慧服務資源

若要使用 Azure AI 文件智慧服務服務，則您需要 Azure 訂用帳戶中的 Azure AI 文件智慧服務或 Azure AI 服務資源。 您將使用 Azure 入口網站來建立資源。

1. 在瀏覽器索引標籤中，開啟 Azure 入口網站 (位於 `https://portal.azure.com`)，使用與您的 Azure 訂用帳戶相關聯的 Microsoft 帳戶登入。
1. 在 Azure 入口網站首頁上，瀏覽至頂端搜尋方塊，輸入 [文件智慧服務]****，然後按 **Enter** 鍵。
1. 在 [文件智慧服務]**** 頁面上，選取 [建立]****。
1. 在 [建立文件智慧服務]**** 頁面上，使用下列項目來設定您的資源：
    - 訂用帳戶：您的 Azure 訂用帳戶。
    - **資源群組**：選取或建立具有唯一名稱的資源群組，例如 DocIntelligenceResources**。
    - **區域**：選取您附近的區域。
    - **名稱**：輸入全域唯一名稱。
    - **定價層**：選取 [免費 F0]**** (如果您沒有可用的免費層，請選取 [標準 S0]****)。
1. 接著選取 [檢閱 + 建立]****，然後選取 [建立]****。 等候 Azure 建立 Azure AI 文件智慧服務資源。
1. 部署完成時，請選取 [移至資源]****，以檢視資源的 [概觀]**** 頁面。 

## 收集用於定型的文件

您將使用像這樣的範例表格來定型和測試模型： 

![此專案中使用的發票影像。](../media/Form_1.jpg)

1. 返回 **Visual Studio Code**。 在 [總管]** 窗格中，開啟 **Labfiles/02-custom-document-intelligence** 資料夾，然後展開 **sample-forms** 資料夾。 請注意，資料夾中有以 **.json** 和 **.jpg** 結尾的檔案。

    您將使用 **.jpg** 檔案來定型您的模型。  

    **.json** 檔案已為您產生，並包含標籤資訊。 檔案會連同表單一起上傳至您的 Blob 儲存體容器。

    您可以在 Visual Studio Code 上選取影像，以檢視我們在 *sample-forms* 資料夾中所使用的影像。

1. 返回 **Azure 入口網站**，如果您尚未瀏覽至資源的 [概觀]**** 頁面，則請瀏覽至該處。 在 [基本]** 區段底下，檢視您在其中所建立文件智慧服務資源的 [資源群組]****。

1. 在資源群組的 [概觀] **** 頁面上，記下 [訂用帳戶識別碼] **** 和 [位置]****。 後續步驟中，您將需要這些值以及您的**資源群組**名稱。

    ![資源群組頁面的範例。](../media/resource_group_variables.png)

1. 在 Visual Studio Code 的 [總管]** 窗格中，瀏覽至 [Labfiles/02-custom-document-intelligence]**** 資料夾，然後根據您的語言偏好設定展開 **CSharp** 或 **Python** 資料夾。 每個資料夾都包含應用程式的特定語言檔案，您將在其中整合 Azure OpenAI 功能。

1. 以滑鼠右鍵按一下包含程式碼檔案的 [CSharp]**** 或 [Python]**** 資料夾，然後選取 [開啟整合式終端]****。 

1. 在終端中，執行下列命令以登入 Azure。 **az login** 命令會開啟 Microsoft Edge 瀏覽器，並使用您所用來建立 Azure AI 文件智慧服務資源的相同帳戶登入。 登入之後，請關閉瀏覽器視窗。

    ```powershell
    az login
    ```

1. 返回 Visual Studio Code。 在終端窗格中，執行下列命令以列出 Azure 位置。

    ```powershell
    az account list-locations -o table
    ```

1. 在輸出中，尋找對應資源群組位置的 **Name** 值 (例如，*美國東部*的對應名稱為 *eastus*)。

    > **重要**：記錄 **Name** 值，並在步驟 11 中加以使用。

1. 在 Visual Studio Code 的 **Labfiles/02-custom-document-intelligence** 資料夾中，選取 [setup.cmd]****。 您將使用此指令碼來執行 Azure 命令列介面 (CLI)，建立所需的其他 Azure 資源所需的命令。

1. 在 **setup.cmd** 指令碼中，檢閱命令。 該程式將會：
    - 在 Azure 資源群組中建立儲存體帳戶
    - 將檔案從本地 *sampleforms* 資料夾上傳至儲存體帳戶中名為 *sampleforms* 的容器
    - 列印共用存取簽章 URI

1. 使用您部署文件智慧服務資源之訂用帳戶、資源群組及位置名稱的適當值，修改 **subscription_id**、**resource_group** 以及**位置**變數宣告。
然後**儲存**您的變更。

    為練習保留 **expiry_date** 變數原狀。 此變數在產生共用存取簽章 (SAS) URI 時使用。 在實務上，您會想要為 SAS 設定適當的到期日期。 您可以在[此處](https://docs.microsoft.com/azure/storage/common/storage-sas-overview#how-a-shared-access-signature-works)深入了解 SAS。  

1. 在 **Labfiles/02-custom-document-intelligence** 資料夾的終端中，輸入以下命令以執行指令碼：

    ```PowerShell
    ./setup.cmd
    ```

1. 當指令碼完成時，請檢閱顯示的輸出。

1. 在 Azure 入口網站中，重新整理資源群組，並確認其中包含稍早才建立的 Azure 儲存體帳戶。 開啟儲存體帳戶，並在左側窗格中，選取 [儲存體瀏覽器]****。 然後在儲存體瀏覽器中，展開 [Blob 容器]****，然後選取 [sampleforms]**** 容器，以確認檔案已從本機 **02-custom-document-intelligence/sample-forms** 資料夾上傳。

## 使用文件智慧服務工作室將模型定型

現在，您將使用上傳至儲存體帳戶的檔案來定型模型。

1. 在您的瀏覽器中，瀏覽至位於 `https://documentintelligence.ai.azure.com/studio` 的文件智慧服務工作室。
1. 向下捲動至 [自訂模型]**** 區段，然後選取 [自訂擷取模型]**** 磚。
1. 如果系統要求您登入您的帳戶，請使用您的 Azure 認證。
1. 如果系統詢問您要使用哪一項 Azure AI 文件智慧服務資源，請選取您在建立 Azure AI 文件智慧服務資源時所使用的訂用帳戶和資源名稱。
1. 在 [我的專案]**** 下，選取 [建立專案]****。 使用下列組態：

    - **專案名稱**：輸入唯一名稱。
        - 選取 [繼續]。
    - **設定服務資源**：選取您先前在此實驗室中所建立的訂用帳戶、資源群組和文件智慧服務資源。 核取 [設定為預設]** 方塊。 保留預設 API 版本。 
        - 選取 [繼續]。
    - **連線定型資料來源**：選取安裝指令碼所建立的訂用帳戶、資源群組和儲存體帳戶。 核取 [設定為預設]** 方塊。 選取 `sampleforms` Blob 容器，並將資料夾路徑保留空白。
        - 選取 [繼續]。
    - 選取 [建立專案]**

1. 建立專案之後，在畫面控制項右上方，選取**定型**以定型模型。 使用下列組態：
    - **模型識別碼**：*提供全域唯一名稱 (在下一個步驟中將需要該模型識別碼名稱)*。 
    - **建置模式**：範本。
1. 選取 **移至模型**。
1. 定型可能需要一些時間。 當通知完成時，您會看到通知。

## 測試您的自訂文件智慧服務模型

1. 返回 Visual Studio Code。 在終端中，根據您的使用語言執行相應命令以安裝文件智慧服務套件：

    **C#：**

    ```powershell
    dotnet add package Azure.AI.FormRecognizer --version 4.1.0
    ```

    **Python**：

    ```powershell
    pip install azure-ai-formrecognizer==3.3.3
    ```

1. 在 Visual Studio Code 的 **Labfiles/02-custom-document-intelligence** 資料夾中，選取您所使用的語言。 使用以下值編輯組態檔 (**appsettings.json** 或 **.env**，具體取決於您的使用語言)：
    - 您的文件智慧服務端點。
    - 您的文件智慧服務金鑰。
    - 您在定型模型時所提供之產生的模型識別碼。 您可以在文件智慧服務工作的 [模型]**** 頁面上找到此項目。 **儲存**您的變更。

1. 在 Visual Studio Code 中，開啟用戶端應用程式的程式碼檔案 (針對 C# 為 *Program.cs*；而針對 Python 則為 *test-model.py*)，並檢閱其包含的程式碼，特別是 URL 中的影像指的是網頁上 GitHub 存放庫中的檔案。

1. 傳回終端，然後輸入下列命令以執行程式：

    **C#**

    ```powershell
    dotnet build
    dotnet run
    ```

    **Python**

    ```powershell
    python test-model.py
    ```

1. 檢視輸出並觀察模型的輸出如何提供欄位名稱，例如 「`Merchant`」和 「`CompanyPhoneNumber`」。

## 清理

如果您已完成使用 Azure 資源，請記得刪除 [Azure 入口網站](https://portal.azure.com/?azure-portal=true)中的資源，以避免產生更多費用。

## 其他相關資訊

如需有關文件智慧服務的詳細資訊，請參閱[文件智慧服務的文件](https://learn.microsoft.com/azure/ai-services/document-intelligence/?azure-portal=true)。
