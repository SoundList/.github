<div align="center">

  <img src="https://media.giphy.com/media/v1.Y2lkPTc5MGI3NjExbm95aW1wZnF4aDN2ZnJ5bm95aW1wZnF4aDN2ZnJ5bm95aW1wZnF4aDN2ZnJ5/7lCJ3fD2d8O09/giphy.gif" width="100" />

  <h1>SoundList 🎧</h1>
  <h3>Distributed Social Music Platform powered by AI</h3>

  <p>
    <b>Microservicios • Event-Driven Architecture • Artificial Intelligence</b>
  </p>

  <p>
    <a href="https://soundlist-front-end.vercel.app/">
      <img src="https://img.shields.io/badge/LIVE_DEMO-b54640?style=for-the-badge&logo=vercel&logoColor=white" alt="Live Demo" />
    </a>
    <a href="#-arquitectura-del-sistema">
      <img src="https://img.shields.io/badge/VER_ARQUITECTURA-0f0706?style=for-the-badge&logo=mermaid&logoColor=white" alt="Architecture" />
    </a>
  </p>
</div>

---

## 💡 Sobre el Proyecto

**SoundList** no es solo una app de música; es una red social distribuida diseñada para melómanos. Permite a los usuarios reseñar canciones, crear listas colaborativas y recibir análisis de sentimientos generados por IA sobre sus opiniones.

El desafío principal no fue solo consumir la API de Spotify, sino orquestar un ecosistema de **Microservicios en .NET 8** capaz de manejar tareas pesadas (como el procesamiento de IA) sin bloquear la experiencia del usuario, utilizando un enfoque asíncrono con **RabbitMQ**.

### 🌟 Features Clave
* **Integración Musical:** Búsqueda y previsualización en tiempo real vía **Spotify & Deezer APIs**.
* **AI-Powered Insights:** Análisis de reseñas y generación de resúmenes utilizando **Google Vertex AI**.
* **Arquitectura Resiliente:** Comunicación asíncrona para evitar cuellos de botella.
* **Gateway Unificado:** Implementación de **YARP** (Yet Another Reverse Proxy) para enrutamiento seguro.

---

## 🏗️ Arquitectura del Sistema

Como **Tech Lead** del proyecto, diseñé esta arquitectura para asegurar el desacoplamiento de servicios y la escalabilidad.

```mermaid
graph TD
    %% --- ESTILOS DE ALTO CONTRASTE ---
    %% Usamos fondos claros con bordes fuertes y TEXTO NEGRO para máxima legibilidad
    classDef frontend fill:#E1F5FE,stroke:#0277BD,stroke-width:2px,color:#000000;
    classDef gateway fill:#FFF8E1,stroke:#FF8F00,stroke-width:2px,color:#000000;
    classDef service fill:#FFFFFF,stroke:#2E7D32,stroke-width:2px,color:#000000,rx:5,ry:5;
    classDef db fill:#F3E5F5,stroke:#7B1FA2,stroke-width:2px,stroke-dasharray: 5 5,color:#000000;
    classDef queue fill:#FFEBEE,stroke:#C62828,stroke-width:2px,color:#000000;
    classDef external fill:#FAFAFA,stroke:#616161,stroke-width:1px,stroke-dasharray: 5 5,color:#000000;

    %% --- NODOS PRINCIPALES ---
    Client["💻 Frontend Web App"]:::frontend
    YARP["🛡️ API Gateway (YARP)"]:::gateway
    Rabbit["🐰 RabbitMQ (Event Bus)"]:::queue
    DB[("🐘 PostgreSQL\n(Relacional + JSONB)")]:::db

    %% --- CONTEXTO DE MICROSERVICIOS ---
    subgraph Services ["🏗️ Ecosistema de Microservicios"]
        direction TB
        %% Definimos los nodos con texto negro explícito
        UserS["👤 UserService\n(Auth & Profiles)"]:::service
        SocialS["💬 SocialService\n(Reviews, Likes)"]:::service
        ContentS["🎵 ContentService\n(Songs & Albums)"]:::service
        AIS["🤖 AIService\n(ML.NET + Vertex AI)"]:::service
        ContentS ~~~ UserS ~~~ SocialS ~~~ AIS
    end

    %% --- CONTEXTO DE NUBE ---
    subgraph Cloud ["☁️ APIs Externas"]
        Spotify["🎧 Spotify API"]:::external
        Deezer["🎶 Deezer API"]:::external
        Vertex["🧠 Google Vertex AI"]:::external
    end

    %% --- CONEXIONES (FLUJO) ---
    
    %% 1. Entrada
    Client ==>|"1. HTTPS + JWT"| YARP
    YARP ==>|"Proxy"| UserS
    YARP ==>|"Proxy"| SocialS
    YARP ==>|"Proxy"| ContentS

    %% 2. Dependencias Externas
    ContentS -->|"Search Music"| Spotify
    ContentS -->|"Songs Preview"| Deezer
    AIS -->|"Generar Resumen"| Vertex

    %% 3. Comunicación Asíncrona (RabbitMQ)
    SocialS -.->|"2. Publica: ReviewCreated"| Rabbit
    Rabbit -.->|"3. Consume Evento"| AIS

    %% 4. Base de Datos (Conexiones más finas para no ensuciar)
    UserS --- DB
    SocialS --- DB
    ContentS --- DB
    AIS --- DB

    %% Truco visual para alinear (Links invisibles)
    UserS ~~~ AIS
