@startuml
title Component diagram - MaleMail

' --- Styling ---
skinparam componentStyle uml2
skinparam packageStyle rectangle
skinparam linetype ortho
skinparam nodesep 45
skinparam ranksep 55
skinparam defaultFontName "Segoe UI"
skinparam defaultFontSize 12

skinparam package {
    BackgroundColor #F8FAFC
    BorderColor #94A3B8
    FontColor #1E293B
}

skinparam component {
    BackgroundColor #EFF6FF
    BorderColor #2563EB
    FontColor #0F172A
    ArrowColor #334155
}

skinparam database {
    BackgroundColor #DBEAFE
    BorderColor #1D4ED8
}

skinparam folder {
    BackgroundColor #DBEAFE
    BorderColor #1D4ED8
}

skinparam cloud {
    BackgroundColor #F1F5F9
    BorderColor #475569
}

top to bottom direction

' ==========================================
' 1. VIEW LAYER (PySide6 GUI)
' ==========================================
package "View Layer (PySide6 GUI)" {
    component "**Login UI**\n---\n[FR-01, FR-04, OPT-01]" as LoginUI
    component "**Settings UI**\n---\n[FR-09, OPT-01, OPT-03]" as SettingsUI
    component "**Main, Folder & Search UI**\n---\n[FR-10, FR-11, FR-12]\n[FR-13, OPT-01]" as MainUI
    component "**Email Viewer**\n(QWebEngineView)\n---\n[FR-08, FR-14, OPT-02]" as ViewerUI
    component "**Composer UI**\n---\n[FR-05, FR-07, FR-08]\n[FR-09, OPT-01, OPT-02]" as ComposerUI
    component "**Calendar UI**\n---\n[OPT-02]" as CalendarUI
    
    ' Holder GUI-komponenter på en pæn række
    LoginUI -[hidden]right-> SettingsUI
    SettingsUI -[hidden]right-> MainUI
    MainUI -[hidden]right-> ViewerUI
    ViewerUI -[hidden]right-> ComposerUI
    ComposerUI -[hidden]right-> CalendarUI
}

' ==========================================
' 2. CONTROLLER LAYER (Domain-Specific)
' ==========================================
package "Controller Layer" {
    component "**AuthController**\n(Handles Login & Auth Signals)\n---\n[NFR-01]" as AuthCtrl
    component "**MailController**\n(Handles Mail & Sync Events)\n---\n[NFR-01, NFR-03]" as MailCtrl
    component "**CalendarController**\n(Handles Calendar Events)\n---\n[NFR-01]" as CalCtrl

    AuthCtrl -[hidden]right-> MailCtrl
    MailCtrl -[hidden]right-> CalCtrl
}

' ==========================================
' 3. MODEL LAYER (Business Logic & Networking)
' ==========================================
package "Model Layer (Python Core)" {
    component "**Auth & Session Manager**\n---\n[FR-01, FR-02, FR-03, FR-04, OPT-01]" as AuthMgr
    
    component "**Inbound & Sync Service**\n---\n[FR-06, FR-08, FR-10, FR-11, FR-12, OPT-01]" as InboundSvc
    
    component "**Spam Filter Engine**\n---\n[OPT-03]" as SpamFilter
    
    component "**Outbound & MIME Service**\n---\n[FR-05, FR-07, FR-08, FR-09, OPT-01, OPT-02]" as OutboundSvc
    
    component "**Calendar & .ics Service**\n---\n[OPT-02]" as CalendarSvc

    component "**Local Data & Storage Controller**\n---\n[FR-06, FR-08..13, OPT-01..03]" as DataCtrl
}

' ==========================================
' 4. PERSISTENCE & EXTERNAL SERVICES
' ==========================================
package "Local Persistence Layer" {
    folder "**Local File & Token Store**\n(os / pathlib / credentials.json)\n---\n[FR-02, FR-03, FR-08, OPT-01, OPT-02]" as FileStore
    database "**SQLite Database**\n(Emails, Flags, Signatures, Blacklist)\n---\n[FR-09..13, OPT-01..03]" as SQLiteDB
}

package "External Services & Protocols" {
    cloud "**Google Cloud & MS Entra ID**\n(OAuth2 & Calendar APIs)" as OAuthCloud
    cloud "**External Mail Servers**\n(IMAP / SMTP)" as MailServer
}

' ==========================================
' DEPENDENCIES (Pile)
' ==========================================

' View -> Controller Mappings
LoginUI --> AuthCtrl
SettingsUI --> MailCtrl
MainUI --> MailCtrl
ViewerUI --> MailCtrl
ComposerUI --> MailCtrl
CalendarUI --> CalCtrl

' Controller -> Model Mappings
AuthCtrl --> AuthMgr 
MailCtrl --> InboundSvc 
MailCtrl --> OutboundSvc 
MailCtrl --> DataCtrl
CalCtrl --> CalendarSvc 

' Internt i Model Layer
InboundSvc --> AuthMgr
OutboundSvc --> AuthMgr
CalendarSvc --> AuthMgr

InboundSvc --> SpamFilter
SpamFilter --> DataCtrl

InboundSvc --> DataCtrl
OutboundSvc --> DataCtrl
CalendarSvc --> DataCtrl

' Model -> Persistence Layer
AuthMgr --> FileStore
DataCtrl --> FileStore
DataCtrl --> SQLiteDB

' Model -> External Services
AuthMgr --> OAuthCloud 
CalendarSvc --> OAuthCloud 
InboundSvc --> MailServer 
OutboundSvc --> MailServer 

@enduml