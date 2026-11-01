@startuml
skinparam style strictuml
skinparam DefaultFontName Arial
skinparam sequenceMessageAlign center

title Sequence Diagram: Calendar Integration & .ics Parsing

actor "User" as user
participant "Email Viewer UI\n(View Layer)" as view
participant "UI Event Controller\n(Controller Layer)" as ctrl
participant "Calendar & .ics Service\n(Model Layer)" as calService
database "SQLite Database\n(Persistence Layer)" as db

user -> view : 1. Clicks "Add to Calendar" on .ics attachment
activate view

view -> ctrl : 2. handleIcsAttachment(filePath)
activate ctrl

ctrl -> calService : 3. parseAndAddEvent(filePath)
activate calService

calService -> calService : 4. Extract event details (Date, Time, Subject)

calService -> db : 5. Insert event into Calendar tables
activate db
db --> calService : 6. Confirmation (Stored locally)
deactivate db

calService --> ctrl : 7. Event successfully added
deactivate calService

ctrl --> view : 8. Trigger Calendar UI update
deactivate ctrl

view --> user : 9. Display success notification & show event
deactivate view

@enduml