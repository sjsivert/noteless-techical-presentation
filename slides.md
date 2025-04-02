---
# You can also start simply with 'default'
theme: seriph
# random image from a curated Unsplash collection by Anthony
# like them? see https://unsplash.com/collections/94734566/slidev
background: https://cover.sli.dev
# some information about your slides (markdown enabled)
title: Noteless Technical Case
info: |
  ## Noteless Technical Case
  Sindre Sivertsen

# apply unocss classes to the current slide
class: text-center
# https://sli.dev/features/drawing
drawings:
  persist: false
# slide transition: https://sli.dev/guide/animations.html#slide-transitions
transition: slide-left
# enable MDC Syntax: https://sli.dev/features/mdc
mdc: true

---


# Noteless Technical Case

Sindre Sivertsen

<div @click="$slidev.nav.next" class="mt-12 py-1" hover:bg="white op-10">
  Press Space for next page <carbon:arrow-right />
</div>

<div class="abs-br m-6 text-xl">
  <button @click="$slidev.nav.openInEditor" title="Open in Editor" class="slidev-icon-btn">
    <carbon:edit />
  </button>
  <a href="https://github.com/slidevjs/slidev" target="_blank" class="slidev-icon-btn">
    <carbon:logo-github />
  </a>
</div>

<!--
The last comment block of each slide will be treated as slide notes. It will be visible and editable in Presenter Mode along with the slide. [Read more in the docs](https://sli.dev/guide/syntax.html#notes)
-->

---
transition: fade-out
---

# Task 1

Scenario:
Noteless is expanding internationally and must support various local authentication systems
(e.g., Norwegian BankID, Danish MitID, Email OTP, and additional local 2FA methods via
Signicat). The platform also needs to manage segmented user profiles for different
professions (e.g, GPs, Physiotherapists, Psychologists) and countries (Norway, Denmark,
Europe).
Objectives:
Design a scalable authentication service that cleanly abstracts multiple methods while
ensuring security, data privacy, and smooth integration with our Django REST Framework
backend. Incorporate local compliance and language-specific flows to support
internationalization and future B2B requirements.
Deliverables:
- A high-level architecture diagram showing how the authentication service interacts
with the React/TypeScript frontend and Django backend, how third-party auth
providers are integrated, and how user segmentation/localization is managed.
- A brief explanation of your design decisions and

---

# Small Scale Architecture (1,500-5,000 users)


```mermaid
flowchart TB
    Client["React/TypeScript Frontend"]
    API["Django REST Framework API"]
    AuthBackend["Custom Django Auth Backends"]
    UserApp["Django User & Profile App"]
    NoteApp["Django Notes App"]
    DB[(PostgreSQL Database)]
    Signicat["Signicat"]
    Redis[(Redis Cache)]

    Client <--> API
    API <--> AuthBackend
    API <--> UserApp
    API <--> NoteApp
    AuthBackend <--> Signicat
    UserApp <--> DB
    NoteApp <--> DB
    API <--> Redis

    subgraph "Django Project"
        API
        AuthBackend
        UserApp
        NoteApp
    end

    subgraph "User Models"
        UserModel["Django User Model"]
        ProfileModel["Profile Model\n- profession\n- country"]
        ProfessionModel["Profession Model\n- terminology\n- AI models"]
        CountryModel["Country Model\n- regulations\n- auth methods"]

        UserModel --- ProfileModel
        ProfileModel --- ProfessionModel
        ProfileModel --- CountryModel
    end
```

<!--
For mindre brukerbaser er en monolitisk Django-applikasjon effektiv og håndterbar:  
**Nøkkelfunksjoner:**

- **Integrated Django Application**: All funksjonalitet er samlet i ett enkelt Django-prosjekt  
- **Custom Authentication Backends**: Djangos autentiseringssystem er utvidet for å støtte Signicat og andre metoder  
- **Flexible Data Models**: Brukerprofiler inkluderer feltene yrke og land for å støtte segmentering  
- **Redis Caching**: Forbedrer ytelsen for ofte etterspurte data  
- **Single Database**: All data lagres i én PostgreSQL-database med korrekt indeksering  

**Fordeler:**

- **Simplicity**: Enklere å utvikle, deploye og vedlikeholde  
- **Developer Productivity**: Djangos admin-grensesnitt og ORM gir rask utvikling  
- **Low Operational Overhead**: Kun én applikasjon å overvåke og skalere  
- **Future-Proofing**: Modellene er designet for å kunne håndtere fremtidig vekst  

**Implementasjonsnotater:**

- Bruk Djangos **custom authentication backends** for å integrere med Signicat  
- Implementer en fleksibel `Profile`-modell med foreign keys til `Profession`- og `Country`-modeller  
- Bruk Django **signals** for å oppdatere relatert data når profiler endres  
- Utnytt Djangos **permission system** for tilgangskontroll basert på yrke
-->

---
transition: fade-out
---

# Medium Scale Architecture (1,500-5,000 users)

```mermaid
flowchart TB
    Client["React/TypeScript Frontend"]
    APIGateway["API Gateway / Load Balancer"]
    AuthService["Authentication Service"]
    DjangoAPI["Django REST Framework API"]
    UserApp["Django User App"]
    NoteApp["Django Notes App"]
    DomainService["Professional Domain Service"]
    MainDB[(PostgreSQL Main DB)]
    AuthDB[(Auth Database)]
    DomainDB[(Domain-Specific DB)]
    Signicat["Signicat"]
    Redis[(Redis Cache)]
    
    Client <--> APIGateway
    APIGateway <--> AuthService
    APIGateway <--> DjangoAPI
    AuthService <--> Signicat
    AuthService <--> AuthDB
    AuthService <--> UserApp
    DjangoAPI <--> UserApp
    DjangoAPI <--> NoteApp
    UserApp <--> MainDB
    NoteApp <--> MainDB
    DjangoAPI <--> DomainService
    DomainService <--> DomainDB
    DjangoAPI <--> Redis
    
    subgraph "Core Django System"
        DjangoAPI
        UserApp
        NoteApp
    end
    
    subgraph "External Services"
        AuthService
        DomainService
    end
    
    subgraph "Domain-Specific Logic"
        GPLogic["GP-Specific Logic"]
        PhysioLogic["Physiotherapy Logic"]
        PsychLogic["Psychology Logic"]
        
        DomainService --- GPLogic
        DomainService --- PhysioLogic
        DomainService --- PsychLogic
    end

```
<!--
Etter hvert som brukerbasen vokser, begynner en hybrid tilnærming å trekke ut kritiske tjenester:  
**Nøkkelfunksjoner:**

- **API Gateway**: Ruter forespørsler til riktige tjenester  
- **Separate Authentication Service**: Håndterer kompleksiteten med flere autentiseringsmetoder  
- **Core Django Application**: Håndterer fortsatt brukerprofiler og hovedlogikken i applikasjonen  
- **Domain-Specific Service**: Trekker ut yrkesspesifikk logikk i en egen tjeneste  
- **Multiple Databases**: Skiller autentiseringsdata fra hoveddataene i applikasjonen  

**Fordeler:**

- **Improved Scalability**: Autentisering og domenespesifikk logikk kan skaleres uavhengig  
- **Better Separation of Concerns**: Kompleksitet rundt autentisering isoleres fra hovedapplikasjonen  
- **Flexible Growth**: Tjenestene kan utvikles i ulikt tempo  
- **Incremental Migration**: Funksjonalitet kan gradvis flyttes fra Django til separate tjenester  

**Implementasjonsnotater:**

- **Authentication service** utsteder JWT-er som Django validerer  
- **Domain service** håndterer yrkesspesifikke termer og arbeidsflyt  
- Django håndterer fortsatt kjerneprofiler, men koordinerer med eksterne tjenester  
- Databasetilkoblinger er optimalisert for ulike tilgangsmønstre
-->

---
transition: slide-up
level: 2
---

# Large Scale Architecture (15,000+ users)


```mermaid
flowchart TB
    Client["React/TypeScript Frontend"]
    APIGateway["API Gateway"]
    AuthService["Authentication Service"]
    UserService["User Management Service"]
    NoteService["Notes Service"]
    AIService["AI Transcription Service"]
    DomainServices["Professional Domain Services"]
    AuthDB[(Auth Database)]
    UserDB[(User Database)]
    NoteDB[(Notes Database)]
    Signicat["Signicat"]
    Redis[(Redis Cache)]
    MessageQueue[(Kafka/RabbitMQ)]
    
    Client <--> APIGateway
    APIGateway <--> AuthService
    APIGateway <--> UserService
    APIGateway <--> NoteService
    APIGateway <--> AIService
    APIGateway <--> DomainServices
    
    AuthService <--> Signicat
    AuthService <--> AuthDB
    UserService <--> UserDB
    NoteService <--> NoteDB
    
    AuthService <--> MessageQueue
    UserService <--> MessageQueue
    NoteService <--> MessageQueue
    AIService <--> MessageQueue
    DomainServices <--> MessageQueue
    
    subgraph "Microservices"
        AuthService
        UserService
        NoteService
        AIService
        DomainServices
    end
    
    subgraph "Profession-Specific Services"
        GPService["GP Service"]
        PhysioService["Physiotherapy Service"]
        PsychService["Psychology Service"]
        
        DomainServices --- GPService
        DomainServices --- PhysioService
        DomainServices --- PsychService
    end
    
    subgraph "Regional Deployment"
        EUCluster["EU Data Center"]
        NordicCluster["Nordic Data Center"]
        UKCluster["UK Data Center"]
    end

```

<!-- 

For storskala operasjoner på tvers av flere land er en full microservice-arkitektur passende:
Nøkkelfunksjoner:

Complete Microservice Architecture: Hver hovedfunksjon er sin egen tjeneste

Dedicated User Management Service: Håndterer alle aspekter av brukerprofiler og segmentering

Message Queue Integration: Tjenester kommuniserer asynkront for bedre robusthet

Regional Deployment: Tjenester kan deployes i ulike regioner for å møte krav til etterlevelse

Profession-Specific Services: Hver medisinsk profesjon har dedikert forretningslogikk

Fordeler:

Maximum Scalability: Hver tjeneste kan skaleres uavhengig basert på behov

Geographic Distribution: Tjenester kan deployes nær brukere i ulike regioner

Enhanced Resilience: Feil i én tjeneste påvirker ikke de andre

Specialized Teams: Ulike team kan fokusere på ulike tjenester

Regulatory Compliance: Krav til datalagring kan møtes med regionale deployeringer

Implementasjonsnotater:

Authentication service utsteder kortlevde access tokens og lengre refresh tokens

User service vedlikeholder fullstendig profilinformasjon inkludert profesjon og land

Domain services implementerer spesialisert logikk for hver medisinsk profesjon

Event-driven architecture muliggjør sanntidsoppdateringer på tvers av tjenester
-->

---
layout: center
---

# Authentication Flow Comparison 1


```mermaid {scale: 0.7, theme: 'dark'}
sequenceDiagram
    participant C as Client
    participant S1 as Small Scale<br>(Django Auth)
    participant Sig as Signicat
    
    Note over C,S1: Small Scale Flow (1,500-5,000 users)
    C->>S1: Login Request
    S1->>Sig: Redirect to Signicat
    Sig->>C: Authentication UI
    C->>Sig: Provide Credentials
    Sig->>S1: Auth Token
    S1->>S1: Create Django Session
    S1->>C: Session Cookie
```

---
layout: center
---

# Authentication Flow Comparison2

```mermaid {scale: 0.7, theme: 'dark'}
sequenceDiagram
    participant C as Client
    participant S2 as Medium Scale<br>(Auth Service + Django)
    participant Sig as Signicat
    
    Note over C,S2: Medium Scale Flow (5,000-15,000 users)
    C->>S2: Login Request
    S2->>Sig: Redirect to Signicat
    Sig->>C: Authentication UI
    C->>Sig: Provide Credentials
    Sig->>S2: Auth Token
    S2->>S2: Generate JWT Token
    S2->>C: JWT Token

```

---
layout: center
---
# Authentication Flow Comparison3

```mermaid {scale: 0.6 , theme: 'dark'}
sequenceDiagram
    participant C as Client
    participant S3 as Large Scale<br>(Microservices)
    participant Sig as Signicat
    
    Note over C,S3: Large Scale Flow (15,000+ users)
    C->>S3: Login Request
    S3->>Sig: Redirect to Signicat
    Sig->>C: Authentication UI
    C->>Sig: Provide Credentials
    Sig->>S3: Auth Token
    S3->>S3: Generate Short-Lived Access Token
    S3->>S3: Generate Refresh Token
    S3->>C: Access + Refresh Tokens
```

<!-- 

The fourth diagram shows how the authentication flow evolves across the different scales:

Small Scale: Simple session-based authentication with cookies
Medium Scale: JWT-based authentication with a separate authentication service
Large Scale: Sophisticated token management with access and refresh tokens

All three approaches support integration with Signicat for national ID authentication, but with increasing levels of sophistication in token handling and session management.

-->

---

# Task 2
Scenario:
Noteless wants to build a modular architecture for AI features to be able to quickly iterate and
implement new models and features. The company currently uses different AI models hosted
on providers like Azure ML and GCP Vertex. The goal is to be able to quickly implement, test,
and roll out new models and features for specific user groups.
Objectives:
Propose a strategy that allows easy addition or swapping of AI models and services. Also,
design the solution to support A/B testing, feature flags, and KPI monitoring (for example,
user feedback and note accuracy).
Deliverables:
- An integration diagram that illustrates how our Django/DRF backend will call the
various AI services, the data flow from raw audio to structured notes, and the control
points for testing and feature management.
- A short explanation describing your approach to balan

---

# AI 

```mermaid {scale: 0.5 , theme: 'dark'}
flowchart TD
    Frontend["Frontend\n(React/TypeScript)"] <--> APIGateway["API Gateway"]
    APIGateway <--> ReportingService["Reporting Service"]
    
    APIGateway <--> DjangoBackend["Django Backend"]
    APIGateway <--> FeatureManager["Feature Manager Service"]
    
    DjangoBackend <--> Database[(Database)]
    DjangoBackend --> ServiceBus["Service Bus/Message Queue"]
    
    ServiceBus --> AIServices["AI Services\n(Orchestration Layer)"]
    
    AIServices <--> FeatureManager
    AIServices <--> ReportingService
    AIServices <--> AI1["AI 1"]
    AIServices <--> AI2["AI 2"]
    AIServices <--> AI3["AI 3"]
    
    subgraph "Client Layer"
        Frontend
    end
    
    subgraph "API Layer"
        APIGateway
    end
    
    subgraph "Application Layer"
        DjangoBackend
        Database
    end
    
    subgraph "Message Layer"
        ServiceBus
    end
    
    subgraph "AI Processing Layer"
        AIServices
        AI1
        AI2
        AI3
    end
    
    subgraph "Feature Management"
        FeatureManager
    end
    
    subgraph "Reporting & Analytics"
        ReportingService
    end
    
    classDef frontend fill:#e3f2fd,stroke:#2196f3,stroke-width:2px
    classDef api fill:#e8f5e9,stroke:#4caf50,stroke-width:2px
    classDef app fill:#fff3e0,stroke:#ff9800,stroke-width:2px
    classDef message fill:#f3e5f5,stroke:#9c27b0,stroke-width:2px
    classDef ai fill:#e0f7fa,stroke:#00bcd4,stroke-width:2px
    classDef feature fill:#ffebee,stroke:#f44336,stroke-width:2px
    classDef reporting fill:#f5f5f5,stroke:#607d8b,stroke-width:2px
    
    class Frontend frontend
    class APIGateway api
    class DjangoBackend,Database app
    class ServiceBus message
    class AIServices,AI1,AI2,AI3 ai
    class FeatureManager feature
    class ReportingService reporting
```

---

# AI flow sequence diagram

```mermaid {scale: 0.23}
sequenceDiagram
    participant F as Frontend
    participant D as Django Backend
    participant SB as Service Bus
    participant AIS as AI Services
    participant FM as Feature Manager
    participant AI1 as AI Service 1
    participant AI2 as AI Service 2
    participant AI3 as AI Service 3
    participant RS as Reporting Service
    participant WS as WebSocket Channel

    F->>+D: POST /transcribe with audio file
    D->>D: Generate request_id
    D->>+SB: Put transcription request message
    Note over D,SB: {request_id, audio_file, metadata}
    D->>-F: 202 Accepted with WebSocket info
    Note over D,F: {request_id, websocket_channel}
    
    F->>WS: Connect to WebSocket channel
    Note over F,WS: Subscribe to updates for request_id

    SB->>+AIS: Deliver transcription request message
    AIS->>+FM: Get feature config for user/request
    FM->>-AIS: Return AI service selection
    
    alt AI Service 1 Selected
        AIS->>+AI1: Send transcription request
        AI1->>-AIS: Return transcription result
    else AI Service 2 Selected
        AIS->>+AI2: Send transcription request
        AI2->>-AIS: Return transcription result
    else AI Service 3 Selected
        AIS->>+AI3: Send transcription request
        AI3->>-AIS: Return transcription result
    end
    
    AIS->>+RS: Report tokens and cost
    Note over AIS,RS: {request_id, model, tokens, cost}
    RS->>-AIS: Acknowledge
    
    AIS->>-SB: Put completed transcription message
    Note over AIS,SB: {request_id, transcription_result}
    
    SB->>+D: Deliver completed transcription message
    D->>+WS: Publish result to WebSocket channel
    Note over D,WS: {request_id, transcription_result}
    WS->>-F: Deliver transcription result
    
    F->>+RS: Report user KPIs
    Note over F,RS: {request_id, satisfaction_score, edit_count}
    RS->>-F: Acknowledge

```


---
layout: center

---

# Multi armed bandit Flowchart

```mermaid
flowchart TB
    MainServiceBus["Main Service Bus"] <--> AIServices["AI Services\n(Orchestration Layer)"]
    
    AIServices <--> FeatureManager["Feature Manager Service"]
    AIServices <--> ReportingService["Reporting Service"]
    
    AIServices <--> AIServiceBus["AI Service Bus"]
    
    AIServiceBus <--> AI1["AI Microservice 1\n(e.g., Azure Transcription)"]
    AIServiceBus <--> AI2["AI Microservice 2\n(e.g., GCP Medical NER)"]
    AIServiceBus <--> AI3["AI Microservice 3\n(e.g., OpenAI Note Structuring)"]
    
    subgraph "External Systems"
        FeatureManager
        ReportingService
    end
    
    subgraph "AI Processing Layer"
        AIServices
        AIServiceBus
        AI1
        AI2
        AI3
    end
    
    classDef orchestration fill:#e0f7fa,stroke:#00bcd4,stroke-width:2px
    classDef bus fill:#f3e5f5,stroke:#9c27b0,stroke-width:2px
    classDef microservice fill:#bbdefb,stroke:#1976d2,stroke-width:2px
    classDef external fill:#ffebee,stroke:#f44336,stroke-width:2px
    
    class AIServices orchestration
    class AIServiceBus,MainServiceBus bus
    class AI1,AI2,AI3 microservice
    class FeatureManager,ReportingService external
```

---

# Multi armed sequence diagram

```mermaid {scale: 0.35}
sequenceDiagram
    participant AIO as AI Orchestrator
    participant SB as Service Bus
    participant RS as Reporting Service
    participant AI1 as AI Service 1
    participant AI2 as AI Service 2
    participant AI3 as AI Service 3

    AIO->>RS: Get AI service statistics
    RS->>AIO: Return usage statistics
    Note over RS,AIO: Times shown, times chosen,<br/>success rate, avg cost

    AIO->>SB: Publish "need: transcribe_notes"
    Note over AIO,SB: {request_id, audio_data}
    
    SB->>AI1: Deliver request
    SB->>AI2: Deliver request
    SB->>AI3: Deliver request
    
    AI1->>SB: Publish capability response
    AI2->>SB: Publish capability response
    AI3->>SB: Publish capability response
    
    SB->>AIO: Deliver all responses
    
    AIO->>AIO: Calculate scores and select provider
    Note over AIO: Based on capabilities and<br/>historical statistics
    
    AIO->>SB: Publish task assignment
    Note over AIO,SB: {selected_provider: "AI2"}
    
    SB->>AI2: Deliver task assignment
    
    AI2->>SB: Publish transcription result
    
    SB->>AIO: Deliver result
    
    AIO->>RS: Report selection and usage
```
