@startuml
allowmixing
hide circle
skinparam linetype ortho
skinparam nodesep 160
skinparam ranksep 130
skinparam classAttributeIconSize 0

' Styling
skinparam class {
    BackgroundColor White
    HeaderBackgroundColor #D1EDF9
    BorderColor #00AEEF
    BorderThickness 2
    ArrowColor Black
    FontSize 15
    AttributeFontSize 13
}

skinparam rectangle {
    BackgroundColor transparent
    BorderColor transparent
    Shadowing false
    FontSize 14
}

skinparam database {
    BackgroundColor White
    BorderColor #00AEEF
    BorderThickness 2
    ArrowColor Black
}

skinparam cloud {
    BackgroundColor White
    BorderColor #00AEEF
    BorderThickness 2
    ArrowColor Black
}

' User inputs
rectangle "User\ninputs" as Input

' Controller
class Controller {
    - activeThread: QThread
    - isProcessing: bool
    --
    + handleSignal(event: String): void
    - manageAsyncThread(task: Callable): void
    + validateAttachment(fileSize: int): boolean
    + initiateSendEmail(emailData: Dict): void
}

' View
class View {
    - currentFolder: String
    - selectedEmailId: int
    --
    + updateUILists(emails: List): void
    + renderHTML(htmlContent: String): void
    + displayNotification(message: String): void
    + clearUIState(): void
}

' Model
class Model {
    - dbConnection: Connection
    - oauthToken: String
    --
    + authenticateUser(provider: String): boolean
    + syncNetworkIMAP_SMTP(): boolean
    + sendEmailMIME(payload: Dict): boolean
    + queryLocalCache(query: String): List
    - evaluateSpamRules(email: Object): boolean
}

' Cache and Servers
database "Local Cache\n(SQLite / Files)" as Cache
cloud "External Servers\n(IMAP / SMTP / OAuth2)" as Servers

' Core MVC flow
Input -right-> Controller
Controller -right-> View : " view modification\n messages"
Controller -down-> Model : " Model edits"
View <--> Model : " Model queries\n and updates"

' Database and Network flow
Model -down-> Cache : " Read / Write queries"
Model -right-> Servers : " Server communication"

@enduml