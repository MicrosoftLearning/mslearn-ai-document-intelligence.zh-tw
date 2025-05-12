---
lab:
  title: 使用 Azure AI 內容瞭解分析內容
  module: Multimodal analysis with Content Understanding
---

# 使用 Azure AI 內容瞭解分析內容

在本練習中，您將使用 Azure AI Foundry 入口網站建立一個可以從發票中擷取資訊的內容理解專案。 接著，您將在 Azure AI Foundry 入口網站測試內容分析器，並透過內容瞭解 REST 介面加以取用。

本練習大約需要 **30** 分鐘的時間。

## 建立 Azure AI Foundry 專案

讓我們從建立 Azure AI Foundry 專案開始。

1. 在網頁瀏覽器中，開啟 [Azure AI Foundry 入口網站](https://ai.azure.com) 於`https://ai.azure.com` 並使用您的 Azure 認證登入。 關閉首次登入時開啟的所有提示或快速啟動窗格，如有必要，使用左上角的 **Azure AI Foundry** 標誌瀏覽到首頁，首頁類似於下圖：

    ![Azure AI Foundry 入口網站螢幕擷取畫面。](../media/ai-foundry-portal.png)

1. 在首頁中，選取 **+ 建立專案**。
1. 請在 [建立專案精靈]**** 中，輸入專案有效名稱，如果建議使用現有的中樞，請選擇建立新中樞的選項。 然後審查 Azure 資源，將會自動建立，以便支援中樞和專案。
1. 選取**自訂**，然後為您的中樞指定下列設定：
    - **中樞名稱**：*請提供有效的中樞名稱*
    - **訂用帳戶**：您的 Azure 訂用帳戶**
    - **資源群組**：建立或選取資源群組**
    - **位置**：選擇以下區域之一\*
        - 美國西部
        - 瑞典中部
        - 澳大利亞東部
    - **連接 Azure AI 服務或 Azure OpenAI**：*[建立新的 AI 服務資源]*
    - **連線 Azure AI 搜尋服務**： *建立新的 Azure AI 搜尋服務資源，具有唯一名稱*

    > \*在撰寫本文時，Azure AI 內容瞭解僅在這些地區可用。

1. 選取**下一步**並檢閱您的設定。 然後選取**建立**並等待該流程完成。
1. 建立專案後，關閉顯示的所有提示並檢閱 Azure AI Foundry 入口網站中的專案頁面，該頁面應類似於下圖：

    ![Azure AI Foundry 入口網站中 Azure AI 專案詳細資料的螢幕螢幕擷取畫面。](../media/ai-foundry-project.png)

## 建立內容瞭解分析器

您將組建一個可以從發票中擷取資訊的分析器。 您將首先根據範例發票定義一個模式。

1. 在新的瀏覽器索引標籤中，從 `https://github.com/microsoftlearning/mslearn-ai-document-intelligence/raw/main/Labfiles/05-content-understanding/forms/invoice-1234.pdf` 下載 [invoice-1234.pdf](https://github.com/microsoftlearning/mslearn-ai-document-intelligence/raw/main/Labfiles/05-content-understanding/forms/invoice-1234.pdf) 樣本表格並將其儲存在本機資料夾中。
1. 返回包含 Azure AI Foundry 專案首頁的索引標籤；然後在左側的功能窗格中，選取 [內容瞭解]****。
1. 在 [內容瞭解]**** 頁面上，選取頂部的 [自訂分析器]**** 索引標籤。
1. 在 [內容瞭解] 自訂分析器頁面上，選取 [+ 建立]****，然後使用以下設定建立工作：
    - **工作名稱**：發票分析
    - **描述**：從發票擷取資料
    - **Azure AI 服務連接**：*Azure AI Foundry 中心內的 Azure AI 服務資源*
    - **Azure Blob 儲存體帳戶**：*Azure AI Foundry 中心中的儲存體帳戶*
1. 等待工作建立。

    > **提示**：如果存取儲存體時發生錯誤，請等待片刻，然後重試。

1. 在 [定義結構描述]**** 頁面上，上傳您剛剛下載的 **invoice-1234.pdf** 檔案。
1. 選取 [發票分析]**** 範本，然後選取 [建立]****。

    **[發票分析] 範本包括發票中的一般欄位。 您可以使用結構描述編輯器刪除任何您不需要的建議欄位，並新增任何您需要的自訂欄位。

1. 在建議的欄位清單中，選取 **[BillingAddress]**。 您上傳的發票格式不需要此欄位，因此請使用出現的 [刪除欄位]**** (**&#128465;**) 圖示將其刪除。
1. 現在刪除以下建議的不需要的欄位：
    - BillingAddressRecipient
    - BillingAddressRecipient
    - CustomerId
    - CustomerTaxId
    - 到期日期
    - InvoiceTotal
    - PaymentTerm
    - PreviousUnpaidBalance
    - PurchaseOrder
    - RemittanceAddress
    - RemittanceAddressRecipient
    - ServiceAddress
    - BillingAddressRecipient
    - ShippingAddress
    - BillingAddressRecipient
    - TotalDiscount
    - VendorAddressRecipient
    - VendorTaxId
    - TaxDetails
1. 使用 [+ 新增欄位]**** 按鈕來新增下列欄位：

    | 欄位名稱 | 欄位描述 | 值類型 | 方法 |
    |--|--|--|--|
    | `VendorPhone` | `Vendor telephone number` | String | 擷取 |
    | `ShippingFee` | `Fee for shipping` | 數字 | 擷取 |

1. 驗證您完成的結構描述是否如下所示，然後選取 [儲存]** **。
    ![發票結構描述的螢幕擷取畫面。](../media/invoice-schema.png)

1. 在 **測試分析器** 頁面，如果分析未自動開始，請選取 **執行分析**。 然後等候分析完成並檢閱發票被確定為符合結構描述欄位的文字值。
1. 檢閱分析結果，結果應與此類似：

    ![分析器測試結果的螢幕擷取畫面。](../media/analysis-results.png)

1. 檢視 [欄位]**** 窗格中標識的欄位的詳細資料，然後檢視 [結果] **** 索引標籤以查看 JSON 表示形式。

## 組建並測試分析器

現在您已經訓練了一個模型來從發票中擷取欄位，您可以組建一個分析器來與類似的表單一起使用。

1. 選取 [組建分析器] **** 頁面，然後選取 [+ 組建分析器]**** 並使用以下屬性（輸入內容與此處所示完全相同）組建一個新的分析器：
    - **名稱**：`contoso-invoice-analyzer`
    - **描述**：`Contoso invoice analyzer`
1. 等待新的分析器準備就緒（使用**重新整理**按鈕進行檢查）。
1. 從 `https://github.com/microsoftlearning/mslearn-ai-document-intelligence/raw/main/Labfiles/05-content-understanding/forms/invoice-1235.pdf` 下載 [invoice-1235.pdf](https://github.com/microsoftlearning/mslearn-ai-document-intelligence/raw/main/Labfiles/05-content-understanding/forms/invoice-1235.pdf) 並將其儲存在本機資料夾中。
1. 返回 [組建分析器]**** 頁面並選取 **contoso-invoice-analyzer** 連結。 分析器結構描述中定義的欄位隨即顯示。
1. 在 **contoso-invoice-analyzer** 頁面中，選取 [測試]** **索引標籤。
1. 使用 [+ 上傳測試檔案]**** 按鈕上傳 **invoice-1235.pdf** 並執行分析以從測試表單中擷取欄位資料。
1. 檢閱測試結果，並驗證分析器是否從測試發票中擷取了正確的欄位。
1. **關閉 contoso-invoice-analyzer*** 頁面。

## 使用內容瞭解 REST API

現在您已經組建了一個分析器，您可以透過內容瞭解 REST API 從用戶端應用程式中使用它。

1. 在 [專案詳細資料]**** 區域中，記下 [專案連接字串]****。 您將使用此連接字串連線到用戶端應用程式中的專案。
1. 開啟一個新的瀏覽器索引標籤（保持 Azure AI Foundry 入口網站在現有索引標籤中開啟）。 然後在新索引標籤中，瀏覽到 `https://portal.azure.com` 的 [Azure 入口網站](https://portal.azure.com)；如果出現提示，請使用您的 Azure 認證登入。

    關閉任何歡迎通知，以查看 Azure 入口網站 首頁。

1. 使用頁面頂部搜尋欄右側的 **[\>_]** 按鈕在 Azure 入口網站中建立一個新的 Cloud Shell，並選擇 ***PowerShell 環境*** (訂用帳戶中沒有儲存體)。

    Cloud Shell 會在 Azure 入口網站底部的窗格顯示命令列介面。 您可以調整或最大化此窗格的大小，以便更輕鬆地使用。

    > **注意**：如果您之前建立了使用 *Bash* 環境的 Cloud Shell，請將其切換到 ***PowerShell***。

1. 在 Cloud Shell 工具列中，在**設定**功能表中，選擇**轉到經典版本**（這是使用程式碼編輯器所必需的）。

    **<font color="red">繼續之前，請先確定您已切換成 Cloud Shell 傳統版本。</font>**

1. 請在 Cloud Shell 窗格中，輸入下列命令，以便複製包含練習程式碼檔案的 GitHub 存放庫（輸入 [命令]，或將它複製到剪貼簿，然後在命令列上點選滑鼠右鍵，再貼上純文字即可）：

    ```
   rm -r mslearn-ai-foundry -f
   git clone https://github.com/microsoftlearning/mslearn-ai-document-intelligence mslearn-ai-doc
    ```

    > **提示**：當您將命令輸入到 Cloud Shell 中時，輸出可能會佔用大量的螢幕緩衝區。 您可以透過輸入 `cls` 命令來清除螢幕，以便更輕鬆地專注於每個工作。

1. 複製存放庫之後，瀏覽至包含應用程式碼檔案的資料夾：

    ```
   cd mslearn-ai-doc/Labfiles/05-content-understanding/code
   ls -a -l
    ```

1. 在 Cloud Shell 命令列窗格中，輸入下列命令來安裝您將使用的程式庫：

    ```
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install dotenv azure-identity azure-ai-projects
    ```

1. 輸入以下命令，編輯已提供的設定檔：

    ```
   code .env
    ```

    程式碼編輯器中會開啟檔案。

1. 在程式碼檔案中，將 **YOUR_PROJECT_CONNECTION_STRING** 預留位置替換為專案的連接字串（從 Azure AI Foundry 入口網站中的專案 [概觀]**** 頁面複製），並確保 [分析器]**** 設定為指派至分析器的名稱（應為 *contoso-invoice-analyzer*）
1. 取代預留位置後，在程式碼編輯器中使用 **CTRL+S** 命令儲存變更，然後使用 **CTRL+Q** 命令關閉程式碼編輯器，同時保持 Cloud Shell 命令列開啟。

1. 在 Cloud Shell 命令列中，輸入以下命令編輯已提供的 **analyze_invoice.py** Python 程式碼檔案：

    ```
    code analyze_invoice.py
    ```
    Python 程式碼檔案會在程式碼編輯器中開啟：

1. 檢閱程式碼，其中：
    - 標識要分析的發票檔案，預設值為 **invoice-1236.pdf**。
    - 從專案中擷取 Azure AI 服務資源的端點和金鑰。
    - 向您的內容理解端點提交 HTTP POST 請求，指示分析影像。
    - 檢查 POST 操作的回應以擷取分析操作的 ID。
    - 反復向您的內容瞭解服務提交 HTTP GET 請求，直到操作不再執行。
    - 如果操作成功，則解析 JSON 回應並顯示擷取到的值
1. 使用**CTRL+Q**命令關閉程式碼編輯器，同時保持 Cloud Shell 命令列開啟。
1. 在 Cloud Shell 命令列窗格中，輸入以下命令來執行 Python 程式碼：

    ```
    python analyze_invoice.py invoice-1236.pdf
    ```

1. 檢閱程式的輸出。
1. 使用以下命令以不同的發票執行程式：

    ```
    python analyze_invoice.py invoice-1235.pdf
    ```

    > **提示**：程式碼資料夾中有三個發票檔案可供您使用（invoice-1234.pdf、invoice-1235.pdf 和 invoice-1236.pdf） 

## 清理

如果您已完成使用內容瞭解服務，則應刪除在此練習中建立的資源，以避免產生不必要的 Azure 成本。

1. 在 Azure AI Foundry 入口網站中，瀏覽至 **travel-insurance** 專案並加以刪除。
1. 前往您在 Azure 入口網站中針對這些練習所建立的資源群組。

