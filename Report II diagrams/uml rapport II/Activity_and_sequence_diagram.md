@startuml
title Activity and sequence diagram - MaleMail

skinparam defaultFontName "Segoe UI"
skinparam ActivityBackgroundColor #EFF6FF
skinparam ActivityBorderColor #2563EB
skinparam ActivityDiamondBackgroundColor #FEF3C7
skinparam ActivityDiamondBorderColor #D97706
skinparam ArrowColor #334155

start

:Start Application;
:Read credentials for ALL configured accounts;

if (Any account missing valid token?) then (Yes)
    :Start OAuth2 Login Flow for required accounts;
    :Receive, encrypt, and store new tokens;
else (No)
endif

fork
    :Load saved emails & calendar data from SQLite cache;
    :Show UI with default active account;
fork again
    :Start background thread;
    :Synchronize new emails for ALL accounts via IMAP;
    :Run Spam Filter Engine;
    :Save new emails & update SQLite cache;
    :Update UI;
end fork

repeat :Wait for user input;

if (Which action does the user perform?) then (Compose & Send Email)
    :Select sender account;
    :Receive text and attachments in Composer UI;
    if (Is attachment < 25 MB?) then (Yes)
        :Start background thread;
        :Generate MIME package & add signature;
        :Send via external SMTP server;
        :Update "Sent" folder in SQLite;
        :Show Success notification in UI;
    else (No)
        :Show Error notification in UI;
    endif

elseif (Action?) then (Switch Active Account)
    :Select target account from UI;
    :Load account-specific data from SQLite;
    :Refresh UI with selected account data;

elseif (Action?) then (Open Calendar)
    :Request calendar data;
    :Fetch calendar events (.ics / API) via background thread;
    :Update local calendar cache;
    :Render Calendar View;

elseif (Action?) then (Search / Switch folder)
    :Controller receives search query/folder switch;
    :Execute SQL query in local SQLite;
    :Update UI lists with results;

else (Open specific email)
    :Fetch specific email data from SQLite;
    :Render HTML via QWebEngineView in Display;
endif

repeat while (Does the user close the application?) is (No)
->Yes;

:Clear temporary resources;
stop

@enduml