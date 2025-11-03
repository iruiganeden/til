flowchart LR
    %% ================================================
    %% Netcool 論理構成（2台 ホットスタンバイ）
    %% - Probe: Ping / SNMP / EIF（Netcoolサーバ同居）
    %% - 監視対象: HW機器（PING・SNMP）, サーバ（LFA・TEMA→TEMS）
    %% - 監視端末: NativeDesktop は ObjectServer へ、WebBrowser は WebGUI へ
    %% ================================================

    %% ---- 監視対象 ----
    subgraph TARGET["監視対象"]
        HW[HW機器群]
        SV[サーバ群]
        LFA[LFA Agent]
        TEMA[TEMA Agent]
        SV --> LFA
        SV --> TEMA
    end

    %% ---- Netcool基盤（Probe同居 / 2台構成）----
    subgraph NETCOOL["Netcool 基盤（2台 ホットスタンバイ）"]
        direction TB

        subgraph NODE1["Node1 プライマリ"]
            PING1[Ping Probe]
            SNMP1[SNMP Probe]
            EIF1[EIF Probe]
            TEMS1[TEMS]
            OS1[ObjectServer Primary]
            WEB[WebGUI / NOI コンソール]
        end

        subgraph NODE2["Node2 スタンバイ"]
            PING2[Ping Probe]
            SNMP2[SNMP Probe]
            EIF2[EIF Probe]
            TEMS2[TEMS]
            OS2[ObjectServer Standby]
        end

    end

    %% ---- 監視端末 ----
    subgraph OP["監視端末"]
        WebBrowser[WebBrowser]
        NativeDesktop[NativeDesktop]
    end

    %% ---- フロー ----
    %% HW機器 → Ping/SNMP（Active優先、Standbyは待機）
    HW --> PING1
    HW --> SNMP1
    HW -.-> PING2
    HW -.-> SNMP2

    %% サーバ → LFA は EIF Probe へ
    LFA --> EIF1
    LFA -.-> EIF2

    %% サーバ → TEMA は TEMS へ（TEMS 経由で取り込み）
    TEMA --> TEMS1
    TEMA -.-> TEMS2

    %% Probe から ObjectServer
    PING1 --> OS1
    SNMP1 --> OS1
    EIF1  --> OS1

    PING2 --> OS2
    SNMP2 --> OS2
    EIF2  --> OS2

    %% 監視端末の接続
    WebBrowser --> WEB
    NativeDesktop --> OS1
    NativeDesktop -.-> OS2
