# Implementación de n8n en Docker con Base de datos Postgres y memoria Redis

## Descripción del caso

Este proyecto contiene un flujo de trabajo (workflow) de n8n diseñado para implementar un asistente virtual (agente) con el rol de "camarera" y que es la encargada de atender las consultas de los clientes de un restaurante de comida oriental en una ciudad de Perú.
El workflow se activa a través de un Webhook, integrado en la landing page del restaurante y que esta disponible 24/7. El asistente tiene información de la carta del restaurante con los precios actualizados al 2025.

## Estructura del proyecto

n8n-implementation/
├─ LICENSE                            #Licenciamiento Apache 2.0
├─ README.md                          #Este archivo
├─ client
│  └─ landing.html                    #Frontend del Agente IA
├─ data
│  └─ Carta_2025.csv                  #Datos de la carta
├─ docker-compose.yml                 #Archivo de despliegue en docker
├─ presentation
│  └─ SAT2025_Agentes_n8n_AIFoundry.pdf 
└─ workflow
   └─ Demo_ RAG_with_AIFoundry.json   #Workflow para importar en n8n


## Estructura del Workflow

El flujo de trabajo sigue una secuencia lógica de 5 pasos principales:
1. **Trigger**: Webhook recibe la solicitud (mensaje)
2. **Processing**: Nodo AI Agent en n8n.
3. **Itelligence**: Conexión con Azure Open AI.
4. **Action**: Uso de herramientas (Tools).
5. **Response**: Respuesta al cliente.


## Despliegue

1. Crear el archivo docker-compose.yml con las imagenes necesarias
```bash
version: '3.8'

services:
  postgres:
    image: postgres:16
    container_name: n8n_postgres_db
    restart: always
    ports:
      - "5432:5432"
    environment:
      - POSTGRES_USER=n8nuser
      - POSTGRES_PASSWORD=MyDbPassword
      - POSTGRES_DB=n8n
    volumes:
      - postgres_data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U n8nuser -d n8n"]
      interval: 5s
      timeout: 5s
      retries: 10

  redis:
    image: redis:latest
    container_name: n8n_redis
    restart: always
    volumes:
      - redis_data:/data

  n8n:
    image: n8nio/n8n:1.113.1
    container_name: n8n_app
    restart: always
    ports:
      - "5678:5678"
    environment:
      - DB_TYPE=postgresdb
      - DB_POSTGRESDB_HOST=postgres
      - DB_POSTGRESDB_PORT=5432
      - DB_POSTGRESDB_DATABASE=n8n
      - DB_POSTGRESDB_USER=n8nuser
      - DB_POSTGRESDB_PASSWORD=MyDbPassword
      - N8N_BASIC_AUTH_ACTIVE=true
      - N8N_BASIC_AUTH_USER=admin
      - N8N_BASIC_AUTH_PASSWORD=MyAdminPassword
      - N8N_HOST=n8n.mycustomserver.com
      - WEBHOOK_URL=https://n8n.mycustomserver.com/
      - VUE_APP_URL_BASE_API=https://n8n.mycustomserver.com/
      - GENERIC_TIMEZONE=America/Lima
      - N8N_EMAIL_FOOTER="Powered by MyCustomServer"
    depends_on:
      postgres:
        condition: service_healthy
    volumes:
      - n8n_data:/home/node/.n8n

volumes:
  postgres_data:
  n8n_data:
```
2. Ejecutar el despliegue
```shell
sudo docker-compose up -d
```

## Implementación en el Cliente (Frontend)

Para integrar el chatbot en tu página web debes utilizar las librerias de n8n y el siguiente código de ejemplo:
```javascript
  <script src="https://cdn.jsdelivr.net/npm/n8n-embedded-chat-interface@latest/output/index.js"></script>
	<n8n-embedded-chat-interface label="My AI Assistant" hostname="" open-on-start="false">
	</n8n-embedded-chat-interface>
	<link href="https://cdn.jsdelivr.net/npm/@n8n/chat/dist/style.css" rel="stylesheet" />
	<script type="module">
		import { createChat } from 'https://cdn.jsdelivr.net/npm/@n8n/chat/dist/chat.bundle.es.js';

		createChat({
			webhookUrl: 'https://n8n.mycustomserver.com/webhook/xxxxxx-xxxx-xxxx-xxxx-xxxxxx/chat',
			defaultLanguage: 'es',
			initialMessages: [
				'Hola! 👋',
				'Mi nombre es Alicia. ¿Como puedo ayudarte hoy?'
			],
			i18n: {
				es: {
					title: 'Hola! 👋',
					subtitle: "Estamos atendiendo 24/7.",
					footer: '',
					getStarted: 'Nueva conversación',
					inputPlaceholder: 'Ingresa tu pregunta..',
				},
			},
		});
	</script>
```

## Credenciales Requeridas

Para que el workflow funcione, se deben configurar las siguientes credenciales dentro de la interfaz de n8n:
- **Azure AI Foundry**: Para utilizar el LLM con modelo gpt-4o.

## Contribución

Autor: Marlon Leandro @ 2025
Perfil: https://www.linkedin.com/in/marlonleandro/