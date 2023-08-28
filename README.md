# 2FAuth

![Stato della build Docker](https://img.shields.io/github/workflow/status/bubka/2fauth/ci-docker-latest/master?style=flat-square)
![https://codecov.io/gh/Bubka/2FAuth](https://img.shields.io/codecov/c/github/Bubka/2FAuth?style=flat-square)
![https://github.com/Bubka/2FAuth/blob/master/LICENSE](https://img.shields.io/github/license/Bubka/2FAuth.svg?style=flat-square)

Un'applicazione web per gestire i tuoi account di autenticazione a due fattori (2FA) e generare i loro codici di sicurezza

![screens](https://user-images.githubusercontent.com/858858/100485897-18c21400-3102-11eb-9c72-ea0b1b46ef2e.png)

[**Demo 2FAuth**](https://demo.2fauth.app/)  
Credenziali (login - password): *demo@2fauth.app* - *demo*

## Scopo

2FAuth è un'alternativa auto-ospitata basata su web ai generatori di codici monouso (OTP) come Google Authenticator, progettata sia per dispositivi mobili che desktop.

Ha lo scopo di semplificare le tue operazioni di autenticazione 2FA su qualsiasi dispositivo tu stia utilizzando, fornendo un'interfaccia pulita e adatta.

L'ho creato perché:

* La maggior parte delle interfacce utente per questo tipo di app mostrano token per tutti gli account contemporaneamente, con contatori stressanti (a mio parere)
* Volevo che i miei account 2FA fossero memorizzati in un database autonomo che potessi facilmente eseguire il backup e ripristinare (hai mai perso uno smartphone con tutti i tuoi account 2FA in Google Auth? Io sì...)
* Odio dover tirar fuori il mio smartphone per ottenere un OTP quando uso un computer desktop
* Adoro programmare e amo le soluzioni auto-ospitate

## Funzionalità principali

* Gestisci i tuoi account 2FA e organizzali utilizzando Gruppi
* Scansiona e decodifica qualsiasi codice QR per aggiungere un account in pochissimo tempo
* Aggiungi un account personalizzato senza codice QR grazie a un modulo avanzato
* Modifica account, anche quelli importati
* Genera codici di sicurezza TOTP e HOTP e codici di Steam Guard

Attualmente, 2FAuth è completamente localizzato in inglese e francese. Consulta la sezione [Contributi](#contributing) se desideri contribuire aggiungendo altre lingue.

## Sicurezza

2FAuth fornisce diversi meccanismi di sicurezza per proteggere al meglio i tuoi dati 2FA.

### App per utente singolo

Devi creare un account utente e autenticarti per utilizzare l'app. Non è possibile creare più di un account utente, l'app è pensata per un uso personale.

### Autenticazione moderna

Puoi accedere a 2FAuth utilizzando una chiave di sicurezza come una Yubikey o una chiave Titan e disabilitare il modulo di accesso tradizionale.

### Crittografia dei dati

I dati sensibili memorizzati nel database possono essere crittografati per proteggerli in caso di compromissione del database. La crittografia è fornita come opzione disattivata per impostazione predefinita. Si consiglia vivamente di eseguire il backup del valore APP_KEY del tuo file .env (o dell'intero file) quando la crittografia è attiva.

### Logout automatico

2FAuth esegue automaticamente il logout dopo un periodo di inattività per prevenire sessioni prolungate. Il logout automatico può essere disattivato o attivato quando viene copiato un codice di sicurezza.

### Conformità RFC

2FAuth genera OTP in conformità con RFC 4226 (algoritmo HOTP) e RFC 6238 (algoritmo TOTP) grazie alla libreria php [Spomky-Labs/OTPHP](https://github.com/Spomky-Labs/otphp).

## Requisiti

* [![Richiede PHP8](https://img.shields.io/badge/php-^8.1-red.svg?style=flat-square)](https://secure.php.net/downloads.php)
* Consulta i [requisiti del server Laravel](https://laravel.com/docs/7.x/installation#server-requirements)
* Qualsiasi database [supportato da Laravel](https://laravel.com/docs/7.x/database)

## Guide all'installazione

* [Server auto-ospitato](https://docs.2fauth.app/getting-started/installation/self-hosted-server/)

* [Docker (cli)](https://docs.2fauth.app/getting-started/installation/docker/docker-cli/)

* [Docker (compose)](https://docs.2fauth.app/getting-started/installation/docker/docker-compose/)

* [Heroku](https://docs.2fauth.app/getting-started/installation/heroku/)

## Aggiornamento

* [Guida all'aggiornamento](https://docs.2fauth.app/getting-started/upgrade/)

## Migrazione

2FAuth supporta l'importazione dai seguenti formati: 2FAuth (JSON), Google Auth (codice QR), Aegis Auth (JSON, testo semplice), 2FAS Auth (JSON)

* [Guida all'importazione](https://docs.2fauth.app/getting-started/usage/import/)

## Contributi

Puoi contribuire a 2FAuth in molti modi:

* Segnalando [bug](https://github.com/Bubka/2FAuth/issues/new?template=bug_report.md), o ancora meglio, inviando una soluzione con una richiesta pull nel ramo *dev*.
* Suggerendo miglioramenti o nuove funzionalità tramite una [richiesta di nuova funzionalità](https://github.com/Bubka/2FAuth/issues/new?template=feature_request.md). Dai un'occhiata al [progetto di sviluppo di 2FAuth](https://github.com/users/Bubka/projects/1), forse la tua idea è già lì.
* Correggendo o completando traduzioni in una lingua che conosci, utilizzando la piattaforma [Crowdin](https://crowdin.com/project/2fauth). Chiedi per la tua lingua se non è ancora presente.

## Licenza

[AGPL-3.0](https://www.gnu.org/licenses/agpl-3.0.html)
