@startuml
title High-Level Architecture - MaleMail

skinparam linetype ortho
skinparam nodesep 100
skinparam ranksep 130
skinparam defaultTextAlignment center

' Styling af kasser
skinparam package {
    BackgroundColor #F8FAFC
    BorderColor #94A3B8
    FontColor #1E293B
    FontSize 14
}
skinparam component {
    BackgroundColor #EFF6FF
    BorderColor #2563EB
    FontColor #0F172A
    FontSize 14
    BorderThickness 2
}
skinparam database {
    BackgroundColor #DBEAFE
    BorderColor #1D4ED8
    FontColor #0F172A
    BorderThickness 2
}
skinparam cloud {
    BackgroundColor #F1F5F9
    BorderColor #475569
    FontColor #0F172A
    BorderThickness 2
}

' --- Arkitektur Lag ---

package "Presentation Layer" {
    component "**User Interface**\n(Inbox, Composer, Calendar, Settings)" as UI
}

package "Controller Layer" {
    component "**Task Controller**\n(Thread & Event Management)" as Ctrl
}

package "Model Layer" {
    component "**Core Business Logic**\n(Auth, Mail Sync, Spam Filter)" as Core
}

package "Infrastructure & External" {
    database "**Local Persistence**\n(SQLite & File Storage)" as DB
    cloud "**External Services**\n(IMAP, SMTP, OAuth APIs)" as Ext
    
    ' Holder databasen og skyen pænt ved siden af hinanden
    DB -[hidden]right-> Ext
}

' --- Rene, simple pile med masser af usynlig plads (\n) ---

UI <-down-> Ctrl : "\n  User Inputs  \n  & UI Updates  \n"

Ctrl -down-> Core : "\n  Distributes Commands  \n"

Core <-down-> DB : "\n  Read / Write  \n  Cache & Files  \n"

Core <-right-> Ext : "  Network  \n  Communication  "

@enduml