@startuml
skinparam style strictuml
skinparam DefaultFontName Arial
skinparam sequenceMessageAlign center

title Sequence Diagram: Offline Operations (Email Reading & Local Search)

actor "User" as user
participant "UI (Main / Search)\n(View Layer)" as view
participant "UI Event Controller\n(Controller Layer)" as ctrl
participant "Local Data & Storage Controller\n(Model Layer)" as storage
database "SQLite Database\n(Persistence Layer)" as db

alt Local Search (FR-13)
    user -> view : 1a. Enters search keyword
    activate view
    view -> ctrl : 2a. initiateSearch(keyword)
    activate ctrl
    ctrl -> storage : 3a. queryLocalCache(keyword)
    activate storage
    storage -> db : 4a. SELECT * FROM emails WHERE subject/body LIKE %keyword%
    activate db
    db --> storage : 5a. Return matching emails
    deactivate db
    storage --> ctrl : 6a. Return search results list
    deactivate storage
    ctrl --> view : 7a. Update UI lists with results
    deactivate ctrl
    view --> user : 8a. Display filtered inbox
    deactivate view
else Read Specific Email (Offline)
    user -> view : 1b. Selects cached email from list
    activate view
    view -> ctrl : 2b. requestEmailContent(emailId)
    activate ctrl
    ctrl -> storage : 3b. queryLocalCache(emailId)
    activate storage
    storage -> db : 4b. SELECT metadata, body FROM emails WHERE id = emailId
    activate db
    db --> storage : 5b. Return email data
    deactivate db
    note right of db: No external network calls made\n(NFR-07: Offline Mode)
    storage --> ctrl : 6b. Return parsed email object
    deactivate storage
    ctrl --> view : 7b. Deliver content to UI
    deactivate ctrl
    view -> view : 8b. Render HTML via QWebEngineView
    view --> user : 9b. Display email content seamlessly
    deactivate view
end

@enduml
