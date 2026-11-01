@startuml
' Fjernet linetype ortho for at tillade adskilte, parallelle pile
skinparam nodesep 120
skinparam ranksep 120

' Styling til DFD-elementer
skinparam rectangle {
    BackgroundColor #F8FAFC
    BorderColor #475569
    BorderThickness 2
    FontColor #0F172A
    roundCorner 10
}

skinparam usecase {
    BackgroundColor #E0F2FE
    BorderColor #0284C7
    BorderThickness 2
    FontColor #0F172A
}

skinparam database {
    BackgroundColor #FEF3C7
    BorderColor #D97706
    BorderThickness 2
    FontColor #0F172A
}

' 1. External Entities
rectangle "User" as user
rectangle "External Mail Servers\n(IMAP / SMTP)" as mailServers
rectangle "OAuth Providers\n(Google / MS Entra)" as oauth

' 2. Processes
usecase "1.0\nManage UI\n& User Inputs" as p1
usecase "2.0\nProcess & Route\nEmails" as p2
usecase "3.0\nHandle\nAuthentication" as p3

' 3. Data Stores
database "D1: SQLite Database\n(Cache & Metadata)" as db
database "D2: Local File Storage\n(Tokens & Attachments)" as fs

' --- Data Flows ---

' Horisontal akse (Buede parallelle pile)
user -right-> p1 : " Input actions\n(clicks, text)"
user <-right- p1 : " UI displays\n(emails, alerts)"

p1 -right-> p2 : " Email commands\n(send, sync)"
p1 <-right- p2 : " Processed\nemail data"

p2 -right-> db : " Write/Read\nmetadata, flags"

' Vertikal akse for Email Process
p2 -up-> mailServers : " Outgoing MIME\npackages (SMTP)"
p2 <-up- mailServers : " Incoming email\ndata (IMAP)"

p2 -down-> fs : " Save/Load\nattachments"

' Vertikal akse for UI & Auth
p1 -down-> p3 : " Login requests"

p3 -down-> oauth : " Auth requests"
p3 <-down- oauth : " Access/Refresh\ntokens"

' Forbindelse fra Auth til File Storage
p3 -right-> fs : " Store encrypted\ntokens"

@enduml