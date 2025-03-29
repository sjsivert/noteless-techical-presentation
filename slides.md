---
# You can also start simply with 'default'
theme: seriph
# random image from a curated Unsplash collection by Anthony
# like them? see https://unsplash.com/collections/94734566/slidev
background: https://cover.sli.dev
# some information about your slides (markdown enabled)
title: Welcome to Slidev
info: |
  ## Slidev Starter Template
  Presentation slides for developers.

  Learn more at [Sli.dev](https://sli.dev)
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

# Welcome to Slidev

Presentation slides for developers

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

layout: two-cols
layoutClass: gap-16

---

# Table of contents

You can use the `Toc` component to generate a table of contents for your slides:

```html
<Toc minDepth="1" maxDepth="1" />
```

The title will be inferred from your slide content, or you can override it with `title` and `level` in your frontmatter.

::right::

<Toc text-sm minDepth="1" maxDepth="2" />

---

layout: image-right
image: https://cover.sli.dev

---

# Code

Use code snippets and get the highlighting directly, and even types hover!

```ts {all|5|7|7-8|10|all} twoslash
// TwoSlash enables TypeScript hover information
// and errors in markdown code blocks
// More at https://shiki.style/packages/twoslash

import { computed, ref } from "vue";

const count = ref(0);
const doubled = computed(() => count.value * 2);

doubled.value = 2;
```

<arrow v-click="[4, 5]" x1="350" y1="310" x2="195" y2="334" color="#953" width="2" arrowSize="1" />

<!-- This allow you to embed external code blocks -->

<<< @/snippets/external.ts#snippet

<!-- Footer -->

[Learn more](https://sli.dev/features/line-highlighting)

<!-- Inline style -->
<style>
.footnotes-sep {
  @apply mt-5 opacity-10;
}
.footnotes {
  @apply text-sm opacity-75;
}
.footnote-backref {
  display: none;
}
</style>

<!--
Notes can also sync with clicks

[click] This will be highlighted after the first click

[click] Highlighted with `count = ref(0)`

[click:3] Last click (skip two clicks)
-->

---

## level: 2

# Shiki Magic Move

Powered by [shiki-magic-move](https://shiki-magic-move.netlify.app/), Slidev supports animations across multiple code snippets.

Add multiple code blocks and wrap them with <code>````md magic-move</code> (four backticks) to enable the magic move. For example:

````md magic-move {lines: true}
```ts {*|2|*}
// step 1
const author = reactive({
  name: "John Doe",
  books: [
    "Vue 2 - Advanced Guide",
    "Vue 3 - Basic Guide",
    "Vue 4 - The Mystery",
  ],
});
```

```ts {*|1-2|3-4|3-4,8}
// step 2
export default {
  data() {
    return {
      author: {
        name: "John Doe",
        books: [
          "Vue 2 - Advanced Guide",
          "Vue 3 - Basic Guide",
          "Vue 4 - The Mystery",
        ],
      },
    };
  },
};
```

```ts
// step 3
export default {
  data: () => ({
    author: {
      name: "John Doe",
      books: [
        "Vue 2 - Advanced Guide",
        "Vue 3 - Basic Guide",
        "Vue 4 - The Mystery",
      ],
    },
  }),
};
```

Non-code blocks are ignored.

```vue
<!-- step 4 -->
<script setup>
const author = {
  name: "John Doe",
  books: [
    "Vue 2 - Advanced Guide",
    "Vue 3 - Basic Guide",
    "Vue 4 - The Mystery",
  ],
};
</script>
```
````

---

# Components

<div grid="~ cols-2 gap-4">
<div>

You can use Vue components directly inside your slides.

We have provided a few built-in components like `<Tweet/>` and `<Youtube/>` that you can use directly. And adding your custom components is also super easy.

```html
<Counter :count="10" />
```

<!-- ./components/Counter.vue -->
<Counter :count="10" m="t-4" />

Check out [the guides](https://sli.dev/builtin/components.html) for more.

</div>
<div>

```html
<Tweet id="1390115482657726468" />
```

<Tweet id="1390115482657726468" scale="0.65" />

</div>
</div>

<!--
Presenter note with **bold**, *italic*, and ~~striked~~ text.

Also, HTML elements are valid:
<div class="flex w-full">
  <span style="flex-grow: 1;">Left content</span>
  <span>Right content</span>
</div>
-->

---

## class: px-20

# Themes

Slidev comes with powerful theming support. Themes can provide styles, layouts, components, or even configurations for tools. Switching between themes by just **one edit** in your frontmatter:

<div grid="~ cols-2 gap-2" m="t-2">

```yaml
---
theme: default
---
```

```yaml
---
theme: seriph
---
```

<img border="rounded" src="https://github.com/slidevjs/themes/blob/main/screenshots/theme-default/01.png?raw=true" alt="">

<img border="rounded" src="https://github.com/slidevjs/themes/blob/main/screenshots/theme-seriph/01.png?raw=true" alt="">

</div>

Read more about [How to use a theme](https://sli.dev/guide/theme-addon#use-theme) and
check out the [Awesome Themes Gallery](https://sli.dev/resources/theme-gallery).

---

# Clicks Animations

You can add `v-click` to elements to add a click animation.

<div v-click>

This shows up when you click the slide:

```html
<div v-click>This shows up when you click the slide.</div>
```

</div>

<br>

<v-click>

The <span v-mark.red="3"><code>v-mark</code> directive</span>
also allows you to add
<span v-mark.circle.orange="4">inline marks</span>
, powered by [Rough Notation](https://roughnotation.com/):

```html
<span v-mark.underline.orange>inline markers</span>
```

</v-click>

<div mt-20 v-click>

[Learn more](https://sli.dev/guide/animations#click-animation)

</div>

---

# Motions

Motion animations are powered by [@vueuse/motion](https://motion.vueuse.org/), triggered by `v-motion` directive.

```html
<div
  v-motion
  :initial="{ x: -80 }"
  :enter="{ x: 0 }"
  :click-3="{ x: 80 }"
  :leave="{ x: 1000 }"
>
  Slidev
</div>
```

<div class="w-60 relative">
  <div class="relative w-40 h-40">
    <img
      v-motion
      :initial="{ x: 800, y: -100, scale: 1.5, rotate: -50 }"
      :enter="final"
      class="absolute inset-0"
      src="https://sli.dev/logo-square.png"
      alt=""
    />
    <img
      v-motion
      :initial="{ y: 500, x: -100, scale: 2 }"
      :enter="final"
      class="absolute inset-0"
      src="https://sli.dev/logo-circle.png"
      alt=""
    />
    <img
      v-motion
      :initial="{ x: 600, y: 400, scale: 2, rotate: 100 }"
      :enter="final"
      class="absolute inset-0"
      src="https://sli.dev/logo-triangle.png"
      alt=""
    />
  </div>

  <div
    class="text-5xl absolute top-14 left-40 text-[#2B90B6] -z-1"
    v-motion
    :initial="{ x: -80, opacity: 0}"
    :enter="{ x: 0, opacity: 1, transition: { delay: 2000, duration: 1000 } }">
    Slidev
  </div>
</div>

<!-- vue script setup scripts can be directly used in markdown, and will only affects current page -->
<script setup lang="ts">
const final = {
  x: 0,
  y: 0,
  rotate: 0,
  scale: 1,
  transition: {
    type: 'spring',
    damping: 10,
    stiffness: 20,
    mass: 2
  }
}
</script>

<div
  v-motion
  :initial="{ x:35, y: 30, opacity: 0}"
  :enter="{ y: 0, opacity: 1, transition: { delay: 3500 } }">

[Learn more](https://sli.dev/guide/animations.html#motion)

</div>

---

# LaTeX

LaTeX is supported out-of-box. Powered by [KaTeX](https://katex.org/).

<div h-3 />

Inline $\sqrt{3x-1}+(1+x)^2$

Block

$$
{1|3|all}
\begin{aligned}
\nabla \cdot \vec{E} &= \frac{\rho}{\varepsilon_0} \\
\nabla \cdot \vec{B} &= 0 \\
\nabla \times \vec{E} &= -\frac{\partial\vec{B}}{\partial t} \\
\nabla \times \vec{B} &= \mu_0\vec{J} + \mu_0\varepsilon_0\frac{\partial\vec{E}}{\partial t}
\end{aligned}
$$

[Learn more](https://sli.dev/features/latex)

---

# Diagrams

You can create diagrams / graphs from textual descriptions, directly in your Markdown.

<div class="grid grid-cols-4 gap-5 pt-4 -mb-6">

```mermaid {scale: 0.5, alt: 'A simple sequence diagram'}
sequenceDiagram
    Alice->John: Hello John, how are you?
    Note over Alice,John: A typical interaction
```

```mermaid {theme: 'neutral', scale: 0.8}
graph TD
B[Text] --> C{Decision}
C -->|One| D[Result 1]
C -->|Two| E[Result 2]
```

```mermaid
mindmap
  root((mindmap))
    Origins
      Long history
      ::icon(fa fa-book)
      Popularisation
        British popular psychology author Tony Buzan
    Research
      On effectiveness<br/>and features
      On Automatic creation
        Uses
            Creative techniques
            Strategic planning
            Argument mapping
    Tools
      Pen and paper
      Mermaid
```

```plantuml {scale: 0.7}
@startuml

package "Some Group" {
  HTTP - [First Component]
  [Another Component]
}

node "Other Groups" {
  FTP - [Second Component]
  [First Component] --> FTP
}

cloud {
  [Example 1]
}

database "MySql" {
  folder "This is my folder" {
    [Folder 3]
  }
  frame "Foo" {
    [Frame 4]
  }
}

[Another Component] --> [Example 1]
[Example 1] --> [Folder 3]
[Folder 3] --> [Frame 4]

@enduml
```

</div>

Learn more: [Mermaid Diagrams](https://sli.dev/features/mermaid) and [PlantUML Diagrams](https://sli.dev/features/plantuml)

---

foo: bar
dragPos:
square: 691,32,167,\_,-16

---
dragPos:
  square: 0,-422,0,0
---

# Draggable Elements

Double-click on the draggable elements to edit their positions.

<br>

###### Directive Usage

```md
<img v-drag="'square'" src="https://sli.dev/logo.png">
```

<br>

###### Component Usage

```md
<v-drag text-3xl>
  <div class="i-carbon:arrow-up" />
  Use the `v-drag` component to have a draggable container!
</v-drag>
```

<v-drag pos="663,206,261,_,-15">
  <div text-center text-3xl border border-main rounded>
    Double-click me!
  </div>
</v-drag>

<img v-drag="'square'" src="https://sli.dev/logo.png">

###### Draggable Arrow

```md
<v-drag-arrow two-way />
```

<v-drag-arrow pos="67,452,253,46" two-way op70 />

---

src: ./pages/imported-slides.md
hide: false

---


---

# Monaco Editor

Slidev provides built-in Monaco Editor support.

Add `{monaco}` to the code block to turn it into an editor:

```ts {monaco}
import { ref } from "vue";
import { emptyArray } from "./external";

const arr = ref(emptyArray(10));
```

Use `{monaco-run}` to create an editor that can execute the code directly in the slide:

```ts {monaco-run}
import { version } from "vue";
import { emptyArray, sayHello } from "./external";

sayHello();
console.log(`vue ${version}`);
console.log(
  emptyArray<number>(10).reduce(
    (fib) => [...fib, fib.at(-1)! + fib.at(-2)!],
    [1, 1]
  )
);
```

---

layout: center
class: text-center

---

# Learn More

[Documentation](https://sli.dev) · [GitHub](https://github.com/slidevjs/slidev) · [Showcases](https://sli.dev/resources/showcases)

<PoweredBySlidev mt-10 />
