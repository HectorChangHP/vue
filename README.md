'''
graph TD
    A[程式啟動] --> B(InstallCpuTechFeatureByteHandler 函數被呼叫)

    B --> C{是否在 SMM 模式下運行? (InMpm())}
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

    J --> K{變數是否存在? (EFI_NOT_FOUND)}
    K -- 否 --> I
    K -- 是 --> L(呼叫 SetDefaultSGFbFactoryConfigFlags 函數)
    L --> I

    subgraph Feature Byte 處理流程 (當協議被觸發時)
        START_FB[Feature Byte 處理開始] --> M(PrepareVariables 函數被呼叫)

        M --> N{mSGFbFactoryConfigFlags 存在 且 PcdSgFeatureByteApplyTxtDefaultEnable 啟用?}
        N -- 是 --> O(呼叫 SgFbInitFactoryConfigFlags 初始化旗標)
        N -- 否 --> P[不做初始化]
        O --> P

        P --> Q(設定 mVtEnableOnAndroidFb = FALSE)
        Q --> R(對每個 Feature Byte 呼叫 ApplyFbCode)

        R --> S{FeatureByte 是否為空?}
        S -- 否 --> END_FB_APPLY[跳過此 Feature Byte]
        S -- 是 --> T(呼叫 GetOrdinal 識別 Feature Byte 類型)

        T --> U{Feature Byte 類型?}
        U -- FbSystemGuardEnable --> V(設定 mSGFbFactoryConfigFlags->IsSysGuardFbExist = TRUE)
        U -- FbAndroid --> W(設定 mVtEnableOnAndroidFb = TRUE)
        U -- 其他 --> X[不做處理]

        V --> R
        W --> R
        X --> R
        R --> Y{所有 Feature Byte 都處理完畢?}
        Y -- 是 --> Z(CommitVariables 函數被呼叫)
        Y -- 否 --> R

        Z --> AA{mSGFbFactoryConfigFlags 存在 且 PcdSgFeatureByteApplyTxtDefaultEnable 啟用?}
        AA -- 是 --> BB(呼叫 SgFbSetFactoryConfigFlags 保存旗標)
        AA -- 否 --> CC[跳過保存]
        BB --> CC

        CC --> DD{IsSysGuardFbExist 旗標被設定?}
        DD -- 是 --> EE(呼叫 SetTxtEnable 啟用 TXT 功能)
        EE --> FF[釋放 mSGFbFactoryConfigFlags 記憶體]
        DD -- 否 --> FF

        FF --> GG{mVtEnableOnAndroidFb 旗標被設定?}
        GG -- 是 --> HH(呼叫 SetVtEnable 啟用 VTx/VTd 功能)
        HH --> END_FB_PROCESS[Feature Byte 處理完成]
        GG -- 否 --> END_FB_PROCESS
    end   
'''
