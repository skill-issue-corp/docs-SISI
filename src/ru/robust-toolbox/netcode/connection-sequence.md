# Последовательность подключения RobustToolbox

Это полная последовательность шагов, которые выполняются при подключении клиента в RobustToolbox (и в некоторой степени в Space Station 14). Это невероятно запутанная и беспорядочная система, развивавшаяся годами и имеющая множество движущихся частей и наборов состояний. Уф.

## Базовый обзор

```mermaid
sequenceDiagram
    participant C as Клиент
    participant S as Сервер
    
    C->>S: Попытка подключения Lidgren
    S->>C: Одобрение подключения Lidgren
    
    rect
        Note over C,S: Начальное рукопожатие
        
        C->>S: MsgLoginStart
        Note left of S: Сервер решает, аутентифицировать или нет
        opt Аутентификация
            S->>C: MsgEncryptionRequest
            create participant A as Сервер аутентификации
            C->>A: POST /api/session/join
            C->>S: MsgEncryptionResponse
            %% Я бы использовал <<->>, но в моём Trilium пока нет Mermaid v11.0.0.
            destroy A
            S->A: GET /api/session/hasJoined
        end
        Note over S: Возникает событие Connecting,<br/>контент может отклонить подключение при желании
        S->>C: MsgLoginSuccess
        Note over C,S: Обе стороны включают шифрование при необходимости
        Note over C,S: Обе стороны создают NetChannel
    end
    S->>C: MsgStringTableEntries
    rect
        Note over C,S: Рукопожатие сериализатора
        S->>C: MsgMapStrServerHandshake
        C->>S: MsgMapStrClientHandshake
        opt Клиенту нужны строки
            S->>C: MsgMapStrStrings
            C->>S: MsgMapStrClientHandshake
        end
    end
    Note over S,C: Событие NetManager.Connected<br/>Сообщения, не относящиеся к рукопожатию, теперь разрешены
    C-->>S: MsgConVars
    C-->>S: MsgConCmdReg
    par PlayerManager.NewSession
        Note over S: Создаётся серверная сессия игрока,<br/>PlayerStatusChanged запускается впервые
        S->>C: MsgSyncTimeBase
        Note over S: NetConfigurationManager.SyncConnectingClient
        S->>C: MsgConVars
        Note over C: Создаётся клиентская сессия игрока
        C->>S: MsgPlayerListReq
    and UploadedContentManager
        loop Отправка загруженных ресурсов
            S->>C: NetworkResourceUploadMessage
        end
        loop Отправка загруженных прототипов
            S->>C: GamePrototypeLoadMessage
        end
    end
    Note over S: Сессия игрока устанавливается в Connected
    S->>C: MsgPlayerList<br/>Содержит статус Connected
    Note over C: Сессия игрока устанавливается в Connected
    Note over S: Контент устанавливает клиентскую сессию в InGame
    S->>C: Начинается отправка игровых состояний
    Note over C: Сессия игрока устанавливается в InGame
```
