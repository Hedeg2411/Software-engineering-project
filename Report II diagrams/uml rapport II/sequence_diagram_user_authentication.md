@startuml
skinparam style strictuml
skinparam DefaultFontName Arial
skinparam sequenceMessageAlign center

title Sequence Diagram: User Authentication & OAuth2 Handshake

actor "User" as user
participant "Login UI\n(View Layer)" as view
participant "Auth & Session Manager\n(Model Layer)" as auth
participant "Google / MS Entra ID\n(External API)" as api
database "Local File & Token Store\n(Persistence Layer)" as db

user -> view : 1. Selects "Sign in with Google/Microsoft"
activate view

view -> auth : 2. Initiate OAuth2 login(provider)
activate auth

auth -> api : 3. Request Authorization URL & Client ID
activate api
api --> auth : 4. Return Auth URL
deactivate api

auth -> view : 5. Open browser for user consent
view -> user : 6. Display external login page in browser

user -> api : 7. Enters credentials & approves access
activate api
api --> auth : 8. Return OAuth2 Access & Refresh Tokens
deactivate api

auth -> auth : 9. Encrypt tokens

auth -> db : 10. Store encrypted tokens locally
activate db
db --> auth : 11. Confirmation (Stored)
deactivate db

auth --> view : 12. Authentication Successful
deactivate auth

view -> view : 13. Load Main Page & Inbox
view --> user : 14. Display Main UI
deactivate view

@enduml