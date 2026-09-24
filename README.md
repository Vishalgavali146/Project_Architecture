flowchart TD
    classDef userStyle fill:#1a1a2e,stroke:#e94560,color:#fff,font-weight:bold
    classDef frontendStyle fill:#16213e,stroke:#0f3460,color:#a8d8ea,font-weight:bold
    classDef backendStyle fill:#0f3460,stroke:#533483,color:#fff,font-weight:bold
    classDef middlewareStyle fill:#533483,stroke:#e94560,color:#fff
    classDef dbStyle fill:#1b4332,stroke:#52b788,color:#b7e4c7,font-weight:bold
    classDef cacheStyle fill:#7f4f24,stroke:#e9c46a,color:#fefae0,font-weight:bold
    classDef infraStyle fill:#212529,stroke:#6c757d,color:#adb5bd

    %% ─────────────────────────────────────────
    %% LAYER 1 — USER
    %% ─────────────────────────────────────────
    subgraph USER["👤 USER LAYER"]
        U1(["Lab Assistant"])
        U2(["Lab Incharge"])
        U3(["DSR Incharge"])
        U4(["HOD"])
        U5(["Principal"])
    end

    %% ─────────────────────────────────────────
    %% LAYER 2 — FRONTEND (Client)
    %% ─────────────────────────────────────────
    subgraph FRONTEND["🖥️ FRONTEND — React + Vite"]
        FE1["UI Components\nChakra UI + TailwindCSS"]
        FE2["Pages & Routing\nReact Router v7"]
        FE3["Animations\nFramer Motion"]
        FE4["Role-Based UI\nHide/Show by Role"]
        FE5["API Client\nAxios + JWT in Header"]
        FE6["Token Storage\nlocalStorage (JWT)"]
    end

    %% ─────────────────────────────────────────
    %% LAYER 3 — BACKEND (Server)
    %% ─────────────────────────────────────────
    subgraph BACKEND["⚙️ BACKEND — Node.js + Express.js v5"]

        subgraph SECURITY["🔐 Security & Middleware Layer"]
            MW1["CORS\nAllow only frontend origin"]
            MW2["Rate Limiter\nexpress-rate-limit"]
            MW3["JWT Auth Middleware\nVerify Token Signature"]
            MW4["Role Guard Middleware\nAuth-HOD / Auth-Incharge etc."]
            MW5["Zod Validation\nValidate request body shape"]
        end

        subgraph ROUTES["🛣️ Routes Layer"]
            R1["RegisterRoute.js"]
            R2["RequisitionRouter.js"]
            R3["StaffRouter.js"]
            R4["ApprovalEquipment.js"]
            R5["BudgetAnalysis.js"]
            R6["NotificationRouter.js"]
        end

        subgraph CONTROLLERS["🧠 Controllers — Business Logic"]
            C1["LoginController\nBcrypt verify + JWT sign"]
            C2["RequisitionController\nApproval Workflow Logic"]
            C3["DepartmentLabCreationController"]
            C4["StaffController"]
            C5["BudgetController"]
            C6["NotificationController"]
            C7["FetchDepartmentLabs\nCache-Aside Pattern"]
        end
    end

    %% ─────────────────────────────────────────
    %% LAYER 4 — DATA LAYER
    %% ─────────────────────────────────────────
    subgraph DATA["🗄️ DATA LAYER"]

        subgraph CACHE["⚡ Redis Cache (Planned)"]
            RD1[("Redis\nIn-Memory Store")]
            RD2["Cache Key\ndepartment_labs:deptId"]
            RD3["TTL: 1800s (30 min)"]
        end

        subgraph DATABASE["🍃 MongoDB Atlas"]
            DB1[("MongoDB\nDatabase")]
            DB2["Users Collection\nrole / dept / lab"]
            DB3["Department Collection\nbudget / labs"]
            DB4["Equipment Collection\nPO / Invoice / DSR Nos."]
            DB5["EachEquipment Collection\nSerial No / Status"]
            DB6["Requisition Collection\nApproval States"]
            DB7["HistoryCard Collection\nMaintenance Logs"]
        end
    end

    %% ─────────────────────────────────────────
    %% LAYER 5 — INFRASTRUCTURE
    %% ─────────────────────────────────────────
    subgraph INFRA["🚀 INFRASTRUCTURE (Planned)"]
        I1["Docker\nContainerize Server + Client"]
        I2["docker-compose.yml\nOrchestrate all services"]
        I3["Nginx\nServe React build files"]
    end

    %% ─────────────────────────────────────────
    %% DATA FLOW ARROWS
    %% ─────────────────────────────────────────

    U1 & U2 & U3 & U4 & U5 -->|"Clicks UI"| FE1
    FE1 --> FE2 --> FE3 --> FE4
    FE4 -->|"Axios + Bearer Token"| FE5
    FE5 -->|"HTTP Request"| MW1

    MW1 --> MW2 --> MW3 --> MW4 --> MW5
    MW5 --> R1 & R2 & R3 & R4 & R5 & R6

    R1 --> C1
    R2 --> C2
    R3 --> C4
    R4 --> C3
    R5 --> C5
    R6 --> C6
    R2 --> C7

    C7 -->|"1. Check Redis First"| RD1
    RD1 -->|"Cache Hit ⚡"| C7
    RD1 -->|"Cache Miss 🔴"| DB1
    C7 -->|"2. SETEX (save copy)"| RD2
    RD2 --> RD3

    C1 & C2 & C3 & C4 & C5 & C6 -->|"Mongoose Queries"| DB1
    DB1 --- DB2 & DB3 & DB4 & DB5 & DB6 & DB7

    DB1 -->|"JSON Response"| CONTROLLERS
    CONTROLLERS -->|"res.json()"| FE5
    FE5 -->|"Updates State"| FE1
    FE1 -->|"Renders UI"| U1

    BACKEND --> I1
    FRONTEND --> I3
    I1 & I3 --> I2

    class U1,U2,U3,U4,U5 userStyle
    class FE1,FE2,FE3,FE4,FE5,FE6 frontendStyle
    class MW1,MW2,MW3,MW4,MW5 middlewareStyle
    class R1,R2,R3,R4,R5,R6,C1,C2,C3,C4,C5,C6,C7 backendStyle
    class DB1,DB2,DB3,DB4,DB5,DB6,DB7 dbStyle
    class RD1,RD2,RD3 cacheStyle
    class I1,I2,I3 infraStyle
