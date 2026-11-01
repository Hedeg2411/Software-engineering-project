@startuml
skinparam style strictuml
skinparam DefaultFontName Arial
skinparam sequenceMessageAlign center

title Sequence Diagram: Email Composition & Sending (with Attachment Check)

actor "User" as user
participant "Composer UI\n(View Layer)" as view
participant "UI Event Controller\n(Controller Layer)" as ctrl
participant "Outbound & MIME Service\n(Model Layer)" as outbound
database "SQLite Database\n(Persistence Layer)" as db
participant "External Mail Servers\n(SMTP)" as smtp

user -> view : 1. Writes email & attaches file
activate view

view -> ctrl : 2. validateAttachment(fileSize)
activate ctrl

alt Attachment > 25MB
    ctrl --> view : 3a. Validation Failed (Limit Exceeded)
    view --> user : 4a. Display File Size Warning
else Attachment <= 25MB
    ctrl --> view : 3b. Validation Passed

    user -> view : 5. Clicks "Send"

    view -> ctrl : 6. initiateSendEmail(payload)

    ctrl -> outbound : 7. Start Async Thread (MIME packaging)
    activate outbound

    outbound -> smtp : 8. Transmit outgoing payload (SMTP)
    activate smtp
    smtp --> outbound : 9. Confirmation (Message Accepted)
    deactivate smtp

    outbound -> db : 10. Save copy to Sent Folder metadata
    activate db
    db --> outbound : 11. Confirmation (Stored in Cache)
    deactivate db

    outbound --> ctrl : 12. Task Complete Signal
    deactivate outbound

    ctrl --> view : 13. Clear Composer UI State
    deactivate ctrl

    view --> user : 14. Display "Email Sent" Notification
end
deactivate view

@enduml