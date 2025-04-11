---
title: Ejemplo Master-doc para usar con Vibe Coding
date: 2025-04-07
author: Ray Dalio
description: Esto es un híbdirdo entre un Product Requirements Document (PRD), especificaiones técnicas y guías de implementación.
---

# RecipeGenius

- **Versión:** 1.0
- **Fecha:** 10 de abril de 2025
- **Autor:** [Fabián Acuña Hernández](https://fablab.blog) + Gemini 2.5 Pro Preview 03-25
- **Última actualización:** 10 de abril de 2025

## Índice

1.  Introducción
2.  Objetivos
3.  Flujo de Usuario y Funcionalidades Principales
4.  Requisitos Funcionales (RF)
5.  Requisitos No Funcionales (RNF)
6.  Diseño y Experiencia de Usuario (UX)
7.  Modelo de Datos
8.  Pila Tecnológica y Arquitectura
9.  Estructura de Carpetas del Proyecto (Next.js 15)
10. Modelo de Negocio
11. Variables de Entorno Requeridas (.env)
12. Métricas de Éxito (KPIs)
13. Consideraciones Futuras
14. Anexos

---

## 1. Introducción

*   **1.1 Propósito:** RecipeGenius es una plataforma web que utiliza inteligencia artificial (IA) para generar recetas creativas y planes de comidas personalizados basados en los ingredientes disponibles del usuario, sus preferencias dietéticas, restricciones y objetivos nutricionales. Su objetivo es combatir el desperdicio de alimentos, facilitar la planificación de comidas y inspirar a los usuarios en la cocina, ofreciendo sugerencias inteligentes y adaptadas.
*   **1.2 Alcance:** Este documento define los requisitos para la Versión 1.0 de RecipeGenius. El alcance incluye:
    *   Landing Page pública.
    *   Gestión de usuarios registrados (vía Clerk) y sesiones de invitados.
    *   Funcionalidad de entrada de ingredientes disponibles (texto libre, selección de lista predefinida).
    *   Entrada de preferencias (dietas, alergias, tipo de cocina, tiempo disponible, nivel de dificultad).
    *   Generación base gratuita de una receta única basada en los inputs usando Gemini 2.0 Flash.
    *   Funcionalidad premium de generación de Plan de Comidas Semanal (requiere registro y créditos) usando agentes de IA especializados (equilibrio nutricional, variedad, uso eficiente de ingredientes) coordinados por un Orquestador Estratégico.
    *   Sistema de créditos (compra y uso) integrado con una pasarela de pago.
    *   Interfaz para visualizar y opcionalmente ajustar el plan generado.
    *   Generación y descarga del Plan de Comidas en formato PDF/Markdown.
    *   Dashboard para usuarios registrados (historial de recetas/planes guardados, gestión de créditos).
    *   Implementación utilizando la pila tecnológica y estructura de carpetas especificadas.
*   **1.3 Fuera del Alcance (v1.0):** Aplicaciones móviles nativas, generación automática de listas de compras, integración con servicios de entrega de comestibles, análisis nutricional detallado por micronutriente, seguimiento de inventario de despensa, carga de recetas existentes para análisis, soporte multi-idioma inicial, streaming de respuestas de IA.
*   **1.4 Audiencia Objetivo:** Cocineros caseros de todos los niveles (principiantes a experimentados), individuos y familias que buscan simplificar la planificación de comidas, personas con restricciones dietéticas o alergias, y cualquiera interesado en reducir el desperdicio de alimentos y descubrir nuevas ideas culinarias.

## 2. Objetivos

*   **2.1 Objetivos de Negocio:**
    *   Adquirir una base de usuarios inicial a través de la oferta de generación de receta única gratuita y un primer plan de comidas gratuito al registrarse.
    *   Generar ingresos sostenibles mediante un modelo de negocio basado en la compra de créditos para la generación de planes de comida premium.
    *   Posicionar a RecipeGenius como una herramienta innovadora y útil para la planificación de comidas asistida por IA.
    *   Lograr una tasa de conversión saludable de usuarios invitados/gratuitos a usuarios de pago (compradores de créditos).
*   **2.2 Objetivos del Producto:**
    *   Proporcionar sugerencias de recetas y planes de comida relevantes, creativos y factibles basados en las entradas del usuario.
    *   Entregar planes de comida equilibrados, variados y que optimicen el uso de los ingredientes indicados.
    *   Ofrecer una experiencia de usuario (UX) fluida, intuitiva, inspiradora y eficiente.
    *   Garantizar la máxima seguridad y privacidad de los datos de preferencias y perfiles de los usuarios.
    *   Asegurar la integración fiable y eficiente de los modelos de IA (Gemini) y otros servicios de terceros (Clerk, Supabase, R2 - para futuras imágenes?, Pasarela de Pago, Upstash).
    *   Equilibrar la automatización de la IA con la capacidad del usuario para personalizar y ajustar los resultados.

## 3. Flujo de Usuario y Funcionalidades Principales

*   **3.1 Flujo de Usuario Principal (Ver Anexo A: Diagrama de Flujo Adaptado)**
    1.  **Acceso Inicial (Landing Page):** Presentación del servicio, opciones de Sign In/Sign Up (Clerk) o Continuar como Invitado.
    2.  **Entrada de Datos:**
        *   Ingreso de Ingredientes Disponibles (campo de texto, autocompletado/sugerencias opcionales).
        *   Selección/Ingreso de Preferencias: Dieta (vegetariana, vegana, etc.), Alergias, Tipo de Cocina preferido, Tiempo estimado por comida, Nivel de dificultad deseado.
    3.  **Generación de Receta Base (Gratuita/Disponible para todos):**
        *   El Orquestador Estratégico de IA gestiona el proceso.
        *   Se ejecuta: Generación de 1 Receta compatible con inputs usando Gemini 2.0 Flash.
        *   Visualización de la receta generada (ingredientes, pasos, etc.).
    4.  **Generación de Plan de Comidas Premium (Requiere Créditos y Registro):**
        *   *Check de Usuario:* Si es invitado, se le pide Registrarse/Iniciar Sesión (vía Clerk).
        *   *Check de Créditos:* Se verifica si el usuario registrado tiene créditos (`> 0`). Si no, se le dirige a la sección de compra (`/billing`).
        *   *Consumo de Crédito:* Se descuenta 1 crédito al iniciar la generación del plan.
        *   *Proceso IA Premium:* Agentes especializados (Equilibrio Nutricional, Variedad Culinaria, Optimización de Ingredientes) actúan para crear un plan semanal.
        *   *Generación de Borrador:* Se presenta un borrador del Plan de Comidas Semanal (ej. 7 cenas) en un formato claro (Markdown/JSON).
    5.  **Revisión y Ajuste (Opcional):** Interfaz para que el usuario revise el plan y potencialmente solicite reemplazos o ajustes menores (funcionalidad limitada en v1.0).
    6.  **Finalización:**
        *   Generación del Plan de Comidas final.
        *   Opciones: Ver en pantalla, Descargar como PDF/Markdown, Guardar en la cuenta (usuarios registrados).
        *   Redirección al Dashboard para usuarios registrados.

*   **3.2 Funcionalidades Detalladas:**
    *   **Gestión de Usuarios (Clerk):** Registro, Inicio/Cierre de Sesión, Gestión de Perfil (preferencias guardadas). Gestión interna de sesiones de invitados. Dashboard de usuario con historial de recetas/planes, créditos restantes y enlace a compra. Migración de datos de invitado a usuario registrado.
    *   **Entrada de Ingredientes y Preferencias:** Formularios validados, potencialmente con autocompletado para ingredientes comunes y selectores claros para preferencias.
    *   **Sistema de IA (Orquestador + Gemini 2.0 Flash):** Implementación del Orquestador para coordinar llamadas a Gemini. Desarrollo de los prompts y lógica para la Generación de Receta Base y los Agentes de Planificación Premium.
    *   **Sistema de Créditos y Pagos:** Lógica para asignar crédito inicial, verificar, descontar créditos. Integración con Pasarela de Pago [Definir Proveedor] para compra de paquetes (10 créditos/$5 USD). Tabla `PAYMENT`. Interfaz clara de créditos y compra.
    *   **Visualización y Salida:** Presentación clara de recetas y planes. Generador de PDF/Markdown a partir del contenido final. Funcionalidad de descarga. Almacenamiento estructurado en tabla `REQUEST` (ver Modelo de Datos).
    *   **Rate Limiting (Upstash):** Implementación para prevenir abuso en endpoints críticos (generación de recetas/planes).

## 4. Requisitos Funcionales (RF)

*   **RF-AUTH-001:** El sistema utilizará Clerk para la autenticación y gestión de usuarios registrados.
*   **RF-AUTH-002:** El sistema permitirá a los usuarios operar como invitados para la generación de receta base.
*   **RF-AUTH-003:** El sistema asignará 1 crédito gratuito inicial a cada nuevo usuario al registrarse en la base de datos local (`USER` table).
*   **RF-INPUT-001:** El sistema permitirá introducir una lista de ingredientes disponibles mediante texto libre.
*   **RF-INPUT-002:** El sistema permitirá seleccionar/introducir preferencias: dieta, alergias, tipo de cocina, tiempo, dificultad.
*   **RF-AI-001:** El sistema generará una Receta Base (ingredientes, pasos) usando Gemini 2.0 Flash basada en los inputs y la mostrará al usuario.
*   **RF-AI-002:** El sistema requerirá registro/inicio de sesión y créditos disponibles (>0 en la tabla `USER`) para iniciar la Generación de Plan de Comidas Premium.
*   **RF-AI-003:** El sistema verificará y descontará 1 crédito de la cuenta del usuario en la tabla `USER` antes de ejecutar la Generación de Plan Premium.
*   **RF-AI-004:** El sistema utilizará agentes de IA (Gemini) coordinados por un orquestador para la Generación de Plan Premium (Equilibrio, Variedad, Optimización).
*   **RF-VIEW-001:** El sistema presentará el Plan de Comidas generado en un formato claro y estructurado.
*   **RF-OUTPUT-001:** El sistema permitirá la descarga del Plan de Comidas generado en formato PDF o Markdown.
*   **RF-STORE-001:** El sistema guardará los detalles de la solicitud (inputs, receta generada, plan generado - si aplica) en la tabla `REQUEST` para usuarios registrados que elijan guardar.
*   **RF-STORE-002:** El sistema asociará las solicitudes procesadas a `user_id` (registrados) o `guest_id`.
*   **RF-BILL-001:** El sistema implementará un modelo de créditos (1 crédito = 1 generación de plan premium).
*   **RF-BILL-002:** El sistema permitirá a usuarios registrados comprar paquetes de créditos (10 por $5 USD) a través de una pasarela de pago integrada.
*   **RF-BILL-003:** El sistema registrará las transacciones de pago en la tabla `PAYMENT`.
*   **RF-BILL-004:** El sistema mostrará claramente los créditos disponibles al usuario registrado (Dashboard, Navbar).
*   **RF-DASH-001:** El sistema proporcionará un Dashboard a usuarios registrados para ver Recetas/Planes guardados, créditos y enlace a compra.
*   **RF-RATE-001:** El sistema implementará rate limiting usando Upstash en operaciones clave para prevenir abuso.

## 5. Requisitos No Funcionales (RNF)

*   **RNF-PERF-001 (Rendimiento):** Carga inicial de páginas < 3 segundos. Generación de Receta Base < 20 segundos. Generación de Plan Premium < 90 segundos promedio (con feedback visual claro).
*   **RNF-SCAL-001 (Escalabilidad):** La arquitectura (Vercel, Supabase, R2, Upstash, Clerk) deberá soportar crecimiento de usuarios y carga de trabajo sin degradación significativa del rendimiento.
*   **RNF-SECU-001 (Seguridad):** Autenticación segura (Clerk). Protección OWASP Top 10. Row Level Security (RLS) en Supabase. Almacenamiento seguro (DB encriptada en reposo). Gestión segura de API Keys y secrets (variables de entorno). HTTPS obligatorio.
*   **RNF-USAB-001 (Usabilidad):** Interfaz intuitiva, limpia y visualmente atractiva (basada en Shadcn/UI). Flujo de usuario claro y fácil de seguir. Feedback adecuado durante operaciones largas. Diseño responsive (prioridad escritorio y móvil). Textos claros y concisos.
*   **RNF-RELI-001 (Fiabilidad):** Alta disponibilidad (>99.9%). Manejo robusto de errores (UI, API, IA). Backups automáticos de la base de datos (gestionado por Supabase).
*   **RNF-MAIN-001 (Mantenibilidad):** Código limpio, bien documentado y siguiendo las mejores prácticas (TypeScript, Next.js, React). Estructura de carpetas por características (Sección 9). Uso consistente de ORM (Drizzle). Principios SOLID.
*   **RNF-PRIV-001 (Privacidad):** Cumplimiento de normativas de protección de datos. Política de privacidad clara. Tratamiento seguro y confidencial de preferencias dietéticas y alergias.

## 6. Diseño y Experiencia de Usuario (UX)

*   Adherencia estricta a los componentes y principios de diseño de Shadcn/UI y Tailwind 4 para consistencia.
*   Uso coherente de Lucid Icons.
*   Priorizar la facilidad en la entrada de ingredientes y preferencias.
*   Presentación clara y apetecible de las recetas y planes generados.
*   Minimizar la fricción en el flujo, especialmente en la compra/uso de créditos.
*   Proporcionar indicadores de progreso visuales claros durante la generación IA.
*   El diseño general debe proyectar inspiración culinaria, facilidad, inteligencia y confianza.

## 7. Modelo de Datos (Ver Anexo B: Diagrama ER Adaptado)

*   **USER:** `id` (**uuid**, PK), `clerk_id` (text, unique, not null), `credits` (integer, default 1, not null), `created_at`, `updated_at`.
*   **GUEST:** `id` (**uuid**, PK), `user_id` (**uuid**, FK to USER, nullable), `created_at`, `updated_at`.
*   **REQUEST:** `id` (**uuid**, PK), `user_id` (**uuid**, FK to USER, nullable), `guest_id` (**uuid**, FK to GUEST, nullable), `ingredients_input` (text), `preferences_input` (jsonb), `generated_recipe_markdown` (text), `generated_recipe_created_at` (timestamp), `recipe_agent_outputs` (jsonb), `generated_plan_markdown` (text), `generated_plan_created_at` (timestamp), `plan_agent_outputs` (jsonb), `created_at`, `updated_at`. (Constraint: CHECK (`user_id` IS NOT NULL OR `guest_id` IS NOT NULL)).
*   **PAYMENT:** `id` (**uuid**, PK), `user_id` (**uuid**, FK to USER, not null), `amount` (integer - en centavos), `currency` (text - e.g., 'USD'), `provider_payment_id` (text, nullable), `status` (text - e.g., 'succeeded', 'pending', 'failed'), `created_at`, `updated_at`.
*   **Relaciones:** USER 1..N REQUEST, GUEST 1..N REQUEST, USER 1..N PAYMENT, GUEST 1..1 USER (opcional, en migración).

*(**Nota:** Al implementar esto en la base de datos, típicamente configurarías las columnas `id` de tipo `uuid` para tener un valor predeterminado generado automáticamente, como `DEFAULT gen_random_uuid()` en PostgreSQL).*

## 8. Pila Tecnológica y Arquitectura

*   **8.1 Pila Tecnológica Principal:**
    *   Frontend: Next.js 15, React 19, TypeScript, Tailwind 4
    *   UI: Shadcn/UI, Radix UI, Lucid Icons
    *   Base de Datos: Supabase (PostgreSQL)
    *   ORM: Drizzle ORM con Drizzle Zod
    *   Autenticación: Clerk
    *   IA - Generación: Gemini 2.0 Flash API (via Google AI SDK)
    *   Rate Limiting: Upstash (@upstash/ratelimit)
    *   Almacenamiento de Archivos: Cloudflare R2 (Potencialmente para futuras imágenes de recetas o inputs)
    *   Pasarela de Pago: [Definir Proveedor - e.g., Stripe]
    *   Despliegue: Vercel
*   **8.2 Arquitectura General:** Aplicación web serverless monolítica (modularizada por características) desplegada en Vercel. Utiliza Supabase como BaaS (DB), Clerk para autenticación, APIs de Google AI, R2 (opcional inicial), y Upstash para rate limiting.
*   **8.3 Estrategia de Comunicación Servidor-Cliente:**
    *   **Prioridad a Server Actions:** Método principal para comunicación C/S, mutaciones, lógica de negocio desde UI.
    *   **Uso de API Routes (Route Handlers):** Limitado a Webhooks externos (Pasarela de pago).
*   **8.4 Sincronización de Autenticación (Clerk <> Supabase DB):**
    *   **Enfoque Principal:** Evitar webhooks de Clerk. Usar `auth()` de `@clerk/nextjs` en servidor para datos de usuario. Creación/actualización de `USER` local bajo demanda/oportunista.
*   **8.5 Principios Arquitectónicos Clave:**
    *   Colocación por Características (`app/`).
    *   Separación de Responsabilidades (UI, Actions, Lib).
    *   Single Responsibility Principle (SRP).
    *   Don't Repeat Yourself (DRY) (lógica de créditos en `lib/`).
    *   Límites Claros entre características.

## 9. Estructura de Carpetas del Proyecto (Next.js 15)

*   **9.1 Metodología:** Colocación por Características dentro de `app/`.
*   **9.2 Estructura Detallada:**
    ```
    recipegenius-app/
    ├── app/                       # App Router Root
    │   ├── recipe-generator/      # FEATURE: Core Recipe/Plan Generation
    │   │   ├── _components/
    │   │   ├── actions.ts
    │   │   ├── page.tsx           # Main input/output page
    │   │   └── [requestId]/       # Maybe for viewing saved requests
    │   │       └── ...
    │   ├── dashboard/             # FEATURE: User Dashboard
    │   │   └── ...
    │   ├── auth/                  # FEATURE: Authentication (Clerk)
    │   │   └── ...
    │   ├── billing/               # FEATURE: Credit Purchase
    │   │   └── ... (Similar a CVFácil)
    │   │
    │   ├── (root)/                # Root level app files
    │   │   ├── layout.tsx
    │   │   └── page.tsx           # Landing Page
    │
    ├── lib/                       # SHARED Logic & Central Configuration
    │   ├── db/
    │   │   ├── index.ts
    │   │   └── schema.ts  <-- FUENTE DE VERDAD PARA TIPOS DE DB
    │   ├── ai/                    # AI interaction logic, prompts
    │   ├── storage/               # R2 interactions (if used)
    │   ├── utils/
    │   ├── auth.ts
    │   ├── payment.ts
    │   └── rate-limit.ts
    │
    ├── components/                # SHARED UI Components
    │   ├── ui/                    # Base Shadcn/UI components
    │   └── shared/                # Custom reusable components
    │
    ├── public/
    ├── styles/
    ├── .env.example
    ├── .env.local
    ├── next.config.mjs
    ├── tsconfig.json
    └── package.json
    ```
*   **9.3 Convenciones Clave:** `_components`, `actions.ts`, `lib/`, `components/`.
*   **9.4 Gestión de Tipos (TypeScript):** Priorizar tipos inferidos por Drizzle desde `lib/db/schema.ts`. Evitar tipos manuales duplicados.

## 10. Modelo de Negocio

*   **Plan Gratuito/Inicial:** Generación de receta única ilimitada (sujeto a rate limiting). **1 crédito gratuito** al registrarse para **1 generación de plan de comidas premium**.
*   **Compra de Créditos:** Paquetes de **10 créditos por $5 USD** para usuarios registrados.
*   **Uso de Créditos:** **1 crédito** consumido por cada **Generación de Plan de Comidas Premium** ejecutada.

## 11. Variables de Entorno Requeridas (.env)

(La lista sería idéntica a la de CVFácil, excepto que `MISTRAL_API_KEY` podría no ser necesaria inicialmente si no se implementa OCR de imágenes de ingredientes. Las claves de Gemini, Clerk, Supabase, R2, Pasarela de Pago y Upstash serían requeridas).

*   **Google AI (Gemini):** `GEMINI_API_KEY`
*   **Clerk:** `NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY`, `CLERK_SECRET_KEY`, `NEXT_PUBLIC_CLERK_SIGN_IN_URL`, `NEXT_PUBLIC_CLERK_SIGN_UP_URL`
*   **Cloudflare R2:** `CLOUDFLARE_ACCOUNT_ID`, `CLOUDFLARE_ACCESS_KEY_ID`, `CLOUDFLARE_SECRET_ACCESS_KEY`, `CLOUDFLARE_BUCKET_NAME`, `CLOUDFLARE_ENDPOINT_URL`, `CLOUDFLARE_PUBLIC_URL` (Si se usa)
*   **Supabase:** `DATABASE_URL`, `NEXT_PUBLIC_SUPABASE_URL`, `NEXT_PUBLIC_SUPABASE_ANON_KEY`, `SUPABASE_SERVICE_ROLE_KEY`, `SUPABASE_JWT_SECRET`
*   **Otros:** `STRIPE_SECRET_KEY`, `STRIPE_WEBHOOK_SECRET`, etc. (Pasarela Pago), `UPSTASH_REDIS_REST_URL`, `UPSTASH_REDIS_REST_TOKEN` (Upstash)

## 12. Métricas de Éxito (KPIs)

*   **Adquisición:** Nº Usuarios Registrados, Nº Sesiones de Invitados.
*   **Activación/Conversión:** Tasa de Conversión Invitado -> Registrado, Nº Recetas Base Generadas, Nº Planes Premium Generados (Créditos Usados).
*   **Retención/Engagement:** Usuarios Activos Mensuales (MAU), Nº Recetas/Planes Guardados, Tasa de Finalización Generación Plan.
*   **Monetización:** Nº Transacciones de Compra de Créditos, Ingresos Totales, ARPU, Tasa de Conversión a Pago.
*   **Satisfacción:** CSAT/NPS (vía encuestas), Tasa de Abandono (Churn).
*   **Rendimiento Técnico:** Tiempo Medio de Generación IA, Uptime (>99.9%), Tasa de Errores.

## 13. Consideraciones Futuras (Post-v1.0)

*   Generación Automática de Lista de Compras.
*   Integración con APIs de supermercados/entrega.
*   Análisis Nutricional Detallado.
*   Input de Ingredientes mediante Imagen (OCR con Mistral AI u otro).
*   Guardar Perfiles de Preferencias Múltiples.
*   Funcionalidades Comunitarias (compartir recetas/planes).
*   Modelos de suscripción.
*   Soporte Multi-idioma.
*   Mejoras PWA / App Móvil.

## 14. Anexos

*   **Anexo A:** Diagrama de Flujo de Usuario Principal (Adaptado para RecipeGenius - Referencia Externa).
*   **Anexo B:** Diagrama Entidad-Relación (ERD) del Modelo de Datos (Adaptado para RecipeGenius - Referencia Externa).

---

## Anexo A
flowchart TD
    %% Start & Authentication
    A([Inicio]) --> B[Landing Page]
    B --> C{¿Iniciar Sesión/Registrarse o Invitado?}
    C -->|Iniciar Sesión| D[Autenticación Clerk: Sign In]
    C -->|Registrarse| E[Autenticación Clerk: Sign Up]
    C -->|Continuar como Invitado| F[Sesión de Invitado]

    %% Input Collection
    subgraph InputSection[1. Entrada de Datos]
        G[Ingresar Ingredientes Disponibles]
        H[Seleccionar Preferencias (Dieta, Alergias, Tiempo, etc.)]
    end

    D --> G
    E --> G
    F --> G
    G --> H

    %% Base Recipe Generation (Free)
    subgraph BaseRecipeGen[2. Generación Receta Base (Gratuita)]
        I[Orquestador IA gestiona]
        J[IA genera 1 Receta (Gemini Flash)]
        K[Mostrar Receta Generada]
    end

    H --> I
    I --> J
    J --> K

    %% Premium Plan Generation Decision
    K --> L{¿Generar Plan Semanal? (Premium)}

    %% Premium Path
    subgraph PremiumPlanGen[3. Generación Plan Premium]
        M{Usuario Registrado y con Créditos (>0)?}
        N[Redirigir a Registro/Login o Compra Créditos (/billing)]
        O[Consumir 1 Crédito]
        P[Orquestador IA gestiona Agentes de Planificación (Equilibrio, Variedad, Optimización)]
        Q[IA Genera Borrador Plan Semanal]
        R[Mostrar Borrador del Plan]
        S{¿Ajustar Plan? (v1.0 - Opcional/Limitado)}
        T[Plan Final Generado]
    end

    L -- Sí --> M
    M -- No --> N
    N -- Tras Éxito --> O  %% Asume éxito tras registro/compra para simplificar diagrama
    M -- Sí --> O
    O --> P
    P --> Q
    Q --> R
    R --> S
    S -- Aceptar/Finalizar --> T

    %% Finalization Options
    subgraph Finalization[4. Finalización]
        U[Ver Plan en Pantalla]
        V[Descargar PDF/Markdown]
        W[Guardar en Cuenta (Solo Registrados)]
        X[Ir al Dashboard (Solo Registrados)]
    end

    T --> U
    T --> V
    T --> W
    W --> X

    %% End Paths
    L -- No --> Z1([Fin Flujo / Volver])
    V --> Z2([Fin Flujo])
    X --> Z3([Fin Flujo])

## Anexo B

---
description:
globs:
alwaysApply: true
---
	# Anexo B: Diagrama Entidad-Relación (ERD) del Modelo de Datos (Referencia al diagrama proporcionado externamente).

	erDiagram

    %% USERS
    USER {
        uuid id PK
        text clerk_id UK
        integer credits "default 1"
        timestamp created_at
        timestamp updated_at
    }

    %% GUESTS
    GUEST {
        uuid id PK
        uuid user_id FK "nullable"
        timestamp created_at
        timestamp updated_at
    }

    %% REQUESTS (Combines Recipe and Plan Generation)
    REQUEST {
        uuid id PK
        uuid user_id FK "nullable"
        uuid guest_id FK "nullable"
        text ingredients_input
        jsonb preferences_input
        text generated_recipe_markdown
        timestamp generated_recipe_created_at
        jsonb recipe_agent_outputs
        text generated_plan_markdown
        timestamp generated_plan_created_at
        jsonb plan_agent_outputs
        timestamp created_at
        timestamp updated_at
        CHECK "user_id IS NOT NULL OR guest_id IS NOT NULL"
    }

    %% PAYMENTS
    PAYMENT {
        uuid id PK
        uuid user_id FK
        integer amount "cents"
        text currency
        text provider_payment_id "nullable"
        text status
        timestamp created_at
        timestamp updated_at
    }

    %% RELATIONS
    USER ||--o{ REQUEST : generates
    GUEST ||--o{ REQUEST : generates
    USER ||--o{ PAYMENT : makes
    GUEST }o--|| USER : converts_to
