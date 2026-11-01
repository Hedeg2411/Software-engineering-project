@startuml
skinparam style strictuml
skinparam DefaultFontName Arial
skinparam ActivityBackgroundColor #EAEAEA
skinparam ActivityBorderColor #333333

title Activity Diagram: Asynchronous Inbound & Sync Process

|View Layer|
start
:Trigger Inbox Sync (User or Auto);

|Controller Layer (QThread)|
:Start Async Background Thread;
note right: Ensures UI responsiveness (NFR-03)

|Inbound & Sync Service|
:Fetch incoming emails via IMAP;
:Parse MIME body and attachments;

|Spam Filter Engine|
:Evaluate against local spam rules;
if (Is flagged as Spam?) then (yes)
  :Set folder = Spam;
else (no)
  :Set folder = Inbox;
endif

|Local Data & Storage Controller|
:Write email metadata & body to SQLite;
:Cache attachments to Local File Store;
note right: Local caching for offline use (NFR-02)

|Controller Layer (QThread)|
:Emit 'Sync Complete' signal;
:Terminate Background Thread;

|View Layer|
:Update UI Lists with new cached emails;
stop

@enduml