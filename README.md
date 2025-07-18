```mermaid
graph TD
    A[程式啟動] --> B(InstallCpuTechFeatureByteHandler 函數被呼叫)
    B --> C{是否在 SMM 模式下運行?}
    C -- 否 --> D[返回 EFI_OUT_OF_RESOURCES]
    C -- 是 --> E(分配 FEATURE_BYTE_HANDLER_PROTOCOL 記憶體)
    E --> F(設定 FbHandler 的回調函數)
    F --> F1[PrepareFbVariables 指向 PrepareVariables]
    F --> F2[ApplyFbCode 指向 ApplyFbCode]
    F --> F3[CommitFbVariables 指向 CommitVariables]
    F3 --> G(安裝 FEATURE_BYTE_HANDLER_PROTOCOL)
    G --> H{是否啟用 PCD: PcdSgFeatureByteApplyTxtDefaultEnable?}
    H -- 否 --> I[安裝完成，返回 EFI_SUCCESS]
    H -- 是 --> J(嘗試獲取 SYS_GUARD_FB_FACTORYCONFIG_FLAGS 變數)
    J --> K{變數是否存在?}
    K -- 否 --> I
    K -- 是 --> L(呼叫 SetDefaultSGFbFactoryConfigFlags 函數)
    L --> I
